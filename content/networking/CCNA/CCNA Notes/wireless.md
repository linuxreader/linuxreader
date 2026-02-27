This chapter covers the following exam topics:

1.0 Network Fundamentals

1.1 Explain the role and function of network components

1.1.d Access Points

1.11 Describe wireless principles

1.11.a Nonoverlapping Wi-Fi channels

1.11.b SSID

1.11.c RF

Comparing Wired and Wireless Networks

Wireless devices must adhere to a common standard (IEEE 802.11).

Wireless coverage must exist in the area where devices are expected to use it.

Wireless LAN Topologies

IEEE 802.11 WLANs are always half duplex because transmissions between stations use the same frequency or channel.

Basic Service Set

before a device can participate, it must advertise its capabilities and then be granted permission to join. The 802.11 standard calls this a basic service set (BSS). At the heart of every BSS is a wireless access point (AP),

The AP operates in infrastructure mode, which means it offers the services that are necessary to form the infrastructure of a wireless network. The AP also establishes its BSS over a single wireless channel. The AP and the members of the BSS must all use the same channel to communicate properly.


the operation of a BSS hinges on the AP, the BSS is bounded by the area where the AP’s signal is usable. This is known as the basic service area (BSA) or cell.

the cell is shown as a simple shaded circular area that centers around the AP itself.

It advertises the existence of the BSS so that devices can find it and try to join. To do that, the AP uses a unique BSS identifier (BSSID) that is based on the AP’s own radio MAC address.

Wireless devices must also have unique MAC addresses to send wireless frames at Layer 2 over the air.

the AP advertises the wireless network with a Service Set Identifier (SSID), which is a text string containing a logical name. Think of the BSSID as a machine-readable name tag that uniquely identifies the BSS ambassador (the AP), and the SSID as a nonunique, human-readable name tag that identifies the wireless service.

Membership with the BSS is called an association. A wireless device must send an association request to the AP and the AP must either grant or deny the request. Once associated, a device becomes a client, or an 802.11 station (STA), of the BSS.

As long as a wireless client remains associated with a BSS, most communications to and from the client must pass through the AP,

By using the BSSID as a source or destination address, data frames can be relayed to or from the AP.

SSID. 
• 'MyNetwork&quot; 
BSSID: ">

By sending data through the AP first, the BSS remains stable and under control.

Distribution System

The 802.11 standard refers to the upstream wired Ethernet as the distribution system (DS) for the wireless BSS,

the AP is in charge of mapping a virtual local-area network (VLAN) to an SSID.

SSID. &quot; 
• MyNetwork&quot; 
BSS 
BSSID: 

The AP must be connected to the switch by a trunk link that carries the VLANs.

The AP uses the 802.1Q tag to map the VLAN numbers to the appropriate SSIDs.

VLANs 10, 20, 30 
SSID: 'MyNetwork&quot; 
SSID: &quot;YourNetwork&quot; 
BSSID: 
SSID: 'Guest&quot; 
BSSID: 
BSSID: 
F igure 26-7 Supporting Multiple SSIDs on One AP ">

The AP then appears as multiple logical APs—one per BSS—with a unique BSSID for each. With Cisco APs, this is usually accomplished by incrementing the last digit of the radio’s MAC address for each SSID.

Even though wireless clients can be distributed across many SSIDs, all of those clients must share the same AP’s hardware and must contend for airtime on the same channel.

Extended Service Set

When APs are placed at different geographic locations, they can all be interconnected by a switched infrastructure. The 802.11 standard calls this an extended service set (ESS),

Ideally, any SSIDs that are defined on one AP should be defined on all the APs in an ESS


BSS-I 
AP-I 
BSSID: 
SSID: &quot; 
My Network&quot; 
BSS-2 
AP-2 
BSSID: 
SSID: &quot;MyNetwork&quot; 

Passing from one AP to another is called roaming

Each AP offers its own BSS on its own channel, to prevent interference between the APs. As a client device roams from one AP to another, it must scan the available channels to find a new AP (and BSS) to roam toward.

Independent Basic Service Set

The 802.11 standard allows two or more wireless clients to communicate directly with each other, with no other means of network connectivity. This is known as an ad hoc wireless network, or an independent basic service set (IBSS),

One of the devices must take the lead and begin advertising a network name and the necessary radio parameters, much like an AP would do.

Other Wireless Topologies

Repeater

A wireless repeater takes the signal it receives and repeats or retransmits it in a new cell area around the repeater.

Client A 
Repeater 
Client B ">

Some repeaters can use two transmitters and receivers to keep the original and repeated signals isolated on different channels. One transmitter and receiver pair is dedicated to signals in the AP’s cell, while the other pair is dedicated to signals in the repeater’s own cell.

Workgroup Bridge

WGB acts as an external wireless network adapter for a device that has none.

Client A 
WGB 
Cliente 

You might encounter two types of workgroup bridges:

Universal workgroup bridge (uWGB): A single wired device can be bridged to a wireless network.

Workgroup bridge (WGB): A Cisco-proprietary implementation that allows multiple wired devices to be bridged to a wireless network.

Outdoor Bridge

act as a bridge to form a single wireless link from one LAN to another over a long distance.

