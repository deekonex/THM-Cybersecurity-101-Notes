# Networking Concepts

## Learning Objectives

By the end of this module, you will understand:
- The ISO OSI network model and its seven layers
- IP addresses, subnets, routing, and network design
- TCP and UDP protocols, and how they differ
- Port numbers and their role in identifying services
- How to connect to services using telnet from the command line
- Data encapsulation and how packets are structured

## Core Models and Protocols

### The OSI Model (Open Systems Interconnection)

The OSI Model is a conceptual framework developed by the International Organization for Standardization (ISO) that describes how communications occur in a computer network. Although theoretical, it is essential for understanding networking concepts at a deeper level. The model defines seven layers, each with specific responsibilities.

| Layer | Name | Responsibility | Data Unit (PDU) |
|-------|------|-----------------|-----------------|
| 7 | Application | Interfaces directly with user applications and services | Data |
| 6 | Presentation | Formats data (encryption, compression, encoding) | Data |
| 5 | Session | Manages and maintains connections between applications | Data |
| 4 | Transport | Provides end-to-end communication (reliable or unreliable) | Segment (TCP) / Datagram (UDP) |
| 3 | Network | Routes packets across different networks | Packet (IP) |
| 2 | Data Link | Ensures error-free data transfer between nodes on the same network | Frame |
| 1 | Physical | Handles hardware transmission through cables and signals | Bits |

**Key Concepts by Layer:**

- **Layer 7 (Application)**: Where user applications interact with the network. Examples: HTTP, FTP, SMTP, DNS.
- **Layer 6 (Presentation)**: Handles encoding formats, encryption, and compression. Ensures data readability across different systems.
- **Layer 5 (Session)**: Manages the establishment, maintenance, and termination of sessions between applications.
- **Layer 4 (Transport)**: Determines whether communication is reliable (TCP) or fast (UDP). Adds port numbers.
- **Layer 3 (Network)**: Routes packets using IP addresses. Routers operate at this layer.
- **Layer 2 (Data Link)**: Uses MAC addresses to transfer frames on the local network. Switches operate at this layer.
- **Layer 1 (Physical)**: Deals with physical media like cables, connectors, and electrical signals.

**Quick Reference Questions:**
- Which layer is responsible for connecting one application to another? **Layer 4 (Transport)**
- Which layer is responsible for routing packets to the proper network? **Layer 3 (Network)**
- Which layer is responsible for encoding the application data? **Layer 6 (Presentation)**
- Which layer is responsible for transferring data between hosts on the same network segment? **Layer 2 (Data Link)**

### The TCP/IP Model

The TCP/IP Model is a practical four-layer framework used to describe Internet protocols. It is less theoretical than the OSI model and directly reflects how the modern Internet actually functions.

**TCP/IP Layers and OSI Mapping:**

| TCP/IP Layer | OSI Layers | Examples | Primary Protocols |
|--------------|-----------|----------|-------------------|
| Application | 5, 6, 7 | HTTP, HTTPS, FTP, SMTP, SSH, DNS | Application protocols |
| Transport | 4 | TCP, UDP | Transport protocols |
| Internet | 3 | IP addressing and routing | IP (IPv4, IPv6) |
| Link | 1, 2 | Ethernet, WiFi, MAC addressing | Ethernet, WiFi |

The TCP/IP model is the framework you will most commonly encounter in practice. Notice that the Application layer in TCP/IP covers three OSI layers, while the Link layer combines two OSI layers.

**Quick Reference Questions:**
- To which layer does HTTP belong in the TCP/IP model? **Application Layer**
- How many layers of the OSI model does the application layer in the TCP/IP model cover? **3 layers (5, 6, and 7)**

## IP Addresses and Subnets

### Understanding IP Addresses

An IP address is a unique numerical identifier assigned to each device connected to a network. Without a unique identifier, other hosts cannot find and communicate with a device.

