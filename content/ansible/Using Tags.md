+++
title = 'Using Tags'
description = 'Using Tags'
+++
### Using Tags

When you are using larger playbooks, Ansible enables you to use the
**tags** attribute. A tag is a label that is applied to a task or
another item like a block or a play, and while using the
**ansible-playbook \--tags** or **ansible-playbook \--skip-tags**
command, you can specify which tags need to be executed. [Listing
11-15](#ch11.xhtml#list11_15) shows a simple playbook example where tags
are used, and in [Listing 11-16](#ch11.xhtml#list11_16) you can see the
output generated while running this playbook.

**Listing 11-15** Using **tags** in a Playbook

::: pre_1
    ---
    - name: using tags example
      hosts: all
      vars:
        service:
        - vsftpd
        - httpd
      tasks:
      - yum:
          name:
          - httpd
          - vsftpd
          state: present
        tags:
        - install
      - service:
          name: "{{ item }}"
          state: started
          enabled: yes
        loop: "{{ services }}"
        tags:
        - services
:::

**Listing 11-16 ansible-playbook \--tags "install" listing1115.yaml**
Output

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook --tags "install" listing1115.yaml
    
    PLAY [using tags example] ******************************************************
    
    TASK [Gathering Facts] *********************************************************
    ok: [ansible2]
    ok: [ansible1]
    ok: [ansible4]
    ok: [ansible3]
    
    TASK [yum] *********************************************************************
    ok: [ansible2]
    ok: [ansible1]
    changed: [ansible3]
    changed: [ansible4]
    
    PLAY RECAP *********************************************************************
    ansible1                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible2                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible3                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible4                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
:::

Tags can be applied to many structures, such as imported plays, tasks,
and roles, but the easiest way to get familiar with tags is to use them
on a task. Note that tags cannot be applied on items that are
dynamically included (instead of imported), using **include_roles** or
**include_tasks**.

While writing playbooks, you may apply the same tag multiple times. This
capability allows you to define groups of tasks, where multiple tasks
are configured with the same tag, and as a result, you can easily run a
specific part of the requested configuration. When multiple tasks with
multiple tags are used, you can get an overview of each using the
**ansible-playbook \--list-tasks \--list-tags** command. In [Listing
11-17](#ch11.xhtml#list11_17) you can see an example that is based on
the playbook listing1114.yaml.

**Listing 11-17** Listing Tasks and Tags

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook --list-tags --list-tasks listing1115.yaml
    
    playbook: listing1115.yaml
    
      play #1 (all): using tags example.    TAGS: []
        tasks:
          yum.       TAGS: [install]
          service.   TAGS: [services]
          TASK TAGS: [install, services]
:::

When working with tags, you can use some special tags. [Table
11-5](#ch11.xhtml#tab11_5) gives an overview.

**Table 11-5** Special Tags Overview

![](/images/11tab05.jpg)

Apart from these special tags, you might also want to set a **debug**
tag to easily identify tasks that should be run only if you specifically
want to run debug tasks as well. If combined with the **never** tag, the
task that is tagged with the **debug,never** tasks runs only if the
**debug** tag is specifically requested. So in case you want to run the
entire playbook, including tasks that have been tagged with **debug**,
you need to use the **ansible-playbook \--tags all,debug** command. In
[Exercise 11-3](#ch11.xhtml#exe11_3) you can see how this can be used to
optimize the playbook that was previously used in [Exercise
11-2](#ch11.xhtml#exe11_2).

::: box
**Exercise 11-3 Using Tags to Make Debugging Easier**

1\. Rewrite the exercise112.yaml playbook that you created in the
previous exercise, and include the line **tags: \[never, debug \]** in
the **debug** task. The complete playbook looks as follows:

``` pre1
---
- name: using assert to check if volume group vgdata exists
  hosts: all
  tasks:
  - name: check if vgdata exists
    command: vgs vgdata
    register: vg_result
    ignore_errors: true
  - name: show vg_result variable
    debug:
      var: vg_result
    tags: [ never, debug ]
  - name: print a message
    assert:
      that:
      - vg_result.rc == 0
      fail_msg: volume group not found
      success_msg: volume group was found
```

2\. Run the playbook using **ansible-playbook \--tags all
exercise113.yaml**. Notice that it does not run the **debug** task.

3\. Run the playbook using **ansible-playbook \--tags all,debug
exercise113.yaml**. Notice that it now does run the **debug** task as
well.
:::
