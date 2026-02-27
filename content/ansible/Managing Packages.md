+++
title = 'Managing Packages'
description = 'Managing Packages'
+++
### Using Modules to Manage Packages

Modules used to manage packages:
![Image](/Images/12tab02.jpg)

#### Configuring Repository Access

The **yum_repository** module lets you work with yum repository files in the /etc/yum.repos.d/ directory.

```yaml
    ---
    - name: setting up repository access
      hosts: all
      tasks:
      - name: connect to example repo
        yum_repository:
          name: example repo
          description: RHCE8 example repo
          file: examplerepo
          baseurl: ftp://control.example.com/repo/
          gpgcheck: no
```

yum_repository Key Arguments

![Image](/Images/12tab03.jpg)

Notice that use of the **gpgcheck** argument is recommended but not
mandatory. Most repositories are provided with a GPG key to verify that
packages in the repository have not been tampered with. However, if no
GPG key is set up for the repository, the **gpgcheck** parameter can be
set to no to skip checking the GPG key.

#### Managing Software with yum {.h4}

The yum module can be used to manage software packages. You use it to
install and remove packages or to update packages. This can be done for
individual packages, as well as package groups and modules. Let's look
at some examples that go beyond the mere installation or removal of
packages, which was covered sufficiently in earlier chapters.

[Listing 12-2](#ch12.xhtml#list12_2) shows a module that will update all
packages on this system.

**Listing 12-2** Using yum to Perform a System Update

::: pre_1
    ---
    - name: updating all packages
      hosts: ansible2
      tasks:
      - name: system update
        yum:
          name: ’*’
          state: latest
:::

Notice the use of the **name** argument to the yum module. It has
**'\*'** as its argument. To prevent the wildcard from being interpreted
by the shell, you must make sure it is placed between single quotes.

[Listing 12-3](#ch12.xhtml#list12_3) shows an example where yum package
groups are used to install the Virtualization Host package group.

**Listing 12-3** Installing Package Groups

::: pre_1
    ---
    - name: install or update a package group
      hosts: ansible2
      tasks:
      - name: install or update a package group
        yum:
          name: ’@Virtualization Host’
          state: latest
:::

When a yum package group instead of an individual package needs to be
installed, the name of the package group needs to start with an at sign
(@), and the entire package group name needs to be put between single
quotes. Also notice the use of **state: latest** in [Listing
12-3](#ch12.xhtml#list12_3). This line ensures that the packages in the
package group are installed if they have not been installed yet. If they
have already been installed, they are updated to the latest version.

A new feature in RHEL 8 is the yum AppStream module. Modules as listed
by the Linux **yum modules list** command can be managed with the
Ansible yum module also. Working with yum modules is similar to working
with yum package groups. In the example in [Listing
12-4](#ch12.xhtml#list12_4), the main difference is that a version
number and the installation profile are included in the module name.

**Listing 12-4** Installing AppStream Modules with the yum Module

::: pre_1
    ---
    - name: installing an AppStream module
      hosts: ansible2
      tasks:
      - name: install or update an AppStream module
        yum:
          name: ’@php:7.3/devel’
          state: present
:::

::: note

------------------------------------------------------------------------

**Note**

When using the yum module to install multiple packages, you can provide
the **name** argument with a list of multiple packages. Alternatively,
you can provide multiple packages in a loop. Of these solutions, using a
list of multiple packages as the argument to **name** is always
preferred. If multiple package names are provided in a loop, the module
must execute a task for every single package. If multiple package names
are provided as the argument to **name**, yum can install all these
packages in one single task.

------------------------------------------------------------------------
:::

#### Managing Package Facts {.h4}

When Ansible is gathering facts, package facts are not included. To
include package facts as well, you need to run a separate task; that is,
you need to use the package_facts module. Facts that have been gathered
about packages are stored to the ansible_facts.packages variable. The
sample playbook in [Listing 12-5](#ch12.xhtml#list12_5) shows how to use
the package_facts module.

**Listing 12-5** Using the package_facts Module to Show Package Details

::: pre_1
    ---
    - name: using package facts
      hosts: ansible2
      vars:
        my_package: nmap
      tasks:
      - name: install package
        yum:
          name: "{{ my_package }}"
          state: present
      - name: update package facts
        package_facts:
          manager: auto
      - name: show package facts for {{ my_package }}
        debug:
          var: ansible_facts.packages[my_package]
        when: my_package in ansible_facts.packages
:::

As you can see, the package_facts module does not need much to do its
work. The only argument used here is the **manager** argument, which
specifies which package manager to communicate to. Its default value of
**auto** automatically detects the appropriate package manager and uses
that. If you want, you can specify the package manager manually, using
any package manager such as yum or dnf. [Listing
12-6](#ch12.xhtml#list12_6) shows the output of running the [Listing
12-5](#ch12.xhtml#list12_5) playbook, where you can see details that are
collected by the package_facts module.

**Listing 12-6** Running **ansible-playbook listing125.yaml** Results

::: pre_1
    [ansible@control rhce8-book]$ ansible-playbook listing125.yaml
    
    PLAY [using package facts] **************************************************************
    
    TASK [Gathering Facts] ******************************************************************
    ok: [ansible2]
    
    TASK [install package] ******************************************************************
    ok: [ansible2]
    
    TASK [update package facts] *************************************************************
    ok: [ansible2]
    
    TASK [show package facts for my_package] ************************************************
    ok: [ansible2] => {
        "ansible_facts.packages[my_package]": [
            {
                "arch": "x86_64",
                "epoch": 2,
                "name": "nmap",
                "release": "5.el8",
                "source": "rpm",
                "version": "7.70"
            }
        ]
    }
    
    PLAY RECAP ******************************************************************************
    ansible2                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
:::

In [Exercise 12-1](#ch12.xhtml#exe12_1) you can practice working with
the different tools Ansible provides for module management.

::: box
**Exercise 12-1 Managing Software Packages**

1\. Use your editor to create a new file with the name exercise121.yaml.

2\. Write a play header that defines the variable **my_package** and
sets its value to **virt-manager**:

``` pre1
---
- name: exercise121
  hosts: ansible2
  vars:
    my_package: virt-manager
  tasks:
```

3\. Add a task that installs the package based on the name of the
variable that was provided:

``` pre1
- name: install package
  yum:
    name: "{{ my_package }}"
    state: present
```

4\. Add a task that gathers facts about installed packages:

``` pre1
- name: update package facts
  package_facts:
    manager: auto
```

5\. As the last part of this exercise, add a task that shows facts about
the package that you have just installed:

``` pre1
- name: show package facts for {{ my_package }}
  debug:
    var: ansible_facts.packages[my_package]
  when: my_package in ansible_facts.packages
```

6\. Run the playbook using **ansible-playbook exercise121.yaml** and
verify its output.
:::
