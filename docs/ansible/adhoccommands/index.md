# Ad Hoc Ansible Commands

Ad hoc commands are ansible tasks you can run against managed hosts without the need of a playbook or script. These are used for bringing nodes to their desired states, verifying playbook results, and verifying nodes meet any needed criteria/pre-requisites. These must be ran as the ansible user (whatever your remote_user directive is set to under \[defaults\] in ansible.cfg)

Run the user module with the argument name=lisa on all hosts to make sure the user "lisa" exists. If the user doesn't exist, it will be created on the remote system:
`ansible all -m user -a "name=lisa"`

`{command} {host} -m {module} -a {"argument1 argument2 argument3"}`

This Ad Hoc command created user "Lisa" on ansible1 and ansible2. If we run the command again, we get "SUCCESS" on the first line instead of "CHANGED". Which means the hosts already meet the requirements:
```bash
[ansible@control base]$ ansible all -m user -a "name=lisa"
```

*indempotent* 
Regardless of current condition, the host is brought to the desired state. Even if you run the command multiple times. 

Run the command `id lisa` on all managed hosts:
```bash
[ansible@control base]$ ansible all -m command -a "id lisa"
```

Here, the command module is used to run a command on the specified hosts. And the output is displayed on screen. To note, this does not show up in our ansible user's command history on the host:
```bash
[ansible@ansible1 ~]$ history
```

Remove the user lisa from all managed hosts:
```bash
[ansible@control base]$ ansible all -m user -a "name=lisa state=absent"
```

You can also use the `-u` option to specify the Ansible user that Ansible will use to run the command. Remember, with no modules specified, ansible uses the `command` module:
`ansible all -a "free -m" -u david`
`
## Ad hoc commands in Scripts

Follow normal bash scripting guidelines to run ansible commands in a script:
```bash
[ansible@control base]$ vim httpd-ansible.sh
```

Let's set up a script that installs and starts/enables httpd, creates a user called "anna", and copies the ansible control node's /etc/hosts file to /tmp/ on the managed nodes:

```
#!/bin/bash

ansible all -m yum -a "name=httpd state=latest"
ansible all -m service -a "name=httpd state=started enabled=yes"
ansible all -m user -a "name=anna"
ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"

```

```bash
[ansible@control base]$ chmod +x httpd-ansible.sh
[ansible@control base]$ ./httpd-ansible.sh 
web2 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web2: Name or service not known",
    "unreachable": true
}
web1 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web1: Name or service not known",
    "unreachable": true
}
ansible1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
ansible2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
... <-- Results truncated
```

And from the ansible1 node we can verify:
```bash
[ansible@ansible1 ~]$ cat /etc/passwd | grep anna
anna:x:1001:1001::/home/anna:/bin/bash
```

```bash
[ansible@ansible1 ~]$ cat /tmp/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.124.201 ansible1
192.168.124.202 ansible2
```

View a file from a managed node:
`ansible ansible1 -a "cat /somfile.txt"`

`ansible` uses the command module by default. 

Verify host names
```bash
ansible all -a "hostname"
```

Run the same on 5 hosts at a time
```bash
ansible all -a "hostname" -f 5
```

View ansible_facts for ansible1 server:
```bash
ansible ansible1 -m setup
```

Check to see if dates are in sync:
```bash
ansible all -a "date"
```

## More Ad Hoc commands

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

