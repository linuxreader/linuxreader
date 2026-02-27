+++
title = 'Variables'
description = 'All about Ansible variables'
+++

Using and working with variables
- Capture command output using register

### Variables

Three types of variables:
- Fact
- Variable
- Magic Variable

**Variables** make Ansible really flexible. Especially when used in combination with conditionals. These are defined at the discretion of the user.:
```yaml
---
- name: create a user using a variable
  hosts: ansible1
  vars:
    users: lisa <-- defaults value for this play
  tasks:
    - name: create a user {{ users }} on host {{ ansible_hostname }} <-- ansible fact variable
      user:
        name: "{{ users }}" <-- If value starts with variable, the whole line must have double quotes
``` 


### Working with Variables


- Variables can be used to refer to a wide range of dynamic data, such as names of files, services, packages, users, URLs to specific servers, etc.

#### Defining Variables

To define a variable
- key: value structure in a vars section in the play header. 

```
    ---
    - name: using variables
      hosts: ansible1
      vars: <-------------
        ftp_package: vsftpd <------------
      tasks:
      - name: install package
        yum:
          name: "{{ ftp_package }}" <------------
          state: latest
```

- As the variable is the first item in the value, its name must be placed between double curly brackets as well as double quotes.

Variable equirements:

• Must start with a letter.
• Case sensitive.
• Can contain only letters, numbers, and underscores.

#### Using Include Files

- It is common to define variables in include files. Specific host and host group variables can be used as include files 
- it's also possible to include an arbitrary file as a variable file, using the **vars_files:** statement. 
- The **vars_files:** parameter can have a single value or a list providing multiple values. If a list is used, each item needs to start with a dash
- When you include variables from files, it's a good idea to work with a separate directory that contains all variables because that makes it easier to manage as your projects grow bigger.

```
    ---
    - name: using a variable include file
      hosts: ansible1
      vars_files: vars/common <--------------
      tasks:
      - name: install package
        yum:
          name: "{{ my_package }}" <------------
          state: latest
```

**vars/common**
```
    my_package: nmap
    my_ftp_service: vsftpd
    my_file_service: smb
```

-  If variables are defined in individual playbooks, they are spread all over, and it may be difficult to get an overview of all variables that are used on a site.

#### Managing Host and Group Variables

**host_vars** and **group_vars**  
- set variables for specific hosts or specific host groups. 
- In older versions of Ansible, it was common to set host variables and group variables in inventory, but this practice is now deprecated.

**host_vars**
- Must create a subdirectory with the name **host_vars** within the Ansible project directory. 
- In this directory, create a file that matches the inventory name of the host to which the variables should be applied. 
- So the variables for host ansible1 are defined in **host_vars/ansible1**. 

**group_vars**  
- Must create a directory with the name **group_vars**. 
- In this directory, a file with the name of the host group is created, and in this file all variables are defined. 
- ie: **group_vars/webservers**

If no variables are defined at the command prompt, it will use the variable set for the play. You can also define the variables with the `-e` flag when running the playbook:
```bash
[ansible@control base]$ ansible-playbook variable-pb.yaml -e users=john

PLAY [create a user using a variable] ************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [ansible1]

TASK [create a user john on host ansible1] *******************************************************************************************************************
changed: [ansible1]

PLAY RECAP ***************************************************************************************************************************************************
ansible1                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

### LAB: Using Host and Host Group Variables

1\. Create a project directory in your home directory. Type **mkdir ~/chapter6** to create the chapter6 project directory, and use **cd \~/chapter6** to go into this directory.

2\. Type **cp ../ansible.cfg .** to copy the ansible.cfg file that you used before. No further modifications to this file are required.

3\. Type **vim inventory** to create a file with the name inventory, and ensure it has the following contents:

``` pre1
[webservers]
ansible1

[dbservers]
ansible2
```

4\. Create the file webservers.yaml, containing the following contents. Notice that nothing is really changed by running this playbook. It just uses the debug module to show the current value of the variables.

``` pre1
---
- name: configure web services
  hosts: webservers
  tasks:
  - name: this is the {{ web_package }} package
    debug:
      msg: "Installing {{ web_package }}"
  - name: this is the {{ web_service }} service
    debug:
      msg: "Starting the {{ web_service }}"
