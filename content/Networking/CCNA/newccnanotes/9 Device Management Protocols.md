# 9 Device Management Protocols

Monday, October 4, 2021 2:02 PM

```
To configure a router or switch to logging host {address | hostname } send log messages to a syslog serverglobal command , add the
```

```
*Dec 18 17:10:15.079: %LINEPROTOchanged state to down - 5 - UPDOWN: Line protocol on Interface FastEthernet0/0,
A timestamp: *Dec 18 17:10:15.
The facility on the router that generated the message : %LINEPROTO
The severity level : 5
A mnemonic for the message : UPDOWN
The description of the message: Line protocol on Interface FastEthernet0/0, changed
state to down
you can at least toggle on and off the use of thelog message sequence number (which is not enabled by default)timestamp (which is included by default). Example 9-1 reverses those and a
defaults by turning off timestamps and turning on sequence numbers.
```

Log Message Format

Log Message Severity Levels  
the lower the number, the more severe the event that caused the message.

```
last level in the figure is used for messages requested by the debug command
Table 9severity level for each type.-2 summarizes the configuration commands used to enable logging and to set the
For example, the command logging console 4 causes IOS to send severity level 0the console –4 messages to
```

Configuring and Verifying System Logging

the example configures the same message level at the console and for terminal monitoring  
(level 7, or debug), and the same level for both buffered and logging to the syslog server (level 4, or warning). The levels may be set using the numeric severity level or the name as  
shown earlier in Figure 9-3.  
**show logging** messages per the logging buffered configuration.commandconfirms those same configuration settings and also lists the log

If any log messages had been buffered, the actual log messages would be listed at  
the end of the command  
clear out the old messages from the log with the **clear logging** EXEC command.

The debug Command and Log Messages  
**debug** internal events, with that monitoring process continuing over time, so that IOS can issue log EXEC command gives the network engineer a way to ask IOS to monitor for certain  
messages when those events occur  
debug remains active until some user issues the **no debug command** with the same parameters,  
disabling the debug.

```
anyone logged in with SSH at the time Example 9the output, even with the logging monitor debug - 4’s output was gathered would not have seen command configured on router R1, without
first issuing a terminal monitor command.
```

```
all enabled debug options use router CPU, which can cause problems for the router. You can
monitor CPU use withtheshow process cpucommand
use caution when using debug commands carefully on production devices.
the more CLI users that receive debug messages, the more CPU that is consumed.
```

#### Network Time Protocol (NTP)

```
Devices send timestamps to each other with NTP messages, continually exchanging messages, with one device changing its clock to match the other, eventually synchronizing the clocks.
How NTP defines the sources of time data (stratum). (reference clocks) andhow good each time source is
```

**Setting the Time and Timezone**  
NTP works best if you set the device clock to a reasonably close time before enabling the NTP  
client functionwith the **ntp server** command  
I shouldset the time to 8:52 p.m., set the correct date and timezone, and even tell the device to  
adjust for daylight savings time—and then enable NTP  
Example 9-7 shows how to set the date, time, timezone, and daylight savings time.

```
You should set the first two commands before setting the time of day with the EXEC command because the two configuration commands impact the time that is set. clock set
chose EDT because it is the acronym for daylight savings time in that same EST time zone.
Finally, thehour automatically over the years.recurring keyword tells the router to spring forward an hour and fall back an
clock set EXEC command
uses a time syntax with a 24-hour format, not with a 12-hour format plus a.m./p.m.).
```

```
uses a time syntax with a 24-hour format, not with a 12-hour format plus a.m./p.m.).
The EDT, rather than UTC time. show clock command (issued seconds later)lists that time, but also notes the time as
```

**Basic NTP Configuration**  
two ntp configuration commands  
**ntp master {stratum** not as an NTP client. The device gets its time information from the internal clock on the **- level}:** NTP server mode—the device acts only as an NTP server, and  
device.  
**ntp server {address | hostname}** : NTP client/server mode—the device acts as both client  
and serversynchronized, the device can then act as an NTP server, to supply time to other NTP. First, it acts as an NTP client, to synchronize time with a server. Once  
clients.

```
show ntp status command
```

