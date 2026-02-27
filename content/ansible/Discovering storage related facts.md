+++
title = 'Discovering storage related facts'
description = 'Discovering storage related facts'
+++

**Table 15-2** Modules for Managing Storage

![Image](/Images/15tab02.jpg)

To make sure that your playbook is applied to the right devices, you first need to find which devices are available on your managed system. 

After you find them, you can use conditionals to make sure that tasks are executed on the right devices.

#### Using Storage-Related Facts

Ansible_facts related to storage

ansible_devices
- Available storage and device info
ansible_device_links
- info on how to access storage and other device info
ansible_mounts
- Mount point info


`ansible ansible1 -m setup -a 'filter=ansible_devices'` 
- Find generic information about storage devices. 

- The **filter** argument to the setup module uses a shell-style wildcard to search for matching items and for that reason can search in the highest level facts, such as **ansible_devices**, but it is incapable of further specifying what is searched for. For that reason, in the **filter** argument to the setup module, you cannot use a construction like `ansible ansible1 -m setup -a "filter=ansible_devices.sda"` which is common when looking up the variable in conditional statements.

#### Using Storage-Related Facts in Conditional Statements

Assert module
- show an error message if a device does not exist and to perform a task if the device exists. 
- For an easier solution, you can also use a **when** statement to look for the existence of a device. 
- The advantage of using the assert module is that an error message can be printed if the condition is not met.

**Listing 15-2** Using assert to Run a Task Only If a Device Exists

```yml
    ---
    - name: search for /dev/sdb continue only if it is found
      hosts: all
      vars:
        disk_name: sdb
      tasks:
      - name: abort if second disk does not exist
        assert:
          that:
            - "ansible_facts['devices']['{{ disk_name }}'] is defined"
          fail_msg: second hard disk not found
      - debug:
          msg: "{{ disk_name }} was found, lets continue"
```

Write a playbook that finds out the name of the disk device and puts that in a variable that you can work with further on in the playbook. 

The **set_fact** argument comes in handy to do so. 

You can use it in combination with a **when** conditional statement to store a detected device name in a variable. 

Storing the Detected Disk Device Name in a Variable
```yaml
    ---
    - name: define variable according to diskname detected
      hosts: all
      tasks:
      - ignore_errors: yes
        set_fact:
          disk2name: sdb
        when: ansible_facts[’devices’][’sdb’]
```


```yaml
  - name: Detect secondary disk name
    ignore_errors: yes
    set_fact:
      disk2name: vda
    when: ansible_facts['devices']['vda'] is defined

  - name: Search for second disk, continue only if it is found
    assert:
      that:
        - "ansible_facts['devices'][disk2name] is defined"
      fail_msg: second hard disk not found

  - name: Debug detected disk
    debug:
      msg: "{{ disk2name }} was found. Moving forward."
~                                                      
```

Next, see [Managing Partitions and LVM](Managing%20Partitions%20and%20LVM.md)