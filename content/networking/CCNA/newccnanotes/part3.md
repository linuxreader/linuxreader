1.0 Network Fundamentals  
1.10 Identify IP parameters for Client OS (Windows, Mac OS, Linux)  
4.0 IP Services  
4.3 Explain the role of DHCP and DNS within the network  
4.6 Configure and verify DHCP client and relay  
DHCP Concepts

```
Four messages between the client and server (DORA):
```

- Discover: - Sent by the DHCP client to find a willing DHCP server
- Offer: - Sent by a DHCP server to offer to lease  
    Request: - Sent by the DHCP client to ask the server to lease the IPv4 address listed in the Offer message  
    Acknowledgment: - Sent by the DHCP server to assign the address and to list the mask, default router, and DNS server IP addresses

```
IPv4 addresses that allow a host that has no IP address to still be able to send and receive messages on the local subnet:
0.0.0.0: - An address reserved for use as a source IPv4 address for hosts that do not yet have an IP address.
255.255.255.255: - The local broadcast IP address. Packets sent to this destination address are broadcast on the local data link, but routers do not forward them.
```

- - a client, sends a Discover message, with source IP address of 0.0.0.0Host A sends the packet to destination 255.255.255.255 router R1 will not forward this packet.
        - - assuming the DHCP client chooses to use a DHCP option called the broadcast flag; all examples in this book assume the broadcast flag is used.
        - all hosts in the subnet receive the Offer message
        - the original Discover message lists a number called the client ID□□ includes the host’s MAC address, that identifies the original host (host A in this case).As a result, host A knows that the Offer message is meant for host A, which.

Supporting DHCP for Remote Subnets with DHCP Relay

```
ip helper - Watch for incoming DHCP messages, with destination IP address 255.255.255.255 - address server-ip.
```

[7 DHCP config](7%20DHCP%20config.md)
[12 IP Services](12%20IP%20Services.md)
[13 LAN Architecture](13%20LAN%20Architecture.md)