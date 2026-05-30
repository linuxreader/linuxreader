
1.0 Network Fundamentals
1.6 Configure and verify IPv4 addressing and subnetting
3.0 IP Connectivity
3.1 Interpret the components of routing table
3.1.a Routing protocol code
3.1.b Prefix
3.1.c Network mask
3.1.d Next hop
3.1.e Administrative distance
3.1.f Metric
3.1.g Gateway of last resort
3.2 Determine how a router makes a forwarding decision by default
3.2.a Longest match
3.2.b Administrative distance
3.3 Configure and verify IPv4 and IPv6 static routing
3.3.a Default route
3.3.b Network route
3.3.c Host route
3.3.d Floating static
```

```
Routers first learn connected routes,Routers can also use static routes, which are which are routes created through a configuration command (ip routes for subnets attached to a router interface.
route)protocol, in which routers tell each other about all their known routes, so that all routers can learn that tells the router what route to put in the IPv4 routing table.And routers can use a routing
and build routes to all networks and subnets.
Step 1. If the destination is local, send directly:
A. ''Find the destination host’s MAC address. Use the already-known Address Resolution Protocol
```

# 16 Static Routes

Friday, November 12, 2021 9:33 PM

```
A. ''(ARP) table entry, or use ARP messages to learn the information.Find the destination host’s MAC address. Use the already-known Address Resolution Protocol
B. Encapsulate the IP packet in a data-link frame, with the destination data-link address of the
destination host.
```

1. The frame has no errors (per the dataThe frame’s destination data-link address is the router’s address (or an appropriate multicast or -link trailer Frame Check Sequence [FCS] field).  
    broadcast address).

```
2.
```

```
For each received data-link frame, choose whether or not to process the frame. Process it if
```

1. If choosing to process the frame at Step 1, de-encapsulate the packet from inside the data-link frame.  
    Make a routing decision. To do so, compare the packet’s destination IP address to the routing table  
    and find the route that matches the destination address. This route identifies the outgoing interface of the router and possibly the next-hop router.
    

```
Encapsulate the packet into a data-link frame appropriate for the outgoing interface. When
forwarding out LAN interfaces, use ARP as needed to find the next device’s MAC address.
```

4. Transmit the frameout the outgoing interface, as listed in the matched IP route.
    
    - The interface is in a working state. In other words, the interface status in theinterfacescommand lists a line status of up and a protocol status of up. show
    - The interface has an IP address assigned through theip addressinterface subcommand.  
        Routing Protocol Code:The legend at the top of the show ip routeoutput (about nine lines) lists all  
        the routing protocol codes (exam topic 3.1.a). (C), local (L), static (S), and OSPF (O). This book references the codes for connected routes  
        Prefix: The word prefix (exam topic 3.1.b) is just another name for subnet ID.  
        Mask: Each route lists a prefix (subnet ID) and network mask (exam topic 3.1.c) in prefix format, for example, /24.

###### The ARP Table on a Cisco Router

```
Dynamically learned ARP table entries have an upward counter, like the 35ARP table entry for IP address 172.16.1.9.By default, IOS will time out (remove) an ARP table -minute value for the
entry after 240 minutes in which the entry is not used.
```

clear ip arp [ip-address]EXEC command.  
Configuring Static Routes  
The static route is considered a network route when the destination listed in the ip route command defines a subnet, or an entire Class A, B, or C network.In contrast, a default route  
matches all destination IP addresses, while a host route matches a single IP addressaddress of one host.) (that is, an

```
However, troute; this is just a quirk of the output of the he route that used the outgoing interface configuration is also noted as a connected show ip route command.
also lists a few statistics about all IPv4 routes. For example, the example shows two lines, for
the two static routes configured in Example 16for eight subnets. -4, but statistics state that this router has routes
IOS adds and removes these static routes dynamically over time, based on whether the outgoing interface is working or not.For example, in this case, if R1’s S0/0/0 interface fails, R1
removes the static route to 172.16.2.0/24 from the IPv4 routingcomes up again, IOS adds the route back to the routing table. table. Later, when the interface
```

Static Host Routes  
To configure such a static route, theof 255.255.255.255 so that the matching logic matches just that one address.ip routecommand uses an IP address plus a mask

```
An engineer might use host routes to direct packets sent to one host over one path, with all other traffic to that host’s subnet over some other path. For instance, you could
define these two static routes for subnet 10.1.1.0/24 and host 10.1.1.9, with two different next-hop addresses, as follows:
```

```
routers use the most specific route (that is, the route with the longest prefix length)
```

###### the router must first decide which routing source has the betterdistance, with lower being better, and then use the route learned from the better administrative

###### source

###### By default, IOS considers static routes better than OSPF-learned routes. By

###### default,IOS gives static routes an administrative distance of 1and OSPF routes

###### an administrative distance of 110

Floating Static Routes

###### an administrative distance of 110

###### Torouteimplement a floating static routecommand that sets the administrative distance for just that route, making , you need to use a parameter on theip

###### the value larger than the default administrative distance of the routing protocol.

###### For example, theip route 172.16.2.0 255.255.255.0 172.16.5.3 130

###### while theshow ip routecommand lists the administrative distance of most

###### routes, as the first of two numbers inside two brackets, theroutesubnetcommand plainly lists the administrative distance.show ip

###### ip route 0.0.0.0 0.0.0.0 S0/0/1creates a static default routeon Router B1—a

###### route that matches all IP packets—and sends those packets out interface S0/0/1.

###### a than one default route, and the router then has to choose which one to use; *, meaning it is acandidate default route. A router can learn about more

###### the * means that it is at least a candidate to become the default route.

###### show ip route

Static Default Routes

- The route is in the routing table but is incorrect.
    - Is there a subnetting math error in the subnet ID and mask?
    - Is the nextrouter? -hop IP address correct and referencing an IP address on a neighboring
    - Does the nextIs the outgoing interface correct, and referencing an interface on the local router -hop IP address identify the correct router?
    - (that is, the same router where the static route is configured)?
- The route is not in the routing table.

###### Troubleshooting Static Routes

- The route is not in the routing table.  
    table. IOS also considers the following before adding the route to its routing table:
- Forup/up state.ip routecommands that list an outgoing interface, that interface must be in an
- Fora route to reach that nextip routecommands that list a next-hop address.-hop IP address, the local router must have

###### You can configure a static route so that IOS ignores these basic checks, always putting the IP route in the routing table. To do so, just use

###### thethepermanentpermanentkeywordkeyword to the end of the two commands as demonstrated on theip routecommand. For example, by adding

###### inExample 16- 7 , R1 would now add these routes, regardless of whether the

###### two WAN links were up.

- The route is in the routing table and is correct, but the packets do not arrive at the destination host.  
    Many legitimate router features can cause these multiple routes to appear in a router’s routing table, including
- • Static routesRoute autosummarization
- Manual route summarization

###### When a particular destination IP address matches more than one route in a router’s

###### IPv4 routing table, the router uses the most specific route—in other words, the route

###### with the longest prefix length mask.

###### A second way to identify the route a router will use, one that does not require any subnetting math, is theshow ip routeaddresscommand.The last parameter on this

###### command is the IP address of an assumed IP packet. The router replies by listing the route it would use to route a packet sent to that address.

A. Use the switch forwarding ASIC settings to make space for IPv4 routes at the next reload of the switch. **sdm prefer lanbase-routing** command (or similar) in global configuration modeto change the

B. Use the prefer command setting. **reload EXEC** command in enable mode toreload (reboot) the switch to pick up the new sdm

C. Once reloaded, use the **ip routing** command in global configuration mode to enable the IPv4 routing  
function in IOS software and to enable key commands like **show ip route**.  
if you then enabled OSPF on the Layer 3 switch, the configuration and verification would work the same as it does on a router, as discussed in Chapter 20, “Implementing OSPF.” The routes that IOS adds to the  
Layer 3 switch’s IP routing table would list the VLAN interfaces as outgoing interfaces.  
Troubleshooting Routing with SVIs  
look to those first few configuration commands listed in the configuration checklist found in the  
earlier section “Configuring Routing Using Switch SVIs.” Those commands are (followed by a reload) and then ip routing(after the reload). sdm prefer  
Theforwarding tables, and changes to those tables require a reload of the switch. By default, many sdm prefercommand changes how the switch forwarding chips allocate memory for different  
access switches that support Layer 3 switching still have an SDM default that does not allocate space for an IP routing table. Once changed and reloaded, the **ip routing** command then enables  
IPv4 routing in IOS software. Both are necessary before some Cisco switches will act as a Layer 3 switch.  
Scenario 1: The last access interface in VLAN 10 is shut down (F0/1), so IOS shuts down the VLAN 10 interface.  
Scenario 2: VLAN 20 (not VLAN interface 20, but VLAN 20) is deleted, which results in IOS then  
bringing down (not shutting down) the VLAN 20 interface.  
Scenario 3: VLAN 30 (not VLAN interface 30, but VLAN 30) is shut down, which results in IOS then  
bringing down (not shutting down) the VLAN 30 interface.  
VLAN Routing with Layer 3 Switch Routed Ports  
On a routed port, the switch does not perform Layer 2 switching logic on that frame. Instead, frames arriving in a routed port trigger the Layer 3 routing logic, including  
Stripping off the incoming frame’s Ethernet data-link header/trailer  
Making a Layer 3 forwarding decision by comparing the destination IP address to the IP routing table  
Adding a new Ethernet data-link header/trailer to the packet  
Forwarding the packet, encapsulated in a new frame

# 17 Routing in the LAN

Monday, November 22, 2021 9:39 AM

```
Forwarding the packet, encapsulated in a new frame
The exam topics do not mention routed interfaces specifically, but the exam topics do mention L3 EtherChannels, meaning Layer 3 EtherChannels.
```

Implementing Routed Interfaces on Switches  
Enabling a switch interface to be a routed interface instead of a switched interfaceuse the no switchportsubcommand on the physical interface. is simple: just  
To make the port stop acting like a switch port and instead act like a router port, use the no  
switchport command on the interface.  
Once the port is acting as a routed port, think of it like a router interface  
the routed interface will show up differently in command output in the switch.In particular, for an  
interface configured as a routed port with an IP address, like interface GigabitEthernet0/1 in the previous example:  
Key Topic. **show interfaces:** Similar to the same command on a router, the output will display the IP address  
of the interface. (Conversely, for switch ports, this command does not list an IP address.)  
**show interfaces status:** trunk, the output lists the word routedUnder the “VLAN” heading, instead of listing the access VLAN or the word , meaning that it is a routed port.  
**show ip route** : Lists the routed port as an outgoing interface in routes.  
**show interfaces type number switchport** the port is not a switch port. (If the port is a Layer 2 port, this command lists many configuration :If a routed port, the output is short and confirms that  
and status details.)  
All the ports that are links directly between the Layer 3 switches can be routed interfaces.

```
each pair of switches has one routing protocol neighbor relationship with the neighbor, and not two. Each switch learns one route per destination per pair of links, and not two. IOS then balances
the traffic, often with better balancing than the balancing that occurs with the use of multiple IP routes to the same subnet.
```

Implementing Layer 3 EtherChannels

Step 1.Configure the physical interfaces as follows, in interface configuration mode:  
A. Add thechannel-group number mode oncommand to add it to the channel. Use the same  
number for all physical interfaces on the same switch, but the number used (the channelnumber) can differ on the two neighboring switches. -group

B. Add the **no switchport** command to make each physical port a routed port.  
Step 2. Configure the PortChannel interface:  
A. Use thefor the same channel number configured on the physical interfaces. **interface port-channel number** command to move to port-channel configuration mode

B. Add the port. (IOS may have already added this command.) **no switchport** command to make sure that the port-channel interface acts as a routed

C. Use the **ip address address mask** command to configure the address and mask.  
Cisco uses the term EtherChannel in concepts discussed in this section and then uses the term PortChannel, with command keyword port-channel, when verifying and configuring  
EtherChannels.

although the physical interfaces and PortChannel interface are all routed ports,should be placed on the PortChannel interface only. In fact,when the no switchport command is the IP address  
configured on an interface, IOS adds the no ip address command to the interface.

Troubleshooting Layer 3 EtherChannels

```
look at the configuration of the EtherChannel. Second, you should check a list of settings that must match on the interfaces for a channel-group command, which enables an interface for an
Layer 3 EtherChannel to work correctly.
As for the channel-group interface subcommand, this command can enable EtherChannel
statically or dynamically. If dynamic, this command’s keywords imply either Port Aggregation Protocol (PAgP) or Link Aggregation Control Protocol (LACP) as the protocol to negotiate between
the neighboring switches whether they put the link into the EtherChannel.
no switchportand so must the physical interfaces. : The PortChannel interface must be configured with the no switchport command,
Speed: The physical ports in the channel must use the same speed.
duplex: The physical ports in the channel must use the same duplex.
```

Troubleshooting Layer 3 EtherChannels

Problem Isolation Using the ping Command  
functions as part of Layer 3, as a control protocol to assist IP by helping manage the IP network  
functions.  
The name or IP address of the destination, how many times the command should send an echo request,  
how long the command should wait (timeout) for an echo reply,how big to make the packets, and many other options.

```
Ping options
```

```
Extended ping allows R1’s ping command to use R1’s LAN IP address from within subnet
172.16.1.0/24.
```

```
This same command could have been issued from the command line as ping 172.16.2.101 source
172.16.1.1.
R1# pingProtocol [ip]:
Target IP address: 172.16.2.101Repeat count [5]:
Datagram size [100]:
Timeout in seconds [2]:Extended commands [n]: y
Source address or interface: 172.16.1.1Type of service [0]:
Set DF bit in IP header? [no]:Validate reply data? [no]:
Data pattern [0xABCD]:
Loose, Strict, Record, Timestamp, Verbose[none]:Sweep range of sizes [n]:
Type escape sequence to abort.Sending 5, 100-byte ICMP Echos to 172.16.2.101, timeout is 2 seconds:
Packet sent with a source address of 172.16.1.1!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
Problems to test. ( ping from router or from host?)
```

# 18 T-Shoot Routing

Tuesday, November 23, 2021 5:09 AM

```
IP ACLs that discard packets based on host A’s IP address but allow packets that match the
router’s IP address
```

- LAN switch port security that filters A’s frames (based on A’s MAC address)
- IP routes on routers that happen to match host A’s 172.16.1.51 address, with different routes that match R1’s 172.16.1.1 address
- Problems with host A’s default router setting

```
If the ping works, it confirms the following, which rules out some potential issues:
The host with address 172.16.1.51 replied.
The LAN can pass unicast frames from R1 to host 172.16.1.51 and vice versa.
You can reasonably assume that the switches learned the MAC addresses of the router and
the host, adding those to the MAC address tables.
Host A and Router R1 completed the ARP process and list each other in their respective Address Resolution Protocol (ARP) tables.
```

- **IP addressing problem:DHCP problems:** If you are using Dynamic Host Configuration Protocol (DHCP), many problems Host A could be statically configured with the wrong IP address.  
    could exist.
- **VLAN trunking problems:** The router could be configured for 802.1Q trunking, when the switch is  
    not (or vice versa).
- **LAN problems:** A wide variety of issues could exist with the Layer 2 switches, preventing any  
    frames from flowing between host A and the router.

Testing LAN Neighbors with Extended Ping  
an extended ping can test the host’s default router setting. Both tests can be useful, especially for  
problem isolation, because

- If a standard ping of a local LAN host works...
- • But an extended ping of the same LAN host fails...The problem likely relates somehow to the host’s default router setting.

Testing WAN Neighbors with Standard Ping

```
A successful ping of the IP address on the other end of an Ethernet WAN link that sits between two routers confirms several specific facts, such as the following:
Both routers’ WAN interfaces are in an up/up state.
The Layer 1 and 2 features of the link work.
The routers believe that the neighboring router’s IP address is in the same subnet.
Inbound ACLs on both routers do not filter the incoming packets, respectively.
The remote router is configured with the expected IP address (172.16.4.2 in this case).
```

###### Using Ping with Names and with IP Addresses

Problem Isolation Using the traceroute Command  
ping vs Tracert

- Both send messages in the network to test connectivity.

- • Both send messages in the network to test connectivity.Both rely on other devices to send back a reply.
- • Both have wide support on many different operating systems.Both can use a hostname or an IP address to identify the destination.
- On routers, both have a standard and extended version, allowing better testing of the reverse route.  
    Standard and Extended traceroute  
    a standardthe packet sent by the command. So, in this example, the packets sent by R1 come from **traceroute** command chooses an IP address based on the outgoing interface for  
    source IP address 172.16.4.1, R1’s G0/0/0 IP address.

Host OS traceroute commands usually create ICMP echo requests.TheCisco IOS traceroute  
command instead creates IP packets with a UDP headertrivial at this point. However, note that an ACL may actually filter the traffic from a host’s. This bit of information may seem  
traceroute messages but not the router traceroute command, or vice versa.  
Telnet and SSH

This is a offline tool, your data stays locally and is not send to any server!