+++
title = 'Building an Ansible lab with Ansible'
description = 'Building an Ansible lab with Ansible using Ansible and Libvirt'
+++

When I started studying for RHCE, the study guide had me manually set up virtual machines for the Ansible lab environment. I thought.. Why not start my automation journey right, and automate them using Vagrant. 

I use Libvirt to manage KVM/QEMU Virtual Machines and the Virt-Manager app to set them up. I figured I could use Vagrant to automatically build this lab from a file. And I got part of the way. I ended up with this Vagrant file:

```bash
Vagrant.configure("2") do |config|
  config.vm.box = "almalinux/9"

  config.vm.provider :libvirt do |libvirt|
    libvirt.uri = "qemu:///system"
    libvirt.cpus = 2
    libvirt.memory = 2048
  end

   config.vm.define "control" do |control|
    control.vm.network "private_network", ip: "192.168.124.200"
    control.vm.hostname = "control.example.com"
  end

  config.vm.define "ansible1" do |ansible1|
    ansible1.vm.network "private_network", ip: "192.168.124.201"
    ansible1.vm.hostname = "ansible1.example.com"

  end

  config.vm.define "ansible2" do |ansible2|
    ansible2.vm.network "private_network", ip: "192.168.124.202"
    ansible2.vm.hostname = "ansible2.example.com"
  end

end
```

I could run this Vagrant file and Build and destroy the lab in seconds. But there was a problem. The Libvirt plugin, or Vagrant itself, I'm not sure which, kept me from doing a couple important things. 

First, I could not specify the initial disk creation size. I could add additional disks of varying sizes but, if I wanted to change the size of the first disk, I would have to go back in after the fact and expand it manually...

Second, the Libvirt plugin networking settings were a bit confusing. When you add the private network option as seen in the Vagrant file, it would add this as a secondary connection, and route everything through a different public connection. 

Now I couldn't get the VMs to run using the public connection for whatever reason, and it seems the only workaround was to make DHCP reservations for the guests Mac addresses which gave me even more problems to solve. But I won't go there..

So why not get my feet wet and learn how to deploy VMs with Ansible? This way, I would get the granularity and control that Ansible gives me, some extra practice with Ansible, and not having to use software that has just enough abstraction to get in the way. 

The guide I followed to set this up can be found on Redhat's blog [here.](https://www.redhat.com/en/blog/build-VM-fast-ansible) And it was pretty easy to set up all things considered. 

I'll rehash the steps here:
1. Download a cloud image
2. Customize the image
3. Install and start a VM
4. Access the VM

### Creating the role

Move to roles directory
`cd roles`

Initialize the role
`ansible-galaxy role init kvm_provision`

Switch into the role directory
`cd kvm_provision/`

Remove unused directories
`rm -r files handlers vars`

### Define variables

Add default variables to main.yml
`cd defaults/ && vim main.yml`

```yaml
---
# defaults file for kvm_provision
base_image_name: AlmaLinux-9-GenericCloud-9.5-20241120.x86_64.qcow2
base_image_url: https://repo.almalinux.org/almalinux/9/cloud/x86_64/images/{{ base_image_name }}
base_image_sha: abddf01589d46c841f718cec239392924a03b34c4fe84929af5d543c50e37e37
libvirt_pool_dir: "/var/lib/libvirt/images"
vm_name: f34-dev
vm_vcpus: 2
vm_ram_mb: 2048
vm_net: default
vm_root_pass: test123
cleanup_tmp: no
ssh_key: /root/.ssh/id_rsa.pub
# Added option to configure ip address
ip_addr: 192.168.124.250
gw_addr: 192.168.124.1
# Added option to configure disk size
vm_disksize: 20

```

### Defining a VM template

The community.libvirt.virt module is used to provision a KVM VM. This module uses a VM definition in XML format with libvirt syntax. You can dump a VM definition of a current VM and then convert it to a template from there. Or you can just use this:

`cd templates/ && vim vm-template.xml.j2`

