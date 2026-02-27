+++
title = 'Managing Services'
description = 'Managing Services with Ansible'
+++
### Managing Services

Services can be managed in many ways. You can manage systemd services,
but Ansible also allows for management of tasks using Linux cron and at.
Apart from that, you can use Ansible to manage the desired systemd
target that a managed system should be started in, and it can reboot
running machines. [Table 14-2](#ch14.xhtml#tab14_2) gives an overview of
the most significant modules for managing services.

**Table 14-2** Modules Related to Service Management

![Image](/Images/14tab02.jpg)
#### Managing Systemd Services

Throughout this book you have used the service module a lot. This module
enables you to manage services, regardless of the init system that is
used, so it works with System-V init, with Upstart, as well as systemd.
In many cases, you can use the service module for any service-related
task.

If systemd specifics need to be addressed, you must use the systemd
module instead of the service module. Such systemd-specific features
include **daemon_reload** and **mask**. The **daemon_reload** feature
forces the systemd daemon to reread its configuration files, which is
useful after applying changes (or after editing the service files
directory, without using the Linux **systemctl** command). The **mask**
feature marks a systemd service in such a way that it cannot be started,
not even by accident. [Listing 14-1](#ch14.xhtml#list14_1) shows an
example where the systemd module is used to manage services.

**Listing 14-1** Using systemd Module Features

::: pre_1
    ---
    - name: using systemd module to manage services
      hosts: ansible2
      tasks:
      - name: enable service httpd and ensure it is not masked
        systemd:
          name: httpd
          enabled: yes
          masked: no
          daemon_reload: yes
:::

Given the large amount of functionality that is available in systemd,
the functions that are offered by the systemd services are a bit
limited, and for many specific features, you must use generic modules
such as file and command instead. An example is setting the default
target, which is done by creating a symbolic link using the file module.

#### Managing cron Jobs

The cron module can be used to manage cron jobs. A Linux cron job is one
that is periodically executed by the Linux crond daemon at a specific
time. The cron module can manage jobs in different ways:

• Write the job directly to a user's crontab

• Write the job to /etc/crontab or under the /etc/cron.d directory

• Pass the job to anacron so that it will be run once an hour, day,
week, month, or year without specifically defining when exactly

If you are familiar with Linux cron, using the Ansible cron module is
straightforward. [Listing 14-2](#ch14.xhtml#list14_2) shows an example
that runs the **fstrim** command every day at 4:05 and at 19:05.

**Listing 14-2** Running a cron Job


    ---
    - name: run a cron job
      hosts: ansible2
      tasks:
      - name: run a periodic job
        cron:
          name: "run fstrim"
          minute: "5"
          hour: "4,19"
          job: "fstrim"


As a result of this playbook, a crontab file is created for user root.
To create a crontab file for another user, you can use the **user**
attribute. Notice that while managing cron jobs using the cron module, a
**name** attribute is specified. This attribute is required for Ansible
to manage the cron jobs and has no meaning for Linux crontab itself. If,
for instance, you later want to remove a cron job, you must use the name
of the job as an identifier.

[Listing 14-3](#ch14.xhtml#list14_3) shows a sample playbook that
removes the job that was created in [Listing
14-2](#ch14.xhtml#list14_2). Notice that it just specifies **state:
absent** as well as the name of the job that was previously created; no
other parameters are required.

**Listing 14-3** Removing a cron Job Using the **name** Attribute

::: pre_1
    ---
    - name: run a cron job
      hosts: ansible2
      tasks:
      - name: run a periodic job
        cron:
          name: "run fstrim"
          state: absent
:::

#### Managing at Jobs {.h4}

Whereas you use Linux cron to schedule tasks at a regular interval, you
use Linux at to manage tasks that need to run once only. To interface
with Linux at, the Ansible at module is provided. [Table
14-3](#ch14.xhtml#tab14_3) gives an overview of the arguments it takes
to specify how the task should be executed.

::: group
**Table 14-3** at Module Arguments Overview

![Image](/Images/14tab03.jpg)

The most important point to understand when working with at is that it
is used to defined how far from now a task has to be executed. This is
done using **count** and **units**. If, for example, you want to run a
task five minutes from now, you specify the job with the arguments
**count: 5** and **units: minutes**. Also notice the use of the
**unique** argument. If set to yes, the task is ignored if a similar job
is scheduled to run already. [Listing 14-4](#ch14.xhtml#list14_4) shows
an example.

**Listing 14-4** Running Commands in the Future with at

::: pre_1
    ---
    - name: run an at task
      hosts: ansible2
      tasks:
        - name: run command and write output to file
          at:
            command: "date > /tmp/my-at-file"
            count: 5
            units: minutes
            unique: yes
            state: present
:::

In [Exercise 14-1](#ch14.xhtml#exe14_1) you practice your skills working
with the cron module.

::: box
**Exercise 14-1 Managing cron Jobs**

1\. Use your editor to create the playbook exercise141-1.yaml and give
it the following contents:

``` pre1
---
- name: run a cron job
  hosts: ansible2
  tasks:
  - name: run a periodic job
    cron:
      name: "run logger"
      minute: "0"
      hour: "5"
      job: "logger IT IS 5 AM"
```

2\. Use **ansible-playbook exercise141-1.yaml** to run the job.

3\. Use the command **ansible ansible2 -a "crontab -l"** to verify the
cron job has been added. The output should look as follows:

``` pre1
ansible2 | CHANGED | rc=0 >>
#Ansible: run logger
0 5 * * * logger IT IS 5 AM
```

4\. Create a new playbook with the name exercise141-2 that runs a new
cron job but uses the same name:

``` pre1
---
- name: run a cron job
  hosts: ansible2
  tasks:
  - name: run a periodic job
    cron:
      name: "run logger"
      minute: "0"
      hour: "6"
      job: "logger IT IS 6 AM"
```

5\. Run this new playbook by using **ansible-playbook
exercise141-2.yaml**. Notice that the job runs with a **changed**
status.

6\. Repeat the command **ansible ansible2 -a "crontab -l"**. This shows
you that the new cron job has overwritten the old job because it was
using the same name. Here is something important to remember: all cron
jobs should have a unique name!

7\. Write the playbook exercise141-3.yaml to remove the cron job that
you just created:

``` pre1
---
- name: run a cron job
  hosts: ansible2
  tasks:
  - name: run logger
    cron:
      name: "run logger"
      state: absent
```

8\. Use **ansible-playbook exercise141-3.yaml** to run the last
playbook. Next, use **ansible ansible2 -a "crontab -l"** to verify that
the cron job was indeed removed.