**IPv4 Address Format:**
- 32-bit address divided into four octets (8 bits each)
- Each octet ranges from 0 to 255
- Displayed as four decimal numbers separated by periods
- Example: 192.168.0.1

**How to Calculate Binary Representation:**
Each octet is a binary number. For example, the first octet 192 in binary is 11000000.

### Looking Up Your Network Configuration

**Windows Command:**
```
ipconfig
```

Output includes your IP address, subnet mask, and default gateway.

**Linux/UNIX Commands:**
```
ifconfig
```
or
```
ip a s
(ip address show)
```

**Example Output Interpretation:**

From `ifconfig`:
```
wlo1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.66.89  netmask 255.255.255.0  broadcast 192.168.66.255
        ether cc:5e:f8:02:21:a7  txqueuelen 1000  (Ethernet)
```

This tells us:
- IP address: 192.168.66.89
- Subnet mask: 255.255.255.0
- Broadcast address: 192.168.66.255
- MAC address (Layer 2): cc:5e:f8:02:21:a7

From `ip a s`:
```
inet 192.168.66.89/24 brd 192.168.66.255 scope global dynamic noprefixroute
```

This tells us:
- IP address: 192.168.66.89 with /24 CIDR notation
- Broadcast address: 192.168.66.255

### Subnet Masks and Subnetting

**Purpose of Subnetting:**
Subnetting divides a larger network into smaller subnetworks (subnets). This improves security, network performance, and organization.

**Subnet Mask Basics:**
A subnet mask is a 32-bit number that divides an IP address into two parts:
- **Network portion**: Identifies the network itself
- **Host portion**: Identifies individual devices on that network

**Traditional vs. CIDR Notation:**

The subnet mask 255.255.255.0 can be written in two ways:
- **Traditional**: 255.255.255.0
- **CIDR Notation**: /24

The /24 notation means the leftmost 24 bits are the network portion, leaving 8 bits for host addresses.

**Practical Example: 192.168.66.89/24 (or 255.255.255.0)**

- IP address: 192.168.66.89
- First three octets (network): 192.168.66 (24 bits)
- Last octet (host): 89
- Network address: 192.168.66.0
- Broadcast address: 192.168.66.255
- Usable host range: 192.168.66.1 to 192.168.66.254 (254 usable addresses)

**Common Subnet Masks:**

| CIDR | Subnet Mask | Usable Hosts | Network Size |
|------|------------|--------------|--------------|
| /8 | 255.0.0.0 | 16,777,214 | 16,777,216 |
| /16 | 255.255.0.0 | 65,534 | 65,536 |
| /24 | 255.255.255.0 | 254 | 256 |
| /30 | 255.255.255.252 | 2 | 4 |

### Private vs. Public IP Addresses

Before using an IP address, it is critical to understand whether it is private or public.

**Public IP Addresses:**
- Globally unique across the entire Internet
- Assigned by Internet Service Providers (ISPs)
- Routable on the public Internet
- Each public IP identifies one specific device globally

**Private IP Addresses:**
- Used only within local area networks (LANs)
- Not routable on the public Internet
- Can be freely reused in different private networks
- Used behind routers that perform Network Address Translation (NAT)

**RFC 1918 Private IP Address Ranges (MEMORIZE THESE):**

- 10.0.0.0 to 10.255.255.255 (10/8): Largest private range
- 172.16.0.0 to 172.31.255.255 (172.16/12): Medium private range
- 192.168.0.0 to 192.168.255.255 (192.168/16): Most common private range

**Critical Concept:** If you see an IP address like 10.1.33.7, 172.31.33.7, or 192.168.50.100, you cannot access it from the public Internet because these are private addresses. You can only access them from within their private network.

**Quick Reference Questions:**
- Which of the following is NOT a private IP address? 192.168.250.125, 10.20.141.132, 49.69.147.197, 172.23.182.251
  **Answer: 49.69.147.197 (starts with 49, outside all private ranges)**

