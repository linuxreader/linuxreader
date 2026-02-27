# 6 Switchport security config

Tuesday, September 28, 2021 2:44 PM

- **switchport port** - predefine any allowed source MAC addresses for this interface. **- security mac-address** _mac-address_

```
(Optional)
```

- tell the switch to “sticky learn” dynamically learned MAC addresses.

```
(Optional)- switchport port-security mac-address sticky
```

```
switchport port - enables port security, with all defaults - security
```

```
defines a specific source MAC address. With the default maximum source address setting of
1
```

```
switchport port - - security mac-address 0200.1111.1111
```

```
default violation action disable the interface.
```

- Port security does not save the configuration of the sticky addresses - use the copy running-config startup-config command if desired.

```
make sure to configure the maximum MAC address to at least two (one for the phone,
or for a PC connected to the phone)
```

- voice ports-
    - the port security configuration should be placed on the portthan the individual physical interfaces in the channel. -channel interface, rather
- EtherChannels

```
voice ports and EtherChannels
```

Verifying Port Security

- provides the most insight to how port security operates
- lists the configuration settings for port security on an interface
    - - Port Security: EnabledPort Status: Secure-shutdown
    - - Violation mode: shutdownMaximum MAC Addresses : 1
    - Last source Address:VLAN: 0013:197b:5004 1
- includes information about any security violations

```
Show port-security interface
```

- - Port Security: EnabledPort Status: Secure-shutdown
- - Violation Mode: ShutdownMaximum MAC Addresses: 1
- Sticky MAC Addresses: 1
- Last Source Address: Vlan 0013:197b:5004 1

```
Show port-security interface fastEthernet 0/2
```

```
secure- the interface has been disabled because of port security-shutdown state
```

Port Security MAC Addresses

```
No longer listed as dynamic entries- Do not show up in the results from show mac address-table dynamic EXEC command.
```

##### -

```
you need to use one of these options to see the MAC table entries associated with ports
using port security:
```

##### -

- Port security ports

```
show mac address - Lists MAC addresses associated with ports that use port security - table secure :
```

```
Lists MAC addresses associated with ports that use port security, as well as any
other statically defined MAC addresses
```

```
show mac address - - table static:
```

```
show mac address-table secure interface F0/2
```

```
(All three options cause the switch to discard the offending frame)
```

- Discard frame

```
Protect
```

- - Discard frameSend log and snmp messages
- increment violation counter

```
Restrict
```

- Discard frame
- - Send log and snmp messagesincrement violation counter
- Puts interface into err-disabled state, discarding all traffic

```
Shutdown
```

Port Security Violation Modes

Shutdown Mode

- interface must be shut down with the shutdown command and then enabled with the no shutdown command.
- recover from an err-disabled state
- automatically recover from the err-disabled state:
- automatic recovery for interfaces in an err-disabled state caused by port security

```
errdisable recovery cause psecure-violation
```

```
errdisable recovery interval - set the time to wait before recovering the interface seconds
```

```
lists the current port-security status (secure-shutdown) as well as the configured mode
(shutdown)
```

##### -

```
last line of output lists the number of violations that caused the interface to fail to an err-
disabled state
```

##### -

```
show port-security interface
```

```
disabled statesecond-to-last line identifies the MAC address and VLAN of the device that caused the
violation.
```

##### -

- notes the number of times the interface has been moved to the errshutdown) state. -disabled (secure-
- while errafter shutdown/no shutdown, another violation that causes the interface to fail to an err-disabled, many frames can arrive, but the counter remains at 1. -  
    disabled state will cause the counter to increment to 2.

##### -

```
violations counter
```

Protect and Restrict Modes  
still discard offending traffic, but the interface remains in a connected (up/up) state and in a port  
security state of secure-up.

##### -

- port continues to forward good traffic but discards offending traffic.
    - - discard the frame.does not change the port to an err-disabled state
    - does not generate messages
    - does not even increment the violations counter

```
protect
```