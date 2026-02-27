2.0 Network Access

2.3 Configure and verify Layer 2 discovery protocols (Cisco Discovery Protocol and LLDP)

4.0 IP Services

4.2 Configure and verify NTP operating in a client and server mode

4.5 Describe the use of syslog features including facilities and levels

# System Message Logging (Syslog)

Sending Messages in Real Time to Current Users

-   Telnet and SSH users, the device requires a two-step process before the user sees the messages.
    
-   global configuration setting—logging monitor—tells IOS to enable the sending of log messages to all logged users.
    
-   must also issue the terminal monitor EXEC command during the login session, which tells IOS that this terminal session would like to receive log messages.
ogging console  
Console  
logging monitor

-   IOS -  
    terminal monitor  
    (NO Messages)  
    Figure 9-1 IOS Processing for Log Messages to Current Users ">

Storing Log Messages for Later Review

two primary means to keep a copy

-   can store copies of the log messages in RAM by virtue of the logging buffered global configuration command.

any user can come back later and see the old log messages by using the show logging EXEC command.

store log messages centrally to a syslog server.

syslog protocol - a UDP protocol to send messages to a syslog server for storage

To configure a router or switch to send log messages to a syslog server, add the logging host {address | hostname } global command

![Pasted image 20230120151144 1.png](Pasted%20image%2020230120151144%201.png)

Log Message Format

*Dec 18 17:10:15.079: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to down

A timestamp: *Dec 18 17:10:15.079

The facility on the router that generated the message: %LINEPROTO

The severity level: 5

A mnemonic for the message: UPDOWN

The description of the message: Line protocol on Interface FastEthernet0/0, changed state to down

you can at least toggle on and off the use of the timestamp (which is included by default) and a log message sequence number (which is not enabled by default). Example 9-1 reverses those defaults by turning off timestamps and turning on sequence numbers.

Example 9-1 Disabling Timestamps and Enabling Sequence Numbers in Log 
Wessages 
Click here to view code image 
Rl(config)# no service timestamps 
Rl(config)# service sequence-numbers 
Rl(config)# end 
000011: %SYS-5-CONFIG I: Configured from console by console

Log Message Severity Levels

the lower the number, the more severe the event that caused the message.

