3.0 IP Connectivity

3.3 Configure and verify IPv4 and IPv6 static routing

3.3.a Default route

3.3.b Network route

3.3.c Host route

3.3.d Floating static

Connected and Local IPv6 Routes

a router adds IPv6 routes based on the following:

The configuration of IPv6 addresses on working interfaces (connected and local routes)

The direct configuration of a static route (static routes)

The configuration of a routing protocol, like OSPFv3, on routers that share the same data link (dynamic routes)

Rules for Connected and Local Routes

First, the router looks for any configured unicast addresses on any interfaces by looking for the ipv6 address command.

if the interface is working—if the interface has a “line status is up, protocol status is up” notice in the output of the show interfaces command—the router adds both a connected and local route.

Routers do not create connected or local IPv6 routes for link-local addresses.

The connected route represents the subnet connected to the interface, whereas the local route is a host route for only the specific IPv6 address configured on the interface.

Routers create IPv6 routes based on each unicast IPv6 address on an interface, as configured with the ipv6 address command, as follows:

The router creates a route for the subnet (a connected route).

The router creates a host route (/128 prefix length) for the router IPv6 address (a local route).

Routers do not create routes based on the link-local addresses associated with the interface.

Routers remove the connected and local routes for an interface if the interface fails, and they re-add these routes when the interface is again in a working (up/up) state

rotocols 
Click here to view code image 
RI# show ipv6 route 
IPv6 Routing Table 
Codes: C 
Connected, 
- BGP, HA 
NHRP, 11 
default 
7 
Local, 
L 
Home Agent, 
entries 
Static, 
s 
MR - Mobile 
- ISIS Ll, 12 
ISIS L2, 
Per-user Static rou 
U 
Router, R 
RIP 
IA - ISIS interarea 
OE2 
la 
IA 
EIGRP, EX - EIGRP external, NM - NEMO 
ISIS summary, D 
ND Default, NDP 
ND Prefix, DCE 
Destination, NDr 
Re 
RPL, O - OSPF Intra, 01 - OSPF Inter, OEI - OSPF ext 1 
OSPF ext 2, ONI - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 
LISP alt, Ir 
LISP site-registrations, Id 
LISP dyn-ei 
LISP away, a - Application 
c 
L 
c 
L 
c 
L 
2001 . 
v la 
2øe1. 
via 
2001 . 
v la 
20e1. 
via 
2001 . 
v la 
2øe1. 
via 
via 
GigabitEthernete/Ø, directly connected 
GigabitEthernete/Ø, receive 
Seriale/ø/e, directly connected 
GigabitEthernete/Ø/e, receive 
GigabitEthernete/1/e, directly connected 
GigabitEthernete/1/e, receive 
NullØ, receive ">

Examples of Local IPv6 Routes

show ipv6 route local command

ck here to view code image 
RI# show ipv6 route local 
! Legend omitted for brevity 
L 
L 
L 
2001 . 
via 
2001 . 
via 
2001 . 
via 
r_FØe . 
via 
GigabitEthernete/Ø, receive 
Serialø/e/ø, receive 
GigabitEthernete/1/Ø, receive 
NullØ, receive ">

Static IPv6 Routes

static routes require direct configuration with the ipv6 route command

the ipv6 route command begins with the prefix and prefix length. Then the respective commands list the directions of how this router should forward packets toward that destination subnet or prefix by listing the outgoing interface or the address of the next-hop router.

A static route on R1, for this subnet, will begin with ipv6 route 2001:DB8:1111:2::/64, followed by either the outgoing interface (S0/0/0) or the next-hop IPv6 address, or both.

or Packets Destined to this Subnet 
Send out Here or... Send to There 
GO,'O 
so,'0/1 
Subnet 4 
GO,'O 
Subnet 2 
Figure 25-2 Logic Behind IPvS Static Route Commands (IPvS Route) ">

Static Routes Using the Outgoing Interface

when the command references an interface, the interface is a local interface

 Static route on router RI 
. .:/64 sø/Ø/0 ">

show ipv6 route command will list all the IPv6 routes.

show ipv6 route static command, which lists only static routes

I# show 
! Legend 
via 
ipv6 route static 
omitted for 
Seriale/Ø/e, 
brevity 
directly connected ">

directly connected” might mislead you to think this is a connected route; trust the “S” code.

show ipv6 route 2001:DB8:1111:2::22 command. This command asks the router to list the route that the router would use when forwarding packets to that particular address. Example 25-7 shows an example.

