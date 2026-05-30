This lab covers some basic LAN setup. Real world networks are a lot bigger than this but once you can configure these things. Troubleshooting and setting up bigger networks will be less challenging.

Here are the settings to configure:

-   VLAN setup using VTP
-   Trunking using DTP
-   Router on a stick
-   Spanning Tree
-   Etherchannel

Your goal with this lab is to configure all of the settings above and verify using show commands. And test your working network with pings between all of the PCs. Finally, you will download a broken version of the lab and use your troubleshooting skills to fix it.

The PCs have all been configured with IP addresses and default gateways for their respective VLANs.

Packet Tracer files for this lab



Set up trunk ports on all switches using DTP

You'll need to set up trunk ports using dtp between all of the switches. Switch1s port connecting to router1 should also be a trunk link. Then verify the trunk connections.

Each interface you see here should be a trunk port.

Set all used ports on switch1 to "dynamic desirable". Later we will disable DTP and manually set up the trunk links.

switch1 (config) # int ra fa0/1-6  
switch1 (config-if-range) # switchport mode dynamic desirable

Verify trunk links on switches 3-5 leading to switch1

You should see trunks connecting to switch1 since dtp is set to "dynamic auto" by default on the other switches.

switch3 # show interfaces trunk

Remove the dynamic desirable command

All of that work you did to get the trunks up using DTP? Let's start all over and configure them manually instead.

You know.. just for fun.

switch1 (config-if) # no switchport dynamic desirable

Manually Configure Trunks

DTP is usually disabled for security purposes. So we will configure the trunks manually.

Designate trunk ports with 802.1q encapsulation

Depending on the switch, you may or may not have to do this.

switch3 (config-int) # switchport trunk encapsulation dot1q

Set switch port mode to trunk

switch1 # interface range fa0/1-6  
switch1 (config-if) # switchport mode trunk

Disable dtp

switch1 (config-if) # switchport nonegotiate

Show switchport info on a single interface

switch1 # show interfaces fa0/4 switchport

Use VTP to Populate VLANs

VTP is not on the current CCNA exam objectives. However, I've seen this in the real world so it's good to know how it works and how to configure it.

I've heard that you may see questions on the exam about VTP anyway.

Set up VLANs on switch1

This lab uses VLAN 10, 20, and 30 in networks that match their respective VLAN number. With VLAN 100 being an unused VLAN for security.

-   VLAN 10: 10.0.0.0 /24
-   VLAN 20: 20.0.0.0 /24
-   VLAN 30: 30.0.0.0 /24
-   VLAN 100: NA

switch 1 (config) # vlan 10  
switch 1 (config) # vlan 20  
switch 1 (config) # vlan 30  
switch 1 (config) # vlan 100

Set switch1 to VTP mode "server"

switch1 (config) # vtp mode server

Set the VTP domain name

switch1 (config) # vtp domain cisco

Set the other switches to VTP "client" mode

switch2 (config) # vtp mode client

Confirm VTP settings

switch2 # show vtp status

Confirm VLANs are set up on all switches

switch2 # show vlan brief

Disable VTP on all switches

switch1 (config) # vtp mode transparent

Final VLAN Pieces

Now that your trunks and VLANs are configured. You can assign some ports to the VLANs. Then block the unused VLAN 100 on all of your trunk links.

Set up access ports connected to PCs

Set the switchport type to "access". Then assign that access port to it's respective VLAN.

switch3 (config-int) # switchport mode access  
switch3 (config-int) # switchport access vlan 10

Remove VLAN 100 from all trunks

switch1 (config-int) # switchport trunk allowed vlan remove 100

Create a Router on a Stick

Make sure interface gi0/0 does not have an ip address

router1 # show ip interface brief

Create subinterfaces on router and assign ip addresses to them

-   gi0/1.10
-   gi0/1.20
-   gi0/1.30

router1 # interface gi0/1.10  
router1 (config-if) # ip address 10.0.0.1 255.255.255.0  
router1 (config-if) # encapsulation dot1q 10

