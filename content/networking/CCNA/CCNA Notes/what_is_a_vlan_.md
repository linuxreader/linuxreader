VLANs create smaller broadcast domains. Why create smaller broadcast domains?

- Device have to do less stuff (process less broadcasts)
- Better security
- Group different departments together
- Narrow down problems
- Less spanning tree if you reduce vlan to a single switch

802.1q tagging

Frames are tagged with an 802.1q header as they leave the switch. So the receiving switch knows what vlan the frame is part of.

802.1q header

- 4 bytes
- Includes 12 bit vlan id field that supports 4096 vlans. (0 and 5 are reserved so make that 4094)
- Also include type, priority, and flag fields

VLAN numbers and what they mean

Normal range vlans are vlans 1-1005. All switches are able to use these vlan numbers.

Extended range vlans 1006-4094. Only some switches are able to use these

Native vlan are untagged. This is important. The native vlans is defaulted to 1. But you can manually set it to another number say vlan4.

VLAN Trunking Protocol (VTP)

This is a protocol for spreading VLAN settings throughout a network. A switch can be set to Transparent, client, or server.

Client mode: Cannot configure vlans and can learn new and deleted vlans from other switches.

Server Mode: Can configure VLANS is the standard range. Can learn new and deleted vlans from other switches.

VTP Commands:

Set the vtp mode

#vtp mode (transparent, client, server, off)

Show vtp settings

#show vtp status

Dynamic trunking protocol

Will negotiate ISL or 802.1q trunking with other switches

Switches will default to ISL if they both know that protocol. Otherwise they will use DTP if they both support it. This is because the default setting reads “negotiate”. Or negotiate the protocol used.

You should disable negotiation of trunks for better security.

Switchport settings for DTP

Access: Statically set as access

Trunk: statically set as trunk

Dynamic desirable: Initiates and responds to negotiation messages.

Dynamic auto: responds to negotiation messages only. This is the default setting. If both ends use dynamic auto mode then both ends will be configured as access ports. If the other end is set as Dynamic desirable, then both ports will be set as trunk ports.

Trunking commands:

Setting the encapsulation mode

#switchport trunk encapsulation (dot1q | isl | negotiate)

Disable DTP

#switchport no negotiate

Voice vlans

Only time you can configure two vlans on an access port is if you add a voice vlan to it. This is because phones are often on the same port as user pcs and voice traffic is often prioritized differently than other traffic.

CDP protocol must be enabled for voic vlan to work. Also, voice traffic is tagged with an 802.1q header.

Setting the voice vlan

#switchport voice vlan 12

Showing vlan information and troubleshooting

You will want to see what vlans are configured on what ports. What trunks are allowed on what ports. Verify VTP and DTP settings, etc.

Show trunk info

#Show interfaces interface-id trunk

Choose what vlans are allowed on a trunk

\# Switchport trunk allowed vlan

From <https://d.docs.live.net/6e78e2c308bddf6e/Documents/VLANs%20create%20smaller%20broadcast%20domains.docx