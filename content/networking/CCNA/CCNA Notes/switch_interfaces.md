1.0 Network Fundamentals

1.1 Explain the role and function of network components

1.1.b L2 and L3 switches

1.4 Describe switching concepts

Configuring Speed, Duplex, and Description

-   autonegotiate

-   What speed to use
-   enabled by default

duplex {auto | full | half} and speed {auto | 10 | 100 | 1000}

-   configure the speed and duplex settings

(config-int) # description text

-   add a text description to the interface

show interfaces status

-   lists port #, Name, status, vlan, duplex, speed, and type

a-full and a-100

a- means that the listed speed and duplex values were autonegotiated.

Autonegotiation

IEEE autonegotiation (IEEE standard 802.3u)

-   each node states what it can do, and
-   then each node picks the best options that both nodes support:

-   the fastest speed and the best duplex setting, with full duplex being better than half duplex.

-   disable autonegotiation

-   Configure both the speed and duplex on a  switch interface

-   when a node tries to use autonegotiation but hears nothing from the device.

Speed: Use your slowest supported speed (often 10 Mbps).

Duplex: If your speed = 10 or 100, use half duplex; otherwise, use full duplex.

Cisco switches can actually sense the speed used by other nodes, even without IEEE autonegotiation.

-   Cisco switches use this slightly different logic to choose the speed when autonegotiation fails:

-   Speed: Sense the speed (without using autonegotiation), but if that fails, use the IEEE default (slowest supported speed, often 10 Mbps).

-   Duplex: Use the IEEE defaults: If speed = 10 or 100, use half duplex; otherwise, use full duplex.

-   Ethernet interfaces using speeds faster than 1 Gbps always use full duplex.
-   hubs do not react to autonegotiation messages

 show interfaces and show interfaces description

Shutdown command is configured

-   Line status = administratively down
-   Protocol status = down
-   Interface status = disabled

Cable, speed mismatch, neighbor device is off, shutdown, or err-disabled

-   Line status = down
-   Protocol status = down
-   Interface status = notconnect

Not expected on LAN switch physical interfaces

-   Line status = up
-   Protocol status = down
-   Interface status = notconnect

Port security has disabled the interface

-   Line status = down
-   Protocol status = down (err-disabled)
-   Interface status = err-disabled

the interface is working

-   Line status = up
-   Protocol status = up
-   Interface status = connected

show interfaces fa0/13 (without the status option)

-   lists the speed and duplex for interface Fast Ethernet 0/13
-   with nothing implying that the values were learned through autonegotiation.

speed manually set 10 Mbps on one switch and 100 Mbps on the other

-   both switches would list the port in a down/down or notconnect state

if the duplex settings do not match

-   the switch interface will still be in a connected (up/up) or connected state.

How to identify duplex mismatch problems,

-   check the duplex setting on each end of the link to see if the values mismatch.
-   watch for incrementing collision and late collision counters

Common Layer 1 problems

-   receiving device might receive a frame whose bits have changed values.
-   These frames do not pass the error detection logic as implemented in the FCS field in the Ethernet trailer,
-   The receiving device discards the frame and counts it as some kind of input error.
-   Cisco switches list this error as a CRC error

Runts:

-   Frames that did not meet the minimum frame size requirement

-   (64 bytes, including the 18-byte destination MAC, source MAC, type, and FCS).

-   Can be caused by collisions.

Giants:

-   Frames that exceed the maximum frame size requirement

-   (1518 bytes, including the 18-byte destination MAC, source MAC, type, and FCS)

Input Errors:

-   A total of many counters, including runts, giants, no buffer, CRC, frame, overrun, and ignored counts.

CRC:

-   Received frames that did not pass the FCS math
-   can be caused by collisions

Frame:

-   Received frames that have an illegal format

-   (like ending with a partial byte)

-   can be caused by collisions.

Packets Output:

-   Total number of packets (frames) forwarded out the interface.

Output Errors:

-   Total number of packets (frames) that the switch port tried to transmit, but for which some problem occurred.

Collisions:

-   Counter of all collisions that occur when the interface is transmitting a frame

Late Collisions:

-   The subset of all collisions that happen after the 64th byte

-   (In a properly working Ethernet LAN, collisions should occur within the first 64 bytes

-   Often point to a duplex mismatch

Collisions occur as a normal part of the half-duplex logic imposed by CSMA/CD

a switch interface with an increasing collisions counter might not even have a problem.

-   if the CRC errors grow, but the collisions counters do not, the problem might simply be interference on the cable.