**show ntp status** command  
lists a status of synchronizedchanging its time to match the server’s time. , which confirms the NTP client has completed the process of Any router acting as an NTP client will list  
“unsynchronized” in that first line until the NTP synchronization process completes with at least one server. It also confirms the IP address of the server—this device’s reference  
clock—with the IP address configured in Example 9-8 (172.16.2.2).

```
show ntp associations
lists all the NTP servers that the local device can attempt to use, with status information
about the association between the local device (client) and the various NTP servers.
```

###### NTP Reference Clock and Stratum

```
Devices that act solely as an NTP server get their time from either internal device hardware or from some external clock using mechanisms other than NTP.
NTP servers and clients use a number to show the perceived accuracy of their reference clock data based on stratum level. The lower the stratum level, the more accurate the reference clock
is considered to be.sets its own stratum level. Then,An NTP server that uses its internal hardware or external reference clock an NTP client adds 1 to the stratum level it learns from its NTP
server, so that the stratum level increases the more hops away from the original clock source.
For instance, back in Figure 9which references R3, adds 1 so it has a stratum of 3. R1 uses R2 as its NTP server, so R1 adds 1 to -5, you can see the NTP primary server (R3) with a stratum of 2. R2,
have a stratum of 4. These increasing stratum levels allow devices to refer to several NTP servers
and then use time information from the best NTP server, best being the server with the lowest stratum level.
Routers and switches use the default stratum level of 8 for their internal reference clock based
on the default setting of 8 for the stratum level in the command allows you to set a value from 1 through 15 ntp master [stratum ; in Example 9-8, the - ntp master 2level] command. The
command set router R3’s stratum level to 2.
Note
NTP considers 15 to be the highest useful stratum level, so any devices that calculate their
stratum as 16 consider the time data unusable and do not trust the time. So, avoid setting higher stratum values on the ntp master command.
```

```
stratum values on the ntp master command.
```

###### Redundant NTP Configuration

```
an enterprise could use NTP to reference NTP servers that use an atomic clock as their reference source, like the NTP primary servers in Figure 9-6, which happen to be run by the US National
Institute of Standards and Technology (NIST) (see tf.nist.gov).
```

```
AnNTP primary server acts only as a server, with a reference clock external to the device,
and has a stratum level of 1secondary servers are servers that use client/server mode as described throughout this , like the two NTP primary servers shown in Figure 9-6.NTP
section, relying on synchronization with some other NTP server.
```

```
NTP primary server and NTP secondary server.
```

```
After losing their reference clock, R1 and R2 could no longer be useful NTP servers to the rest of the enterprise.
To overcome this potential issue, the routers can also be configured with the ntp master
command, resulting in this logic:
Establish an association with the NTP servers per thentp server command.
Establish an association with your internal clock using the ntp master stratum command.
Set the stratum level of the internal clock (per thea higher (worse) stratum level than the Internet-based NTP servers.ntp master {stratum-level} command) to
Synchronize with the best (lowest) known time source, which will be one of the Internet NTP servers in this scenario
ntp master 7 command, with a much higher stratum,
```

###### NTP Using a Loopback Interface for Better Availability

```
what happens when one interface on R4 fails
for any NTP clients that had referred to that specific IP address
There would likely still be a route to reach R4 itself.
The NTP client would not be able to send packets to the configured address because that interface is down.
```

```
loopback interface to meet that exact need
once configured, it remains in an up/up state as long as
Key Topic.The router remains up.
You do not issue a shutdown command on that loopback interface.
```

#### Analyzing Topology Using CDP and LLDP

###### Examining Information Learned by CDP

```
Cisco-proprietary
Layer 2 protocol
does not rely on a working Layer 3 protocol
```

does not rely on a working Layer 3 protocol  
Devices that support CDP learn information about others by listening for the advertisements sent by other devices.

CDP discovers several useful details from the neighboring Cisco devices:

**Device identifier** : Typically the host name  
**Address list:** Network and data-link addresses  
**Port identifier:** link that sent the CDP advertisementThe interface on the remote router or switch on the other end of the  
**Capabilities list** : Information on what type of device it is (for example, a router or a  
switch)  
**Platform** : The model and OS level running on the device  
Cisco IP Phones use CDP to learn the data and voice VLAN IDs as configured on the access  
switch.

