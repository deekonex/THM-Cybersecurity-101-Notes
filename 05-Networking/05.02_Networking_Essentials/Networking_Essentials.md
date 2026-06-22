# Networking Essentials

## Learning Objectives

By the end of this module, you will understand:
- Core networking protocols (DHCP, ARP, ICMP, NAT)
- How IP addresses are dynamically assigned to devices
- How Layer 2 and Layer 3 address translation works
- Network diagnostics and troubleshooting tools
- Routing protocols and path determination
- Practical command-line tools for network analysis (tshark, tcpdump, ping, traceroute)

---

## DHCP (Dynamic Host Configuration Protocol)

### Purpose

DHCP automates network configuration by dynamically assigning IP addresses and other network parameters to devices when they connect to a network.

**What DHCP assigns:**
- **IP Address**: Unique address for the device
- **Subnet Mask**: Defines the network and host portions
- **Default Gateway**: Router's IP address for accessing other networks
- **DNS Servers**: Addresses of DNS servers for domain name resolution
- **Lease Time**: Duration the IP address is valid (typically 24 hours - 7 days)

### Benefits

- **Eliminates manual configuration**: Devices automatically obtain network settings
- **Prevents IP conflicts**: Central server tracks all assigned addresses
- **Essential for mobile/portable devices**: Devices can move between networks seamlessly
- **Reduces administrative overhead**: No need to manually configure each device

### The DORA Process

DHCP uses a four-step discovery and assignment process:

| Step | Message | Direction | Description |
|------|---------|-----------|-------------|
| 1 | Discover | Client → Broadcast (255.255.255.255:67) | Client broadcasts looking for any DHCP server |
| 2 | Offer | Server → Client | Server offers available IP address and lease terms |
| 3 | Request | Client → Broadcast | Client accepts the offered IP (broadcasts to inform all servers) |
| 4 | Acknowledge | Server → Client | Server confirms IP assignment and sends final parameters |

**Key Points:**
- The Discover and Request messages use broadcast (255.255.255.255) so all servers/devices hear them
- The Offer and Acknowledge use unicast back to the client
- DHCP operates on UDP port 67 (server) and 68 (client)
- The server reserves the offered IP until acceptance or timeout

### Packet Capture Example

```
1. DHCP Discover   - Source: 0.0.0.0:68 → Dest: 255.255.255.255:67
                     "Does anyone have an IP address for me?"

2. DHCP Offer      - Source: 192.168.66.1:67 → Dest: 192.168.66.133:68
                     "I have 192.168.66.133 available with /24 subnet"

3. DHCP Request    - Source: 0.0.0.0:68 → Dest: 255.255.255.255:67
                     "I want the IP from server 192.168.66.1"

4. DHCP ACK        - Source: 192.168.66.1:67 → Dest: 192.168.66.133:68
                     "Confirmed. Use 192.168.66.133, lease expires in 24 hours"
```

### DHCP Lease Renewal

When a DHCP lease approaches expiration:
- **At 50% of lease time**: Client sends DHCP Request directly to the original server
- **At 87.5% of lease time**: If no response, client broadcasts a new DHCP Request
- **If renewed**: Lease timer resets; process repeats
- **If not renewed**: Address becomes available for reassignment

### DHCP Relay Agents

In large networks, DHCP relay agents (configured on routers) forward DHCP broadcasts across network segments, allowing one DHCP server to serve multiple subnets without needing a server on each segment.

---

## ARP (Address Resolution Protocol)

### Purpose

ARP maps **Layer 3 (IP) addresses to Layer 2 (MAC) addresses** on local network segments. It's essential because Ethernet frames require MAC addresses for local delivery, but applications work with IP addresses.

**Why needed:** 
- IP addresses route globally, but frames only travel locally
- To deliver a frame on the local network, the sender must know the destination's MAC address
- ARP resolves the unknown MAC address given an IP address

### ARP Process

#### ARP Request (Broadcast)
The sender doesn't know the target's MAC address, so it broadcasts an ARP request to everyone:
- **Source MAC**: Sender's MAC address
- **Dest MAC**: ff:ff:ff:ff:ff:ff (broadcast)
- **Message**: "Who has IP X? Tell me at MAC Y"
- All devices on the segment receive this frame

#### ARP Reply (Unicast)
Only the device with the target IP responds:
- **Source MAC**: Target device's MAC
- **Dest MAC**: Requesting device's MAC (direct reply, not broadcast)
- **Message**: "IP X is at MAC Z"

### Packet Example

