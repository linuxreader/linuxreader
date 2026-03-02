# The Cheatcode

![](../images/featured.png)

This week, I focused a lot on, "I can't remember this, [where can I find the answers?](https://www.davidwrites.xyz/micro/you-need-to-learn-man-pages/)"

As always, if you want email updates on this series, subscribe to my newsletter:
<script async src="https://eomail5.com/form/d9b7d338-dbf3-11f0-bae4-65187d72ac9a.js" data-form="d9b7d338-dbf3-11f0-bae4-65187d72ac9a">
</script>

RedHat provides a cheat code during the exam. Documentation. I intend to exploit that as much as I can. Because rote memorization sucks...

I ran through topics in ABC order. Doing labs and finding the right documentation in case I need help. 

Topics:
- Ad Hoc commands
- Ansible.cfg
- Ansible Vault
- Boot process
- Deploying files
- Handlers, testing, and blocks
- Hostname patterns

Click the header to each section to visit my notes for that topic. 
## [Ad Hoc commands](https://www.davidwrites.xyz/notes/rhce-notes/adhoccommands/)

Official doc [here](https://docs.ansible.com/projects/ansible/latest/command_guide/intro_adhoc.html). 

The man page for the `ansible` command:  
```bash
man ansible
```

Ad Hoc commands are not a complicated topic. You can use them to gather information from a server or to make changes.

These are useful for scripts. This one provisions new VMS for my lab. Then bootstraps the VMs for Ansible.
```bash
#!/bin/bash
ansible-playbook kvm_provision.yaml -e vm=podman-01 -e ip_addr=192.168.122.3 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:01 &&
ansible-playbook playbooks/bootstrap.yaml   -e target=podman-01   -e ansible_user=root

ansible-playbook kvm_provision.yaml -e vm=podman-02 -e ip_addr=192.168.122.4 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:02 &&
ansible-playbook playbooks/bootstrap.yaml   -e target=podman-02   -e ansible_user=root

ansible-playbook kvm_provision.yaml -e vm=ansible1 -e ip_addr=192.168.122.5 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:03 &&
ansible-playbook playbooks/bootstrap.yaml   -e target=ansible1   -e ansible_user=root

ansible-playbook kvm_provision.yaml -e vm=ansible2 -e ip_addr=192.168.122.6 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:04 &&
ansible-playbook playbooks/bootstrap.yaml   -e target=ansible2   -e ansible_user=root

ansible-playbook kvm_provision.yaml -e vm=ansible3 -e ip_addr=192.168.122.7 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:05 &&
ansible-playbook playbooks/bootstrap.yaml   -e target=ansible3   -e ansible_user=root
```

## [Ansible.cfg](https://www.davidwrites.xyz/notes/rhce-notes/ansibleconfig/)

It's going to be important to generate an ansible.cfg file on the fly. You can get example options from the `ansible-config` command. 

### `ansible-config` command

Useful command for viewing current Ansible settings and seeing available options. 

See the man page here:  
`man ansible-config`

List available options that go in **ansible.cfg** file:  
`ansible-config list` 

View your current config file:  
`ansible-config view`

View config path and other information:  
`ansible-config --version`

Generate an **ansible.cfg** file with all entries commented out:  
`ansible-config init --disabled > ansible.cfg`

This prints out a large file. You'd have a lot to go through during the exam to get a workable **ansible.cfg**. So it's probably best to create a bare minimum working version by hand.

Here is what I have for my lab:
```yml
❯ cat ansible.cfg 
[defaults]
remote_user = david
host_key_checking = false
inventory = inventory.yaml
vault_password_file = ~/vault-pass
roles_path: ./roles
[privilege_escalation] 
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```


## [Ansible Vault](https://www.davidwrites.xyz/notes/rhce-notes/ansiblevault/)

### `ansible-vault` command

View options and information:  
```
man ansible-vault
```

Create a vault using a vault password file:  
```bash
echo "lima-bean" > vault-pass2

ansible-vault create secrets2.yml --vault-password-file vault-pass2
```

Edit the file:  
```
ansible-vault edit secrets2.yml --vault-password-file vault-pass2
```

If you have the vault listed as a variable file. Either in a playbook or under a variable folder such as: `group_vars/all/secrets2.yml`. You can list the vault password file to decrypt globally in **ansible.cfg**:
```
vault_password_file = ~/vault-pass2
```

You can also add it as an option when you run `ansible-playbook` or have it in the playbook itself.

The man page also shows you how to decrypt a vault, view the contents, encrypt an existing file, and change a vault's password.

### Vault options for `ansible-playbook` command
Use --help and `grep` to quickly see vault options:
```bash
$ ansible-playbook --help | grep vault
                        [-e EXTRA_VARS] [--vault-id VAULT_IDS] [-J |
                        --vault-password-file VAULT_PASSWORD_FILES] [-f FORKS]
  --vault-id VAULT_IDS  the vault identity to use. This argument may be
  --vault-password-file, --vault-pass-file VAULT_PASSWORD_FILES
                        vault password file
  -J, --ask-vault-password, --ask-vault-pass
                        ask for vault password
```

### Vault options for ansible.cfg
Use the same strategy to quickly see vault options to set globally in **ansible.cfg**:
```bash
$ ansible-config list | grep vault
  - This controls whether an Ansible playbook should prompt for a vault password.
  - key: ask_vault_pass
  name: Ask for the vault password(s)
  description: The vault_id to use for encrypting by default. If multiple vault_ids
    are provided, this specifies which to use for encryption. The ``--encrypt-vault-id``
  - key: vault_encrypt_identity
    key: defaults.vault_encrypt_identity
  description: The label to use for the default vault id label in cases where a vault
  - key: vault_identity
    key: defaults.vault_identity
  description: A list of vault-ids to use by default. Equivalent to multiple ``--vault-id``
  - key: vault_identity_list
  name: Default vault ids
    key: defaults.vault_identity_list
  description: If true, decrypting vaults with a vault id will only try the password
    from the matching vault-id.
  - key: vault_id_match
  name: Force vault id match
    key: defaults.vault_id_match
  - The vault password file to use. Equivalent to ``--vault-password-file`` or ``--vault-id``.
  - key: vault_password_file
    key: defaults.vault_password_file
  description: The salt to use for the vault encryption. If it is not provided, a
  - key: vault_encrypt_salt
    YAML or JSON or vaulted versions of these.
```

## [Boot process](https://www.davidwrites.xyz/notes/rhce-notes/bootprocess/)
Managing the boot process will be more challenging because there are no modules that manage this. A solid understanding of the boot process and **systemd** will be needed here. 

### Setting the default systemd target
Say we want to change the default systemd target from multi-user.target to graphical.target.

```bash
$ man -k systemd
```

The relevant man pages here may be:
```bash
systemd.target (5)   - Target unit configuration
systemd.unit (5)     - Unit configuration
```

I didn't find exact instructions on how to change the default target. But there is information on symlinking in the systemd.unit. And also mentions using the `systemctl` command to make the symlink. 

```bash
 As another example, default.target —
       the default system target started at boot — is commonly aliased
       to either multi-user.target or graphical.target to select what
       is started by default.
```

Let's check options for the systemctl command:
```bash
$ systemctl --help | grep default
  get-default                         Get the name of the default target
  set-default TARGET                  Set the default target
```

#### Using the systemctl method:

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

If you have a solid understanding of Ansible this is doable. In Sander Van Vugt's cert guide, he mentions just making the symlink with the `file` module:
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

#### Using the symlink method
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

Come to think of it, the command module with the `systemctl` command is looking a lot easier now. It's good to know both methods in case the exam objectives throw you for a loop.

### Rebooting
You may need to use the reboot module for changes to take effect. The module documentation covers everything you need to know with examples:
```
ansible-doc reboot
```

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

## [Deploying Files](https://www.davidwrites.xyz/notes/rhce-notes/deployingfiles/)

### **Stat** module
The outputs of the stat module can be used as a variable to test files. 

You can see all of the outputs at:
```
ansible-doc stat
```

Example from the docs:
```yaml
- name: Get stats of the FS object
  ansible.builtin.stat:
    path: /path/to/something
  register: sym

- name: Print a debug message
  ansible.builtin.debug:
    msg: "islnk isn't defined (path doesn't exist)"
  when: sym.stat.islnk is not defined
```

### Tests
Another page that will be useful to find quickly during the exam is the [tests page](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_tests.html#tests). Just open the documentation site and type "tests". I may end up just opening this right away during the exam. 

It's probably worth working with tests a bit. I wasn't sure exactly how to test against a bool when I tried but ended up with this:
```yaml
---
- name: stat module test
  hosts: ansible1
  tasks:
  - command: touch /tmp/statfile
  - stat:
      path: /tmp/statfile
    register: st
  - name: show current values
    debug:
      msg: current value of the st variable is {{ st }}
  - fail:
      msg: "File is not writeable"
    when: not st.stat.writeable
```

The `not st.stat.writeable` threw me for a loop cause every scenario I tried thought it was a string. 
### Modules to remember:
**fetch** - Move a file from a the remote host to the ansible control node.  
**synchronize** - Wrapper around `rsync` to sync files.   
**copy** - Move a file from the control node to the managed host.  
**lineinfile** - Copy a single line of text to a file.  
**blockinfile** - Copy multiple lines to a file.  
**template** - Copy templated file to the host. (indempotent)  
**acl** - Work with system ACLs.  
**replace** - Replaces strings in files based on regex.  

## [Handlers, testing, and blocks](https://www.davidwrites.xyz/notes/rhce-notes/handlers/)

A handler won't run if a task in the playbook fails. 

Use **force_handlers** to make handlers run even if a tasks fails. **ignore_errors** can also accomplish this.

Handlers run after the play is finished. If you have multiple plays, the handlers for the first play will run before the second play begins. 

Useful doc to search: [Error handling in playbooks](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_error_handling.html)

The **fail** module also exists and let's you specify a clear failure message:  
```
ansisible-doc fail
```

A good blocks [document](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_blocks.html#block-error-handling) exists and covers blocks, rescue, and always. 

## [Hostname patterns](https://www.davidwrites.xyz/notes/rhce-notes/hostnamepatterns/)
Match hosts and groups when running Ansible commands. This is useful to match a more specific set of hosts in you inventory. See [patterns.](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_patterns.html)
## What now?
Getting distracted has been my biggest challenge this week. And I did not study as much as I would have liked. There are just so many other fun things to do besides study. 

Next week I'll take a practice exam to gauge my progress. I think knowing problem areas will help push me forward. 

