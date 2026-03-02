---
title: 'Manage Users and Groups with Ansible'
description: 'How to Manage Users and Groups with Ansible'
showDate: false
---

## Using Ansible Modules to Manage Users and Groups

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

## Managing SSH Connections

- How to provide for SSH keys for new users in such a way that users are provided with SSH keys without having to set them up themselves. 
- To do this, you use the authorized_key module together with the **generate_ssh_key** argument to the user module.

#### Understanding SSH Connection Management Requirements

How SSH keys are used in the communication process between a user and an SSH server:
1. The user initiates a session with an SSH server.
2. The server sends back an identification token that is encrypted with the server private key to the user.
3. The user uses the server’s public key fingerprint, which is stored in the ~/.ssh/known_hosts file to verify the identification token.
4. If no public key fingerprint was stored yet in the ~/.ssh/known_hosts file, the user is prompted to store the remote server identity in the ~/.ssh/known_hosts file. At this point there is no good way to verify whether the user is indeed communicating with the intended server.
5. After establishing the identity of the remote server, the user can either send over a password or generate an authentication token that is based on the user’s private key.
6. If an authentication token that was based on the user’s private key is sent over, this token is received by the server, which tries to match it against the user’s public key that is stored in the ~/.ssh/authorized_keys file.
7. After the incoming authentication token to the stored user public key in the authorized_keys file is matched, the user is authenticated. If this authentication fails and password authentication is allowed, password authentication is attempted next.

In the authentication procedure, two key pairs play an important role. First, there is the server’s public/private key pair, which is used to establish a secure connection. 
To manage the host public key, you can use the Ansible known_hosts module. Next, there is the user’s public/private key pair, which the user uses to authenticate. To manage the public key in this key pair, you can use the Ansible authorized_key module.

#### Lookup Plug-in

- Enables Ansible to access data from outside sources. 
- Read the file system or contact external datastores and services.
- Ran on the Ansible control host.
- Results are usually stored in variables or templates. 

Set the value of a variable to the contents of a file:
```yml
---
- name: simple demo with the lookup plugin
  hosts: localhost
  vars:
    file_contents: "{{lookup(‘file’, ‘/etc/hosts’)}}"
  tasks:
  - debug:
                var: file_contents
```

#### Setting Up SSH User Keys

- To use SSH to connect to a user account on a managed host you can copy over the local user public key to the remote user ~/.ssh/authorized_keys file. 
- If the target authorized_keys file just has to contain one single key, you could use the copy module to copy it over. 
- To manage multiple keys in the remote user authorized_keys file, you’re better off using the authorized_key module. 

authorized_key module
- Copy the authorized_key for a user 
- /home/ansible/.ssh/id_rsa.pub is used as the source.
- lookup plug-in is used to refer to the file contents that should be used:
```yml
---
- name: authorized_key simple demo
  hosts: ansible2
  tasks:
  - name: copy authorized key for ansible user
    authorized_key:
      user: ansible
      state: present
      key: "{{ lookup(‘file’, ‘/home/ansible/.ssh/id_rsa.pub’) }}"
```

Do the same for multiple users:
vars/users
```yml
---
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

vars/groups
```yml
---
usergroups:
  - groupname: sales
  - groupname: account
```

```yml
---
- name: configure users with SSH keys
  hosts: ansible2
  vars_files:
    - vars/users
    - vars/groups
  tasks:
  - name: add groups
    group:
      name: "{{ item.groupname }}"
    loop: "{{ usergroups }}"
  - name: add users
    user:
      name: "{{ item.username }}"
      groups: "{{ item.groups }}"
    loop: "{{ users }}"
  - name: add SSH public keys
    authorized_key:
      user: "{{ item.username }}"
      key: "{{ lookup(‘file’, ‘files/’+ item.username + ‘/id_rsa.pub’) }}"
    loop: "{{ users }}"
```

- authorized_key module is set up to work on **item.username**, using a **loop** on the **users** variable. 
- The id_rsa.pub files that have to be copied over are expected to exist in the files directory, which exists in the current project directory.

- Copying over the user public keys to the project directory is a solution because the authorized_keys module cannot read files from a hidden directory.
- It would be much nicer to use **key: “{{ lookup(‘file’, ‘/home/’+ item.username + ‘.ssh/id_rsa.pub’) }}”**, but that doesn’t work. 

- In the first task you create a local user, including an SSH key. 
- Because an SSH key should include the name of the user and host that it applies to, you need to use the **generate_ssh_key** argument, as well as the **ssh_key_comment** argument to write the correct comment into the public key. 
- Without this content, the key will have generic content and not be considered a valid key.
```yml
- name: create the local user, including SSH key
  user:
    name: "{{ username }}"
    generate_ssh_key: true
    ssh_key_comment: "{{ username }}@{{ ansible_fqdn }}"
