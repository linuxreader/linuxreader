This chapter covers the following exam topics:

1.0 Network Fundamentals

1.1 Explain the Role of Network Components

1.1.c Next-generation Firewalls and IPS

4.0 IP Services

4.8 Configure network devices for remote access using SSH

5.0 Security Fundamentals

5.3 Configure device access control using local passwords

Encrypting Older IOS Passwords with service password-encryption

service password-encryption

- encrypts passwords that are normally held as clear text

- password password (console or vty mode)
- username name password password
- enable password password

- encoding type of “7”

no service password-encryption

- passwords remain encrypted until password is changed

Hashing the enable secret

- never stores the clear-text password
- IOS computes the MD5 hash of the password in the enable secret command and stores the hash of the password in the configuration.
- IOS hashes the clear-text password as typed by the user.
- IOS compares the two hashed values

no enable secret

- Can be used without having to enter the password

Improved Hashes for Cisco’s Enable Secret

The two newer alternative algorithm types

- Both use an SHA-256 hash instead of MD5

enable \[algorithm-type5\] secret password

- Type 5
- MD5

enable algorithm-type sha256 secret password

- Type 8
- SHA-256

enable algorithm-type scrypt secret password

- Type 9
- SHA-256

- New enable secret commands with different algorithm types replace any existing enable secret command.

Encoding the Passwords for Local Usernames

Username secret command Encoding

username name \[algorithm-type5\] secret password

username name algorithm-type sha-256 secret password

username name algorithm-type scrypt secret password

Controlling Password Attacks with ACLs

(line vty)# access-class 3 in

- Bond ACL 3 to a vty line

(line vty)# access-class 3 out

- looks at the destination IP address instead of the source
- filters based on the device to which the telnet or ssh command is trying to connect.

Traditional Firewalls

- match the source and destination IP addresses
- matching their static well-known TCP and UDP ports
- additional TCP and UDP ports 
- Match the text in the URI of an HTTP request

- state information (stateful firewall)

- storing information about each packet
- make decisions about filtering future packets based on the historical state information (stateful inspection)
- could be tracking the number of TCP connections per second
- recording state information based on earlier packets

- stateless firewall or a router ACL

- would not have had the historical state information to realize that a DoS attack was occurring.

Security Zones

- defining which hosts can initiate new connections.

- Zone Inside

- Secure

- Zone Outside

- Not secure

- DMZ

- Access by public

Intrusion Prevention Systems (IPS

- IPS first downloads a database of exploit signatures
- IPS can log the event, discard packets, or even redirect the packets to another security application for further examination.
- needs to download and keep updating its signature database.

Cisco Next-Generation Firewalls

Cisco  firewall

- Cisco Adaptive Security Appliance (ASA).

- stateful filtering

- comparing fields in the IP, TCP, and UDP headers, and using security zones when defining firewall rules

Application Visibility and Control (AVC)

looks at the application layer data to identify the application

- deep packet inspection
- can identify many applications based on the data sent (application layer headers plus application data structures far past the TCP and UDP headers).

Duties of a NGFW

Traditional firewall:

stateful firewall filtering, NAT/PAT, and VPN termination.

Application Visibility and Control (AVC)

Advanced Malware Protection:

- A network-based antimalware function can run on the firewall itself, blocking file transfers that would install malware, and saving copies of files for later analysis.

URL Filtering: This feature

- examines the URLs in each web request, categorizes the URLs, and either filters or rate limits the traffic based on rules. The Cisco Talos security group monitors and creates reputation scores for each domain known in the Internet, with URL filtering being able to use those scores in its decision to categorize, filter, or rate limit.

NGIPS

Cisco Next-Generation IPS

- examines the context by gathering data from all the hosts and the users of those hosts.
- will know the OS, software revision levels, what apps are running, open ports, the transport protocols and port numbers in use, and so on.
- Armed with that data, the NGIPS can make much more intelligent choices about what events to log.

NGIPS Duties

Traditional IPS:

- using exploit signatures to compare packet flows, creating a log of events, and possibly discarding and/or redirecting packets.

Application Visibility and Control (AVC):

Contextual Awareness:

- gather data from hosts—OS, software version/level, patches applied, applications running, open ports, applications currently sending data, and so on. Those facts inform the NGIPS as to the often more limited vulnerabilities in a portion of the network so that the NGIPS can focus on actual vulnerabilities while greatly reducing the number of logged events.

Reputation-Based Filtering:

can perform reputation-based filtering, taking the scores into account.

Event Impact Level:

provides an assessment based on impact levels
