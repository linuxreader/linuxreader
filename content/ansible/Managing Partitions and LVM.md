+++
title = 'Managing Partitions and LVM'
description = 'Managing Partitions and LVM'
+++
### Managing Partitions and LVM

After detecting the disk device that needs to be used, you can move on and start creating partitions and logical volumes. 

- partition a disk using the parted module, 
- work with the lvg and lvol modules to manage LVM logical volumes, 
- create file systems using the filesystem module and mount them using the mount module
- manage swap storage.

#### Creating Partitions

**Parted Module**
name: 
- Assign unique name, required for GPT partitions
label: 
- type of partition table, msdos is default, gpt for gpt
device: 
- Device where you are creating partition
number: 
- partition number
state: 
- present or absent to add/remove

part_start: 
- Starting position expressed as an offset from the beginning of the disk
part_end: 
- Where to end the partition
If these arguments are not used, the partition starts at 0% and ends at 100% of
the available disk space. 

flags: 
- Set specific partition properties such as LVM partition type. 
- Required for LVM partition type

```yaml
      - name: create new partition
        parted:
          name: files
          label: gpt
          device: /dev/sdb
          number: 1
          state: present
          part_start: 1MiB
          part_end: 2GiB
      - name: create another new partition
        parted:
          name: swap
          label: gpt
          device: /dev/sdb
          number: 2
          state: present
          part_start: 2GiB
          part_end: 4GiB
          flags: [ lvm ]
```

#### Managing Volume Groups and LVM Logical Volumes

**lvg** module
- manage LVM logical volumes
- managing LVM volume groups

**lvol** module
 - managing LVM logical volumes. 
 
Creating an LVM volume group 
- **vg** argument to set the name of the volume group 
- **pvs** argument to identify the physical volume (which is often a partition or a disk device) on which the volume group needs to be created. 
- May need to specify the **pesize** to refer to the size of the physical extents.

```yaml
- name: create a volume group
  lvg:
    vg: vgdata
    pesize: "8"
    pvs: /dev/sdb1
```

After you create an LVM volume group, you can create LVM logical volumes. 


**lvol** Common Options:
lv
- Name of the LV
pvs
- comma separated list of pvs, if it is a partition then it should have the lvm option set
resizefs
- Indicates whether to resize filesystem when the lv is expanded
size
- size of the lv
snapshot
- specify name if this lv is a snapshot
vg
- VG is which the lv should be created

Creating an LVM Logical Volume

```yaml
- name: create a logical volume
    lvol:
      lv: lvdata
      size: 100%FREE
      vg: vgdata
```

#### Creating and Mounting File Systems

**filesystem** module 
- Supports creating as well as resizing file systems.

Options:
dev
- block device name
fstype
- filesystem type
opts
- options passed to mkfs command
resizefs
- Extends the filesystem if set to yes. Extended to the current block size

Creating an XFS File System

```yaml
- name: create an XFS filesystem
  filesystem:
    dev: /dev/vgdata/lvdata
    fstype: xfs
```

### Mounting a filesystem

**mount** module. 
- Used to mount a filesystem

Options:
fstype
-  Filesystem type is not automatically dedected. 
- Used to specify filesystem type
path
- directory to mount the filesystem to
src
- device to be mounted
state
- Current mount state
- mounted to mount device now
- present to set in /etc/fstab but not mount it now

```yaml
      - name: mount the filesystem
        mount:
          src: /dev/vgdata/lvdata
          fstype: xfs
          state: mounted
          path: /mydir
```

#### Configuring Swap Space

- To set up swap space, you first must format a device as swap space and next mount the swap space. 
- To format a device as swap space, you use the filesystem module. 
- There is no specific Ansible module to activate theswap space, so you use the command module to run the Linux **swapon** command.

- Because adding swap space is not always required, it can be done in a conditional statement. 
- In the statement, use the **ansible_swaptotal_mb** fact to discover how much swap is actually available. 
- If that amount falls below a specific threshold, the swap space can be created and activated. 

A conditional check is performed, and additional swap space is configured if the current
amount of swap space is lower than 256 MiB.

```yaml
    ---
    - name: configure swap storage
      hosts: ansible2
      tasks:
      - name: setup swap
        block:
        - name: make the swap filesystem
          filesystem:
            fstype: swap
            dev: /dev/sdb1
        - name: activate swap space
          command: swapon /dev/sdb1
        when: ansible_swaptotal_mb < 256
```


Run an ad hoc command to ensure that /dev/sdb on the target host is empty:
```bash
ansible ansible2 -a "dd if=/dev/zero of=/dev/sdb bs=1M count=10"
```

To make sure that you don't get any errors about partitions that are in use, also reboot the target host:
``` pre1
ansible ansible2 -m reboot
```



- Lack of idempotency if the size is specified as 100%FREE, which is a relative value, not an absolute value. 
- This value works the first time you run the playbook, but it does not the second time you run the playbook. 
- Because no free space is available, the LVM layer interprets the task as if you wanted to create a logical volume with a size of 0 MiB and will complain about that. To ensure that plays are written in an idempotent way, make sure that you use absolute values, not relative values.