```

- After creating the SSH keys this way, you aren’t able to fetch the key directly from the user home directory. 
- To fix that problem, you create a directory with the name of the user in the project directory and copy the user public key from there by using the shell module:
```yml
- name: create a directory to store the file
  file:
    name: "{{ username }}"
    state: directory
- name: copy the local user ssh key to temporary {{ username }} key
  shell: ‘cat /home/{{ username }}/.ssh/id_rsa.pub > {{ username }}/id_rsa.pub’
- name: verify that file exists
  command: ls -l {{ username }}/
```

- Next, in the second play you create the remote user and use the authorized_key module to copy the key from the temporary directory to the new user home directory. 


**Exercise 13-2 Managing Users with SSH Keys**
Steps
1. Create a user on localhost.
2. Use the appropriate arguments to create the SSH public/private key pair according to the required format.
3. Make sure the public key is copied to a directory where it can be accessed. 
4. Uses the user module to create the user, as well as the authorized_key module to fetch the key from localhost and copy it to the .ssh/authorized_keys file in the remote user home directory.
5. Use the command **ansible-playbook exercise132.yaml -e username=radha** to create the user radha with the appropriate SSH keys.
6. To verify it has worked, use **sudo su - radha** on the control host, and type the command **ssh ansible1**. You should able to log in without entering a password.
```yml
---
- name: prepare localhost
  hosts: localhost
  tasks:

  - name: create the local user, including SSH key
    user:
      name: "{{ username }}"
      generate_ssh_key: true
      ssh_key_comment: "{{ username }}@{{ ansible_fqdn }}"
    
  - name: create a directory to store the file
    file:
      name: "{{ username }}"
      state: directory
      
  - name: copy the local user ssh key to temporary {{ username }} key
    shell: ‘cat /home/{{ username }}/.ssh/id_rsa.pub > {{ username }}/id_rsa.pub’
  - name: verify that file exists
    command: ls -l {{ username }}/

- name: setup remote host
  hosts: ansible1
  tasks:
  - name: create remote user, no need for SSH key
    user:
      name: "{{ username }}"
      
  - name: use authorized_key to set the password
    authorized_key:
      user: "{{ username }}"
      key: "{{ lookup(‘file’, ‘./’+ username +’/id_rsa.pub’) }}"
```

## Managing Encrypted Passwords

When managing users in Ansible, you probably want to set user passwords
as well. The challenge is that you cannot just enter a password as the
value to the **password:** argument in the user module because the user
module expects you to use an encrypted string.

#### Understanding Encrypted Passwords

When a user creates a password, it is encrypted. The hash of the
encrypted password is stored in the /etc/shadow file, a file that is
strictly secured and accessible only with root privileges. The string
looks like \$6\$237687687/\$9809erhb8oyw48oih290u09. In this string are
three elements, which are separated by \$ signs:

• The hashing algorithm that was used

• The random salt that was used to encrypt the password

• The encrypted hash of the user password

When a user sets a password, a random salt is used to prevent two users
who have identical passwords from having identical entries in
/etc/shadow. The salt and the unencrypted password are combined and
encrypted, which generates the encrypted hash that is stored in
/etc/shadow. Based on this string, the password that the user enters can
be verified against the password field in /etc/shadow, and if it
matches, the user is authenticated.

#### Generating Encrypted Passwords {.h4}

When you're creating users with the Ansible user module, there is a
password option. This option is not capable of generating an encrypted
password. It expects an encrypted password string as its input. That
means an external utility must be used to generate an encrypted string.
This encrypted string must be stored in a variable to create the
password. Because the variable is basically the user password, the
variable should be stored securely in, for example, an Ansible Vault
secured file.

To generate the encrypted variable, you can choose to create the
variable before creating the user account. Alternatively, you can run
the command to create the variable in the playbook, use **register** to
write the result to a variable, and use that to create the encrypted
user. If you want to generate the variable beforehand, you can use the
following ad hoc command:

    ansible localhost -m debug -a "msg={{ ‘password’ | password_hash(‘sha512’,’myrandomsalt’) }}"

This command generates the encrypted string as shown in [Listing
13-11](#ch13.xhtml#list13_11), and this string can next be used in a
playbook. An example of such a playbook is shown in [Listing
13-12](#ch13.xhtml#list13_12).

**Listing 13-11** Generating the Encrypted Password String

::: pre_1
    [ansible@control ~]$ ansible localhost -m debug -a "msg={{ ‘password’ | password_hash(‘sha512’,’myrandomsalt’) }}"
    localhost | SUCCESS => {
        "msg": "$6$myrandomsalt$McEB.xAVUWe0./6XqZ8n/7k9VV/Gxndy9nIMLyQAiPnhyBoToMWbxX2vA4f.Uv9PKnPRaYUUc76AjLWVAX6U10"
    }
:::

**Listing 13-12** Sample Playbook That Creates an Encrypted User
Password

```yml
    ---
    - name: create user with encrypted pass
      hosts: ansible2.example.com
      vars:
        password: "$6$myrandomsalt$McEB.xAVUWe0./6XqZ8n/7k9VV/Gxndy9nIMLyQAiPnhyBoToMWbxX2vA4f.Uv9PKnPRaYUUc76AjLWVAX6U10"
      tasks:
      - name: create the user
        user:
          name: anna
          password: "{{ password }}"
