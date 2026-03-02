---
title: 'Using Loops and Items'
description: 'Using Loops and Items'
showDate: false
---

## Using Loops and Items

- Some modules enable you to provide a list that needs to be processed.
- Many modules don't, and in these cases, it makes sense to use a loop mechanism to iterate over a list of items. 
- Take, for instance, the yum module. While specifying the names of packages, you can use a list of packages. 
- If, however, you want to do something similar for the service module, you find out that this is not possible. 
- That is where loops come in. 

#### Working with Loops

Install software packages using the yum module and then ensures that services installed from these packages are started using the service module:

```yml
    ---
    - name: install and start services
      hosts: ansible1
      tasks:
      - name: install packages
        yum:
          name:
          - vsftpd
          - httpd
          - samba
          state: latest
      - name: start the services
        service:
          name: "{{ item }}"
          state: started
          enabled: yes
        loop:
        - vsftpd
        - httpd
        - smb
```

- A loop is defined at the same level as the service module. 
- The loop has a list of services in a list (array) statement 
- Items in the loop can be accessed by using the system internal variable **item**. 
- At no place in the playbook is there a definition of the variable **item**; the loop takes care of this.


- When considering whether to use a loop, you should first investigate whether a module offers support for providing lists as values to the keys that are used. 
- If this is the case, just provide a list, as all items in the list can be considered in one run of the module. 
- If not, define the list using **loop** and provide **"{{ item }}"** as the value to the key. 
- When using **loop**, the module is activated again on each iteration.

#### Using Loops on Variables

- Although it's possible to define a loop within a task, it's not the most elegant way. 
- To create a flexible environment where static code is separated from dynamic site-specific parameters, it's a much better idea to define loops outside the static code, in variables. 
- When you define loops within a variable, all the normal rules for working with variables apply: The variables can be defined in the play header, using an include file, or as host/hostgroup variables. 

Include the loop from a variable:

```
    ---
    - name: install and start services
      hosts: ansible1
      vars:
        services:
        - vsftpd
        - httpd
        - smb
      tasks:
      - name: install packages
        yum:
          name:
          - vsftpd
          - httpd
          - samba
          state: latest
      - name: start the services
        service:
          name: "{{ item }}"
          state: started
          enabled: yes
        loop: "{{ services }}"
```

#### Using Loops on Multivalued Variables

An item can be a simple list, but it can also be presented as a multivalued variable, as long as the multivalued variable is presented as a list. 

Use variables that are imported from the file vars/users-list:

```
users:
  - username: linda
    homedir: /home/linda
    shell: /bin/bash
    groups: wheel
  - username: lisa
    homedir: /home/lisa
    shell: /bin/bash
    groups: users
  - username: anna
    homedir: /home/anna
    shell: /bin/bash
    groups: users
```

Use the list in a playbook:
```
    ---
    - name: create users using a loop from a list
      hosts: ansible1
      vars_files: vars/users-list
      tasks:
      - name: create users
        user:
          name: "{{ item.username }}"
          state: present
          groups: "{{ item.groups }}"
          shell: "{{ item.shell }}"
        loop: "{{ users }}"
```

- Working with multivalued variables is possible, but the variables in that case must be presented as a list; using dictionaries is not supported. 
- The only way to loop over dictionaries is to use the **dict2items** filter. 
- Use of filters is not included in the RHCE topics and for that reason is not explained further here. 
- You can look up "Iterating over a dictionary" in the Ansible documentation for more information. 
#### Understanding with_items

- Since Ansible 2.5, using **loop** has been the command way to iterate over the values in a list. 
- In earlier versions of Ansible, the **with\_*keyword*** statement was used instead. 
- In this approach, the ***keyword*** is replaced with the name of an Ansible look-up plug-in, but the rest of the syntax is very common. 
- Will be deprecated in a future version of Ansible.

**With_keyword** Options Overview
with_items
- Used like loop to iterate over values in a list
with_file
- Used to iterate over a list of filenames on the control node
with_sequence
- Used to generate a list of values based on a numeric sequence

Loop over a list using with_keyword:
```yml
    ---
    - name: install and start services
      hosts: ansible1
      vars:
        services:
        - vsftpd
        - httpd
        - smb
      tasks:
      - name: install packages
        yum:
          name:
          - vsftpd
          - httpd
          - samba
          state: latest
      - name: start the services
        service:
          name: "{{ item }}"
          state: started
          enabled: yes
        with_items: "{{ services }}"
```
### Lab:  Working with loop

1\. Use your editor to define a variables file with the name vars/packages and the following contents:

``` pre1
packages:
- name: httpd
  state: absent
- name: vsftpd
  state: installed
- name: mysql-server
  state: latest
```

2\. Use your editor to define a playbook with the name exercise71.yaml and create the play header:

``` pre1
- name: manage packages using a loop from a list
  hosts: ansible1
  vars_files: vars/packages
  tasks:
```

3\. Continue the playbook by adding the yum task that will manage the packages, using the **packages** variable as defined in the vars/packages variable include file:

``` pre1
- name: manage packages using a loop from a list
  hosts: ansible1
  vars_files: vars/packages
  tasks:
  - name: install packages
    yum:
      name: "{{ item.name }}"
      state: "{{ item.state }}"
    loop: "{{ packages }}"
```

4\. Run the playbook using **ansible-playbook exercise71.yaml**, and observe the results. In the results you should see which packages it is trying to manage and in which state it is trying to get the packages.
