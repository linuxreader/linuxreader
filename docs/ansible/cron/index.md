# 

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

### Cron
I struggled a bit with the lab for this [section](../../../notes/rhce-notes/bootprocess/). I do not remember learning about service facts. Nor did I remember about the `logger` command. Curse me for procrastination for so long after RHCSA.

I found some useful documentation though. 

Cron specific documentation:
```
man cron
```

```
ansible-doc cron
```

**service_facts** module:  
```
ansible-doc service_facts
```

`logger` command for printing log messages:
```
man logger
```