```

The method that is used here works but is not elegant. First, you need
to generate the encrypted password manually beforehand. Also, the
encrypted password string is used in a readable way in the playbook. By
seeing the encrypted password and salt, it's possible to get to the
original password, which is why the password should not be visible in
the playbook in a secure environment.

In [Exercise 13-3](#ch13.xhtml#exe13_3) you create a playbook that
prompts for the user password and that uses the debug module, which was
used in [Listing 13-11](#ch13.xhtml#list13_11) inside the playbook,
together with register, so that the password no longer is readable in
clear text. Before looking at [Exercise 13-3](#ch13.xhtml#exe13_3),
though, let's first look at an alternative approach that also works.

The procedure to use encrypted passwords while creating user accounts is
documented in the Frequently Asked Questions from the Ansible
documentation. Because the documentation is available on the exam, make
sure you know where to find this information! Search for the item "How
do I generate encrypted passwords for the user module?"

#### Using an Alternative Approach

As has been mentioned on multiple occasions, in Ansible often different
solutions exist for the same problem. And sometimes, apart from the most
elegant solution, there's also a quick-and-dirty solution, and that
counts for setting a user-encrypted password as well. Instead of using
the solution described in the previous section, "Generating Encrypted
Passwords," you can use the Linux command **echo password \| passwd
\--stdin** to set the user password. [Listing
13-13](#ch13.xhtml#list13_13) shows how to do this. Notice this example
focuses on how to do it, not on security. If you want to make the
playbook more secure, it would be nice to store the password in Ansible
Vault.

**Listing 13-13** Setting the User Password: Alternative Solution

```yml
    ---
    - name: create user with encrypted password
      hosts: ansible3
      vars:
        password: mypassword
        user: anna
      tasks:
      - name: configure user {{ user }}
        user:
          name: "{{ user }}"
          groups: wheel
          append: yes
          state: present
      - name: set a password for {{ user }}
        shell: ‘echo {{ password }} | passwd --stdin {{ user }}’
```

::: box
**Exercise 13-3 Creating Users with Encrypted Passwords**

1\. Use your editor to create the file exercise133.yaml.

2\. Write the play header as follows:

``` pre1
---
- name: create user with encrypted password
  hosts: ansible3
  vars_prompt:
  - name: passw
    prompt: which password do you want to use
  vars:
    user: sharon
  tasks:
```

3\. Add the first task that uses the debug module to generate the
encrypted password string and register to store the string in the
variable **mypass**:

``` pre1
- debug:
    msg: "{{ ‘{{ passw }}’| password_hash(‘sha512’,’myrandomsalt’) }}"
  register: mypass
```

4\. Add a debug module to analyze the exact format of the registered
variable:

``` pre1
- debug:
    var: mypass
```

5\. Use **ansible-playbook exercise133.yaml** to run the playbook the
first time so that you can see the exact name of the variable that you
have to use. This code shows that the **mypass.msg** variable contains
the encrypted password string (see [Listing
13-14](#ch13.xhtml#list13_14)).

**Listing 13-14** Finding the Variable Name Using debug

::: pre_1
``` pre1
TASK [debug] *******************************************************************
ok: [ansible2] => {
    "mypass": {
        "changed": false,
        "failed": false,
        "msg": "$6$myrandomsalt$Jesm4QGoCGAny9ebP85apmh0/uUXrj0louYb03leLoOWSDy/imjVGmcODhrpIJZt0rz.GBp9pZYpfm0SU2/PO."
    }
}
```
:::

6\. Based on the output that you saw with the previous command, you can
now use the user module to refer to the password in the right way. Add
the following task to do so:

``` pre1
- name: create the user
  user:
    name: "{{ user }}"
    password: "{{ mypass.msg }}"
```

7\. Use **ansible-playbook exercise133.yaml** to run the playbook and
verify its output.
:::

### Lab Managing Users

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
