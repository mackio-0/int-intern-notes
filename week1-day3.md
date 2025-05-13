# Day 3

## Networking for Hackers by Khit Min Nyo

### DHCP

* Dynamic Host Configuration Protocol
* LAN - same public IP address for all the devices in the LAN, use private IP address to commnunicate with each other
* When connecting to a new network, device will request an IP address from the DHCP server
* DHCP server after receiving the request will response with DHCP offer, including the IP address, subnet mask, gateway, DNS server, lease duration, etc.
* DHCP offer may be received one or multiple at the same time, device choose one offer and send back another response to confirm what it has chosen and it has accepted the offer.
* After receiving the confirm response, DHCP server will send DHCPACK response and allow the device to use the IP address.
* Device will use the information from DHCPACK response to configure the IP address, subnet mask, gateway, DNS server, etc.

### NAT

* Network Address Translation
* Translates internatl private IP address to public IP address
* can't connect to internet with private IP address ocse the are not unique and more.
* For example, if we have 2 computers in the same network and if we open a website from one computer it should be opened in that computer not the other one.
* Web server need to know which computer the request comes from to send back the respones.
* NAT receives the request to outside internet and note it in a table and translate the machine IP to external IP.
* When the request comes back from server, NAT will check the table and translate the external IP back to internal IP.

### Port

* Port are sub-addresses, virtual doorways on your computer that allow different network services and applications to communicate.
* One IP address, many services: Your computer has one IP address, like a building has one street address. Ports allow multiple applications (like a web browser, email client, game) running on the same computer to use that single network connection simultaneously.
* Software-based: Ports are not physical entities but rather software-level constructs managed by your computer's operating system.
* Communication Endpoints: A port is a virtual point where a network connection starts and ends. Every network communication involves both a source port and a destination port.
* Multiplexing: Ports enable multiplexing, meaning multiple applications or services can share the same underlying network connection. Without ports, each service would require its own dedicated network connection, which would be inefficient.
* Port range: 2 power 16 (65536 but 0 is invalid) - 1-65535 (0-1023 reserved for well-known ports)

### TCP

* Transmission Control Protocol
* Reliable, ordered, error-checked delivery of data segments across a computer network.
* TCP is a connection-based protocol, meaning that it requires a connection to be established before data can be sent.
* TCP handle transmission speed based on how much information the receivers can handle.
* Three-way TCP handshake, sync - sync-ack - ack (synchronize sequent number segment)
* why three-way handshake? - without sequent number sharing of both by it, combining the data segment in order will be messy.
* From security viewpoint, without three-way handshake, unauthorized access can be done.
* When terminating the connection, TCP will send a FIN packet to both sides of the connection.

### OSI model

* Open System Interconnection Model

#### 7 layers of OSI model

* Physical layer, data link layer, network layer, transport layer, session layer, presentation layer, application layer
* Physical layer - physical connection between devices (e.g. ethernet, wifi, hubs, repeaters, etc.), info as bits, 0 and 1, responsible for transmission between one node to another, receives data signals and translate them back to 0 and 1 and send them to the next layer, DLL.
  * Protocol Data Unit - bits
* Datalink Layer - responsible for the reliable transfer of data between two directly connected nodes. It takes the raw bitstream from the Physical Layer and packages it into frames. It also handles physical addressing (MAC addresses), error detection, and flow control within a local network segment, framing - divide bitstream into manageable blocks, MAC adderessing - use physical address of Network Interface Card (NIC) to identify each device in the network. Error detection - detect errors in the data stream. Flow control - control the rate of data transfer between two nodes. Media Access Control (MAC) - determines how multiple devices on a shared medium (like Ethernet) access the network.
  * Protocol Data Unit - frames
* Network Layer - This layer is responsible for routing data packets across a network. It handles logical addressing (IP addresses), determines the best path for data to travel from source to destination across multiple networks, and manages network congestion. Logical addressing (IP addressing): Assigns unique logical addresses to devices to identify them across the internetwork, Routing: Determines the optimal path for data packets to travel from source to destination, Packet forwarding: Moves packets from an input interface to an output interface based on routing decisions, Fragmentation and reassembly, Quality of Service (QoS)
  * Protocol Data Unit (PDU) - packets or datagram
