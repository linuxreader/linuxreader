+++
title = 'Repositories and subscriptions'
description = 'Repositories and subscriptions'
+++
### Using Modules to Manage Repositories and Subscriptions

To work with software packages, you need to make sure that repositories
are accessible and subscriptions are available. In the previous section
you learned how to write a playbook that enables you to access an
existing repository. In this section you learn how to set up the server
part of a repository if that still needs to be done. Also, you learn how
to manage RHEL subscriptions using Ansible.

#### Setting Up Repositories {.h4}

Most managed systems access the default distributions that are provided
while installing the operating system. In some cases external
repositories might not be accessible. If that happens, you need to set
up a repository yourself. Before you can do that, however, it's
important to know what a repository is. A repository is a directory that
contains RPM files, as well as the repository metadata, which is an
index that allows the repository client to figure out which packages are
available in the repository.

Ansible does not provide a specific module to set up a repository. You
must use a number of modules instead. Exactly which modules are involved
depends on how you want to set up the repository. For instance, if you
want to set up an FTP-based repository on the Ansible control host, you
need to accomplish the following tasks:

• Install the FTP package.

• Start and enable the FTP server.

• Open the firewall for FTP traffic.

• Make sure the FTP shared repository directory is available.

• Download packages to the repository directory.

• Use the Linux **createrepo** command to generate the index that is
required in each repository.

