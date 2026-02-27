+++
title = 'Ansible Roles'
description = 'Ansible Roles'
+++

Work with roles and Create roles

### Using Ansible Roles

- Ready-to-use playbook-based Ansible solutions that you can easily include in your own playbooks.
- Community roles are provided through Ansible Galaxy
- Also possible to create your own roles.
- Red Hat provides RHEL System Roles. 
- Roles make it possible to provide Ansible code in a reusable way. 
- You can easily define a specific task in a role, and after defining it in a role, you can easily redistribute that and ensure that tasks are handled the same way, no matter where they are executed. 
- Roles can be custom-made for specific environments, or default roles provided from Ansible Galaxy can be used.

#### Understanding Ansible Roles
- work with include files.
- All the different components that you may use in a playbook are used in roles and stored in separate directories. 
- While defining the role, you don't need to tell the role that it should look in some of these specific directories; it does that automatically. 
- The only thing you need to do is tell your Ansible playbook that it should include a role.
- Different components of the role are stored in different subdirectories. 

Roles Sample Directory Structure:
```
    [ansible@control roles]$ tree testrole/
    testrole/
    |-- defaults
    |   `-- main.yml
    |-- files
    |-- handlers
    |   `-- main.yml
    |-- meta
    |   `-- main.yml
    |-- README.md
    |-- tasks
    |   `-- main.yml
    |-- templates
    |-- tests
    |   |-- inventory
    |   `-- test.yml
    `-- vars
        `-- main.yml
```


Role Directory Structure
**defaults**
- Default variables that may be overwritten by other variables
**files**
- Static files that are needed by role tasks
**handlers**
- Handlers for use in this role
**meta**
- metadata, such as dependencies, plus license and maintainer information
**tasks**
- Role task definitions
**templates**
- Jinja2 templates
**tests**
- Optional inventory and a test.yml file to test the role
**vars**
- Variables that are not meant to be overwritten

- Most of the role directories have a main.yml file. 
- This is the entry-point YAML file that is used to define components in the role.

#### Understanding Role Location

Roles can be stored in different locations:

**./roles**
- store roles in the current project directory. 
- highest precedence.

**~/.ansible/roles** 
- exists in the current user home directory and makes the role available to the current user only. 
- second-highest precedence.

**/etc/ansible/roles**
- Where roles are stored to make them accessible to any user.

**/usr/share/ansible/roles** 
- Where roles are stored after they are installed from RPM files.
- lowest precedence
- should not be used for storing custom-made roles.

`ansible-galaxy init { newrolename }`
- create a custom role
- creates the default role directory structure with a main.yml file 
- includes sample files 

#### Using Roles from Playbooks

- Call roles in a playbook the same way you call a task
- Roles are included as a list.

```yml
    ---
    - name: include some roles
      roles:
      - role1
      - role2
``` 

- Roles are executed before the tasks.
- In specific cases you might have to execute tasks before the roles. To do so, you can specify these tasks in a **pre_tasks** section. 
- Also, it's possible to use the **post_tasks** section to include tasks that will be executed after the roles, but also after tasks specified in the playbook as well as the handlers they call.

#### Creating Custom Roles

- Use `mkdir roles` to create a roles subdirectory in the current directory, and use `cd roles` to get into that subdirectory.
- Use `ansible-galaxy init motd` to create the motd role structure.
- Add contents to **motd/tasks/main.yml** 
- Add contents to motd/templates/motd.j2 
- Add contents to **motd/defaults/main.yml** 
- Add contents to **motd/meta/main.yml**
- Create the playbook **exercise91.yaml** to run the role
- Run the playbook by using `ansible-playbook exercise91.yaml`
- Verify that modifications have been applied correctly by using the ad hoc command `ansible ansible2 -a "cat /etc/motd"`

Sample role all under **roles/motd/**:

**defaults/main.yml**
```yml
---
# defaults file for motd
system_manager: anna@example.com
```

**meta/main.yml**
```yml
galaxy_info:
author: Sander van V
description: your description
company: your company (optional)
license: license (GPLv2, CC-BY, etc)
min_ansible_version: 2.5
```

**tasks/main.yml**
```yml
---
tasks file for motd
- name: copy motd file
  template:
    src: templates/motd.j2
    dest: /etc/motd
    owner: root
    group: root
    mode: 0444
```

**templates/motd.j2**

```
Welcome to {{ ansible_hostname }}
    
This file was created on {{ ansible_date_time.date }}
Disconnect if you have no business being here
    
Contact {{ system_manager }} if anything is wrong
```

Playbook **motd.yml**:
```yml
---
- name: use the motd role playbook
  hosts: ansible2
  roles:
  - role: motd
    system_manager: bob@example.com
```

handlers/main.yml example:
```yml
---
# handlers file for base-config
  - name: source profile
    command: source /etc/profile

  - name: source bash
    command: source /etc/bash.bashrc                               
```
#### Managing Role Dependencies

- Roles may use other roles as a dependency.
- You can put role dependencies in meta/main.yml
- Dependent roles are always executed before the roles that depend on them. 
- Dependent roles are executed once. 
- When two roles that are used in a playbook call the same dependency, the dependent role is executed once only. 
- When calling dependent roles, it is possible to pass variables to the dependent role.
- You can define a **when** statement to ensure that the dependent role is executed only in specific situations.

Defining dependencies in **meta/main.yml**

```yml
    dependencies:
    - role: apache
      port: 8080
    - role: mariabd
      when: environment == ’production’
