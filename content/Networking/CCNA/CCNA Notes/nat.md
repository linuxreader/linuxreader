4.1 Configure and verify inside source NAT using static and pools

### CIDR

ISP3 
ISP4 
To 198. 
To 198._—_ 
To 198. 
Customer A 
198.8.3.0124 
Customer B 
198.0.o.o - 
198.4.2.0124 
198.255.255.0 
198.4.3.0124 
Customer C 
198.1.0.0/24 
Figure 10-1 Typical Use of CIDR ">

most public address assignments for the last 20 years have been a CIDR block rather than an entire class A, B, or C network.

### Private Addressing

Addresses 
10.0.0.0 to 
10.255.255.255 
172.16.O.O to 
172.31.255.255 
192.168.0.0 to 
192.168.255.255 
Network(s) 
10.0.0.0 
172.16.o.o - 
172.31.o.o 
192.168.0.0 - 
192.168.255.0 
Class of 
Networks 
c 
Number of 
Networks 
1 
16 
256 ">

no organization is allowed to advertise these networks using a routing protocol on the Internet.

Feature 
CIDR: 
NAT: 
Private 
Networks 
RFC(s) 
4632 
3022 
1918 
Main Benefits 
Assign more-specific public IPv4 address blocks to 
companies than Class A, B, and C networks. 
Aggregate routes to public IPv4 addresses based on 
worldwide address allocation plan. 
Enable approximately 65,000 TCP/UDP sessions to 
be supported by a single public IPv4 address. 
Enable the use of NAT for enterprise Internet con- 
nections, with private addresses used inside the 
enterprise. ">

### Network Address Translation Concepts

Client 
10.1.1.1 
Source 
10.1.1.1 
Source 
170.1.1.1 
Destination 
170.1.1.1 
NAT Changes 
Destination 
10.1.1.1 
Source 
200.1.1.1 
Source 
170.1.1.1 
170.1 .1 .1 
Destination 
170.1.1.1 
Destination 
200.1.1.1 
Figure 10-2 NAT IP Address Swapping: Private Addressing ">

### Static NAT

IP addresses statically mapped to each other.

10.1.12 
SA 10.1.1.1 
NAT 
Static NATTab1e 
Internet 
SA = 200.1.1.1 
Private Address Public Address 
10.1.1.1 
10.1.1.2 
200.1.1.1 
200.1.1.2 
Se rv'er 
170.1.1.1 
Legend 
SA: Source Address 
Figure 10-3 Static NAT Showing Inside Local and Global Addresses ">

NAT router simply configures a one-to-one mapping between the private address and the registered address that is used on its behalf.

Supporting a second IP host with static NAT requires a second static one-to-one mapping using a second IP address in the public address range.

the router statically maps 10.1.1.2 to 200.1.1.2. Because the enterprise has a single registered Class C network, it can support at most 254 private IP addresses with NAT, with the usual two reserved numbers (the network number and network broadcast address).

inside local for the private IP addresses in this example and inside global for the public IP addresses.

Outside 
10.1.1.1 
10.1.1.2 
10.1.1.1 
Inside Local 
10.1.1.1 
10.1.1.2 
NAT 
Internet 
SA =200.1.1.1 
170.1.1. 
Inside Global 
200.1.1.1 
200.1.1.2 
Figure 10-4 Static NAT Terminolog 
Legend ">

Inside 
local 
Inside 
global 
Values in 
Figures 
10.1.1.1 
200.1.1.1 
Meaning 
Inside: Refers to the permanent location of the 
host, from the enterprise's perspective: it is inside 
the enterprise. 
Local: Means not global; that is, local. It is the ad- 
dress used for that host while the packet flows in 
the local enterprise rather than the global Internet. 
Alternative: Think of it as inside private, because 
this address is typically a private address. 
Inside: Refers to the permanent location of the 
host, from the enterprise's perspective. 
Global: Means global as in the global Internet. It is 
the address used for that host while the packet 
flows in the Internet. 
Alternative: Think of it as inside public, because 
the address is typically a public IPv4 address. ">

global 
Outside 
local 
170.1.1.1 
With source NAT, the one address used by the host 
that resides outside the enterprise, which NAT does 
not change, so there is no need for a contrasting 
term. 
Alternative: Think of it as outside public, because 
the address is typically a public IPv4 address. 
This term is not used with source NAT. With desti- 
nation NAT, the address would represent a host that 
resides outside the enterprise, but the address used 
to represent that host as packets pass through the 
local enterprise. ">

### Dynamic NAT

Dynamic NAT sets up a pool of possible inside global addresses and defines matching criteria to determine which inside local IP addresses should be translated with NAT.

