# 3 Extended ACLs

Friday, September 17, 2021 1:11 PM

IP and TCP Header  
IP Header  
Misc Header Fields▪ 9 bytes

```
▪ 1 byte
▪▪ ie 6 = tcpidentify TCP header
```

```
Protocol
```

```
Header Checksum▪ 2 bytes
```

```
▪ 4 bytes
```

```
Source IP
```

```
Dest. IP▪ 4 bytes
Options▪ variable
TCP Header
Source Port- 2 bytes
```

- 2 bytes

```
Dest. port
```

```
Rest of TCP- 16 bytes
```

tcp or udp keyword

- - can optionally reference the source and/or destination portequal, not equal, less than, greater than, and for a range of port numbers
- can use port numbers or keywords for some well-known application ports  
    positions of the source and destination port fields in the access-list command and these port  
    number keywords.  
    # access-list 101 permit **(protocol)** Source_IP **(source port)** dest_IP **(dest port)**  
    Protocol-- tcpudp  
    - - eq _ne_  
    - lt_

```
Source Port
```

- - lt_gt_
- range_
- - eq _ne_
- - lt_gt_
- range_

```
Dest. Port
```

```
eq: =lt: <
ne: not equal
gt: >range: x to y
ie:
# access-list 101 permit tcp 172.16.1.0 0.0.0.255 172.16.3.0 0.0.0.255 eq 21
```

- eq 21 is in the destination port position  
    Apps and Port number shortcuts for ACL Commands  
    20 21 **ftpftp-data**  
    22 -  
    23 25 **telnetsmtp**  
    53 67 **domainbootps (dhcp server)**  
    68 69 **bootpc (dhcp clienttftp** )  
    80 **www**  
    110 161 **pop3snmp**  
    443 514 - -  
    16,384 -32,767 (RTP/ Voice/ Video) -  
    Extended IP ACL Configuration

- enable the ACL using the sameip access-groupcommand used with standard ACLs.
    - saves some bandwidth.
- Place extended ACLs as close as possible to the source of the packets that will be filtered.
- ACL numbers - 100 – 199 and 2000– 2699

```
(If you were to type eq 80, the config would show eq http://www.)
```

Extended IP Access Lists: Example 1

#(int) ip access-group 101 in

```
Named IP Access Lists
```

- Easier to remember
- - Uses ACL subcommands instead of global config commandsediting features allow deleting individual lines and inserting new ones  
        Config  
        #(ACLmode) permit 1.1.1.1  
        #(ACLmode) permit 2.2.2.2#(ACLmode) permit 3.3.3.3  
        #(ACLmode) deny ip 10.1.2.0 0.0.0.255 10.2.3.0 0.0.0.255

```
# ip access-list (standard/ extended) (name)
```

```
#(int) ip access-group barney out
```

```
#(ACLmode) no deny ip 10.1.2.0 0.0.0.255
```

- deleting a single entry from the ACL.
- delete and add new lines to the ACL from within ACL configuration mode

Named ACLs and ACL Editing

- ACL sequence number is added to each ACL permit or deny statement,
- numbers represent the sequence of statements in the ACLNumbered ACLs can use a configuration style like named ACLs, as well as the traditional
- style, for the same ACL; the new style is required to perform advanced ACL editing.
- Deleting single lines:- delete an ACE with a **no sequence-number** subcommand.  
    New ACEs can be configured with a sequence number before the deny or permit  
    command, dictating the location of the statement within the ACL.
- Inserting new lines: -
- Automatic sequence numbering: - sequence numbers are added to ACEs automatically
    
    # Show ip access- Shows access list 24 and sequence numbers with each entry-lists 24
    
    ```
     - delete entry 20
    ```
    
    #(ACLmode) no 20  
    - - enters this new ace as sequence #5Places the sequence number in the list in order

```
#(ACLmode) 5 deny 10.1.1.1
```

```
although Example 3-6 uses a numbered ACL, named ACLs use the same process to edit (add
and remove) entries.
```

Editing ACLs Using Sequence Numbers (named and numbered (not the global numbered way)

```
numbered ACLs are stored with the original style of configuration, as global access-list commands,
no matter which method is used to configure the ACL.
```

##### -

```
the parts of ACL 24 configured with both new-style commands and old-style commands are all
listed in the same old-style ACL (show running-config).
```

##### -

Numbered ACL Configuration Versus Named ACL Configuration

- Place more specific statements early in the ACL.
    - By doing so, you avoid issues with the ACL during an interim state
- Disable an ACL from its interfacemaking changes to the ACL. (using the **no ip access-group interface** subcommand)before

ACL Implementation Considerations

Mitigating Security Issues with ACLs  
Security threats that can be mitigated with ACLs

- IP address spoofing, inbound
- -IP address spoofing, outboundDoS TCP SYN attacks, blocking external attacks
- Dos TCP SYN attacks, using TCP Intercept
- -DoS smurf attacksDenying/filtering ICMP messages, inbound
- -Denying/filtering ICCMP messages, outboundDenying/filtering Traceroute

```
* Don't allow external packets that have an internal destination address.
Configuring ACLs from the internet
```

- -Deny any source addresses from your internal networks Deny any local host addresses (127.0.0.0/8)
- -Deny any reserved private addresses (RFC 1918)Deny any addresses in the IP multicast address range (224.0.0.0/4)

Controlling VTY (Telnet/ SSH) Access

1. Create a standard IP access list that permits only the host or hosts you want to be able to telnet into the routers.
2. Apply the access list to the VTY line with the access-class in command.  
    (g) # access-list 50 permit host 172.16.10.3  
    (g) # line vty 0 4(int) # access-class 50 in

Monitoring Access Lists

show access-list

shows access lists, parameters, statistics, etc.

show access-list 110

Shows info for access list 110

show ip access-list

shows IP access lists on the router

show ip interface

Shows which interfaces have access lists set on them.

show running-config

Shows ACLs and what interfaces that have them.

