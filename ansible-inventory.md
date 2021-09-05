# Ansible Inventory

### What is Inventory file?

Ansible reads information about which machines you want to manage from your inventory. Ansible has a default inventory file, but you can create your own and define which servers you want Ansible to manage.

### Ansible default inventory file

 The default location for inventory is a file called `/etc/ansible/hosts:`

```text
[root@control1 ~]# cat /etc/ansible/hosts
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers.

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group

## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110

# If you have multiple hosts following a pattern you can specify
# them like this:

## www[001:006].example.com

# Ex 3: A collection of database servers in the 'dbservers' group

## [dbservers]
##
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57

# Here's another example of host ranges, this time there are no
# leading 0s:

## db-[99:101]-node.example.com

```

If you don't create inventory file, Ansible uses this default inventory file. 

### Sample Inventory files

The inventory file is an **ini** like format.It is simply a number of servers, listed one after other:

```text
#Sample Inventory File
server1.company.com
192.168.10.2
```

you can also group different servers together by define it like this:

```text
#Grouping servers
server1.company.com
192.16.10.2

[mail]
192.168.10.3
server4.company.com

[db]
server5.company.com
server6.company.com
```

You can also have a group of groups:

```text
#Group of groups:
server1.company.com
192.168.10.2

[mail]
192.168.10.3
server4.company.com

[db]
server5.company.com
server6.company.com

[all_servers:children]
mail
db
```

in this sample we have created a group called _all\_servers_.  Some other examples:

| Example | Description |
| :--- | :--- |
| web\[1-3\].example.com | If you have a lot of hosts with a similar pattern |
| server1.example.com**:5555** | If you have hosts that run on non-standard SSH ports |

Also If you like to refer to these servers in Ansible using an alias, it is possible:

```text
#Using alias
web1  ansible_host=server1.company.com
db1   ansible_host=server2.company.com
mail1 ansible_host=192.168.10.3
web2  ansible_host=server4.company.com 
```

`ansible_host` is an inventory parameter, used to specify the FQDN or IP Address of a server. There are other inventory parameters too.

### Inventory Parameters

Here, lets take a look at most useful Ansible Inventory parameters ****and examples:

| Example | Description |
| :--- | :--- |
| **ansible\_host=**1.2.3.4 | name of the host to connect to, if different from the alias you wich to give to it |
| **ansible\_port=**5555 | which port to connect to \(default 22/tcp\) |
| **ansible\_connection=**ssh | defines how ansible get connected to the target\[shh / winrm / localhost  \] |
| **ansible\_user=**Linda | defines the user used to make remote connection, if no user is specified current user will be used |
| **ansible\_ssh\_pass=**\*\*\* | define ssh password for linux |

```text
#Sample Inventory parameters
web1  ansible_host=server1.company.com ansible_connection=ssh   ansible_user=root
db1   ansible_host=server2.company.com ansible_connection=winrm ansible_user=admin
mail1 ansible_host=192.168.10.3        ansible_connection=ssh   ansible_ssh_pass=P@S
web2  ansible_host=server4.company.com ansible_connection=winrm

localhost ansible_connection=localhost
```

{% hint style="danger" %}
Note that storing password in plain text format is not a good idea, lookup Ansible Vault to securely store your password in an encrypted format. We will talk about it later. 
{% endhint %}

In production environment using password to establish connectivity between systems is not recommended, it better to use SSH Keys instead.

### Demo - Inventory files

Lets make a test project with a custom inventory file. 

```text
[user1@controller ~]$ mkdir demo-inventory
[user1@controller ~]$ cd demo-inventory/
[user1@controller demo-inventory]$ ll
total 0
[user1@controller demo-inventory]$ vim inventory.txt
[user1@controller demo-inventory]$ cat inventory.txt
ubuntu 
```

The list of machines in the inventory can be found out through the  `ansible --list-hosts all` command :

