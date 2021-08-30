---
description: >-
  In this section we take a quick look at more advanced concepts in ansible and
  introduce useful options.
---

# Advanced Concepts

## How to handle long running tasks in Ansible?

By default Ansible runs tasks synchronously, holding the connection to the remote node open until the action is completed. This means within a playbook, each task blocks the next task by default, meaning subsequent tasks will not run until the current task completes.

![](.gitbook/assets/adv-asyncpoll.jpg)

 This behavior can create challenges. For example, a task may take longer to complete than the SSH session allows for, causing a timeout. Or you may want a long-running process to execute in the background while you perform other tasks concurrently. Asynchronous mode lets you control how long-running tasks execute.

### What is Asynchronous mode?

Asynchronous mode allows us to control the playbook execution flow by defining how long-running tasks completes their execution.

To enable Asynchronous mode within Ansible playbook we need to use few parameters such as **`async`**, **`poll`**.

* **`async:`** Async indicates the Total time to complete the task or its maximum runtime of the task.
* **`poll:`** poll indicates to Ansible, how often to poll to check if the command has been completed. or how frequently you would like to poll for status. with poll we keep checking whether the job is completed or not. The default poll value is 10 seconds.

```text
---
#async and poll example playbook

- hosts: centos
  become: yes

  tasks:

  - name: simulate long running op (15 sec), wait for up to 45 sec, poll every 5 sec
    command: /bin/sleep 15
    async: 45
    poll: 5
```

#### Avoid connection timeouts: poll &gt; 0

 If you want to set a longer timeout limit for a certain task in your playbook, use `async` with `poll` set to a positive value. **Ansible will still block the next task in your playbook, waiting until the async task either completes, fails or times out**. However, the task will only time out if it exceeds the timeout limit you set with the `async` parameter.

```text
---
# async and poll example1 playbook
- name: sleep and create a user 
  hosts: centos
  become: true

  tasks:
    - name: sleep for 15 seconds
      command: /bin/sleep 15
      async: 45 # the total time allowed to complete the sleep task
      poll: 5 # No need to poll just fire and forget the sleep command
      register: sleeping_node

    - name: task-2 to create a test user
      user: name=mona state=present shell=/bin/bash
```

#### Run tasks concurrently: poll = 0

 If you want to run multiple tasks in a playbook concurrently, use `async` with `poll` set to 0. When you set `poll: 0`, Ansible starts the task and immediately **moves on to the next task without waiting for a result**. Each async task runs until it either completes, fails or times out \(runs longer than its `async` value\). The playbook run ends without checking back on async tasks.

```text
---
# async and poll example2 playbook
- name: sleep and create a user 
  hosts: centos
  become: true

  tasks:
    - name: sleep for 60 seconds
      command: /bin/sleep 60
      async: 80 # the total time allowed to complete the sleep task
      poll: 0 # No need to poll just fire and forget the sleep command
      register: sleeping_node

    - name: task-2 to create a test user
      user: name=lisa state=present shell=/bin/bash
```

### Ansible async\_status

Ansible provides the option to get the task status in any time. Using ansible async\_status we can get the status of async task at any time.

```text
---
# async and poll example3 playbook
- name: sleep and create a user and check async-status
  hosts: centos
  become: true

  tasks:
    - name: sleep for 20 seconds
      command: /bin/sleep 20
      async: 30 # the total time allowed to complete the sleep task
      poll: 0 # No need to poll just fire and forget the sleep command
      register: sleeping_node

    - name: task-2 to create a test user
      user: name=lara state=present shell=/bin/bash
      
    - name: Checking the Job Status running in background
      async_status:
        jid: "{{ sleeping_node.ansible_job_id }}"
      register: job_result
      until: job_result.finished # Retry within limit until the job status changed to "finished": 1
      retries: 5 # Maximum number of retries to check job status
```

> execute the playbook with `-v` option so that we can see the Job ID of first task to check its status later.

To check the status of first task on target  node\(s\):

```text
ansible  centos -m async_status -a "jid=WXYZ.WXYZ" 
```

### Asynchronous ad hoc tasks

.

.

.

[https://docs.ansible.com/ansible/latest/user\_guide/playbooks\_async.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_async.html)

[https://blog.learncodeonline.in/ansible-advanced-speed-up-playbook-run-with-async-and-polling](https://blog.learncodeonline.in/ansible-advanced-speed-up-playbook-run-with-async-and-polling)

[https://www.decodingdevops.com/ansible-asynch-poll-with-examples/](https://www.decodingdevops.com/ansible-asynch-poll-with-examples/)

.




