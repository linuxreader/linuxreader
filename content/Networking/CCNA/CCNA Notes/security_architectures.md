This chapter covers the following exam topics:

5.0 Security Fundamentals

5.1 Define key security concepts (threats, vulnerabilities, exploits, and mitigation techniques)

5.2 Describe security program elements (user awareness, training, and physical access control)

5.4 Describe security password policies elements, such as management, complexity, and password alternatives (multifactor authentication, certificates, and biometrics)

5.8 Differentiate authentication, authorization, and accounting concepts

Attacks That Spoof Addresses

- attacker sends packets with a spoofed source IP address
- attacker sends spoofed MAC addresses
- DHCP requests with spoofed MAC addresses can be sent to a  DHCP server

- filling its address lease table and leaving no free IP addresses for normal use.

Denial-of-Service Attacks

- server adds the TCP connection and replies to the fake address with a SYN-ACK.

- no ack replies cause the server to leave open the connection and fill up the table
- Server is no longer able to do legitimate TCP connections

- ICMP echo (ping) packets
- flood of UDP packets, and TCP connections, such as the TCP SYN flood

- distributed denial-of-service (DDoS)

- attack is distributed across a large number of bots, all flooding or attacking the same target.

Reflection and Amplification Attacks

Reflection attack

- Attacker sends packets to the reflector host
- host (reflector) reflects the exchange toward the spoofed address that is the target.
- The attacker might also send the spoofed packets to multiple reflectors, causing the target to receive multiple copies of the unexpected traffic.
- Attackers source packet is the IP of the victim

- when reflector (Corporate server) responds, it sends packets to victim

Amplification attack

- attacker sends a small amount of traffic to a reflector to generate large volume of traffic to a target
- leverages a protocol or service to generate the traffic
- mechanisms of DNS and NTP have been exploited for this

Man-in-the-Middle Attacks

- can exploit the ARP table to communicate with other hosts on the local network
- attacker sends the last ARP reply so that any listening host will update its ARP table with the most recent information.
- attacker can repeat this process by poisoning the ARP entries on multiple hosts and then relaying traffic between them without easy detection.

Address Spoofing Attack Summary

DoS/DDoS

- Exhaust a system service or resource
- Crash target system

Reflection/ Amplification

- Trick unwilling accomplice host to send traffic to target

man-in-the-middle

- Eavesdrop on traffic
- Modify traffic passing through

Reconnaissance Attacks

- discover details about the target and its systems prior to an actual attack.
- use common tools to uncover public details like who owns a domain and what IP address ranges are used there.
- nslookup can reveal the owner of the domain and the IP address space registered to it.
- The whois and dig commands are complementary tools that can query DNS information to reveal detailed information about domain owners, contact information, mail servers, authoritative name servers, and so on.
- using ping sweeps to send pings to each IP address in the target range. Hosts that answer the ping sweep then become live targets.
- Port scanning tools can then sweep through a range of UDP and TCP ports to see if a target host answers on any port numbers. Any replies indicate that a corresponding service is running on the target host.
- not a true attack because nothing is exploited as a result.

Buffer Overflow Attacks

- incoming data might be stored in unexpected memory locations if a buffer is allowed to fill beyond its limit.
- sending data that is larger than expected.
- If a vulnerability exists, the target system might store that data, overflowing its buffer into another area of memory, eventually crashing a service or the entire system.
- attackers specially craft the large message by inserting malicious code in it.

- If the target system stores that data as a result of a buffer overflow, then it can potentially run the malicious code without realizing.

Malware

- Trojan horse

- can spread from one computer to another only through user interaction
- such as opening email attachments, downloading software from the Internet, and inserting a USB drive into a computer.
- Packaged inside other software

- viruses

- can propagate between systems more readily. To spread, virus software must inject itself into another application, then rely on users to transport the infected application software to other victims.
- Self-injected into other software

- Worms

- -propagates automatically

Human Vulnerabilities

Spear phishing

- targets a group of similar users who might work for the same company, shop at the same stores, and so on.
- all receive the same convincing email with a link to a malicious site.
- Whaling

- similar to phishing
- targets high-profile individuals in corporations, governments, and organizations.

- voice calls (vishing)
- SMS text messages (smishing).

Pharming:

- altered DNS service or local hosts file entry for a legitimate site.
- altered name resolution returns the address of a malicious site instead.

Watering Hole:

- frequently visited site is compromised and malware is deposited there.
- infects only the target users who visit the site

Attack Type 
Social 
engineering 
Phishing 
Spear phishing 
Whaling 
Vishing 
Smishing 
Pharming 
Watering hole 
Goal 
Exploits human trust and social behavior 
Disguises a malicious invitation as something legitimate 
Targets group of similar users 
Targets high-profile individuals 
Uses voice calls 
Uses SMS text messages 
Uses legitimate services to send users to a compromised 
site 
Targets specific victims who visit a compromised site ">

Securing Network Access with Cisco AAA

Authentication, Authorization, and Accounting

Authentication

\- Name and password

\- Challenge and response

\- Token cards

Authorization

\- Takes place after authentication is validated

\- Provides needed resources specifically allowed to a certain user and permits the operation that specific user is allowed to perform

Accounting

\- Records what the user did on the network as well as the resources they accessed.

\- Keeps track of how much time they spent using network resources.

Authentication Methods

Least secure to most secure methods:

- No username or password
- Username/password (static)
- Aging username/password
- One-time passwords (OTP
- Token cards/soft tokens
