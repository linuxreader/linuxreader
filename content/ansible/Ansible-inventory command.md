+++
title = 'Ansible-inventory command'
description = 'Ansible-inventory command'
+++
### Inventory commands:

To view the inventory, specify the inventory file such as ~/base/inventory in the command line. You can name the inventory file anything you want. You can also set the default in the ansible.cfg file.

View the current inventory:
`ansible -i inventory <pattern> --list-hosts` 

List inventory hosts in JSON format:
`ansible-inventory -i inventory --list`

Display overview of hosts as a graph:
`ansible-inventory -i inventory --graph`

In our lab example:
```bash
[ansible@control base]$ pwd
/home/ansible/base

[ansible@control base]$ ls
inventory

[ansible@control base]$ cat inventory
ansible1
ansible2

[web]
web1
web2

[ansible@control base]$ ansible-inventory -i inventory --graph
@all:
  |--@ungrouped:
  |  |--ansible1
  |  |--ansible2
  |--@web:
  |  |--web1
  |  |--web2

[ansible@control base]$ ansible-inventory -i inventory --list
{
    "_meta": {
        "hostvars": {}
    },
    "all": {
        "children": [
            "ungrouped",
            "web"
        ]
    },
    "ungrouped": {
        "hosts": [
            "ansible1",
            "ansible2"
        ]
    },
    "web": {
        "hosts": [
            "web1",
            "web2"
        ]
    }
}

[ansible@control base]$ ansible -i inventory all --list-hosts
  hosts (4):
    ansible1
    ansible2
    web1
    web2
    
[ansible@control base]$ ansible -i inventory ungrouped --list-hosts
  hosts (2):
    ansible1
    ansible2

```

#### Using the ansible-inventory Command

- default output of a dynamic inventory script is unformatted. 
- To show formatted JSON output of the scripts, you can use the `ansible-inventory` command. 
- Apart from the `--list` and `--host` options, this command also uses the `--graph` option to show a list of hosts, including the host groups they are a member of. 

```bash
    [ansible@control rhce8-book]$ ansible-inventory -i listing101.py --graph
    [WARNING]: A duplicate localhost-like entry was found (localhost). First found
    localhost was 127.0.0.1
    @all:
      |--@ungrouped:
      |  |--127.0.0.1
      |  |--192.168.4.200
      |  |--192.168.4.201
      |  |--192.168.4.202
      |  |--ansible1
      |  |--ansible1.example.com
      |  |--ansible2
      |  |--ansible2.example.com
      |  |--control
      |  |--control.example.com
      |  |--localhost
      |  |--localhost.localdomain
      |  |--localhost4
      |  |--localhost4.localdomain4
      |  |--localhost6
      |  |--localhost6.localdomain6
```
