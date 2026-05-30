This chapter covers the following exam topics:

2.0 Network Access

2.6 Compare Cisco Wireless Architectures and AP modes

Autonomous AP Architecture

VLANs must be trunked from the distribution layer switch (where routing commonly takes place) to the access layer, where they are extended further over a trunk link to the AP.

Two wireless users that are associated to the same autonomous AP can reach each other through the AP without having to pass up into the wired network

An autonomous AP must also be configured with a management IP address so that you can remotely manage it.

configure SSIDs, VLANs, and many RF parameters like the channel and transmit power to be used.

management address is not normally part of any of the data VLANs, so a dedicated management VLAN (i.e., VLAN 10) must be added to the trunk links to reach the AP.

Each AP must be configured and maintained individually unless you leverage a management platform such as Cisco Prime Infrastructure or Cisco DNA Center.

That might sound straightforward until you have to add a new VLAN and configure every switch and AP in your network to carry and support it.

Cloud-based AP Architecture

Cisco Meraki is cloud-based and offers centralized management of wireless, switched, and security networks built from Meraki products. For example, through the cloud networking service, you can configure and manage APs, monitor wireless performance and activity, generate reports, and so on.

Cisco Meraki APs can be deployed automatically, once you register with the Meraki cloud. Each AP will contact the cloud when it powers up and will self-configure. From that point on, you can manage the AP through the Meraki cloud dashboard.

From the cloud, you can push out code upgrades and configuration changes to the APs in the enterprise

adds the intelligence needed to automatically instruct each AP on which channel and transmit power level to use.

Can also collect information from all of the APs about things such as RF interference, rogue or unexpected wireless devices that were overheard, and wireless usage statistics.

The data path from the wireless network to the wired network is very short; the autonomous AP links the two networks. Data to and from wireless clients does not have to travel up into the cloud and back; the cloud is used to bring management functions into the data plane.

the network in Figure 27-3 consists of two distinct paths—one for data traffic and another for management traffic, corresponding to the following two functions:


Distribution 
Layer 
Access Layer 
Cisco Meraki 
APs 
Cisco Meraki 
Cloud 
Management 
Trunk Link 
Trunk Link 
Data 
Figure 27-3 Cisco Meraki Cloud-Based Wireless Network Architecture ">

A control plane: Traffic used to control, configure, manage, and monitor the AP itself

A data plane: End-user traffic passing through the AP

Split-MAC Architectures



The real-time processes involve sending and receiving 802.11 frames, beacons, and probe messages. 802.11 data encryption is also handled in real time, on a per-packet basis. The AP must interact with wireless clients on some low level, known as the Media Access Control (MAC) layer. These functions must stay with the AP hardware, closest to the clients.

The management functions are not integral to handling frames over the RF channels, but are things that should be centrally administered. Therefore, those functions can be moved to a centrally located platform away from the AP.

When the functions of an autonomous AP are divided, the AP hardware is known as a lightweight access point, and performs only the real-time 802.11 operation.

the AP is left with duties in Layers 1 and 2, where frames are moved into and out of the RF domain. The AP becomes totally dependent on the WLC for every other WLAN function, such as authenticating users, managing security policies, and even selecting RF channels and output power.

The lightweight AP-WLC division of labor is known as a split-MAC architecture, where the normal MAC operations are pulled apart into two distinct locations.

the AP and WLC can be located on the same VLAN or IP subnet, but they do not have to be

CAPWAP relationship actually consists of two separate tunnels, as follows:

CAPWAP control messages: Carries exchanges that are used to configure the AP and manage its operation. The control messages are authenticated and encrypted, so the AP is securely controlled by only the appropriate WLC, then transported over the control tunnel.

CAPWAP data: Used for packets traveling to and from wireless clients that are associated with the AP. Data packets are transported over the data tunnel but are not encrypted by default. When data encryption is enabled for an AP, packets are protected with Datagram Transport Layer Security (DTLS).

Every AP and WLC must also authenticate each other with digital certificates. An X.509 certificate is preinstalled in each device when it is purchased.

Notice how VLAN 100 exists at the WLC and in the air as SSID 100, near the wireless clients—but not in between the AP and the WLC



