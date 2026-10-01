# Ansible Vault

![](/images/ansible-vault.png)
  
  
Ansible Vault has been a great tool for keeping passwords and other sensitive data safe. Without sacrificing usability. 

It stores your passwords as values in variables. All in a password protected and encrypted file. After which, you can use the variables in your ansible scripts.

You can use the `ansible-vault` command to manage your vault.

## Managing Encrypted Files

### `ansible-vault create secret.yaml`
Creates an encrypted Vault named secret.yaml. When ran, you well be prompted to set a password for the Vault. Then, a blank file will be opened in your default editor. (Vim anyone?)

You can also store your vault password in a separate file. If you go this route, you'll want to make sure the file is in a secure location such as `/root` with limited permissions. 

For example, here's how you would create a vault and use a file called `vault-pass` as the Vault password file: 
 `ansible-vault create --vault-password-file=vault-pass secret.yaml`

For the above, the file `vault-pass` must exist and have a single line with the password you want to use for the Vault.
### Commonly used **ansible-vault** commands:
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

### Using Vault in Playbooks 

You can set your default vault password file under defaults in `ansible.cfg` like so:  
```
[defaults]
vault_password_file = ~/vault-pass
```

If you don't have it set, you can also choose to have ansible prompt you whenever it attempts to access your Vault with the option:  
`--vault-id @prompt` 

This also enables a playbook to work with multiple Vault-encrypted files with different passwords set. 

`ansible-playbook --ask-vault-pass` Can be used if you all your Vaults have the same password. And you want to be prompted for the password once. 

`ansible-playbook --vault-password-file=secret` can also be used at the command line to obtain the password from a file.

Here's an example where we use an api token in a task, but we want to keep that a secret. The Vault variable here is set as `my_api_token`:   
```yaml
  - name: Check if VM already exists across all clusters
    ansible.builtin.uri:
      url: "https://proxmox-datacenter-manager/api2/json/resources/list"
      method: GET
      headers:
        Authorization: "myAPIToken=admin@pam!ansible:{{ my_api_token }}"
      validate_certs: "{{ pve_validate_certs | default(false) }}"
      return_content: true
    register: all_cluster_vms
    delegate_to: localhost
```
### Managing Files with Sensitive Variables

Make sure to keep your encrypted and unencrypted [variables](https://www.linuxreader.com/ansible/variables/) separate. You can include your Vault in host or group variables or call it in a playbook using the `vars_files` parameter.

### Vault options for `ansible-playbook` command
Use `--help` and `grep` to quickly see Vault options:  
```bash
$ ansible-playbook --help | grep vault
                        [-e EXTRA_VARS] [--vault-id VAULT_IDS] [-J |
                        --vault-password-file VAULT_PASSWORD_FILES] [-f FORKS]
  --vault-id VAULT_IDS  the vault identity to use. This argument may be
  --vault-password-file, --vault-pass-file VAULT_PASSWORD_FILES
                        vault password file
  -J, --ask-vault-password, --ask-vault-pass
                        ask for vault password
```

### Vault options for ansible.cfg
Use the same strategy to quickly see Vault options to set globally in **ansible.cfg**:
```bash
$ ansible-config list | grep vault
  - This controls whether an Ansible playbook should prompt for a vault password.
  - key: ask_vault_pass
  name: Ask for the vault password(s)
  description: The vault_id to use for encrypting by default. If multiple vault_ids
    are provided, this specifies which to use for encryption. The ``--encrypt-vault-id``
  - key: vault_encrypt_identity
    key: defaults.vault_encrypt_identity
  description: The label to use for the default vault id label in cases where a vault
  - key: vault_identity
    key: defaults.vault_identity
  description: A list of vault-ids to use by default. Equivalent to multiple ``--vault-id``
  - key: vault_identity_list
  name: Default vault ids
    key: defaults.vault_identity_list
  description: If true, decrypting vaults with a vault id will only try the password
    from the matching vault-id.
  - key: vault_id_match
  name: Force vault id match
    key: defaults.vault_id_match
  - The vault password file to use. Equivalent to ``--vault-password-file`` or ``--vault-id``.
  - key: vault_password_file
    key: defaults.vault_password_file
  description: The salt to use for the vault encryption. If it is not provided, a
  - key: vault_encrypt_salt
    YAML or JSON or vaulted versions of these.
```

That should be enough to get you going. Somehow, I still see people not encrypting their password variables. Scary.

Feel free to [reach out](https://www.linuxreader.com/contact/) for any clarifications needed. :)