- Which of the following is NOT a valid IP address? 192.168.250.15, 192.168.254.17, 192.168.305.19, 192.168.199.13
  **Answer: 192.168.305.19 (305 exceeds maximum octet value of 255)**

### Routing

**The Router's Role:**
A router functions like your local post office. You give it a package (packet), and it knows how to deliver it. The router examines the destination address and determines where to send it next.

**How Routers Work:**
- Routers operate at Layer 3 (Network Layer)
- They examine the destination IP address in each packet
- They consult their routing table to find the best path forward
- They forward the packet to the next router (next hop) or directly to the destination
- Each router moves the packet closer to its final destination

**Routing Table:**
Every router maintains a routing table containing:
- Known destination networks
- Which interface or next router to use for each destination
- Metric information (priority/cost of each route)

When a packet arrives, the router matches the destination IP against the routing table to decide where to forward it.

**Key Difference: Routers vs. Switches**
- **Switches** (Layer 2): Use MAC addresses to forward frames within a local network
- **Routers** (Layer 3): Use IP addresses to forward packets between different networks

## Transport Protocols and Port Numbers

Transport layer protocols enable processes on different hosts to communicate. Two main protocols exist: TCP and UDP.

### TCP (Transmission Control Protocol)

**Characteristics:**
- Reliable communication
- Guaranteed delivery of data
- Data arrives in order
- No data loss
- Slower due to reliability overhead

**How TCP Ensures Reliability:**
- Data is split into segments with sequence numbers
- The receiver sends acknowledgments (ACKs) confirming receipt
- If no ACK is received within a timeout period, the sender retransmits the segment
- Segments are reordered if they arrive out of sequence

**TCP Connection Establishment: Three-Way Handshake**

Before data can be transmitted, a TCP connection must be established:

1. **SYN Packet**: The client sends a SYN (Synchronize) packet to the server containing the client's randomly chosen initial sequence number. This requests to open a connection.

2. **SYN-ACK Packet**: The server responds with a SYN-ACK packet, acknowledging the client's sequence number and providing the server's own initial sequence number.

3. **ACK Packet**: The client sends an ACK (Acknowledgment) packet to confirm receipt of the server's sequence number. At this point, the connection is established and data transfer can begin.

After this three-way handshake completes, both sides confirm they are ready to communicate.

**TCP Connection Termination: Four-Way Handshake**

When closing a TCP connection:
1. Sender sends FIN (finish) packet
2. Receiver acknowledges with ACK
3. Receiver sends its own FIN packet
4. Sender acknowledges with ACK

**When to Use TCP:**
- Email (SMTP, POP3, IMAP)
- Web browsing (HTTP, HTTPS)
- File transfer (FTP, SFTP)
- Remote access (SSH)
- Any application requiring guaranteed delivery

**Quick Reference Question:**
- Which protocol requires a three-way handshake? **Answer: TCP**

### UDP (User Datagram Protocol)

**Characteristics:**
- Fast communication
- No reliability guarantees
- Data may not arrive
- Data may arrive out of order
- Faster due to minimal overhead
- Connectionless

**How UDP Works:**
- Application data is wrapped in a UDP header
- Datagrams are sent immediately without connection setup
- No acknowledgments are sent
- No retransmissions for lost datagrams
- Minimal overhead makes it very fast

**When to Use UDP:**
- Video streaming: Occasional frame loss is acceptable for smooth playback
- VoIP (Voice over IP): Low latency is more important than occasional voice loss
- Online gaming: Speed is critical for real-time interaction
- DNS queries: Can resend if no response received
- Network monitoring (SNMP): Lightweight protocol
- Live broadcasts: Retransmitting old data is pointless

**TCP vs. UDP Comparison:**

