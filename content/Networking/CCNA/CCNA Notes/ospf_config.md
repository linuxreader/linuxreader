OSPF Config

This chapter covers the following exam topics:

3.0 IP Connectivity

3.2 Determine how a router makes a forwarding decision by default

3.2.b Administrative distance

3.2.c Routing protocol metric

3.4 Configure and verify single area OSPFv2

3.4.a Neighbor adjacencies

3.4.b Point-to-point

3.4.c Broadcast (DR/BR selection)

3.4.d Router ID

Implementing Single-Area OSPFv2

Step 1. Use the router ospf process-id global command to enter OSPF configuration mode for a particular OSPF process.

Step 2. (Optional) Configure the OSPF router ID by doing the following:

A. Use the router-id id-value router subcommand to define the router ID, or

B. Use the interface loopback number global command, along with an ip address address mask command, to configure an IP address on a loopback interface (chooses the highest IP address of all working loopbacks), or

C. Rely on an interface IP address (chooses the highest IP address of all working nonloopbacks).

Step 3. Use one or more network ip-address wildcard-mask area area-id router subcommands to enable OSPFv2 on any interfaces matched by the configured address and mask, enabling OSPF on the interface for the listed area.

OSPF Single-Area Configuration

for each matched interface, the router enables OSPF on those interfaces, discovers neighbors, creates neighbor relationships, and assigns the interface to the area listed in the network command.

Verifying OSPF Operation

The show ip ospf neighbor, show ip ospf database, and show ip route commands display information to match each of these three steps, respectively.

network and 
ip ospf Commands 
s Enabled 
- Interfaces 
Discover 
with Hello 
Neighbors 
Flood LSAs 
LSDB 
SPF Calculation 
RIB 
Admin Distance 
Routes 
Figure 20-3 OSPF Verification Commands 
show 
show 
show 
show 
Show 
show 
show 
show 
show 
show 
show 
show 
show 
running —config 
protocol s 
ip 
ip 
ip 
ip 
ip 
ip 
ip 
ospf 
ospf 
ospf 
ospf 
ospf 
ospf 
ospf 
interface 
interface type number 
interface brief 
neighbor 
neighbor type number 
database 
rib 
route 
route 
route 
ospf 
subnet mask 
section subnet ">

FULL/  -: The neighbor state is full, with the “-“ instead of letters meaning that the link does not use a DR/BDR.

FULL/DR: The neighbor state is full, and the neighbor is the DR.

FULL/BDR: The neighbor state is full, and the neighbor is the backup DR (BDR).

FULL/DROTHER: The neighbor state is full, and the neighbor is neither the DR nor BDR. (It also implies that the local router is a DR or BDR because the state is FULL.)

2WAY/DROTHER: The neighbor state is 2-way, and the neighbor is neither the DR nor BDR—that is, a DROther router. (It also implies that the local router is also a DROther router because otherwise the state would reach a full state.)

Verifying OSPF Configuration

If you have enable mode access, use the show running-config command to examine the configuration.

If you have only user mode access, use the show ip protocols command to re-create the OSPF configuration.

Use the show ip ospf interface \[brief\] command to determine whether the router enabled OSPF on the correct interfaces or not based on the configuration.

show ip ospf interface brief command shown here. It lists one line per interface, with the list showing all the interfaces on which OSPF has been enabled

the show ip ospf interface command with the brief keyword at the end lists a single line of output per interface, but the show ip ospf interface command (without the brief keyword) displays about 20 lines of output per interface

Configuring the OSPF Router ID

most enterprise networks that use OSPF choose to configure each router’s OSPF router ID

If the router-id rid OSPF subcommand is configured, this value is used as the RID.

If any loopback interfaces have an IP address configured, and the interface has an interface status of up, the router picks the highest numeric IP address among these loopback interfaces.

The router picks the highest numeric IP address from all other interfaces whose interface status code (first status code) is up. (In other words, an interface in up/down state will be included by OSPF when choosing its router ID.)

stops and restarts the OSPF process (with the clear ip ospf process command). So, if OSPF comes up, and later the configuration changes in a way that would impact the OSPF RID, OSPF does not change the RID immediately. Instead, IOS waits until the next time the OSPF process is restarted.

