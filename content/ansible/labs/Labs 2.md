### Managing Users Advanced Scenario Exercise

It's time to work on an advanced scenario now. [Exercise
13-4](#ch13.xhtml#exe13_4) includes a step-by-step procedure that guides
you through the process of setting up a complex playbook. In this
procedure I tried to give you practical guidelines on how to approach
such a complex task on the exam, including the part where you may change
your mind because you have realized there is a more efficient method. It
is important to read the steps carefully because some improvements will
be applied while working on this procedure.

::: note

------------------------------------------------------------------------

**Warning**

This exercise is written such that you can learn from errors that are
made. In early steps, some configuration is created that will be
optimized later. I purposely used this approach, and you are advised to
closely follow the steps in the exercise before investigating the final
solution in the exercise134.yaml playbook.

------------------------------------------------------------------------
:::

::: box
**Exercise 13-4 Setting Up Ansible Users**

In this exercise you create a few Ansible users. The users need to be
created on the Ansible control host as well as on the managed hosts, and
after running the playbook, any user created on the localhost must be
able to log in using SSH keys to the corresponding user account on the
remote host without having to enter a password. Make sure that the setup
meets the following requirements:

• Create users sharon, blair, ashley, and ahmed.

• Users sharon and blair are members of the group admins; users ashley
and ahmed are members of the group students.

• On the managed hosts, members of the group admins should have sudo
privileges to run any command they want.

• All users must be configured with the default password "password".

1\. This time you're going to use a different approach and set up the
framework of the playbook first. This is a good approach to start the
development of more complex playbooks and minimizes chances that you
miss anything in the playbook. To do so, use your editor to create a
file with the name exercise134.yaml, and define the play headers and the
names and modules you intend to use for each of the tasks according to
the following example:

``` pre1
---
- name: create users on localhost
  hosts: localhost
  tasks:
  - name: create groups
    groups:
  - name: create users
    users:

- name: create users on managed hosts
  hosts: ansible4
  tasks:
  - name: create groups
    groups:
  - name: create users
    users:
  - name: copy authorized keys
    authorized_key:
  - name: modify sudo configuration
    template
```

2\. Now that you have defined the global structure, you can start
filling it in with details. Begin with the first play, which should
start with the creation of the user accounts. In this play, users and
groups need to be created. To start with, focus on the basic setup and
fill it in with additional details later:

``` pre1
---
- name: create users on localhost
  hosts: localhost
  vars:
    users:
    - username: sharon
      groups: admins
    - username: blair
      groups: admins
    - username: ashley
      groups: students
    - username: ahmed
      groups: students
  tasks:
  - name: create groups
    groups:
      name: "{{ item.groups }}"
      state: present
    loop: "{{ users }}"
  - name: create users
    user:
      name: "{{ item.username }}"
      groups: "{{ item.groups }}"
    loop: "{{ users }}"
```

3\. Because you're in for a big project this time, now is a good moment
to give it a try. To do so, temporarily comment out the entire second
play and run the playbook in check mode by using **ansible-playbook -C
exercise134.yaml**. If you typed the exact text listed in step 2, you
get an error at the line where the groups module is referred to. That's
right---there is no groups module; the name is group. Correct this and
run the playbook again in check mode. Notice that in check mode you
might get false errors. Just double-check, and if you're convinced
you've done it right, ignore the error. Notice that it also doesn't
really hurt if you just run the playbook. Any later modifications will
be added to the configuration anyway.

4\. Now you're ready to complete the first play by adding the remaining
tasks to it. To do so, you still have to do two things, all of which
must be done on the user module: you need to set the user password, and
you need to create an SSH key pair. To generate the password, you need
to generate an encrypted string that can be used as an argument to the
user module **password** argument. To generate this string, use an ad
hoc command: **ansible localhost -m debug -a "msg={{ 'password' \|
password_hash('sha512', 'mysalt') }}"**. Just copy the crypto string
this generates (it starts with \$6\$) and use that in the next step.

5\. Complete the user task in the first play with the
**generate_ssh_key** and **password** arguments. The complete task looks
as follows:

``` pre1
- name: create users
    user:
      name: "{{ item.username }}"
      groups: "{{ item.groups }}"
      generate_ssh_key: yes
      password: $6$mysalt$khiuhihrb8y349hwbohbuoehr8bhqohoibhro8bohoiheoi
    loop: "{{ users }}"
  tags: setuplocal
```

Notice the use of **tags: setuplocal** on the last line; this tag makes
it easier to run specific parts of the playbook only, which can be
convenient later in the setup procedure. You might want to run the
playbook now by using **ansible-playbook exercise134.yaml
\--tags=setuplocal**.

6\. At this point the local part of the setup seems to be done, so you
can work on the second play. You should start by observing what you're
trying to do. In the second play, a couple of tasks are exactly the same
as in the first play. Because just repeating the same stuff again
wouldn't be very efficient, you can create some imports instead and move
the existing code to a file that will be imported. To start with, create
the exercise134-vars.yaml file and give it the following contents:

``` pre1
---
    users:
    - username: sharon
      groups: admins
    - username: blair
      groups: admins
    - username: ashley
      groups: students
    - username: ahmed
      groups: students
```

7\. Create the exercise134-tasks.yaml file and give it the following
contents:

