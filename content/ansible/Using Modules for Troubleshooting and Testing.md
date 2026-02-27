+++
title = 'Using Modules for Troubleshooting and Testing'
description = 'Using Modules for Troubleshooting and Testing'
+++
### Using Modules for Troubleshooting and Testing 

While working with playbooks, you may use different modules for
troubleshooting. The debug module was used in previous chapters and is
particularly useful for analyzing variable behavior. Some other modules
may prove useful when troubleshooting Ansible. [Table
11-4](#ch11.xhtml#tab11_4) gives an overview.

::: group
**Table 11-4** Troubleshooting Modules Overview

![Image](Images/11tab04.jpg) {width="940" height="295"}
:::

The following sections discuss how these modules can be used.

#### Using the Debug Module {.h4}

The debug module is useful to visualize what is happening at a certain
point in a playbook. It works with two arguments: the **msg** argument
can be used to print a message, and the **var** argument can be used to
print the value of a variable. Notice that when you use the **var**
argument, the variable does not have to be referred to using the usual
**{{ varname }}** structure, just use **varname** instead. If variables
are used in the **msg** argument, they must be referred to the normal
way, using the **{{ varname }}** syntax.

Because you have already seen the debug module in action in numerous
examples in Chapters 6, 7, and 8 of this book, no new examples are
included here.

#### Using the uri Module {.h4}

The best way to learn how to work with these modules is to look at some
examples. [Listing 11-7](#ch11.xhtml#list11_7) shows an example where
the uri module is used.

**Listing 11-7** Using the uri Module

::: pre_1
    ---
    - name: test webserver access
      hosts: localhost
      become: no
      tasks:
      - name: connect to the web server
        uri:
          url: http://ansible2.example.com
          return_content: yes
        register: this
        failed_when: "’welcome’ not in this.content"
      - debug:
          var: this.content
:::

The playbook in [Listing 11-7](#ch11.xhtml#list11_7) uses the uri module
to connect to a web server. The **return_content** argument captures the
web server content, which is stored in a variable using **register**.
Next, the **failed_when** statement makes this module fail if the text
"welcome" is not in the registered variable. For debugging purposes, the
debug module is used to show the contents of the variable.

In [Listing 11-8](#ch11.xhtml#list11_8) you can see the partial result
of running this playbook. Notice that the playbook does not generate a
failure because the default web page that is shown by the Apache web
server contains the text "welcome."

**Listing 11-8 ansible-playbook listing117.yaml** Command Result

```bash
[ansible@control rhce8-book]$ ansible-playbook listing117.yaml
    
    PLAY [test webserver access] ***************************************************
    
    TASK [Gathering Facts] *********************************************************
    ok: [localhost]
    
    TASK [connect to the web server] ***********************************************
    ok: [localhost]
    
    TASK [debug] *******************************************************************
    ok: [localhost] => {
        "this.content": "
    
    PLAY RECAP *********************************************************************
    localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    
```

Using the uri module can be useful to perform a simple test to check
whether a web server is available, but you can also use it to check
accessibility or returned information from an API endpoint.

#### Using the stat Module {.h4}

You can use the stat module to check on the status of files. Although
this module can be useful for checking on the status of just a few
files, it's not a file system integrity checker that was developed to
check file status on a large scale. If you need large-scale file system
integrity checking, you should use Linux utilities such as aide.

The stat module is useful in combination with **register**. In this use,
the stat module is used to register the status of a specific file, and
in a **when** statement, a check can be done to see whether the file
status is not as expected. In combination with the fail module, you can
use this module to generate a failure and error message if the file does
not meet the expected status. [Listing 11-9](#ch11.xhtml#list11_9) shows
an example, and [Listing 11-10](#ch11.xhtml#list11_10) shows the
resulting output, where you can see that the fail module fails the
playbook because the file owner is not root.

**Listing 11-9** Using stat to Check Expected File Status

::: pre_1
    ---
    - name: create a file
      hosts: all
      tasks:
      - file:
          path: /tmp/statfile
          state: touch
          owner: ansible
    
    - name: check file status
      hosts: all
      tasks:
      - stat:
          path: /tmp/statfile
        register: stat_out
      - fail:
          msg: "/tmp/statfile file owner not as expected"
        when: stat_out.stat.pw_name != ’root’
:::

**Listing 11-10 ansible-playbook listing119.yaml** Command Result

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook listing119.yaml
    
    PLAY [create a file] ***********************************************************
    
    TASK [Gathering Facts] *********************************************************
    ok: [ansible2]
    ok: [ansible1]
    ok: [ansible3]
    ok: [ansible4]
    fatal: [ansible6]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ansible@ansible6: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).", "unreachable": true}
    fatal: [ansible5]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: connect to host ansible5 port 22: No route to host", "unreachable": true}
    
    TASK [file] ********************************************************************
    changed: [ansible2]
    changed: [ansible1]
    changed: [ansible3]
    changed: [ansible4]
    
    PLAY [check file status] *******************************************************
    
    TASK [Gathering Facts] *********************************************************
    ok: [ansible1]
    ok: [ansible2]
    ok: [ansible3]
    ok: [ansible4]
    
    TASK [stat] ********************************************************************
    ok: [ansible2]
    ok: [ansible1]
    ok: [ansible3]
    ok: [ansible4]
    
    TASK [fail] ********************************************************************
    fatal: [ansible2]: FAILED! => {"changed": false, "msg": "/tmp/statfile file owner not as expected"}
    fatal: [ansible1]: FAILED! => {"changed": false, "msg": "/tmp/statfile file owner not as expected"}
    fatal: [ansible3]: FAILED! => {"changed": false, "msg": "/tmp/statfile file owner not as expected"}
    fatal: [ansible4]: FAILED! => {"changed": false, "msg": "/tmp/statfile file owner not as expected"}
    
    PLAY RECAP *********************************************************************
    ansible1                   : ok=4    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
    ansible2                   : ok=4    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
    ansible3                   : ok=4    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
    ansible4                   : ok=4    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
    ansible5                   : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
    ansible6                   : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
:::

#### Using the assert Module {.h4}

The assert module is a bit like the fail module. You can use it to
perform a specific conditional action. The assert module works with a
**that** option that defines a list of conditionals. If any one of these
conditionals is false, the task fails, and if all the conditionals are
true, the task is successful. Based on the success or failure of a task,
the module uses the **success_msg** or **fail_msg** options to print a
message. [Listing 11-11](#ch11.xhtml#list11_11) shows an example that
uses the assert module.

**Listing 11-11** Using the assert Module

::: pre_1
    ---
    - hosts: localhost
      vars_prompt:
      - name: filesize
        prompt: "specify a file size in megabytes"
      tasks:
      - name: check if file size is valid
        assert:
          that:
          - "{{ (filesize | int) <= 100 }}"
          - "{{ (filesize | int) >= 1 }}"
          fail_msg: "file size must be between 0 and 100"
          success_msg: "file size is good, let\’s continue"
      - name: create a file
        command: dd if=/dev/zero of=/bigfile bs=1 count={{ filesize }}
:::

The example in [Listing 11-11](#ch11.xhtml#list11_11) contains a few new
items. As you can see, the play header starts with a **vars_prompt**.
This defines a variable named **filesize**, which is based on the input
provided by the user. This **filesize** variable is next used by the
assert module. The **that** statement contains a list in which two
conditions are stated. If specified like this, all conditions stated in
the **that** condition must be true. So you are looking for **filesize**
to be equal to or bigger than 1, and smaller than or equal to 100.

Before this can be done, one little problem needs to be managed: when
**vars_prompt** is used, the variable type is set to be a string by
default. This means that a statement like 

```bash
**filesize left caret= 100**
```
would
fail with a type mismatch. That is why a Jinja2 filter is used to
convert the variable type from string to integer.

Filters are a powerful feature provided by the Jinja2 templating
language and can be used in Ansible to modify variables before
processing. For more information about filters, see
<https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html>.
The int filter can be used to convert the value of a string variable to
an integer. To do this, you need to rewrite the entire variable as a
Jinja2 operation, which is done using **"{{ (filesize | int) left caret= 100
}}"**.

In this line, the entire string is written as a variable. The variable
is further treated in a Jinja2 context. In this context, the part
**(filesize \| int)** ensures that the string is converted to an
integer, which makes it possible to check if the value is smaller than
100.

When you run the code in [Listing 11-11](#ch11.xhtml#list11_11), the
result shown in [Listing 11-12](#ch11.xhtml#list11_12) is produced.

**Listing 11-12 ansible-playbook listing1111.yaml** Output

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook listing1111.yaml
    
    PLAY [localhost] *****************************************************************
    
    TASK [Gathering Facts] ***********************************************************
    ok: [localhost]
    
    TASK [check if file size is valid] ***********************************************
    fatal: [localhost]: FAILED! => {
        "assertion": "filesize left caret= 100",
        "changed": false,
        "evaluated_to": false,
        "msg": "file size must be between 0 and 100"
    }
    
    PLAY RECAP ***********************************************************************
    localhost                  : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
:::

As you can see, the task that is defined with the assert module fails
because the variable has a value that is not between the minimum and
maximum sizes that are defined.

Understanding the need for using the filter to convert the variable type
might not be easy. So, let's also look at [Listing
11-13](#ch11.xhtml#list11_13), which shows an example of a playbook that
will fail. You can see its behavior in [Listing
11-14](#ch11.xhtml#list11_14), where the playbook is executed.

**Listing 11-13** Failing Version of the [Listing
11-11](#ch11.xhtml#list11_11) Playbook

::: pre_1
    ---
    - hosts: localhost
      vars_prompt:
      - name: filesize
        prompt: "specify a file size in megabytes"
      tasks:
      - name: check if file size is valid
        assert:
          that:
          - filesize <= 100
          - filesize >= 1
          fail_msg: "file size must be between 0 and 100"
          success_msg: "file size is good, let\’s continue"
      - name: create a file
        command: dd if=/dev/zero of=/bigfile bs=1 count={{ filesize }}
:::

**Listing 11-14 ansible-playbook listing1113.yaml** Failing Result

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook listing1113.yaml
    specify a file size in megabytes:
    
    PLAY [localhost] *****************************************************************
    
    TASK [Gathering Facts] ***********************************************************
    ok: [localhost]
    
    TASK [check if file size is valid] ***********************************************
    fatal: [localhost]: FAILED! => {"msg": "The conditional check ’filesize left caret= 100’ failed. The error was: Unexpected templating type error occurred on ({% if filesize left caret= 100 %} True {% else %} False {% endif %}): ’left caret=’ not supported between instances of ’str’ and ’int’"}
    
    PLAY RECAP ***********************************************************************
    localhost                  : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
:::

As you can see, the code in [Listing 11-13](#ch11.xhtml#list11_13) fails
because the \left caret= test is not supported between a string and an integer.

In [Exercise 11-2](#ch11.xhtml#exe11_2) you work with some of the
modules discussed in this section.

::: box
**Exercise 11-2 Using Modules for Troubleshooting**

1\. Open your editor to create the file exercise112.yaml and define the
play header:

``` pre1
---
- name: using assert to check if volume group vgdata exists
  hosts: all
  tasks:
```

2\. Add a task that uses the command **vgs vgdata** to check whether a
volume group with the name vgdata exists. The task should use
**register** to register the command result, and it should continue if
this is not the case.

``` pre1
- name: check if vgdata exists
  command: vgs vgdata
  register: vg_result
  ignore_errors: true
```

3\. To make it easier to use **assert** in the next step on the right
variable, include a **debug** task to show the value of the variable:

``` pre1
- name: show vg_result variable
  debug:
    var: vg_result
```

4\. Add a task to print a success or failure message, depending on the
result of the **vgs** command from the first task:

``` pre1
- name: print a message
  assert:
    that:
    - vg_result.rc == 0
    fail_msg: volume group not found
    success_msg: volume group was found
```

5\. Use the command **ansible-playbook exercise112.yaml** to verify its
contents. Assuming that the LVM Volume Group vgdata was not found, it
should print "volume group not found."

6\. Change the playbook to verify that it will print the **success_msg**
if the requested volume group was found. You can do so by having it run
the command **vgs cl**, which on CentOS 8 should give a positive result.
:::

