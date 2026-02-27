# Spanning Tree Protocol

- Prevents loops in a network.
- Loops can create Broadcast storms, Mac table instability, and Multiple frame transmission.

## Broadcast Storms

1. Switch receives a broadcast frame.
2. Switch transmits that frame out of all ports.
3. The broadcast frames will keep being received on each switch then broadcasted again.

## Mac Table instability

1. Broadcast storm is causing broadcast packets going in and out of every port, the 
2. Switch has to keep updating the source mac address of those packets.

## Multiple frame transmission

Issues can be cause when multiples of the same frame are received on a port.

Spanning tree must build a topology so it can figure out if there is a loop in the network, and pick which port(s) to block to stop any network loops.

Spanning tree controls all ports in the network and puts them in different port “states”. These states help spanning tree learn the network topology and decide which ports to forward or block. This process is called convergence. Convergence happens whenever spanning tree realizes that there has been a change in the network. (a new switch is added or a link goes down.)The port states are Blocking, Listening, Learning, Forwarding and Disabled.

Blocking

• Stable state

• Mac addresses are not learned on this port

• Port does not forward any fram

Listening

• Removes old or “stale” mac address entries from this port if no frames are recieved for that mac address while it is in this state.

• Transitory state that is temporary while spanning tree converges.

• A port in this state does not forward any frames.

• Incoming frames mac addresses are not learned.

Learning

• Incoming mac addresses on this port are now learned.

• A port in this state does not forward any frames.

• Also a port in a transitory state.

Disabled

• Stable

• Mac addresses are not learned on this port

• Port does not forward any frames

How spanning tree decides.

To build out the topology, spanning tree elects designated ports and root ports and determines the lowest cost path to the root bridge. (root cost)

Cost is based on the operating speed of the link. Here are the cost values STP uses based on the speed of the link.

These are the cisco defaults:

10 Mbps = 100

100 Mbps = 19

1 gbps = 4

10 gbps = 2

100 gbps = N/A

1 tbps = N/A

You can change the values to a newer form with:

\# spanning-tree pathcost method long

Which changes the values to:

10 Mbps = 2,000,000

100 Mbps = 200,000

1 gbps = 20,000

10 gbps = 2,000

100 gbps = 200

1 tbps = 20

Designated ports and root ports

Spanning tree sends Hello BPDUs to determine the spanning tree topology. A port that sends forwards hello bpdus is called a designated port. A port that receives hello bpdus is called a designated port.

Hello BPDU

Hello bpdu contains a switches Bridge ID, the root bridge ID, the senders root cost. (cost to the root bridge), and the root switches timer values.

Bridge ID (BID)

Has an 8 byte value that is different on each switch. A 2-byte priority field. And a 6 byte system ID that is based on the mac address

Root Bridge ID

The Bridge ID of the switch that the sender thinks is currently the root switch.

Root Switches Timer Values

Hello Timer = How often Hello bpdus are sent

MaxAge Timer = The max time that spanning tree will wait not getting a hello bpdu before it reconverges the network.

Forward Delay Timer = How long a port stays in the listening and learning states.

Root Cost

The cost between the sender and the root bridge

Spanning tree elects a root bridge then decides the best path to the root bridge from all of the other switches. If there are multiple paths to the root bridge, then the worst path is blocked. All ports on the root bridge are set to a forwarding state.

The other switches elect a port that has the lowest cost path to the root bridge. These ports are called root ports. Root ports are always in a forwarding state.