The playbook in [Listing 12-7](#ch12.xhtml#list12_7) provides an example
of how this can be done.

**Listing 12-7** Setting Up an FTP-based Repository

::: pre_1
    - name: install FTP to export repo
      hosts: localhost
      tasks:
      - name: install FTP server
        yum:
          name:
          - vsftpd
          - createrepo_c
          state: latest
      - name: start FTP server
        service:
          name: vsftpd
          state: started
          enabled: yes
      - name: open firewall for FTP
        firewalld:
          service: ftp
          state: enabled
          permanent: yes
    
    - name: setup the repo directory
      hosts: localhost
      tasks:
      - name: make directory
        file:
          path: /var/ftp/repo
          state: directory
      - name: download packages
        yum:
          name: nmap
          download_only: yes
          download_dir: /var/ftp/repo
      - name: createrepo
        command: createrepo /var/ftp/repo
:::

The most significant tasks in setting up the repository are the
**download packages** and **createrepo** tasks. In the **download
packages** task, the yum module is used to download a single package. To
do so, the **download_only** argument is used to ensure that the package
is not installed but downloaded to a directory. When you use the
**download_only** argument, you also must specify where the package
needs to be installed. To do this, the task uses the **download_dir**
argument.

There is one disadvantage in using this approach to download the
package, though: it requires repository access. If repository access is
not available, the fetch module can be used instead to download a file
from a specific URL.

#### Managing GPG Keys {.h4}

To guarantee the integrity of packages, most repositories are set up
with a GPG key. This enables the client to verify that packages have not
been tampered with while transmitted between the repository server and
client. For that reason, if packages are installed from a repository
server on the Internet, you should always make sure that **gpgcheck:
yes** is set while using the yum_repository module.

However, if you want to make sure that a GPG check is performed, you
need to make sure the client knows where to fetch the repository key. To
help with that, you can use the rpm_key module. You can see how to do
this in [Listing 12-8](#ch12.xhtml#list12_8). Notice that the playbook
in this listing doesn't work because no GPG-protected repository is
available. Setting up GPG-protected repositories is complex and outside
the scope of the EX294 objectives, and for that reason is not covered
here.

**Listing 12-8** Using rpm_key to Fetch an RPM Key

::: pre_1
    - name: use rpm_key in repository access
      hosts: all
      tasks:
      - name: get the GPG public key
        rpm_key:
          key: ftp://control.example.com/repo/RPM-GPG-KEY
          state: present
      - name: set up the repository client
        yum_repository:
          file: myrepo
          name: myrepo
          description: example repo
          baseurl: ftp://control.example.com/repo
          enabled: yes
          gpgcheck: yes
          state: present
:::

#### Managing RHEL Subscriptions {.h4}

When you work with Red Hat Enterprise Linux, configuring repository
access using the method described before is not enough. Red Hat
Enterprise Linux works with subscriptions, and to be able to access
software that is provided through your subscription entitlement, you
need to set up managed systems to access these subscriptions.

::: note

------------------------------------------------------------------------

**Tip**

Free developer subscriptions are available for RHEL as well as Ansible.
Register yourself at <https://developers.redhat.com> and sign up for a
free subscription if you want to test the topics described in this
section on RHEL and you don't have a valid subscription yet.

------------------------------------------------------------------------
:::

To understand how to use the Ansible modules to register a RHEL system,
you need to understand how to use the Linux command-line utilities. When
you are managing subscriptions from the Linux command line, multiple
steps are involved.

1\. First, you use the **subscription-manager register** command to
provide your RHEL credentials. Use, for instance, **subscription-manager
register \--username=yourname \--password=yourpassword**.

2\. Next, you need to find out which pools are available in your
account. A pool is a collection of software channels available to your
account. Use **subscription-manager list \--available** for an overview.

3\. Now you can connect to a specific pool using **subscription-manager
attach \--pool=*poolID***. Note that if only one subscription pool is
available in your account, you don't have to provide the **\--pool**
argument.

4\. Next, you need to find out which additional repositories are
available to your account by using **subscription-manager repos
\--list**.

5\. To register to use additional repositories, you use
**subscription-manager repos \--enable "repos name"**. Your system then
has full access to its subscription and related repositories.

Two significant modules are provided by Ansible:

• **redhat_subscription:** This module enables you to perform
subscription and registration in one task.

• **rhsm_repository:** This module enables you to add subscription
manager repositories.

[Listing 12-9](#ch12.xhtml#list12_9) shows an example of a playbook that
uses these modules to fully register a new RHEL 8 machine and add a new
repository to the managed machine. Notice that this playbook is not
runnable as such because important additional information needs to be
provided. [Exercise 12-3](#ch12.xhtml#exe12_3), later in the section
titled "Implementing a Playbook to Manage Software," will guide you to a
scenario that shows how to use this code in production.

**Listing 12-9** Using Subscription Manager to Set Up Ansible

::: pre_1
    ---
    - name: use subscription manager to register and set up repos
      hosts: ansible5
      tasks:
      - name: register and subscribe ansible5
        redhat_subscription:
          username: bob@example.com
          password: verysecretpassword
          state: present
      - name: configure additional repo access
        rhsm_repository:
          name:
          - rh-gluster-3-client-for-rhel-8-x86_64-rpms
          - rhel-8-for-x86_64-appstream-debug-rpms
          state: present
:::

In the sample playbook in [Listing 12-9](#ch12.xhtml#list12_9), you can
see how the redhat_subscription and rhsm_repository modules are used.
Notice that redhat_subscription requires a password. In [Listing
12-9](#ch12.xhtml#list12_9) the username and password are provided as
clear-text values in the playbook. From a security perspective, this is
very bad practice. You should use Ansible Vault instead. [Exercise
12-3](#ch12.xhtml#exe12_3) will guide you through a setup where this is
done.

In [Exercise 12-2](#ch12.xhtml#exe12_2) you are guided through the
procedure of setting up your own repository and using it. This procedure
consists of two distinct parts. In the first part you set up a
repository server that is based on FTP. Because in Ansible you often
need to configure topics that don't have your primary attention, you set
up the FTP server and also change its configuration. Next, you write a
second playbook that configures the clients with appropriate repository
access, and after doing so, install a package.

::: box
**Exercise 12-2 Setting Up a Repository**

1\. Use your editor to create the file exercise122-server.yaml.

2\. Define the play that sets up the basic FTP configuration. Because
all its tasks should be familiar to you at this point, you can enter all
the tasks at once:

``` pre1
---
- name: install, configure, start and enable FTP
  hosts: localhost
  tasks:
  - name: install FTP server
    yum:
      name: vsftpd
      state: latest
  - name: allow anonymous access to FTP
    lineinfile:
      path: /etc/vsftpd/vsftpd.conf
      regexp: ’^anonymous_enable=NO’
      line: anonymous_enable=YES
  - name: start FTP server
    service:
      name: vsftpd
      state: started
      enabled: yes
  - name: open firewall for FTP
    firewalld:
      service: ftp
      state: enabled
      immediate: yes
      permanent: yes
```

3\. Set up a repository directory. Add the following play to the
playbook. Notice the use of the download packages task, which uses the
yum module to download a package without installing it. Also notice the
createrepo task, which creates the repository metadata that converts the
/var/ftp/repo directory into a repository.

``` pre1
- name: setup the repo directory
  hosts: localhost
  tasks:
  - name: make directory
    file:
      path: /var/ftp/repo
      state: directory
  - name: download packages
    yum:
      name: nmap
      download_only: yes
      download_dir: /var/ftp/repo
  - name: install createrepo package
    yum:
      name: createrepo_c
      state: latest
  - name: createrepo
    command: createrepo /var/ftp/repo
    notify:
    - restart_ftp
  handlers:
  - name: restart_ftp
    service:
      name: vsftpd
      state: restarted
```

4\. Use the command **ansible-playbook exercise122-server.yaml** to set
up the FTP server on control.example.com. If you haven't made any typos,
you shouldn't encounter any errors.

5\. Now that the repository server has been installed, it's time to set
up the repository client. Use your editor to create the file
exercise122-client.yaml and write the play header as follows:

``` pre1
---
- name: configure repository
  hosts: all
  vars:
    my_package: nmap
  tasks:
```

6\. Add a task that uses the yum_repository module to configure access
to the new repository:

``` pre1
- name: connect to example repo
  yum_repository:
    name: exercise122
    description: RHCE8 exercise 122 repo
    file: exercise122
    baseurl: ftp://control.example.com/repo/
    gpgcheck: no
```

7\. After setting up the repository client, you also need to make sure
that the clients know how to reach the repository server by addressing
its name. Add the next task that writes a new line to /etc/hosts to make
sure host name resolving on the clients is set up correctly:

``` pre1
- name: ensure control is resolvable
  lineinfile:
    path: /etc/hosts
    line: 192.168.4.200  control.example.com  control

- name: install package
  yum:
    name: "{{ my_package }}"
    state: present
```

8\. If you are using the package_facts module, you need to remember to
update it after installing new packages. Add the following task to get
this done:

``` pre1
- name: update package facts
  package_facts:
    manager: auto
```

9\. As the last task, just because it's fun, use the debug module
together with the package facts to get information about the newly
installed package:

``` pre1
- name: show package facts for {{ my_package }}
  debug:
    var: ansible_facts.packages[my_package]
  when: my_package in ansible_facts.packages
```

10\. Use the command **ansible-playbook exercise122-client.yaml -e
my_package=redis**. That's right; this command overwrites the my_package
variable that was set in the playbook---just to remind you a bit about
variable precedence.
:::
