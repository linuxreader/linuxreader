+++
title = 'Encrypted passwords'
description = 'Encrypted passwords'
+++
### Managing Encrypted Passwords

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
