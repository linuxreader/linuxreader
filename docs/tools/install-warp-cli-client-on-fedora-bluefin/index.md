# 

Create a distrobox:  
```
distrobox create -i docker.io/library/ubuntu:latest -n cfw-dbx--root -H ~/distrobox/cfw-dbx--root --volume /run/dbus/system_bus_socket:/run/dbus/system_bus_socket --additional-flags "--device=/dev/net/tun --cap-add=NET_ADMIN --cap-add=SYS_ADMIN" -r
```

Enter the distrobox:  
```
distrobox enter --root cfw-dbx--root
```

## Install CloudFlare Warp client in the distrobox
Official install doc: https://pkg.cloudflareclient.com/
### Install cloudflare warp client pre-reqs:  
```
sudo apt install curl lsb-release
```

### Add cloudflare gpg key
```
curl -fsSL https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg
```

### Add this repo to your apt repositories
```
echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list
```

### Install
```
sudo apt-get update && sudo apt-get install cloudflare-warp
```


## Set up the client in the distrobox

### Run the client
Leave this open in it's own window:  
```
sudo warp-svc
```

In a new window, enter the distrobox:  
```
distrobox enter --root cfw-dbx--root
```

Register the client:  
```
warp-cli registration new {{ insert_domain_here }}
```

Get the token by inspecting the token id of the browser button that shows up:  
```
warp-cli registration token com.cloudflare.warp://{{ domain }}.cloudflareaccess.com/auth?token={{ token_id }}
```

## Security Settings
### Turn on Malware filtering:  
```
warp-cli dns families malware
```

### Encrypt DNS Queries
```
warp-cli mode warp+doh
```

## Connect

Make a test connection: 
```
warp-cli connect
```

```
warp-cli status
```

## Create a startup file
```
sudo vim /etc/systemd/system/cfw-dbx-start.service
```

```
[Unit]
Description=Start Cloudflare WARP client
After=network-online.target
Wants=network-online.target
RequiresMountsFor=%t/containers
StartLimitIntervalSec=60
StartLimitBurst=5

[Service]
Type=exec
ExecStartPre=/bin/podman start cfw-dbx--root
ExecStart=/bin/podman exec cfw-dbx--root bash -c "warp-svc"
Restart=on-failure
RestartSec=7
RemainAfterExit=yes
```

Create the timer file:   
```
sudo vim /etc/systemd/system/cfw-dbx-start.timer
```

Reload and enable timer:  
```
sudo systemctl daemon-reload && sudo systemctl enable cfw-dbx-start.timer
```

## Create aliases in .bashrc
```bash
alias vpnon='distrobox-enter --root cfw-dbx--root -- warp-cli connect && until ip link show CloudflareWARP > /dev/null 2>&1; do sleep 0.5; done && sudo systemd-resolve --interface=CloudflareWARP --set-domain={{ your_domain }}'
alias vpnoff='distrobox-enter --root cfw-dbx--root -- warp-cli disconnect'
```

## Container maintenance
Create a service file for automatic updates:  
```
sudo vim /etc/systemd/system/cfw-dbx-upgrade.service
```

```
[Unit]
Description=Upgrade cfw-dbx--root
After=network-online.target
Wants=network-online.target
RequiresMountsFor=%t/containers
StartLimitIntervalSec=600
StartLimitBurst=5

[Service]
Type=exec
ExecStartPre=/bin/podman start cfw-dbx--root
ExecStart=/bin/podman exec cfw-dbx--root bash -c "apt update -y && apt full-upgrade -y"
Restart=on-failure
RestartSec=60
RemainAfterExit=yes
```

Timer file:   
```
sudo vim /etc/systemd/system/cfw-dbx-upgrade.timer
```

```
[Unit]
Description=Upgrade cfw-dbx--root daily.

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=300

[Install]
WantedBy=timers.target
```

Reload and enable:  
```
sudo systemctl daemon-reload && sudo systemctl enable cfw-dbx-upgrade.timer
```