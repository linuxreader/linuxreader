5.0 Security Fundamentals

5.7 Configure Layer 2 security features (DHCP snooping, dynamic ARP inspection, and port security)

DHCP Snooping

- notices DHCP messages that fall outside the normal use of DHCP
- watches the DHCP messages that flow through a LAN switch
- builds a table that lists the details of legitimate DHCP flows
- other switch features can know what legitimate DHCP leases exist for devices connected to the switch.

Dynamic ARP Inspection (DAI)

- helps prevent packets being redirected to an attacking host.
- Some ARP attacks try to convince hosts to send packets to the attacker’s device instead of the true destination.
- The switch watches ARP messages as they flow through the switch.

- checks incoming ARP messages,
- checks those against normal ARP operation as well as
- checks the details against other data sources, including the DHCP Snooping binding table.
- When the ARP message does not match the known information about the legitimate addresses in the network, the switch filters the ARP message.

DHCP Snooping Concepts

- analyzes incoming messages on the specified subset of ports in a VLAN
- never filters non-DHCP messages
- allow the incoming DHCP message or discard the message.
- operates on LAN switches and is commonly used on Layer 2 LAN switches and enabled on Layer 2 ports.
- Client facing devices are untrusted
- DHCP facing devices (server, router, switch) are trusted
- works first on all ports in a VLAN, but with each port being trusted or untrusted by DHCP Snooping

- the rules differentiate between messages normally sent by servers (like DHCPOFFER and DHCPACK) versus those normally sent by DHCP clients:

- DHCP messages received on an untrusted port, for messages normally sent by a server, will always be discarded.
- DHCP messages received on an untrusted port, as normally sent by a DHCP client, may be filtered if they appear to be part of an attack.
- DHCP messages received on a trusted port will be forwarded; trusted ports do not filter (discard) any DHCP messages.

A Sample Attack: A Spurious DHCP Server

