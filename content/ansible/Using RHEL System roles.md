+++
title = 'Using RHEL System roles'
description = 'Using RHEL System roles'
+++
### Using RHEL System Roles

- Allows for a uniform approach while managing multiple RHEL versions
- Red Hat provides RHEL System Roles. 
- RHEL System Roles make managing different parts of the operating system easy.


RHEL System Roles:

**rhel-system-roles.kdump**
- Configures the kdump crash recovery service
**rhel system-roles.network**
- Configures network interfaces
**rhel system-roles.postfix**
- Configures hosts as a Mail Transfer Agent using Postfix
**rhel system-roles.selinux**
- Manages SELinux settings
**rhel system-roles.storage**
- Configures storage
**rhel system-roles.timesync**
- Configures time synchronization


#### Understanding RHEL System Roles

- RHEL System Roles are based on the community Linux System Roles
- Provide a uniform interface to make configuration tasks easier where significant differences may exist between versions of the managed operating system. 
- RHEL System Roles can be used to manage Red Hat Enterprise Linux 6.10 and later, as well as RHEL 7.4 and later, and all versions of RHEL 8. 
- Linux System Roles are not supported by RHEL technical support.

#### Installing RHEL System Roles
- To use RHEL System Roles, you need to install the **rhel-system-roles** package on the control node by using `sudo yum install rhel-system-roles`. 
- This package can be found in the RHEL 8 AppStream repository. 
- After installation, the roles are copied to the **/usr/share/ansible/roles** directory, a directory that is a default part of the Ansible **roles_path** setting. 
- If a modification to the **roles_path** setting has been made in ansible.cfg, the roles are applied to the first directory listed in the **roles_path**. 
- With the roles, some very useful documentation is installed also; you can find it in the **/usr/share/doc/rhel-system-roles** directory.

- To pass configuration to the RHEL System Roles, variables are important.
- In the documentation directory, you can find information about variables that are required and used by the role. 
- Some roles also contain a sample playbook that can be used as a blueprint when defining your own role.
- It's a good idea to use these as the basis for your own RHEL System Roles--based configuration. 
- The next two sections describe the SELinux and the TimeSync System Roles, which provide nice and easy-to-implement examples of how you can use the RHEL System Roles.

#### Using the RHEL SELinux System Role

- You learned earlier how to manage SELinux settings using task definitions in your own playbooks. 
- Using the RHEL SELinux System Role provides an easy-to-use alternative. 
- To use this role, start by looking at the documentation, which is in the /usr/share/doc/rhel-system-roles/selinux directory. 
- A good file to start with is the README.md file, which provides lists of all the ingredients that can be used.
 
- The SELinux System Role also comes with a sample playbook file. 
- The most important part of this file is the **vars:** section, which defines the variables that should be applied by SELinux.  

Variable Definition in the SELinux System Role:

```yml
    ---
    - hosts: all
      become: true
      become_method: sudo
      become_user: root
      vars:
        selinux_policy: targeted
        selinux_state: enforcing
        selinux_booleans:
          - { name: ’samba_enable_home_dirs’, state: ’on’ }
          - { name: ’ssh_sysadm_login’, state: ’on’, persistent: ’yes’ }
        selinux_fcontexts:
          - { target: ’/tmp/test_dir(/.*)?’, setype: ’user_home_dir_t’, ftype: ’d’ }
        selinux_restore_dirs:
          - /tmp/test_dir
        selinux_ports:
          - { ports: ’22100’, proto: ’tcp’, setype: ’ssh_port_t’, state: ’present’ }
        selinux_logins:
          - { login: ’sar-user’, seuser: ’staff_u’, serange: ’s0-s0:c0.c1023’, state: ’present’ }
```

SELinux Variables Overview

**selinux_policy**
- Policy to use, usually set to targeted
**selinux_state**
- SELinux state, as managed with setenforce
**selinux_booleans**
- List of Booleans that need to be set
**selinux_fcontext**
- List of file contexts that need to be set, including the target file or directory to which they should be applied.
**selinux_restore_dir**
- List of directories at which the Linux restorecon command needs to be executed to apply new context.
**selinux_ports**
- List of ports and SELinux port types
**selinux_logins**
- A list of SELinux user and roles that can be created

