# Ansible Configuration File

![](/images/ansible-config.png)
  
  
Ansible's configuration file is named `ansible.cfg`. It is used to set default Ansible behaviors per project, per user, or for all projects and users on a system. 

It can be stored in a project's directory, a user's home directory (if multiple user's want to have their own Ansible configuration), or in `/etc/ansible` (if the configuration will be the same for every user and every project). 

You can also specify Ansible settings in Ansible playbooks. The settings in a playbook take precedence over `ansible.cfg`. 

**ansible.cfg** precedence (Ansible uses the first one it finds and ignores the rest.)
1. `ANSIBLE_CONFIG` environment variable
2. `ansible.cfg` in current directory
3. `~/.ansible.cfg`
4. `/etc/ansible/ansible.cfg`

You can generate an example config file with the `ansible-config` command.  Here's how to do it with all directive are commented out:  
```bash
[ansible@control base]$ ansible-config init --disabled > ansible.cfg
```

See the help option for more info:  
```bash
📦[davidt@fedora ~]$ ansible-config init --help
usage: ansible-config init [-h] [-v] [-c CONFIG_FILE]
                           [-t {all,base,become,cache,callback,cliconf,connection,httpapi,inventory,lookup,netconf,shell,vars}]
                           [--format {ini,env,vars}] [--disabled]
                           [args ...]

positional arguments:
  args                  Specific plugin to target, requires type of plugin to
                        be set

options:
  -h, --help            show this help message and exit
  -v, --verbose         Causes Ansible to print more debug messages. Adding
                        multiple -v will increase the verbosity, the builtin
                        plugins currently evaluate up to -vvvvvv. A
                        reasonable level to start is -vvv, connection
                        debugging might require -vvvv. This argument may be
                        specified multiple times.
  -c, --config CONFIG_FILE
                        path to configuration file, defaults to first file
                        found in precedence.
  -t, --type {all,base,become,cache,callback,cliconf,connection,httpapi,inventory,lookup,netconf,shell,vars}
                        Filter down to a specific plugin type.
  --format, -f {ini,env,vars}
                        Output format for init
  --disabled            Prefixes all entries with a comment character to
                        disable them
```

Here is a very basic `ansible.cfg setup`:  
```ini
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

Privilege escalation parameters can also be specified in ansible.cfg, playbooks, and on the command line. 

Here is what I have for my lab, which is stored in the base of my Ansible directory:  
```ini
[defaults]
remote_user = usename
host_key_checking = false
inventory = inventory.yaml
vault_password_file = /root/vault-pass
roles_path: ./roles
[privilege_escalation] 
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

## `ansible-config` command

`ansible-config` is a useful command for viewing current Ansible settings and seeing available options. You can list available options that go in **ansible.cfg** file with `ansible-config list`.

To view your current config file the `ansible-config view` option can be used. And to view config path and other information, use the `--version` flag.

See the man page here:  
`man ansible-config`

Feel free to [reach out](https://www.linuxreader.com/contact/) for any clarifications needed. :)




