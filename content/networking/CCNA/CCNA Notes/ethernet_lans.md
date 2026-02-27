

# 1.0 Network Fundamentals

# 1.1 Explain the role and function of network components

# 1.1.b L2 and L3 Switches

# 1.2 Describe characteristics of network topology architectures

# 1.2.e Small office/home office (SOHO)

# 1.3 Compare physical interface and cabling types

# 1.3.a Single-mode fiber, multimode fiber, copper

# 1.3.b Connections (Ethernet shared media and point-to-point)

802.3 (Ethernet Standards

10BASE-T

-   10 Mbps
-   Ethernet
-   802.3
-   Copper/ 100 m

100BASE-T

-   100 Mbps
-   Fast Ethernet
-   802.3u
-   Copper/ 100m

1000BASE-LX

-   1000 Mbps
-   Gigabit Ethernet
-   802.3z
-   Fiber, 5000 m

1000BASE-T

-   1000 Mbps
-   Gigabit Ethernet
-   802.3ab
-   Copper, 100m

10GBASE-T

-   10 Gbps
-   10 Gig Ethernet
-   802.3an
-   Copper 100m

Three most common

-   10BASE-T, 100BASE-T, and 1000BASE-T

Twisting of the wires helps reduce EMI.

Crosstalk

-EMI between wire pairs

Gigabit Ethernet interface Converter (GBIC)

-   The original form factor for a removeable transceiver for Gigabit interfaces; larger than SFPs

Small Form Pluggable (SFP)

-   The replacement for GBICs, used on gigabit interfaces, with a smaller size, taking less space on the side of the networking card or switch.

Small Form Pluggable Plus (SFP+)

-   Same size as the SFP, but used on 10-Gbps interfaces

UTP Cabling Pinouts for 10BASE-T and 100BASE-T

10BASE-T and 100BASE-T

-   2 wire pairs,
-   one pair for each direction

Crossover

PC/ Router/ AP

-   transmits on pins 1/2
-   RECEIVES ON PINS 3/6

Switch/ Hub

-   Transmits on pins 3/6
-   receives on pins 1/2

Ethernet NIC transmitters send on pins 1 and 2

NIC receivers receive on pins 3 and 6

Straight through cable

-   Pins 1/2 > 1/2
-   Pins 3/6 > 3/6

autoix (cisco)

-   Notices when a wrong cable is used
-   automatically changes it's logic to make the link work.

1000BASE-T

-   4 wire pairs
-   Both ends transmit and receive simultaneously on each wire pair.
-   pairs 1/2, 3/6, 4/5, 7/8
-   Crossover cable crosses the pairs at pins 1/2 and 3/6, it also crosses pairs 4/5 with 7/8

Fiber Cabling Transmission Concepts

Physical cable

-   Core >
-   Cladding >
-   Buffer >
-   Strengthener >
-   Outer Jacket

Optical Transmitter

-   Shines light into the core

Multimode fiber

-   multiple angles (modes) of light waves
-   Less expensive
-   10 gigabit over ethernet allows for distance up to 400m

Single mode fiber

-   Smaller diameter (around 1/5 of multimode) core
-   laser-based transmitter
-   single angle

More expensive SFP/ SFP+ hardware

Distances up to tens of kilometers

Transmit port on one end of the fiber connects to the receive port on the other end (Tx and Rx)

10Gbps Fiber Standards

-   10GBASE-S/ MM/ 400m
-   10GBASE-LX4/ MM/ 300m
-   10GBASE-LR/ SM/ 10km
-   10GBASE-E/ SM/ 30km

UTP, MM, and SM comparisons

UTP

-   low cable cost
-   low switch port cost
-   100m Max Distance
-   Some susceptibility to interference
-   Some risk of copying from cable emissions

Multimode

-   Medium cable cost
-   Medium switch port cost
-   500m Max Distance
-   No susceptibility to interference
-   No risk of copying from cable emissions

Single-Mode

-   Medium cable cost
-   High switch port cost
-   40Km Max Distance
-   No susceptibility to interference
-   No risk of copying from cable emissions

Sending Data In Ethernet Networks

Ethernet Header (preamble/sfd/Destination/Source/Type/Dataandpad/FCS)

Header

-   Preamble

-   7 bites
-   Synchronization

-   Start Frame Delimiter (SFD)

-   1 Byte
-   Signifies next byte begins the Destination MAC Address Field

-   Destination MAC Address

-   6 Bytes

-   Source MAC Address

-   6 Bytes

-   Type

-   2 bytes
-   Type of protocol listed in the frame (IPv4 or IPv6)

-   Data and Pad

-   46-1500Bytes
-   padding can be added to meet the minimum length requirement

Trailer

-   Frame Check Sequence (FCS)
-   Used to determine if the frame experienced transmission errors

Maximum Transmission Unit (MTU)

-   Maximum layer 3 packet that can be sent

Ethernet Addressing

-   6 bytes long (48 bits)
-   12 digit hexadecimal
-   Cisco switch may list a mac address with periods: 0000.0C12.3456
-   Unicast address

-   an address for a single NIC or port

OUI (Organizationally unique indentifier)

-   Universally unique manufacturer code
-   24 Bits
-   6 Hex Digits

Vendor Assigned

-   24 Bits
-   6 Hex Digits

Group addresses

Broadcast Address

-   Delivered to all devices on the Ethernet LAN.
-   FFFF.FFFF.FFFF

Multicast Address

-   Copied and forwarded to a subset of devices on the LAN

Identifying Network Layer Protocols with the Ethernet Type Field

The type field identifies which type of layer 3 packet exists within the ethernet frame (IPv6 or IPv4)

Ethertype is the term used for the type field in an ethernet frame

Error Detection with FCS

-   Error detection does not mean error recovery
-   Ethernet decide whether the frame should be discarded and does not attempt to recover the lost frame.

Sending Ethernet frames with switches and hubs

-   Switches allow the use of Full Duplex Logic
-   Hubs use half-duplex logic

Sending in Modern Ethernet LANs Using Full Duplex

Half duplex

-   must wait to send if it is currently receiving a frame
-   cannot send and receive at the same time

Full Duplex

-   Does not have to wait before sending,
-   send and receive at the same time.

Hubs

-   Uses physical link standards instead of data link standards and are considered layer 1 devices

Carrier Sense multiple access with Collision Detection (CSMA/CD)

1.  Listen until line is not busy
2.  Send frame
3.  Listen for a collision while sending if a collision occurs:

1.  Send jamming signal telling all nodes a collision has occurred
2.  each node waits a random time then tries to send again
3.  Back to step one

-   All links between PCs and switches use full duplex
-   A link connected to a hub should be half duplex
-   Ethernet Shared media

-   refers to hubs that use CSMA/CD and share bandwidth

-   Ethernet point-to-point

-   network built with switches where links work independently of others
-   A frame can be sent on every Point-to-point link in an ethernet at the same time.