routers and switches support the same CDP commands, with the same parameters and  
same types of output.

To ensure all devices receive a CDP message, tdestination MAC address (0100.0CCC.CCCC). he Ethernet header uses a multicast

the device processes the message and then discards it, rather than forwarding it  
**show cdp neighbors detail**  
lists the full name of the switch model (WSconfigured on the neighboring device. -2960XR-24TS-I) and the IP address

```
The show cdp entry name command lists the exact same details shown in the output of
the command. show cdp neighbors detail command, but for only the one neighbor listed in the
Cisco recommends that CDP be disabled on any interface that might not have a need for CDP. For switches, any switch port connected to another switch, a router, or to an IP
phone should use CDP.
```

###### Configuring and Verifying CDP

```
IOS typically enables CDP globally and on each interface by defaultinterface with the no cdp enable interface subcommand. You can then disable CDP per
re-enable it with the cdp enable interface subcommand
disable and re-enable CDP globally on the device
no cdp run and cdp run global commands
```

```
send time and the hold time. CDP sends messages every 60 seconds by default, with a hold time
of 180 seconds.device before removing those details from the CDP tables. You can override the defaults with The hold time tells the device how long to wait after no longer hearing from a
the cdp timer seconds and cdp holdtime seconds global commands, respectively.
```

#### Examining Information Learned by LLDP

the LLDP output in the example does differ from CDP in a few important ways:

```
LLDP uses B as the capability code for switching, referring to bridgetype that existed before switches that performed the same basic functions., a term for the device
LLDP does not identify IGMP as a capability, while CDP does (I).
CDP lists the neighbor’s platform, a code that defines the device type, while LLDP does not.
LLDP lists capabilities with different conventions (see upcoming Example 9-19).
```

```
System Capabilities : What the device can do
Enabled Capabilities : What the device does now with its current configuration
LLDP uses the same messaging concepts as CDP, encapsulating messages directly in dataheaders. Devices do not forward LLDP messages so that LLDP learns only of directly connected -link
neighbors. LLDP does use a differentmulticast MAC address(0180.C200.000E)
```

```
Cisco devices default to disable LLDP
LLDP separates the sending and receiving of LLDP messages as separate functions.
LLDP support processing receives LLDP messages on an interface so that the switch or router
learns about the neighboring device while not transmitting LLDP messages to the neighboring device.
the commands include options to toggle on|off the transmission of LLDP messages separately
from the processing of received messages.
[no] lldp run: A global configuration command thatsets the default mode of LLDP operationfor
any interface that does not have more specific LLDP subcommands (lldp transmit, lldp receive). The lldp run global command enables LLDP in both directions on those interfaces, while no lldp
```

###### Configuring and Verifying LLDP

The lldp run global command enables LLDP in both directions on those interfaces, while no lldp run disables LLDP.

**[no] lldp transmit** : An interface sub-command that defines the operation of LLDP on the  
interface subcommand causes the device to transmit LLDP messages, while no lldp transmit causes it to regardless of the global [no] lldp run command. The lldp transmit interface  
not transmit LLDP messages.  
**[no] lldp receive** :An interface subcommand that defines the operation of LLDP on the interface  
regardless of the global [no] lldp run command.the device to process received LLDP messages, while no lldp receive causes it to not process The lldp receive interface subcommand causes  
received LLDP messages.

**show lldp interface** lists the interfaces on which LLDP is enabled.

like CDP,shows the default settings of 30 seconds for the send timer and 120 seconds for the hold timer. LLDP uses a send timer and hold timer for the same purposes as CDP.The example  
You can override the defaults with the commands **lldp timer seconds** and **lldp holdtime seconds** global

4.0 IP Services  
4.1 Configure and verify inside source NAT using static and pools

CIDR

```
most public address assignments for the last 20 years have been a CIDR block rather than an entire class A, B, or C network.
```

Private Addressing

```
no organization is allowed to advertise these networks using a routing protocol on the Internet.
```

## 10 NAT

Friday, October 8, 2021 7:12 AM

Network Address Translation Concepts

Static NAT  
IP addresses statically mapped to each other.

