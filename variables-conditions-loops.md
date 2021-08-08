# Variables, Conditionals, Loops

Now that we know how ansible works, lets get familiar with more advanced options in ansible. By using these options complicated tasks seem a lot easier.

## variables

Just like any other Scripting or programming language we can use variable in ansible playbooks. Variables could store different values for different items. Variables help us to have  shorter and more readable playbooks. Imagine we want to apply  patches on hundreds  of servers, the only thing we need is single playbook with some variables  for all hundred servers! It's the variables that store information about different IP addresses, host names, username or passwords,... 

### Naming Variables

Variable names must start with a letter, and they can only contain letters, numbers, and underscores. The following table illustrates the difference between invalid and valid variable names.

| INVALID VARIABLE NAMES | VALID VARIABLE NAMES |
| :--- | :--- |
| web server | web\_server |
| remote.file | remote\_file |
| 1st file | file\_1, file1 |
| remoteserver$1 | remote\_server\_1, remote\_server1 |

### Defining Variables

Variables can be defined in a variety of places in an Ansible project. However, this can be simplified to three basic scope levels:

* **Global scope**: Variables set from the command line or Ansible configuration.
* **Play scope**: Variables set in the play and related structures.
* **Host scope**: Variables set on host groups and individual hosts by the inventory, fact gathering, or registered tasks.

If the same variable name is defined at more than one level, the level with the highest precedence wins. A narrow scope takes precedence over a wider scope: variables defined by the inventory are overridden by variables defined by the playbook, which are overridden by variables defined on the command line.

**Host scope:** We have already seen using variables when we talked about Ansible inventory files. As as   example lets write down a playbook to configure multiple firewall configuration. We want to make it reusable for some one else to change ports, for that lets move variables to the inventory file:

```text
#Sample inventory file with variables-inventory.txt
centos http_port=8080 snmp_port=161-162 internal_ip_range=192.168.100.0
```

```text
---
# sample firewall playbook  firewall-playbook.yaml

-
  name: Set Firewall Configurations
  hosts: centos
  become: true
  tasks:
    -  firewalld:
         service: https
         permanent: true
         state: enabled

    -  firewalld:
         port: "{{ http_port }}/tcp"
         permanent: true
         state: disabled

    -  firewalld:
         port: "{{ snmp_port }}/udp"
         permanent: true
         state: disabled

    -  firewalld:
         source: "{{ internal_ip_range }}/24"
         permanent: true
         zone: internal
         state: enabled
```

```text
[user1@controller demo-var]$  ansible-playbook -i inventory.txt  firewall-playbook.yaml
```

**play scope:** Ansible playbook supports defining the variable in two forms, Either a Single liner variable declaration like we do in any common programming languages  or as a separate file with full of variables and values like a properties file.

* **`vars`** to define inline variables within the playbook
* **`vars_files`** to import files with variables

 Lets repeat previous example by  moving  variables in to the playbook ,It can be done with `vars` like this:

```text
---

#sample firewall playbook with vars firewall-playbook.yaml

-
  name: Set Firewall Configurations
  hosts: centos
  become: true
  vars:
    http_port: 8080
    snmp_port: 161-162
    internal_ip_range: 192.168.100.0
  tasks:
    -  firewalld:
         service: https
         permanent: true
         state: enabled

    -  firewalld:
         port: "{{ http_port }}/tcp"
         permanent: true
         state: disabled

    -  firewalld:
         port: "{{ snmp_port }}/udp"
         permanent: true
         state: disabled

    -  firewalld:
         source: "{{ internal_ip_range }}/24"
         permanent: true
         zone: internal
         state: enabled
```

```text
[user1@controller demo-var]$ ansible-playbook  firewall-playbook.yaml
```

If you want to keep the variables in a separate file and import it with `vars_files`You have to first save the variables and values in the same format you have written in the playbook and the file can later be imported using vars\_files like this:

```text
# vars.yaml
http_port: 8080
snmp_port: 161-162
internal_ip_range: 192.168.100.0
```

```text
---

#sample firewall playbook with var_files firewall-playbook.yaml

-
  name: Set Firewall Configurations
  hosts: centos
  become: true
  vars_files:
    - vars.yaml
  tasks:
    -  firewalld:
         service: https
         permanent: true
         state: enabled

    -  firewalld:
         port: "{{ http_port }}/tcp"
         permanent: true
         state: disabled

    -  firewalld:
         port: "{{ snmp_port }}/udp"
         permanent: true
         state: disabled

    -  firewalld:
         source: "{{ internal_ip_range }}/24"
         permanent: true
         zone: internal
         state: enabled
```

