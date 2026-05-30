# 2 ACLs
This chapter covers the following exam topics:  
5.0 Security Fundamentals  
5.6 Configure and verify access control lists

Can be used to match packets for applying Quality of Service (QoS) features.  
ACL Location and Direction

- inbound to the router, before the router makes its forwarding (routing) decisionoutbound, after the router makes its forwarding decision and has determined the exit
- interface to use.enable an ACL on an interface that processes the packet, in the direction the packet flows
- - through that interface.the router then processes every inbound or outbound IP packet using that ACL

Taking Action When a Match Occurs

- deny or permit  
    Types of IP ACLs
- - Standard numbered ACLs (1Extended numbered ACLs (100–99) or (1300–199) or (2000-1999)-2699)

Named ACLs

- - Editing with sequence numbersconfiguration identifies the ACL either using a number or a name. ACLs will also be
- either standard or extended  
    Standard Numbered IPv4 ACLs
- - matches only the source IP address identify the ACL using numbers rather than names (numbered)
- Looks at IPv4 packets.  
    List Logic with IP ACLs
- - router takes the action listed in that line of the ACL and stops looking further in the ACLevery IP ACL has a deny all statement implied at the end of the ACL

Matching Logic and Command Syntax

- - ACL is one or more accessany number from the ranges shown in the preceding line of syntax. -list commands with the same number,
- - (One number is no better than the other.) IOS refers to each line in an ACL is an Access Control Entry (ACE
- - engineers just call them ACL statements.each access-list command also lists the action (permit or deny), plus the matching logic.

Matching the Exact IP Address

### 2 Standard ACLs

Friday, September 17, 2021 12:28 PM

Matching the Exact IP Address

- permit if source = 10.1.1.1
    - - accessIf you use Host keyword IOS will remove the keyword in the config-list 1 permit 10.1.1.1
    - access-list 1 permit any

Matching Any/All Addresses

ACL show commands list

- - counters for the number of packets matched by each command in the ACLno counter for that implicit denyany concept at the end of the ACL. , but there is
- Configure deny any command to see deny counts  
    Implementing Standard IP ACLs


access-list _access-list-number_ {deny | permit} _source_ [source-wildcard]

- Plan the location (router and interface) and direction (in or out) on that interface:
    - placed near to the destination of the packetsdiscard packets that should not be discarded.so that they do not unintentionally
    - identify the source IP addresses of packets as they go in the direction that the ACL is examining.
    - # access-list _access-list-number_ {deny | permit} _source_ [source-wildcard]
        
- Configure one or more access-list
- Enable the ACL
    - _(config-if)# ip access-group number {in | out}_  
        Standard Numbered ACL Example 1  
        R2(config)# accessR2(config)# access--list 1 permit 10.1.1.1list 1 deny 10.1.1.0 0.0.0.255  
        R2(config)# access-list 1 permit 10.0.0.0 0.255.255.255  
        R2(config-if)# ip access-group 1 in

show ip access-lists

- details about IPv4 ACLs only

show access-lists

- lists details about any configure ACL, not just IPv4
- lists the number or name of any IP ACL enabled on the interface

show ip interface s0/0/1

Standard Numbered ACL Example 2

- standard ACLs cannot check the destination IP address.

- - standard ACLs cannot check the destination IP address.extended ACL lets you check both the source and destination IP address.
- - accessrouter checks packets that it routes against the ACL for outbound ACLs- to leave text documentation that stays with the ACL.-list remark parameter
- a router does not filter packets that the router itself creates with an outbound ACL  
    Troubleshooting and Verification Tips
- IOS keeps statistics about the packets matched by each line of an ACL  
    logkeyword  
    ▪ add to end of accessIOS then issues log messages with occasional statistics about matches of that -list command  
    ▪ ACL line
- Double check the ACL is enabled on the right interface, or for the right direction  
    Practice Building access-list Commands  
    Tips to consider when choosing matching parameters to any access-list command:
- - To match a specific address, just list the address.To match any and all addresses, use the any keyword.

several practice problems (wildcard)

- Packets from 172.16.5.4- 0.0.0.0
- Packets from hosts with 192.168.6- 0.0.0.255
- Packets from hosts with 192.168- 0.0.255.255
- Packets from any hosts- 255.255.255.255
- Packets from subnet 10.1.200.0/21- 0.0.7.255
- Packets from subnet 172.20.112.0/23- 0.0.1.255
- Packets from subnet 172.20.112.0/26- 0.0.0.63
- Packets from subnet 192.168.9.64/28- 0.0.0.15
- Packets from subnet 192.168.9.64/30- 0.0.0.3

Reverse Engineering from ACL to Address Range (practice problems)  
1.2. one address192.168.4.0 -192.168.4.127  
3.4. 192.168.6.0 172.30.96.0 --192.168.6. 31172.30.96.255  
5.6. 172.30.96.0 10.1.192.0 --10.1.192..3172.30.96. 63  
7.8. 10.1.192.0 10.1.192.0 --10.1.193.25510.1.255.255

```
128 64 32 16 8 4 2 1
128 192 224 240 248 252 254 255
```

This chapter covers the following exam topics:  
5.0 Security Fundamentals  
5.6 Configure and verify access control lists

- all the parameters must be matched correctly to match that one ACE..  
    Matching the Protocol, Source IP, and Destination IP 9Extended)
- Uses the access-list global command. The
- - syntax is identical up until permit or deny keywordRequires three matching parameters:  
        ○ IP protocol type  
        ○ source IP address  
        ○ destination IP address.
- - identifies the header that follows the IP header (layer 4)TCP, UDP, EIGRP, IGMP, etc
- - Use protocol as keywordKeyword IP means all IPv4 packets

IP header’s Protocol Type field

Syntax

Access(Destination-list 101 (list #) permit/ Deny tcp (protocol) 10.0.0.1 0.0.0.0 (Source) 10.1.0.1 0.0.0.255

- Requires the use of the host keyword for specific address
- Examples

```
▪ Any IP packet that has a TCP header
```

- access-list 101 deny tcp any any
- access▪ Any IP packet that that has a UDP header-list 101 deny udp any any
- access▪ Any IP packet that has a ICMP header-list 101 deny icmp any any

```
▪ All IP packets from host 1.1.1.1 going to host 2.2.2.2
```

- access-list 101 deny ip host 1.1.1.1 host 2.2.2.2

```
access▪ All IP packets that have a UDP header following the IP header, from subnet 1.1.1.0/24 going to any destination-list 101 deny udp 1.1.1.0 0.0.0.255 any
```

##### -

IP and TCP Header