![Keyword 
Emergency 
Alert 
Critical 
Error 
Warning 
Notification 
Informational 
Debug 
Numeral 
2 
3 
4 
5 
6 
7 
Description 
System unusable 
Immediate action required 
Critical Event (Highest Of 3) 
Error Event (Middle Of 3) 
Warning Event (Lowest Of 3) 
Normal, More Important 
Normal, Less Important 
Requested by User Debug 
Impactful 
Normal 
Debug 
Figure 9-3 Syslog Message Severity Levels by Keyword and Numeral](blob:file:///980bce15-c623-489b-bd14-11e70f4fffc2)

last level in the figure is used for messages requested by the debug command

Table 9-2 summarizes the configuration commands used to enable logging and to set the severity level for each type.

For example, the command logging console 4 causes IOS to send severity level 0–4 messages to the console

![Table 9-2 How to Configure Logging Message Levels for Each Log Service 
Key 
Topic 
Service 
Console 
Monitor 
Buffered 
Syslog 
To Enable Logging 
logging console 
logging monitor 
logging buffered 
logging host address I 
hostname 
To Set Message Levels 
logging console level-name I level- 
number 
logging monitor level-name I 
level-number 
logging buffered level-name I 
level-number 
logging trap level-name I level- 
number](blob:file:///b598183b-1e6d-47a2-bd46-61cdb8416e46)

Configuring and Verifying System Logging

![172.16.1.0/24 
172.16.1.1 
GO/I 
RI 
17216.21 
GO/2 
172.162.2 
172.16.3.0/24 
SW2 
172.16.3.9 
Figure 9-4 Sample Network Used in Logging Examples](blob:file:///441c47eb-3766-4f0d-8251-7181b1dd2aee)

![Example 9-2 Syslog Configuration on RI 
Click here to view code image 
logging 
logging 
logging 
logging 
logging 
console 7 
monitor debug 
buffered 4 
host 172.16.3.9 
trap warning](blob:file:///89cbe1a2-0745-46e2-923a-30bb5f816e43)

the example configures the same message level at the console and for terminal monitoring (level 7, or debug), and the same level for both buffered and logging to the syslog server (level 4, or warning). The levels may be set using the numeric severity level or the name as shown earlier in Figure 9-3.

show logging command confirms those same configuration settings and also lists the log messages per the logging buffered configuration.

![Example 9-3 Viewing the Configured Log Settings per the Earlier Example 
Click here to view code image 
Rl# show logging 
Syslog logging: enabled (0 messages dropped, 
No Active Message Discriminator. 
No Inactive Message Discriminator. 
3 messages rate-limited 
Console logging: level debugging, 45 messages logged, xml disabl 
filtering disabled 
Monitor logging: level debugging, 0 messages logged, xml disable 
filtering disabled 
Buffer logging: level warnings, e messages logged, xml disabled, 
filtering disabled 
Exception Logging: size (8192 bytes) 
Count and timestamp logging messages: 
Persistent logging: disabled 
disabled](blob:file:///09d59061-71bc-43e6-a415-cca2e3f9e5a8)

![No active filter modules. 
Trap logging: level warnings, e message lines logged 
Logging to 172.16.3.9 (udp port 514, audit disabled, 
link up), 
0 message lines logged, 
0 message lines rate-limited, 
0 message lines dropped-by-MD, 
xml disabled, sequence number disabled 
filtering disabled 
Logging Source-Interface: 
Log Buffer (8192 bytes): 
VRF Name:](blob:file:///b920da0f-adeb-4912-8aeb-78422dee63af)

If any log messages had been buffered, the actual log messages would be listed at the end of the command

clear out the old messages from the log with the clear logging EXEC command.

![Rl# configure terminal 
Enter configuration commands, 
RI (config)# interface gø/l 
RI (config-if)# shutdown 
RI 
one per line. 
End with CNTL/Z. 
*Oct 21 %LINK-5-CHANGED: Interface GigabitEthernetO/1, 
*Oct 21 %LINEPROTO-5-UPDOWN: Line protocol on Interfac 
RI (config-if)# no shutdown 
RI 
*Oct 21 
: %LINK-3-UPDOWN: Interface GigabitEthernetØ/1, 
*Oct 21 %LINEPROTO-5-UPDOWN: Line protocol on Interfac 
RI (config-if)# AZ 
*Oct 21 
: %SYS-5-CONFIG I: Configured from console by co 
Rl# show logging 
! Skipping about 20 lines, the same lines in Example 9-3, 
until the 
Log Buffer (8192 bytes) : 
*Oct 21 %LINK-3-UPDOWN: 
Interface GigabitEthernetØ/1,](blob:file:///50a65acc-0691-4338-a954-5e888ca303e9)

The debug Command and Log Messages

debug EXEC command gives the network engineer a way to ask IOS to monitor for certain internal events, with that monitoring process continuing over time, so that IOS can issue log messages when those events occur

debug remains active until some user issues the no debug command with the same parameters, disabling the debug.

![Example 9-5 Using debug ip ospf hello from RI's Console 
Click here to view code image 
Rl# debug ip ospf hello 
OSPF 
*Aug 
*Aug 
*Aug 
hello 
IS on 
10 OSPF-I HELLO GiØ/1: 
10 OSPF-I HELLO GiØ/2: 
10 OSPF-I HELLO GiØ/2: 
debuggi ng 
Send hello to 224.Ø.Ø.5 area 
Rcv hello from 2.2.2.2 area 
Send hello to 224.0.0.5 area](blob:file:///0a51c53d-835e-4dcd-ae27-8ad10d86b942)

anyone logged in with SSH at the time Example 9-4’s output was gathered would not have seen the output, even with the logging monitor debug command configured on router R1, without first issuing a terminal monitor command.

all enabled debug options use router CPU, which can cause problems for the router. You can monitor CPU use with the show process cpu command

use caution when using debug commands carefully on production devices.

the more CLI users that receive debug messages, the more CPU that is consumed.

## Network Time Protocol (NTP)

Devices send timestamps to each other with NTP messages, continually exchanging messages, with one device changing its clock to match the other, eventually synchronizing the clocks.

How NTP defines the sources of time data (reference clocks) and how good each time source is (stratum).

Setting the Time and Timezone

NTP works best if you set the device clock to a reasonably close time before enabling the NTP client function with the ntp server command

I should set the time to 8:52 p.m., set the correct date and timezone, and even tell the device to adjust for daylight savings time—and then enable NTP

Example 9-7 shows how to set the date, time, timezone, and daylight savings time.

![Example 9-7 Setting the Date/Time with clock set, Plus Timezone/DST 
Click here to view code image 
Rl# configure terminal 
Enter configuration commands, one per line. End with CNTL/Z. 
RI (config)# clock timezone EST -5 
RI clock summer-time EDT recurring 
RI (config)# AZ 
clock set 21 October 2ø15 
*Oct 21 me: %SYS-6-CLOCKUPDATE: 
System clock has been upda 
Rl# show clock 
EDT Wed 
Oct 21 2015](blob:file:///12712855-da00-4f5f-9301-b908790790c9)

You should set the first two commands before setting the time of day with the clock set EXEC command because the two configuration commands impact the time that is set.

chose EDT because it is the acronym for daylight savings time in that same EST time zone. Finally, the recurring keyword tells the router to spring forward an hour and fall back an hour automatically over the years.

clock set EXEC command

uses a time syntax with a 24-hour format, not with a 12-hour format plus a.m./p.m.).

The show clock command (issued seconds later) lists that time, but also notes the time as EDT, rather than UTC time.

Basic NTP Configuration

two ntp configuration commands

ntp master {stratum-level}: NTP server mode—the device acts only as an NTP server, and not as an NTP client. The device gets its time information from the internal clock on the device.

ntp server {address | hostname} : NTP client/server mode—the device acts as both client and server. First, it acts as an NTP client, to synchronize time with a server. Once synchronized, the device can then act as an NTP server, to supply time to other NTP clients.

![ntp server 172.162.2 
NTP Client/ server 
Stratum 4 
ntp server 172.16.3.3 
NTP Client / server 
Stratum 3 
172.16.3_3 
GOII 
Mp master 
GO,'2 
NTP server 
Stratum 2 
Figure 9-5 RI as NTP Client, R2 as Client/Server, R3 as Server](blob:file:///ad94907d-f81d-499c-ad54-b5db7eb548e3)

![Example 9-8 NTP Client/Server Configuration 
Click here to view code image 
! Configuration on RI: 
ntp server 172.16.2.2 
Click here to view code image 
! Configuration on R2: 
ntp server 172.16.3.3 
Click here to view code image 
! Configuration on R3: 
ntp master 2](blob:file:///643a88fe-2d60-4ee1-9684-09c87d798ae2)

show ntp status command

lists a status of synchronized, which confirms the NTP client has completed the process of changing its time to match the server’s time. Any router acting as an NTP client will list “unsynchronized” in that first line until the NTP synchronization process completes with at least one server. It also confirms the IP address of the server—this device’s reference clock—with the IP address configured in Example 9-8 (172.16.2.2).

![Example 9-9 Verifying NTP Client Status on RI 
Click here to view code image 
Rl# show ntp status 
Clock is synchronized, stratum 4, reference is 172.16.2.2 
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 
ntp uptime is 1553800 (1/100 of seconds), resolution is 4000 
reference time is DA5E7147.56CADEA7 (19:54:31.339 EST Thu Feb 4 2016) 
clock offset is 0.0986 msec, root delay is 2.46 msec 
root dispersion is 22.19 msec, peer dispersion is 5.33 msec 
loopfilter state is 'CTRL ' 
system poll interval is 64, 
(Normal Controlled Loop), drift is e. 000000G 
last update was 530 sec ago.](blob:file:///43b9da43-57f0-4a98-a836-4e093bd06ef3)

show ntp associations

lists all the NTP servers that the local device can attempt to use, with status information about the association between the local device (client) and the various NTP servers.

Status on RI and R2  
Click here to view code image  
RI# show ntp associations  
! This output is  
taken from router RI, acting in client/server mode  
address  
sys . peer ,  
ref  
clock  
172.16. 3.3

# selected,

st  
3  
when poll  
50  
64

-   candidate,  
    reach  
    377  
    outlyer ,  
    delay offset  
    1.223 0.090  
    x falseticker,  
    disp  
    4.46f  
    cor  
    Click here to view code image  
    R2# show ntp associations  
    ! This output is taken from router R2, acting in  
    client/server mode  
    address  
    ref clock  
    1  
    st  
    2  
    when poll  
    49  
    64  
    reach  
    377  
    outlyer,  
    delay  
    1.220  
    offset  
    -7.758  
    disp  
    3.69  
    *A-172.16.3.3 127.127.1.  
    sys .peer, # selected,  
    candidate,  
    x  
    falseticker, ">

### NTP Reference Clock and Stratum

Devices that act solely as an NTP server get their time from either internal device hardware or from some external clock using mechanisms other than NTP.

NTP servers and clients use a number to show the perceived accuracy of their reference clock data based on stratum level. The lower the stratum level, the more accurate the reference clock is considered to be. An NTP server that uses its internal hardware or external reference clock sets its own stratum level. Then, an NTP client adds 1 to the stratum level it learns from its NTP server, so that the stratum level increases the more hops away from the original clock source.

For instance, back in Figure 9-5, you can see the NTP primary server (R3) with a stratum of 2. R2, which references R3, adds 1 so it has a stratum of 3. R1 uses R2 as its NTP server, so R1 adds 1 to have a stratum of 4. These increasing stratum levels allow devices to refer to several NTP servers and then use time information from the best NTP server, best being the server with the lowest stratum level.

Routers and switches use the default stratum level of 8 for their internal reference clock based on the default setting of 8 for the stratum level in the ntp master [stratum-level] command. The command allows you to set a value from 1 through 15; in Example 9-8, the ntp master 2 command set router R3’s stratum level to 2.

Note

NTP considers 15 to be the highest useful stratum level, so any devices that calculate their stratum as 16 consider the time data unusable and do not trust the time. So, avoid setting higher stratum values on the ntp master command.

![Example 9-11 Examining NTP Server, Reference Clock, and Stratum Data 
Click here to view code image 
R3# show ntp status 
Clock is synchronized, stratum 2, reference is 127.127.1.1 
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision i 
ntp uptime is 595300 (1/100 of seconds), resolution is 4000 
reference time is EOF9174C.87277EBB (16:13:32.527 daylight sat Aug 1 
clock offset is 0.0000 msec, root delay is 0.00 msec 
root dispersion is 0.33 msec, peer dispersion is 0.23 msec 
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is e. 00001 
system poll interval is 16, last update was 8 sec ago. 
R3# show ntp associations 
address 
*n.127 • 127 • 1.1 
ref clock 
. LOCL 
st 
1 
when 
15 
poll 
16 
reach delay 
377 0.øoe 
falseticker, 
off 
o. 
sys .peer, # selected, + candidate, 
outlyer, x](blob:file:///f789da20-724e-41ae-9ebd-7fe1c5ef7c68)

### Redundant NTP Configuration

an enterprise could use NTP to reference NTP servers that use an atomic clock as their reference source, like the NTP primary servers in Figure 9-6, which happen to be run by the US National Institute of Standards and Technology (NIST) (see tf.nist.gov).

![Stratum 1 
Stratum 2 
NTP Primary 
servers (MST) 
Internet 
NTP Clienvserver 
Stratum 3 
9-6 Stratum Levels VVhen Using an Internet-based Stratum 1 NTP Server](blob:file:///b0dca087-9689-487e-86ec-04b89f2d82ee)

NTP primary server and NTP secondary server.

An NTP primary server acts only as a server, with a reference clock external to the device, and has a stratum level of 1, like the two NTP primary servers shown in Figure 9-6. NTP secondary servers are servers that use client/server mode as described throughout this section, relying on synchronization with some other NTP server.

![Example 9-12 NTP Configuration on RI, R2 per Eigure 9-6 
Click here to view code image 
ntp server time-a-b-nist .gov 
ntp server time-a-g.nist.gov](blob:file:///77c1af69-5be5-421e-983f-f15baed94077)

After losing their reference clock, R1 and R2 could no longer be useful NTP servers to the rest of the enterprise.

To overcome this potential issue, the routers can also be configured with the ntp master command, resulting in this logic:

Establish an association with the NTP servers per the ntp server command.

Establish an association with your internal clock using the ntp master stratum command.

Set the stratum level of the internal clock (per the ntp master {stratum-level} command) to a higher (worse) stratum level than the Internet-based NTP servers.

Synchronize with the best (lowest) known time source, which will be one of the Internet NTP servers in this scenario

ntp master 7 command, with a much higher stratum,

![Example 9-13 NTP Configuration on RI and R2 to Protect Against Internet 
Failures 
Click here to view code image 
ntp server time-a-b-nist .gov 
ntp server time-a-g.nist.gov 
ntp master 7](blob:file:///6679839f-8a17-4d63-a07f-e4e09cda1598)

### NTP Using a Loopback Interface for Better Availability

what happens when one interface on R4 fails

for any NTP clients that had referred to that specific IP address

There would likely still be a route to reach R4 itself.

The NTP client would not be able to send packets to the configured address because that interface is down.

![R2 
NTP server 
Figure 9-7 The Availability Issue ofReferencing an NTP Server's Physical Interface IP Address](blob:file:///efe7ee03-a9b4-4d2e-ad83-ba6db3b913a7)

loopback interface to meet that exact need

once configured, it remains in an up/up state as long as

Key Topic.

The router remains up.

You do not issue a shutdown command on that loopback interface.

![Example 9-14 NTP Client/Server Configuration on RI and R2 Using a Loopback 
Interface 
Click here to view code image 
! Configuration on RI, 
ntp server 172.16.9.9 
Click here to view code image 
a client 
! Configuration on R2 for its server function 
interface loopback ø 
L 
ip address 172.16.9.9 255.255.255.ø 
ntp master 4 
ntp source loopback 
! Verification on router R2 
R2# show interfaces loopback ø 
Loopback0 is up, line protocol is up 
Hardware is Loopback](blob:file:///46fedc78-dcd4-46c6-86d5-61d91857c93b)

![Internet address is 172.16.9.9/24 
! lines omitted for brevity](blob:file:///a0215c7c-df76-4921-8d73-7d5d7b11785c)

## Analyzing Topology Using CDP and LLDP

### Examining Information Learned by CDP

Cisco-proprietary

Layer 2 protocol

does not rely on a working Layer 3 protocol

Devices that support CDP learn information about others by listening for the advertisements sent by other devices.

CDP discovers several useful details from the neighboring Cisco devices:

Device identifier: Typically the host name

Address list: Network and data-link addresses

Port identifier: The interface on the remote router or switch on the other end of the link that sent the CDP advertisement

Capabilities list: Information on what type of device it is (for example, a router or a switch)

Platform: The model and OS level running on the device

Cisco IP Phones use CDP to learn the data and voice VLAN IDs as configured on the access switch.

![Table 9-3 show cdp Commands That List Information About Neighbors 
Command 
show cdp neigh- 
bors [type number] 
show cdp neigh- 
bors detail 
show cdp entry 
name 
Description 
Lists one summary line of information about each 
neighbor or just the neighbor found on a specific inter- 
face if an interface was listed 
Lists one large set (approximately 15 lines) of informa- 
tion, one set for every neighbor 
Lists the same information as the show cdp neighbors 
detail command, but only for the named neighbor 
(case sensitive)](blob:file:///2398a655-1078-49a1-bd8e-0103fbbca8ff)

routers and switches support the same CDP commands, with the same parameters and same types of output.

![Cisco 2960XR Switches (WS-2960XR-24TS-l) 
Gil/0/1 
Fred 
GilW24 
SWI 
Gil /0/2 
Barney 
02002222.2222 
Gil,W21 
Gil/O'2 
GiO,W1 0200.5555.5555 
Cisco ISRIK Router 
Figure 9-8 Small Network Used in CDP Examples](blob:file:///65b036b9-4ce9-4ab5-abbf-e54bbaa228fb)

Example 9-15 show cdp neighbors Command Examples: SW2  
Click here to view code image  
SW2# show cdp neighbors  
Capability Codes: R  
S  
D  
Router,

-   Switch,  
    Remote,  
    T  
    c  
    Trans  
    Host,  
    CVTA,  
    Bridge, B  
    Source Route Bri  
    1  
    M  
    IGMP, r  
    Repeater, p  
    Two-port Mac Relay  
    Device ID  
    SWI  
    RI  
    Local Intrfce  
    Gig 1/0/21  
    Gig 1/0/2  
    Holdtme  
    155  
    131  
    Capability  
    Platform  
    ws-C296ex  
    Cili1-8P  
    Total cdp entries displayed  
    .2 ">

To ensure all devices receive a CDP message, the Ethernet header uses a multicast destination MAC address (0100.0CCC.CCCC).

the device processes the message and then discards it, rather than forwarding it

show cdp neighbors detail

lists the full name of the switch model (WS-2960XR-24TS-I) and the IP address configured on the neighboring device.

![SW2# show cdp neighbors detail 
Device ID: SWI 
Entry address(es): 
IP address: 1.1.1.1 
Platform: cisco WS-C2960XR-24TS-1, 
Capabilities: Switch IGMP 
Interface: 
Holdtime . 
Version : 
GigabitEthernet1/0/21, 
144 sec 
Port ID (outgoing port): 
GigabitE 
Version 
Cisco IOS Software, Software (C2960X-UNIVERSALK9-M), 
Technical Support: http://www.cisco.com/techsupport 
Copyright (c) 1986-2018 by Cisco Systems, Inc. 
Compiled Thu 13-Sep-18 03 by prod rel team](blob:file:///79b58992-46ce-4683-8122-3a240e3e3588)

![advertisement version: 2 
Protocol Hello: OUI=oxoeoooc, 
VTP Management Domain: 'fred' 
Native VLAN: 1 
Duplex: full 
Management address(es) : 
IP address: 1.1.1.1 
Device ID: RI 
Entry address(es): 
IP address: 10.12.25.5 
Protocol ID=oxØ112; 
payload len=27, 
Interface: 
Holdtime . 
Capabilities: Router Switch IGMP 
Port ID (outgoing port): GigabitEt 
GigabitEthernet1/0/2, 
151 sec](blob:file:///41b70243-659b-48eb-8a21-e4eeb14899e3)

![Version : 
Cisco IOS Software [Fuji], ISR Software (ARMV8EB LINUX IOSD-UNIVERSA 
Technical Support: http://www.cisco.com/techsupport 
Copyright (c) 1986-2018 by Cisco Systems, Inc. 
Compiled Tue 27-Mar-18 10:56 by mcpre 
advertisement version : 
2 
VTP Management Domain: ' ' 
Duplex: full 
Management address(es) : 
IP address: 10.12.25.5 
Total cdp entries displayed 
.2](blob:file:///b8cc1ddc-e463-4ddf-9f81-4ecebf1796c2)

The show cdp entry name command lists the exact same details shown in the output of the show cdp neighbors detail command, but for only the one neighbor listed in the command.

Cisco recommends that CDP be disabled on any interface that might not have a need for CDP. For switches, any switch port connected to another switch, a router, or to an IP phone should use CDP.

### Configuring and Verifying CDP

IOS typically enables CDP globally and on each interface by default. You can then disable CDP per interface with the no cdp enable interface subcommand

re-enable it with the cdp enable interface subcommand

disable and re-enable CDP globally on the device

no cdp run and cdp run global commands

![Table 9-4 Commands Used to Verify CDP Operations 
Command 
show cdp 
show cdp inter- 
face [type 
number] 
show cdp traffic 
Description 
States whether CDP is enabled globally and lists the de- 
fault update and holdtime timers 
States whether CDP is enabled on each interface, or a 
single interface if the interface is listed, and states up- 
date and holdtime timers on those interfaces 
Lists global statistics for the number of CDP advertise- 
ments sent and received](blob:file:///518b3cfe-18e1-4a81-9187-be7eea7b496b)

![Example 9-17 show cdp Commands That Show CDP Status 
Click here to view code image 
SW2# show cdp 
Global CDP information: 
Sending CDP packets every 60 seconds 
Sending a holdtime value of 180 seconds 
Sending CDPv2 advertisements is enabled 
SW2# show cdp interface GigabitEthernet1/0/2 
GigabitEthernet1/Ø/2 is up, line protocol is up 
Encapsulation ARPA 
Sending CDP packets every 60 seconds 
Holdtime is 180 seconds 
SW2# show cdp traffic 
CDP counters : 
Total packets 
Hdr syntax: 0, 
No memory: 0, 
CDP version 1 
CDP version 2 
output: 304, Input: 305 
Chksum error: 
Invalid packet 
advertisements 
advertisements 
0, Encaps failed 
output : 
output : 
0, Input 
304, Input: 305](blob:file:///7787792d-cb59-4452-b119-10c3deeda082)

send time and the hold time. CDP sends messages every 60 seconds by default, with a hold time of 180 seconds. The hold time tells the device how long to wait after no longer hearing from a device before removing those details from the CDP tables. You can override the defaults with the cdp timer seconds and cdp holdtime seconds global commands, respectively.

## Examining Information Learned by LLDP

![SW2# show Ildp neighbors 
Capability codes: 
(R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device 
Hold-time 
(W) WLAN Access Point, (P) Repeater, 
(S) 
Station, (O) Other 
Device ID 
RI 
SWI 
Local Intf 
Gil/O/2 
Gil /0/21 
Capability 
120 
120 
Trans 
Host, 
CVTA, 
Port 
GiO/O/ 
Gil/Oj 
Total entries displayed: 2 
SW2# show cdp neighbors 
Capability Codes: R 
S 
D 
Router, 
Switch, 
Remote, 
T 
c 
Bridge, B 
Source Route Bri 
1 
IGMP, r 
Repeater, p 
M - Two-port Mac 
Device ID 
Local Intrfce 
Gig 1/0/21 
Gig 1/0/2 
displayed: 2 
Holdtme 
155 
131 
Capability 
SWI 
RI 
Total 
Relay 
Platform 
ws-C296ex 
Cili1-8P 
entries](blob:file:///c3dc6bd1-0f19-4496-9d7a-8b042e9acc13)

the LLDP output in the example does differ from CDP in a few important ways:

LLDP uses B as the capability code for switching, referring to bridge, a term for the device type that existed before switches that performed the same basic functions.

LLDP does not identify IGMP as a capability, while CDP does (I).

CDP lists the neighbor’s platform, a code that defines the device type, while LLDP does not.

LLDP lists capabilities with different conventions (see upcoming Example 9-19).

![SW2# show Ildp entry RI 
Capability codes: 
(R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device 
(W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other 
Local Intf: Gil/0/2 
Chassis id: 70ea .1a9a.d3Ø0 
Port id: GiØ/0/1 
Port Description: GigabitEthernet0/Ø/1 
System Name: RI 
System Description: 
Cisco IOS Software [Fuji], ISR Software (ARMV8EB LINUX IOSD-UNIVERSA 
Technical Support: http://www.cisco.com/techsupport 
Copyright (c) 1986-2018 by Cisco Systems, Inc. 
Compiled Tue 27-Mar-18 10:56 by mcpre 
Time remaining: 100 seconds 
System Capabilities: B,R 
Enabled Capabilities: R 
Management Addresses : 
IP: 10.12.25.5 
Auto Negotiation 
not supported 
Physical media capabilities 
not advertised 
Media Attachment Unit type 
not advertised 
Vlan ID: 
not advertised](blob:file:///947e4912-5bcb-4289-be21-d45a96066b85)

System Capabilities: What the device can do

Enabled Capabilities: What the device does now with its current configuration

LLDP uses the same messaging concepts as CDP, encapsulating messages directly in data-link headers. Devices do not forward LLDP messages so that LLDP learns only of directly connected neighbors. LLDP does use a different multicast MAC address (0180.C200.000E)

### Configuring and Verifying LLDP

Cisco devices default to disable LLDP

LLDP separates the sending and receiving of LLDP messages as separate functions.

LLDP support processing receives LLDP messages on an interface so that the switch or router learns about the neighboring device while not transmitting LLDP messages to the neighboring device.

the commands include options to toggle on|off the transmission of LLDP messages separately from the processing of received messages.

[no] lldp run: A global configuration command that sets the default mode of LLDP operation for any interface that does not have more specific LLDP subcommands (lldp transmit, lldp receive). The lldp run global command enables LLDP in both directions on those interfaces, while no lldp run disables LLDP.

[no] lldp transmit: An interface sub-command that defines the operation of LLDP on the interface regardless of the global [no] lldp run command. The lldp transmit interface subcommand causes the device to transmit LLDP messages, while no lldp transmit causes it to not transmit LLDP messages.

[no] lldp receive: An interface subcommand that defines the operation of LLDP on the interface regardless of the global [no] lldp run command. The lldp receive interface subcommand causes the device to process received LLDP messages, while no lldp receive causes it to not process received LLDP messages.

![Example 9-20 Enabling LLDP on All Ports, Disabling on a Few Ports 
Click here to view code image 
lldp run 
interface gigabitEthernet1/Ø/17 
no lldp transmit 
no lldp receive 
interface gigabitEthernet1/Ø/18 
no lldp receive](blob:file:///48b49040-6e65-45ee-a590-a3b5e0fcc11b)

![Example 9-21 Enabling LLDP on Limited Ports, Leaving Disabled on Most 
Click here to view code image 
interface gigabitEthernet1/Ø/19 
lldp transmit 
lldp receive 
interface gigabitEthernet1/0/20 
lldp receive](blob:file:///7640bbe6-77fa-41fe-97a7-8d5fd4d05b6e)

show lldp interface lists the interfaces on which LLDP is enabled.

![SW2# show Ildp 
Global LLDP Information: 
status: ACTIVE 
LLDP advertisements are sent every 30 seconds 
LLDP hold time advertised is 120 seconds 
LLDP interface reinitialisation delay is 2 seconds 
SW2# show Ildp interface gl/Ø/2 
GigabitEthernet1/Ø/2: 
Tx: enabled 
Rx: enabled 
Tx state: IDLE 
Rx state: WAIT FOR FRAME 
SW2# show Ildp traffic 
LLDP traffic statistics: 
Total 
Total 
Total 
Total 
Total 
Total 
Total 
frames out: 259 
entries aged: 0 
frames in: 257 
frames received in error 
frames discarded: 0 
TLVs discarded: 0 
TLVs unrecognized: 0](blob:file:///56b09a3a-ac43-4d2d-9691-279d8f24afdb)

like CDP, LLDP uses a send timer and hold timer for the same purposes as CDP. The example shows the default settings of 30 seconds for the send timer and 120 seconds for the hold timer. You can override the defaults with the lldp timer seconds and lldp holdtime seconds global commands
