+++
title = 'Deploying files'
description = 'Manipulating files, SELinux, and Jinja2 templates'
+++

This chapter covers the following subjects:

• Using Modules to Manipulate Files
• Managing SELinux Properties
• Using Jinja2 Templates

### RHCE exam topics

• Use Ansible modules for system administration tasks that work with:
• File contents
• Use advanced Ansible features
• Create and use templates to create customized configuration files

### Using Modules to Manipulate Files

#### File Module Manipulation Overview

Common modules to manipulate files
**copy**
- Copies files to remote locations
**fetch**
- Fetches files from remote locations
**file**
- Manage file and file properties
- Create new files or directories
- Create links
- Remove files
- Set permissions and ownership

**acl**
- Work with file system ACLs
**find**
- Find files based on properties
**lineinfile**
- Manages lines in text files
**blockinfile**
- Manage blocks in text files
**replace**
- Replaces strings in text files based on regex
**synchronize**
- Performs rsync-based synchronization tasks
**stat**
- Retrieves file or file system status
- enables you to retrieve file status information. 
- gets status information and is not used to change anything
- use it to check specific file and perform an action if the properties are not set as expected. 
Shows:
- which permission mode is set,
- whether it is a link, 
- which checksum is set on the file
- etc.
- See `ansible-doc stat` for list of full output
### Lab: View information about /etc/hosts file
```yml
- name: stat module tests
  hosts: ansible1
  tasks:
  - stat:
      path: /etc/hosts
    register: st
  - name: show current values
    debug:
      msg: current value of the st variable is {{ st }}

```
### Lab: write a message if the expected permission mode is not set.
```yml
---
- name: stat module test
  hosts: ansible1
  tasks:
  - command: touch /tmp/statfile
  - stat:
      path: /tmp/statfile
    register: st
  - name: show current values
    debug:
      msg: current value of the st variable is {{ st }}
  - fail:
      msg: "unexpected file mode, should be set to 0640"
    when: st.stat.mode != '0640'                               
```


### Lab:  Use the file Module to Correct File Properties Discovered with stat
```yml
---
- name: stat module tests
  hosts: ansible1
  tasks:
  - command: touch /tmp/statfile
  - stat:
      path: /tmp/statfile
    register: st
  - name: show current values
    debug:
      msg: current value of the st variable is {{ st }}
  - name: changing file permissions if that's needed
    file:
      path: /tmp/statfile
      mode: 0640
    when: st.stat.mode != '0640'
```


#### Managing File Contents 

Use lineinfile or blockinfile instead of copy to manage text in a file
### Lab: Change a string, based on a regular expression.

```yml
---
- name: configuring SSH
  hosts: all
  tasks:
  - name: disable root SSH login
    lineinfile:
      dest: /etc/ssh/sshd_config
      regexp: "^PermitRootLogin"
      line: "PermitRootLogin no"
    notify: restart sshd

  handlers:
  - name: restart sshd
    service:
      name: sshd
      state: restarted
```


### Lab: Manipulate multiple lines

```yml
---
- name: modifying file
  hosts: all
  tasks:
  - name: ensure /tmp/hosts exists
    file:
      path: /tmp/hosts
      state: touch
  - name: add some lines to /tmp/hosts
    blockinfile:
      path: /tmp/hosts
      block: |
        192.168.4.110 host1.example.com
        192.168.4.120 host2.example.com
      state: present
```


When blockinfile is used, the text specified in the block is copied with a start and end indicator. 
```bash
[ansible@ansible1 ~]$ cat /tmp/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.122.201 ansible1
192.168.122.202 ansible2
192.168.122.203 ansible3
# BEGIN ANSIBLE MANAGED BLOCK
192.168.4.110 host1.example.com
192.168.4.120 host2.example.com
# END ANSIBLE MANAGED BLOCK
```

### Lab: Creating and Removing Files

Use the file module to create a new directory and in that directory create an empty file, then remove the directory recursively. 

```yml
---
- name: using the file module
  hosts: ansible1
  tasks:
  - name: create directory
    file:
      path: /newdir
      owner: ansible
      group: ansible
      mode: 770
      state: directory
  - name: create file in that directory
    file:
      path: /newdir/newfile
      state: touch
  - name: show the new file
    stat:
      path: /newdir/newfile
    register: result
  - debug:
      msg: |
        This shows that newfile was created
        "{{ result }}"
  - name: removing everything again
    file:
      path: /newdir
      state: absent               
```

- **state: absent** recursively removes the directory. 

#### Moving Files Around

copy module 
copies a file from the Ansible control host to a managed machine.

fetch module 
enables you to do the opposite

synchronize module 
performs Linux rsync-like tasks, 
ensuring that a file from the control host is synchronized to a file with that name on the managed host. 

copy module always creates a new file, whereas the synchronize module updates a current existing file. 

### Lab: Moving a File Around with Ansible

```yml
---
- name: file copy modules
  hosts: all
  tasks:
  - name: copy file demo
    copy:
      src: /etc/hosts
      dest: /tmp/
  - name: add some lines to /tmp/hosts
    blockinfile:
      path: /tmp/hosts
      block: |
        192.168.4.110 host1.example.com
        192.168.4.120 host2.example.com
      state: present
  - name: verify file checksum
    stat:
      path: /tmp/hosts
      checksum_algorithm: md5
    register: result
  - debug:
      msg: "The checksum of /tmp/hosts is {{ result.stat.checksum }}"
  - name: fetch a file
    fetch:
      src: /tmp/hosts
      dest: /tmp/

```

- Ansible creates a subdirectory on the control node for each managed host in the dest directory and puts the file that fetch has copied from the remote host in that subdirectory:
```bash
/tmp/ansible1/tmp/hosts
/tmp/ansible2/tmp/hosts
```

### Lab: Managing Files with Ansible

1\. Create a file with the name exercise81.yaml and give it the following play header:
2\. Add a task that creates a new empty file:
3\. Use the stat module to check on the status of the new file:
4\. To see what the status module is doing, add a line that uses the debug module:
5\. Now that you understand which values are stored in newfile, you can add a conditional play that changes the current owner if not set correctly:
6\. Add a second play to the playbook that fetches a remote file:
7\. Now that you have fetched the file so that it is on the Ansible control machine, use blockinfile to edit it:
8\. In the final step, copy the modified file to ansible2 by including the following play:
9\. At this point you're ready to run the playbook. Type `ansible-playbook exercise81.yaml` to run it and observe the results.
10\. Type `ansible ansible2 -a "cat /tmp/motd"` to verify that the modified motd file was successfully copied to ansible2.

```yml
---
- name: testing file manipulation skills
  hosts: ansible1
  tasks:
    - name: create new file
      file:
        name: /tmp/newfile
        state: touch
    - name: check the status of the new file
      stat:
        path: /tmp/newfile
      register: newfile
    - name: for debugging only
      debug:
        msg: the current values for newfile are {{ newfile }}
    - name: change file owner if needed
      file:
        path: /tmp/newfile
        owner: ansible
      when: newfile.stat.pw_name != 'ansible'

- name: fetching a remote file
  hosts: ansible1
  tasks:
    - name: fetch file from remote machine
      fetch:
        src: /etc/motd
        dest: /tmp
        
- name: adding text to the text file that is now on localhost
  hosts: localhost
  tasks:
  - name: add a message
    blockinfile:
      path: /tmp/ansible1/etc/motd
      block: |
        welcome to this server
        for authorized users only
      state: present

- name: copy the modified file to ansible2
  hosts: ansible2
  tasks:
  - name: copy motd file
    copy:
      src: /tmp/ansible1/etc/motd
      dest: /tmp

```