- Spurious DHCP server (attacker) responds to dhcp message by a client pc
- attacker replies with false dhcp information by naming itself as the default gateway
- Pc1 send all messages to another network to attacker, becoming a man-in-the-middle attack (pc2 could forward the messages to the actual default gateway
- the legitimate DHCP also returns a DHCPOFFER message to host PC1, but most hosts use the first received DHCPOFFER, and the attacker will likely be first in this scenario.

DHCP Snooping Logic

- stops attacks by making ports untrusted
- DHCP clients also use the DHCP RELEASE and DHCP DECLINE messages.

- DHCP RELEASE

- client can tell the DHCP server it no longer needs the address, releasing it back to the DHCP server

- DHCP DECLINE

- turn down the use of an IP address during the normal DORA flow on messages.

DHCP Snooping rules:

- Examine all incoming DHCP messages.
- If normally sent by servers, discard the message.
- If normally sent by clients, filter as follows:

- DISCOVER and REQUEST

- check for MAC address consistency between the Ethernet frame and the DHCP message.

RELEASE or DECLINE

- check the incoming interface plus IP address versus the DHCP Snooping binding table.

- For messages not filtered that result in a DHCP lease, build a new entry to the DHCP Snooping binding table.

DHCP snooping checking MAC Address

- chaddr (client hardware address)

- Source MAC address included in DISCOVER message
- frame encapsulating the dhcp message also includes the source mac address
- DHCP Snooping checks to make sure those values match.

- if not the packet is dropped

- Used on untrusted ports

- stops attacker from leasing all available addresses so no other client can get a lease

Filtering Messages that Release IP Addresses

- DHCP Snooping binding table

- creates entry for all legitimate DHCP entries
- DHCP Snooping, and other features like Dynamic ARP Inspection, can use the table to make decisions.
- DHCP Snooping Binding Table:

- MAC
- IP
- VLAN
- INTERFACE

- checks client-sent messages like RELEASE and DECLINE that would cause the DHCP server to be allowed to release an address

- attacker could RELEASE an address that is not his and attempt steal it for himself

- attacker pretends his source IP is the legitimate IP trying to RELEASE
- the source interface in the DHCP Snooping Binding Table does not match so dhcp snooping bloacks the request

DHCP Snooping Configuration

Required:

- enable DHCP Snooping

- ip dhcp snooping

- list the VLANs on which to use DHCP Snooping

- ip dhcp snooping vlan vlan-id

Often used:

- configure trusted ports

- default is untrusted.
- (config-if) # ip dhcp snooping trust

DHCP Snooping verification

show ip dhcp snooping

- enabled or disabled
- operational in which vlans
- trusted interfaces
- Insertion of option 82 is disabled

- (enabled by default)
- no ip dhcp information option command is configured

- (because this switch is not a dhcp relay agent)

- DHCP relay agents add option 82 DHCP header fields (RFC 3046)
- If enabled on a non relay agent then dhcp stops working for end users
- The switch sets fields in the DHCP messages as if it were a DHCP relay agent
- changes to those messages cause most DHCP servers (and most DHCP relay agents) to ignore the received DHCP messages.

- Rate limit on each interface

Show ip dhcp snooping binding

Shows actual dhcp snooping binding table

DHCP Rate Limiting

- attacker generates large volumes of DHCP messages to overload the DHCP Snooping feature and the switch CPU itself

- optional feature that tracks the number of incoming DHCP messages.
- If the number of incoming DHCP messages exceeds that limit over a one-second period:

- port changes to an err-disabled state.

-can be on both on trusted and untrusted interfaces.

- Default of no rate limit

how to enable DHCP Snooping rate limits and err-disabled recovery.

(config-int) # ip dhcp snooping rate limit number

errdisable recovery cause dhcp-rate-limit

- recover from errdisable if caused by dhcp snooping

errdisable recovery interval number

- seconds to wait before recovering

(config-int) # no ip dhcp snooping rate limit

Dynamic ARP Inspection (DAI)

- examines incoming ARP messages on untrusted ports to filter those it believes to be part of an attack.
- compares incoming ARP messages with two sources of data:

- DHCP Snooping binding table
- configured ARP ACLs.

- If the incoming ARP message does not match the tables in the switch, the switch discards the ARP message.

DAI Concepts

gratuitous ARP (arp reply)—

triggers hosts to add incorrect ARP entries to their ARP tables.

Review of Normal IP ARP

Both hosts learn the other host’s MAC address with this two-message flow. Not only does PC1 learn R2’s MAC address based on the ARP reply (message 2), but router R2 learns PC1’s IP and MAC address because of the ARP request (message 1).

ARP Request

- Ethernet Frame

- source mac
- destination mac (broadcast)

- ARP

- Origin IP
- Target IP
- Origin MAC
- Target MAC ??

ARP Reply

- Ethernet Frame

- Source MAC
- Dest. MAC

- ARP

- Origin IP
- Target IP
- Origin MAC
- Target MAC

Gratuitous ARP as an Attack Vector

- when a host changes its MAC address it can send a gratuitous ARP message with these features:

- ARP reply.
- sent without having first received an ARP request.
- sent to an Ethernet destination broadcast address so that all hosts in the subnet receive the message.

- could be part of a man-in-the-middle attack

- attacker uses gratuitous arp to make replies go back to it instead of original host

Dynamic ARP Inspection Logic

- Host does not need arp if it doesn't have an IP yet
- Once DHCP assigns and IP then ARP is used
- for untrusted interfaces DAI confirms an ARP’s correctness based on the DHCP Snooping Binding table
- compares origin MAC and IP
- if not found on the table, discard the arp
- same trusted and untrusted rules as dhcp snooping

ARP ACL

- statically configured
- list correct pairs of IP and mac addresses
- Useful for ip addresses configured statically because they will not be in the dhcp snooping binding table

- DAI can check Ethernet header and arp source and destination messages mac addresses to make sure they match
- DAI can also check Messages with unexpected IP addresses in the two ARP IP address fields
- does its work in the switch CPU rather than in the switch ASIC

- DAI itself can be more susceptible to DoS attacks.
- attacker could generate large numbers of ARP messages, driving up CPU usage in the switch.
- DAI can avoid these problems through rate limiting the number of ARP messages on a port over time.

Configuring ARP Inspection on a Layer 2 Switch

decisions include the following:

- Choose whether to rely on DHCP Snooping, ARP ACLs, or both.
- If using DHCP Snooping, configure it and make the correct ports trusted for DHCP Snooping.
- Choose the VLAN(s) on which to enable DAI.
- Make DAI trusted (rather than the default setting of untrusted) on select ports in those VLANs, typically for the same ports you trusted for DHCP Snooping.

ip arp inspection vlan vlan-id

- enable arp DAI on that vlan

(config-if)# ip arp inspection trust

- dhcp snooping should be enabled or DAI will filter all packets

ip dhcp snooping

ip dhcp snooping vlan vlan-id

ip dhcp snooping trust

(configure arp ACLs)

show ip arp inspection

- gives both configuration settings along with status variables and counters

- forwarded, dropped, dhcp drops, acl drop counts
- Is logging enabled?

show ip arp inspection statistics

- counts per vlan

- forwarded, dropped, dhcp drops, acl drops

- permits

ARP Rate Limiting

- Default for all interfaces

- (trusted and untrusted)

burst interval (a number of seconds)

- x ARP messages over y seconds”
- (DHCP Snooping does not define a burst setting)
- optional configuration

ip arp inspection limit rate 8 burst interval 4

- 8 messages in a burst of 4 seconds

- default of 15 messages over a one second burst
- show ip arp inspection interfaces

- rate, burst interval, trust state, per interface

ip arp inspection limit rate number

ip arp inspection limit rate none

- disable rate limiting

errdisable recovery cause arp-inspection

errdisable recovery interval 30

Configuring Optional DAI Message Checks

ip arp inspection validate {dst-mac, ip, src-mac}

- one, two, or all three of the options
- command with overrides previous versions of the same command
- show ip arp inspection to validate

drop ARP packets when the IP addresses in the packets are invalid or when the MAC addresses in the body of the ARP packets do not match the addresses specified in the Ethernet header.

- For src-mac, check the source MAC address in the Ethernet header against the sender MAC address in the ARP body. This check is performed on both ARP requests and responses. When enabled, packets with different MAC addresses are classified as invalid and are dropped.
- For dst-mac, check the destination MAC address in the Ethernet header against the target MAC address in ARP body. This check is performed for ARP responses. When enabled, packets with different MAC addresses are classified as invalid and are dropped.
- For ip, check the ARP body for invalid and unexpected IP addresses. Addresses include 0.0.0.0, 255.255.255.255, and all IP multicast addresses. Sender IP addresses are checked in all ARP requests and responses, and target IP addresses are checked only in ARP responses