NAT router simply configures a oneregistered address that is used on its behalf. -to-one mapping between the private address and the

Supporting a second IP host with static NAT requires a second static onesecond IP address in the public address range. -to-one mappingusing a

the router statically maps 10.1.1.2 to 200.1.1.2. Because the enterprise has a single registered Class C network, it can support at most 254 private IP addresses with NAT, with the usual two  
reserved numbers (the network number and network broadcast address).  
inside local for the private IP addresses in this example and inside global for the public IP addresses.

Dynamic NAT  
Dynamic NAT sets up a pool of possible inside global addresses and defines matching criteria to determine which inside local IP addresses should be translated with NAT.

```
Host 10.1.1.1 sends its first packet to the server at 170.1.1.1.
As the packet enters the NAT router, the router applies some matching logic to decide whether
```

```
As the packet enters the NAT router, the router applies some matching logic to decide whether the packet should have NAT applied. Because the logic has been configured to match source IP
addresses that begin with 10.1.1, the router adds an entry in the NAT table for 10.1.1.1 as an inside local address.
The NAT router needs to allocate an IP address from the pool of valid inside global addresses. It picks the first one available (200.1.1.1, in this case) and adds it to the NAT table to complete the
entry.
The NAT router translates the source IP address and forwards the packet.
The dynamic entry stays in the table as long as traffic flows occasionally. You can configure a timeout value that defines how long the router should wait, having not translated any packets
with that address, before removing the dynamic entry.entries from the table using the clear ip nat translation You can also * command.manually clear the dynamic
NAT can be configured with more IP addresses in the inside local address list than in the inside global address pool.
If a new packet arrives from yet another inside host, and it needs a NAT entry, but all the pooled IP addresses are in use, the router simply discards the packet. The user must try again until a NAT
entry times out, at which point the NAT function works for the next host that sends a packet. Essentially, the inside global pool of addresses needs to be as large as the maximum number of
concurrent hosts that need to use the Internet at the same timeexplained in the next section. —unless you use PAT, as is
```

Overloading NAT with Port Address Translation  
NAT Overload feature, also called Port Address Translation (PAT)

```
The NAT router keeps a NAT table entry for every unique combination of inside local IP address and port, with translation to the inside global address and a unique port number associated with
```

```
and port, with translation to the inside global address and a unique port number associated with the inside global address.
NAT overload can use more than 65,000 port numbers
PAT is by far the most popular option
```

##### NAT Configuration and Troubleshooting

Static NAT Configuration  
Each static mapping between a local (private) address and a global (public) address must be configured.  
Those same interface subcommands tell NAT whether the interface is inside or outside.  
Step 1. Use thebe in the inside part of the NAT design. **ip nat inside** command in interface configuration mode to configure interfaces to  
Step 2. Use the be in the outside part of the NAT design. **ip nat outside** command in interface configuration mode to configure interfaces to  
Step 3. Use theconfiguration mode to configure the static mappings. **ip nat inside source static inside-local inside-global** command in global

```
static mappings are created using the ip nat inside source static command.
Source keyword means that NAT translates the source IP address of packets coming into its inside interfaces. The static keyword means that the parameters define a static entry, which should
never be removed from the NAT table because of timeout.hosts—10.1.1.1 and 10.1.1.2—to have Internet access, two ip nat inside commands are needed.Because the design calls for two
show ip nat translations The show ip nat statistics command lists the two static NAT entries created in the configuration. command lists statistics, listing things such as the number of currently
active translation table entries. The statistics also include the number of hits, which increments for every packet for which NAT must translate addresses.
```

Dynamic NAT Configuration  
Dynamic NAT still requires that each interface be identified as either an inside or outside interface  
Dynamic NAT uses an access control list (ACL) to identify which inside local (private) IP addresses need to have their addresses translated, and it defines a pool of registered public IP addresses to  
allocate.  
Step 1. Use the be in the inside part of the NAT design (just like with static NAT). **ip nat inside** command in interface configuration mode to configure interfaces to  
Step 2. Use the be in the outside part of the NAT design (just like with static NAT). **ip nat outside** command in interface configuration mode to configure interfaces to  
Step 3. be performed.Configure an ACL that matches the packets entering inside interfaces for which NAT should  
Step 4. Use the global configuration mode to configure the pool of public registered IP addresses. **ip nat pool name first-address last-address netmask subnet-mask** command in

