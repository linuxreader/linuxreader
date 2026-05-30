RSTP is the most common spanning tree protocol. And Cisco devices default to using RSTP.

MAC table instability

-   The switches MAC address tables keep changing because frames with the same source MAC arrive on different ports.

Broadcast storms

-   forwarding of a frame repeatedly on the same links

Multiple frame transmission

-   side effect of looping frames
-   Multiple copies are delivered to a host, confusing the host.

What Spanning Tree Does

-   blocking state

-   interfaces does not process any frames
-   except STP/RSTP messages and some other overhead messages

STP Convergence

-   switches collectively realize that something has changed in the LAN topology
-   determine whether they need to change which ports block and which port forward

How Spanning Tree works

three criteria to choose whether to put and interface in forwarding state:

elect a root switch.

-   STP puts all working interfaces on the root switch in forwarding state

nonroot switches

-   select the port with the least administrative cost (root port)

-   cost between itself and the root switch (root cost) (root cost path)
-   root port (RP) gets put in a forwarding state

-   The switch with the lowest root cost, as compared with the other switches attached to the same link, is placed in forwarding state.

-   That switch is the designated switch, and that switch's interface, attached to that segment, is called the designated port (DP)

STP States

All the root switches ports

-   forwarding
-   root switch is always the designated switch

All nonroot switch's root ports

-   forwarding
-   port with the least cost to the root switch (lowest root cost)

Each LAN's designated port

-   forwarding
-   switch forwarding the Hello on to the segment with the lowest root cost is the designated switch for the segment

All other working ports

-   blocking
-   Not used for forwarding frames
-   frames received on these interfaces to not forward

The STP Bridge ID and Hello BPDU

-   bridge ID (BID)

-   8-byte value unique to each switch.
-   2-byte priority field
-   6-byte system ID

-   based on a universal (burned-in) MAC address

-   bridge protocol data units (BPDU)

-   configuration BPDUs, which switches
-   used to exchange information with each other (switches) The most common BPDU, called a
-   hello BPDU,

-   sending switch's BID

-   switches can tell which switch sent which Hello BPDU

-   Root Bridge ID

-   BID the sender currently believes to be the root switch

-   Sender's root cost

-   STP cost between this switch and current root

-   Timer values on the root switch

-   Hello timer, MaxAge timer, and forward delay timer

Electing the Root Switch

-   based on the BIDs in the BPDUs.
-   root switch is the switch with the lowest numeric value for the BID.
-   Because the two-part BID starts with the priority value,

-   essentially the switch with the lowest priority becomes the root.

-   If a tie occurs based on the priority portion of the BID,

-   the switch with the lowest MAC address portion of the BID is the root.

-   Mac addresses are the second part of the BID

-   all switches claim to be the root by sending Hello BPDUs listing their own BID as the root BID.
-   If a switch hears a Hello that lists a better (lower) BID

-   that switch stops advertising itself as root and starts forwarding the superior Hello.

-   The Hello sent by the better switch lists the better switch’s BID as the root.

-   superior hello (better hello)

-   the listed root’s BID is better (numerically lower),

-   Inferior hello (worse Hello), meaning that

-   the listed root’s BID is not as good (numerically higher)

-   each nonroot switch chooses its one and only root port. A switch’s RP is its interface through which it has the least STP/RSTP cost to reach the root switch (least root cost).

Choosing Each Switch's Root Port

-   each nonroot switch chooses its one and only root port.

-   its interface through which it has the least STP/RSTP cost to reach the root switch (least root cost).

-   The STP/RSTP port cost is simply an integer value assigned to each interface, per VLAN

-   The switches also look at their neighbor’s root cost, as announced in Hello BPDUs received from each neighbor.

