---
title: Crash and Burn
date: 2026-02-11
series: ["45 Days to RHCE"]
series_order: 3
draft: false
---

## Tasks 1-2: Configuration and adhoc commands
These tasks are all about setting up inventory, ansible.cfg, and access to hosts. There were a few things I needed to search documentation for.
### Inventory
I ended up opening the [official inventory page](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html#intro-inventory) to make sure this was done right.

### Bootstrap
Here is the adhoc script I came up with for task 2. 
```bash
#!/bin/bash
ansible ansible2 --become-user root -m user -K --ask-become-pass --become -a "name=automation state=present groups=wheel"
ansible ansible3 --become-user root -m user -K --ask-become-pass --become -a "name=automation state=present groups=wheel"
ansible ansible4 --become-user root -m user -K --ask-become-pass --become -a "name=automation state=present groups=wheel"
ansible ansible5 --become-user root -m user -K --ask-become-pass --become -a "name=automation state=present groups=wheel"

ansible all:!localhost -m file -K --ask-become-pass --become  --become-user root -a "path=/home/automation/.ssh state=directory mode='0755'"

ansible all:!localhost -m file -K --ask-become-pass --become  --become-user root -a "path=/home/automation/.ssh state=directory mode='0755'"

ansible all:!localhost -m lineinfile -K --ask-become-pass --become  --become-user root -a "path=/home/automation/.ssh/authorized_keys state=present create=true line='ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCucgpRJSz8pNX3MgjAdRLJA3FHmrNconvssiO0sgtC6nvgO4PTcVQYdBTHeATPJkXRTHdn8GKZnDx7fRrI4WqbqztmtYRPk24QZJ2AZUgoBHsYwge+aNFFKfcEdY2D9dIQQQZl8GpmTnlcSzkbB0bAwaG0ezmmSr63V0nPh62ITQ/ipIy7IMJNuKc9pzus/FhTVI6J6RVbe7u6go4PTsyIAYQGqtvmV0c7g4s6tYuriBwQkeQYj38BopxAak9jCrs2rUm5wIwsA4sIpI3zj4/eHXGH19tklVZsuJgpXbV8F+eJVHCwj9sTCMYFasElTfNB6cwcgjV+DbOMLhaa9kDUa8p8xoXshDzK6P0ACd5UD9ZNYbfaD9M0xcHC8YtmPmaHwMrfnbw6ki91Z3AMGSolY4lY8SP7wkgBpwOKZqwOfDBdGCFYd002zeKwpFeSxWUPNpnXfYZ4fGufWAxpMX0i8h0ia91kVIlkzhdEB3sZkG1L80roBFRSKvm8TOGswX0='"

ansible all:!localhost -m lineinfile -K --ask-become-pass --become  --become-user root -a "path=/etc/sudoers state=present regexp='^%wheel' line='%wheel ALL=(ALL) NOPASSWD: ALL'"

```

## Task 3: File content
This task was pretty straightforward. 
```yaml
---
- name: motd message for Proxy Servers
  become: yes
  hosts: proxy
  tasks:
  - name: Add text to motd proxy servers
    lineinfile:
      path: /etc/motd
      line: "Welcome to HAProxy server"

- name: motd message for web Servers
  become: yes
  hosts: webservers
  tasks:
  - name: Add text to motd webserver
    lineinfile:
      path: /etc/motd
      line: "Welcome to Apache server"
 
- name: motd message for database Servers
  become: yes
  hosts: database
  tasks:
  - name: Add text to motd database servers
    lineinfile:
      path: /etc/motd
      line: "Welcome to MySQL server"
```

## Task 4: Configure sshd
Another easy one. Just open up the sshd_config file on the target host to see what to regex.
```yaml
[automation@ansible-control plays]$ cat sshd.yml 
---
- name: configure sshd
  become: yes
  hosts: all
  tasks:
  - name: Set banner
    lineinfile:
      path: /etc/ssh/sshd_config
      state: present
      line: 'Banner /etc/motd'
    notify: restart sshd

  - name: Disable X11
    lineinfile:
      path: /etc/ssh/sshd_config
      state: present
      regexp: '^#X11Forwarding'
      line: 'X11Forwarding no'
    notify: restart sshd

  - name: Max Auth
    lineinfile:
      path: /etc/ssh/sshd_config
      state: present
      regexp: '^#MaxAuthTries'
      line: 'MaxAuthTries 3'
    notify: restart sshd

  handlers:
  - name: restart sshd
    service:
      name: sshd
      state: restarted
```

## Task 5: Ansible vault
I added the user password as plain text at first. Then remembered later this needed to be stored as an MD5 hash. Luckily, the `ansible-doc user` document had a link to directions for generating the hashed password.

## Task 6: Users and groups
This is when I first had to leave official documentation for help. The answer actually existed in the documentation but I didn't know what I was searching for. In this task, you need a way to match user id's based on the number that the id begins with. 

I spent a full hour on this task before I ran for help..

The **regex_search** function is how I solved the problem:
```yaml
[automation@ansible-control plays]$ cat users.yml
---
- name: webserver users
  vars_files:
    - secret.yml
    - /home/automation/plays/vars/user_list.yml
  hosts: webservers
  become: yes
  tasks:
  - name: add uid beginning with 1 to webservers
    user: 
      name: "{{ item.username }}"
      state: present
      groups: "wheel"
      uid: "{{ item.uid }}"
      password: " {{ user_password }}"
      shell: /bin/bash
    loop: "{{ users }}"
    when: item.uid | string | regex_search('^1')

  - name: Make ssh directories
    file: 
      path: /home/{{ item.username }}/.ssh
      state: directory
      mode: '0755'
    loop: "{{ users }}"
    when: item.uid | string | regex_search('^1')

  - name: add public key
    lineinfile:
      path: /home/{{ item.username }}/.ssh/authorized_keys
      create: true
      line: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCucgpRJSz8pNX3MgjAdRLJA3FHmrNconvssiO0sgtC6nvgO4PTcVQYdBTHeATPJkXRTHdn8GKZnDx7fRrI4WqbqztmtYRPk24QZJ2AZUgoBHsYwge+aNFFKfcEdY2D9dIQQQZl8GpmTnlcSzkbB0bAwaG0ezmmSr63V0nPh62ITQ/ipIy7IMJNuKc9pzus/FhTVI6J6RVbe7u6go4PTsyIAYQGqtvmV0c7g4s6tYuriBwQkeQYj38BopxAak9jCrs2rUm5wIwsA4sIpI3zj4/eHXGH19tklVZsuJgpXbV8F+eJVHCwj9sTCMYFasElTfNB6cwcgjV+DbOMLhaa9kDUa8p8xoXshDzK6P0ACd5UD9ZNYbfaD9M0xcHC8YtmPmaHwMrfnbw6ki91Z3AMGSolY4lY8SP7wkgBpwOKZqwOfDBdGCFYd002zeKwpFeSxWUPNpnXfYZ4fGufWAxpMX0i8h0ia91kVIlkzhdEB3sZkG1L80roBFRSKvm8TOGswX0= 
      state: present
    loop: "{{ users }}"
    when: item.uid | string | regex_search('^1')

- name: database users
  vars_files:
    - secret.yml
    - /home/automation/plays/vars/user_list.yml
  hosts: database
  become: yes
  tasks:
  - name: add uid beginning with 2 to database
    user:
      name: "{{ item.username }}"
      state: present
      groups: "wheel"
      uid: "{{ item.uid }}"
      password: "{{ user_password }}"
      shell: /bin/bash
    loop: "{{ users }}"
    when: item.uid | string | regex_search('^2')
  
  - name: Make ssh directories
    file: 
      path: /home/{{ item.username }}/.ssh
      state: directory
      mode: '0755'
    loop: "{{ users }}"
    when: item.uid | string | regex_search('^2')

  - name: add public key
    lineinfile:
      path: /home/{{ item.username }}/.ssh/authorized_keys
      create: true
      line: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCucgpRJSz8pNX3MgjAdRLJA3FHmrNconvssiO0sgtC6nvgO4PTcVQYdBTHeATPJkXRTHdn8GKZnDx7fRrI4WqbqztmtYRPk24QZJ2AZUgoBHsYwge+aNFFKfcEdY2D9dIQQQZl8GpmTnlcSzkbB0bAwaG0ezmmSr63V0nPh62ITQ/ipIy7IMJNuKc9pzus/FhTVI6J6RVbe7u6go4PTsyIAYQGqtvmV0c7g4s6tYuriBwQkeQYj38BopxAak9jCrs2rUm5wIwsA4sIpI3zj4/eHXGH19tklVZsuJgpXbV8F+eJVHCwj9sTCMYFasElTfNB6cwcgjV+DbOMLhaa9kDUa8p8xoXshDzK6P0ACd5UD9ZNYbfaD9M0xcHC8YtmPmaHwMrfnbw6ki91Z3AMGSolY4lY8SP7wkgBpwOKZqwOfDBdGCFYd002zeKwpFeSxWUPNpnXfYZ4fGufWAxpMX0i8h0ia91kVIlkzhdEB3sZkG1L80roBFRSKvm8TOGswX0=
      state: present
    loop: "{{ users }}"
    when: item.uid | string | regex_search('^2')
```

## Task 7: Scheduled tasks
Another easy one! The cron module documentation is great here.
```yaml
[automation@ansible-control plays]$ cat regular_tasks.yml 
---
- name: cron for proxy servers
  hosts: proxy
  become: yes
  tasks:
  - name: append date to log
    cron:
      name: "time"
      minute: 0
      job: "date >> /var/log/time.log"
      user: root
```


## Task 8: Software repositories
This task is dated and later tasks will fail because of it. This shouldn’t be a problem during the actual exam. Just follow examples in `ansible-doc yum_repository`. 

Ended up with this:
```yaml
[automation@ansible-control plays]$ cat repository.yml 
---
- name: software repositories
  hosts: database
  become: yes
  tasks:
  - name: mysql-80 repo
    ansible.builtin.yum_repository:
      name: mysql80-community
      description: "MySQL 8.0 YUM Repo"
      baseurl: http://repo.mysql.com/yum/mysql-8.0-community/el/8/x86_64/
      gpgkey: http://repo.mysql.com/RPM-GPG-KEY-mysql
      gpgcheck: true
      state: present
      enabled: yes
```