``` pre1
- name: create groups
    group:
      name: "{{ item.groups }}"
      state: present
    loop: "{{ users }}"
- name: create users
    user:
      name: "{{ item.username }}"
      groups: "{{ item.groups }}"
      generate_ssh_key: yes
      ssh_key_comment: "{{ item.username }}@{{ ansible_fqdn }}"
      password: $6$mysalt$khiuhihrb8y349hwbohbuoehr8bhqohoibhro8bohoiheoi
    loop: "{{ users }}"
```

8\. Now it's time to rewrite the playbook so that the entire playbook
looks as follows (note that the last two tasks still need to be
defined):

``` pre1
---
- name: create users on localhost
  hosts: localhost
  vars_files:
  - exercise134-vars.yaml
  tasks:
  - name: include user and group setup
    import_tasks: exercise134-tasks.yaml
  tags: setuplocal

- name: create users on managed hosts
  hosts: ansible4
  vars_files:
  - exercise134-vars.yaml
  tasks:
  - name: include user and group setup
    import_tasks: exercise134-tasks.yaml
  - name: copy authorized keys
    authorized_key:
  - name: modify sudo configuration
```

9\. You can work on the copy authorized keys tasks at this point.
Because the users were created on localhost and each user has its own
SSH key pair, this step appears to be fairly easy. The challenge in this
task is the use of the lookup plug-in. Complete the authorized_key task
such that the second play looks as follows:

``` pre1
- name: create users on managed hosts
  hosts: ansible4
  vars_files:
  - exercise134-vars.yaml
  tasks:
  - name: include user and group setup
    import_tasks: exercise134-tasks.yaml
  - name: copy authorized keys
    authorized_key:
      user: "{{ item.username }}"
      key: "{{ lookup(‘file’, ‘/home/’+ item.username + ‘/.ssh/id_rsa.pub’) }}"
    loop: "{{ users }}"
  - name: modify sudo configuration
  tags: setupremote
```

10\. Because you can easily make an error while using the lookup
plug-in, it's a good idea to run the second play by using
**ansible-playbook exercise134.yaml \--tags=setupremote**. Notice that
this play works only if the first play has been executed successfully.
And oops! That doesn't work out well! You can see the error shown in
[Listing 13-15](#ch13.xhtml#list13_15). This error is generated because
the authorized_keys module cannot access the id_rsa.pub file directly
from the hidden .ssh directory in the user home directory.

**Listing 13-15** Task 10 Error Output

::: pre_1
``` pre1
TASK [copy authorized keys] *******************************************************
[WARNING]: Unable to find ‘/home/laksmi/id_rsa.pub’ in expected paths (use -vvvvv
to see paths)
fatal: [ansible3]: FAILED! => {"msg": "An unhandled exception occurred while running the lookup plugin ‘file’. Error was a <class ‘ansible.errors.AnsibleError’>, original message: could not locate file in lookup: /home/laksmi/id_rsa.pub"}
```
:::

11\. To fix the error that occurred in step 10, you must rewrite the
first play with the solution discussed in the earlier section "Managing
SSH Connections." The following code shows the entire first play, with
the modifications you need to make applied after the **import_tasks:**
statement:

``` pre1
---
- name: create users on localhost
  hosts: localhost
  vars_files:
  - exercise134-vars.yaml
  tasks:
  - name: include user and group setup
    import_tasks: exercise134-tasks.yaml
  - name: create a directory to store the key file
    file:
      name: "{{ item.username }}"
      state: directory
    loop: "{{ users }}"
  - name: copy the local user ssh key to temporary {{ item.username }} key
    shell: ‘cat /home/{{ item.username }}/.ssh/id_rsa.pub > {{ item.username }}/id_rsa.pub’
    loop: "{{ users }}"
  - name: verify that file exists
    command: ls -l {{ item.username }}/
    loop: "{{ users }}"
  tags: setuplocal
```

12\. Now it's time to configure the sudo file in the /etc/sudoers.d/
directory. While you've been setting up the rough structure of the
playbook so far, using the template module has been suggested. But the
fact is that the file that needs to be created is simple and
straightforward, and just needs to contain the line **%admins
ALL=(ALL:ALL) NOPASSWD:ALL**. Because this is a simple task, you don't
need to use the template module. Just use the copy module instead, such
that after the authorized_key task, only the following task is included:

``` pre1
- name: copy sudoers file
  copy:
    content: ‘%admins ALL=(ALL:ALL) NOPASSWD:ALL’
    dest: /etc/sudoers.d/admins
```

13\. Before running the playbook, you may verify what you have typed
with the sample playbook in the GitHub repository at
<https://github.com/sandervanvugt/rhce8-book/exercise134.yaml>.

14\. At this point, you can run the playbook by using **ansible-playbook
exercise134.yaml**, and you should encounter no errors.

15\. To verify that all works well, on the control host, type **sudo
su - ahmed**, and once in a shell as user ahmed, type **ssh ansible2**.
Ansible should let the user in without asking for a password.
:::

### Lab 13-1 {#ch13.xhtml#h3_178 .h3}

Write a playbook that creates users according to the following
specifications:

• Create users kim, christina, kelly, and bill.

• Users kim and kelly must be members of the profs group; users
christina and bill are members of the students group.

• While creating the users, set the encrypted password to "password".

• Ensure that members of the group profs have sudo privileges.
:::

[]{#ch14.xhtml}

::: {#ch14.xhtml#sbo-rt-content}
