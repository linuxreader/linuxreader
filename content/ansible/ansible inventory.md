+++
title = 'Ansible Inventory and Ansible.cfg'
description = 'Setting up and Ansible environment'
+++
## Ansible projects

For small companies, you can use a single Ansible configuration. But for larger ones, it's a good idea to use different project directories. A project directory contains everything you need to work on a single project. Including:
- playbooks
- variable files
- task files
- inventory files
- ansible.cfg

*playbook*
An Ansible script written in YAML that enforce the desired configuration on manage hosts.

## Inventory
A file that Identifies hosts that Ansible has to manage. You can also use this to list and group hosts and specify host variables. Each project should have it's own inventory file. 

/etc/ansible/hosts 
- can be used for system wide inventory. 
- default if no inventory file is specified. 
- has some basic inventory formatting info if you forget) 
- Ansible will also target localhosts if no hosts are found in the inventory file. 
- It's a good idea to store inventory files in large environments in their own project folders. 

localhost is not defined in inventory. It is an implicit host that is usable and refers to the Ansible control machine. Using localhost can be a good way to verify the accessibility of services on managed hosts. 

### Listing hosts
List hosts by IP address or hostname. You can list a range of hosts in an inventory file as well such as `web-server[1:10].example.com`
```yaml
ansible1:2222 < specify ssh port if the host is not using the default port 22
ansible2
10.0.10.55
web-server[1:10].example.com
```

### Listing groups
You can list groups and groups of groups. See the groups web and db are included in the group "servers:children"
```yaml
ansible1
ansible2
10.0.10.55
web-server[1:10].example.com

[web]
web-server[1:10].example.com

[db]
db1
db2

[servers:children] <-- servers is the group of groups and children is the parameter that specifies child groups
web
db
```

There are 3 general approaches to using groups:

**Functional groups**
Address a specific group of hosts according to use. Such as web servers or database servers. 

**Regional host groups**
Used when working with region oriented infrastructure. Such as USA, Canada.

**Staging host groups**
Used to address different hosts according to the staging phase that the current environment is in. Such as testing, development, production. 

Undefined host groups are called implicit host groups. These are **all**, **ungrouped**, and **localhost**. Names making the meaning obvious. 


### Host variables
In older versions of Ansible you could define variables for hosts. This is no longer used. Example:
```yaml

[groupname:vars]
ansible=ansible_user
```

Variables are now set using **host_vars** and **group_vars** directories instead. 


### Multiple inventory files
Put all inventory files in a directory and specify the directory as the inventory to be used. For dynamic directories you also need to set the execution bit on the inventory file. 
