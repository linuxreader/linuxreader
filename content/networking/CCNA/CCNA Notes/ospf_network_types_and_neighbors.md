This chapter covers the following exam topics:

3.0 IP Connectivity

3.4 Configure and verify single area OSPFv2

3.4.a Neighbor adjacencies

3.4.b Point-to-point

3.4.c Broadcast (DR/BDR selection)

3.4.d Router ID

OSPF Network Types

Network Type 
Keyword 
broadcast 
point-to-point 
Dynamically Discovers 
Neighbors 
Yes 
Yes 
Uses a 
DR/BDR 
Yes 
No ">

The OSPF Broadcast Network Type

default to use network type broadcast

Attempt to discover neighbors by sending OSPF Hellos to the 224.0.0.5 multicast address (an address reserved for sending packets to all OSPF routers in the subnet)

Attempt to elect a DR and BDR on each subnet

On the interface with no other routers on the subnet (G0/1), become the DR

On the interface with three other routers on the subnet (G0/0), be either DR, BDR, or a DROther router

all discovered routers on the link should become neighbors and at least reach the 2-way state. For all neighbor relationships that include the DR and/or BDR, the neighbor relationship should further reach the full state. That section defined the term fully adjacent as a special term that refers to neighbors that reach this full state.

To see the setting, use the show ip ospf interface command, as shown in Example 21-4. The first highlighted item identifies the network type

Click here to view code image 
RI* show ip ospf interface ge/ø 
GigabitEthernete/e is up, line protocol is up 
Internet Address 10.1.1.1/24, Area e, Attached via Interface Enabl 
Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, cost: 1 
Topology-MTID 
e 
Cost 
1 
Disabled 
no 
Shutdown 
no 
Topology Name 
Base 
Enabled by interface config, including secondary ip addresses 
Transmit Delay is 1 sec, State DROTHER, Priority 1 
Designated Router (ID) 4.4.4.4, Interface address 10.1.1.4 
Backup Designated router (ID) 3.3.3.3, Interface address 10.1.1.3 
Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 
oob-resync timeout 40 
Hello due in ee:ee:ee 
Supports Link-local Signaling (LLS) 
Cisco NSF helper support enabled 
IETF NSF helper support enabled 
Index 1/1/1, flood queue length e 
Next 
Last flood scan length is e, maximum is 1 
Last flood scan time is e msec, maximum is e 
Nei ghhnr r Olin* i e R Ad-iarpnt- nei ghhnr c-nllnt- 
msec ">

Adjacent with neighbor 4.4.4.4 (Designated Router) 
Suppress hello for e neighbor(s) ">

ip ospf network broadcast interface subcommand would configure the setting.

Configuring to Influence the DR/BDR Election

If the DR fails, the BDR becomes the DR, and a new BDR is elected.

When a better router enters the subnet, no preemption of the existing DR or BDR occurs.

- Preemption: If a new router is configured to be the DR, it will not become the DR until the OSPF process is reset.

As a result of these rules, while you can configure a router to be the best (highest priority) router to become the DR in an election, doing so only increases that router’s statistical chances of being the DR at a given point in time. If the router fails, other routers will become DR and BDR, and the best router will not be DR again until the current DR and BDR fail.

The highest OSPF interface priority: The highest value wins during an election, with values ranging from 0 to 255.

The highest OSPF Router ID: If the priority ties, the election chooses the router with the highest OSPF RID.

If an engineer preferred that R1 be the DR, the engineer could add the configuration in Example 21-5 to set R1’s interface priority to 99. (default of 1)

RI* configure terminal 
Configuring from terminal, memory, or network [terminal]? 
Enter configuration commands, one per line. 
End with CNTL/Z. 
RI (config)# interface gø,'ø 
RI (config-if)# ip ospf priority 99 
RI (config-if)# AZ 
RI* show ip ospf interface ge/ø I include Priority 
Transmit Delay is 1 sec, State DROTHER, Priority 99 
10.1.1.2 
10.1.1.3 
10.1. 
1.4 
RI* show ip 
Neighbor ID 
2.2. 2.2 
3.3. 3.3 
4.4.4.4 
RI* show ip 
Interface 
Gie/l 
Gie/e 
ospf neighbor 
pri 
1 
1 
1 
State 
2WAY/DROTHER 
FULL/BDR 
FULL/DR 
Dead Time 
ospf 
PID 
1 
1 
interface brief 
Area 
e 
e 
Address/Mask 
IP 
le.1.11.1/24 
le.1.1.1/24 
Address 
Cost 
1 
1 
State 
DR 
DROTH 
In 
2/ ">

show that the DR and BDR have not changed at all

The OSPF Point-to-Point Network Type

(HDLC and PPP) do not support data-link broadcasts.

to use OSPF network type point-to-point. R2, on the other end of the WAN link, would need the same configuration command on its matching interface.

RI 
Click here to view code image 
RI* configure terminal 
Enter configuration commands, one per line. 
End with CNTL/Z. 
RI (config)# interface ge/Ø/0 
RI (config-if)# ip ospf network point-to-point 
RI (config-if)# 
RI* show ip ospf interface ge/0/0 
GigabitEthernete/0/e is up, line protocol is up 
Internet Address 10.1.12.1/24, Area e, Attached via Interface Enab 
Process ID 1, Router ID 1.1.1.1, Network Type POINT TO POINT, cost 
Topology-MT ID 
e 
Cost 
4 
Disabled 
no 
Shutdown 
no 
Topology Name 
Base 
Enabled by interface config, including secondary ip addresses 
Transmit Delay is 1 sec, State POINT TO POINT 
Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 
oob-resync timeout 40 
Hello due in ee:ee:el 
Supports Link-local Signaling (LLS) 
Cisco NSF helper support enabled ">