10.1.1.1 
Outside 
Internet 
NAT 
10.1.1.2 
NAT Table Before First Packet 
Server 
170.1.1.1 
Criteria for NAT: 
10.1.1. 
Inside Local _lnside Global 
NAT Table After First Packet 
Inside Local 
10.1.1 
Inside Global 
200.1.1.1 
NAT Pool: 
200.1.1.1 
200.1.1.2 
200.1.1.3 
200.1.1.4 
200.1.1.5 
Figure 10-5 Dynamic NAT ">

Host 10.1.1.1 sends its first packet to the server at 170.1.1.1.

As the packet enters the NAT router, the router applies some matching logic to decide whether the packet should have NAT applied. Because the logic has been configured to match source IP addresses that begin with 10.1.1, the router adds an entry in the NAT table for 10.1.1.1 as an inside local address.

The NAT router needs to allocate an IP address from the pool of valid inside global addresses. It picks the first one available (200.1.1.1, in this case) and adds it to the NAT table to complete the entry.

The NAT router translates the source IP address and forwards the packet.

The dynamic entry stays in the table as long as traffic flows occasionally. You can configure a timeout value that defines how long the router should wait, having not translated any packets with that address, before removing the dynamic entry. You can also manually clear the dynamic entries from the table using the clear ip nat translation \* command.

NAT can be configured with more IP addresses in the inside local address list than in the inside global address pool.

If a new packet arrives from yet another inside host, and it needs a NAT entry, but all the pooled IP addresses are in use, the router simply discards the packet. The user must try again until a NAT entry times out, at which point the NAT function works for the next host that sends a packet. Essentially, the inside global pool of addresses needs to be as large as the maximum number of concurrent hosts that need to use the Internet at the same time—unless you use PAT, as is explained in the next section.

### Overloading NAT with Port Address Translation

NAT Overload feature, also called Port Address Translation (PAT)

170.1.1.1 Port 80 
10.1.1.2 
10.1.1.3 
10.1.1 1, Port49724 
10.1.1.2, port 49724 
10.1.1.3, Port 49733 
170111 
170111 
port 80 
Port 80 
Server 
170.1.1.1 
Figure 10-6 Three TCP Connectionsfrom Three PCs ">

10.1.1.1 
170.1.1 1 Port80 
10.1.1.2 
10.1.1 2 
10.1 .1 .3 
10.1.1.3 
170.1.1 1 Port 80 
Port 49724 
port 49724 
Port 49733 
NAT 
200.1.1.2 
Port 49724 
200.1.1.2, Port 49725 
200.1.1.2 
Port 49726 
170.1.1.1 Port 80 
Server 
170.1.1.1 
Inside Local 
10.1.1.1: 49724 
10.1.12: 49724 
10.1.1.3: 49733 
Inside Global 
200.1.1 .2: 49724 
200.1.12: 49725 
200.1.1 .2: 49726 
Dynamic NAT Table, With Overloading 
Figure 10-8 NAT Overload (PAT) ">

The NAT router keeps a NAT table entry for every unique combination of inside local IP address and port, with translation to the inside global address and a unique port number associated with the inside global address.

NAT overload can use more than 65,000 port numbers

PAT is by far the most popular option

## NAT Configuration and Troubleshooting

### Static NAT Configuration

Each static mapping between a local (private) address and a global (public) address must be configured.

Those same interface subcommands tell NAT whether the interface is inside or outside.

Step 1. Use the ip nat inside command in interface configuration mode to configure interfaces to be in the inside part of the NAT design.

Step 2. Use the ip nat outside command in interface configuration mode to configure interfaces to be in the outside part of the NAT design.

Step 3. Use the ip nat inside source static inside-local inside-global command in global configuration mode to configure the static mappings.

Certskills 
200.1.1.252 
10.1.1.1 
SO/O/O 
10.1.1.2 
Inside 
Internet 
rve 
170.1.1.1 
Outside 
Static NAT Confi 
Inside Local 
10.1.1.1 
10.1.1.2 
ration 
Inside Global 
200.1.1.1 
200.1.1.2 
Figure 10-9 Sample Networkfor NAT Examples, with Public Class C 200.1.1.0/24 ">

