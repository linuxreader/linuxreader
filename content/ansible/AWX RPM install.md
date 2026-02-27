https://awx.wiki/installation

https://github.com/MrMEEE/awx-rpm-v2

Error: fatal: [localhost]: FAILED! => {"msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host."}

Create a file `ansible/ansible.cfg` in your project directory (i.e. `ansible.cfg` in the `provisioning_path` on the target) with the following contents:

```
[defaults]
host_key_checking = false
```

Error:
```bash
TASK [awx-rpm : Install AWX-RPM] ******************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "failures": ["No package awx-rpm available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
```


