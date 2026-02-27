1.0 Network Fundamentals1.13 Describe switching concepts  
1.13.a MAC learning and aging1.13.b Frame switching  
1.13.c Frame flooding1.13.d MAC address table  
2.0 Network Access2.1 Configure and verify VLANs (normal range) spanning multiple switches  
2.1.a Access ports (data and voice)2.1.b Default VLAN2.1.c Connectivity  
2.2 Configure and verify interswitch connectivity2.2.a Trunk ports  
2.2.b 802.1Q2.2.c Native VLAN  
Virtual LAN Concepts  
reasons for choosing to create smaller broadcast domains (VLANs):

- -reduce CPU overhead on each devicereduce security risks
- different security policies per VLAN
- more flexible designs that -group users by department, or by groups that work together, instead of by physical location
- solve problems more quickly-failure domain for many problems is the same set of devices as those in the same broadcast domain
- reduce the workload for the Spanning Tree Protocol (STP) -by limiting a VLAN to a single access switch  
    802.1q and ISL  
    802.1Q -inserts a 4-byte 802.1Q VLAN header into the Ethernet header  
    12 - bit VLAN ID field inside the 802.1Q header --supports a theoretical maximum of 212 (4096) VLANs, but in practice it supports a maximum of 4094. Both 802.1Q and ISL use 12 bits to tag the VLAN ID, withtwo reserved values [0 and 4095].
- 802.1q header includes Type, priority, Flag, Vlan ID  
    normal--1 to 1005. all switches can use-range  
    Only some switches can use 1006 to 4094  
    depends on the configuration of the VLAN Trunking Protocol (VTP)

```
Extended range
```

- Cisco switches break the range of VLAN IDs (1–4094) into the normal range and the extended range.

```
231852+
```

- 802.1Q simply does not add an 802.1Q header to frames in the native VLAN  
    **#show vlan brief**  
    VLAN Trunking Protocol (VTP)  
    **vtp mode transparent vtp mode off**
- -The server switches can configure VLANs in the standard range only (1The client switches cannot configure VLANs. –1005).
- Both servers and clients may be learning new VLANs from other switches and seeing their VLANs deleted by other switches because of VTP.

```
If your switch usesVTP server or client mode
```

```
show running - does not list any vlan commands - confi g
```

```
show vtp status
```

- If possible to learn more about VTP for other purposes.in the lab, switch to disable VTP and ignore VTP for your switch configuration practice until you decide
    - negotiate ISL or 802.1q
    - If both switches support both protocols, they use ISL; -otherwise, they use the protocol that both support.

```
Dynamic Trunking Protocol (DTP).
switchport trunk encapsulation {dot1q | isl | negotiate} - configure the type or allow DTP to negotiate the type.
Access-always access
trunk-always trunk
dynamic desirable--initiates negotiation messages and responds to negotiation messagesAccess if other side is access, otherwise trunk
dynamic auto-passively waits to receive trunk negotiation messages
```

VLAN Trunking Configuration

```
Quick Commands
#show vtp status
```

```
VTP
Trunking
#switchport trunk encapsulation dot1q/isl/negotiate#switchport mode access/trunk/dynamic desirable/dynamic auto
#switchport trunk allowed vlan#show interfaces trunk output
#show interfaces trunk#show interfaces switchport
Voice#switchport trunk native vlan 2
#show int f0/4 trunk#switchport voice vlan 13
VLAN
#show vlan brief#show vlan
#show spanning-tree vlan 2
```

8. VLANSFriday, July 2, 2021 2:50 PM

- -passively waits to receive trunk negotiation messagesdefault setting
- -access if both ends use thistrunk if other end is trunk or Dynamic desirable
- On a switch that supports both ISL and 802.1Q, this value would by default list “negotiate,” to mean that the type of encapsulation is negotiated.
- Cisco recommends disabling trunk negotiation on most ports for better security  
    **(config** Disable DTP **- if) switchport nonegotiate**  
    Data and Voice VLAN Concepts  
    **switchport voice** - -can configure on the same access port that has a normal vlan assignedCDP must be enabled* _vlan 11_
- Voice Data is tagged with 802.1Q header
- -see the voice vlanadministrative and operational mode
- access mode vlan

```
show interfaces FastEthernet 0/4 switchport
```

**show interfaces trunkshow interfaces** _f0/4_ **trunk**

- vlans allowed on trunk-- (^1) minus vlans removed by the - (^4094) **switchport trunk allowed** command
    
- vlans allowed and active in management domain--the first list minus vlans that are not configuredminus vlans that are **shutdown**
    
- vlans in spanning tree forwarding state and not (VTP) pruned--minus vlans that are in a STP blocking stateminus vlans that are VTP pruned
    
- **Show interfaces trunk** will not show the voice VLAN as a trunk, it will only show it if you specify the interface.  
    Confirm that all VLANs are both defined and active. **show vlanShow vlan brief**  
    Check the allowed VLAN lists on both ends of each trunk  
    **show interfaces** - lists information about currently operational trunks _interface-id_ **trunk**  
    **#switchport trunk allowed vlan**  
    **Show vlan** - (does the vlan exist and is it active?
    
- Has the vlan been vtp pruned?
    
- Is the vlan in an STP forwarding state? **#show spanning-tree** _vlan 2_  
    Check for incorrect trunk configuration settings that result in one switch operating as a trunk, with the neighboring switch not operating as a trunk.  
    **#show interfaces trunk**  
    **#show interfaces switchport.** The trunk is in an STP forwarding state in that VLAN (as also seen in the -check administrative and operational modes **show spanning-tree vlan** _vlan-id_
    
- command). **#switchport trunk allowed vlan**
    
- DTP on one switch but not the other  
    Check the native VLAN settings on both ends  
    - Native vlan must match on both switches. **#switchport trunk native vlan** _2_  
    - vlan hoppinga frame being sent in one vlan but then being believed to be in a different vlan
    

Troubleshooting VLANS and VLAN trunks