! Lines omitted for brevity 
interface GigabitEthernet0/Ø 
ip address 10.1.1.3 255.255.255.0 
ip nat inside 
interface SerialØ/0/0 
ip address 200.1.1.251 255.255.255.0 
ip nat outside 
ip nat inside source static 10.1.1.2 200.1.1.2 
ip nat inside source static 10.1.1.1 200.1.1.1 
NAT # show ip nat translations 
Pro Inside global Inside local 
Outside local 
200.1.1.1 
200.1.1.2 
10.1.1.1 
10.1.1.2 
dynamic; 
Outside glo 
0 extended) 
NAT # show ip nat statistics 
Total active translations: 2 (2 static, 
Outside interfaces: 
Seria10/0/Ø 
Inside interfaces: 
GigabitEthernetØ/Ø 
Hits: 100 Misses: 0 
Expired translations ">

static mappings are created using the ip nat inside source static command.

Source keyword means that NAT translates the source IP address of packets coming into its inside interfaces. The static keyword means that the parameters define a static entry, which should never be removed from the NAT table because of timeout. Because the design calls for two hosts—10.1.1.1 and 10.1.1.2—to have Internet access, two ip nat inside commands are needed.

show ip nat translations command lists the two static NAT entries created in the configuration. The show ip nat statistics command lists statistics, listing things such as the number of currently active translation table entries. The statistics also include the number of hits, which increments for every packet for which NAT must translate addresses.

### Dynamic NAT Configuration

Dynamic NAT still requires that each interface be identified as either an inside or outside interface

Dynamic NAT uses an access control list (ACL) to identify which inside local (private) IP addresses need to have their addresses translated, and it defines a pool of registered public IP addresses to allocate.

Step 1. Use the ip nat inside command in interface configuration mode to configure interfaces to be in the inside part of the NAT design (just like with static NAT).

Step 2. Use the ip nat outside command in interface configuration mode to configure interfaces to be in the outside part of the NAT design (just like with static NAT).

Step 3. Configure an ACL that matches the packets entering inside interfaces for which NAT should be performed.

Step 4. Use the ip nat pool name first-address last-address netmask subnet-mask command in global configuration mode to configure the pool of public registered IP addresses.

Step 5. Use the ip nat inside source list acl-number pool pool-name command in global configuration mode to enable dynamic NAT. Note the command references the ACL (step 3) and pool (step 4) per previous steps.

Click here to view code image 
NAT# show running-config 
! Lines omitted for brevity 
interface GigabitEthernet0/0 
ip address 10.1.1.3 255.255.255.0 
ip nat inside 
interface Seria10/0/0 
ip address 200.1.1.251 255.255.255.e 
ip nat outside 
10.1.1.2 
10.1.1.1 
ip nat 
ip nat 
access 
access 
pool fred 200. 
1.1.1 200.1.1.2 netmask 255.255.255.252 
inside 
-list 1 
-list 1 
source 
permit 
permit 
list 
1 
pool fred ">

Click here to view code image 
The next command lists one empty line because no entries have been 
! created yet. 
NAT# show ip nat translations 
NAT# show ip nat statistics 
Total active translations: e (e static, e dynamic; 
Peak translations: 8, occurred ago 
e extended) 
Outside interfaces: 
seriale/e/e 
Inside interfaces: 
GigabitEthernete/e 
Hits: e Misses: e 
CEF Translated packets: e, 
Expired translations: e 
Dynamic mappings: 
- Inside Source 
CEF Punted packets: 
e 
[id 1] access-list 1 pool fred refcount e 
pool Fred: net-mask 255.255.255.252 
start 2ee.1.1.1 end 2ee.1.1.2 
type generic, total addresses 2, 
Total doors: e 
Appl doors: e 
Normal doors: e 
allocated 0 (8%), 
misses e ">

“misses,” as highlighted in the example. The first occurrence of this counter counts the number of times a new packet comes along, needing a NAT entry, and not finding one.

The second misses counter toward the end of the command output lists the number of misses in the pool. This counter increments only when dynamic NAT tries to allocate a new NAT table entry and finds no available addresses

Click here to view code image 
NAT# show ip nat translations 
Pro Inside global 
- 2ee.1.1.1 
Inside 
local 
le.l.l.l 
Outside local 
Outsic 
NAT# show ip nat statistics 
Total active translations: 1 (e static, 1 dynamic; 
Peak translations: 11, occurred ago 
e extended) 
Outside interfaces: 
seriale/e/e 
Inside interfaces: 
GigabitEthernete/e 
Hits: 69 Misses: 1 
Expired translations: 
Dynamic mappings: 
- Inside Source 
e 
access-list 1 pool fred refcount 1 
[eml Fred: net-mask 255.255.255.252 
start 2ee.1.1.1 end 2ee.1.1.2 
type generic, total addresses 2, 
allocated 1 (59%), 
misses e ">

Click here to view code imac 
! Host 10.1.1.1 currently uses inside global 200.1.1.1 
NAT# show ip nat translations 
19:23 
Pro Inside global 
- 200.1.1.1 
1 
Inside loca 
10.1.1.1 
Outside 
NAT# clear ip nat translation * 
! telnet from 10.1.1.2 to 170.1.1.1 happened next; 
! Now host 10.1.1.2 uses inside global 200.1.1.1 
NAT# show ip nat translations 
Pro Inside global 
- 200.1.1.1 
! Telnet from 10.1.1.1 
NAT# debug ip nat 
Inside 
local 
10.1.1.2 
to 170 
Outside 
local 
not shown 
local 
not shown 
IP NAT 
Oct 
Oct 
Oct 
Oct 
20 
20 
20 
20 
debugging is on 
19:23 : 03 . 263: 
19:23 267: 
: 03 
568: 
.1.1.1 happened next; 
s-170.1.1.1, d=2øø.1.1 
s-170.1.1.1, d=2øø.1.1 
d=17e .1.1 
.2->1e.1.1 
d=17e .1.1 
.2->1e.1.1 
.1 
.1 
.1 
.1 
Outsid 
Outsid 
c 348] 
c 348] 
[349] 
[349] ">

