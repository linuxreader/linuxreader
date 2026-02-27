+++
title = 'Users and Groups'
description = 'Users and Groups'
+++
### Using Ansible Modules to Manage Users and Groups

- management of the user and group accounts and their direct properties. 
- management of sudo privilege escalation
- Setting up SSH connections and setting user passwords

#### Modules

user
- manage users and their base properties

group
- Manage groups and their properties

pamd
- Manage advanced authentication configuration through linux pluggable authentication modules (PAM)

known_hosts
- manage ssh known hosts

authorized_key
- copy authorized key to a managed host

lineinfile
- modify config file


#### Managing Users and Groups 

```yaml
    ---
    - name: creating a user and group
      hosts: ansible2
      tasks:
      - name: setup the group account
        group:
          name: students
          state: present
      - name: setup the user account
        user:
          name: anna
          create_home: yes
          groups: wheel,students
          append: yes
          generate_ssh_key: yes
          ssh_key_bits: 2048
          ssh_key_file: .ssh/id_rsa
```

**group** argument is 
- used to specify the primary group of the user.

**groups** argument is 
- used to make the user a member of additional groups. 

- While using the **groups** argument for existing users, make sure to include the **append** argument as well.
- Without **append**, all current secondary group assignments are overwritten.

Also notice that the user module has some options that cannot normally be managed with the Linux **useradd** command. The module can also be used to generate an SSH key and specify its properties.

#### Managing sudo 

No Ansible module specifically targets managing a sudo configuration

two options:

1. You can use the template module to create a sudo configuration file in the directory /etc/sudoers.d.
   - Using such a file is recommended because the file is managed independently, and as such, there is no risk it will be overwritten by an RPM update. 
2. The alternative is to use the lineinfile module to manage the /etc/sudoers main configuration file directly.

Users are created and added to a sudo file that is generated from a template:
```yml
    [ansible@control rhce8-book]$ cat vars/sudo
    sudo_groups:
      - name: developers
        groupid: 5000
        sudo: false
      - name: admins
        groupid: 5001
        sudo: true
      - name: dbas
        groupid: 5002
        sudo: false
      - name: sales
        groupid: 5003
        sudo: true
      - name: account
        groupid: 5004
        sudo: false
    [ansible@control rhce8-book]$ cat vars/users
    users:
      - username: linda
        groups: sales
      - username: lori
        groups: sales
      - username: lisa
        groups: account
      - username: lucy
        groups: account
```

- vars/users file defines users and the groups they should be a member of. 
- vars/sudo file defines new groups and, for each of these groups, sets a sudo parameter, which will be used in the template file:
```yml
{% for item in sudo_groups %}
{% if item.sudo %}
%{{ item.name}} ALL=(ALL:ALL) NOPASSWD:ALL
{% endif %}
{% endfor %}
```

- a **for** loop is used to walk through all items that have been defined in the **sudo_groups** variable in the vars/sudo file.
- for each of these groups an **if** statement is used to check the value of the Boolean variable sudo. If this variable is set to the Boolean value true, the group is added as a sudo group to the /etc/sudoers.d/sudogroups file.

**Listing 13-4** Managing sudo
```yml
    ---
    - name: configure sudo
      hosts: ansible2
      vars_files:
        - vars/sudo
        - vars/users
      tasks:
      - name: add groups
        group:
          name: "{{ item.name }}"
        loop: "{{ sudo_groups }}"
      - name: add users
        user:
          name: "{{ item.username }}"
          groups: "{{ item.groups }}"
        loop: "{{ users }}"
      - name: allow group members in sudo
        template:
          src: listing133.j2
          dest: /etc/sudoers.d/sudogroups
          validate: ‘visudo -cf %s’
          mode: 0440
```