```text
[user1@controller demo-var]$ ansible-playbook  firewall-playbook.yaml
```

{% hint style="info" %}
**Jinja2 templating :**  the format {{ }} we are using to use variables is called Jinja2 templating. Be careful about **Quotes** :

* ~~**source: {{ http\_port }}**~~
* **source: "{{ http\_port }}"**
* **source: " Somthing {{ http\_port }} Somthing "**
{% endhint %}

### Ansible facts

 Ansible _facts_ are data gathered about target nodes and returned back to controller nodes. Ansible facts are stored in JSON format and are used to make important decisions about tasks based on their statistics. Facts are in an **ansible\_facts** variable, which is managed by Ansible Engine. Ansible facts play a major role in syncing with hosts in accordance with _real-time data_.

{% hint style="success" %}
Normally, every play runs the setup module automatically before the first task in order to gather facts. This is reported as the Gathering Facts task in Ansible 2.3 and later, or simply as setup in older versions of Ansible. By default, you do not need to have a task to run setup in your play. It is normally run automatically for you.
{% endhint %}

Some of the facts gathered for a managed host might include:

* The hostname
* The kernel version
* The network interfaces
* The IP addresses
* The version of the operating system
* Various environment variables
* The number of CPUs
* The available or free memory
* The available disk space

### Accessing the facts

Ansible facts use the `setup` module for gathering facts every time before running the playbooks. 

#### **Using the Ansible ad-hoc commands**

1. Access Ansible facts using ad-hoc commands: **`ansible all -m setup`** The `setup` module fetches all the details from the remote hosts to our controller nodes and dumps them directly to our screen for the facts to be visible to users.

```text
[user1@controller demo-var]$ ansible ubuntu -m setup
ubuntu | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.100.11"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::20c:29ff:fee3:9b49"
        ],
        "ansible_apparmor": {
            "status": "enabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "07/22/2020",
        "ansible_bios_version": "6.00",
        "ansible_cmdline": {
            "BOOT_IMAGE": "/boot/vmlinuz-5.4.0-42-generic",
            "auto": true,
            "find_preseed": "/preseed.cfg",
            "locale": "en_US",
            "noprompt": true,
            "priority": "critical",
            "quiet": true,
            "ro": true,
            "root": "UUID=e9b96f31-80d2-48ac-a242-b91e5e1be9b0"
        },
        "ansible_date_time": {
            "date": "2021-06-30",
            "day": "30",
            "epoch": "1625048412",
            "hour": "03",
.
.
.
        "ansible_userspace_architecture": "x86_64",
        "ansible_userspace_bits": "64",
        "ansible_virtualization_role": "guest",
        "ansible_virtualization_type": "VMware",
        "discovered_interpreter_python": "/usr/bin/python3",
        "gather_subset": [
            "all"
        ],
        "module_setup": true
    },
    "changed": false
}
```

2. Filtering out a specific value from Ansible facts: **`ansible all -m setup -a "filter=YourFilterHere"`**Here, the `setup` module is used to fetch the facts about the system, and further, it will use the **filter** argument to display the value from the Ansible facts. 

```text
[user1@controller demo-var]$ ansible ubuntu -m setup -a "filter=*family*"
ubuntu | SUCCESS => {
    "ansible_facts": {
        "ansible_os_family": "Debian",
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
```

#### **Using the Ansible playbook**

To access the variables from Ansible facts in the Ansible playbook, we need to use the actual name without using the **ansible** keyword.

* ansible\_facts\["ansible\_system"\] ❌
* ansible\_facts\["system"\] ✔️

 The `gather_facts` module from the Ansible playbook runs the `setup` module by default at the start of each playbook to gather the facts about remote hosts.

3. Accessing facts using Ansible playbook: Fetch the Ansible facts and display them using a playbook.

```text
---

#sample playbook for gathering facts facts-playbook.yaml

-  hosts: ubuntu
   tasks:
   - debug:
       var: ansible_facts
```

```text
[user1@controller demo-var]$  ansible-playbook facts-playbook.yaml
```

_and it will gather and shows all facts!_

4. Accessing a specific fact using an Ansible playbook: Fetching the Ansible facts, filtering them, and displaying them using a playbook.

```text
---

#Sample playbook for Accessing specific fact fact-playbook.yaml

- hosts: all
  tasks:
  - debug:
      var: ansible_facts["cmdline"]
```