OSPF Interface Configuration Example

Step 1. Use the no network network-id area area-id subcommands in OSPF configuration mode to remove the network commands.

Step 2. Add one ip ospf process-id area area-id command in interface configuration mode under each interface on which OSPF should operate, with the correct OSPF process (process-id) and the correct OSPF area number.

Verifying OSPF Interface Configuration

The show ip protocols command relists most of the routing protocol configuration, so it does list some different details if you use interface configuration versus the network command

face Configuration 
Click here to view code image 
! First, with the new interface configuration 
RI* show ip ospf interface ge/0/0 
GigabitEthernete/0/e is up, line protocol is up 
Internet Address 10.1.12.1/24, Area e, Attached via Interface Enab 
! Lines omitted for brevity 
! For comparison, the old results with the use of the OSPF network c 
RI* show ip ospf interface ge/0/0 
GigabitEthernete/0/e is up, line protocol is up 
Internet Address 10.1.12.1/24, Area e, Attached via Network Statem 
. ending line omitted for brevity ">

Additional OSPFv2 Features

Passive interfaces

Default routes

Metrics

Load balancing

OSPF Passive Interfaces

When no routers exist on a link

OSPF continues to advertise about the subnet that is connected to the interface.

OSPF no longer sends OSPF Hellos on the interface.

OSPF no longer processes any received Hellos on the interface.

OSPF still advertises about the connected subnet, but OSPF also does not form neighbor relationships over the interface.

First, you can add the following command to the configuration of the OSPF process, in router configuration mode:

passive-interface type number

Alternately, the configuration can change the default setting so that all interfaces are passive by default and then add a no passive-interface command for all interfaces that need to not be passive:

passive-interface default

no passive-interface type number

Click here to view code image 
! First, make each subinterface passive directly 
router ospf 1 
passive-interface GigabitEthernetø/ø.1 
passive-interface GigabitEthernetø/0.2 
! Or, change the default to passive, and make the other interfaces n 
router ospf 1 
passive-interface default 
no passive-interface GigabitEthernetØ/Ø/0 
no passive-interface GigabitEthernetØ/1/0 
no passive-interface GigabitEthernetØ/2/0 ">

The show ip ospf interface brief command lists all interfaces on which OSPF is enabled, including passive interfaces.

The show ip ospf interface command lists a single line that mentions that the interface is passive.

OSPF Default Routes

All routers learn specific (nondefault) routes for subnets inside the company; a default route is not needed when forwarding packets to these destinations.

One router connects to the Internet, and it has a default route that points toward the Internet.

All routers should dynamically learn a default route, used for all traffic going to the Internet, so that all packets destined to locations in the Internet go to the one router connected to the Internet.

default-information originate command (Step 2) makes the router advertise a default route using OSPF to the remote routers (B1 and B2).

: default-information originate 
GO/I/O 
GO/O/O 
GO/O/O 
GO/I/O 
GO/3/0 
RI 
192.0.2.1 
Internet 
ISP 1 
ip route 0.0-0.0 0.O.O.0 192.0.2.1 
: default-information originate 
OSPFv2 Advertises Default 
Figure 20-6 Using OSPF to Create and Flood a Default Route ">

The default-information originate always router subcommand tells the router to always advertise the default route, no matter whether the router’s default route is working or not.

RI* show ip route static 
static, 
Codes: L - 
local, C - 
connected, S - 
! Rest of the legend omitted for brevity 
RIP, M 
mobile, B 
Gateway of last resort is 192.e.2.1 to network e. e. e. e 
e.e.e.e/e [254/0] via 192.0.2.1 ">

Bl# show ip route ospf 
Codes: L - 
D 
NI - 
El - 
static, R - 
local, C - 
connected, S - 
- EIGRP, EX - EIGRP external, O - OSPF, 
RIP, M - mobile, B 
IA - OSPF inter are 
OSPF NSSA external type 1, N2 - OSPF NSSA external type 
OSPF external type 1, E2 - OSPF external type 2 
! Rest of the legend omitted for brevity 
Gateway of last resort is 10.1.12.1 to network e. e. e. e 
O*E2 
o 
o 
e.e.e.e/e [110/1] via le.1.12.1, GigabitEthernete/1 
le.e.e.e/8 is variably subnetted, 6 subnets, 2 masks 
10.1.3.e/24 [lie/3] via le.1.12.1, GigabitEthern 
10.1.13.e/24 [110/2] via 10.1.12.1, GigabitEther ">