- Most of the time while configuring SELinux, you need to apply the correct state as well as file context. 
- To set the appropriate file context, you first need to define the **selinux_fcontext** variable.
- Next, you have to define **selinux_restore_dirs** also to ensure that the desired context is applied correctly. 

### Lab: Sets the **httpd_sys_content_t** context type to the **/web** directory.
- Sample doc is used and unnecessary lines are removed and the values of two variables have been set
- When you use the RHEL SELinux System Role, some changes require the managed host to be rebooted. 
- To take care of this, a block structure is used, where the System Role runs in the block. 
- When a change that requires a reboot is applied, the SELinux System Role sets the variable **selinux_reboot_required** and fails. 
- As a result, the rescue section in the playbook is executed. 
- This rescue section first makes sure that the playbook fails because of the **selinux_reboot_required** variable being set to true. 
- If that is the case, the reboot module is called to reboot the managed host. 
- While rebooting, playbook execution waits for the rebooted host to reappear, and when that happens, the RHEL SELinux System Role is called again to complete its work.
```yml
---
- hosts: ansible2
  vars:
    selinux_policy: targeted
    selinux_state: enforcing
    selinux_fcontexts:
      - { target: ’/web(/.*)?’, setype: ’httpd_sys_content_t’, ftype: ’d’ }
    selinux_restore_dirs:
      - /web
    
# prepare prerequisites which are used in this playbook
  tasks:
    - name: Creates directory
        file:
        path: /web
        state: directory
    - name: execute the role and catch errors
        block:
        - include_role:
            name: rhel-system-roles.selinux
        rescue:
            # Fail if failed for a different reason than selinux_reboot_required.
            - name: handle errors
              fail:
                msg: "role failed"
              when: not selinux_reboot_required
    
            - name: restart managed host
              shell: sleep 2 && shutdown -r now "Ansible updates triggered"
              async: 1
              poll: 0
              ignore_errors: true
    
            - name: wait for managed host to come back
              wait_for_connection:
                delay: 10
                timeout: 300
    
            - name: reapply the role
              include_role:
                name: rhel-system-roles.selinux
```

#### Using the RHEL TimeSync System Role

**timesync_ntp_servers** variable
- most important setting
- specifies attributes to indicate which time servers should be used. 

- The **hostname** attribute identifies the name of IP address of the time server. 
- The **iburst** option is used to enable or disable fast initial time synchronization using the **timesync_ntp_servers** variable.

- The System Role finds out which version of RHEL is used, and according to the currently used version, it either configures NTP or Chronyd.

### Lab: Using an RHEL System Role to Manage Time Synchronization

1\. Copy the sample timesync playbook to the current directory:
`cp /usr/share/doc/rhel-system-roles/timesync/example-single-pool-playbook.yml timesync.yaml`

2\. Add the target host, NTP hostname pool.ntp.org, and remove pool true in the file timesync.yaml:
```yaml
---
- name: Configure NTP
  hosts: "{{ host }}"
  vars:
    timesync_ntp_servers:
      - hostname: pool.ntp.org
        iburst: true
  roles:
    - rhel-system-roles.timesync

```

3\. Add the timezone module and the **timezone** variable to the playbook to set the timezone as well. The complete playbook should look like the following:
```yml
---
- hosts: ansible2
  vars:
    timesync_ntp_servers:
    - hostname: pool.ntp.org
      iburst: yes
    timezone: UTC
  roles:
  - rhel-system-roles.timesync
  tasks:
  - name: set timezone
    timezone:
      name: "{{ timezone }}"
```

4\. Use `ansible-playbook timesync.yaml` to run the playbook. Observe its output. Notice that some messages in red are shown, but these can safely be ignored.

5\. Use `ansible ansible2 -a "timedatectl show"` and notice that the **timezone** variable is set to UTC.
