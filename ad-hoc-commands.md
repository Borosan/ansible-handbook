# ad-hoc commands

### What are ad-hoc command?

One of the simplest ways Ansible can be used is by using ad-hoc commands. These can be used when you want to issue some commands on a server or a bunch of servers. Ad-hoc commands are not stored for future uses but represent a fast way to interact with the desired servers.

```
ansible [pattern] -m [module] -a "[module options]"
```

```
[user1@controller demo-adhoc]$ ansible all  -a "uptime"
centos | CHANGED | rc=0 >>
 11:31:04 up 1 day, 51 min,  3 users,  load average: 0.00, 0.01, 0.05
ubuntu | CHANGED | rc=0 >>
 00:01:04 up 20:38,  3 users,  load average: 0.00, 0.03, 0.00
```

### Use cases for ad hoc tasks 

ad hoc tasks can be used to reboot servers, copy files, manage packages and users, and much more. You can use any Ansible module in an ad hoc task. ad hoc tasks, like playbooks, use a declarative model, calculating and executing the actions required to reach a specified final state. They achieve a form of idempotence by checking the current state before they begin and doing nothing unless the current state is different from the specified final state.

###  privilege escalation

By default ansible in ad-hoc mode  is not going to ** **escalate privilege

```
[user1@controller demo-adhoc]$ ansible all -a "whoami"
ubuntu | CHANGED | rc=0 >>
user1
centos | CHANGED | rc=0 >>
user1
```

as you can see, the command `whoami` has been execute as regular user on remote targets. If we need to run thing that requires more access or we want to   escalate privileges we have to use **`-b`** or** become** switch:

```
[user1@controller demo-adhoc]$ ansible all -b -a "whoami"
ubuntu | CHANGED | rc=0 >>
root
centos | CHANGED | rc=0 >>
root
```

### Rebooting servers

 The default module for the ansible command-line utility is the ansible built-in **command** module. You can use an ad-hoc task to call the command module and reboot all servers:

```
[user1@controller demo-adhoc]$ ansible all -m command -a "reboot"
centos | FAILED | rc=2 >>
[Errno 2] No such file or directory
ubuntu | FAILED | rc=1 >>
Failed to set wall message, ignoring: Interactive authentication required.
Failed to reboot system via logind: Interactive authentication required.
Failed to open /dev/initctl: Permission denied
Failed to talk to init daemon.non-zero return code
```

it gives us errors, because it doesn't escalate privilege, lets use` -b` switch:

```
[user1@controller demo-adhoc]$ ansible all -b  -a "reboot"
centos | FAILED | rc=-1 >>
Failed to connect to the host via ssh: ssh: connect to host centos port 22: Connection refused
ubuntu | FAILED | rc=-1 >>
Failed to connect to the host via ssh: ssh: connect to host ubuntu port 22: Connection refused
```

And all targets have been rebooted. Lets wait for a minute  and check:

```
[user1@controller demo-adhoc]$ ansible all -m shell -a "uptime"
centos | CHANGED | rc=0 >>
 13:53:58 up 0 min,  2 users,  load average: 0.21, 0.08, 0.03
ubuntu | CHANGED | rc=0 >>
 02:23:58 up 1 min,  2 users,  load average: 0.30, 0.14, 0.05
```

### shutting down servers

```
[user1@controller demo-adhoc]$ ansible all -b -a "shutdown -r"
centos | CHANGED | rc=0 >>
Shutdown scheduled for Mon 2021-06-28 13:58:27 +0430, use 'shutdown -c' to cancel.
ubuntu | CHANGED | rc=0 >>
Shutdown scheduled for Mon 2021-06-28 02:28:27 PDT, use 'shutdown -c' to cancel.
```

and lets quickly cancel it:

```
[user1@controller demo-adhoc]$ date
Mon Jun 28 17:27:36 +0430 2021
[user1@controller demo-adhoc]$ ansible all -b -a "shutdown -c"
ubuntu | CHANGED | rc=0 >>

centos | CHANGED | rc=0 >> 
```

### Installing a package

Lets install a web server on ubuntu machine:

