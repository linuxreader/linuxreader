Write a playbook that meets the following requirements. Use multiple
plays in a way that makes sense.

• Write a first play that installs the httpd and mod_ssl packages on host ansible1.
• Use variable inclusion to define the package names in a separate file.
• Use a conditional to loop over the list of packages to be installed.
• Install the packages only if the current operating system is CentOS or
RedHat (but not Fedora) version 8.0 or later. If that is not the case,
the playbook should fail with the error message "Host *hostname* does
not meet minimal requirements," where *hostname* is replaced with the
current host name.
• On the Ansible control host, create a file /tmp/index.html. This file
must have the contents "welcome to my webserver".
• If the file /tmp/index.html is successfully copied to /var/www/html,
the web server process must be restarted. If copying the package fails,
the playbook should show an error message.
• The firewall must be opened for the http as well as the https
services.

packages
```yml
web:
  - httpd
  - mod_ssl
```

web.yaml
```yml
---
- name: set up a webserver
  vars_files: packages
  hosts: ansible1
  become: yes
  tasks:
    - name: block
      block:
      - name: install web packages
        yum:
          name: "{{ item }}"
          state: present
        loop: "{{ web }}"
        when: > 
          ( ansible_facts['distribution'] == "AlmaLinux" ) 
          or 
          ( ansible_facts['distribution'] == "RedHat" )
          and 
          ( ansible_facts['distribution_version'] > "8" ) 
          and 
          (ansible_facts['distribution'] != "Fedora" )
      rescue:
        - name: distribution is invalid
          debug:
            msg: Host {{ ansible_facts['hostname'] }} does not meet minimal requirements"

    - name: copy file to webserver
      copy:
        src: /tmp/index.html
        dest: /var/www/html/
      notify: restart_httpd

    - name: open http
      ansible.posix.firewalld:
        service: http
        state: enabled
        permanent: true
        immediate: true

    - name: open https
      ansible.posix.firewalld:
        service: https
        state: enabled
        permanent: true
        immediate: true

  handlers:
    - name: restart_httpd 
      service:
        name: httpd
        state: restarted
```