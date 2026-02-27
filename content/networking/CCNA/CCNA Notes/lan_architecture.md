1.2 Describe characteristics of network topology architectures

1.2.a 2 tier

1.2.b 3 tier

1.2.e Small office/home office (SOHO)

1.3 Compare physical interface and cabling types

1.3.c Concepts of PoE

Two-Tier Campus Design (Collapsed Core)

access, distribution, and core.

I 
GigE 
A2 
10/100/1000 10/100/1000 
To WAN 
2x 10 GbE 
Uplinks 
Distribution 
2 Distribution 
DI 
A39 
Switches 
GigE 
40 Access 
A40 
Switches 
1000 pcs 
Layer 
Access 
Layer 
10/100/1000 10/100/1000 
Figure 13-1 Campus LAN with Design Terminology Listed ">

Access switches connect directly to end users,

Distribution switches provide a path through which the access switches can forward traffic to each other

each of the access switches connects to at least one distribution switch, typically to two distribution switches for redundancy.

most designs use at least two uplinks to two different distribution switches (as shown in Figure 13-1) for redundancy.

Provides a place to connect end-user devices (the access layer, with access switches)

Connects the switches with a reasonable number of cables and switch ports by connecting all 40 access switches to two distribution switches

Topology Terminology Seen Within a Two-Tier Design

Star: A design in which one central device connects to several others, so that if you drew the links out in all directions, the design would look like a star with light shining in all directions.

Full mesh: For any set of network nodes, a design that connects a link between each pair of nodes.

Partial mesh: For any set of network nodes, a design that connects a link between some pairs of nodes, but not all. In other words, a mesh that is not a full mesh.

Hybrid: A design that combines topology design concepts into a larger (typically more complex) design.

two-tier design is indeed a hybrid design that uses both a star topology at the access layer and a partial mesh at the distribution layer.

The distribution layer creates a partial mesh.

none of the access layer switches connect to each other.

3 
D4 
Figure 13-3 Using a Full Mesh at the Distribution Layer, 6 Switches, 15 Links ">

Three-Tier Campus Design (Core)

collapsed core refers to the fact that the two-tier design does not have a third tier, the core tier

ll 
A12 
A13 
Al 4 
Dil 
D12 
A31 
031 
A32 
Building 2 
A21 
D21 
A22 
A23 
D22 
,A24 
D32 
A33 
Building 3 
Figure 13-4 Two-Tier Building Design, No Core, Three Buildings ">

Sometimes the center of the network uses a full mesh, sometimes a partial mesh, depending on the availability of cables between the buildings.

provide one function: to connect the distribution switches.

All 
Dil 
A12 
A13 
D12 
Al 4 
Building 2 
Core 1 
031 
A32 
Core21' 
032 
A33 
Building 3 
D21 
D22 
A34 
A21 
A22 
A23 
A24 
A31 
Figure 13-5 Three-Tier Building Design (Core Design), Three Buildings ">

Access: Provides a connection point (access) for end-user devices. Does not forward frames between two other access switches under normal circumstances.

Distribution: Provides an aggregation point for access switches, providing connectivity to the rest of the devices in the LAN, forwarding frames between switches, but not connecting directly to end-user devices.

Core: Aggregates distribution switches in very large campus LANs, providing very high forwarding rates for the larger volume of traffic due to the size of the network.

Topology Design Terminology

SWI 
Access Switch: Star 
Figure 13-6 LAN Design Terminolog 
Uplinks: Partial Mesh ">

the right side of Figure 13-6 combines the star topology of the access layer with the partial mesh of the distribution layer. So you might hear these designs that combine concepts called a hybrid design.

### Small Office/Home Office

000000 
UTP 
CATV 
ISP/lnternet 
cable 
RI 
Figure 13-7 A Typical Home Wired and Wireless LAN ">

that one wireless router acts like separate devices you would find in an enterprise campus:

Key Topic.

An Ethernet switch, for the wired Ethernet connections

A wireless access point (AP), to communicate with the wireless devices and forward the frames to/from the wired network

A router, to route IP packets to/from the LAN and WAN (Internet) interfaces

A firewall, which often defaults to allow only clients to connect to servers in the Internet, but not vice versa

UTP 
CATV 
Cable 
UTP 
ISP/ 
Internet 
UTP 
Cable 
Modem 
Figure 13-8 A Representation of the Functions Inside a Consumer Wireless Routing Product ">

## Power over Ethernet (PoE)

a LAN switch, acts as the Power Sourcing Equipment (PSE

supplies DC power  so the device does not need an AC/DC converter.

A device that has the capability to be powered over the Ethernet cable

is called the Powered Device (PD)

Devices (PDs) 
Ethernet Cables 
DC Power 
e eee 
Figure 13-9 
Power Supply 
Power Sourcing 
Equipment (PSE) 
AC Power 
cable 
AC Power 
Outlet 
Power over Ethernet Terminolog ">

PoE Operation

PoE must (and does) have processes in place to determine if PoE is needed, and for how much power, before applying any potentially harmful power levels to the circuit.

autonegotiation mechanisms need to work before the PD has booted,

By using these IEEE autonegotiation messages and watching for the return signal levels, PoE can determine whether the device on the end of the cable requires power (that is, it is a PD) and how much power to supply

Step 1. Do not supply power on a PoE-capable port unless negotiation identifies that the device needs power.

Step 2. Use Ethernet autonegotiation techniques, sending low power signals and monitoring the return signal, to determine the PoE power class, which determines how much power to supply to the device.

Step 3. If the device is identified as a PD, supply the power per the power class, which allows the device to boot.

Step 4. Monitor for changes to the power class, both with autonegotiation and listening for CDP and LLDP messages from the PD.

Step 5. If a new power class is identified, adjust the power level per that class.

PDs signaling how many watts of power they would like to receive from the PSE. Depending on the specific PoE standard, the PSE will then supply the power, either over two pairs or four pairs, as noted in Table 13-2.

Table 13-2 Power over Ethernet Standards 
802.3af 
802.3at 
802.3bt 
802.3bt 
Name 
Cisco Inline Power 
PoE 
POE + 
UPoE 
UpoE+ 
Standard 
Cisco 
Watts at PSE 
7 
15 
30 
60 
100 
Powered Wire Pairs 
2 
2 
2 
4 
4 ">

PoE and LAN Design

some of the key points to consider when planning a LAN design that includes PoE:

Powered Devices: Determine the types of devices and specific models, along with their power requirements.

Power Requirements: Plan the numbers of different types of PDs to connect into each wiring closet to build a power budget. That power budget can then be processed to determine the amount of PoE power to make available through each switch.

Switch Ports: Some switches support PoE standards on all ports, some on no ports, some on a subset of ports. Research the various switch models so that you purchase enough PoE-capable ports for the switches planned for each wiring closet.

Switch Power Supplies: Without PoE, when purchasing a switch, you choose a power supply so that it delivers enough power to power the switch itself. With PoE, the switch acts as a distributor of electrical power, so the switch power supply must deliver many more watts than it needs to run the switch itself. You will need to create a power budget per switch, based on the number of connected PDs, and purchase power supplies to match those requirements.

PoE Standards versus Actual: Consider the number of PoE switch ports needed, the standards they support, the standards supported by the PDs, and how much power they consume. For instance, a PD and a switch port may both support PoE+, which supports up to 30 watts supplied by the PSE. However, that powered device may need at most 9 watts to operate, so your power budget needs to reserve less power than the maximum for those devices.

show power inline

power inline never

power inline auto