```
[user1@controller demo-adhoc]$ ansible all -b -m apt -a "name=apache2 state=present"
[WARNING]: Updating cache and auto-installing missing dependency: python-apt
centos | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "cmd": "apt-get update",
    "msg": "[Errno 2] No such file or directory",
    "rc": 2
}
ubuntu | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1624869705,
    "cache_updated": false,
    "changed": true,
    "stderr": "",
    "stderr_lines": [],
    "stdout": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nThe following additional packages will be installed:\n  apache2-bin apache2-data apache2-utils libapr1 libaprutil1\n  libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.2-0\nSuggested packages:\n  apache2-doc apache2-suexec-pristine | apache2-suexec-custom\nThe following NEW packages will be installed:\n  apache2 apache2-bin apache2-data apache2-utils libapr1 libaprutil1\n  libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.2-0\n0 upgraded, 9 newly installed, 0 to remove and 283 not upgraded.\nNeed to get 1713 kB of archives.\nAfter this operation, 6920 kB of additional disk space will be used.\nGet:1 http://us.archive.ubuntu.com/ubuntu bionic/main amd64 libapr1 amd64 1.6.3-2 [90.9 kB]\nGet:2 http://us.archive.ubuntu.com/ubuntu bionic/main amd64 libaprutil1 amd64 1.6.1-2 [84.4 kB]\nGet:3 http://us.archive.ubuntu.com/ubuntu bionic/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.1-2 [10.6 kB]\nGet:4 http://us.archive.ubuntu.com/ubuntu bionic/main amd64 libaprutil1-ldap amd64 1.6.1-2 [8764 B]\nGet:5 http://us.archive.ubuntu.com/ubuntu bionic/main amd64 liblua5.2-0 amd64 5.2.4-1.1build1 [108 kB]\nGet:6 http://us.archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2-bin amd64 2.4.29-1ubuntu4.16 [1070 kB]\nGet:7 http://us.archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2-utils amd64 2.4.29-1ubuntu4.16 [84.0 kB]\nGet:8 http://us.archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2-data all 2.4.29-1ubuntu4.16 [160 kB]\nGet:9 http://us.archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2 amd64 2.4.29-1ubuntu4.16 [95.1 kB]\nFetched 1713 kB in 19s (89.3 kB/s)\nSelecting previously unselected package libapr1:amd64.\r\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 132028 files and directories currently installed.)\r\nPreparing to unpack .../0-libapr1_1.6.3-2_amd64.deb ...\r\nUnpacking libapr1:amd64 (1.6.3-2) ...\r\nSelecting previously unselected package libaprutil1:amd64.\r\nPreparing to unpack .../1-libaprutil1_1.6.1-2_amd64.deb ...\r\nUnpacking libaprutil1:amd64 (1.6.1-2) ...\r\nSelecting previously unselected package libaprutil1-dbd-sqlite3:amd64.\r\nPreparing to unpack .../2-libaprutil1-dbd-sqlite3_1.6.1-2_amd64.deb ...\r\nUnpacking libaprutil1-dbd-sqlite3:amd64 (1.6.1-2) ...\r\nSelecting previously unselected package libaprutil1-ldap:amd64.\r\nPreparing to unpack .../3-libaprutil1-ldap_1.6.1-2_amd64.deb ...\r\nUnpacking libaprutil1-ldap:amd64 (1.6.1-2) ...\r\nSelecting previously unselected package liblua5.2-0:amd64.\r\nPreparing to unpack .../4-liblua5.2-0_5.2.4-1.1build1_amd64.deb ...\r\nUnpacking liblua5.2-0:amd64 (5.2.4-1.1build1) ...\r\nSelecting previously unselected package apache2-bin.\r\nPreparing to unpack .../5-apache2-bin_2.4.29-1ubuntu4.16_amd64.deb ...\r\nUnpacking apache2-bin (2.4.29-1ubuntu4.16) ...\r\nSelecting previously unselected package apache2-utils.\r\nPreparing to unpack .../6-apache2-utils_2.4.29-1ubuntu4.16_amd64.deb ...\r\nUnpacking apache2-utils (2.4.29-1ubuntu4.16) ...\r\nSelecting previously unselected package apache2-data.\r\nPreparing to unpack .../7-apache2-data_2.4.29-1ubuntu4.16_all.deb ...\r\nUnpacking apache2-data (2.4.29-1ubuntu4.16) ...\r\nSelecting previously unselected package apache2.\r\nPreparing to unpack .../8-apache2_2.4.29-1ubuntu4.16_amd64.deb ...\r\nUnpacking apache2 (2.4.29-1ubuntu4.16) ...\r\nSetting up libapr1:amd64 (1.6.3-2) ...\r\nSetting up apache2-data (2.4.29-1ubuntu4.16) ...\r\nSetting up libaprutil1:amd64 (1.6.1-2) ...\r\nSetting up liblua5.2-0:amd64 (5.2.4-1.1build1) ...\r\nSetting up libaprutil1-ldap:amd64 (1.6.1-2) ...\r\nSetting up libaprutil1-dbd-sqlite3:amd64 (1.6.1-2) ...\r\nSetting up apache2-utils (2.4.29-1ubuntu4.16) ...\r\nSetting up apache2-bin (2.4.29-1ubuntu4.16) ...\r\nSetting up apache2 (2.4.29-1ubuntu4.16) ...\r\nEnabling module mpm_event.\r\nEnabling module authz_core.\r\nEnabling module authz_host.\r\nEnabling module authn_core.\r\nEnabling module auth_basic.\r\nEnabling module access_compat.\r\nEnabling module authn_file.\r\nEnabling module authz_user.\r\nEnabling module alias.\r\nEnabling module dir.\r\nEnabling module autoindex.\r\nEnabling module env.\r\nEnabling module mime.\r\nEnabling module negotiation.\r\nEnabling module setenvif.\r\nEnabling module filter.\r\nEnabling module deflate.\r\nEnabling module status.\r\nEnabling module reqtimeout.\r\nEnabling conf charset.\r\nEnabling conf localized-error-pages.\r\nEnabling conf other-vhosts-access-log.\r\nEnabling conf security.\r\nEnabling conf serve-cgi-bin.\r\nEnabling site 000-default.\r\nCreated symlink /etc/systemd/system/multi-user.target.wants/apache2.service -> /lib/systemd/system/apache2.service.\r\nCreated symlink /etc/systemd/system/multi-user.target.wants/apache-htcacheclean.service -> /lib/systemd/system/apache-htcacheclean.service.\r\nProcessing triggers for libc-bin (2.27-3ubuntu1.2) ...\r\nProcessing triggers for systemd (237-3ubuntu10.42) ...\r\nProcessing triggers for man-db (2.8.3-2ubuntu0.1) ...\r\nProcessing triggers for ufw (0.36-0ubuntu0.18.04.1) ...\r\nProcessing triggers for ureadahead (0.100.0-21) ...\r\n",
    "stdout_lines": [
        "Reading package lists...",
        "Building dependency tree...",
        .
        .
        .
        "  libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.2-0",
        "0 upgraded, 9 newly installed, 0 to remove and 283 not upgraded.",
        "Need to get 1713 kB of archives.",
        "After this operation, 6920 kB of additional disk space will be used.",
        "Get:1 http://us.archive.ubuntu.com/ubuntu bionic/main amd64 libapr1 amd64 1.6.3-2 [90.9 kB]",
        "Get:2 http://us.archive.ubuntu.com/ubuntu bionic/main amd64 libaprutil1 amd64 1.6.1-2 [84.4 kB]",
        "Get:3 http://us.archive.ubuntu.com/ubuntu bionic/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.1-2 [10.6 kB]",
        "Get:4 http://us.archive.ubuntu.com/ubuntu bionic/main amd64 libaprutil1-ldap amd64 1.6.1-2 [8764 B]",
        "Get:5 http://us.archive.ubuntu.com/ubuntu bionic/main amd64 liblua5.2-0 amd64 5.2.4-1.1build1 [108 kB]",
        "Get:6 http://us.archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2-bin amd64 2.4.29-1ubuntu4.16 [1070 kB]",
        "Get:7 http://us.archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2-utils amd64 2.4.29-1ubuntu4.16 [84.0 kB]",
        "Get:8 http://us.archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2-data all 2.4.29-1ubuntu4.16 [160 kB]",
        "Get:9 http://us.archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2 amd64 2.4.29-1ubuntu4.16 [95.1 kB]",
        "Fetched 1713 kB in 19s (89.3 kB/s)",
        "Selecting previously unselected package libapr1:amd64.",
        "(Reading database ... ",
        "(Reading database ... 5%",
        "(Reading database ... 10%",
        "(Reading database ... 15%",
        "(Reading database ... 20%",
        "(Reading database ... 25%",
        "(Reading database ... 30%",
        "(Reading database ... 35%",
        "(Reading database ... 40%",
        "(Reading database ... 45%",
        "(Reading database ... 50%",
        "(Reading database ... 55%",
        "(Reading database ... 60%",
        "(Reading database ... 65%",
        "(Reading database ... 70%",
        "(Reading database ... 75%",
        "(Reading database ... 80%",
        "(Reading database ... 85%",
        "(Reading database ... 90%",
        "(Reading database ... 95%",
        "(Reading database ... 100%",
        "(Reading database ... 132028 files and directories currently installed.)",
        "Preparing to unpack .../0-libapr1_1.6.3-2_amd64.deb ...",
        .
        .
        .
        "Enabling module reqtimeout.",
        "Enabling conf charset.",
        "Enabling conf localized-error-pages.",
        "Enabling conf other-vhosts-access-log.",
        "Enabling conf security.",
        "Enabling conf serve-cgi-bin.",
        "Enabling site 000-default.",
        "Created symlink /etc/systemd/system/multi-user.target.wants/apache2.service -> /lib/systemd/system/apache2.service.",
        "Created symlink /etc/systemd/system/multi-user.target.wants/apache-htcacheclean.service -> /lib/systemd/system/apache-htcacheclean.service.",
        "Processing triggers for libc-bin (2.27-3ubuntu1.2) ...",
        "Processing triggers for systemd (237-3ubuntu10.42) ...",
        "Processing triggers for man-db (2.8.3-2ubuntu0.1) ...",
        "Processing triggers for ufw (0.36-0ubuntu0.18.04.1) ...",
        "Processing triggers for ureadahead (0.100.0-21) ..."
    ]
}
```