![Root Cost = O 
SWI 
GO/I 
GiO'2 
Root Cost = O 
RootCostoutGO/1 is 5 
Root Cost out GO,'2 is 8 
0+5=5 
cost 5 
Cost = 4 
GiO,Q 
SW3 
4 
Cost 4 
GiO,Q 
My Root Cost 
Out GOO is 4 
sv-V2 
GiO'1 
Root Cost = 4 

-   SW3 calculates its cost to reach the root over the two possible paths by adding the advertised cost (in Hello messages) to the interface costs listed in the figure.

-   The root switch sends Hellos, with a listed root cost of 0. The idea is that the root’s cost to reach itself is 0.

-   Each switch places its root port into a forwarding state.

-   tiebreaker to use in case the best root cost ties for two or more paths.

-   Choose based on the lowest neighbor bridge ID.
-   Choose based on the lowest neighbor port priority.
-   Choose based on the lowest neighbor internal port number.

Choosing the Designated Port on each LAN Segment

-   final step to choose the STP/RSTP topology is to choose the designated port on each LAN segment.
-   The designated port (DP) on each LAN segment is the switch port that advertises the lowest-cost Hello onto a LAN segment.
-   When a nonroot switch forwards a Hello, the nonroot switch sets the root cost field in the Hello to that switch’s cost to reach the root. In effect, the switch with the lower cost to reach the root, among all switches connected to a segment, becomes the DP on that segment.

-   All DPs are placed into a forwarding state

Two additional tiebreakers are needed in some cases

-   these would be unlikely today.
-   A single switch can connect two or more interfaces to the same collision domain by connecting to a hub.

-   In that case, the one switch hears its own BPDUs.
-   So, if a switch ties with itself, two additional tiebreakers are used:

-   the lowest interface STP/RSTP priority and,
-   if that ties, the lowest internal interface number.

-   switch ports connected to endpoint devices should become DPs and settle into a forwarding state.

Configuring to influence the STP Topology

-   configure the bridge ID and  change STP/RSTP port costs.

-   change the BID, the engineer can

-   set the priority used by the switch, while
-   continues to use the universal MAC address as the final 48 bits of the BID.

-   giving a switch the lowest priority value among all switches will cause that switch to win the root election.

-   to favor one link, give the ports on that link a lower cost, or to avoid a link, give the ports a higher cost.

Link Costs

10 Mbps

-   2,000,000
-   100 (old)

100Mbps

-   200,000
-   19 (old)

1gbps

-   20,000
-   4 (old)

10Gbps

-   2000
-   2 (old)

100Gbps

-   200
-   N/A (old)

1Tbps

-   20
-   N/A (old)

-   the cost defaults based on the operating speed of the link, not the maximum speed

-   (config) # spanning-tree pathcost method long

-   Cisco Catalyst switches can be configured to use the long values as defaults

Details specific to STP

STP Activity When the Network Remains Stable

-   An STP root switch sends a new Hello BPDU every 2 seconds by default.
-   Each nonroot switch forwards the Hello on all DPs, but only after changing items listed in the Hello.
-   (As a result, the Hello flows once over every working link in the LAN.)

-   When forwarding the Hello BPDU, each switch sets the root cost to that local switch’s calculated root cost. The switch also sets the “sender’s bridge ID” field to its own bridge ID. (The root’s bridge ID field is not changed.)

Step 1. The root creates and sends a Hello BPDU, with a root cost of 0, out all its working interfaces (those in a forwarding state).

Step 2. The nonroot switches receive the Hello on their root ports. After changing the Hello to list their own BID as the sender’s BID and listing that switch’s root cost, the switch forwards the Hello out all designated ports.

Step 3. Steps 1 and 2 repeat until something changes.

-   When a switch ceases to receive the Hellos, or receives a Hello that lists different details, something has failed, so the switch reacts and starts the process of changing the spanning-tree topology.

STP Timers That Manage STP Convergence

-   STP convergence process requires the use of three timers

-   All switches use the timers as dictated by the root switch, which the root lists in its periodic Hello BPDU messages.

hello

-   2 seconds by default
-   Period between hellos created by the root

MaxAge

-   10 times the hello (20 seconds by default hello)
-   How long the switch will go without receiving any hellos before it attempts to change the stp topology

Forward Delay

-   15 seconds
-   how long the port stays in listening and learning state

-   After MaxAge expires, the switch essentially makes all its STP choices again, based on any Hellos it receives from other switches.

Changing Interface States with STP

Roles, like root port and designated port, relate to how STP analyzes the LAN topology. States, like forwarding and blocking, tell a switch whether to send or receive frames.

When STP converges, a switch chooses new port roles, and the port roles determine the state (forwarding or blocking).

Switches using STP can simply move immediately from forwarding to blocking state, but they must take extra time to transition from blocking state to forwarding state.

when a port that formerly blocked needs to transition to forwarding, the switch first puts the port through two intermediate interface states.

Listening:

-   interface does not forward frames.
-   switch removes old stale (unused) MAC table entries for which no frames are received from each MAC address during this period.
-   These stale MAC table entries could be the cause of the temporary loops.
-   Transitory

Learning:

-   do not forward frames
-   switch begins to learn the MAC addresses of frames received on the interface.
-   Transitory

Blocking

-   does not forward frames
-   does not learn mac addresses
-   stable

Forwarding

-   learns mac addresses
-   forwards frames
-   stable

Disabled

-   does not forward or learn mac addresses
-   stable

blocking > listening, > learning > forwarding.

STP leaves the interface in each interim state for a time equal to the forward delay timer, which defaults to 15 seconds.

a convergence event that causes an interface to change from blocking to forwarding requires 30 seconds to transition from blocking to forwarding.

a switch might have to wait MaxAge seconds (default 20 seconds) before even choosing to move an interface from blocking to forwarding state.

Rapid STP Concepts

-   802.1w
-   Sits in 802.1q standards document

Comparing STP and RSTP

-   similarities

-   elect the root switch using the same rules and tiebreakers.
-   switches select their root ports with the same rules.
-   elect designated ports on each LAN segment with the same rules and tiebreakers.
-   place each port in either forwarding or blocking state

-   (RSTP calls the blocking state the discarding state.)

-   they can both be used in the same network.

-   RSTP improves network convergence when topology changes occur, usually converging within a few seconds (or in slow conditions, in about 10 seconds).

RSTP defines more cases in which the switch can avoid waiting for a timer to expire, such as the following:

-   a switch can replace its root port, without any waiting to reach a forwarding state (in some conditions).
-   replace a designated port, without any waiting to reach a forwarding state (in some conditions).
-   lowers waiting times for cases in which RSTP must wait for a timer.
-   MaxAge is only 3 times the hello
-   uses the term alternate port to refer to a switch’s other ports that could be used as the root port if the root port ever fails.
-   The backup port concept provides a backup port on the local switch for a designated port.

-   backup ports apply only to designs that use hubs, so they are unlikely to be useful today.)

