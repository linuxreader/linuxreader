---
title: Proxmox PfSense HaProxy Setup
summary: Access Proxmox PVE GUI through HaProxy on PfSense
date: 2026-01-17
---
Just wanted to share my setup to get local-only Proxmox GUI working through PfSense HaProxy.

I tried virtually every option I could find. The gotcha ended up being the health check option for the backend server. Everything else is pretty standard. 

General HaProxy setup pretty much matches the LS video here: https://www.youtube.com/watch?v=bU85dgHSb2E

## Frontend configuration

**External address**

![](Pasted%20image%2020260117041515.png)

**Type**
![](Pasted%20image%2020260117041615.png)

**ACL**
![](Pasted%20image%2020260117041708.png)

**ACL Actions**
![](Pasted%20image%2020260117041758.png)

**SSL Offloading**
![](Pasted%20image%2020260117041849.png)

## Backend configuration

**Backend Server Pool**
![](Pasted%20image%2020260117042001.png)

**Health check**
![](Pasted%20image%2020260117042031.png)

## It's working finally!
![](proxmox-gui.png)