Obviously there is not apt program on centos machine so it troughs an error but install apache2 package on ubuntu machine. Lets run it again:

```
[user1@controller demo-adhoc]$ ansible all -b -m apt -a "name=apache2 state=present"
[WARNING]: Updating cache and auto-installing missing dependency: python-apt
centos | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "cmd": "apt-get update",
    "msg": "[Errno 2] No such file or directory",
    "rc": 2
}
ubuntu | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1624869705,
    "cache_updated": false,
    "changed": false
}
```

lets run the command again  with yum module:   

```
[user1@controller demo-adhoc]$ ansible all -b -m yum -a "name=httpd state=latest"
ubuntu | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1624869705,
    "cache_updated": false,
    "changed": false,
    "msg": "'/usr/bin/apt-get -y -o \"Dpkg::Options::=--force-confdef\" -o \"Dpkg::Options::=--force-confold\"      install 'httpd'' failed: E: Package 'httpd' has no installation candidate\n",
    "rc": 100,
    "stderr": "E: Package 'httpd' has no installation candidate\n",
    "stderr_lines": [
        "E: Package 'httpd' has no installation candidate"
    ],
    "stdout": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nPackage httpd is a virtual package provided by:\n  yaws 2.0.4+dfsg-2ubuntu0.1\n  nginx-light 1.14.0-0ubuntu1.9\n  nginx-full 1.14.0-0ubuntu1.9\n  nginx-extras 1.14.0-0ubuntu1.9\n  lighttpd 1.4.45-1ubuntu3.18.04\n  nginx-core 1.14.0-0ubuntu1.9\n  apache2 2.4.29-1ubuntu4.16\n  webfs 1.21+ds1-12\n  tntnet 2.2.1-3build1\n  mini-httpd 1.23-1.2build1\n  micro-httpd 20051212-15.1\n  ebhttpd 1:1.0.dfsg.1-4.3build1\n  aolserver4-daemon 4.5.1-18.1\n  aolserver4-core 4.5.1-18.1\n\n",
    "stdout_lines": [
        "Reading package lists...",
        "Building dependency tree...",
        "Reading state information...",
        "Package httpd is a virtual package provided by:",
        .
        .
        .
        "  aolserver4-daemon 4.5.1-18.1",
        "  aolserver4-core 4.5.1-18.1",
        ""
    ]
}
centos | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "installed": [
            "httpd"
        ],
        "updated": []
    },
    "msg": "",
    "rc": 0,
    "results": [
        "Loaded plugins: fastestmirror\nLoading mirror speeds from cached hostfile\n * base: repo.boun.edu.tr\n * extras: mirror.netdirekt.com.tr\n * updates: mirror.netdirekt.com.tr\nResolving Dependencies\n--> Running transaction check\n---> Package httpd.x86_64 0:2.4.6-97.el7.centos will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package       Arch           Version                     Repository       Size\n================================================================================\nInstalling:\n httpd         x86_64         2.4.6-97.el7.centos         updates         2.7 M\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal download size: 2.7 M\nInstalled size: 9.4 M\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : httpd-2.4.6-97.el7.centos.x86_64                             1/1 \n  Verifying  : httpd-2.4.6-97.el7.centos.x86_64                             1/1 \n\nInstalled:\n  httpd.x86_64 0:2.4.6-97.el7.centos                                            \n\nComplete!\n"
    ]
}
```

