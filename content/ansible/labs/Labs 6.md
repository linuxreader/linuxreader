## LABS

### Lab: Configure a playbook that works with custom facts 

Requirements:
• Use the project directory chapter6.
• Create an inventory file where ansible1 is member of the host group named **file** and ansible2 is member of the host group named **lamp**.
• Create a custom facts file that contains a section named packages and set the following variables:

``` pre1
smb_package = samba
ftp_package = vsftpd
db_package = mariadb
web_package = httpd
firewall_package = firewalld
```

• Create another custom facts file that contains a section named services and set the following variables:

``` pre1
smb_service = smbd
ftp_service = vsftpd
db_service = mariadb
web_service = httpd
firewall_service = firewalld
```

• Create a playbook with the name **copy_facts.yaml** that copies these facts to all managed hosts. In this playbook 
- Define a variable **remote_dir** to specify the directory the fact files should be copied to. 
- Use the variable **fact_file** to copy the fact files to the appropriate directories.
- Run the playbook and verify whether it works.

### Lab 6-2 After copying over the facts files, create a playbook that uses the facts to set up the rest of the environment. 

Requirements:
• Use a variable inclusion file with the name **allvars.yaml** and set the following variables:

``` pre1
web_root = /var/www/html
ftp_root = /var/ftp
```

• Create a playbook that sets up the file services and the web services. Also ensure the playbook opens the firewalld firewall to provide access to these servers.

• Make sure the webservice provides access to a file index.html, which contains the text "Welcome to the Ansible Web Server."

• Run the playbook and use ad hoc commands to verify that the services have been started.