One AP configured in bridge mode is needed on each end of the wireless link. Special purpose antennas are normally used with the bridges to focus their signals in one direction

Bridge 
LAN В ">

A point-to-multipoint bridged link allows a central site to be bridged to several other sites

00 
V NVI ">

Mesh Network

In a mesh topology, wireless traffic is bridged from AP to AP, in a daisy-chain fashion, using another wireless channel.

Mesh APs can leverage dual radios—one using a channel in one range of frequencies and one a different range. Each mesh AP usually maintains a BSS on one channel, with which wireless clients can associate. Client traffic is then usually bridged from AP to AP over other channels as a backhaul network.

The mesh network runs its own dynamic routing protocol to work out the best path for backhaul traffic to take across the mesh APs.


RF Overview

Electromagnetic waves do not travel in a straight line. Instead, they travel by expanding in all directions away from the antenna.

frequency of the wave, or the number of times the signal makes one complete up and down cycle in 1 second

1 Second 
Frequency = 4 cycles/second 
= 4 Hertz 

A hertz (Hz) is the most commonly used frequency unit and is nothing other than one cycle per second.

Unit 
Hertz 
Kilohertz 
Megahertz 
Gigahertz 
Abbreviation 
kHz 
MHz 
GHz 
Meaning 
Cycles per second 
1000 Hz ">

3 kHz to 300 GHz is commonly called radio frequency (RF).

Wireless Bands and Channels

Because a range of frequencies might be used for the same purpose, it is customary to refer to the range as a band of frequencies.

the range from 530 kHz to around 1710 kHz is used by AM radio stations; therefore, it is commonly called the AM band or the AM broadcast band.

One of the two main frequency ranges used for wireless LAN communication lies between 2.400 and 2.4835 GHz. This is usually called the 2.4-GHz band, even though it does not encompass the entire range between 2.4 and 2.5 GHz

The other wireless LAN range is usually called the 5-GHz band because it lies between 5.150 and 5.825 GHz. The 5-GHz band actually contains the following four separate and distinct bands:

5.150 to 5.250 GHz

5.250 to 5.350 GHz

5.470 to 5.725 GHz

5.725 to 5.825 GHz

Gap between 5.350 and 5.470

this gap exists and cannot be used for wireless LANs.

Do not worry about memorizing the band names or exact frequency ranges; just be aware of the two main bands at 2.4 and 5 GHz.

A frequency band contains a continuous range of frequencies.

bands are usually divided into a number of distinct channels. Each channel is known by a channel number and is assigned to a specific frequency.

2.412 
2.417 
2.422 
2427 
6 
2432 2437 2.442 
Dsss: 22 MHz 
OFDM: 20 MHz 
2.44? 
2,452 
2457 
11 
2462 2467 2.472 
2484 

Channel 
U-Nll-2 
U-Nll-2 Extended 
112 116 124 128132 136 140 
5.700 
U-Nll-3 
149153157161 
36 40 44 48 52 56 60 64 
GHz 
5180 5240 5260 5.320 
5.745 
5825 ">

In the 5-GHz band, this is the case because each channel is allocated a frequency range that does not encroach on or overlap the frequencies allocated for any other channel. In other words, the 5-GHz band consists of nonoverlapping channels.

APs and Wireless Standards

Amend- 
ment 
802.11- 
1997 
802.11b 
802.118 
802.11a 
802.11n 
802.11ac 
802.11ax 
2.4 
GHz 
Yes 
Yes 
Yes 
No 
Yes 
No 
Yes 
GHz 
No 
No 
No 
Yes 
Yes 
Yes 
Yes 
Max Data 
Rate 
2 Mbps 
11 Mbps 
54 Mbps 
54 Mbps 
600 Mbps 
6.93 Gbps 
4x 
802.11ac 
Notes 
The original 802.11 standard rati- 
fied in 1997 
Introduced in 1999 
Introduced in 2003 
Introduced in 1999 
HT (high throughput), introduced 
in 2009 
VHT (very high throughput), in- 
troduced in 2013 
High Efficiency Wireless, Wi-Fi6; 
expected late 2019; will operate 
on other bands too, as they be- 
come available ">

Wireless client devices and APs can be compatible with one or more amendments;

APs can usually operate on both bands simultaneously to support any clients that might be present on each band.

wireless clients typically associate with an AP on one band at a time, while scanning for potential APs on both bands. The band used to connect to an AP is chosen according to the operating system, wireless adapter driver, and other internal configuration.

Cisco APs have dual radios (sets of transmitters and receivers) to support BSSs on one 2.4-GHz channel and other BSSs on one 5-GHz channel simultaneously. Some models also have two 5-GHz radios that can be configured to operate BSSs on two different channels at the same time, providing wireless coverage to higher densities of users that are located in the same vicinity.

RF signals propagate or reach further on the 2.4-GHz band than on the 5-GHz band. They also tend to penetrate indoor walls and objects easier at 2.4 GHz than 5 GHz. However, the 2.4-GHz band is commonly more crowded with wireless devices. Remember that only three nonoverlapping channels are available, so the chances of other neighboring APs using the same channels is greater. In contrast, the 5-GHz band has many more channels available to use, making channels less crowded and experiencing less interference.