| Feature | TCP | UDP |
|---------|-----|-----|
| Reliability | Guaranteed delivery | No guarantees |
| Ordering | Maintains order | No ordering |
| Connection Setup | Requires handshake | Connectionless |
| Speed | Slower | Faster |
| Overhead | High | Low |
| Error Checking | Extensive | Minimal |
| Use Case | Accuracy critical | Speed critical |

### Port Numbers

Ports are logical endpoints that identify specific services or applications running on a device. They enable multiple applications to run simultaneously on a single device and receive their appropriate network traffic.

**Port Number Range:**
- Valid range: 1 to 65535
- Total of 2^16 - 1 ports (65,535 ports)
- Port 0 is reserved

**Port Number Categories:**

- **Well-Known Ports (0-1023)**: Reserved for standard services
  - Port 7: Echo (test service)
  - Port 13: Daytime (returns current date/time)
  - Port 21: FTP (File Transfer Protocol)
  - Port 22: SSH (Secure Shell)
  - Port 25: SMTP (Simple Mail Transfer Protocol)
  - Port 53: DNS (Domain Name System)
  - Port 80: HTTP (HyperText Transfer Protocol)
  - Port 443: HTTPS (HTTP Secure)
  - Port 3306: MySQL (Database)

- **Registered Ports (1024-49,151)**: Available for applications and services to register and use

- **Dynamic/Private Ports (49,152-65,535)**: Temporary ports used by client applications for outgoing connections

**Port Notation:**
When specifying both IP and port, use colon notation:
- 192.168.1.100:80 means IP 192.168.1.100 on port 80
- 10.10.63.37:443 means IP 10.10.63.37 on port 443

**Quick Reference Question:**
- What is the approximate number of port numbers (in thousands)? **Answer: 65**

## Data Encapsulation Process

Encapsulation refers to the process where data is wrapped with protocol-specific headers at each layer of the OSI or TCP/IP model. This hierarchical approach allows each layer to focus on its specific function without concerning itself with other layers.

**The Encapsulation Process (Top to Bottom):**

1. **Application Layer**: User generates data (e.g., typing an email or message)
2. **Transport Layer**: TCP or UDP adds headers with port numbers and sequence information, creating a segment or datagram
3. **Network Layer**: IP adds headers with source and destination IP addresses, creating a packet
4. **Data Link Layer**: Ethernet or WiFi adds headers and trailers with source and destination MAC addresses, creating a frame
5. **Physical Layer**: Frame is converted to bits and transmitted as electrical or radio signals

**PDU (Protocol Data Unit) Names by Layer:**

| Layer | PDU Name | Headers Added |
|-------|----------|---------------|
| Application (7) | Application Data | Format, content |
| Transport (4) | Segment (TCP) / Datagram (UDP) | Source port, destination port, sequence numbers |
| Network (3) | Packet (IP) | Source IP, destination IP, TTL |
| Data Link (2) | Frame (Ethernet/WiFi) | Source MAC, destination MAC |
| Physical (1) | Bits | Signal encoding |

**Practical Example: Sending an HTTP Request**

When you visit a website:
1. Application layer creates HTTP request data
2. Transport layer wraps it: TCP header with port 80, sequence numbers
3. Network layer wraps it: IP header with source and destination IPs
4. Data link layer wraps it: Ethernet or WiFi header with MAC addresses
5. Physical layer converts it: Electrical signals sent on the cable

**De-encapsulation on Receipt:**

The receiving host reverses this process, removing each layer's header and passing the data up to the next layer until the application receives the original data.

**Quick Reference Questions:**
- On a WiFi, within what will an IP packet be encapsulated? **Answer: Frame**
- What do you call the UDP data unit that encapsulates the application data? **Answer: Datagram**
- What do you call the data unit that encapsulates the application data sent over TCP? **Answer: Segment**

## Telnet and Network Testing

### Telnet Command

Telnet is a command-line utility used for remote communication over a network. It allows you to establish a connection to a specific IP address and port, making it useful for testing services and network connectivity.