```xml
<domain type='kvm'>
  <name>{{ vm_name }}</name>
  <memory unit='MiB'>{{ vm_ram_mb }}</memory>
  <vcpu placement='static'>{{ vm_vcpus }}</vcpu>
  <os>
    <type arch='x86_64' machine='pc-q35-5.2'>hvm</type>
    <boot dev='hd'/>
  </os>
  <cpu mode='host-model' check='none'/>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='{{ libvirt_pool_dir }}/{{ vm_name }}.qcow2'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x05' slot='0x00' function='0x0'/>
       <!-- Added: Specify the disk size using a variable -->
      <size unit='GiB'>{{ disk_size }}</size>
    </disk>
    <interface type='network'>
      <source network='{{ vm_net }}'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
    </interface>
    <channel type='unix'>
      <target type='virtio' name='org.qemu.guest_agent.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
    </channel>
    <channel type='spicevmc'>
      <target type='virtio' name='com.redhat.spice.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='2'/>
    </channel>
    <input type='tablet' bus='usb'>
      <address type='usb' bus='0' port='1'/>
    </input>
    <input type='mouse' bus='ps2'/>
    <input type='keyboard' bus='ps2'/>
    <graphics type='spice' autoport='yes'>
      <listen type='address'/>
      <image compression='off'/>
    </graphics>
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1' primary='yes'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x0'/>
    </video>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x06' slot='0x00' function='0x0'/>
    </memballoon>
    <rng model='virtio'>
      <backend model='random'>/dev/urandom</backend>
      <address type='pci' domain='0x0000' bus='0x07' slot='0x00' function='0x0'/>
    </rng>
  </devices>
</domain>
```

The template uses some of the variables from earlier. This allows flexibility to changes things by just changing the variables.

## Define tasks for the role to perform

`cd ../tasks/ && vim main.yml`

```yaml
---
# tasks file for kvm_provision

# ensure the required package dependencies `guestfs-tools` and `python3-libvirt` are installed. This role requires these packages to connect to `libvirt` and to customize the virtual image in a later step. These package names work on Fedora Linux. If you're using RHEL 8 or CentOS, use `libguestfs-tools` instead of `guestfs-tools`. For other distributions, adjust accordingly.

- name: Ensure requirements in place
  package:
    name:
      - guestfs-tools
      - python3-libvirt
    state: present
  become: yes

# obtain a list of existing VMs so that you don't overwrite an existing VM on accident. uses the `virt` module from the collection `community.libvirt`, which interacts with a running instance of KVM with `libvirt`. It obtains the list of VMs by specifying the parameter `command: list_vms` and saves the results in a variable `existing_vms`. `changed_when: no` for this task to ensure that it's not marked as changed in the playbook results. This task doesn't make any change in the machine; it only checks the existing VMs. This is a good practice when developing Ansible automation to prevent false reports of changes.
- name: Get VMs list
  community.libvirt.virt:
    command: list_vms
  register: existing_vms
  changed_when: no

#execute only when the VM name the user provides doesn't exist. And uses the module `get_url` to download the base cloud image into the `/tmp` directory
- name: Create VM if not exists
  block:
  - name: Download base image
    get_url:
      url: "{{ base_image_url }}"
      dest: "/tmp/{{ base_image_name }}"
      checksum: "sha256:{{ base_image_sha }}"
      
# copy the file to libvirt's pool directory so we don't edit the original, which can be used to provision other VMS later
  - name: Copy base image to libvirt directory
    copy:
      dest: "{{ libvirt_pool_dir }}/{{ vm_name }}.qcow2"
      src: "/tmp/{{ base_image_name }}"
      force: no
      remote_src: yes 
      mode: 0660
    register: copy_results
  - 
# Resize the VM disk
  - name: Resize VM disk
    command: qemu-img resize "{{ libvirt_pool_dir }}/{{ vm_name }}.qcow2" "{{ disk_size }}G"
    when: copy_results is changed

# uses command module to run virt-customize to customize the image
  - name: Configure the image
    command: |
      virt-customize -a {{ libvirt_pool_dir }}/{{ vm_name }}.qcow2 \
      --hostname {{ vm_name }} \
      --root-password password:{{ vm_root_pass }} \
      --ssh-inject 'root:file:{{ ssh_key }}' \
      --uninstall cloud-init --selinux-relabel
# Added option to configure an IP address
      --firstboot-command "nmcli c m eth0 con-name eth0 ip4 {{ ip_addr }}/24 gw4 {{ gw_addr }} ipv4.method manual && nmcli c d eth0 && nmcli c u eth0"

    when: copy_results is changed

  - name: Define vm
    community.libvirt.virt:
      command: define
      xml: "{{ lookup('template', 'vm-template.xml.j2') }}"
    when: "vm_name not in existing_vms.list_vms"

- name: Ensure VM is started
  community.libvirt.virt:
    name: "{{ vm_name }}"
    state: running
  register: vm_start_results
  until: "vm_start_results is success"
  retries: 15
  delay: 2

- name: Ensure temporary file is deleted
  file:
    path: "/tmp/{{ base_image_name }}"
    state: absent
  when: cleanup_tmp | bool
```