nown via &quot;static&quot; 
distance 1, 
Route count is 1/1, share count 
Routing paths: 
metric 
directly connected via Seria10/Ø/0 
Last updated ago ">

Static Routes Using Next-Hop IPv6 Address

Static IPv6 routes that refer to a next-hop address have two options: the unicast address on the neighboring router (global unicast or unique local) or the link-local address of that same neighboring router

ubnet 1 
For Packets Destined to this Subnet 
Send to Global Unicast 
Send to Link-Local 
GO/O 
Subnet 2 
Figure 25-3 Using Unicast or Link-Local as the Next-Hop Address for Static Routes ">

ck here to view code image 
RI# show 
! Legend 
s 
2001 . 
v la 
RI# show 
ipv6 route static 
omitted for brevity 
Known via &quot;static&quot; 
distance 1, 
Backup from &quot;ospf 1 [110] &quot; 
Route count is 1/1, share count 
Routing paths: 
Last updated ee:07 ago 
metric e ">

Example Static Route with a Link-Local Next-Hop Address

ipv6 route command cannot simply refer to a link-local next-hop address by itself because the link-local address does not, by itself, tell the local router which outgoing interface to use.

With a link-local next-hop address, a router cannot work through this same logic, so the outgoing interface must also be configured.

lick here to view code image 
! The first command is on router RI, listing R2's link-local address 
! The next command is on router R2, listing RI's link-local address ">

show commands also list both the next-hop (link-local) address and the outgoing interface

ick here to view code image 
RI# show ipv6 route static 
! Legend omitted for brevity 
s 
via FE8Ø: SerialØ/Ø/Ø 
Known via &quot;static&quot; 
distance 1, 
Backup from &quot;ospf 1 [110] &quot; 
Route count is 1/1, share count 
Routing paths: 
FE80: SerialO/Ø/Ø 
Last updated ago 
metric e ">

Static Routes over Ethernet Links

To configure a static route that uses an Ethernet interface, the ipv6 route command’s forwarding parameters should always include a next-hop IPv6 address. IOS allows you to configure the ipv6 route command using only the outgoing-interface parameter, without listing a next-hop address. The router will accept the command; however, if that outgoing interface happens to be an Ethernet interface, the router cannot successfully forward IPv6 packets using the route.

To configure the ipv6 route correctly when directing packets out an Ethernet interface, the configuration should use one of these styles:

Refer to the next-hop global unicast address (or unique local address) only

Refer to both the outgoing interface and next-hop global unicast address (or unique local address)

Refer to both the outgoing interface and next-hop link-local address

end to Global Unicast 
GO/O 
:11 
Subnet 1 
GO/I,'O 
Send to Neighbor Unicast and Out GONO • 
@For Packets Destined to this Subnet 
c 
GO/O 
Subnet 2 
Figure 254 Network Details for IPOS Static Routes on an Ethernet Interface ">

ick here to view code image 
! The first command is on router RI, listing R3's global unicast add 
! The next command is on router R3, listing RI's link-local address ">

Static Default Routes

Branch routers could use default routes instead of a routing protocol.

ore 
Default Route 
so,wo 
GO/I/O 
Default Route (::/O) 
Branch 
Offices 
GO/O/O 
Figure 25-5 Using Static Default Routes at Branches to Forward Back to the Core ">

use a specific value to note the route as a default route: ::/0

lick here to view code image 
IForward out 81' s Se/ø/l local interface. 
81 ipv6 route : SO/Ø/I ">

ck here to view code image 
BI# show ipv6 route static 
IPv6 Routing Table 
default 
le entries 
Codes: C 
IA 
o 
Connected, L 
Local, S 
Static, 
Per-user Static rou 
U 
- ISIS L2 
- BGP, R 
RIP, 11 
ISIS interarea, 
ISIS Ll, 12 
ISIS summary, D 
- EIGRP, EX - EIGRP 
ND Prefix, DCE 
Destination, NDr 
Re 
ND Default, 
OSPF Intra, 
NDP 
ONI - OSPF BISSA 
01 - OSPF Inter, OEI - OSPF ext 1, OE2 - OSPF 
ext 1, ON2 - OSPF NSSA ext 2 
s 
via Seriale/ø/l, directly connected ">

Static IPv6 Host Routes

a route to a single host IP address. With IPv4, those routes use a /32 mask, which identifies a single IPv4 address in the ip route command; with IPv6, a /128 mask identifies that single host in the ipv6 route command.

 The first command lists host B's address, prefix length /128, 