* Transport Layer - This layer provides end-to-end communication between applications running on different hosts. It's responsible for segmenting data from the Application Layer, ensuring reliable or unreliable delivery, flow control, and multiplexing/demultiplexing of data streams. transmit ack and error Detects and retransmits lost or corrupted segments (in TCP), TCP, UDP, NetBIOS and PPTV works on this layer
  * Protocol Data Unit (PDU) - segment (for TCP) or Datagram (for UDP)
* Session Layer - starting the connection, maintaing session and synchronizes communication between applications.
 API and Remote Procedure Call (RPC) works on this layer. In TCP/IP model layer 5, 6, 7 are application layer.
  * Protocol Data Unit (PDU) - data
* Presentation Layer - This layer is responsible for data formatting, encryption, and compression. It ensures that data sent by the Application Layer of one system is in a format that the Application Layer of another system can understand. performs encryption, compression and data formatting. SSL/TLS in https, SSH, FTP, IMAP, MPEG etc works on this layer.
  * Protocol Data Unit (PDU) - data
* Application Layer - This is the layer closest to the end-user. It provides the interface between network applications and the underlying network services. It includes protocols that applications use to exchange data, such as web browsing (HTTP/HTTPS), email (SMTP, POP3, IMAP), and file transfer (FTP). User interface (indirectly): While the layer itself doesn't have a user interface, it supports the applications that do.

### Subnetting and CIDR notation

* Subnetting - dividing a larger network into smaller networks that is easier to manage
* CIDR - Classless Inter-Domain Routing
* Wildcard Mask - opposite of subnetmask, 255.255.255.0 -> 0.0.0.255, used in Access Control List (ACL),  example. apply 0.0.0.254 wildcard mask to 10.10.10.1 and it will only match odd ending IP address between 10.10.10.1 and 10.10.10.255, vice versa 0.0.0.254 to 10.10.10.2 will only match even ending IP address between 10.10.10.2 and 10.10.10.254

#### Understanding the leading bits

* Class A - 0.0.0.0 to 127.255.255.255 why? in class A the first leading bit is 0, so 00000000 to 01111111 being 127
* Class B - 128.0.0.0 to 191.255.255.255 why? in class B the first leading bit is 10, so 10000000 to 10111111 being 128-191
* Class C - 192.0.0.0 to 223.255.255.255 why? in class C the first leading bit is 110, so 11000000 to 11011111 being 192-223
* Class D - 224.0.0.0 to 239.255.255.255 why? in class D the first leading bit is 1110, so 11100000 to 11101111 being 224-239
* Class E - 240.0.0.0 to 255.255.255.255 why? in class E the first leading bit is 11110, so 11110000 to 11111111 being 240-255

### Address Resolution Protocol (ARP)

* ARP - Address Resolution Protocol is a communication protocol used to resolve IP addresses to physical hardware addresses. (MAC addresses) - mapping IP address to MAC address, find the MAC address associated with a given IPv4 address on the local network.
* How ARP Works (The Basic Process):

Let's say Host A (IP: 192.168.1.10, MAC: AA-AA-AA-AA-AA-AA) wants to send data to Host B (IP: 192.168.1.20, MAC: needs to be discovered).

ARP Request:

1. Host A first checks its ARP cache. The ARP cache is a table stored in the device's memory that contains recently resolved IP-to-MAC address mappings.
2. If Host B's IP address is found in Host A's ARP cache, Host A can directly use the associated MAC address to encapsulate the data frame and send it.
3. If Host B's IP address is not in Host A's ARP cache, Host A needs to perform ARP. It creates an ARP request packet.
4. The ARP request packet contains:
    * The sender's MAC address (AA-AA-AA-AA-AA-AA).
    * The sender's IP address (192.168.1.10).
    * The target IP address (192.168.1.20) for which it needs the MAC address.
    * The target MAC address field is filled with a broadcast MAC address (FF:FF:FF:FF:FF:FF) because Host A doesn't know it yet.
5. This ARP request is encapsulated in an Ethernet frame with the destination MAC address set to the broadcast address, ensuring that all devices on the local network receive it.