```
ARP Request (broadcast):
  From MAC: 00:11:22:33:44:55
  To MAC:   ff:ff:ff:ff:ff:ff
  Message: "Who has 192.168.66.1? Tell 192.168.66.89"

ARP Reply (unicast):
  From MAC: 44:df:65:d8:fe:6c
  To MAC:   00:11:22:33:44:55
  Message: "192.168.66.1 is at 44:df:65:d8:fe:6c"
```

### Key Characteristics

- **Layer 2/3 hybrid**: Encapsulated directly in Ethernet frames (Layer 2), not inside IP packets
- **Local network only**: ARP only works on the same subnet; routers don't forward ARP requests
- **Non-routable**: Unlike IP, MAC addresses have no meaning beyond the local segment
- **Dynamic**: ARP results are cached locally but expire (typically 15-20 minutes)

### ARP Cache

Every device maintains an ARP cache:
```bash
# View ARP cache on Linux
arp -a
# or
ip neighbor show

# View ARP cache on Windows
arp -a
```

The cache maps recently discovered IP↔MAC pairs, reducing ARP broadcast traffic.

### ARP Vulnerabilities

**ARP Spoofing**: Attacker sends unsolicited ARP replies claiming to have another device's IP, causing traffic to be redirected (man-in-the-middle attacks).

**Prevention**: Gratuitous ARP and ARP-based monitoring/detection tools, though no built-in authentication in ARP.

---

## ICMP (Internet Control Message Protocol)

### Purpose

ICMP provides **network diagnostics and error reporting**. It is NOT used for data transfer but for control and informational messages.

**Key differences from TCP/UDP:**
- Operates at Layer 3 (Network layer)
- Connectionless and stateless
- Messages are sent directly without prior connection setup
- Used by network management and troubleshooting tools

### Ping (Echo Request/Reply)

Ping tests connectivity and measures round-trip time (latency):

| Type | Description | Purpose |
|------|-------------|---------|
| ICMP Type 8 | Echo Request | Sender requests a response |
| ICMP Type 0 | Echo Reply | Target responds to echo request |

**How it works:**
1. Source sends ICMP Echo Request (Type 8) with a sequence number and timestamp
2. Destination receives it and immediately sends Echo Reply (Type 0) with same sequence number
3. Source calculates RTT = (current time) - (timestamp from packet)

**Common command:**
```bash
ping <target> -c <count>
ping 8.8.8.8 -c 4    # Send 4 ping packets to Google DNS
```

**Output interpretation:**
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=119 time=20.5 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=119 time=19.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=119 time=21.2 ms
```

**Common reasons for ping failure:**
- Target is offline or unreachable
- Firewall is blocking ICMP (many firewalls drop echo requests)
- Network path is broken (routing issue)
- TTL expired before reaching target

### Traceroute (Path Trace)

Traceroute shows the **path packets take through the network** by displaying each router hop.

**How it works:**
1. Uses TTL (Time-To-Live) field in IP header
2. Sends packets with increasing TTL values (1, 2, 3, ...)
3. Each router decrements TTL by 1
4. When TTL reaches 0, router sends ICMP Time Exceeded (Type 11)
5. Traceroute collects responses to map the path

| ICMP Type | Message | When |
|-----------|---------|------|
| Type 11 | Time Exceeded | TTL reaches 0 at intermediate router |
| Type 0 | Echo Reply | Final destination reached |

**Common command:**
```bash
traceroute <target>
traceroute google.com
```

**Output example:**
```
 1  192.168.1.1 (192.168.1.1)  1.234 ms
 2  10.0.0.1 (10.0.0.1)  5.432 ms
 3  203.0.113.1 (203.0.113.1)  12.345 ms
 4  209.85.233.1 (209.85.233.1)  20.123 ms
