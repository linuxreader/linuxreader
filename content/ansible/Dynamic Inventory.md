+++
title = 'Dynamic Inventory'
description = 'Using Dynamic Ansible Inventories'
+++
### Dynamic inventory scripts
A script is used to detect inventory hosts so that you do not have to manually enter them. This is good for larger environments. You can find community provided dynamic inventory scripts that come with an .ini file that provides information on how to connect to a resource. 

Inventory scripts must include --list and --host options and output must be JSON formatted. Here is an example from [sandervanvught](https://github.com/sandervanvugt/rhce8-book/blob/master/listing33.py) that generates an inventory script using /etc/hosts:
```python
[ansible@control base]$ cat inventory-helper.py

#!/usr/bin/python

from subprocess import Popen,PIPE
import sys

try:
     import json
except ImportError:
     import simplejson as json



result = {}

result['all'] = {}



pipe = Popen(['getent', 'hosts'], stdout=PIPE, universal_newlines=True)


result['all']['hosts'] = []

for line in pipe.stdout.readlines():
    s = line.split()
    result['all']['hosts']=result['all']['hosts']+s


result['all']['vars'] = {}


if len(sys.argv) == 2 and sys.argv[1] == '--list':
    print(json.dumps(result))

elif len(sys.argv) == 3 and sys.argv[1] == '--host':
    print(json.dumps({}))

else:
    print("Requires an argument, please use --list or --host <host>")
```

When ran on our sample lab:
```bash
[ansible@control base]$sudo python3 ./inventory-helper.py
Requires an argument, please use --list or --host <host>

[ansible@control base]$ sudo python3 ./inventory-helper.py --list
{"all": {"hosts": ["127.0.0.1", "localhost", "localhost.localdomain", "localhost4", "localhost4.localdomain4", "127.0.0.1", "localhost", "localhost.localdomain", "localhost6", "localhost6.localdomain6", "192.168.124.201", "ansible1", "192.168.124.202", "ansible2"], "vars": {}}}

```

To use a dynamic inventory script:
```bash
[ansible@control base]$ chmod u+x inventory-helper.py 
[ansible@control base]$ sudo ansible -i inventory-helper.py all --list-hosts
[WARNING]: A duplicate localhost-like entry was found (localhost). First found localhost was 127.0.0.1
  hosts (11):
    127.0.0.1
    localhost
    localhost.localdomain
    localhost4
    localhost4.localdomain4
    localhost6
    localhost6.localdomain6
    192.168.124.201
    ansible1
    192.168.124.202
    ansible2
```

#### Configuring Dynamic Inventory

dynamic inventory
- script that can be used to detect whether new hosts have been added to the managed environment.

- Dynamic inventory scripts are provided by the community and exist for many different environments. 
- easy to write your own dynamic inventory script. 
- The main requirement is that the dynamic inventory script works with a **\--list** and a **\--host \<hostname\>** option and produces its output in JSON format.
- Script must have the Linux execute permission set. 
- Many dynamic inventory scripts are written in Python, but this is not a requirement.

- Writing dynamic inventory scripts is not an exam requirement

```python
#!/usr/bin/python
    
from subprocess import Popen,PIPE
import sys
    
try:
     import json
except ImportError:
     import simplejson as json
    
result = {}
result['all'] = {}
    
pipe = Popen(['getent', 'hosts'], stdout=PIPE, universal_newlines=True)
    
result['all']['hosts'] = []
    
for line in pipe.stdout.readlines():
    s = line.split()
    result['all']['hosts']=result['all']['hosts']+s
    
result['all']['vars'] = {}
    
if len(sys.argv) == 2 and sys.argv[1] == '--list':
    print(json.dumps(result))
    
elif len(sys.argv) == 3 and sys.argv[1] == '--host':
    print(json.dumps({}))
    
else:
    print("Requires an argument, please use --list or --host <host>")
```

`pipe = Popen(\['getent', 'hosts'\], stdout=PIPE, universal_newline=True)`
- gets a list of hosts using the `getent` function. 
- This queries all hosts in /etc/hosts and other mechanisms where host name resolving is enabled. 
- To show the resulting host list, you can use the `\--list` command
- To show details for a specific host, you can use the option `\--host hostname`. 

```bash
    [ansible@control rhce8-book]$ ./listing101.py --list
    {"all": {"hosts": ["127.0.0.1", "localhost", "localhost.localdomain", "localhost4", "localhost4.localdomain4", "127.0.0.1", "localhost", "localhost.localdomain", "localhost6", "localhost6.localdomain6", "192.168.4.200", "control.example.com", "control", "192.168.4.201", "ansible1.example.com", "ansible1", "192.168.4.202", "ansible2.example.com", "ansible2"], "vars": {}}}
```

- Dynamic inventory scripts are activated in the same way as regular inventory scripts: you use the `-i` option to either the `ansible` or the `ansible-playbook` command to pass the name of the inventory script as an argument. 

External directory service can be based on a wide range of solutions:
- FreeIPA
- Active Directory
- Red Hat Satellite
- etc.

- Also are available for virtual machine-based infrastructures such as VMware of Red Hat Enterprise Virtualization, where virtual machines can be discovered dynamically.
- Can be found in cloud environments, where scripts are available for many solutions, including AWS, GCE, Azure, and OpenStack.

When you are working with dynamic inventory, additional parameters are normally required:
- To get an inventory from an EC2 cloud environment, you need to enter your web keys. 
- To pass these parameters, many inventory scripts come with an additional configuration file that is formatted in .ini style. 
- The community-provided ec2.py script, for instance, comes with an ec2.ini parameter file.

Another feature that is seen in many inventory scripts is cache management:
- Can use a cache to store names and parameters of recently discovered hosts. 
- If a cache is provided, options exist to manage the cache, allowing you, for instance, to make sure that the inventory information really is recently discovered.
