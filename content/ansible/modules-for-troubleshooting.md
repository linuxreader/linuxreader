---
title: 'Using Modules for Troubleshooting and Testing'
description: 'Using Modules for Troubleshooting and Testing'
showDate: false
---

## Using Modules for Troubleshooting and Testing 

Modules used for troubleshooting

debug - Write debug information. Useful for checking variable values
uri - Test answer coming from any URL. Useful for API calls
fail - Uses when conditional to determine when a module should be considered failing
script - Allows you to execute a shell script on the managed host
stat - Gathers status information about files
assert - tests whether the expected result is present and otherwise fails
 
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

**debug:**
Prints statements during execution. Used for debugging variables or expressions without stopping a playbook. 

Print out the value of the ansible_facts variable:
```yaml
debug:
  var: ansible_facts
```
#### Using the uri Module {.h4}

**uri:**
Interacts with basic http and https web services. (Verify connectivity to a web server
+9)

Test httpd accessibility:
```yaml
uri:
  url: http://ansible1
```

Show result of the command while running the playbook:
```yaml
uri:
  url: http://ansible1
  return_content: yes
```

Show the status code that signifies the success of the request:
```yaml
uri:
  url: http://ansible1
  status_code: 200
```

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

### Troubleshooting Common Scenarios

Apart from the problems that may arise in playbooks, another type of
error relates to connectivity issues. To connect to managed hosts, SSH
must be configured correctly, and also authentication and privilege
escalation must work as expected.

#### Analyzing Connectivity Issues {.h4}

To be able to connect to a managed host, you need to have an IP network
connection. Apart from that, you need to make sure that the host has
been set up correctly:

• The SSH service needs to be accessible on the remote host.

• Python must be installed.

• Privilege escalation needs to be set up.

Apart from these, inventory settings may be specified to indicate how to
connect to a remote host. Normally, the inventory contains a host name
only. If a host resolves to multiple IP addresses, you may want to
specify how exactly the remote host must be connected to. The
**ansible_host** parameter can be configured to do so. In inventory, for
instance, you may include the following line to ensure that your host is
connected in the right way:

    ansible5.example.com ansible_host=192.168.4.55

Notice that this setting makes sense only in an environment where a host
can be reached on multiple different IP addresses.

To test connectivity to remote hosts, you can use the ping module. It
checks for IP connectivity, accessibility of the SSH service, sudo
privilege escalation, and the availability of a Python stack. The ping
module does not take any arguments. [Listing
11-18](#ch11.xhtml#list11_18) shows the result of running on the ad hoc
command **ansible all -m ping** where hosts that are available send
"pong" as a reply, and for hosts that are not available, you see why
they are not available.

**Listing 11-18** Verifying Connectivity Using the ping Module

::: pre_1
    [ansible@control rhce8-book]$ ansible all -m ping
    ansible2 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/libexec/platform-python"
        },
        "changed": false,
        "ping": "pong"
    }
    ansible1 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/libexec/platform-python"
        },
        "changed": false,
        "ping": "pong"
    }
    ansible3 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/libexec/platform-python"
        },
        "changed": false,
        "ping": "pong"
    }
    ansible4 | FAILED! => {
        "msg": "Missing sudo password"
    }
:::

#### Analyzing Authentication Issues {.h4}

A few settings play a role in authentication on the remote host to
execute tasks:

• The **remote_user** setting determines which user account to use on
the managed nodes.

• SSH keys need to be configured for the remote_user to enable smooth
authentication.

• The **become** parameter needs to be set to true.

• The **become_user** needs to be set to the root user account.

• Linux sudo needs to be set up correctly.

