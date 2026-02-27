+++
title = 'proxmox-storage role'
description = 'proxmox-storage role'
draft = true
+++
storage-dir1.yaml
```yaml
---
- block:
  - name: Detect secondary disk name
    ignore_errors: yes
    set_fact:
      disk2name: vda
    when: ansible_facts['devices']['vda'] is defined

  - name: Search for second disk, continue only if it is found
    assert:
      that:
        - disk2name is defined
      fail_msg: second hard disk not found

  - name: Debug detected disk
    debug:
      msg: "{{ disk2name }} was found. Moving forward."  
      
  - name: Create LVM and partitions
    block:
    - name: Create LVM Partition on second disk
      parted: 
        name: "{{ storage_dir1 | regex_replace('^/', '') | regex_replace('/', '_') }}"
        label: gpt
        device: /dev/{{ disk2name }}
        number: 1
        state: present
        flags: [ lvm ]

    - name: Create an LVM volume group
      lvg:
        vg: vg{{ storage_dir1 | regex_replace('^/', '') | regex_replace('/', '_') }}
        pvs: /dev/{{ disk2name }}1

    - name: Create lv
      lvol:
        lv: lv{{ storage_dir1 | regex_replace('^/', '') | regex_replace('/', '_') }}
        size: 100%FREE
        vg: vg{{ storage_dir1 | regex_replace('^/', '') | regex_replace('/', '_') }}

    - name: create filesystem
      filesystem:
        dev: /dev/vg{{ storage_dir1 | regex_replace('^/', '') | regex_replace('/', '_') }}/lv{{ storage_dir1 | regex_replace('^/', '') | regex_replace('/', '_') }}
        fstype: xfs

    when: ansible_facts['devices']['vda']['partitions'] is not defined


  - name: Create data directory
    file:
      dest: "{{ storage_dir1 }}"
      mode: 777
      state: directory
    
  - name: Mount the filesystem
    mount:
      src: /dev/vg{{ storage_dir1 | regex_replace('^/', '') | regex_replace('/', '_') }}/lv{{ storage_dir1 | regex_replace('^/', '') | regex_replace('/', '_') }}
      fstype: xfs
      state: present
      path: "{{ storage_dir1 }}"
  
  - name: Reset permissions on mounted filesystem
    file:
      path: "{{ storage_dir1 }}"
      state: directory
      mode: '0777'
when: storage_dir1 is defined
```


Variables:
```yaml
---
# Example:
# storage_dir1: /data
storage_dir1: []
storage_dir2: []
storage_dir3: []
storage_dir4: []
storage_dir5: []
storage_dir6: []

```

tasks/main.yaml
```yaml
---
import_tasks: storage-dir1.yaml
import_tasks: storage-dir2.yaml
import_tasks: storage-dir3.yaml
import_tasks: storage-dir4.yaml
import_tasks: storage-dir5.yaml
import_tasks: storage-dir6.yaml
```