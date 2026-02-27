## 12 Managing Software with Ansible

### Implementing a Playbook to Manage Software 

In the previous sections you learned how to use different modules to
manage software. In this section you apply this knowledge in a more
advanced playbook example. You'll also find an advanced example like
this in Chapters 13 through 15 so that you get the best possible
preparation for the EX294 exam. To match the exam style of questions,
the example is scenario based, and the assignment is formatted as a
step-by-step exercise. As is the case for all exercises, the complete
playbook discussed here is available in the GitHub repository at
<https://github.com/sandervanvugt/rhce8-book/exercise123.yaml>.

To run this assignment on a RHEL 8 target, you need access to a valid
RHEL 8 subscription. If you don't have a current subscription, you can
request it from <https://developers.redhat.com>.

::: box
**Exercise 12-3 Configuring a New RHEL Managed Node**

Create a playbook that will fully automate the setup of a new RHEL host.
Write this playbook according to the following requirements:

• Add the new host information to the inventory and /etc/hosts file on
the control host.

• Work with variables for the name of the host you want to set up.

• Connect as user root to the new host. While running the playbook, run
it with the appropriate option so that you will be prompted for the root
password.

• Set up the user ansible on the new host and make sure this user is a
member of the group wheel.

• Also, set a password for user ansible using the playbook.

• Modify the sudoers file such that the user ansible can run root
commands using sudo without having to enter a password.

• Automatically register the host with the RHEL Subscription Manager.

• Use Ansible Vault to provide the username and password in a secure
way.

• Add the new host to the rh-gluster-3-client-for-rhel-8-x86_64-rpms
repository and the rhel-8-for-x86_64-appstream-debug-rpms repository.

• Use tags so that you can run individual parts of the playbook.

1\. On the control host, use **sudo yum install sshpass** to install the
sshpass package. This package enables you to work with SSH passwords in
a noninteractive way, and you need it to automate working with SSH
passwords from a playbook environment.

2\. To start, you need to set up the control host to include information
about the new host. To make this playbook flexible, this playbook
requires variables to be set from the command line because that is the
only way to ensure that the variable is available in all of the plays.
Using **vars_prompt** would have been an option, but variables that are
set with **vars_prompt** apply only to the play in which they are set.
To check that the variables were indeed passed as an argument to the
**ansible-playbook** command, use the fail module as follows to check
whether the variable **newhost** and the variable **newhostip** are
provided as startup arguments. Create a file with the name
exercise123.yaml as follows:

``` pre1
- name: add host to inventory
  hosts: localhost
  tasks:
  - fail:
      msg: "add the options -e newhost=hostname -e newhostip=ip.ad.dr.ess and try again"
    when: (newhost is undefined) or (newhostip is undefined)
```

3\. Write the tasks for this first play. In these tasks, you want to
make sure that the local inventory file and the /etc/hosts file are
modified. To do this, the lineinfile module provides good service.
Notice the use of the line the second time the lineinfile module is
called. The line contains only variables, and for that reason the entire
variable string must be between double quotes. Also, at the end of the
play, include the **tags: addhost** line, which makes it easy to skip
this task after it has run successfully in case it is needed to run the
playbook again. Make sure to add the following text to complete the
first play:

``` pre1
- name: add new host to inventory
  lineinfile:
    path: inventory
    state: present
    line: "{{ newhost }}"
- name: add new host to /etc/hosts
  lineinfile:
    path: /etc/hosts
    state: present
    line: "{{ newhostip }}  {{ newhost}}"
tags: addhost
```

4\. At this point it's a good idea to test that all is going well so
far. Use **ansible-playbook -C exercise123.yaml** and observe playbook
output. It should fail because no arguments are provided on the command
line. Use **ansible-playbook -C exercise123.yaml -e newhost=ansible5 -e
newhostip=192.168.4.205** and try again.

5\. The first play is ready at this point, so it's time to configure the
second play. This play is executed on the new host. The target host name
is set to the variable **newhost**, which is defined while running the
**ansible-playbook** command. Also notice that the **remote_user** and
the **become** statements must be set because the default user ansible
is not configured for privilege escalation yet. Write the play header
for this second play as follows:

``` pre1
- name: configure a new RHEL host
  hosts: "{{ newhost }}"
  remote_user: root
  become: false
  tasks:
```