! with R2's link-local address as next-hop, with an outgoing interface 
. •:22/128 
sø/ø/e FE8Ø: 
RI (config)# 
! The next command also lists host B 's 
address, 
prefix length /128, 
! but with R2's global unicast address 
-hop, and no outgoing ir 
as next 
. •:22/128 ">

Floating Static IPv6 Routes

ackup Link (Tl; Static) 
Subnet 
Core Of the 
Enterprise 
Network ">

IOS considers static routes better than OSPF-learned routes by default due to administrative distance. IOS uses the same administrative distance concept and default values for IPv6 as it does for IPv4

IPv6 floating static route floats or moves into and out of the IPv6 routing table depending on whether the better (lower) administrative distance route learned by the routing protocol happens to exist currently.

To implement an IPv6 floating static route, just override the default administrative distance on the static route, making the value larger than the default administrative distance of the routing protocol.

ipv6 route 2001:db8:1111:7::/64 2001:db8:1111:9::3 130 command on R1 would do exactly that, setting the static route’s administrative distance to 130.

lick here to view code image 
RI# show ipv6 route static 
! Legend omitted for brevity 
s 
Known via &quot;static&quot; 
distance 130, metric 0 
Route count is 1/1, share count 0 
Routing paths: 
Last updated ago 
Table 25-2 lists some of the default administrative distance values used with IPv6. ">

oute Source 
Connected routes 
Static routes 
NDP 
EIGRP 
OSPF 
RIP 
Unknown or unbelievable 
Administrative Distance 
1 
2 
90 
110 
120 
255 ">

Troubleshooting Incorrect Static Routes That Appear in the IPv6 Routing Table

IOS cannot tell if you choose the incorrect outgoing interface, incorrect next-hop address, or incorrect prefix/prefix-length in a static route.

IOS puts the route into the IP routing table—even though the route may not work because of the poorly chosen parameters.

R1 cannot use its own IPv6 address as a next-hop address.

routes may have incorrect parameters. Check for these types of mistakes:

Step 1. Prefix/Length: Does the ipv6 route command reference the correct subnet ID (prefix) and mask (prefix length)?

Step 2. If using a next-hop IPv6 address that is a link-local address:

A. Is the link-local address an address on the correct neighboring router? (It should be an address on another router on a shared link.)

B. Does the ipv6 route command also refer to the correct outgoing interface on the local router?

Step 3. If using a next-hop IPv6 address that is a global unicast or unique local address, is the address the correct unicast address of the neighboring router?

Step 4. If referencing an outgoing interface, does the ipv6 route command reference the interface on the local router (that is, the same router where the static route is configured)?

The Static Route Does Not Appear in the IPv6 Routing Table

IOS makes the following checks before adding the route;

For ipv6 route commands that list an outgoing interface, that interface must be in an up/up state.

For ipv6 route commands that list a global unicast or unique local next-hop IP address (that is, not a link-local address), the local router must have a route to reach that next-hop address.

If another IPv6 route exists for that exact same prefix/prefix-length, the static route must have a better (lower) administrative distance.

The Neighbor Discovery Protocol

with 1pv4 ARP works as a separate protocol; with IPv6, the Neighbor Discovery Protocol (NDP), a part of ICMPv6, performs the same functions.

few of the functions of the NDP protocol (RFC 4861). Some of those NDP functions are

Neighbor MAC Discovery: An IPv6 LAN-based host will need to learn the MAC address of other hosts in the same subnet. NDP replaces IPv4’s ARP, providing messages that replace the ARP Request and Reply messages.

Router Discovery: Hosts learn the IPv6 addresses of the available IPv6 routers in the same subnet.

SLAAC: When using Stateless Address Auto Configuration (SLAAC), the host uses NDP messages to learn the subnet (prefix) used on the link plus the prefix length.

DAD: Before using an IPv6 address, hosts use NDP to perform a Duplicate Address Detection (DAD) process, to ensure no other host uses the same IPv6 address before attempting to use it.

Discovering Neighbor Link Addresses with NDP NS and NA

using a pair of matched solicitation and advertisement messages: the Neighbor Solicitation (NS) and Neighbor Advertisement (NA) messages.

the NS acts like an IPv4 ARP request, asking the host with a particular unicast IPv6 address to send back a reply. The NA message acts like an IPv4 ARP Reply, listing that host’s MAC address.

Neighbor Solicitation (NS): This message asks the host with a particular IPv6 address (the target address) to reply with an NA message that lists its MAC address. The NS message is sent to the solicited-node multicast address associated with the target address, so the message is processed only by hosts whose last six hex digits match the address that is being queried.