OSPF Metrics (Cost)

Cisco routers allow three different ways to change the OSPF interface cost:

Directly, using the interface subcommand ip ospf cost x.

Using the default calculation per interface, and changing the interface bandwidth setting, which changes the calculated value.

Using the default calculation per interface, and changing the OSPF reference bandwidth setting, which changes the calculated value.

Click here to view code image 
RI* conf t 
Enter configuration commands, one per 
RI (config)# interface gø,'0/0 
RI (config-if)# ip ospf cost 4 
RI (config-if)# interface go/ 1/0 
RI (config-if)# ip ospf cost 5 
RI (config-if)# AZ 
RI* show ip ospf interface brief 
line. 
End with CNTL/Z. 
Interface 
Gie/e.2 
Gie/e.l 
Gie/e/e 
Gie/l/e 
Gie/2/e 
PID 
1 
1 
1 
1 
1 
Area 
e 
e 
e 
e 
e 
IP 
le. 
Address/Mask 
Cost 
1 
1 
4 
5 
1 
State Nb 
le.1.2.1/24 
le.1.1.1/24 
le.1.12.1/24 
le.1.13.1/24 
14.1/24 
DR 
DR 
DR 
BDR 
DR 
1/ 
1/ 
1/ ">

The output also shows a cost value of 1 for the other Gigabit interfaces, which is the default OSPF cost for any interface faster than 100 Mbps.

Setting the Cost Based on Interface and Reference Bandwidth

interface bandwidth setting does not influence the actual transmission speed. Instead, the interface bandwidth acts as a configurable setting to represent the speed of the interface, with the option to configure the bandwidth to match the actual transmission speed…or not

configuration of the interface bandwidth using bandwidth speed interface subcommand.

Reference\_bandwidth / Interface\_bandwidth

you should avoid changing the interface bandwidth as a means to influence the default OSPF cost.

any interface with an interface bandwidth of 100 Mbps or faster ties with a calculated OSPF cost of 1 when using the default reference bandwidth. So, when relying on the default OSPF cost calculation, it helps to configure the reference bandwidth to another value.

Interface 
Serial 
Ethernet 
Fast Ethernet 
Gigabit 
Ethernet 
10 Gigabit 
Ethernet 
100 Gigabit 
Ethernet 
Interface Default Band- 
width (Kbps) 
1544 Kbps 
10,000 Kbps 
100,000 Kbps 
Kbps 
Kbps 
Kbps 
Formula (Kbps) 
100,000 / 1544 
100,000 / 10,000 
OSPF 
Cost 
64 
10 
1 
1 
1 
1 ">

You can still use OSPF’s default cost calculation (and many do) just by changing the reference bandwidth with the auto-cost reference-bandwidth speed OSPF mode subcommand

For instance, in an enterprise whose fastest links are 10 Gbps (10,000 Mbps), you could set all routers to use auto-cost reference-bandwidth 10000, meaning 10,000 Mbps or 10 Gbps. In that case, by default, a 10-Gbps link would have an OSPF cost of 1, while a 1-Gbps link would have a cost of 10, and a 100-MBps link a cost of 100.

Set the cost explicitly, using the ip ospf cost x interface subcommand, to a value between 1 and 65,535, inclusive.

Although it should be avoided, change the interface bandwidth with the bandwidth speed command, with speed being a number in kilobits per second (Kbps).

Change the reference bandwidth, using router OSPF subcommand auto-cost reference-bandwidth ref-bw, with a unit of megabits per second (Mbps).

OSPF Load Balancing

when the metrics tie for multiple routes to the same subnet, the router can put multiple equal-cost routes in the routing table (the default is four different routes) based on the setting of the maximum-paths number router subcommand

the default (and better) method, the load balancing could be on a per-destination IP address basis.