```

5\. Create the file group_vars/webservers with the following contents:

``` pre1
web_package: httpd
web_service: httpd
```

6\. Run the playbook with some verbosity to verify it is working by using `ansible-playbook -vv webservers.yaml`

#### Using Multivalued Variables

Two types of multivalued variables:

**array** (list)
- key that can have multiple items as its value.
- Each item in a list starts with a dash (-). 
- Individual items in a list can be addressed using the index number (starting at zero), as in {{ users\[1\] }} (which would print the key-value pairs that are set for user lisa)

```
    users:
      - linda:
        username: linda
        homedir: /home/linda
        shell: /bin/bash
      - lisa:
        username: lisa
        homedir: /home/lisa
        shell: /bin/bash
      - anna:
        username: anna
        homedir: /home/anna
        shell: /bin/bash
```


**dictionary** (hash)

- Unordered collection of items, a collection of key-value pairs. 
- In Python, a dictionary is defined as my_dict = { key1: 'car', key2:'bike' }. 
- Because it is based on Python, Ansible lets users use dictionaries as an alternative notation to arrays
- not as common in use as arrays. 
- Items in values in a dictionary are not started with a dash. 
```
    users:
      linda:
        username: linda
        homedir: /home/linda
        shell: /bin/bash
      lisa:
        username: lisa
        homedir: /home/lisa
        shell: /bin/bash
      anna:
        username: anna
        homedir: /home/anna
        shell: /bin/bash
```

Addressing Specific Keys in a Dictionary Multivalued Variable:
```
    ---
    - name: show dictionary also known as hash
      hosts: ansible1
      vars_files:
      - vars/users-dictionary
      tasks:
      - name: print dictionary values
        debug:
          msg: "User {{ users.linda.username }} has homedirectory {{ users.linda.homedir }} and shell {{ users.linda.shell }}"
```

Using the Square Brackets Notation to Address Multivalued Variables (recommended method)

```
    ---
    - name: show dictionary also known as hash
      hosts: ansible1
      vars_files:
        - vars/users-dictionary
      tasks:
        - name: print dictionary values
          debug:
            msg: "User {{ users[’linda’][’username’] }} has homedirectory {{ users[’linda’][’homedir’] }} and shell {{ users[’linda’][’shell’]  }}"
```

**Magic Variables**

- Variables that are set automatically by Ansible to reflect an Ansible internal state. 
- There are about 30 magic variables
- Common Magic Variables

![Image](/Images/06tab05.jpg)

- you cannot use their name for anything else. 
- If you try to set a magic variable to another value anyway, it always resets to the default internal value. 

Debug module can be used to show the current values assigned to the hostvars magic variable.
- Shows many settings that you can change by modifying the ansible.cfg configuration file. 
- If local facts are defined on the host, you will see them also.

```
    [ansible@control ~]$ ansible localhost -m debug -a 'var=hostvars["ansible1"]'
    localhost | SUCCESS => {
        "hostvars[\"ansible1\"]": {
            "ansible_check_mode": false,
            "ansible_diff_mode": false,
            "ansible_facts": {},
            "ansible_forks": 5,
            "ansible_inventory_sources": [
                "/home/ansible/inventory"
            ],
            "ansible_playbook_python": "/usr/bin/python3.6",
            "ansible_verbosity": 0,
            "ansible_version": {
                "full": "2.9.5",
                "major": 2,
                "minor": 9,
                "revision": 5,
                "string": "2.9.5"
            },
            "group_names": [
                "ungrouped"
            ],
            "groups": {
                "all": [
                    "ansible1",
                    "ansible2"
                ],
                "ungrouped": [
                    "ansible1",
                    "ansible2"
                ]
            },
            "inventory_dir": "/home/ansible",
            "inventory_file": "/home/ansible/inventory",
            "inventory_hostname": "ansible1",
            "inventory_hostname_short": "ansible1",
            "omit": "__omit_place_holder__38849508966537e44da5c665d4a784c3bc0060de",
            "playbook_dir": "/home/ansible"
        }
    }