```text
[user1@controller demo-var]$  ansible-playbook fact-playbook.yaml

PLAY [all] ******************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [centos]
ok: [ubuntu]

TASK [debug] ****************************************************************************************************************************
ok: [ubuntu] => {
    "ansible_facts[\"cmdline\"]": {
        "BOOT_IMAGE": "/boot/vmlinuz-5.4.0-42-generic",
        "auto": true,
        "find_preseed": "/preseed.cfg",
        "locale": "en_US",
        "noprompt": true,
        "priority": "critical",
        "quiet": true,
        "ro": true,
        "root": "UUID=e9b96f31-80d2-48ac-a242-b91e5e1be9b0"
    }
}
ok: [centos] => {
    "ansible_facts[\"cmdline\"]": {
        "BOOT_IMAGE": "/vmlinuz-3.10.0-1127.el7.x86_64",
        "LANG": "en_US.UTF-8",
        "crashkernel": "auto",
        "quiet": true,
        "rd.lvm.lv": "vgos/lvswap",
        "rhgb": true,
        "ro": true,
        "root": "/dev/mapper/vgos-lvroot",
        "spectre_v2": "retpoline"
    }
}

PLAY RECAP ******************************************************************************************************************************
centos                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### **Debug Module** 

 ****When you are working with Ansible playbooks, it’s great to have some debug options. Ansible provides a debug module that makes this task easier. It is used to print the message in the log output. The message is nothing but any variable values or output of any task.

```text
---

#Sample playbook for debugging debug-playbook.yaml
- hosts: centos
  vars:
    - first_var: "HAhaha"

  tasks:
    - name: show results
      debug: msg="The variable first_var is set to - {{ first_var }}"
```

```text
[user1@controller demo-var]$ ansible-playbook debug-playbook.yaml

PLAY [centos] ***************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [centos]

TASK [show results] *********************************************************************************************************************
ok: [centos] => {
    "msg": "The variable first_var is set to - HAhaha"
}

PLAY RECAP ******************************************************************************************************************************
centos                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Register Variables

 Ansible registers are used when you want to capture the output of a task to a variable. You can then use the value of these registers for different scenarios like a conditional statement, logging etc.

