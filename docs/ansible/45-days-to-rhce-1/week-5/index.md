# 

This week, I want to review more RHCE topics and do lots of labs. I think I have a decent understanding of where to find answers during the exam. Now my goal is to shorten the time it takes do do exam tasks by practicing labs and knowing things without looking them up. 

## Include and import

What can you do with them?

Includes and imports  are used to include and import files such as: 
- Roles
- Playbooks
- Tasks
- Variable

Just make sure to use them at the level of the type of file you are calling. IE, Playbooks cannot be imported at the task level, etc. 
### What are the main differences of Include and Import?

#### Include
Includes are dynamically processed when Ansible reaches the content. This means you can't use Ansible features such as `--start-at-task` with include_tasks because Ansible doesn't see those yet. 
#### Import

Cannot be used with loops. When statements are used on each item, rather than on an entire list of tasks. I treat these as if there are actually in the file that is calling them. 

## Inventory

You can specify multiple inventories at the command line. Like:
```
ansible-playbook -i inventory -i inventory2 playbook.yaml
```

You can also specify a directory, which will use all files in the directory as inventory files:
```
ansible-playbook -i inventory-dir playbook.yaml
```

/etc/ansible/hosts has some examples for quick reference.
## Jinja2
Great docs for more details: https://jinja.palletsprojects.com/en/stable/

### Practicing control structures

#### Add host entries to /etc/hosts

I thought I could add the lines with a for loop like this:
```j2
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
{% for node in groups['all'] %}
{{ ansible_facts['default_ipv4']['address'] }} {{ node }}
{% endfor %}         
```

But this inserted the IP of the host that the play was running on.
```bash
ansible3 | CHANGED | rc=0 >>
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.122.5 ansible2
192.168.122.5 ansible5
192.168.122.5 ansible3
192.168.122.5 ansible4
```

So I need a way to detect the ip of each node dynamically. Found out how by using hostvars:
```j2
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
{% for node in groups['all'] %}
{{ hostvars[node]['ansible_default_ipv4']['address'] }} {{ node }}
{% endfor %}
```

This creates the output we want:
```bash
ansible3 | CHANGED | rc=0 >>
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.122.4 ansible2
192.168.122.7 ansible5
192.168.122.5 ansible3
192.168.122.6 ansible4
```

The Van Vugt cert guide has a similar solution listed like this:
```jinja
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }} {{ hostvars[host]['ansible_fqdn'] }} {{ hostvars[host]['ansible_hostname'] }}
{% endfor %}
```

## Loops and Items

## Optimizing Ansible Processing

## Packages Repositories and Subscriptions

## Playbooks