RSTP Port roles

Root Port

-   Nonroot switch's port that has the best path to the root

Alternate port

-   Replaces the root port when the root port fails

Designated port

-   port designated to forward onto a collision domain

Backup Port

-   Replaces designated port when designated port fails

Disabled ports

-   administratively disabled

-   each switch independently generates its own Hellos.
-   allows for queries between neighbors

-   (rather than waiting on timers to expire to learn new information)

RSTP and the Alternate (Root) Port Role

alternate port

-   both the RP and the alternate port must receive Hellos that identify the same root switch.

the switch changes the former root port’s role and state:

-   the role from root port to a disabled port, and
-   the state from forwarding to discarding
-   without waiting on any timers, the switch changes roles and state for the alternate port:

-   its role changes to be the root port, with a forwarding state.

-   the new root port also does not need to spend time in other states, such as learning state, instead moving immediately to forwarding state.

Step 1. The link between SW1 and SW3 fails, so SW3’s current root port (Gi0/1) fails.

Step 2. SW3 and SW2 exchange RSTP messages to confirm that SW3 will now transition its former alternate port (Gi0/2) to be the root port. This action causes SW2 to flush the required MAC table entries.

Step 3. SW3 transitions Gi0/1 to the disabled role and Gi0/2 to the root port role.