```
#### Understanding File Organization Best Practices

- Working with roles splits the contents of the role off the tasks that are run through the playbook. 
- Splitting files to store them in a location that makes sense is common in Ansible 


- When you're working with Ansible, it's a good idea to work with project directories in bigger environments. 
- Working with project directories makes it easier to delegate tasks and have the right people responsible for the right things.

- Each project directory may have its own ansible.cfg file, inventory file, and playbooks.

- If the project grows bigger, variable files and other include files may be used, and they are normally stored in subdirectories.

- At the top-level directory, create the main playbook from which other playbooks are included. The suggested name for the main playbook is **site.yml**.

- Use **group_vars/** and **host_vars/** to set host-related variables and do not define them in inventory.

- Consider using different inventory files to differentiate between production and staging phases.

- Use roles to standardize common tasks.


When you are working with roles, some additional recommendations apply:

- Use a version control repository to maintain roles in a consistent way. Git is commonly used for this purpose.

- Sensitive information should never be included in roles. Use Ansible Vault to store sensitive information in an encrypted way.

- Use `ansible-galaxy init` to create the role base structure. Remove files and directories you don't use.

- Don't forget to provide additional information in the role's **README.md** and **meta/main.yml** files.

- Keep roles focused on a specific function. It is better to use multiple roles to perform multiple tasks.

- Try to develop roles in a generic way, such that they can be used for multiple purposes.



### Lab 9-1

Create a playbook that starts the Nginx web server on ansible1, according to the following requirements:
• A requirements file must be used to install the Nginx web server. Do NOT use the latest version of the Galaxy role, but instead use the version before that.
• The same requirements file must also be used to install the latest version of postgresql.
• The playbook needs to ensure that neither httpd nor mysql is currently installed.

### Lab 9-2

Use the RHEL SELinux System Role to manage SELinux properties according
to the following requirements:

• A Boolean is set to allow SELinux relabeling to be automated using
cron.
• The directory /var/ftp/uploads is created, permissions are set to 777,
and the context label is set to public_content_rw_t.
• SELinux should allow web servers to use port 82 instead of port 80.
• SELinux is in enforcing state.
Subjects:
`ansible-playbook timesync.yaml` to run the playbook. Observe its output. Notice that some messages in red are shown, but these can safely be ignored.

5\. Use `ansible ansible2 -a "timedatectl show"` and notice that the **timezone** variable is set to UTC.

### Lab 9-1

Create a playbook that starts the Nginx web server on ansible1, according to the following requirements:
• A requirements file must be used to install the Nginx web server. Do NOT use the latest version of the Galaxy role, but instead use the version before that.
• The same requirements file must also be used to install the latest version of postgresql.
`ansible-galaxy install -r roles/requirements.yml`

`cat roles/requirements.yml`

```yml
- src: geerlingguy.nginx
  version: "3.1.4"

- src: geerlingguy.postgresql
```

• The playbook needs to ensure that neither httpd nor mysql is currently installed.
```yml
---
- name: ensure conflicting packages are not installed
  hosts: web1
  tasks:
  - name: remove packages
    yum:
      name: 
      - mysql
      - httpd
      state: absent

- name: nginx web server
  hosts: web1
  roles: 
  - geerlingguy.nginx
  - geerlingguy.postgresql
```
(Had to add a variable file for redhat 10 into the role. )
### Lab 9-2

Use the RHEL SELinux System Role to manage SELinux properties according to the following requirements:

• A Boolean is set to allow SELinux relabeling to be automated using
cron.
• The directory /var/ftp/uploads is created, permissions are set to 777,
and the context label is set to public_content_rw_t.
• SELinux should allow web servers to use port 82 instead of port 80.
• SELinux is in enforcing state.

`vim lab92.yml`

```yml
---
- name: manage ftp selinux properties
  hosts: ftp1
  vars:
    selinux_booleans: 
      - name: cron_can_relabel
        state: true
        persistent: true
    selinux_state: enforcing
    selinux_ports:
    - ports: 82
      proto: tcp
      setype: http_port_t
      state: present
      local: true
 
  tasks:

  - name: create /var/ftp/uploads/
    file:
      path: /var/ftp/uploads
      state: directory
      mode: 777
  
  - name: set selinux context
    sefcontext:
      target: '/var/ftp/uploads(/.*)?'
      setype: public_content_rw_t
      ftype: d
      state: present
    notify: run restorecon

  - name: Execute the role and reboot in a rescue block
    block:
      - name: Include selinux role
        include_role:
          name: rhel-system-roles.selinux
    rescue:
      - name: >-
          Fail if failed for a different reason than selinux_reboot_required
        fail:
          msg: "role failed"
        when: not selinux_reboot_required

      - name: Restart managed host
        reboot:

      - name: Wait for managed host to come back
        wait_for_connection:
          delay: 10
          timeout: 300

      - name: Reapply the role
        include_role:
          name: rhel-system-roles.selinux
  
  handlers:
    - name:  run restorecon
      command: restorecon -v /var/ftp/uploads
```