```

**Interpretation:**
- Each line is one hop (router)
- First hop is usually the default gateway
- Times shown are RTT in milliseconds
- Can identify where network issues occur

**Windows variant:**
```bash
tracert <target>    # Windows uses tracert instead of traceroute
```

### Other ICMP Types

| Type | Name | Purpose |
|------|------|---------|
| 3 | Destination Unreachable | Host/network unreachable, port closed |
| 5 | Redirect | Router tells sender to use different route |
| 8 | Echo Request | Ping request |
| 0 | Echo Reply | Ping response |
| 11 | Time Exceeded | TTL expired or fragment reassembly timeout |

---

## NAT (Network Address Translation)

### Purpose

NAT allows **multiple private IP addresses to share a single public IP address** for internet access. It translates addresses between private and public address spaces.

**Benefits:**
- **Conserves IPv4 addresses**: Millions of devices can share one public IP
- **Provides security**: Internal network structure is hidden from the internet
- **Enables private networks**: RFC 1918 addresses only work locally without NAT

### How NAT Works

1. **Outgoing traffic**: Router intercepts packets with private source IP
   - Replaces private source IP:port with public IP:new_port
   - Updates source port to one from the router's available pool
   - Maintains a translation table mapping old↔new addresses

2. **Return traffic**: Router sees incoming packets on public IP
   - Looks up destination port in translation table
   - Replaces public destination with original private IP:port
   - Forwards to internal device

3. **Translation table**: Router maintains entries like:
   ```
   Private IP:Port  ←→  Public IP:Port
   192.168.0.129:15401  212.3.4.5:19273
   192.168.0.130:8080   212.3.4.5:19274
   ```

### Types of NAT

| Type | How It Works | Use Case |
|------|--------------|----------|
| **Source NAT (SNAT)** | Translates source address of outgoing packets | Default for internal hosts accessing internet |
| **Destination NAT (DNAT)** | Translates destination address of incoming packets | Port forwarding to internal servers |
| **Static NAT** | One-to-one mapping (public IP ↔ private IP) | Hosting servers visible from internet |
| **Dynamic NAT** | Multiple private IPs share one/few public IPs | Typical home/corporate router |
| **PAT (Port Address Translation)** | Maps private IP:port to public IP:different_port | Most common in home routers |

### Port Capacity

A NAT router can support approximately **65,000 simultaneous TCP connections**:
- 16-bit port field = 2^16 = 65,536 possible port numbers
- Typical range for NAT: 1024-65535 = 64,512 ports
- Each connection uses one port mapping

**Practical implication**: Home routers rarely hit this limit; servers behind NAT are the bottleneck.

### NAT Limitations

- **Breaks peer-to-peer**: Devices behind different NATs can't directly connect without protocols like UPnP or manual port forwarding
- **Slows connections**: Every packet must be rewritten (minimal overhead but still processing)
- **IPv6 eliminates need**: IPv6 has enough addresses so NAT is unnecessary
- **Stateful**: NAT router must maintain state, vulnerable to state exhaustion attacks

---

## Routing Protocols

### Purpose

Routing protocols help routers **determine optimal paths** for packet forwarding. They share information about known networks and best routes.

### Routing Categories

#### **Distance-Vector Protocols**
Routers share their entire routing table with neighbors; each router independently calculates best path based on metric.

| Protocol | Metric | Convergence | Scale | Status |
|----------|--------|-------------|-------|--------|
| **RIP** | Hop count | Slow (15 min) | Small networks | Obsolete |
| **EIGRP** | Bandwidth, latency, reliability, load | Fast | Medium-large | Modern (Cisco proprietary) |

**How it works:**
- Router A tells Router B: "I can reach network 10.0.0.0/24 in 2 hops"
- Router B: "I'll reach it via Router A (3 hops total)"
- Metric: Number of router hops to destination

**Problem (RIP):** 
- Takes ~15 minutes to detect downed links (slow convergence)
- Poor scaling for large networks (max 15 hops)
- Simple but inefficient

#### **Link-State Protocols**
Routers build complete topology map; each router independently calculates shortest path using Dijkstra's algorithm.

| Protocol | Metric | Convergence | Scale | Status |
|----------|--------|-------------|-------|--------|
| **OSPF** | Cost (bandwidth-based) | Fast (seconds) | Large networks | Industry standard |
| **IS-IS** | Cost | Fast | Large networks | Carrier grade |

**How it works:**
- Each router learns full network topology
- Routers advertise link states (available neighbors and costs)
- Each router calculates shortest path independently
- Faster convergence than distance-vector

#### **Path-Vector Protocols**
Used between autonomous systems (ISPs); shares policy and reachability information.

| Protocol | Metric | Convergence | Scale | Status |
|----------|--------|-------------|-------|--------|
| **BGP** | Policy, AS-path length | Slow (can be minutes) | Internet backbone | Internet routing |

**How it works:**
- Each autonomous system advertises: "I can reach network X through path A→B→C"
- Neighbors evaluate paths based on configured policies
- Scales to entire Internet

---

## Network Diagnostics Tools

### tshark (Terminal Shark - Wireshark CLI)

Powerful command-line packet capture and analysis tool.

```bash
# Capture packets to a file
tshark -w capture.pcap -i eth0

# Read existing capture file
tshark -r capture.pcap -n

# Display specific protocol
tshark -r capture.pcap -Y "ip.src==192.168.1.1"

