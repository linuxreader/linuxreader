### Managing the Boot Process and Services Advanced Exercise

The advanced exercise, 14-3, is a relatively easy task this time. You
are guided through the procedure of creating a playbook that runs a
command before the reboot, schedules a cron job at the next reboot, and,
using that cron job, ensures that after rebooting a specific command is
used as well. To make sure you see what happens when, you work with a
temporary file to which lines are added.

::: box
**Exercise 14-3 Managing the Boot Process and Services**

1\. Use your editor to create the file exercise143.yaml and write the
playbook header as follows:

``` pre1
---
- name: exercise143
  hosts: ansible2
  tasks:
```

2\. Write the first task. This task is not really functional but enables
you to check what is happening in the remaining tasks in the playbook.
In this task, you use the lineinfile module to add a line to the end of
the check file /tmp/rebooted. Notice how the time, including a second
indicator, is written using two Ansible facts. It's required to operate
this way because not one single fact has the time in an hh:mm:ss format.
Write this task code as follows:

``` pre1
- name: add a line to file before rebooting
  lineinfile:
    create: true
    state: present
    path: /tmp/rebooted
    insertafter: EOF
    line: rebooted at {{ ansible_facts[’date_time’][’time’] }}:{{ ansible_facts[’date_time’][’second’] }}
```

3\. At this point, you can add a cron job that runs at reboot. The job
that runs will add a message to the /tmp/rebooted file, and to make sure
that it is working correctly, you use bash shell command substitution to
print the results of the Linux **date** command. Notice that this is
possible in the cron module because commands are executed by a bash
shell, but it's not possible in the previous task that uses the
lineinfile module because in that task the commands are not processed by
a shell. Now add the task as follows:

``` pre1
- name: run a cron job on reboot
  cron:
    name: "run on reboot"
    state: present
    special_time: reboot
    job: "echo rebooted at $(date) >> /tmp/rebooted"
```

4\. Add another task that uses the reboot module to reboot the managed
host:

``` pre1
- name: reboot managed host
  reboot:
    msg: reboot initiated by Ansible
    test_command: whoami
- name: show reboot success
  debug:
    msg: just rebooted successfully
```

5\. Now that the playbook is complete, you can run it by using
**ansible-playbook exercise143.yaml**. Notice that it needs a minute
because it has to wait until the target host is back. When it is back,
use **ansible ansible2 -a "cat /tmp/rebooted"**. You then see that both
reboot messages are written, and there is about 30 seconds between the
pre-reboot and the post-reboot commands.
:::


### Lab 14-1

Write a playbook according to the following specifications:

• The cron module must be used to restart your managed servers at 2 a.m.
each weekday.

• After rebooting, a message must be written to syslog, with the text
"CRON initiated reboot just completed."

• The default systemd target must be set to multi-user.target.

• The last task should use service facts to show the current version of
the cron process.