debug ip nat command. This debug command causes the router to issue a message every time a packet has its address translated for NAT.

### NAT Overload (PAT) Configuration

If PAT uses a pool of inside global addresses, the configuration looks exactly like dynamic NAT, except the ip nat inside source list global command has an overload keyword added to the end. If PAT just needs to use one inside global IP address, the router can use one of its interface IP addresses

configuration when using an interface IP address as the sole inside global IP address:

Step 1. As with dynamic and static NAT, configure the ip nat inside interface sub-command to identify inside interfaces.

Step 2. As with dynamic and static NAT, configure the ip nat outside interface subcommand to identify outside interfaces.

Step 3. As with dynamic NAT, configure an ACL that matches the packets entering inside interfaces.

Step 4. Configure the ip nat inside source list acl-number interface type/number overload global configuration command, referring to the ACL created in step 3 and to the interface whose IP address will be used for translations.

10.1.1.1 
10.1.1.2 
Inside 
200.1.1.249 
200.1.1.250 
Internet 
NAT Table overload 
Inside Local 
Inside Global 
10.1.1.1: 49712 200.1.1.249: 49712 
10.1.1.2: 49713 200.1.1_249: 49713 
10.1.1.2: 49913 200.1.1.249:49913 
Figure 10-10 NAT Overload and PAT 
170.1.1.1 
Outside ">

Click to im 
NAT* show running-config 
Lines Omitted for Brevity 
interface GigabitEthernetØ/ø 
ip address 10.1.1.3 255.255.255.a 
ip nat inside 
interface Seriale/e/a 
ip address 200.1.1.249 255.255.255.252 
ip nat outside 
ip nat inside source list I interface Seriale/ø/ø overload 
access-list I permit 10.1.1.2 
access-list I permit 10.1.1.1 
dynamic; 3 extended) 
NAT* 
Pro 
tcp 
tcp 
tcp 
NAT* 
show ip nat translations 
Inside global 
Inside 
local 
10.1.1.1:49712 
10.1.1.2:49713 
10.1.1.2:49913 
show ip nat statistics 
Out s ide 
local 
17e.1.1.1:23 
17e.1.1.1:23 
17e.1.1.1:23 
Outsid 
170.1. 
170.1. 
170.1. 
Total active translations: 3 (e static, 
Peak translations: 12, occurred ago 
Outside interfaces: 
Serialø/ø/ø 
Inside interfaces: 
Gigabit Ethernete/a 
Hits: IE Misses: 3 
Expired translations . 
Dynamic mappings: 
Inside Source 
access-list I interface Seriale/e/a refcount 3 ">

ip nat inside source list 1 interface serial 0/0/0 overload command has several parameters

The list 1 parameter means the same thing as it does for dynamic NAT: inside local IP addresses matching ACL 1 have their addresses translated. The interface serial 0/0/0 parameter means that the only inside global IP address available is the IP address of the NAT router’s interface serial 0/0/0.

The router creates one NAT table entry for each unique combination of inside local IP address and port

NAT Troubleshooting

Reversed inside and outside

Static NAT: Check the ip nat inside source static command to ensure it lists the inside local address first and the inside global IP address second.

Dynamic NAT (ACL): Ensure that the ACL configured to match packets sent by the inside hosts match that host’s packets before any NAT translation has occurred.

Dynamic NAT (pool): For dynamic NAT without PAT, ensure that the pool has enough IP addresses\

A large or growing value in the second misses counter in the show ip nat statistics command output can indicate this problem. Also, compare the configured pool to the list of addresses in the NAT translation table (show ip nat translations). Finally, if the pool is small, the problem may be that the configuration intended to use PAT and is missing the overload keyword

PAT: It is easy to forget to add the overload option

perhaps NAT has been configured correctly, but an ACL exists on one of the interfaces, discarding the packets.

IOS processes ACLs before NAT. For packets exiting an interface, IOS processes any outbound ACL after translating the addresses with NAT.

User traffic required

IPv4 routing
