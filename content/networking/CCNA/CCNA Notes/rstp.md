2.0 Network Access

2.4 Configure and verify (Layer 2/Layer 3) EtherChannel (LACP)

2.5 Describe the need for and basic operations of Rapid PVST+ Spanning Tree Protocol and identify basic operations

2.5.a Root port, root bridge (primary/secondary), and other port names

2.5.b Port states (forwarding/blocking)

2.5.c PortFast benefits

Most network engineers make the distribution layer switches be the root.

STP Modes and Standards

Three options to configure on the spanning-tree mode command, which tells the switch which type of STP to use

 the switches do not support STP or RSTP with the single tree (CST). They can use either the Cisco-proprietary and STP-based PVST+, Cisco-proprietary and RSTP-based RPVST+, or the IEEE standard MSTP.


MSTP allows the definition of as many instances (multiple spanning tree instances, or MSTIs) as chosen by the network designer but does not require one per VLAN.


STP and MSTP now exist as part of the 802.1Q standard, which defines VLANs and VLAN trunking.

The revised rules divide the original priority field into two separate fields, as shown in Figure 10-4: a 4-bit priority field and a 12-bit subfield called the system ID extension (which represents the VLAN ID


Cisco switches let you configure the BID, but only the priority part.

The switch fills in its universal (burned-in) MAC address as the system ID. It also plugs in the VLAN ID of a VLAN in the 12-bit system ID extension field; you cannot change that behavior either. The only part configurable by the network engineer is the 4-bit priority field.

the configuration command (spanning-tree vlan vlan-id priority x) requires a decimal number between 0 and 65,535. But not just any number in that range will suffice; it must be a multiple of 4096, as emphasized in the help text shown in Example 10-2.


#spanning-tree vlan x root primary (on the switch that should be primary)

#spanning-tree vlan x root secondary (on the switch that should be secondary)

These two commands cause the switch to make a choice of priority value but then store the chosen priority value in the spanning-tree vlan x priority value command. The command with root primary or root secondary does not appear in the configuration. When configuring root primary, the switch looks at the priority of the current root switch and chooses either (a) 24,576 or (b) 4096 less than the current root’s priority (if the current root’s priority is 24,576 or less) to the configuration instead. When configuring, root secondary always results in that switch using a priority of 28,672, with the assumption that the value will be less than other switches that use the default of 32,768, and higher than any switch configured as root primary.

How Switches Use the Priority and System ID Extension

Cisco Catalyst switches configure the priority value using a number that represents a 16-bit value; however, the system ID extension exists as the low-order 12 bits of that same number.

When the switch builds its BID to use for RSTP in a VLAN, it must combine the configured priority with the VLAN ID of that VLAN.

the configured priority results in a 16-bit priority that always ends with 12 binary 0s.


Root Switch: 24,576 (priority) + 9 (VLAN ID) = 24585

Local Switch: 32,768 (priority) + 9 (VLAN ID) = 32777


RSTP Methods to Support Multiple Spanning Trees

RSTP creates one tree—the Common Spanning Tree (CST)—while RPVST+ creates one tree for each and every VLAN.

RSTP sends one set of RSTP messages (BPDUs) in the network, no matter the number of VLANs, while RPVST+ sends one set of messages per VLAN.

RSTP and RPVST+ use different destination MAC addresses: RSTP with multicast address 0180.C200.0000 (an address defined in the IEEE standard), and RPVST+ with multicast address 0100.0CCC.CCCD (an address chosen by Cisco).

When transmitting messages on VLAN trunks, RSTP sends the messages in the native VLAN with no VLAN header/tag. RPVST+ sends each VLAN’s messages inside that VLAN—for instance, BPDUs about VLAN 9 have an 802.1Q header that lists VLAN 9.

RPVST+ adds an extra type-length value (TLV) to the BPDU that identifies the VLAN ID, while RSTP does not (because it does not need to, as RSTP ignores VLANs.)

Both view the 16-bit priority as having a 12-bit System ID Extension, with RSTP setting the value to 0000.0000.0000, meaning “no VLAN,” while RPVST+ uses the VLAN ID.

Other RSTP Configuration Options

Switch Priority: The global command spanning-tree vlan x priority y lets an engineer set the switch’s priority in that VLAN.

Primary and Secondary Root Switches: The global command spanning-tree vlan x root primary | secondary also lets you set the priority, but the switch decides on a value to make that switch likely to be the primary root switch (the root) or the secondary root switch (the switch that becomes root if the primary fails).

Port Costs: The interface subcommand spanning-tree [vlan x] cost y lets an engineer set the switch’s STP/RSTP cost on that port, either for all VLANs or for a specific VLAN on that port. Changing those costs then changes the root cost for some switches, which impacts the choice of root ports and designated ports.

