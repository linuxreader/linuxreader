+++
title = 'Ansible Facts'
description = 'Ansible Facts'
+++
An Ansible **fact** is a variable that contains information about a target system.This information can be used in conditional statements to tailor playbooks to that system. **Systems facts** are system property values. **Custom facts** are user-defined variables stored on managed hosts. 
system. 

Facts are collected when Ansible executes on the remote system. You'll see a "Gathering Facts" task  everytime you run a playbook. These facts are then stored in the variable **ansible_facts**.

Use the **debug** module to check the value of variables. This module requires variables to be enclosed in curly brackets. This example shows a large list of facts from managed nodes:
```yaml
---
- name: show facts
  hosts: all
  tasks:
  - name: show facts
    debug:
      var: ansible_facts
```

There are two supported formats for using Ansible fact variables:

It's recommended to use square brackets: `ansible_facts['default_ipv4']['address']` but dotted notation is also supported for now: `ansible_facts.default_ipv4.address`

Commonly used ansible_facts:

![](../../../images/Pasted%20image%2020250518052059.png)

There are additional Ansible modules for gathering more information. See `ansible-doc -l | grep fact

---

`package_facts` module collects information about software packages installed on managed hosts.


### Two ways facts are displayed

**Ansible_facts variable** (current way)
- All facts are stored in a dictionary with the name **ansible_facts**, and items in this dictionary are addressed using the notation with square brackets
- ie: `ansible_facts['distribution_version']`
- Recommended to use this. 

**injected variables** (old way)
- Variable are prefixed with the string ansible_  
- Will lose support eventually

- Old approach and the new approach both still occur. 
	- `ansible ansible1 -m setup` command Ansible facts are injected as variables.

```
    ansible1 | SUCCESS => {
        "ansible_facts": {
            "ansible_all_ipv4_addresses": [
                "192.168.122.1",
                "192.168.4.201"
            ],
            "ansible_all_ipv6_addresses": [
                "fe80::e564:5033:5dec:aead"
            ],
            "ansible_apparmor": {
```


Comparing ansible_facts Versus Injected Facts as Variables
```
ansible_facts                               Injected Variable
--------------------------------------------------------------
ansible_facts['hostname']                  ansible_hostname
ansible_facts['distribution']              ansible_distribution
ansible_facts['default_ipv4']['address']   ansible_default_ipv4['address']
ansible_facts['interfaces']                ansible_interfaces
ansible_facts['devices']                   ansible_devices
ansible_facts['devices']['sda']\
['partitions']['sda1']['size']             ansible_devices['sda']['partitions']['sda1']['size']
ansible_facts['distribution_version']      ansible_distribution_version
```


Different notations can be used in either method, the listings address the facts in dotted notation, not in the notation with square brackets.

Addressing Facts with Injected Variables:

```bash
    - hosts: all
      tasks:
      - name: show IP address
        debug:
          msg: >
            This host uses IP address {{ ansible_default_ipv4.address }}
```

Addressing Facts Using the ansible_facts Variable

```
    ---
    - hosts: all
      tasks:
      - name: show IP address
        debug:
          msg: >
            This host uses IP address {{ ansible_facts.default_ipv4.address }}
```


If, for some reason, you want the method where facts are injected into variables to be the default method, you can use
**inject_facts_as_vars=true** in the **\[default\]** section of the ansible.cfg file.

• In Ansible versions since 2.5, all facts are stored in one variable: **ansible_facts**. This method is used while gathering facts from a playbook.

• Before Ansible version 2.5, facts were injected into variables such as **ansible_hostname**. This method is used by the setup module. (Note that this may change in future versions of Ansible.)

• Facts can be addressed in dotted notation: 
`{{ansible_facts.default_ipv4.address }}`

• Alternatively, facts can be addressed in square brackets notation:
`{{ ansible_facts['default_ipv4']['address'] }}`. (preferred)

#### Managing Fact Gathering

By default, upon execution of each playbook, facts are gathered. This does slow down playbooks, and for that reason, it is possible to disable fact gathering completely. To do so, you can use the **gather_facts: no** parameter in the play header. If later in the same playbook it is necessary to gather facts, you can do this by running the setup module in a task.

Even if it is possible to disable fact gathering for all of your Ansible configuration, this practice is not recommended. Too many playbooks use conditionals that are based on the current value of facts, and all of these conditionals would stop working if fact gathering were disabled altogether.

As an alternative to make working with facts more efficient, you can disable a fact cache. To do so, you need to install an external plug-in.
Currently, two plug-ins are available for this purpose: **jsonfile** and **redis**. To configure fact caching using the redis plug-in, you need to
install it first. Next, you can enable fact caching through ansible.cfg.

The following procedure describes how to do this:

1\. Use **yum install redis**.

2\. Use **service redis start**.

3\. Use **pip install redis**.

4\. Edit /etc/ansible/ansible.cfg and ensure it contains the following
parameters:

``` pre1
[defaults]
gathering = smart
fact_caching = redis
fact_caching_timeout = 86400
```


**Note**

Fact caching can be convenient but should be used with caution. If, for instance, a playbook installs a certain package only if a sufficient
amount of disk space is available, it should not do this based on information that may be up to 24 hours old. For that reason, using a
fact cache is not recommended in many situations.

#### Custom Facts

- Used to provide a host with arbitrary values that Ansible can use to change the behavior of plays. 
- can be provided as static files. 
- files must 
	- be in either INI or JSON format, 
	- have the extension .fact, and 
	- on the managed hosts must be stored in the /etc/ansible/facts.d directory.

- can be generated by a script, and 
	- in that case the only requirement is that the script must generate its output in JSON format.

Dynamic custom facts are useful because they allow the facts to be determined at the moment that a script is running.  provides an example of a static custom fact file.

Custom Facts Sample File:
```
    [packages]
    web_package = httpd
    ftp_package = vsftpd
    
    [services]
    web_service = httpd
    ftp_service = vsftpd
```

To get the custom facts files on the managed hosts, you can use a playbook that copies a local custom fact file (existing in the current Ansible project directory) to the appropriate location on the managed hosts. Notice that this playbook uses variables, which are explained in more detail in the section titled "Working with Variables."

```
    ---
    - name: Install custom facts
      hosts: all
      vars:
        remote_dir: /etc/ansible/facts.d
        facts_file: listing68.fact
      tasks:
      - name: create remote directory
        file:
          state: directory
          recurse: yes
          path: "{{ remote_dir }}"
      - name: install new facts
        copy:
          src: "{{ facts_file }}"
          dest: "{{ remote_dir }}"
```

Custom facts are stored in the variable **ansible_facts.ansible_local**. 
In this variable, you use the filename of the custom fact file and the label in the custom fact file. For instance, after you run the playbook
in [Listing 6-9](#ch06.xhtml#list6_9), the web_package fact that was defined in listing68.fact is accessible as

    {{ ansible_facts[’ansible_local’][’listing67’][’packages’][’web_package’] }}

To verify, you can use the setup module with the filter argument. Notice that because the setup module produces injected variables as a result, the ad hoc command to use is `ansible all -m setup -a "filter=ansible_local"` . The command `ansible all -m setup -a  "filter=ansible_facts\['ansible_local'\]"` does not work.  

### Lab Working with Ansible Facts

1\. Create a custom fact file with the name custom.fact and the following contents:

``` pre1
[software]
package = httpd
service = httpd
state = started
enabled = true
```

2\. Write a playbook with the name copy_facts.yaml and the following contents:

``` pre1
---
- name: copy custom facts
  become: yes
  hosts: ansible1
  tasks:
  - name: create the custom facts directory
    file:
      state: directory
      recurse: yes
      path: /etc/ansible/facts.d
  - name: copy the custom facts
    copy:
      src: custom.fact
      dest: /etc/ansible/facts.d
```

3\. Apply the playbook using `ansible-playbook copy_facts.yaml -i inventory`

4\. Check the availability of the custom facts by using `ansible all -m setup -a "filter=ansible_local" -i inventory`

5\. Use an ad hoc command to ensure that the httpd service is not installed on any of the managed servers: `ansible all -m yum -a "name=httpd state=absent" -i inventory -b`

6\. Create a playbook with the name **setup_with_facts.yaml** that installs and enables the httpd service, using the custom facts:

``` pre1
---
- name: install and start the web service
  hosts: ansible1
  tasks:
  - name: install the package
    yum:
      name: "{{ ansible_facts['ansible_local']['custom']['software']['package'] }}"
      state: latest
  - name: start the service
    service:
      name: "{{ ansible_facts['ansible_local']['custom']['software']['service'] }}"
      state: "{{ ansible_facts['ansible_local']['custom']['software']['state'] }}"
      enabled: "{{ ansible_facts['ansible_local']['custom']['software']['enabled'] }}"

```

7\. Run the playbook to install and set up the service by using `ansible-playbook setup_with_facts.yaml -i inventory -b`

8\. Use an ad hoc command to verify the service is running: `ansible ansible1 -a "systemctl status httpd" -i inventory -b`

