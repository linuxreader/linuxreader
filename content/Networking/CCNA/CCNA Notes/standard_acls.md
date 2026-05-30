# Standard Access Control Lists

Can be used to match packets for applying Quality of Service (QoS) features.

### ACL location and direction

- inbound to the router, before the router makes its forwarding (routing) decision
- outbound, after the router makes its forwarding decision and has determined the exit interface to use.
- enable an ACL on an interface that processes the packet, in the direction the packet flows through that interface.
- the router then processes every inbound or outbound IP packet using that ACL

## Actions when a match occurs

- deny or permit

### Types of IP ACLs

- Standard numbered ACLs (1–99) or (1300-1999)
- Extended numbered ACLs (100–199) or (2000-2699)

#### Named ACLs

- Editing with sequence numbers
- configuration identifies the ACL either using a number or a name. ACLs will also be
- either standard or extended

#### Standard Numbered IPv4 ACLs

- matches only the source IP address
- identify the ACL using numbers rather than names (numbered)
- Looks at IPv4 packets.

List Logic with IP ACLs

- router takes the action listed in that line of the ACL and stops looking further in the ACL
- every IP ACL has a deny all statement implied at the end of the ACL

Matching Logic and Command Syntax

- Access list is one or more access-list commands with the same number.
- any number from the ranges shown in the preceding line of syntax.
- (One number is no better than the other.) IOS refers to each line in an ACL is an Access Control Entry (ACE engineers just call them ACL statements.
- each access-list command also lists the action (permit or deny), plus the matching logic.

Matching the Exact IP Address

- permit if source = 10.1.1.1

- access-list 1 permit 10.1.1.1
- If you use Host keyword IOS will remove the keyword in the config

Matching Any/All Addresses

- access-list 1 permit any

ACL show commands list

- counters for the number of packets matched by each command in the ACL,
- no counter for that implicit deny any concept at the end of the ACL.
- Configure deny any command to see deny counts

### Implementing Standard IP ACLs

\# access-list access-list-number {deny | permit} source \[source-wildcard\]

- Plan the location (router and interface) and direction (in or out) on that interface:

- placed near to the destination of the packets so that they do not unintentionally discard packets that should not be discarded.
- identify the source IP addresses of packets as they go in the direction that the ACL is examining.

- Configure one or more access-list

- \# access-list access-list-number {deny | permit} source \[source-wildcard\]

- Enable the ACL

- (config-if)# ip access-group number {in | out}

Standard Numbered ACL Example 1

R2(config)# access-list 1 permit 10.1.1.1

R2(config)# access-list 1 deny 10.1.1.0 0.0.0.255

R2(config)# access-list 1 permit 10.0.0.0 0.255.255.255

R2(config-if)# ip access-group 1 in

\# show ip access-lists

- details about IPv4 ACLs only

\# show access-lists

- lists details about any configure ACL, not just IPv4

\# show ip interface s0/0/1

- lists the number or name of any IP ACL enabled on the interface

Standard Numbered ACL Example 2

- standard ACLs cannot check the destination IP address.
- extended ACL  lets you check both the source and destination IP address.
- access-list remark parameter

- to leave text documentation that stays with the ACL.

- router checks packets that it routes against the ACL for outbound ACLs
- a router does not filter packets that the router itself creates with an outbound ACL

Troubleshooting and Verification Tips

- IOS keeps statistics about the packets matched by each line of an ACL

log keyword

- add to end of access-list command
- IOS then issues log messages with occasional statistics about matches of that ACL line

- Double check the ACL is enabled on the right interface, or for the right direction

Practice Building access-list Commands

Tips to consider when choosing matching parameters to any access-list command:

- To match a specific address, just list the address.
- To match any and all addresses, use the any keyword.

several practice problems (wildcard)

- Packets from 172.16.5.4

- 0.0.0.0

- Packets from hosts with 192.168.6

- 0.0.0.255

- Packets from hosts with 192.168

- 0.0.255.255

- Packets from any hosts

- 255.255.255.255

- Packets from subnet 10.1.200.0/21

- 0.0.7.255

- Packets from subnet 172.20.112.0/23

- 0.0.1.255

- Packets from subnet 172.20.112.0/26

- 0.0.0.63

- Packets from subnet 192.168.9.64/28

- 0.0.0.15

- Packets from subnet 192.168.9.64/30

- 0.0.0.3

Reverse Engineering from ACL to Address Range (practice problems)

1.  one address
2.  192.168.4.0 - 192.168.4.127
3.  192.168.6.0 - 192.168.6. 31
4.  172.30.96.0 - 172.30.96.255
5.  172.30.96.0 - 172.30.96. 63
6.  10.1.192.0 - 10.1.192..3
7.  10.1.192.0 - 10.1.193.255
8.  10.1.192.0 - 10.1.255.255
