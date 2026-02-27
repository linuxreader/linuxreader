This chapter covers the following exam topics:

1.0 Network Fundamentals

1.8 Configure and verify IPv6 addressing and prefix

Introduction to IPv6

IPv6 increases the address to 128 bits in length.

Older OSPF Version 2 Upgraded to OSPF Version 3:

a newer version, OSPF version 3, was created to support IPv6. (Note: OSPFv3 was later upgraded to support advertising both IPv4 and IPv6 routes.)

ICMP Upgraded to ICMP Version 6:

ARP Replaced by Neighbor Discovery Protocol:

32 hexadecimal digits (one hex digit per 4 bits)

Figure 22-3 shows the required 40-byte part of the IPv6 header.

lass 
Payload Length 
4 Bytes 
Flow Label 
Next Header 
Source Address 
(16 Bytes) 
Destination Address 
(16 Bytes) 
Figure 22-3 IPv6 Header 
Hop Limit 
. 40 
Bytes ">

IPv6 Routing

To be able to build and send IPv6 packets out an interface, end-user devices need an IPv6 address on that interface.

End-user hosts need to know the IPv6 address of a default router, to which the host sends IPv6 packets if the host is in a different subnet.

IPv6 routers de-encapsulate and re-encapsulate each IPv6 packet when routing the packet.

IPv6 routers make routing decisions by comparing the IPv6 packet’s destination address to the router’s IPv6 routing table; the matched route lists directions of where to send the IPv6 packet next.

Note

You could take the preceding list and replace every instance of IPv6 with IPv4, and all the statements would be true of IPv4 as well.

the router must look at a protocol type field in the data-link header, which identifies the type of packet inside the data-link frame. Today, most data-link frames carry either an IPv4 packet or an IPv6 packet.

To route an IPv6 packet, a router must use its IPv6 routing table instead of the IPv4 routing table.

the process works like IPv4, except that the IPv6 packet lists IPv6 addresses, and the IPv6 routing table lists routing information for IPv6 subnets (called prefixes).

(The migration strategy of running both IPv4 and IPv6 is called dual stack.) All you have to do is configure the router to route IPv6 packets, in addition to the existing configuration for routing IPv4 packets.

IPv6 Routing Protocols

Routing Information Protocol (RIP), Open Shortest Path First (OSPF), Enhanced Interior Gateway Routing Protocol (EIGRP), and Border Gateway Protocol (BGP) were all updated to support IPv6.


these routing protocols also follow the same interior gateway protocol (IGP) and exterior gateway protocol (EGP) conventions as their IPv4 cousins. RIPng, EIGRPv6, and OSPFv3 act as interior gateway protocols, advertising IPv6 routes inside an enterprise.

Representing the Prefix Length of an Address

IPv6 uses a mask concept, called the prefix length, similar to IPv4 subnet masks.

the IPv6 prefix length is written as a /, followed by a decimal number. The prefix length defines how many bits of the IPv6 address define the IPv6 prefix, which is basically the same concept as the IPv4 subnet ID.

When writing an IPv6 address and prefix length in documentation, you can choose to leave a space before the /, or not, as shown in the next two examples.

2222:1111:0:1:A:B:C:D/64

2222:1111:0:1:A:B:C:D /64

Finding the IPv6 Prefix

by zeroing out the last 64 bits (16 digits) of the address, you find the following prefix value:

2000:1234:5678:9ABC:0000:0000:0000:0000/64

This value can be abbreviated, with four quartets of all 0s at the end, as follows:

2000:1234:5678:9ABC::/64
