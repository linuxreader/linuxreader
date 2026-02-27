
This chapter covers the following exam topics:

5.0 Security Fundamentals

5.6 Configure and verify access control lists

- all the parameters must be matched correctly to match that one ACE. .

Matching the Protocol, Source IP, and Destination IP 9Extended)

- Uses the access-list global command. The
- syntax is identical up until permit or deny keyword
- Requires three matching parameters:

- IP protocol type
- source IP address
- destination IP address.

IP header’s Protocol Type field

- identifies the header that follows the IP header (layer 4)
- TCP, UDP, EIGRP, IGMP, etc
- Use protocol as keyword
- Keyword IP means all IPv4 packets

Syntax

\# Access-list 101 (list #) permit/ Deny tcp (protocol) 10.0.0.1 0.0.0.0 (Source) 10.1.0.1 0.0.0.255 (Destination

- Requires the use of the host keyword for specific address
- Examples

- access-list 101 deny tcp any any

- Any IP packet that has a TCP header

- access-list 101 deny udp any any

- Any IP packet that that has a UDP header

- access-list 101 deny icmp any any

- Any IP packet that has a ICMP header

- access-list 101 deny ip host 1.1.1.1 host 2.2.2.2

- All IP packets from host 1.1.1.1 going to host 2.2.2.2

- access-list 101 deny udp 1.1.1.0 0.0.0.255 any

- All IP packets that have a UDP header following the IP header, from subnet 1.1.1.0/24 going to any destination

IP and TCP Header

IP Header

Misc Header Fields

- 9 bytes

Protocol

- 1 byte
- ie 6 = tcp
- identify TCP header

Header Checksum

- 2 bytes

Source IP

- 4 bytes

Dest. IP

- 4 bytes

Options

- variable

TCP Header

Source Port

- 2 bytes

Dest. port

- 2 bytes

Rest of TCP

- 16 bytes

tcp or udp keyword

- can optionally reference the source and/or destination port
- equal, not equal, less than, greater than, and for a range of port numbers
- can use port numbers or  keywords for some well-known application ports

positions of the source and destination port fields in the access-list command and these port number keywords.

\# access-list 101 permit (protocol) Source_IP (source port) dest_IP (dest port)

Protocol

- tcp
- udp

Source Port

- eq _
- ne_
- lt_
- gt_
- range_

Dest. Port

- eq _
- ne_
- lt_
- gt_
- range_

eq: =

lt: <

ne: not equal

gt: >

range: x to y

ie:

\# access-list 101 permit tcp 172.16.1.0 0.0.0.255 172.16.3.0 0.0.0.255 eq 21

- eq 21 is in the destination port position

Apps and Port number shortcuts for ACL Commands

20 ftp-data

21 ftp

22 -

23 telnet

25 smtp

53 domain

67 bootps (dhcp server)

68 bootpc (dhcp client)

69 tftp

80 www

110 pop3

161 snmp

443 -

514 -

16,384 - 32,767 (RTP/ Voice/ Video) -

Extended IP ACL Configuration

ommand 
access-list access-list-number {deny I 
permit} protocol source source-wildcard 
destination destination-wildcard [log I 
log-input] 
access-list access-list-number {deny I 
permit} {tcp I udp} source source-wild- 
card [operator [port]] destination destina- 
tion-wildcard [operator [port]] [estab- 
fished] [log] 
Configuration Mode and 
Description 
Global command for extended 
numbered access lists. Use a 
number between 100 and 199 or 
2000 and 2699, inclusive. 
A version of the access-list com- 
mand with parameters specific 
to TCP and/or UDP. ">

- enable the ACL using the same ip access-group command used with standard ACLs.
- Place extended ACLs as close as possible to the source of the packets that will be filtered.

\- saves some bandwidth.

- ACL numbers

- 100–199 and 2000–2699

Extended IP Access Lists: Example 1

(If you were to type eq 80, the config would show eq www.)

#(int) ip access-group 101 in

Named ACLs and ACL Editing

Named IP Access Lists

- Easier to remember
- Uses ACL subcommands instead of global config commands
- editing features allow deleting individual lines and inserting new ones

Config

\# ip access-list (standard/ extended) (name)

#(ACLmode) permit 1.1.1.1

#(ACLmode) permit 2.2.2.2

#(ACLmode) permit 3.3.3.3

#(ACLmode) deny ip 10.1.2.0 0.0.0.255 10.2.3.0 0.0.0.255

#(int) ip access-group barney out

- delete and add new lines to the ACL from within ACL configuration mode

- deleting a single entry from the ACL.

#(ACLmode) no deny ip 10.1.2.0 0.0.0.255

Editing ACLs Using Sequence Numbers (named and numbered (not the global numbered way)

- ACL sequence number is added to each ACL permit or deny statement,
- numbers represent the sequence of statements in the ACL
- Numbered ACLs can use a configuration style like named ACLs, as well as the traditional style, for the same ACL; the new style is required to perform advanced ACL editing.
- Deleting single lines:

- delete an ACE with a no sequence-number subcommand.

- Inserting new lines:

- New ACEs can be configured with a sequence number before the deny or permit command, dictating the location of the statement within the ACL.

- Automatic sequence numbering:

- sequence numbers are added to ACEs automatically

\# Show ip access-lists 24

- Shows access list 24 and sequence numbers with each entry

#(ACLmode) no 20

- delete entry 20

#(ACLmode) 5 deny 10.1.1.1

- enters this new ace as sequence #5
- Places the sequence number in the list in order

although Example 3-6 uses a numbered ACL, named ACLs use the same process to edit (add and remove) entries.

Numbered ACL Configuration Versus Named ACL Configuration

- numbered ACLs are stored with the original style of configuration, as global access-list commands, no matter which method is used to configure the ACL.
- the parts of ACL 24 configured with both new-style commands and old-style commands are all listed in the same old-style ACL (show running-config).

ACL Implementation Considerations

- Place more specific statements early in the ACL.
- Disable an ACL from its interface (using the no ip access-group interface subcommand) before making changes to the ACL.

- By doing so, you avoid issues with the ACL during an interim state

Mitigating Security Issues with ACLs

Security threats that can be mitigated with ACLs

\- IP address spoofing, inbound

\- IP address spoofing, outbound

\- DoS TCP SYN attacks, blocking external attacks

\- Dos TCP SYN attacks, using TCP Intercept

\- DoS smurf attacks

\- Denying/filtering ICMP messages, inbound

\- Denying/filtering ICCMP messages, outbound

\- Denying/filtering Traceroute

\* Don't allow external packets that have an internal destination address.

Configuring ACLs from the internet

- Deny any source addresses from your internal networks

\- Deny any local host addresses (127.0.0.0/8)

\- Deny any reserved private addresses (RFC 1918)

\- Deny any addresses in the IP multicast address range (224.0.0.0/4)

Controlling VTY (Telnet/ SSH) Access

1\. Create a standard IP access list that permits only the host or hosts you want to be able to telnet into the routers.

2\. Apply the access list to the VTY line with the access-class in command.

(g) # access-list 50 permit host 172.16.10.3

(g) # line vty  0 4

(int) # access-class 50 in

Monitoring Access Lists

\# show access-list

shows access lists, parameters, statistics, etc.

\# show access-list 110

Shows info for access list 110

\# show ip access-list

shows IP access lists on the router

\# show ip interface

Shows which interfaces have access lists set on them.

\# show running-config

Shows ACLs and what interfaces that have them.
