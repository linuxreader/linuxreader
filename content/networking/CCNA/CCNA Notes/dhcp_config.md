1.0 Network Fundamentals

1.10 Identify IP parameters for Client OS (Windows, Mac OS, Linux)

4.0 IP Services

4.3 Explain the role of DHCP and DNS within the network

4.6 Configure and verify DHCP client and relay

DHCP Concepts

Four messages between the client and server (DORA):

-   Discover:
    
-   Sent by the DHCP client to find a willing DHCP server
    
-   Offer:
    
-   Sent by a DHCP server to offer to lease
    

Request:

-   Sent by the DHCP client to ask the server to lease the IPv4 address listed in the Offer message

Acknowledgment:

-   Sent by the DHCP server to assign the address and to list the mask, default router, and DNS server IP addresses

IPv4 addresses that allow a host that has no IP address to still be able to send and receive messages on the local subnet:

0.0.0.0:

-   An address reserved for use as a source IPv4 address for hosts that do not yet have an IP address.

255.255.255.255:

-   The local broadcast IP address. Packets sent to this destination address are broadcast on the local data link, but routers do not forward them.
    
-   a client, sends a Discover message, with source IP address of 0.0.0.0
    
-   Host A sends the packet to destination 255.255.255.255 router R1 will not forward this packet.
    
-   assuming the DHCP client chooses to use a DHCP option called the broadcast flag;
    
-   all examples in this book assume the broadcast flag is used.
    
-   all hosts in the subnet receive the Offer message
    
-   the original Discover message lists a number called the client ID, which
    
-   includes the host’s MAC address, that identifies the original host (host A in this case).
    
-   As a result, host A knows that the Offer message is meant for host A.
    

Supporting DHCP for Remote Subnets with DHCP Relay

ip helper-address server-ip

-   Watch for incoming DHCP messages, with destination IP address 255.255.255.255.
    
-   Change that packet’s source IP address to the router’s incoming interface IP address.
    
-   Change that packet’s destination IP address to the address of the DHCP server (as configured in the ip helper-address command).
    
-   Route the packet to the DHCP server.
    
-   for the return packet from the DHCP server
    
-   the server simply reverses the source and destination IP address of the packet received from the router (relay agent)
    
-   the Discover message lists source IP address 172.16.1.1, so the server sends the Offer message back to destination IP address 172.16.1.1.
    
-   the DHCP relay agent (router R1) needs to change the destination IP address, so that the real DHCP client (host A), which does not have an IP address yet, can receive and process the packet.
    
-   Sends as broadcast back to the LAN
    

Information Stored at the DHCP Server

types of settings the DHCP server needs to know to support DHCP clients:

-   Subnet ID and mask:
    
-   Reserved (excluded) addresses:
    
-   which addresses in the subnet to not lease.
    
-   Default router(s)
    
-   DNS IP address(es)
    

other parameters

Maximum lease time

-   time limit for leasing an IP address

DHCP uses three allocation modes

Dynamic allocation

-   the DHCP mechanisms and configuration described throughout this chapter.

automatic allocation

- sets the DHCP lease time to infinite.

static allocation

-   preconfigures the specific IP address for a client based on the client’s MAC address.

Trivial File Transfer Protocol (TFTP) server address

Configuring DHCP Relay

What for?

-   DHCP clients exist in the subnet
-   DHCP servers do not exist in the subnet

(config-if)# ip helper-address address

Show ip interface g0/0

-   Helper Address is address or Helper address is not set

Configuring a Switch as DHCP Client

(config-if) ip address dhcp

- the switch does not attempt DHCP until the interface reaches an up/up state

show interfaces interface

-   verify ip address

show dhcp lease

-   see the (temporarily) leased IP address and other parameters
-   the switch does not store the DHCP-learned IP configuration in the running-config

show ip default-gateway

-   view address of the default gateway

Configuring a Router as DHCP Client

(config-if) # ip address dhcp

- IOS displays this route as a static route (destination 0.0.0.0/0)

To recognize this route as a DHCP-learned default route, look to the administrative distance value of 254.

-   administrative distance of 1 for static routes configured with the ip route configuration command
-   default of 254 for default routes added because of DHCP.

Host Settings for IPv4

To work correctly, an IPv4 host needs to know these values:

-   DNS server IP addresses
-   Default gateway (router) IP address
-   Device’s own IP address
-   Device’s own subnet mask

Host IP Settings on Windows

netstat -rn

-   lists the host’s IP routing table
-   the top of the table lists a route based on the default gateway (0.0.0.0 and 0.0.0.0)
-   top of the output also lists several other routes related to having a working interface, like a route to the subnet connected to the interface.

gateway of “on-link”

the PC thinks the destination is on the local subnet (link)

Host IP Settings on macOS

ifconfig

-   does not have an /all option
-   does not list the default gateway or DNS servers
-   Shows mac address and ip address

networksetup -getinfo Ethernet

-   Shows dhcp config, ip, subnet mask, and default gateway

networksetup -getdnsservers Ethernet

-   Shows dns server
    
-   adds a default route to its host routing table based on the default gateway
    
-   adds route to the local subnet calculated based on the IP address and mask learned with DHCP
    
-   uses the netstat -rn command to list those routes
    
-   output represents the default route using the word default rather than 0.0.0.0 and 0.0.0.0
    

Host IP Settings on Linux

some Linux distributions do not include net-tools.

You can add net-tools to most Linux distributions.

includes ifconfig and netstat -rn.

iproute library

-   preinstalled on Linux
-   includes a set of replacement commands and functions, many performed with the ip command and some parameters.
-   similar to the macOS version but also shows some interface counters.

ipaddress

-   shows basic addressing info
-   Shows subnet in prefix notation rather than in dotted decimal

ifconfig wlan0

-   shows l2 and l3 addresses
-   from net-tools

netstat -rn

-   from net-tools
    
-   lists a default route, but with a style that shows the destination as 0.0.0.0.
    
-   points to the default gateway as learned with DHCP:
    
-   lists a route to the local subnet
    

The bottom of the example shows the command meant to replace netstat -rn: ip route. Note that it also shows a default route that references the default router, along with a route for the local subnet.
