---
title: TCP/IP Basics
---

ApplicationTransport
NetworkData Link
Physical  
Application Layer
- services for applications
- http provides interface between software and the network  
    ○ HTTP Header GET home.html  
    ○ HTTP Header OK/ Data


Same layer interaction on different computers

- Communicate with the same layer on another computer.  
    Adjacent layer interaction on the same computer
- - One lower layer provides a service to the layer just above. The higher layer makes the next lower layer perform a function

*Wireless protocols are Layer 2  
Encapsulation

- - TCP/ Data (segment)IP/ Data (Packet)
- LH/ Data/ LT (Frame) (Link header & Link Trailer)  
    Protocol Data Units
- - L7H/ Data (L7PDU)L6H/ Data (L6PDU)
- L5H/ Data (L5PDU)
- L4H/ Data (L4PDU)
- - L4H/ Data (L4PDU)L3H/ Data (L3PDU)
- L2H/ Data/ L2T (L2PDU)
