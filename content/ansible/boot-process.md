---
title: 'Boot Process'
description: 'Managing Boot Process with Ansible'
showDate: false
---

### Managing the Boot Process

No modules for managing boot process.

**file** module
- manage the systemd boot targets

**lineinfile** module 
- manage the GRUB configuration.

**reboot** module
- enables you to reboot a host and pick up after the reboot at the exact same location.

#### Managing Systemd Targets

To manage the default systemd target:
- /etc/systemd/system/default.target file must exist as a symbolic link to the desired default target.

```bash
ls -l /etc/systemd/system/default.target
    lrwxrwxrwx. 1 root root 37 Mar 23 05:33 /etc/systemd/system/default.target -> /lib/systemd/system/multi-user.target
```

```yaml
---
- name: set default boot target
    hosts: ansible2
    tasks:
    - name: set boot target to graphical
      file:            
        src: /usr/lib/systemd/system/graphical.target
        dest: /etc/systemd/system/default.target
        state: link
```

#### Rebooting Managed Hosts

**reboot** module.
- Restart managed nodes. 

**test_command** argument
- Verify the renewed availability of the managed hosts
- Specifies an arbitrary command that Ansible should run successfully on the managed hosts after the reboot. The success of this command indicates that the rebooted host is available again.

Equally useful while using the reboot module are the arguments that
relate to timeouts. The reboot module uses no fewer than four of them:

• **connect_timeout:** The maximum seconds to wait for a successful
connection before trying again

• **post_reboot_delay:** The number of seconds to wait after the
**reboot** command before trying to validate the managed host is
available again

• **pre_reboot_delay:** The number of seconds to wait before actually
issuing the reboot

• **reboot_timeout:** The maximum seconds to wait for the rebooted
machine to respond to the **test** command

- When the rebooted host is back, the current playbook continues its tasks. 

```
---
    - name: reboot all hosts
      hosts: all
      gather_facts: no
      tasks:
      - name: reboot hosts
        reboot:
          msg: reboot initiated by Ansible
          test_command: whoami
      - name: print message to show host is back
        debug:
          msg: successfully rebooted
```

Change target and reboot:
```yaml
---
- name: Set default target to graphical
  hosts: practice
  become: yes
  tasks:
  - name: Link graphical.target to default.target
    file:
      src: /usr/lib/systemd/system/graphical.target
      dest: /etc/systemd/system/default.target
      state: link

  - name: reboot
    reboot:
      test_command: whoami
      msg: rebooting...

  - name: print success message
    debug:
      msg: Reboot successful                            
```

Test that the reboot was issued successfully by using 
```
ansible practice -a "systemctl get-default"
```

### Lab: Managing the Boot Process and Services

Create a playbook that:
- Runs a command before the reboot, 
- Schedules a cron job at the next reboot
- Using that cron job, ensures that after rebooting a specific command is used as well. 
- To make sure you see what happens when, you work with a temporary file to which lines are added.

```yaml
❯ cat labs/reboot-msg.yaml
---
- name: reboot-msg
  hosts: practice
  tasks:
  - name: add a line to a file before rebooting
    lineinfile:
      create: true
      state: present
      path: /tmp/rebooted
      insertafter: EOF
      line: rebooted at {{ ansible_facts['date_time']['time'] }}:{{ ansible_facts['date_time']['second'] }}

  - name: run a cron job on reboot
    cron:
      name: "run on reboot"
      state: present
      special_time: reboot
      job: "echo rebooted at $(date) >> /tmp/rebooted"

  - name: reboot managed host
    reboot:
      msg: reboot initiated
      test_command: whoami

  - name: print reboot success message
    debug:
      msg: reboot success
```

Time, including a second indicator, is written using two Ansible facts. Not one single fact has the time in an hh:mm:ss format.

Bash shell command substitution is possible in the cron module because commands are executed by a bash shell.

This is not possible with the lineinfile module because the commands are not processed by a shell. 

See results:
```
ansible practice -a "cat /tmp/rebooted"
```

### Lab: cron job

Write a playbook according to the following specifications:

• The cron module must be used to restart your managed servers at 2 a.m.
each weekday.
• After rebooting, a message must be written to syslog, with the text
"CRON initiated reboot just completed."
• The default systemd target must be set to multi-user.target.
• The last task should use service facts to show the current version of the cron process.

```yaml
---
- name: cron job
  hosts: ansible1
  tasks:
  - name: cron job to restart servers at 2am each weekday
    cron:
      name: restart servers
      weekday: MON-FRI
      hour: 2
      job: "reboot"

  - name: After reboot send log message to syslog
    cron: 
      name: Print reboot message to syslog
      special_time: reboot
      job: "logger Sytem rebooted"  

  - name: set the default systemd target to multi-user
    file:
      src: /usr/lib/systemd/system/multi-user.target
      dest: /etc/systemd/system/default.target
      state: link  

  - name: populate service facts
    service_facts:

  - name: show current version of cron process using service facts
    debug:
      var: ansible_facts.services['crond.service']

```

```bash
❯ ansible ansible1 -a "crontab -l"
[WARNING]: Host 'ansible1' is using the discovered Python interpreter at '/usr/bin/python3.12', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.20/reference_appendices/interpreter_discovery.html for more information.
ansible1 | CHANGED | rc=0 >>
#Ansible: run on reboot
@reboot echo rebooted at $(date) >> /tmp/rebooted
#Ansible: restart servers
* 2 * * MON-FRI reboot
#Ansible: Print reboot message to syslog
@reboot logger Sytem rebooted
```