Index 1/3/3, flood queue length e 
Next 
Last flood scan length is 1, maximum is 3 
Last flood scan time is e msec, maximum is e msec 
Neighbor Count is 1, Adjacent neighbor count is 1 
Adjacent with neighbor 2.2.2.2 
Suppress hello for e neighbor(s) ">

RI 
Click here to view code image 
show ip ospf neighbor 
eighbor ID 
.2.2.2 
pri 
e 
State 
FULL/ 
Dead Time 
lines omitted for brevity 
show ip ospf interface brief 
nterface 
ie/e/e 
PID 
1 
Area 
e 
Address/Mask 
IP 
le.1.12.1/24 
Address 
10.1. 12.2 
Cost 
4 
Interfac 
GigabitE 
State Nbrs F 
1/1 
P2P 
lines omitted for brevity ">

OSPF Neighbor Relationships

They must have compatible values for several settings as listed in the Hellos exchanged between the two routers

Requirement 
Interfaces must be in an up/up state. 
Access control lists (ACL) must not filter 
routing protocol messages. 
Interfaces must be in the same subnet. 
They must pass routing protocol neigh- 
bor authentication (if configured). 
Hello and hold/dead timers must match. 
Router IDs (RID) must be unique. 
They must be in the same area. 
OSPF process must not be shut down. 
Required 
for OSPF 
Yes 
Yes 
Yes 
Yes 
Yes 
Yes 
Yes 
Yes 
Neighbor Missing 
if Incorrect 
Yes 
Yes 
Yes 
Yes 
Yes 
Yes 
Yes 
Yes ">

MTU setting. 
Neighboring interfaces must use same 
OSPF network type. 
Yes 
Yes 
No 
No ">

For items listing a “yes” in this column, if that item is configured incorrectly, the neighbor will not appear in lists of OSPF neighbors—for instance, with the show ip ospf neighbor command.

the last section (shaded) lists a couple of OSPF settings that give a different symptom when incorrect

for these two items, when incorrect, a router can list the other router as a neighbor, but the neighbor relationship does not work properly in that the routers do not exchange LSAs as they should

Requirement 
Hello and dead timers must match. 
They must be in the same area. 
RIDs must be unique. 
They must pass any neighbor 
authentication. 
OSPF process must not be shut down. 
Best show Command 
show ip ospf interface 
show ip ospf interface brief 
show ip ospf 
show ip ospf interface 
show ip ospf, show ip ospf 
interface ">

10/40 is the default hello/dead timer

Finding Area Mismatches

Check the output of show running-config to look for

ip ospf process-id area area-number interface subcommands

network commands in OSPF configuration mode

Use the show ip ospf interface \[brief\] command to list the area number

Finding Duplicate OSPF Router IDs

both routers automatically generate a log message for the duplicate OSPF RID problem between R1 and R3;

show ip ospf commands on both R3 and R1 to easily list the RID on each router,

use the router-id 3.3.3.3 OSPF subcommand and use the EXEC mode command clear ip ospf process. (OSPF will not begin using a new RID value until the process restarts, either via command or reload.)

Finding OSPF Hello and Dead Timer Mismatches

Hello interval/timer: The per-interface timer that tells a router how often to send OSPF Hello messages on an interface.

Dead interval/timer: The per-interface timer that tells the router how long to wait without having received a Hello from a neighbor before believing that neighbor has failed. (Defaults to four times the Hello timer.)

Click here to view code image 
RI* show ip ospf interface GO/O 
GigabitEthernete/e is up, line protocol is up 
Internet Address 10.1.1.1/24, Area e, Attached via Network Stateme 
Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, cost: 1 
Topology-MT ID Cost Disabled Shutdown 
e 
1 
no 
no 
Topology Name 
Base 
Transmit Delay is 1 sec, State DR, Priority 1 
Designated Router (ID) 1.1.1.1, Interface address 10.1.1.1 
No backup designated router on this network 
Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 
lines omitted for brevity ">

R4# show ip ospf interface Gi€/€ 
GigabitEthernete/e is up, line protocol is up 
Internet Address le.1.1.4/24, Area e, Attached via Network Statem 
Process ID 4, Router ID le.1.44.4, Network Type BROADCAST, cost: 
Topology-MT ID Cost Disabled Shutdown 
e 
1 
no 
no 
Topology Name 
Base 
Transmit Delay is 1 sec, State DR, Priority 1 
Transmit Delay is 1 sec, State DR, Priority 1 
Designated Router (ID) 10.1 
No backup designated router 
Timer intervals configured, 
lines omitted for brevity 
.44.4, Interface address 10.1.1.4 
on this network 
Hello 5, Dead 20, Wait 2e, Retransmit ">

Shutting Down the OSPF Process

When a routing protocol process is shut down, IOS does the following:

Brings down all neighbor relationships and clears the OSPF neighbor table

Clears the LSDB

Clears the IP routing table of any OSPF-learned routes

IOS retains all OSPF configuration.

IOS still lists all OSPF-enabled interfaces in the OSPF interface list (show ip ospf interface) but in a DOWN state.

Mismatched MTU Settings

defining the largest network layer packet that the router will forward out each interface.

default MTU size of 1500 bytes,

The ip mtu size interface subcommand defines the IPv4 MTU setting, and the ipv6 mtu size command sets the equivalent for IPv6 packets.

Symptom: they fail to exchange their LSDBs. Eventually, after trying and failing to exchange their LSDBs, the neighbor relationship also fails.

show interfaces command (which lists the IP MTU).

Mismatched OSPF Network Types

one router uses broadcast, and the other uses point-to-point, the following occurs:

The two routers become fully adjacent neighbors (that is, they reach a full state).

They exchange their LSDBs.

They do not add IP routes to the IP routing table.