ARP Reply:

1. Every device on the LAN receives the ARP request.
2. Each device examines the target IP address in the ARP request.
3. If a device's IP address matches the target IP address in the ARP request (in this case, Host B with IP 192.168.1.20), it prepares an ARP reply packet.
4. he ARP reply packet contains:
    * The responder's MAC address (BB-BB-BB-BB-BB-BB - Host B's MAC).
    * The responder's IP address (192.168.1.20).
    * The sender's MAC address from the request (AA-AA-AA-AA-AA-AA).
    * The sender's IP address from the request (192.168.1.10).
5. The ARP reply is encapsulated in an Ethernet frame with the destination MAC address set to the unicast MAC address of the requesting host (AA-AA-AA-AA-AA-AA), so only Host A receives it.

ARP Cache Update:

1. When Host A receives the ARP reply, it extracts the IP-to-MAC address mapping (192.168.1.20 is now associated with BB-BB-BB-BB-BB-BB).
2. Host A stores this mapping in its ARP cache.
3. Now that Host A has Host B's MAC address in its cache, it can encapsulate the original data packets in Ethernet frames with the destination MAC address of Host B and send them directly.
4. Host B also typically updates its ARP cache with Host A's IP-to-MAC mapping from the ARP request it received.

### DNS

* Domain Name System (DNS) - Note in day1 note
* DNS resolvers - also known as recursive DNS servers, are the workhorses of the Domain Name System (DNS) lookup process. They act as intermediaries between your computer or device (the DNS client or stub resolver) and the authoritative DNS servers that hold the actual DNS records for websites and other internet resources.
  * Recursive Querying: The key characteristic of a DNS resolver is that it performs recursive queries. This means that if the resolver doesn't have the answer in its cache, it takes on the responsibility of contacting other DNS servers on your behalf to find the answer. It will traverse the DNS hierarchy, starting from the root servers, then the top-level domain (TLD) servers (.com, .org, etc.), and finally the authoritative name servers for the specific domain.
  * Caching: DNS resolvers maintain a cache of recently resolved DNS records. If a client requests the same domain name again (or if another client on the same network requests it), the resolver can often provide the answer directly from its cache, without needing to go through the entire resolution process again. This significantly speeds up DNS lookups and reduces the load on authoritative name servers.
  * Returning the Answer: Once the resolver finds the IP address (or another requested DNS record), it sends the answer back to the original client that made the request.
* Name Servers - Root nameservers, TDL nameservers, and authoritative nameservers
* Name Space - like a big library, it contains all possible domain name and respective IP addresses. Unresolved DNS query wil end up here.
* DNS records
  * SOA records - Start of Authority records - display important information about the DNS zone, such as primary master nameserver, root, serial number, zone admin email address, refresh interval, retry interval, and cache info expiration time, time to live (TTL).
  * A and AAAA records - Address records - map domain names to IP addresses. A for IPv4 and AAAA for IPv6.
  * CNAME records - Canonical Name records - map nickname of the domain name to another domain name.
  * NS Record (Name Server) - An NS record specifies the authoritative name servers for a particular domain or subdomain. It tells the rest of the internet which servers are responsible for holding and providing the DNS records for that domain.
  * MX record - Mail Exchange records - map domain names to mail servers.
  * PTR record - Pointer records - map IP addresses to domain names.
  * SRV records - Service records - map domain names to services.

### SMTP

* Simple Mail Transfer Protocol (SMTP) - Application layer protocol use TCP connection for reliable ordered data stream channel.

### HTTP

* Hypertext Transfer Protocol (HTTP)
* HTTP Methods (Verbs)
  * GET - retrieve data
  * POST - send data
  * PUT - update data
  * DELETE - delete data as a whole
  * PATCH - partially update data
  * HEAD - retrieve headers only
  * OPTIONS - retrieve options
  * TRACE - trace route
  * CONNECT - tunneling
* HTTP Headers - Metadata, exchange of information between client and server
  * Request Headers - sent by the client to the server
  * Response Headers - sent by the server to the client
* HTTP Cookies - HTTP is stateless so cookies are used to store information on the client side