# Show packet details
tshark -r capture.pcap -V
```

**Common options:**
- `-r`: Read from file
- `-w`: Write to file
- `-n`: No DNS resolution (faster)
- `-Y`: Filter with Wireshark display filters
- `-V`: Verbose output showing all layers
- `-i`: Capture interface

### tcpdump

Lightweight packet capture tool, often available on minimal systems.

```bash
# Simple packet capture
tcpdump -i eth0

# Read from file
tcpdump -r capture.pcap -n -v

# Capture and save
tcpdump -w capture.pcap -i eth0

# Filter by protocol
tcpdump tcp -r capture.pcap

# Filter by host
tcpdump host 192.168.1.1 -r capture.pcap
```

**Common options:**
- `-i`: Interface to capture on
- `-r`: Read from file
- `-w`: Write to file
- `-n`: No DNS lookups
- `-v`: Verbose output (-vv for more detail)
- `-A`: Print payload in ASCII
- `-X`: Print payload in hex and ASCII

### ping

Test basic connectivity and measure latency.

```bash
ping <target> -c <count>          # Linux/Mac
ping <target> -n <count>          # Windows
ping 8.8.8.8 -c 4                 # Send 4 pings
```

**Output interpretation:**
```
PING example.com (93.184.216.34) 56(84) bytes of data.
64 bytes from 93.184.216.34: icmp_seq=1 ttl=56 time=45.2 ms
```

- **64 bytes**: Ping payload size
- **icmp_seq**: Sequence number (detects lost packets)
- **ttl**: Time-To-Live remaining
- **time**: Round-trip time in milliseconds

### traceroute / tracert

Show path to destination through intermediate routers.

```bash
traceroute <target>               # Linux/Mac
tracert <target>                  # Windows
traceroute google.com -m 20       # Max 20 hops
```

**Output shows:** Each hop's IP address and latency (usually 3 measurements per hop).

### netstat

Display network connections, listening ports, and routing information.

```bash
netstat -an                       # All connections with IPs (not DNS names)
netstat -rn                       # Routing table
netstat -ln                       # Listening ports only
```

**Interpreting netstat output:**
```
Proto Local Address     Foreign Address    State
tcp   0.0.0.0:22       0.0.0.0:*         LISTEN
tcp   192.168.1.5:45023 8.8.8.8:443       ESTABLISHED
```

---

## Practical Examples

### Example 1: Diagnosing Connection Issue

**Problem:** Can't reach 10.0.0.50

**Troubleshooting steps:**
```bash
# 1. Check if target is online
ping 10.0.0.50
# Result: No response

# 2. Trace path to target
traceroute 10.0.0.50
# Shows where packets stop responding

# 3. Check local ARP
arp -a | grep 10.0.0
# See if target is on local network

# 4. Capture packets to see what's happening
tcpdump -w debug.pcap host 10.0.0.50
# Analyze with tshark or Wireshark
```

### Example 2: Finding Service on Network

**Problem:** Need to find which server is running on port 443

```bash
# 1. Scan for listening services
netstat -ln | grep 443
# Shows if any service is listening locally

# 2. Try connecting
telnet 10.0.0.50 443
# Test if port is open and responsive

# 3. Capture and inspect traffic
tcpdump -w ssl.pcap -i eth0 port 443
# Analyze captured packets
```

---

## Key Takeaways

1. **DHCP** automatically assigns network configuration (IP, gateway, DNS) to devices when they join a network through a four-step DORA process.

2. **ARP** maps IP addresses to MAC addresses on local networks, enabling frame delivery at Layer 2. ARP requests are broadcasts; replies are unicasts.

3. **ICMP** provides diagnostics through:
   - **Ping (Echo Request/Reply)**: Tests connectivity and measures latency
   - **Traceroute (Time Exceeded)**: Maps path through routers using increasing TTL

4. **NAT** translates private IPs to public IPs, allowing multiple devices to share one public address. Common in home/corporate networks.

5. **Routing protocols** determine optimal paths:
   - **Distance-vector (RIP, EIGRP)**: Share routing tables
   - **Link-state (OSPF)**: Build complete topology
   - **Path-vector (BGP)**: Internet backbone routing

6. **Diagnostic tools** are essential for troubleshooting:
   - `ping` and `traceroute` for path testing
   - `tcpdump`/`tshark` for packet capture and analysis
   - `netstat` for connection and routing information
   - `arp` for viewing MAC address mappings

7. Each protocol operates at a specific layer: DHCP/DNS (Application), ICMP/NAT/Routing (Network), ARP (Data Link/Network).

8. Understanding these protocols is fundamental for network troubleshooting, security analysis, and system administration.
