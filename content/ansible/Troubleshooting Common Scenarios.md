+++
title = 'Troubleshooting Common Scenarios'
description = 'Troubleshooting Common Scenarios'
+++
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