```

#### Variable Precedence


- Avoid using variables with the same names that are defined at different levels. 
- If a variable with the same name is defined at different levels, the most specific variable always wins. 
- Variables that are defined while running the **playbook** command using the **-e key=value** command-line argument have the highest precedence. 
- After variables that are passed as command-line options, playbook variables are considered. 
- Next are variables that are defined for inventory hosts or host groups. 
- Consult the Ansible documentation item "Variable precedence" for more details and an overview of the 22 different levels where variables can be set and how precedence works for them.

1\. Variables passed on the command line
2\. Variables defined in or included from a playbook
3\. Inventory variables


### Capturing Command Output Using register

The result of commands can also be used as a variable byusing the **register** parameter in a task. 

```
    ---
    - name: test register
      hosts: ansible1
      tasks:
      - shell: cat /etc/passwd
        register: passwd_contents
      - debug:
          var: "passwd_contents"
```

The `cat /etc/passwd` command is executed by the shell module. Notice that in this playbook no names are used for tasks. Using names for tasks is
not mandatory; it's just recommended in more complex playbooks because this convention makes identification of the tasks easier. The entire contents of the command are next stored in the variable **passwd_contents**.

This variable contains the output of the command, stored in different keys. [Table 6-7](#ch06.xhtml#tab6_7) provides an overview of the most
useful keys, and [Listing 6-19](#ch06.xhtml#list6_19) shows the partial result of the `ansible-playbook listing618.yaml` command.

Keys Used with **register**
cmd
- Command that was used
rc
- Return code of the command
stderr
- Error messages
stderr_lines
- Errors line by line
stdout
- command output
stdout_line
- Command output line by line

```
    [ansible@control ~]$ ansible-playbook listing618.yaml
    
    PLAY [test register] *******************************************************************
    
    TASK [Gathering Facts] *****************************************************************
    ok: [ansible2]
    ok: [ansible1]
    
    TASK [shell] ***************************************************************************
    changed: [ansible2]
    changed: [ansible1]
    
    TASK [debug] ***************************************************************************
    ok: [ansible1] => {
        "passwd_contents": {
            "changed": true,
            "cmd": "cat /etc/passwd",
            "delta": "0:00:00.004149",
            "end": "2020-04-02 02:28:10.692306",
            "failed": false,
            "rc": 0,
            "start": "2020-04-02 02:28:10.688157",
            "stderr": "",
            "stderr_lines": [],
            "stdout": "root:x:0:0:root:/root:/bin/bash\nbin:x:1:1:bin:/bin:/sbin/nologin\ndaemon:x:2:2:daemon:/sbin:/sbin/nologin\nadm:x:3:4:adm:/var/adm:/sbin/nologin\nlp:x:4:7:lp:/var/spool/lpd:/sbin/nologin\nsync:x:5:0:sync:/sbin:/bin/sync\nshutdown:x:6:0:shutdown:/sbin:/sbin/shutdown\nhalt:x:7:0:halt:/sbin:/sbin/halt\nansible:x:1000:1000:ansible:/home/ansible:/bin/bash\napache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin\nlinda:x:1002:1002::/home/linda:/bin/bash\nlisa:x:1003:1003::/home/lisa:/bin/bash",
            "stdout_lines": [
                "root:x:0:0:root:/root:/bin/bash",
                "bin:x:1:1:bin:/bin:/sbin/nologin",
                "daemon:x:2:2:daemon:/sbin:/sbin/nologin",
                "adm:x:3:4:adm:/var/adm:/sbin/nologin",
                "lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin",
                "sync:x:5:0:sync:/sbin:/bin/sync",
                "shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown",
                "halt:x:7:0:halt:/sbin:/sbin/halt",
                "ansible:x:1000:1000:ansible:/home/ansible:/bin/bash",
                "apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin",
                "linda:x:1002:1002::/home/linda:/bin/bash",
                "lisa:x:1003:1003::/home/lisa:/bin/bash"
            ]
        }
    }
```

Ensure that a task runs only if a command produces a specific result by using **register** with conditionals.

**register** shows the values that are returned by specific tasks. Tasks have common return values, but modules may have specific
return values. That means you cannot assume, based on the result of an example using a specific module, that the return values you see are
available for all modules. Consult the module documentation for more information about specific return values.


