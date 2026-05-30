1.0 Network Fundamentals

1.1 Explain the role and function of network components

1.1.a Routers

1.2 Describe characteristics of network topology architectures

1.2.d WAN

Leased-Line WANs

Physical Details of Leased Lines

-   predetermined speed
-   Full Duplex
-   Uses two pairs of wires one for each direction
-   Conceptually crossover

Leased Circuit

-   Electrical circuit (line) between 2 endpoints

Serial Link (line)

-   Bits flow serially
-   Routers use serial interfaces

Point to point link (line)

-   two points only

T1

-   1.544 Mbps

WAN link

-   General term

Private Line

-   Data is private

HDLC Data-Link Details of Leased Lines

-   Leased line specifies layer 1
-   HDLC and PPP are the most popular Layer 2 protocols used on leased lines

HDLC

-   less work than ethernet because of point to point leased line
-   has an address field, but the destination is implied
-   Cant use between cisco and non cisco
-   Cisco HDLC type field is proprietary

Comparing HDLC Header Fields to Ethernet

HDLC Header

Flag

-   Like preamble, SFD
-   1 byte

Destination address

-   1 byte

Control

-   No longer used
-   1 byte

Type

-   Type of layer 3 packet inside the frame
-   2 bytes

Data

FCS

-   Error detection
-   2 bytes

How Router Use a WAN Data Link

How routers use HDLC when sending data

-   LAN1 802.3header/IP Packet/ 802.3trailer >
-   HDLC HDLCheader/IP Packet/ HDLC Trailer >
-   LAN2 802.3header/Ippacket/802.3trailer >

Leased line negatives

-   Higher cost and
-   long install times
-   Slow speeds

Ethernet as a WAN technology

Customer Router

-   CPE

Service provider

-   point of Presence (PoP)

-   Where the fiber connects at the provider

Common ethernet WAN names

-   Ethernet WAN
-   Ethernet Line Service (E-Line)
-   Ethernet emulation
-   Ethernet over MPLS (EoMPLS) (Multi protocol label switching) A technology used to create ethernet service for a customer.

-   Acts like simple ethernet link between two routers

How Routers Route IP Packets Using Ethernet Emulation

EoMPLS WAN (Provider network simulating an ethernet link)

-   802.3 header and trailer

IP Routing

1.  Use FCS field to ensure that the frame had no errors; if errors occurred, discard the frame.

1.  Discard the old data-link header and trailer, leaving the IP packet.

1.  Compare the IP packet’s destination IP address to the routing table, and find the route that best matches the destination address. This route identifies the outgoing interface of the router and possibly the next-hop router IP address.

1.  Encapsulate the IP packet inside a new data-link header and trailer, appropriate for the outgoing interface, and forward the frame

The IP Header (20 bytes)

-   Version, Length, DS Field, Packet Length (4 bytes)
-   Identification, Flags, Fragment Offset (4 bytes)
-   Time to Live, Protocol, Header Checksum (4 bytes)
-   Source IP (4 bytes)
-   Destination IP (4 bytes)

IP Routing Protocols

1.  add route for each directly connected subnet
2.  tell neighbors about routes in routing table
3.  add new routes learned to routing table, with next hop as the router the address was learned from   

ARP

-   Ethernet broadcast arp request >

-   Sender IP, sender MAC, Target IP, target MAC ??

-   < Ethernet unicast ARP reply

-   Target IP, Target MAC, Sender IP, sender MAC

arp -a

-   to see arp cache on most operating systems