**Basic Usage:**
```
telnet [IP_address] [port]
```

**Common Examples:**
- `telnet 10.10.63.37 7` - Connect to echo server on port 7
- `telnet 10.10.63.37 13` - Connect to daytime server on port 13
- `telnet 192.168.1.1 80` - Connect to web server on port 80
- `telnet scanme.nmap.org 22` - Connect to SSH service on port 22

**How to Exit Telnet:**
- Press CTRL + ] to access the telnet prompt
- Type `quit` at the telnet> prompt
- Press Enter

### Practical Telnet Examples

**Echo Server (Port 7):**

The echo server simply returns whatever you send to it. This is useful for testing basic connectivity.

```
user@machine$ telnet 10.10.63.37 7
Trying 10.10.63.37...
Connected to 10.10.63.37.
Escape character is '^]'.
Hi
Hi
How are you?
How are you?
Bye
Bye
^]
telnet> quit
Connection closed.
```

When you type "Hi", the server echoes back "Hi". This confirms the connection is working.

**Daytime Server (Port 13):**

The daytime server responds with the current date and time, then closes the connection.

```
user@machine$ telnet 10.10.63.37 13
Trying 10.10.63.37...
Connected to 10.10.63.37.
Escape character is '^]'.
Thu Jun 20 12:36:32 PM UTC 2024
Connection closed by foreign host.
```

The connection automatically closes after the server sends the date and time.

**Web Server (Port 80):**

To request a web page using telnet, connect to port 80 and issue an HTTP GET request.

```
user@machine$ telnet 10.10.63.37 80
Trying 10.10.63.37...
Connected to 10.10.63.37.
Escape character is '^]'.
GET / HTTP/1.1
Host: telnet.thm

HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234
[Web page content here...]
```

Key points:
- Type: `GET / HTTP/1.1`
- Then type: `Host: [hostname]` (use any hostname)
- Press Enter twice (blank line indicates end of headers)
- You may need to press Enter again to see the response

This demonstrates that HTTP is a text-based protocol running over TCP.

### Security Note

Telnet transmits all data in plain text, including passwords. It should never be used for anything sensitive. SSH is the modern, secure alternative for remote command-line access.

### Other Useful Troubleshooting Commands

**ping:**
- Test connectivity to a remote host
- Command: `ping [hostname or IP]`
- Sends ICMP echo requests and waits for responses

**tracert (Windows) / traceroute (Linux):**
- Shows the path packets take to reach a destination
- Displays each router (hop) along the way
- Useful for identifying where network issues occur

**netstat:**
- Displays active network connections and listening ports
- `netstat -an` shows all connections with IP addresses and ports
- `netstat -rn` displays the routing table
- Helps identify which services are listening on which ports

## Key Takeaways

1. The OSI model provides a seven-layer conceptual framework for understanding networks, while TCP/IP is a practical four-layer model for the modern Internet.

2. Each OSI layer has specific responsibilities and communicates with adjacent layers through standardized interfaces.

3. IP addresses uniquely identify devices on networks; private addresses (RFC 1918 ranges) are used internally and cannot route on the public Internet.

4. Subnetting divides networks using subnet masks or CIDR notation, allowing efficient IP address allocation and network organization.

5. Routers use IP addresses to forward packets between networks, determining the best path based on their routing tables.

6. TCP provides reliable, ordered communication through connection setup and acknowledgments; UDP provides fast, connectionless communication without reliability guarantees.

7. The three-way handshake (SYN, SYN-ACK, ACK) establishes TCP connections before data transmission begins.

8. Ports (0-65535) identify specific services and applications on hosts; well-known ports (0-1023) are reserved for standard services.

9. Encapsulation wraps data with headers at each layer; de-encapsulation removes headers at the destination.

10. Telnet and diagnostic commands (ipconfig, ping, traceroute, netstat) are essential tools for testing network connectivity and troubleshooting issues.