Step 4. SW3 transitions Gi0/2 to a forwarding state immediately, without using learning state, because this is one case in which RSTP knows the transition will not create a loop.

RSTP States and Processes

-   RSTP keeps both the learning and forwarding states as compared with STP, for the same purposes

-   RSTP does not even define a listening state,

-   RSTP renames the blocking state to the discarding state and redefines its use slightly.

-   RSTP uses the discarding state for what STP defines as two states: disabled state and blocking state.

![Table 9-10 Port States Compared: STP and RSTP 
Function 
Port is administratively disabled 
Stable state that ignores incoming data frames and is not used to 
forward data frames 
Interim state without MAC learning and without forwarding 
Interim state with MAC learning and without forwarding 
Stable state that allows MAC learning and forwarding of data 
STP State 
Disabled 
Blocking 
Listening 
Learning 
Forward- 
RSTp 
State 
Discard- 
Discard- 
Not used 
Learning 
Forward

-   RSTP switches tell each other (using messages) that the topology has changed.
-   Those messages also direct neighboring switches to flush the contents of their MAC tables in a way that removes all the potentially loop-causing entries, without a wait.

-   RSTP backup port role creates a way for RSTP to quickly replace a switch’s designated port on some LAN.

RSTP Port Types

-   several links between two switches. RSTP considers these links to be point-to-point links and the ports connected to them to be point-to-point ports

Point-to-point 
port 
SWI 
SW3 
Point-to-point 
Edge Port 
Point-to-point 
Port 
SW2 
swa 
Shared 
Port 
Hub 

-   Ports that instead connect to a single endpoint device at the edge of the network, like a PC or server, are called point-to-point edge ports, or simply edge ports.

-   "shared" to describe ports connected to a hub.

-   hubs also force the attached switch port to use half-duplex logic. RSTP assumes that all half-duplex ports may be connected to hubs, treating ports that use half duplex as shared ports.
-   RSTP converges more slowly on shared ports as compared to all point-to-point ports.

Optional STP Features

## Etherchannel

-   The switches treat the EtherChannel as a single interface with regard to STP.

Layer 2 EtherChannels combine links that switches use as switch ports, with the switches using Layer 2 switching logic to forward and receive Ethernet frames over the EtherChannels. Layer 3 EtherChannels also combine links, but the switches use Layer 3 routing logic to forward packets over the EtherChannels.

PortFast

PortFast

-   allows a switch to immediately transition from blocking to forwarding, bypassing listening and learning states.

-   only ports on which you can safely enable PortFast are ports on which you know that no bridges, switches, or other STP-speaking devices are connected.

-   Cisco switches enable RSTP point-to-point edge ports by enabling PortFast on the port.

## BPDU Guard

An attacker could connect a switch to a port with a low STP/RSTP priority value. And make itself the root switch. The new STP/RSTP topology could have worse performance than the desired topology.

-   The attacker could plug into multiple ports, into multiple switches, become root, and actually forward much of the traffic in the LAN. Without the networking staff realizing it, the attacker could use a LAN analyzer to copy large numbers of data frames sent through the LAN.

-   Users could innocently harm the LAN when they buy and connect an inexpensive consumer LAN switch (one that does not use STP/RSTP). Such a switch, without any STP/RSTP function, would not choose to block any ports and could cause a loop.

-   Cisco BPDU Guard feature helps defeat these kinds of problems by disabling a port if any BPDUs are received on the port. So, this feature is particularly useful on ports that should be used only as an access port and never connected to another switch.

-   In addition, the BPDU Guard feature helps prevent problems with PortFast. PortFast should be enabled only on access ports that connect to user devices, not to other LAN switches. Using BPDU Guard on these same ports makes sense because if another switch connects to such a port, the local switch can disable the port before a loop is created.