> state=latest not only install the package if it is not there , but it will update it if it is there with an older version! 

### removing a package

lets remove apache2 from ubuntu machine:

```
[user1@controller demo-adhoc]$ ansible ubuntu -b -m apt -a "name=apache2 state=absent"
ubuntu | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "stderr": "",
    "stderr_lines": [],
    "stdout": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nThe following packages were automatically installed and are no longer required:\n  apache2-bin apache2-data apache2-utils libapr1 libaprutil1\n  libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.2-0\nUse 'sudo apt autoremove' to remove them.\nThe following packages will be REMOVED:\n  apache2\n0 upgraded, 0 newly installed, 1 to remove and 283 not upgraded.\nAfter this operation, 536 kB disk space will be freed.\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 132733 files and directories currently installed.)\r\nRemoving apache2 (2.4.29-1ubuntu4.16) ...\r\nProcessing triggers for man-db (2.8.3-2ubuntu0.1) ...\r\nProcessing triggers for ufw (0.36-0ubuntu0.18.04.1) ...\r\n",
    "stdout_lines": [
        "Reading package lists...",
        "Building dependency tree...",
        "Reading state information...",
        "The following packages were automatically installed and are no longer required:",
        "  apache2-bin apache2-data apache2-utils libapr1 libaprutil1",
        "  libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.2-0",
        "Use 'sudo apt autoremove' to remove them.",
        "The following packages will be REMOVED:",
        "  apache2",
        "0 upgraded, 0 newly installed, 1 to remove and 283 not upgraded.",
        "After this operation, 536 kB disk space will be freed.",
        "(Reading database ... ",
        "(Reading database ... 5%",
        "(Reading database ... 10%",
        "(Reading database ... 15%",
        "(Reading database ... 20%",
        "(Reading database ... 25%",
        "(Reading database ... 30%",
        "(Reading database ... 35%",
        "(Reading database ... 40%",
        "(Reading database ... 45%",
        "(Reading database ... 50%",
        "(Reading database ... 55%",
        "(Reading database ... 60%",
        "(Reading database ... 65%",
        "(Reading database ... 70%",
        "(Reading database ... 75%",
        "(Reading database ... 80%",
        "(Reading database ... 85%",
        "(Reading database ... 90%",
        "(Reading database ... 95%",
        "(Reading database ... 100%",
        "(Reading database ... 132733 files and directories currently installed.)",
        "Removing apache2 (2.4.29-1ubuntu4.16) ...",
        "Processing triggers for man-db (2.8.3-2ubuntu0.1) ...",
        "Processing triggers for ufw (0.36-0ubuntu0.18.04.1) ..."
    ]
}
```