Neighbor Advertisement (NA): This message lists the sender’s IPv6 and MAC addresses. It can be sent in reply to an NS message, and if so, the packet is sent to the IPv6 unicast address of the host that sent the original NS message. A host can also send an unsolicited NA, announcing its IPv6 and MAC addresses, in which case the message is sent to the all-IPv6-hosts local-scope multicast address FF02::1.

NS 
PC2 
MAC 
I am MAC 
Figure 25-8 Example NDP NS/NA Process to Find the Neighbor's Link Addresses ">

lick here to view code image 
R3# show ipv6 neighbors 
IPv6 Address 
FE8Ø: 
Age 
e 
e 
Link-layer Addr 
0201. ame. oeel 
0201. ame. oeel 
State 
REACH 
REACH ">

To view a host’s NDP neighbor table, use these commands: (Windows) netsh interface ipv6 show neighbors; (Linux) ip -6 neighbor show; (Mac OS) ndp -an.

Discovering Routers with NDP RS and RA

NDP defines two messages that allow any host to discover all routers in the subnet:

Router Solicitation (RS): This message is sent to the “all-IPv6-routers” local-scope multicast address of FF02::2 so that the message asks all routers, on the local link only, to identify themselves.

Router Advertisement (RA): This message, sent by the router, lists many facts, including the link-local IPv6 address of the router. When sent in response to an RS message, it flows back to either the unicast address of the host that sent the RS or to the all-IPv6-hosts address FF02::1. Routers also send RA messages without being asked, sent to the all-IPv6-hosts local-scope multicast address of FF02::1.

Link-Local) 
RS 
All Routers—identify Yourselves 
I Am: 3: 
Figure 25-9 Example NDP RS/RA Process to Find the Default Routers 
RI 
2 ">

IPv6 allows multiple prefixes and multiple default routers to be listed in the RA message; Figure 25-9 just shows one of each for simplicity’s sake.

also periodically send unsolicited RA messages

the RA messages flow to the FF02::1 all-nodes IPv6 multicast address.

Using SLAAC with NDP RS and RA

IPv6 supports an alternative method for IPv6 hosts to dynamically choose an unused IPv6 address to use

The process goes by the name Stateless Address Autoconfiguration (SLAAC).

Learn the IPv6 prefix used on the link, from any router, using NDP RS/RA messages.

Build an address from the prefix plus an interface ID, chosen either by using EUI-64 rules or as a random value.

Before using the address, first use DAD to make sure that no other host is already using the same address.

refix 
Figure 25-10 Host IPv6 Address Formation Using SLA4C 
Chosen by Host 
Interface ID 
E UI-64 ">

Discovering Duplicate Addresses Using NDP NS and NA

IPv6 uses the Duplicate Address Detection (DAD) process before using a unicast address to make sure that no other node on that link is already using the address. Hosts use DAD not only at the end of the SLAAC process, but also any time that a host interface initializes, no matter whether using SLAAC, DHCP, or static address configuration. When performing DAD, if another host already uses that address, the first host simply does not use the address until the problem is resolved.

a host sends an NS message for its own IPv6 address. No other host should be using that address, so no other host should send an NDP NA in reply. However, if another host already uses that address, that host will reply with an NA, identifying a duplicate use of the address.

Hosts do the DAD check for each of their unicast addresses, link-local addresses included, both when the address is first used and each time the host’s interface comes up.

NDP Summary

NDP does more than what is listed in this chapter, and the protocol allows for addition of other functions, so NDP might continue to grow over time. For now, use Table 25-3 as a study reference for the four NDP features discussed here.

unction 
Router 
discovery 
Prefix/length 
discovery 
Neighbor 
discovery 
Duplicate Ad- 
dress Detection 
tocol 
sages 
RS and 
RS and 
NS and 
NS and 
Who 
Discov- 
ers Info 
Any 
IPv6 
host 
Any 
IPv6 
host 
Any 
IPv6 
host 
Any 
IPv6 
host 
Supplies 
Info 
Any IPv6 
router 
Any IPv6 
router 
Any IPv6 
host 
Any IPv6 
host 
Info Supplied 
Link-local IPv6 ad- 
dress of router 
Prefix(es) and associ- 
ated prefix lengths 
used on local link 
Link-layer address 
(for example, MAC 
address) used by a 
neighbor 
Simple confirmation 
whether a unicast ad- 
dress is already in use ">