```text
[user1@controller demo-inventory]$ ansible --list-hosts all -i inventory.txt
  hosts (1):
    ubuntu
```

 _We can specify a different inventory file at the command line using the `-i <path>` option._

 And now, First Ansible Task!  ****You can ping all of your inventory machines using the following command:

```text
[user1@controller demo-inventory]$  ansible ubuntu -m ping -i inventory.txt
ubuntu | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

so that confirms that our ansible controller can successfully communicate or connect to the target machines. lets update inventory.txt file by adding second target:

```text
[user1@controller demo-inventory]$ cat inventory.txt
ubuntu 
centos 
```

and lets  see the results:

```text
[user1@controller demo-inventory]$ ansible all  -m ping -i inventory.txt
centos | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
ubuntu | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

There is a group that Ansible creates by default and that's called the ``**`all`** group. The all group is a built-in group that Ansible creates and it has all the servers in our inventory file part of that group.

If there is a problem with python on one of your target nodes, you can send a raw module \(we will talk about it later\):

```text
[user1@controller demo-inventory]$ ansible -m raw -a "/usr/bin/uptime" -i inventory.txt all
centos | CHANGED | rc=0 >>
 10:12:35 up 23:33,  2 users,  load average: 0.00, 0.01, 0.05
Shared connection to centos closed.

ubuntu | CHANGED | rc=0 >>
 22:42:35 up 19:20,  2 users,  load average: 0.00, 0.00, 0.00
Shared connection to ubuntu closed.
```

And if you like to see which python version has been installed on remote machines use shell module\(we will talk about it later\):

```text
[user1@controller demo-inventory]$ ansible -m shell -a "python -V" -i inventory.txt all
centos | CHANGED | rc=0 >>
Python 2.7.5
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host ubuntu should use
/usr/bin/python3, but is using /usr/bin/python for backward compatibility with
prior Ansible releases. A future Ansible release will default to using the
discovered platform python for this host. See https://docs.ansible.com/ansible/
2.9/reference_appendices/interpreter_discovery.html for more information. This
feature will be removed in version 2.12. Deprecation warnings can be disabled
by setting deprecation_warnings=False in ansible.cfg.
ubuntu | CHANGED | rc=0 >>
Python 2.7.17
```

> Deprecation warnings can be disabled by setting deprecation\_warnings=False in ansible.cfg

Now that you know about inventory files let put our targets nodes information on `/etc/ansible/hosts` :

```text
[root@controller ~]# tail -n7 /etc/ansible/hosts

ubuntu
centos

[lab]
ubuntu
centos
```

 this way we won't need to specify inventory file while running a command. 

{% hint style="success" %}
### Dynamic inventories

Most infrastructure can be managed with a custom inventory file, but there are many situations where more control is needed. Ansible will accept any kind of executable file as an inventory file, as long as you can pass it to Ansible as JSON.

You could create an executable binary, a script, or anything else that can be run and will output JSON to stdout, and Ansible will call it with the argument `--list` when you run, as an example, `ansible all -i my-inventory-script -m ping`.

 You can always check ansible github web page and other sources for [examples](https://github.com/ansible/ansible/tree/devel/examples), but  that's more advanced topic.
{% endhint %}

that's all.

.

.

.

[https://docs.ansible.com/ansible/2.7/user\_guide/intro\_inventory.html](https://docs.ansible.com/ansible/2.7/user_guide/intro_inventory.html)

[https://linuxhint.com/ansible-tutorial-beginners/](https://linuxhint.com/ansible-tutorial-beginners/)

[https://allandenot.com/devops/2015/01/16/ansible-with-multiple-inventory-files.html\#:~:text=TL%3BDR%3A%20Inventory%20can%20be,scripts%20like%20ec2.py\).](https://allandenot.com/devops/2015/01/16/ansible-with-multiple-inventory-files.html#:~:text=TL%3BDR%3A%20Inventory%20can%20be,scripts%20like%20ec2.py%29.)

[https://www.jeffgeerling.com/blog/creating-custom-dynamic-inventories-ansible](https://www.jeffgeerling.com/blog/creating-custom-dynamic-inventories-ansible)

.