6\. Now it's time to add the tasks, as well as the tag to this play. In
the tasks you need to make sure a user ansible exists and is a member of
the group wheel. Next, you use the shell module to set a password for
the user ansible. It's an ugly approach, but it works. In [Chapter
13](#ch13.xhtml#ch13), "[Managing Users](#ch13.xhtml#ch13)," you'll
learn how to do this in a much nicer way. As the next task you use the
lineinfile module to modify the /etc/sudoers file and allow members of
the group wheel to escalate permissions without entering a password. Add
the tasks to do this as follows:

``` pre1
- name: configure user ansible
  user:
    name: ansible
    groups: wheel
    append: yes
    state: present
- name: set a password for this user
  shell: ’echo password | passwd --stdin ansible’
- name: enable sudo without passwords
  lineinfile:
    path: /etc/sudoers
    regexp: ’^%wheel’
    line: ’%wheel ALL=(ALL)  NOPASSWD: ALL’
    validate: /usr/sbin/visudo -cf %s
```

7\. With this part you have set up the user ansible on the managed host,
but one element is still missing: the user cannot log in with an SSH
public/private key pair yet. In [Chapter 13](#ch13.xhtml#ch13) you'll
learn about a nice way to add the SSH public key to the remote user; for
now you can do it in a quick-and-dirty way that will also work. Add the
following lines to the playbook to conclude the second play:

``` pre1
- name: create SSH directory in user ansible home
  file:
    path: /home/ansible/.ssh
    state: directory
    owner: ansible
    group: ansible
- name: copy SSH public key to remote host
  copy:
    src: /home/ansible/.ssh/id_rsa.pub
    dest: /home/ansible/.ssh/authorized_keys
tags: setuphost
```

8\. Feel free to test the second play at this point; it's better to
filter out any errors now than to do it later. To test, use
**ansible-playbook -C -k exercise123.yaml -e newhost=ansible5 -e
newhostip=192.168.4.205**. Notice the use of the **-k** option, which
prompts for the SSH password that user root in this play needs to set up
the target host.

9\. At this point your RHEL host should be ready for use. The only thing
that is still missing is that it has not been registered against Red Hat
Subscription Manager. To do this, you need your Red Hat credentials.
Because these credentials are sensitive information, you should use
Ansible Vault. So let's start creating the vault file and define the
username and password variables in the Vault file. To create the Vault
file, use **ansible-vault create exercise123-vault.yaml** and provide
the following input, where you should use your real username and
password and not *XXXXXXXXX*. Your rhsm_user name is the name (typically
an email address) that you use to log in at redhat.com, and the
rhsm_password is the password that you use with it. Also notice that for
obvious security reasons, this file is NOT provided in the GitHub
repository that comes with this book:

``` pre1
rhsm_user: XXXXXXXXXXX
rhsm_password: XXXXXXXXXXX
```

10\. Now that you have created the Vault file, you can write the header
for the third and last play in the file exercise123.yaml. The most
important part of this header is the **vars_files** part, which refers
to the Vault file:

``` pre1
- name: use subscription manager to register and set up repos
  hosts: "{{ newhost }}"
  vars_files:
  - exercise123-vault.yaml
  tasks:
```

11\. At this point you can complete the playbook and add the remaining
tasks:

``` pre1
- name: register and subscribe {{ newhost }}
  redhat_subscription:
    username: "{{ rhsm_user }}"
    password: "{{ rhsm_password }}"
    state: present
- name: configure additional repo access
  rhsm_repository:
    name:
    - rh-gluster-3-client-for-rhel-8-x86_64-rpms
    - rhel-8-for-x86_64-appstream-debug-rpms
    state: present
tags: registerhost
```

12\. At this point the playbook is ready. Compare what you have written
to the sample playbook exercise123.yaml that is in the GitHub repository
and give it a try. To do so, use the **ansible-playbook -k
\--ask-vault-pass exercise123.yaml -e newhost=ansible5 -e
newhostip=192.168.4.205** command. Everything should be running
smoothly, but because this is a large playbook and it is very difficult
to write it without typos right from the beginning, you might have to do
a little bit of troubleshooting. To do so, I recommend that you use the
tags that have been provided with the plays. If, after running the first
and second plays successfully, the third play generates an error, you
can run just that play again, using **ansible-playbook
\--tags=registerhost exercise123.yaml -e newhost=ansible5**. (Notice
that this command doesn't use as many command-line options as the
command you used just a minute ago because these parameters don't apply
to the **registerhost** tag.)
:::


### Lab 12-1 

Configure the control.example.com host as a repository server, according
to the following requirements:

• Create a directory with the name /repo, and in that directory copy all
packages that have a name starting with nginx.

• Generate the metadata that makes this directory a repository.

• Configure the Apache web server to provide access to the repository
server. You just have to make sure that the DocumentRoot in Apache is
going to be set to the /repo directory.

### Lab 12-2 {#ch12.xhtml#h3_164 .h3}

Write a playbook to configure all managed servers according to the
following requirements:

• All hosts can access the repository that was created in Lab 12-1.

• Have the same playbook install the nginx package.

• Do NOT start the service. Use the appropriate module to gather
information about the installed nginx package, and let the playbook
print a message stating the name of the nginx package as well as the
version.
:::

[]{#ch13.xhtml}

::: {#ch13.xhtml#sbo-rt-content}