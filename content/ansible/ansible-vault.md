---
title: 'Ansible Vault'
description: 'Ansible Vault'
showDate: false
---

### Ansible Vault

- For webkeys, passwords, and other types of sensitive data that you really shouldn't store as plain text in a playbook. 
- Can use Ansible Vault to encrypt and decrypt sensitive data to make it unreadable, and only while accessing data does it ask for a password so that it is decrypted.

1\. Sensitive data is stored as values in variables in a separate variable file.
2\. The variable file is encrypted, using the **ansible-vault** command.
3\. While accessing the variable file from a playbook, you enter a password to decrypt.

#### Managing Encrypted Files

`ansible-vault create secret.yaml`
- Ansible Vault prompts for a password and then opens the file using the default editor. 
- The password can be provided in a password file.(must be really well protected (for example, by putting it in the user root home directory))
- If a password file is used, the encrypted variable file can be created using `ansible-vault create \--vault-password-file=passfile secret.yaml`

`ansible-vault encrypt` 
- encrypt one or more existing files. 
- The encrypted file can next be used from a playbook, where a password needs to be entered to decrypt.

`ansible-vault decrypt` 
- used to decrypt the file. 

Commonly used **ansible-vault** commands:
`create`
- Creates new encrypted file
`encrypt`
- Encrypts an existing file
`encrypt_string`
- Encrypts a string
`decrypt`
- Decrypts an existing file
`rekey`
- Changes password on an existing file
`view`
- Shows contents of an existing file
`edit`
- Edits an existing encrypted file

#### Using Vault in Playbooks 

`--vault-id @prompt` 
- When a Vault-encrypted file is accessed from a playbook, a password must be entered. 
- Has the `ansible-playbook` command prompt for a password for each of the Vault-encrypted files that may be used
- Enables a playbook to work with multiple Vault-encrypted files where these files are allowed to have different passwords set. 

`ansible-playbook --ask-vault-pass`
- Used if all Vault-encrypted files a playbook refers to have the same password set.

`ansible-playbook --vault-password-file=secret` 
- Obtain the Vault password from a password file. 
- Password file should contain a string that is stored as a single line in the file. 
- Make sure the vault password file is protected through file permissions, such that it is not accessible by unauthorized users!

#### Managing Files with Sensitive Variables

- You should separate files containing unencrypted variables from files that contain encrypted variables. 
- Use **group_vars** and **host_vars** variable inclusion for this. 

- You may create a directory (instead of a file) with the name of the host or host group. 
- Within that directory you can create a file with the name **vars**, which contains unencrypted variables, and a file with the name **vault**, which contains Vault-encrypted variables. 
- Vault-encrypted variables can be included from a file using the `vars_files` parameter. 

### Lab:  Working with Ansible Vault

1\. Create a secret file containing encrypted values for a variable user and a variable password by using `ansible-vault create secrets.yaml`

Set the password to **password** and enter the following lines:
``` pre1
username: bob
pwhash: password
```

When creating users, you cannot provide the password in plain text; it needs to be provided as a hashed value. 
Because this exercise focuses on the use of Vault, the password is not provided as a hashed value, and as a result, a warning is displayed. You may ignore this warning.

2\. Create the file create-users.yaml and provide the following contents:

``` pre1
---
- name: create a user with vaulted variables
  hosts: ansible1
  vars_files:
    - secrets.yaml
  tasks:
  - name: creating user
    user:
      name: "{{ username }}"
      password: "{{ pwhash }}"
```

3\. Run the playbook by using `ansible-playbook --ask-vault-pass create-users.yaml` 

4\. Change the current password on **secrets.yaml** by using `ansible-vault rekey secrets.yaml` and set the new password to
**secretpassword**.

5\. To automate the process of entering the password, use `echo secretpassword > vault-pass`

6\. Use `chmod 400 vault-pass` to ensure the file is readable for the ansible user only; this is about as much as you can do to secure the file.

7\. Verify that it's working by using `ansible-playbook --vault-password-file=vault-pass create-users.yaml`
JunctionScallopPoise