The variables will contain the value returned by the task. The common return values are documented in [Ansible docs](http://docs.ansible.com/ansible/latest/common_return_values.html). Some of the modules like shell, command etc. have module specific return values. These will be documented in the module docs.

> Each registered variables will be valid on the remote host where the task was run for the rest of the playbook execution.

```text
---

#Sample playbook for registering vars  register-playbook.yaml
- hosts: centos
  vars:
    - var_thing: "eldorado"

  tasks:
    - name: say something
      command: echo -e "{{ var_thing }}!\n Do you believe {{ var_thing }} exist?!\nI like to know about {{ var_thing }}"
      register: results

    - name: show results
      debug: msg={{ results }}
```

```text
[user1@controller demo-var]$ ansible-playbook register-playbook.yaml

PLAY [centos] ***************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [centos]

TASK [say something] *********************************************************************************************************************
changed: [centos]

TASK [show results] *********************************************************************************************************************
ok: [centos] => {
    "msg": {
        "changed": true,
        "cmd": [
            "echo",
            "-e",
            "eldorado!\\n Do you believe eldorado exist?!\\nI like to know about eldorado"
        ],
        "delta": "0:00:00.002305",
        "end": "2021-06-30 16:57:46.480738",
        "failed": false,
        "rc": 0,
        "start": "2021-06-30 16:57:46.478433",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "eldorado!\n Do you believe eldorado exist?!\nI like to know about eldorado",
        "stdout_lines": [
            "eldorado!",
            " Do you believe eldorado exist?!",
            "I like to know about eldorado"
        ]
    }
}
```

use **`debug: msg={{ results.stdout_lines }}`** in playbook to see just the output results.

## Conditionals

Some time we need to add a condition to each task and saying when we want the task to run. As an example lets try to install/remove apache and httpd packages on both ubuntu and centos, but this time lets use conditions. 

First lets make sure no apache or httpd is installed on both targets:

```text
[user1@controller demo-var]$ ansible ubuntu -b -m apt -a "name=apache2 state=absent"
ubuntu | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}

[user1@controller demo-var]$ ansible centos -b -m yum -a "name=httpd state=absent"
centos | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "removed": [
            "httpd"
        ]
    },
    "msg": "",
    "rc": 0,
    "results": [
        "Loaded plugins: fastestmirror\nResolving Dependencies\n--> Running transaction check\n---> Package httpd.x86_64 0:2.4.6-97.el7.centos will be erased\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package      Arch          Version                       Repository       Size\n================================================================================\nRemoving:\n httpd        x86_64        2.4.6-97.el7.centos           @updates        9.4 M\n\nTransaction Summary\n================================================================================\nRemove  1 Package\n\nInstalled size: 9.4 M\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Erasing    : httpd-2.4.6-97.el7.centos.x86_64                             1/1 \n  Verifying  : httpd-2.4.6-97.el7.centos.x86_64                             1/1 \n\nRemoved:\n  httpd.x86_64 0:2.4.6-97.el7.centos                                            \n\nComplete!\n"
    ]
}
```

### Conditions based on registered variables:

Often in a playbook you want to execute or skip a task based on the outcome of an earlier task. To create a conditional based on a registered variable:

1. Register the outcome of the earlier task as a variable.
2. Create a conditional test based on the registered variable.

 You create the name of the registered variable using the `register` keyword. A registered variable always contains the status of the task that created it as well as any output that task generated. You can use registered variables in conditional , also it is possible to  access the string contents of the registered variable :

```text
---

#sample playbook for conditionals  condition-playbook1.yaml
- hosts: all
  become: yes

  tasks:
    - name: install apache2
      apt: name=apache2 state=latest
      ignore_errors: yes
      register: results

    - name: install httpd
      yum: name=httpd state=latest
      failed_when: "'FAILED' in results"
```

and run:

```text
[user1@controller demo-var]$ ansible-playbook condition-playbook1.yaml

PLAY [all] ******************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [centos]
ok: [ubuntu]

TASK [install apache2] ******************************************************************************************************************
[WARNING]: Updating cache and auto-installing missing dependency: python-apt
fatal: [centos]: FAILED! => {"changed": false, "cmd": "apt-get update", "msg": "[Errno 2] No such file or directory", "rc": 2}
...ignoring
changed: [ubuntu]

TASK [install httpd] ********************************************************************************************************************
ok: [ubuntu]
changed: [centos]

PLAY RECAP ******************************************************************************************************************************
centos                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
ubuntu                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

###  __Conditionals based on ansible\_facts

Often you want to execute or skip a task based on facts. As we mentioned before Facts are attributes of individual hosts, including IP address, operating system, the status of a filesystem, and many more. With conditionals based on facts:

* You can install a certain package only when the operating system is a particular version.
* You can skip configuring a firewall on hosts with internal IP addresses.
* You can perform cleanup tasks only when a filesystem is getting full.

> Not all facts exist for all hosts. For example, the ‘lsb\_major\_release’ fact  only exists when the lsb\_release package is installed on the target host.

 In example below, we remove web server package from each server based on its os family:

```text
---
#sample conditional with facts condition-playbook2.yaml

- hosts: all
  become: yes

  tasks:

    - name: Remove Apache on Ubuntu Server
      apt: name=apache2 state=absent
      when: ansible_os_family == "Debian"

    - name: Remove Apache on CentOS  Server
      yum: name=httpd  state=absent
      when: ansible_os_family == "RedHat"
```

and lets run it:

```text
[user1@controller demo-var]$ ansible-playbook condition-playbook2.yaml

PLAY [all] ******************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [centos]
ok: [ubuntu]

TASK [Remove Apache on Ubuntu Server] **************************************************************************************************
skipping: [centos]
changed: [ubuntu]

TASK [Remove Apache on CentOS  Server] *************************************************************************************************
skipping: [ubuntu]
changed: [centos]

PLAY RECAP ******************************************************************************************************************************
centos                     : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
ubuntu                     : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

## Loops

When automating server setup, sometimes you’ll need to repeat the execution of the same task using different values. For instance, you may need to install  multiple packages , or ... . 

```text
---

#sample playbook install packages noloop-playbook.yaml

- hosts: ubuntu
  become: yes

  tasks:
    - yum: name=vim state=present
    - yum: name=nano state=present
    - yum: name=apache2 state=present
```

To avoid repeating the task several times in your playbook file, it’s better to use loops instead. There are different kinds of [loops](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html). Here we take a look at standard ones.

### with\_items

```text
[user1@controller demo-var]$ cat loop-playbook1.yaml
---

# sample loop with with_itemp loop-playbook1.yaml

- hosts: ubuntu
  become: yes

  tasks:
   - name: install pakcages
     apt: name={{ item }} update_cache=yes state=latest
     with_items:
      - vim
      - nano
      - apache2
```

and lets run it:

```text
[user1@controller demo-var]$ ansible-playbook loop-playbook1.yaml

PLAY [ubuntu] *********************************************************************************

TASK [Gathering Facts] ************************************************************************
ok: [ubuntu]

TASK [install pakcages] ***********************************************************************
changed: [ubuntu] => (item=[u'vim', u'nano', u'apache2'])

PLAY RECAP ************************************************************************************
ubuntu                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

and if we run it again:

```text
[user1@controller demo-var]$ ansible-playbook loop-playbook1.yaml

PLAY [ubuntu] ***************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [ubuntu]

TASK [install pakcages] *****************************************************************************************************************
ok: [ubuntu] => (item=[u'vim', u'nano', u'apache2'])

PLAY RECAP ******************************************************************************************************************************
ubuntu                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

{% hint style="info" %}
 `with_items` is replaced by `loop` and the `flatten` filter.
{% endhint %}

### with\_file

```text
---

# sample loop with with_file loop-playbook2.yaml

- hosts: ubuntu
  become: yes

  tasks:
   - name: show file(s) contents
     debug: msg={{ item }}
     with_file:
      - myfile1.txt
      - myfile2.txt
```

```text
[user1@controller demo-var]$ cat myfile1.txt
This is myfile1.txt, first line :-)
[user1@controller demo-var]$
[user1@controller demo-var]$ cat myfile2.txt
This is myfile2.txt, first line :-0
```

And lets run it:

```text
[user1@controller demo-var]$ ansible-playbook loop-playbook2.yaml

PLAY [ubuntu] ***************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [ubuntu]

TASK [show file(s) contents] ************************************************************************************************************
ok: [ubuntu] => (item=This is myfile1.txt, first line :-)) => {
    "msg": "This is myfile1.txt, first line :-)"
}
ok: [ubuntu] => (item=This is myfile2.txt, first line :-0) => {
    "msg": "This is myfile2.txt, first line :-0"
}

PLAY RECAP ******************************************************************************************************************************
ubuntu                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### with\_sequenece

```text
---

# sample loop with with_sequenece loop-playbook3.yaml

- hosts: ubuntu
  become: yes

  tasks:
   - name: show file(s) contents
     debug: msg={{ item }}
     with_sequence: start=1 end=5
```

run:

```text
[user1@controller demo-var]$ ansible-playbook loop-playbook3.yaml

PLAY [ubuntu] ***************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [ubuntu]

TASK [show file(s) contents] ************************************************************************************************************
ok: [ubuntu] => (item=1) => {
    "msg": "1"
}
ok: [ubuntu] => (item=2) => {
    "msg": "2"
}
ok: [ubuntu] => (item=3) => {
    "msg": "3"
}
ok: [ubuntu] => (item=4) => {
    "msg": "4"
}
ok: [ubuntu] => (item=5) => {
    "msg": "5"
}

PLAY RECAP ******************************************************************************************************************************
ubuntu                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

{% hint style="info" %}
 `with_subelements` is replaced by `loop` and the `subelements` filter.
{% endhint %}

Do not forget that the loop definitions comes at the end of your task.

.

.

.

.

[https://www.middlewareinventory.com/blog/ansible-playbook-example/](https://www.middlewareinventory.com/blog/ansible-playbook-example/)

[https://www.thegeeksearch.com/how-to-to-create-and-reference-variables-in-ansible-playbooks/](https://www.thegeeksearch.com/how-to-to-create-and-reference-variables-in-ansible-playbooks/)

[https://www.redhat.com/sysadmin/playing-ansible-facts](https://www.redhat.com/sysadmin/playing-ansible-facts)

[https://www.thegeeksearch.com/beginners-guide-to-ansible-facts/](https://www.thegeeksearch.com/beginners-guide-to-ansible-facts/)

[https://linuxhint.com/ansible\_debug\_module/](https://linuxhint.com/ansible_debug_module/)

[https://www.decodingdevops.com/ansible-debug-module-with-examples/](https://www.decodingdevops.com/ansible-debug-module-with-examples/)

[http://www.mydailytutorials.com/ansible-register-variables/](http://www.mydailytutorials.com/ansible-register-variables/)

[https://docs.ansible.com/ansible/latest/user\_guide/playbooks\_conditionals.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)

[https://www.linuxtechi.com/use-when-conditions-in-ansible-playbook/](https://www.linuxtechi.com/use-when-conditions-in-ansible-playbook/)

[https://docs.ansible.com/ansible/latest/user\_guide/playbooks\_loops.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)

.

