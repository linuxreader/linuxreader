+++
title = 'Including and importing Files'
description = 'Including and importing Files'
+++
When content is included, it is dynamically processed at the moment that Ansible reaches that content. 
- If content is imported, Ansible performs the import operation before starting to work on the tasks in the playbook.

Files can be included and imported at different levels:

• **Roles:** Roles are typically used to process a complete set of instructions provided by the role. Roles have a specific structure as well.

• **Playbooks:** Playbooks can be imported as a complete playbook. You cannot do this from within a play. Playbooks can be imported only at the
top level of the playbook.

• **Tasks:** A task file is just a list of tasks and can be imported or
included in another task.

• **Variables:** As discussed in [Chapter 6](#ch06.xhtml#ch06),
"[Working with Variables and Facts](#ch06.xhtml#ch06)," variables can be
maintained in external files and included in a playbook. This makes
managing generic multipurpose variables easier.

#### Importing Playbooks {.h4}

Importing playbooks is common in a setup where one master playbook is
used, from which different additional playbooks are included. According
to the Ansible Best Practices Guide (which is a part of the Ansible
documentation), the master playbook could have the name site.yaml, and
it can be used to include playbooks for each specific set of servers,
for instance. When a playbook is imported, this replaces the entire
play. So, you cannot import a playbook at a task level; it needs to
happen at a play level. [Listing 10-4](#ch10.xhtml#list10_4) gives an
example of the playbook imported in [Listing
10-5](#ch10.xhtml#list10_5). In [Listing 10-6](#ch10.xhtml#list10_6),
you can see the result of running the **ansible-playbook
listing105.yaml** command.

**Listing 10-4** Sample Playbook to Be Imported

::: pre_1
    - hosts: all
      tasks:
      - debug:
          msg: running the imported play
:::

**Listing 10-5** Importing a Playbook

::: pre_1
    ---
    - name: run a task
      hosts: all
      tasks:
      - debug:
          msg: running task1
    
    - name: importing a playbook
      import_playbook: listing104.yaml
:::

**Listing 10-6** Running **ansible-playbook listing105.yaml** Result

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook listing105.yaml
    
    PLAY [run a task] **************************************************************
    
    TASK [Gathering Facts] *********************************************************
    ok: [ansible2]
    ok: [ansible1]
    ok: [ansible3]
    ok: [ansible4]
    
    TASK [debug] *******************************************************************
    ok: [ansible1] => {
        "msg": "running task1"
    }
    ok: [ansible2] => {
        "msg": "running task1"
    }
    ok: [ansible3] => {
        "msg": "running task1"
    }
    ok: [ansible4] => {
        "msg": "running task1"
    }
    
    PLAY [all] *********************************************************************
    
    TASK [Gathering Facts] *********************************************************
    ok: [ansible2]
    ok: [ansible1]
    ok: [ansible3]
    ok: [ansible4]
    
    TASK [debug] *******************************************************************
    ok: [ansible1] => {
        "msg": "running the imported play"
    }
    ok: [ansible2] => {
        "msg": "running the imported play"
    }
    ok: [ansible3] => {
        "msg": "running the imported play"
    }
    ok: [ansible4] => {
        "msg": "running the imported play"
    }
    
    PLAY RECAP *********************************************************************
    ansible1                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible2                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible3                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible4                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
:::

#### Importing and Including Task Files {.h4}

**import_tasks**
- tasks are statically imported while executing the playbook. 

**include_tasks**
- tasks are dynamically included at the moment they are needed. 
- Dynamically including task files is recommended when the task file is used in a conditional statement. 
- If task files are mainly used to make development easier by working with separate task files, they can be statically imported.

There are a few considerations when working with **import_tasks** to
statically import tasks:

• Loops cannot be used with **import_tasks**.

• If a variable is used to specify the name of the file to import, this
cannot be a host or group inventory variable.

• When you use a **when** statement on the entire **import_tasks** file,
the conditional statements are applied to each task that is involved.

As an alternative, **include_tasks** can be used to dynamically include
a task file. This approach also comes with some considerations:

• When you use the **ansible-playbook \--list-tasks** command, tasks
that are in the included tasks are not displayed.

• You cannot use **ansible-playbook \--start-at-task** to start a
playbook on a task that comes from an included task file.

• You cannot use a **notify** statement in the main playbook to trigger
a handler that is in the included tasks file.

::: note

------------------------------------------------------------------------

**Tip**

When you use includes and imports to work with task files, the
recommendation is to store the task files in a separate directory. Doing
so makes it easier to delegate task management to specific users.

------------------------------------------------------------------------
:::

#### Using Variables When Importing and Including Files {.h4}

The main goal to work with imported and included files is to make
working with reusable code easy. To make sure you reach this goal, the
imported and included files should be as generic as possible. That means
it's a bad idea to include names of specific items that may change when
used in a different context. Think, for instance, of the names of
packages, users, services, and more.

To deal with include files in a flexible way, you should define specific
items as variables. Within the **include_tasks** file, for instance, you
refer to {{ package }}, and in the main playbook from which the include
files are called, you can define the variables. Obviously, you can use
this approach with a straight variable definition or by using host
variable or group variable include files.

::: note

------------------------------------------------------------------------

**Exam tip**

It's always possible to configure items in a way that is brilliant but
quite complex. On the exam it's not a smart idea to go for complex. Just
keep your solution as easy as possible. The only requirement on the exam
is to get things working, and it doesn't matter exactly how you do that.

------------------------------------------------------------------------
:::

In [Listings 10-7](#ch10.xhtml#list10_7) through
[10-10](#ch10.xhtml#list10_10), you can see how include and import files
are used to work on one project. The main playbook, shown in [Listing
10-9](#ch10.xhtml#list10_9), defines the variables to be used, as well
as the names of the include and import files. Listings 10-7 and 10-8
show the code from the include files, which use the variables that are
defined in [Listing 10-9](#ch10.xhtml#list10_9). The result of running
the playbook in [Listing 10-9](#ch10.xhtml#list10_9) can be seen in
[Listing 10-10](#ch10.xhtml#list10_10).

**Listing 10-7** The Include Tasks File tasks/service.yaml Used for
Services Definition

::: pre_1
    - name: install {{ package }}
      yum:
        name: "{{ package }}"
        state: latest
    - name: start {{ service }}
      service:
        name: "{{ service }}"
        enabled: true
        state: started
:::

The sample tasks file in [Listing 10-7](#ch10.xhtml#list10_7) is
straightforward; it uses the yum module to install a package and the
service module to start and enable the package. The variables this file
refers to are defined in the main playbook in [Listing
10-9](#ch10.xhtml#list10_9).

**Listing 10-8** The Import Tasks File tasks/firewall.yaml Used for
Firewall Definition

::: pre_1
    - name: install the firewall
      package:
        name: "{{ firewall_package }}"
        state: latest
    - name: start the firewall
      service:
        name: "{{ firewall_service }}"
        enabled: true
        state: started
    - name: open the port for the service
      firewalld:
        service: "{{ item }}"
        immediate: true
        permanent: true
        state: enabled
      loop: "{{ firewall_rules }}"
:::

In the sample firewall file in [Listing 10-8](#ch10.xhtml#list10_8), the
firewall service is installed, defined, and configured. In the
configuration of the firewalld service, a loop is used on the variable
firewall_rules. This variable obviously is defined in [Listing
10-9](#ch10.xhtml#list10_9), which is the file where site-specific
contents such as variables are defined.

**Listing 10-9** Main Playbook Example

::: pre_1
    ---
    - name: setup a service
      hosts: ansible2
      tasks:
      - name: include the services task file
        include_tasks: tasks/service.yaml
        vars:
          package: httpd
          service: httpd
        when: ansible_facts[’os_family’] == ’RedHat’
      - name: import the firewall file
        import_tasks: tasks/firewall.yaml
        vars:
          firewall_package: firewalld
          firewall_service: firewalld
          firewall_rules:
          - http
          - https
:::

The main playbook in [Listing 10-9](#ch10.xhtml#list10_9) shows the
site-specific configuration. It performs two main tasks: it defines
variables, and it calls an include file and an import file. The
variables that are defined are used by the include and import files. The
**include_tasks** statement is executed in a **when** statement. Notice
that the **firewall_rules** variable contains a list as its value, which
is used by the loop that is defined in the import file.

**Listing 10-10** Running **ansible-playbook listing109.yaml**

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook listing109.yaml
    
    PLAY [setup a service] *********************************************************
    
    TASK [Gathering Facts] *********************************************************
    ok: [ansible2]
    
    TASK [include the services task file] ******************************************
    included: /home/ansible/rhce8-book/tasks/service.yaml for ansible2
    
    TASK [install httpd] ***********************************************************
    ok: [ansible2]
    
    TASK [start httpd] *************************************************************
    changed: [ansible2]
    
    TASK [install the firewall] ****************************************************
    changed: [ansible2]
    
    TASK [start the firewall] ******************************************************
    ok: [ansible2]
    
    TASK [open the port for the service] *******************************************
    changed: [ansible2] => (item=http)
    changed: [ansible2] => (item=https)
    
    PLAY RECAP *********************************************************************
    ansible2                   : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
:::

The interesting thing in the [Listing 10-10](#ch10.xhtml#list10_10)
output is that the include file is dynamically included while running
the playbook. This is not the case for the statically imported file. In
[Exercise 10-3](#ch10.xhtml#exe10_3) you practice working with include
files.

::: box
**Exercise 10-3 Using Includes and Imports**

In this exercise you create a simple master playbook that installs a
service. The name of the service is defined in a variable file, and the
specific tasks are included through task files.

1\. Open the file exercise103-vars.yaml and define three variables as
follows:

``` pre1
packagename: vsftpd
servicename: vsftpd
firewalld_servicename: ftp
```

2\. Create the exercise103-ftp.yaml file and give it the following
contents to install, enable, and start the vsftpd service and also to
make it accessible in the firewall:

``` pre1
- name: install {{ packagename }}
  yum:
    name: "{{ packagename }}"
    state: latest
- name: enable and start {{ servicename }}
  service:
    name: "{{ servicename }}"
    state: started
    enabled: true
- name: open the service in the firewall
  firewalld:
    service: "{{ firewalld_servicename }}"
    permanent: yes
    state: enabled
```

3\. Create the exercise103-copy.yaml file that manages the
/var/ftp/pub/README file and make sure it has the following contents:

``` pre1
- name: copy a file
  copy:
    content: "welcome to this server"
    dest: /var/ftp/pub/README
```

4\. Create the master playbook exercise103.yaml that includes all of
them and give it the following contents:

``` pre1
---
- name: install vsftpd on ansible2
  vars_files: exercise103-vars.yaml
  hosts: ansible2
  tasks:
  - name: install and enable vsftpd
    import_tasks: exercise103-ftp.yaml
  - name: copy the README file
    import_tasks: exercise103-copy.yaml
```

5\. Run the playbook and verify its output

6\. Run an ad hoc command to verify the /var/ftp/pub/README file has
been created: **ansible ansible2 -a "cat /var/ftp/pub/README"**.


### End-of-Chapter Lab

In the end-of-chapter lab with this chapter, you reorganize a playbook
to work with several different files instead of one big file. Do this
according to the instructions in Lab 10-1.

### Lab 10-1

The lab82.yaml file, which you can find in the GitHub repository that
goes with this course, is an optimal candidate for optimization.
Optimize this playbook according to the following requirements:

• Use includes and import to make this a modular playbook where
different files are used to distinguish between the different tasks.

• Optimize this playbook such that it will run on no more than two hosts
at the same time and completes the entire playbook on these two hosts
before continuing with the next host.