In [Exercise 11-4](#ch11.xhtml#exe11_4) you work on troubleshooting some
common scenarios.

::: box
**Exercise 11-4 Troubleshooting Connectivity Issues**

1\. Use an editor to create the file exercise114-1.yaml and give it the
following contents:

``` pre1
---
- name: remove user from wheel group
  hosts: ansible4
  tasks:
  - user:
      name: ansible
      groups: ’’
```

2\. Run the playbook using **ansible-playbook exercise114-1.yaml** and
use **ansible ansible4 -m reboot** to reboot node ansible4.

3\. Once the reboot is completed, use **ansible all -m ping** to verify
connectivity. Host ansible4 should give a "Missing sudo password" error.

4\. Type **ansible ansible4 -m raw -a "usermod -aG wheel ansible" -u
root -k** to make user ansible a member of the group wheel again.

5\. Repeat the **ansible all -m ping** command. You should now be able
to connect normally to the host ansible4 again.
:::

# Managing Ansible Errors and Logs

#### Using Check Mode


Before actually running a playbook in a way that all changes are
implemented, you can start the playbooks in check mode. To do this, you
use the **\--check** or **-C** command-line argument to the **ansible**
or **ansible-playbook** command. The effect of using check mode is that
changes that would have been made are shown but not executed. You should
realize, though, that check mode is not supported in all cases. You
will, for instance, have problems with check mode if it is applied to
conditionals, where a specific task can do its work only after a
preceding task has made some changes. Also, to successfully use check
mode, the modules need to support it, but some don't. Modules that don't
support check mode don't show any result while running check mode, but
also they don't make any changes.

Apart from the command-line argument, you can use **check_mode: yes** or
**check_mode: no** with any task in a playbook. If **check_mode: yes**
is used, the task always runs in check mode (and does not implement any
changes), regardless of the use of the **\--check** option. If a task
has **check_mode: no** set, it never runs in check mode and just does
its work, even if the **ansible-playbook** command is used with the
**\--check** option. Using check mode on individual tasks might be a
good idea if using check mode on the entire playbook gives unpredicted
results: you can enable it on just a couple of tasks to ensure that they
run successfully before proceeding to the next set of tasks. Notice that
using **check_mode: no** for specific tasks can be dangerous; these
tasks will make changes, even if the entire playbook was started with
the **\--check** option!

::: note

------------------------------------------------------------------------

**Note**

The **check_mode** argument is a replacement for the **always_run**
option that was used in Ansible 2.5 and earlier. In current Ansible
versions, you should not use **always_run** anymore.

Another option that is commonly used with the **\--check** option is
**\--diff**. This option reports changes to template files without
actually applying them. [Listing 11-1](#ch11.xhtml#list11_1) shows a
sample playbook, [Listing 11-2](#ch11.xhtml#list11_2) shows the template
that it is processing, and [Listing 11-3](#ch11.xhtml#list11_3) shows
the result of running this playbook with the **ansible-playbook
listing111.yaml \--check \--diff** command.

------------------------------------------------------------------------
```yaml
    ---
    - name: simple template example
      hosts: ansible2
      tasks:
      - template:
          src: listing112.j2
          dest: /etc/issue
:::

**Listing 11-2** Sample Template File

::: pre_1
    {# /etc/issue #}
    Welcome to {{ ansible_facts[’hostname’] }}
:::

**Listing 11-3** Running the listing111.yaml Sample Playbook

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook listing111.yaml --check --diff
    
    PLAY [simple template example] *************************************************
    
    TASK [Gathering Facts] *********************************************************
    ok: [ansible2]
    
    TASK [template] ****************************************************************
    --- before
    +++ after: /home/ansible/.ansible/tmp/ansible-local-4493uxbpju1e/tmpm5gn7crg/listing112.j2
    @@ -0,0 +1,3 @@
    +Welcome to ansible2
    +
    +
    
    changed: [ansible2]
    
    PLAY RECAP *********************************************************************
    ansible2                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
#### Understanding Output

When you run the **ansible-playbook** command, output is generated.
You've probably had a glimpse of it before, but let's look at the output
in a more structured way now. [Listing 11-4](#ch11.xhtml#list11_4) shows
some typical sample output generated by running the **ansible-playbook**
command.

**Listing 11-4 ansible-playbook** Command Output

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook listing52.yaml
    
    PLAY [install start and enable httpd] ******************************************
    
    TASK [Gathering Facts] *********************************************************
    ok: [ansible2]
    ok: [ansible1]
    ok: [ansible3]
    ok: [ansible4]
    
    TASK [install package] *********************************************************
    changed: [ansible2]
    changed: [ansible1]
    changed: [ansible3]
    changed: [ansible4]
    
    TASK [start and enable service] ************************************************
    changed: [ansible2]
    changed: [ansible1]
    changed: [ansible3]
    changed: [ansible4]
    
    PLAY RECAP *********************************************************************
    ansible1                   : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible2                   : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible3                   : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible4                   : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
:::

In the output of any **ansible-playbook** command, you can see different
items:

![Image](/Images/key_topic_icon.jpg){width="64" height="51"}

• An indicator of the play that is started

• If not disabled, the Gathering Facts task that is executed for each
play

• Each individual task, including the task name if that was specified

• The Play Recap, which summarizes the play results

In the Play Recap, different results can be shown. [Table
11-2](#ch11.xhtml#tab11_2) gives an overview.

::: group
**Table 11-2** Playbook Recap Overview

![Image](../../../../images/11tab02%201.jpg){width="941" height="338"}
:::

As discussed before, when you use the **ansible-playbook** command, you
can increase the output verbosity level using one or more **-v**
options. [Table 11-3](#ch11.xhtml#tab11_3) lists what these options
accomplish. For generic troubleshooting, you might want to consider
using **-vv**, which shows output as well as input data. In particular
cases using the **-vvv** option can be useful because it adds connection
information as well.

The **-vvvv** option just brings too much information in many cases but
can be useful if you need to analyze which exact scripts are executed or
whether any problems were encountered in privilege escalation. Make sure
to capture the output of any command that runs with **-vvvv** to a text
file, though, so that you can read it in an easy way. Even for a simple
playbook, it can easily generate more than 10 screens of output.

::: group
**Table 11-3** Verbosity Options Overview

![Image](../../../../images/11tab03%201.jpg){width="941" height="209"}
:::

In [Listing 11-5](#ch11.xhtml#list11_5) you can see the output of a
small playbook that runs different tasks on the managed hosts. [Listing
11-5](#ch11.xhtml#list11_5) shows details about execution of one task on
host ansible4, and as you can see, it goes deep in the amount of detail
that is shown. One component is worth looking at, and that is the
escalation succeeded that you can see in the output. This means that
privilege escalation was successful and tasks were executed because
**become_user** was defined in ansible.cfg. Failing privilege escalation
is one of the common reasons why playbook execution may go wrong, which
is why it's worth keeping an eye on this indicator.

**Listing 11-5** Analyzing Partial **-vvvv** Output

```
    <ansible4> ESTABLISH SSH CONNECTION FOR USER: ansible
    <ansible4> SSH: EXEC ssh -vvv -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o ’User="ansible"’ -o ConnectTimeout=10 -o ControlPath=/home/ansible/.ansible/cp/859d5267e3 ansible4 ’/bin/sh -c ’"’"’chmod u+x /home/ansible/.ansible/tmp/ansible-tmp-1587544652.4716983-118789810824208/ /home/ansible/.ansible/tmp/ansible-tmp-1587544652.4716983-118789810824208/AnsiballZ_systemd.py && sleep 0’"’"’’
    Escalation succeeded
    <ansible4> (0, b’’, b"OpenSSH_8.0p1, OpenSSL 1.1.1c FIPS  28 May 2019\r\ndebug1: Reading configuration data /etc/ssh/ssh_config\r\ndebug3: /etc/ssh/ssh_config line 51: Including file /etc/ssh/ssh_config.d/05-redhat.conf depth 0\r\ndebug1: Reading configuration data /etc/ssh/ssh_config.d/05-redhat.conf\r\ndebug2: checking match for ’final all’ host ansible4 originally ansible4\r\ndebug3: /etc/ssh/ssh_config.d/05-redhat.conf line 3: not matched ’final’\r\ndebug2: match not found\r\ndebug3: /etc/ssh/ssh_config.d/05-redhat.conf line 5: Including file /etc/crypto-policies/back-ends/openssh.config depth 1 (parse only)\r\ndebug1: Reading configuration data /etc/crypto-policies/back-ends/openssh.config\r\ndebug3: gss kex names ok: [gss-gex-sha1-,gss-group14-sha1-]\r\ndebug3: kex names ok: [curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1]\r\ndebug1: configuration requests final Match pass\r\ndebug1: re-parsing configuration\r\ndebug1: Reading configuration data /etc/ssh/ssh_config\r\ndebug3: /etc/ssh/ssh_config line 51: Including file /etc/ssh/ssh_config.d/05-redhat.conf depth 0\r\ndebug1: Reading configuration data /etc/ssh/ssh_config.d/05-redhat.conf\r\ndebug2: checking match for ’final all’ host ansible4 originally ansible4\r\ndebug3: /etc/ssh/ssh_config.d/05-redhat.conf line 3: matched ’final’\r\ndebug2: match found\r\ndebug3: /etc/ssh/ssh_config.d/05-redhat.conf line 5: Including file /etc/crypto-policies/back-ends/openssh.config depth 1\r\ndebug1: Reading configuration data /etc/crypto-policies/back-ends/openssh.config\r\ndebug3: gss kex names ok: [gss-gex-sha1-,gss-group14-sha1-]\r\ndebug3: kex names ok: [curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1]\r\ndebug1: auto-mux: Trying existing master\r\ndebug2: fd 4 setting O_NONBLOCK\r\ndebug2: mux_client_hello_exchange: master version 4\r\ndebug3: mux_client_forwards: request forwardings: 0 local, 0 remote\r\ndebug3: mux_client_request_session: entering\r\ndebug3: mux_client_request_alive: entering\r\ndebug3: mux_client_request_alive: done pid = 4764\r\ndebug3: mux_client_request_session: session request sent\r\ndebug3: mux_client_read_packet: read header failed: Broken pipe\r\ndebug2: Received exit status from master 0\r\n")
    <ansible4> ESTABLISH SSH CONNECTION FOR USER: ansible
    <ansible4> SSH: EXEC ssh -vvv -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o ’User="ansible"’ -o ConnectTimeout=10 -o ControlPath=/home/ansible/.ansible/cp/859d5267e3 -tt ansible4 ’/bin/sh -c ’"’"’sudo -H -S -n  -u root /bin/sh -c ’"’"’"’"’"’"’"’"’echo BECOME-SUCCESS-muvtpdvqkslnlegyhoibfcrilvlyjcqp ; /usr/libexec/platform-python /home/ansible/.ansible/tmp/ansible-tmp-1587544652.4716983-118789810824208/AnsiballZ_systemd.py’"’"’"’"’"’"’"’"’ && sleep 0’"’"’’
    Escalation succeeded
```

#### Optimizing Command Output Error Formatting

You might have noticed that the formatting of error messages in Ansible
command output can be a bit hard to read. Fortunately, there's an easy
way to make it a little more readable by including two options in the
ansible.cfg file. These options are **stdout_callback = debug** and
**stdout_callback = error**. After including these options, you'll
notice it's a lot easier to read error output and distinguish between
its different components!

#### Logging to Files {.h4}

By default, Ansible does not write anything to log files. The reason is
that the Ansible commands have all the options that may be useful to
write output to the STDOUT. If so required, it's always possible to use
shell redirection to write the command output to a file.

If you do need Ansible to write log files, you can set the **log_path**
parameter in ansible.cfg. Alternatively, Ansible can log to the filename
that is specified as the argument to the \$ANSIBLE_LOG_PATH variable.
Notice that Ansible logs can grow big very fast, so if logging to output
files is enabled, make sure that Linux log rotation is configured to
ensure that files cannot grow beyond a specific maximum size.

#### Running Task by Task {.h4}

When you analyze playbook behavior, it's possible to run playbook tasks
one by one or to start running a playbook at a specific task. The
**ansible-playbook \--step** command runs playbooks task by task and
prompts for confirmation before running the next task. Alternatively,
you can use the **ansible-playbook \--start-at-task=\"task name\"**
command to start playbook execution as a specific task. Before using
this command, you might want to use **ansible-playbook \--list-tasks**
for a list of all tasks that have been configured. To use these options
in an efficient way, you must configure each task with its own name. In
[Listing 11-6](#ch11.xhtml#list11_6) you can see what running playbooks
this way looks like. This listing first shows how to list tasks in a
playbook and next how the **\--start-at-task** and **\--step** options
are used.

**Listing 11-6** Running Tasks One by One

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook --list-tasks exercise81.yaml
    
    playbook: exercise81.yaml
    
      play #1 (ansible1): testing file manipulation skills.    TAGS: []
        tasks:
          create a new file              TAGS: []
          check status of the new file   TAGS: []
          for debugging purposes only    TAGS: []
          change file owner if needed    TAGS: []
    
      play #2 (ansible1): fetching a remote file.    TAGS: []
        tasks:
          fetch file from remote machine.    TAGS: []
    
      play #3 (localhost): adding text to the file that is now on localhost TAGS: []
        tasks:
          add a message.    TAGS: []
    
      play #4 (ansible2): copy the modified file to ansible2.    TAGS: []
        tasks:
          copy motd file.    TAGS: []
    [ansible@control rhce8-book]$ ansible-playbook --start-at-task "add a message"  --step exercise81.yaml
    
    PLAY [testing file manipulation skills] ****************************************
    
    PLAY [fetching a remote file] **************************************************
    
    PLAY [adding text to the file that is now on localhost] ************************
    Perform task: TASK: Gathering Facts (N)o/(y)es/(c)ontinue:
:::

In [Exercise 11-1](#ch11.xhtml#exe11_1) you learn how to apply check
mode while working with templates.

::: box
**Exercise 11-1 Using Templates in Check Mode**

1\. Locate the file httpd.conf; you can find it in the rhce8-book
directory, which you can download from the GitHub repository at
<https://github.com/sandervanvugt/rhce8-book>. Use **mv httpd.conf
exercise111-httpd.j2** to rename it to a Jinja2 template file.

2\. Open the exercise111-httpd.j2 file with an editor, and apply
modifications to existing parameters so that they look like the
following:

``` pre1
ServerRoot "{{ apache_root }}"
User {{ apache_user }}
Group {{ apache_group }}
```

3\. Write a playbook that takes care of the complete Apache web server
setup and installation, starts and enables the service, opens a port in
the firewall, and uses the template module to create the
/etc/httpd/conf/httpd.conf file based on the template that you created
in step 2 of this exercise. The complete playbook with the name
exercise111.yaml looks like the following (make sure you have the
*exact* contents shown below and do *not* correct any typos):

``` pre1
---
- name: perform basic apache setup
  hosts: ansible2
  vars:
    apache_root: /etc/httpd
    apache_user: httpd
    apache_group: httpd
  tasks:
  - name: install RPM package
    yum:
      name: httpd
      state: latest
  - name: copy template file
    template:
      src: exercise111-httpd.j2
      dest: /etc/httpd/httpd.conf
  - name: start and enable service
    service:
      name: httpd
      state: started
      enabled: yes
  - name: open port in firewall
    firewalld:
      service: http
      permanent: yes
      state: enabled
      immediate: yes
```

4\. Run the command **ansible-playbook \--syntax-check
exercise111.yaml**. If no errors are found in the playbook syntax, you
should just see the name of the playbook.

5\. Run the command **ansible-playbook \--check \--diff
exercise111.yaml**. In the output of the command, pay attention to the
task copy template file. After the line that starts with **+++ after**,
you should see the lines in the template that were configured to use a
variable, using the right variables.

6\. Run the playbook to perform all its tasks step by step, using the
command **ansible-playbook \--step exercise111.yaml**. Press **y** to
confirm the first step. Next, press **c** to automatically continue. The
playbook will fail on the copy template file task because the target
directory does not exist. Notice that the **\--syntax-check** and the
**\--check** options do not check for any logical errors in the playbook
and for that reason have not detected this problem.

7\. Edit the exercise111.yaml file and ensure the **template** task
contains the following corrected line: (replace the old line starting
with **dest:**):

``` pre1
dest: /etc/httpd/conf/httpd.conf
```

8\. Type **ansible-playbook \--list-tasks exercise111.yaml** to list all
the tasks in the playbook.

9\. To avoid running the entire playbook again, use **ansible-playbook
\--start-at-task=\"copy template file\" exercise111.yaml** to run the
playbook to completion.
:::