```
[user1@controller demo-adhoc]$ ansible ubuntu -b -m apt -a "name=apache2 state=absent"
ubuntu | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
```

### enabling a service

Lets enable httpd service on all hosts:

```
[user1@controller demo-adhoc]$ ansible all -b -m service -a "name=apache2 enabled=true"
centos | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "msg": "Could not find the requested service apache2: host"
}
ubuntu | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "enabled": true,
    "name": "apache2",
    "status": {
        "ActiveEnterTimestamp": "Mon 2021-06-28 02:31:29 PDT",
        "ActiveEnterTimestampMonotonic": "521030058",
        "ActiveExitTimestamp": "Mon 2021-06-28 02:41:48 PDT",
        "ActiveExitTimestampMonotonic": "1140587029",
        "ActiveState": "inactive",
        .
        .
        .
        "UMask": "0022",
        "UnitFileState": "bad",
        "UtmpMode": "init",
        "WantedBy": "multi-user.target",
        "WatchdogTimestampMonotonic": "0",
        "WatchdogUSec": "0"
    }
}

```

There is no apache2 service on centos  so it couldn't start it, but it enabled apache2 service on ubuntu!!! we have already removed that! how it is possible?

{% hint style="danger" %}
Service module does not make sure that the service exist! So sometimes you should make sure what state your system is in!
{% endhint %}

 We can also start a service with`ansible all -m service -a "name=httpd state=started" `command`.` 

