# Boot Process

![](/images/boot-process.png)



There are no specific modules for managing a linux system's boot process with Ansible. However, you can use your pre-existing knowledge of the boot process to manage related files. You can use the **file** module to manage systemd boot targets. Or **lineinfile** to manage the GRUB configuration. 

The **reboot** module does let you to reboot a host though. And it will pick up after the reboot at the exact same location in a playbook.

## Managing Systemd Targets (symlink method)

To manage the default systemd target, the `/etc/systemd/system/default.target` file must exist as a symbolic link to the desired default target:  

```bash
ls -l /etc/systemd/system/default.target
    lrwxrwxrwx. 1 root root 37 Mar 23 05:33 /etc/systemd/system/default.target -> /lib/systemd/system/multi-user.target
```

Let's use the **file** module to change the symlink, targeting graphical mode instead:  
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

## Managing Systemd Targets (systemctl method)  

This method isn't great because you have to run around with your head cut off trying to figure out a way to make it indempotent.
```yml
- block:
  - name: Install GUI package group
    ansible.builtin.dnf:
      name: "@Server with GUI"
      state: present

  - name: get current target
    ansible.builtin.command: "systemctl get-default"
    changed_when: false
    register: systemdefault

  - name: Set default to graphical target
    ansible.builtin.command: "systemctl set-default graphical.target"
    when: "'graphical' not in systemdefault.stdout"
    changed_when: true

  when: gui_enabled | bool
```

## Rebooting Managed Hosts

As mentioned, the **reboot** module is used to restart managed nodes. And the **test_command** argument verifies that the host is available after the reboot. **test_command** specifies a command that Ansible should run on the managed hosts after the reboot. If the command is successful, then Ansible knows to continue. 

There are also arguments related to timeouts:  

| Argument              | Purpose                                                                                     |
| --------------------- | ------------------------------------------------------------------------------------------- |
| **connect_timeout**   | Maximum seconds to wait for a successful connection before trying again.                    |
| **post_reboot_delay** | Seconds to wait after the **reboot** command before trying to check if a host is available. |
| **pre_reboot_delay**  | Seconds to wait before issuing the reboot.                                                  |
| **reboot_timeout**    | Maximum seconds to wait for the rebooted machine to respond to the **test** command.        |

Let's change default target and reboot the system:  
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

You can test that the reboot was issued successfully by using:  
```
ansible some-host -a "systemctl get-default"
```

## Finding help

You may need to search [man pages](https://www.linuxreader.com/tools/you-need-to-learn-man-pages/) to find more info:  
```bash
$ man -k systemd
```

The relevant man pages here may be:  
```bash
systemd.target (5)   - Target unit configuration
systemd.unit (5)     - Unit configuration
```

I didn't find exact instructions on how to change the default target. But there is information on symlinking in the systemd.unit. And also mentions using the `systemctl` command to make the symlink:  
```bash
 As another example, default.target —
       the default system target started at boot — is commonly aliased
       to either multi-user.target or graphical.target to select what
       is started by default.
```

If you can't remember the search path, the paths are listed at the top of the systemd-unit man page:
```bash
$ man systemd.unit
```

```bash
  System Unit Search Path
       /etc/systemd/system.control/*
       /run/systemd/system.control/*
       /run/systemd/transient/*
       /run/systemd/generator.early/*
       /etc/systemd/system/*
       /etc/systemd/system.attached/*
       /run/systemd/system/*
       /run/systemd/system.attached/*
       /run/systemd/generator/*
       ...
       /usr/lib/systemd/system/*
       /run/systemd/generator.late/*
```

If you can remember that you need to symlink a unit to **/etc/systemd/system/default.target** then I think you'll be good here. 

You can also check options for the `systemctl` command:  
```bash
$ systemctl --help | grep default
  get-default                         Get the name of the default target
  set-default TARGET                  Set the default target
```

You'll also want to remember where systemd keeps all of the default unit files. List is listed at the top of the systemd man page:
```bash
$ man systemd | head
SYSTEMD(1)                      systemd                     SYSTEMD(1)

NAME
       systemd, init - systemd system and service manager

SYNOPSIS

       /usr/lib/systemd/systemd [OPTIONS...]

       init [OPTIONS...] {COMMAND}

```

The file module documentation has a symlink example at the bottom:
```bash
ansible-doc file
```

```yaml
- name: Create a symbolic link
  ansible.builtin.file:
    src: /file/to/link/to
    dest: /path/to/symlink
    owner: foo
    group: foo
    state: link
```

Just use similar methods (managing settings file) to change GRUB settings and other boot options. Feel free to [reach out](https://www.linuxreader.com/contact/) for any clarifications needed. :)