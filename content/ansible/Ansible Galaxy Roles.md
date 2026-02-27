+++
title = 'Ansible Galaxy Roles'
description = 'Ansible Galaxy Roles'
+++
### Using Ansible Galaxy Roles

- Ansible Galaxy is a public library of Ansible content and contains thousands of roles that have been provided by community members.

#### Working with Galaxy

The easiest way to work with Ansible Galaxy is to use the website at https://galaxy.ansible.com:

![Image](/Images/2436_09fig01.jpg)

![Image](/Images/2436_09fig02.jpg)

- Use the Search Feature to Search for Specific Packages

- In the result of any Search action, you see a list of collections as well as a list of roles. 
- An Ansible Galaxy collection is a distribution format for Ansible content. 
- It can contain roles, but also playbooks, modules, and plug-ins. 
- In most cases you just need the roles, not the collection: roles contain all that you include in the playbooks you're working with.

- Some important indicators are the number of times the role has been downloaded and the score of the role. 
- This information enables you to easily distinguish between commonly used roles and roles that are not used that often. 
- Also, you can use tags to make identifying Galaxy roles easier. 
- These tags provide more information about a role and make it  possible to search for roles in a more efficient way.

![Image](/Images/2436_09fig03.jpg)

- You can download roles directly from the Ansible Galaxy website
- You can also use the `ansible-galaxy` command

#### Using the `ansible-galaxy` Command

`ansible-galaxy search`
-  Find roles based on many different keywords and manage them. 
- Must provide a string as an argument. 
- Ansible searches for this string in the name and description of the roles. 

Useful Command-Line Options:
**--platforms**
- Operating system platform to search for
**--author**
- GitHub username of the author
**--galaxy-tags**
- Additional tags to filter by

`ansible-galaxy info
- Get more information about a role.

```bash
[ansible@control ansible-lab]$ ansible-galaxy info geerlingguy.docker

Role: geerlingguy.docker
        description: Docker for Linux.
        commit: 9115e969c1e57a1639160d9af3477f09734c94ac
        commit_message: Merge pull request #501 from adamus1red/adamus1red/alpine-compose

add compose package to Alpine specific variables
        created: 2023-05-08T20:49:45.679874Z
        download_count: 23592264
        github_branch: master
        github_repo: ansible-role-docker
        github_user: geerlingguy
        id: 10923
        imported: 2025-03-24T00:01:45.901567
        modified: 2025-03-24T00:01:47.840887Z
        path: ('/home/ansible/.ansible/roles', '/usr/share/ansible/roles', '/etc/ansible/roles')
        upstream_id: None
        username: geerlingguy
```

#### Managing Ansible Galaxy Roles 

`ansible-galaxy install`
- Install a role 
- normally installs the role into the \~/.ansible/roles directory because this role is specified in the **role_path** setting in ansible.cfg. 
- If you want roles to be installed in another directory, consider changing this parameter. 
`-p`
- option to install the role to a different role path directory.

**requirements** file. 
- YAML file that you can include when using the `ansible-roles` command. 
```yml
    - src: geerlingguy.nginx
      version: "2.7.0"
```

- possible to add roles from sources other than Ansible Galaxy, such as a Git repository or a tarball. 
- In that case, you must specify the exact URL to the role using the `src` option. 
- When you are installing roles from a Git repository, the `scm` keyword is also required and must be set to `git`.

To install a role using the requirements file, you can use the `-r` option with the `ansible-galaxy install` command:
`ansible-galaxy install -r roles/requirements.yml`

`ansible-galaxy list`
- Get a list of currently installed roles

`ansible-galaxy remove` 
- Remove roles from your system. 

### LAB: Using `ansible-galaxy` to Manage Roles

- Type `ansible-galaxy search --author geerlingguy --platforms EL` to see a list of roles that geerlingguy has created.
- Make the command more specific and type `ansible-galaxy search nginx --author geerlingguy --platforms EL` to find the **geerlingguy.nginx** role.
- Request more information about this role by using `ansible-galaxy info geerlingguy.nginx`.
- Create a requirements file with the name **listing96.yaml** and give this file the following contents:

```yml
- src: geerlingguy.nginx
  version: "2.7.0"
```

- Add the line `roles_path = /home/ansible/roles` to the ansible.cfg file.

- Use the command `ansible-galaxy install -r listing96.yaml` to install the role from the requirements file. It is possible that by the time you run this exercise, the specified version 2.7.0 is no longer available. If that is the case, use `ansible-galaxy info` again to find a version that still is available, and change the requirements file accordingly.

- Type `ansible-galaxy list` to verify that the new role was successfully installed on your system.

- Write a playbook with the name exercise92.yaml that uses the role and has the following contents:

```yml
---
- name: install nginx using Galaxy role
  hosts: ansible2
  roles:
  - geerlingguy.nginx
```

- Run the playbook using `ansible-playbook exercise92.yaml` and observe that the new role is installed from the custom roles path.