###  File system Management using ad-hoc commands

Ansible can interact with system but it can also interact with remote file system on all of the computers. There are different modules which behave in suddenly different ways. **command module, shell module, raw module,** Lets compare them

| **command Module**               | **shell module**                                    | **raw module**                   |
| -------------------------------- | --------------------------------------------------- | -------------------------------- |
| **doesn't use shell (bash/sh)**  | **supports pipes and redirects**                    | **just sends commands over SSH** |
| **Can't use pipes or redirects** | **Can get messed up by user settings(/etc/bashrc)** | **doesn't need Python**          |
| _safest_                         | _safer_                                             | _safe_                           |

```
[user1@controller demo-adhoc]$ ansible lab -b -m command -a 'echo "By command"  > /root/myfile.txt'
centos | CHANGED | rc=0 >>
By command > /root/myfile.txt
ubuntu | CHANGED | rc=0 >>
By command > /root/myfile.txt
[user1@controller demo-adhoc]$ ansible lab -b -m shell -a 'echo "By shell"  >> /root/myfile.txt'
centos | CHANGED | rc=0 >>

ubuntu | CHANGED | rc=0 >>

[user1@controller demo-adhoc]$ ansible lab -b -m raw -a 'echo "By raw"  >> /root/myfile.txt'
centos | CHANGED | rc=0 >>
Shared connection to centos closed.

ubuntu | CHANGED | rc=0 >>
Shared connection to ubuntu closed.

```

and check the results on one of target machines:

```
[root@centos ~]# pwd
/root
[root@centos ~]# ls -l myfile.txt
-rw-r--r-- 1 root root 16 Jun 28 14:41 myfile.txt
[root@centos ~]# cat myfile.txt
By shell
By raw
```

> As you can see command modules does not support redirection.

**file module: i**t gives the power of file management to us, lets remove a file:

```
[user1@controller demo-adhoc]$ ansible lab -b -m file -a "path=/root/myfile.txt state=absent"
centos | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "path": "/root/myfile.txt",
    "state": "absent"
}
ubuntu | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "path": "/root/myfile.txt",
    "state": "absent"
}
```

and run it again:

```
[user1@controller demo-adhoc]$ ansible lab -b -m file -a "path=/root/myfile.txt state=absent"
centos | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "path": "/root/myfile.txt",
    "state": "absent"
}
ubuntu | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "path": "/root/myfile.txt",
    "state": "absent"
}
```

**Copy module: **We can send  files from one server to the other one:

```
[user1@controller demo-adhoc]$ ansible centos -b -m copy -a "src=/etc/hosts dest=/etc/hosts"
centos | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "5f95076cb101d9e5643bd37830042a0a02e86d74",
    "dest": "/etc/hosts",
    "gid": 0,
    "group": "root",
    "md5sum": "c7b10f9d51946c3274735771de5b69b6",
    "mode": "0644",
    "owner": "root",
    "size": 337,
    "src": "/home/user1/.ansible/tmp/ansible-tmp-1624888401.02-66448-234070214950598/source",
    "state": "file",
    "uid": 0
}
```

The ad-hoc system is powerful with some limitations. Keep reading to see other Ansible options and  more complicated things with play-books.

.

.

.

[https://www.guru99.com/ansible-tutorial.html#6](https://www.guru99.com/ansible-tutorial.html#6)

[https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html)

.