Step 5. Use theconfiguration mode to enable dynamic NAT. Note the command references the ACL (step 3) and **ip nat inside source list acl-number pool pool-name** command in global  
pool (step 4) per previous steps.

“misses,” as highlighted in the example. The first occurrence of this counter counts the number of times a new packet comes along, needing a NAT entry, and not finding one.

The second misses counter toward the end of the command output lists the number of misses in the pool. This counter increments only when dynamic NAT tries to allocate a new NAT table entry  
and finds no available addresses

**debug ip nat** packet has its address translated for NAT.command. This debug command causes the router to issue a message every time a

NAT Overload (PAT) Configuration  
If PAT uses a pool of inside global addresses, the configuration looks exactly like dynamic NAT, except the ip nat inside source list global command has an **overload** keyword added to the end. If  
PAT just needs to use one inside global IP address, the router can use one of its interface IP addresses  
configuration when using an interface IP address as the sole inside global IP address:  
Step 1. As with dynamic and static NAT, configure theto identify inside interfaces. **ip nat inside** interface sub-command  
Step 2. As with dynamic and static NAT, configure theto identify outside interfaces. **ip nat outside** interface subcommand  
Step 3. As with dynamic NAT, interfaces. configure an ACL that matches the packets entering inside  
Step 4. Configure the global configuration command, referring to the ACL created in step 3 and to the interface **ip nat inside source list acl-number interface type/number overload**  
whose IP address will be used for translations.

```
ip nat inside source list 1 interface serial 0/0/0 overload command has several parameters
The matching ACL 1 have their addresses translated. The list 1 parameter means the same thing as it does for dynamic NAT: inside local IP addresses interface serial 0/0/0 parameter means that
the only inside global IP address available is the IP address of the NAT router’s interface serial 0/0/0.
The router creates one NAT table entry for each unique combination of inside local IP address and port
```

NAT Troubleshooting  
Reversed inside and outside  
Static NAT: Check the address first and the inside global IP address second. **ip nat inside source static** command to ensure it lists the inside local

address first and the inside global IP address second.  
Dynamic NAT (ACL): Ensure that the ACL configured to match packets sent by the inside hosts match that host’s packets before any NAT translation has occurred.

Dynamic NAT (pool): For dynamic NAT without PAT, ensure that the pool has enough IP addresses\

A large or growing value in the second misses counter in thecommand output can indicate this problem. Also, compare the configured pool to the list of **show ip nat statistics**  
addresses in the NAT translation table the problem may be that the configuration intended to use PAT and is missing the overload **(show ip nat translations** ). Finally, if the pool is small,  
keyword  
PAT: It is easy to forget to add the overload option  
perhaps NAT has been configured correctly, but the packets. an ACL exists on one of the interfaces, discarding

IOS processes ACLs before NATafter translating the addresses with NAT.. For packets exiting an interface, IOS processes any outbound ACL

User traffic required  
IPv4 routing

4.0 IP Services  
4.7 Explain the forwarding percongestion, policing, shaping -hop behavior (PHB) for QoS such as classification, marking, queuing,

QoSthan storing and forwarding a message. defines these actions as per-hop behaviors (PHBs),which isa formal term to refer to actions other

## QoS: Managing Bandwidth, Delay, Jitter, and Loss

```
four characteristics of network traffic:
Bandwidth
refers to the speed of a link, in bits per second (bps)
helps to also think of bandwidth as the capacity of the link, in terms of how many bits can be sent over the link per second.
Delay
the time between sending one packet and that same packet arriving at the destination host
Round-trip delay
Jitter the time it takes to send one packet between two hosts and receive one back
variation in one-way delay between consecutive packets sent by the same application
Loss
the number of lost messages, usually as a percentage of packets sent.
people think of loss as something caused by faulty cabling or poor WAN services. That is one cause. However, more loss happens because of the normal operation of the
networking devices, in which the devices’ queues get too full, so the device has nowhere to put new packets, and it discards the packet.
```

Types of Traffic  
Data Applications
