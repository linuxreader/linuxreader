+++
title = 'Ansible.cfg'
description = 'Ansible.cfg'
+++
## ansible.cfg
You can store this in a project's directory or a user's home directory, in the case that multiple user's want to have their own Ansible configuration. Or in /etc/ansible if the configuration will be the same for every user and every project. You can also specify these settings in Ansible playbooks. The settings in a playbook take precedence over the .cfg file. 

**ansible.cfg precedence** (Ansible uses the first one it finds and ignores the rest.)
1. ANSIBLE_CONFIG environment variable
2. ansible.cfg in current directory
3. ~/.ansible.cfg
4. /etc/ansible/ansible.cfg

Generate an example config file in the current directory.  All directive are commented out by default:
`[ansible@control base]$ ansible-config init --disabled > ansible.cfg`

Include existing plugin to the file:
`ansible-config init --disabled -t all > ansible.cfg`

This generates an extremely large file. So I'll just show Van Vugt's example in .ini format:
```bash
[defaults] <-- General information
remote_user = ansible <--Required
host_key_checking = false <-- Disable SSH host key validity check
inventory = inventory

[privilege_escalation] <-- Define how ansible user requires admin rights to connect to hosts
become = True <-- Escalation required
become_method = sudo
become_user = root <-- Escalated user
become_ask_pass = False <-- Do not ask for escalation password
```

Privilege escalation parameters can be specified in ansible.cfg, playbooks, and on the command line. 