Changed my user to own the libvirt directory:
`chown -R david:david /var/lib/libvirt/images`

Create playbook `kvm_provision.yaml`
```yml
---
- name: Deploys VM based on cloud image
  hosts: localhost
  gather_facts: yes
  become: yes
  vars:
    pool_dir: "/var/lib/libvirt/images"
    vm: control
    vcpus: 2
    ram_mb: 2048
    cleanup: no
    net: default
    ssh_pub_key: "/home/davidt/.ssh/id_ed25519.pub"
    disksize: 20

  tasks:
    - name: KVM Provision role
      include_role:
        name: kvm_provision
      vars:
        libvirt_pool_dir: "{{ pool_dir }}"
        vm_name: "{{ vm }}"
        vm_vcpus: "{{ vcpus }}"
        vm_ram_mb: "{{ ram_mb }}"
        vm_net: "{{ net }}"
        cleanup_tmp: "{{ cleanup }}"
        ssh_key: "{{ ssh_pub_key }}"
```

Add the libvirt collection
`ansible-galaxy collection install community.libvirt`

Create a VM with a new name
`ansible-playbook -K kvm_provision.yaml -e vm=ansible1`

 --run-command 'nmcli c a type Ethernet ifname eth0 con-name eth0 ip4 192.168.124.200 gw4 192.168.124.1'

parted /dev/vda resizepargit t 4 100%
Warning: Partition /dev/vda4 is being used. Are you sure you want to continue?
Yes/No? y                                                                 
Information: You may need to update /etc/fstab.

lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    252:0    0   20G  0 disk 
├─vda2 252:2    0  200M  0 part /boot/efi
├─vda3 252:3    0    1G  0 part /boot
└─vda4 252:4    0  8.8G  0 part /

variables
{{ ansible_user }}
{{ ansible_password }}
{{ gw_addr }}
{{ ip_addr }}

; useradd -m -p {{ ansible_user }} ; chage -d 0 {{ ansible_user }} ; cat {{ ansible_password }} > passwd {{ ansible_user }} --stdin" \

```
  - name: Configure the image
    command: |
      virt-customize -a {{ libvirt_pool_dir }}/{{ vm_name }}.qcow2 \
      --hostname {{ vm_name }} \
      --root-password password:{{ vm_root_pass }} \
      --uninstall cloud-init --selinux-relabel \
      --firstboot-command "nmcli c m eth0 con-name eth0 ip4 \
                           {{ ip_addr }}/24 gw4 {{ gw_addr }} \
                           ipv4.method manual && nmcli c d eth0 \
                           && nmcli c u eth0 && adduser \
                           {{ ansible_user }} && echo \
                           "{{ ansible_password }}" | passwd \
                           --stdin {{ ansible_user }}"
    when: copy_results is changed

  - name: Add ssh keys
    command: |
      virt-customize -a {{ libvirt_pool_dir }}/{{ vm_name }}.qcow2 \
      --ssh-inject '{{ ansible_user }}:file:{{ ssh_key }}'

```