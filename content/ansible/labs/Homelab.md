Configure passwordless ssh access to all servers
	- write an adhoc script to copy keys to other servers
- Storage has secondary disk attached
- Configure ansible user "davidt" with password for all servers.
	- With passwordless sudo
	- Add this to the ad hoc script
- Add roles path to ansible.cfg as ansible/roles
- Disable privillege escalation by default?
- Set ansible to manage 10 hosts at a time. 
- Set davidt as ansible user

VMS
- podman-01
- mariadb-01
- apache-01
- storage-01

Groups
- webserver
- containership
- storage
- database

- Make a playbook motd.yaml that makes /etc/motd display "Welcome to {{ service }} server"
- Make a playbook sshd.yaml that runs on all hosts and sets the ssh banner to /etc/motd, X11 Forwarding is disabled, and MaxAuthTries is set to 3
- Create Ansible vault, configure vault password file, make vault and password files utomatically used.
- Add {{ database_pass }} {{ user_password }} and {{ ansible_user_pass }} variables to vault.
- Add user list under /group_vars/all/user_list.yaml
```yaml
---
users:
  - username: alice
    uid: 1201
  - username: vincent
    uid: 1202
  - username: sandy
    uid: 2201
  - username: patrick
    uid: 2202
```
- Create playbook users.yaml
	- Users who's ID begins with 1 is created on webservers group VMs
	- Users Who's id begins with 2 created on the database group
	- user passwords should be used from the {{ user_password }} variable
	- All users added to Wheel group
	- Shell set to /bin/bash for all users
	- Account passwords should use the SHA512 hash format.
	- Each user should have an SSH key uploaded for passwordless ssh access to thier respective servers

- Create a playbook `regular_tasks.yml` that runs on servers in the **proxy** host group and does the following:
	- A root crontab record is created that runs every hour.
	- The cron job appends the file `/var/log/time.log` with the output from the **date** command.

- Create a playbook `repository.yml` that runs on servers in the database host group and does the following:
	- A YUM repository file is created.
	- The name of the repository is mysql80-community.
	- The description of the repository is “MySQL 8.0 YUM Repo”.
	- Repository baseurl is http://repo.mysql.com/yum/mysql-8.0-community/el/8/x86_64/.
	- Repository GPG key is at http://repo.mysql.com/RPM-GPG-KEY-mysql.
	- Repository GPG check is enabled.
	- Repository is enabled.

Create a role called **sample-mysql** and store it in `/home/automation/plays/roles`. The role should satisfy the following requirements:

1. A primary partition number 1 of size 800MB on device `/dev/sdb` is created.
2. An LVM volume group called `vg_database` is created that uses the primary partition created above.
3. An LVM logical volume called `lv_mysql` is created of size 512MB in the volume group `vg_database`.
4. An XFS filesystem on the logical volume `lv_mysql` is created.
5. Logical volume `lv_mysql` is permanently mounted on `/mnt/mysql_backups`.
6. **mysql-community-server** package is installed.
7. Firewall is configured to allow all incoming traffic on MySQL port TCP 3306.
8. MySQL root user password should be set from the variable **database_password** (see task #5).
9. MySQL server should be started and enabled on boot.
10. MySQL server configuration file is generated from the `my.cnf.j2` Jinja2 template with the following content:

[mysqld]
bind_address = {{ ansible_default_ipv4.address }}
skip_name_resolve
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

symbolic-links=0
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

Create a playbook `/home/automation/plays/mysql.yml` that uses the role and runs on hosts in the **database** host group.

### Task 10: Create and Work with Roles (Some More)

Create a role called **sample-apache** and store it in `/home/automation/plays/roles`. The role should satisfy the following requirements:

1. The **httpd**, **mod_ssl** and **php** packages are installed. Apache service is running and enabled on boot.
2. Firewall is configured to allow all incoming traffic on HTTP port TCP 80 and HTTPS port TCP 443.
3. Apache service should be restarted every time the file `/var/www/html/index.html` is modified.
4. A Jinja2 template file `index.html.j2` is used to create the file `/var/www/html/index.html` with the following content:

The address of the server is: IPV4ADDRESS

IPV4ADDRESS is the IP address of the managed node.

Create a playbook `/home/automation/plays/apache.yml` that uses the role and runs on hosts in the **webservers** host group.

### Task 11: Download Roles From Ansible Galaxy and Use Them

Use Ansible Galaxy to download and install **geerlingguy.haproxy** role in `/home/automation/plays/roles`.

Create a playbook `/home/automation/plays/haproxy.yml` that runs on servers in the **proxy** host group and does the following:

1. Use geerlingguy.haproxy role to load balance request between hosts in the **webservers** host group.
2. Use **roundrobin** load balancing method.
3. HAProxy backend servers should be configured for HTTP only (port 80).
4. Firewall is configured to allow all incoming traffic on port TCP 80.

If your playbook works, then doing “**curl http://ansible2.hl.local/**” should return output from the web server (see task #10). Running the command again should return output from the other web server.

### Task 12: Security

Create a playbook `/home/automation/plays/selinux.yml` that runs on hosts in the **webservers** host group and does the following:

1. Uses the selinux **RHEL system role**.
2. Enables **httpd_can_network_connect** SELinux boolean.
3. The change must survive system reboot.

### Task 13: Use Conditionals to Control Play Execution

Create a playbook `/home/automation/plays/sysctl.yml` that runs on all inventory hosts and does the following:

1. If a server has more than 2048MB of RAM, then parameter **vm.swappiness** is set to 10.
2. If a server has less than 2048MB of RAM, then the following error message is displayed:

**Server memory less than 2048MB**

### Task 14: Use Archiving

Create a playbook `/home/automation/plays/archive.yml` that runs on hosts in the **database** host group and does the following:

1. A file `/mnt/mysql_backups/database_list.txt` is created that contains the following line: dev,test,qa,prod.
2. A gzip archive of the file `/mnt/mysql_backups/database_list.txt` is created and stored in `/mnt/mysql_backups/archive.gz`.

### Task 15: Work with Ansible Facts

Create a playbook `/home/automation/plays/facts.yml` that runs on hosts in the **database** host group and does the following:

1. A custom Ansible fact **server_role=mysql** is created that can be retrieved from **ansible_local.custom.sample_exam** when using Ansible setup module.

### Task 16: Software Packages

Create a playbook `/home/automation/plays/packages.yml` that runs on all inventory hosts and does the following:

1. Installs **tcpdump** and **mailx** packages on hosts in the **proxy** host groups.
2. Installs **lsof** and **mailx** packages on hosts in the **database** host groups.

### Task 17: Services

Create a playbook `/home/automation/plays/target.yml` that runs on hosts in the **webservers** host group and does the following:

1. Sets the default boot target to **multi-user**.

### Task 18. Create and Use Templates to Create Customised Configuration Files

Create a playbook `/home/automation/plays/server_list.yml` that does the following:

1. Playbook uses a Jinja2 template `server_list.j2` to create a file `/etc/server_list.txt` on hosts in the **database** host group.
2. The file `/etc/server_list.txt` is owned by the **automation** user.
3. File permissions are set to **0600**.
4. SELinux file label should be set to **net_conf_t**.
5. The content of the file is a list of FQDNs of all inventory hosts.

After running the playbook, the content of the file `/etc/server_list.txt` should be the following:

ansible2.hl.local
ansible3.hl.local
ansible4.hl.local
ansible5.hl.local

Note: if the FQDN of any inventory host changes, re-running the playbook should update the file with the new values