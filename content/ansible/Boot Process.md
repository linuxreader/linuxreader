+++
title = 'Boot Process'
description = 'Managing Boot Process with Ansible'
+++
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

When the rebooted host is back, the current playbook continues its
tasks. This scenario is shown in the example in [Listing
14-7](#ch14.xhtml#list14_7), where first all managed hosts are rebooted,
and after a successful reboot is issued, the message "successfully
rebooted" is shown. [Listing 14-8](#ch14.xhtml#list14_8) shows the
result of running this playbook. In [Exercise 14-2](#ch14.xhtml#exe14_2)
you can practice rebooting hosts using the reboot module.

**Listing 14-7** Rebooting Managed Hosts

::: pre_1
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
:::

**Listing 14-8** Verifying the Success of the reboot Module

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook listing147.yaml
    
    PLAY [reboot all hosts] *************************************************************************************************
    
    TASK [reboot hosts] *****************************************************************************************************
    changed: [ansible2]
    changed: [ansible1]
    changed: [ansible3]
    changed: [ansible4]
    changed: [ansible5]
    
    TASK [print message to show host is back] *******************************************************************************
    ok: [ansible1] => {
        "msg": "successfully rebooted"
    }
    ok: [ansible2] => {
        "msg": "successfully rebooted"
    }
    ok: [ansible3] => {
        "msg": "successfully rebooted"
    }
    ok: [ansible4] => {
        "msg": "successfully rebooted"
    }
    ok: [ansible5] => {
        "msg": "successfully rebooted"
    }
    
    PLAY RECAP **************************************************************************************************************
    ansible1                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible2                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible3                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible4                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible5                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
:::

::: box
**Exercise 14-2 Managing Boot State**

1\. As a preparation for this playbook, so that it actually changes the
default boot target on the managed host, use **ansible ansible2 -m file
-a "state=link src=/usr/lib/systemd/system/graphical.target
dest=/etc/systemd/system/default.target"**.

2\. Use your editor to create the file exercise142.yaml and write the
following playbook header:

``` pre1
---
- name: set default boot target and reboot
  hosts: ansible2
  tasks:
```

3\. Now you set the default boot target to multi-user.target. Add the
following task to do so:

``` pre1
- name: set default boot target
  file:
    src: /usr/lib/systemd/system/multi-user.target
    dest: /etc/systemd/system/default.target
    state: link
```

4\. Complete the playbook to reboot the managed hosts by including the
following tasks:

``` pre1
- name: reboot hosts
  reboot:
    msg: reboot initiated by Ansible
    test_command: whoami
- name: print message to show host is back
  debug:
    msg: successfully rebooted
```

5\. Run the playbook by using **ansible-playbook exercise142.yaml**.

6\. Test that the reboot was issued successfully by using **ansible
ansible2 -a "systemctl get-default"**.
:::