Set port on opposing switch to trunk mode

You'll need to make sure the switch1 port connecting to router1 is a trunk link. This is so that all VLANs can travel to router1 for routing.

The PCs are already configured to use router1 as their default gateway. Assuming you configure the subinterfaces with the .1 ip address for their respective VLANS

Spanning Tree Protocol

This is a very simplified Spanning Tree lab. Leave a comment if you'd like a more detailed one in the future.

Optimize spanning tree for best path

You'll want to make switch1 the root bridge for Spanning Tree. To do this, make switch1 have the lowest priority. Or manually set it as the root bridge with the "primary" command.

This setting should be configured for each VLAN.

Set the priority to change the root bridge

switch1 (config) # spanning-tree vlan 10 priority 24,576

Change the root bridge with the primary command

This will set the switches priority from 32,868 to 24,586. (24,576 + 10 for VLAN 10) The lowest priority will win the root election.

switch1 (config) # spanning-tree vlan 10 root primary

View spanning tree information

switch1 # show spanning-tree

View STP information on a specific VLAN

switch1 # show spanning-tee vlan 10

Show summary of Spanning Tree

switch1 # show spanning-tree summary

Enable PortFast on access ports for switches 3-5

You can set each interface manually. Or you can enable portfast on all non trunking ports with the "default" option.

switch3 (config-if) # spanning-tree portfast

Enable BPDU Guard on all interfaces with PortFast

switch3 (config-if) # spanning-tree bpduguard enable

Etherchannel (Layer 2)

ou'll need to make sure that all of the Etherchannel ports are either in access mode or trunk mode. It cannot be a mix of both. Also, I had an error (twice) in Packet Tracer where the P01 interface was stuck on STP listening mode. It would not resolve until I reset Packet Tracer.

Keep in mind when troubleshooting errors that Packet Tracer can be buggy. Your issue may be with PT software and not the IOS configuration.

Set up an Etherchannel using PAGP

switch1 (config-if) # channel-group 1 mode desirable

switch2 (config-if) # channel-group 1 mode desirable/ auto

Set up an Etherchannel using LACP

switch1 (conf-if) # channel-group 1 mode active

switch2 (config-if) # channel-group 1 mode active/ passive

View Etherchannel info

switch1 # show etherchannel 1 port-channel

View Etherchannel summary

switch1 # show etherchannel summary

Modify Etherchannel load distribution

switch1 # port-channel load-balance dst-ip

Show Etherchannel load balance information

switch1 # show etherchannel load-balance

Port Security

Implement port security and add an unauthorized device.

View port security details

switch3 # show port-security interface fa 0/3

Enable port security

Switch ports must be not be set as dynamic. Set them manually as access ports instead.

switch3 (config-if-range) # switchport mode access  
switch3 (config-if-range) # switchport port-security

Using the sticky command

switch3 (config-if-range) # switchport port-security mac-address sticky

Set the maximum allowed mac addresses on a port to 2

switch3 (config-if-range) # switchport port-security maximum 2

Set port violation to "protect"

switch3 (config-if-range) # switchport port-security violation protect

Set port violation to "restrict"

switch3 (config-if-range) # switchport port-security violation restrict

Set port violation to "shutdown"

switch3 (config-if-range) # switchport port-security violation shutdown

Set port security to a single mac-address

switch3 (config-if-range) # switchport port-security mac-address aa.bb.cc.dd.ee.ff

Shutdown unused ports

switch3 (config-if-range) # Shutdown

Place unused ports into an unused vlan

switch3 (config-if-range) # switchport access vlan 100

Bring an interface back up from an "err-disabled" state

This is optional. To get a port to an err-disabled state, you'll have to plug in an unauthorized PC if using a static mac-address. Or if you are using the maximum allowed option, plug too many PCs into a single port. Either via hub or by unplugging the PCs and plugging new ones in a few times.

switch3 (config-if) # shutdown  
switch3 (config-if) # no shutdown

Show port security Info