Because the AP sits on the access layer where its CAPWAP tunnels terminate, it can use one IP address for both management and tunneling. No trunk link is needed because all of the VLANs it supports are encapsulated and tunneled as Layer 3 IP packets, rather than individual Layer 2 VLANs.

As the wireless network grows, the WLC simply builds more CAPWAP tunnels to reach more Aps



WLC can begin offering a variety of additional functions:

Dynamic channel assignment: The WLC can automatically choose and configure the RF channel used by each AP, based on other active access points in the area.

Transmit power optimization: The WLC can automatically set the transmit power of each AP based on the coverage area needed.

Self-healing wireless coverage: If an AP radio dies, the coverage hole can be “healed” by turning up the transmit power of surrounding APs automatically.

Flexible client roaming: Clients can roam between APs with very fast roaming times.

Dynamic client load balancing: If two or more APs are positioned to cover the same geographic area, the WLC can associate clients with the least used AP. This distributes the client load across the APs.

RF monitoring: The WLC manages each AP so that it scans channels to monitor the RF usage. By listening to a channel, the WLC can remotely gather information about RF interference, noise, signals from neighboring APs, and signals from rogue APs or ad hoc clients.

Security management: The WLC can authenticate clients from a central service and can require wireless clients to obtain an IP address from a trusted DHCP server before allowing them to associate and access the WLAN.

Wireless intrusion protection system: Leveraging its central location, the WLC can monitor client data to detect and prevent malicious activity.

Comparing Wireless LAN Controller Deployments

locate the WLC in a central location so that you can maximize the number of APs joined to it. This is usually called a unified or centralized WLC deployment



Typical unified WLCs can support a maximum of 6000 APs

A WLC can also be located in a central position in the network, inside a data center in a private cloud, as shown in Figure 27-9. This is known as a cloud-based WLC deployment, where the WLC exists as a virtual machine rather than a physical device.

Such a controller can typically support up to 3000 APs.



For small campuses or distributed branch locations, where the number of APs is relatively small in each, the WLC can be co-located with a stack of switches, as shown in Figure 27-10. This is known as an embedded WLC deployment

Typical Cisco embedded WLCs can support up to 200 APs. The APs do not necessarily have to be connected to the switches that host the WLC



the WLC function can be co-located with an AP that is installed at the branch site. This is known as a Cisco Mobility Express WLC deployment

A Mobility Express WLC can support up to 100 APs.





Cisco AP Modes

WLC, you can also configure a lightweight AP to operate in one of the following special-purpose modes:

Local: The default lightweight mode that offers one or more functioning BSSs on a specific channel. During times that it is not transmitting, the AP will scan the other channels to measure the level of noise, measure interference, discover rogue devices, and match against intrusion detection system (IDS) events.

Monitor: The AP does not transmit at all, but its receiver is enabled to act as a dedicated sensor. The AP checks for IDS events, detects rogue access points, and determines the position of stations through location-based services.

FlexConnect: An AP at a remote site can locally switch traffic between an SSID and a VLAN if its CAPWAP tunnel to the WLC is down and if it is configured to do so.

Sniffer: An AP dedicates its radios to receiving 802.11 traffic from other sources, much like a sniffer or packet capture device. The captured traffic is then forwarded to a PC running network analyzer software such as Wildpackets OmniPeek or WireShark, where it can be analyzed further.

Rogue detector: An AP dedicates itself to detecting rogue devices by correlating MAC addresses heard on the wired network with those heard over the air. Rogue devices are those that appear on both networks.

Bridge: An AP becomes a dedicated bridge (point-to-point or point-to-multipoint) between two networks. Two APs in bridge mode can be used to link two locations separated by a distance. Multiple APs in bridge mode can form an indoor or outdoor mesh network.

Flex+Bridge: FlexConnect operation is enabled on a mesh AP.

SE-Connect: The AP dedicates its radios to spectrum analysis on all wireless channels. You can remotely connect a PC running software such as MetaGeek Chanalyzer or Cisco Spectrum Expert to the AP to collect and analyze the spectrum analysis data to discover sources of interference.

Remember that a lightweight AP is normally in local mode when it is providing BSSs and allowing client devices to associate to wireless LANs. When an AP is configured to operate in one of the other modes, local mode (and the BSSs) is disabled.
