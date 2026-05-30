Welcome to the CCNA OSPF lab. You'll need to configure OSPF so that there are full routing capabilities in the network. As well as use show commands to verify your network. Then you will troubleshoot and fix a broken OSPF network.

Once again, this will all be done in Packet Tracer. But feel free to do this lab on real equipment, GNS-3, etc.

## OSPF Setup

There are two main ways to configure OSPF. You can go into OSPF configuration mode. Or you can configure OSPF in Interface configuration mode. I will show you how to configure router1. You will need to configure the other routers as well.

## Step 1. Configure OSPF on routers 1, 2, and 3.

Do not configure OSPF on router3 link to isprouter. Also, make sure to include the loopback interfaces in your OSPF networks.

### Enter OSPF Configuration Mode

router1 (config) # router ospf 1

### Define networks for OSPF to advertise

router1 (config-router) network 10.0.1.0 0.0.0.255 area 0

router1 (config-router) network 192.168.1.0 0.0.0.255 area 0

router1 (config-router) network 1.1.1.1 0.0.0.0 area 0

or

### Configure OSPF in Interface configuration mode

Interface configuration

router1 (config-if) # Ip ospf 1 area 0

## Step 2.  Configure router1 connection to the 192.168.1.0/24 network as a passive interface

Make ospf interfaces that are not connected to a router stop trying to for neighbor relationships. (Stop sending hellos and stop processing hellos)

passive interface type number

## Step 3. Configure router3 to us isprouter as the default gateway and to advertise the default route.

Set a default route

ip route 0.0.0.0 0.0.0.0 19.0.0.1

Advertise a default route

default-information originate

## Step 4. Ping isprouter from PC1 to verify connection

## Step 5. Use Show Commands to view OSPF neighbors, routing Table, and the OSPF LSDB

Show Commands

Show IP OSPF neighbor (neighbors)

Show ip ospf database (Link State database)

show ip route

show ip ospf interface

20 lines of output per interface

show ip ospf interface gi0/0

show ip ospf interface brief

To check if ospf is enabled on the proper interfaces. Single line of output per interface

show ip ospf rib (spf calculation)

Show ip route

Show ip route ospf

Show ip protocols

Broken OSPF Lab

More OSPF Labs

## Other helpful OSPF commands

Configure OSPF Router ID (preferred by router)

router1 (config-router) # router-id 1.1.1.1

Create a loopback interface

Highest loopback interface with the "up" status is used as the RID if the RID is not manually configured.

If no loopbacks are configured, and the RID is not manually configured, then the highest IP with an "up" ststus is used as the RID.

interface loopback 0.

ip address 1.1.1.1

Clear IP OSPF process

Restart the OSPF process. This can be used so RID configurations will take effect.

Change the cost of an interface

ip ospf cost 4

## Change the reference bandwidth

router ospf # auto-cost reference bandwidth 10000

## Change the interface bandwidth

bandwidth speed (kbps)

## OSPF Load balancing

maximum paths 5

## Set Ospf Network to point-to-point

Must be set on both routers

router1 (config-if) # ip ospf network point-to-point

## Set the ospf network type back to broadcast

ip ospf network broadcast
