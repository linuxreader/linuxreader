## TCP/IP Model Layers
- Application
- Transport
- Network
- Data Link
- Physical

### Application Layer

The application level layer controls services for the applications themselves. Including:

#### HTTP
-   HTTP provides interface between software and the network.
-   HTTP traffic
    - PC > | HTTP Header | GET home.html | > Web server
    - Web server > | HTTP Header | OK | Data | home.html | > PC

## Key terms
Term: Same layer interaction on different computers.
Defenition: Communicate with the same layer on another computer.

Term: Adjacent layer interaction on the same computer
Definition: One lower layer provides a service to the layer just above. The higher layer makes the next lower layer perform a function.

Note: Wireless protocols function on Layer 2.

Encapsulation

-   TCP/ Data (segment)
-   IP/ Data (Packet)
-   LH/ Data/ LT (Frame) (Link header & Link Trailer)

Protocol Data Units

-   L7H/ Data (L7PDU)
-   L6H/ Data (L6PDU)
-   L5H/ Data (L5PDU)
-   L4H/ Data (L4PDU)
-   L3H/ Data (L3PDU)
-   L2H/ Data/ L2T (L2PDU)
