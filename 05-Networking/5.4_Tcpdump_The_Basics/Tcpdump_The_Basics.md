# Networking — Tcpdump: The Basics

## Introduction

Tcpdump is a command-line packet capture tool that captures and displays network traffic in real-time. Unlike Wireshark's GUI, tcpdump operates from the terminal, making it ideal for remote systems, server environments, and scripted automation. It saves captures to PCAP files for later analysis.

⚠️ **Important**: Tcpdump requires root or elevated privileges to capture network traffic. Use with appropriate permissions and authorization.

---

## Installation & Prerequisites

### Linux/macOS
```bash
# Debian/Ubuntu
sudo apt-get install tcpdump

# macOS (Homebrew)
brew install tcpdump

# CentOS/RHEL
sudo yum install tcpdump
```

### Windows
- Install **Npcap** (packet capture library) or **WinPcap**
- Use **Nmap** package which includes tcpdump

### Verify Installation
```bash
tcpdump --version
```

---

## Basic Syntax

```bash
sudo tcpdump [OPTIONS] [FILTER]
```

| Component | Purpose |
|-----------|---------|
| **OPTIONS** | Control capture behavior (interface, output format, packet count, etc.) |
| **FILTER** | BPF (Berkeley Packet Filter) expression to specify which packets to capture |

---

## Core Options

| Option | Purpose | Example |
|--------|---------|---------|
| `-i <interface>` | Capture on specific network interface | `tcpdump -i eth0` |
| `-i any` | Capture on all interfaces | `tcpdump -i any` |
| `-c <count>` | Stop after capturing N packets | `tcpdump -c 100` |
| `-w <file>` | Write packets to PCAP file | `tcpdump -w capture.pcap` |
| `-r <file>` | Read packets from PCAP file | `tcpdump -r capture.pcap` |
| `-A` | Print packet data in ASCII | `tcpdump -A` |
| `-x` | Print packet data in hexadecimal | `tcpdump -x` |
| `-X` | Print packet data in hex and ASCII | `tcpdump -X` |
| `-v / -vv / -vvv` | Increase verbosity (1-3 levels) | `tcpdump -vv` |
| `-n` | Don't resolve hostnames (show IPs) | `tcpdump -n` |
| `-nn` | Don't resolve hostnames or port names | `tcpdump -nn` |
| `-q` | Quiet output (minimal detail) | `tcpdump -q` |
| `-l` | Line-buffered output (for piping) | `tcpdump -l` |
| `-D` | List available network interfaces | `tcpdump -D` |
| `-e` | Show MAC addresses (Link layer) | `tcpdump -e` |

---

## Network Interface Selection

### List Available Interfaces
```bash
tcpdump -D
```

**Output Example**:
```
1. eth0 (Ethernet)
2. lo (Loopback)
3. docker0 (Docker)
4. wlan0 (Wireless)
```

### Capture on Specific Interface
```bash
sudo tcpdump -i eth0
```

### Capture on All Interfaces
```bash
sudo tcpdump -i any
```

---

## Basic Capture Examples

### Capture First 10 Packets
```bash
sudo tcpdump -c 10
```

### Capture and Save to File
```bash
sudo tcpdump -i eth0 -w mycapture.pcap
```

### Capture with Verbose Output
```bash
sudo tcpdump -i eth0 -vv
```

### Capture with Hex & ASCII Display
```bash
sudo tcpdump -i eth0 -X
```

### Read and Display Saved Capture
```bash
tcpdump -r mycapture.pcap
```

---

## Display Formats

### Default Output
```
timestamp source > destination: protocol info
```

**Example**:
```
14:23:45.123456 192.168.1.100.52341 > 8.8.8.8.53: 12345+ A google.com. (32)
```

### Verbose Output (-vv)
Shows detailed protocol headers, flags, and sequence numbers.

### Quiet Output (-q)
Shows minimal information (protocol and flow direction only).

---

## Berkeley Packet Filter (BPF) Syntax

### Protocol Filters

| Filter | Purpose |
|--------|---------|
| `tcp` | Only TCP packets |
| `udp` | Only UDP packets |
| `icmp` | Only ICMP packets (ping) |
| `arp` | Only ARP packets |
| `ip` | All IPv4 packets |
| `ipv6` | All IPv6 packets |
| `http` | HTTP traffic (port 80) |
| `dns` | DNS traffic (port 53) |

### Host Filters

| Filter | Purpose |
|--------|---------|
| `host 192.168.1.1` | Traffic to/from specific IP |
| `src host 192.168.1.1` | Traffic from specific source |
| `dst host 192.168.1.1` | Traffic to specific destination |
| `net 192.168.1.0/24` | Traffic to/from subnet |

### Port Filters

| Filter | Purpose |
|--------|---------|
| `port 80` | Traffic on port 80 (either direction) |
| `src port 443` | Traffic from port 443 |
| `dst port 22` | Traffic to port 22 (SSH) |
| `portrange 8000-9000` | Traffic on ports 8000-9000 |

### Combining Filters (Boolean Logic)

| Operator | Meaning |
|----------|---------|
| `and` or `&&` | Both conditions must match |
| `or` or `\|\|` | Either condition can match |
| `not` or `!` | Negate the condition |

---

## Common Filter Examples

```bash
# Capture all TCP traffic
sudo tcpdump -i eth0 tcp

# Capture DNS queries only
sudo tcpdump -i eth0 -nn udp port 53

# Capture SSH traffic
sudo tcpdump -i eth0 tcp port 22

# Capture traffic to/from specific IP
sudo tcpdump -i eth0 host 192.168.1.100

# Capture HTTP (port 80)
sudo tcpdump -i eth0 tcp port 80

# Capture HTTPS (port 443)
sudo tcpdump -i eth0 tcp port 443

# Capture traffic except SSH
sudo tcpdump -i eth0 "tcp and not port 22"

# Capture traffic on subnet
sudo tcpdump -i eth0 net 192.168.1.0/24

# Capture ARP traffic
sudo tcpdump -i eth0 arp

# Capture ICMP (ping)
sudo tcpdump -i eth0 icmp
```

---

## Advanced Capture Techniques

### Capture with Packet Limit and Save
```bash
sudo tcpdump -i eth0 -c 1000 -w capture.pcap tcp port 80
```

### Capture and Rotate Files (by Size)
```bash
sudo tcpdump -i eth0 -w capture.pcap -C 10 -W 5 tcp
# Rotate capture files when they reach 10 MB, keep 5 files
```

### Capture Specific Bytes (Payload)
```bash
sudo tcpdump -i eth0 -A dst port 80
# Show ASCII payload for HTTP traffic
```

### Capture with Timestamp Precision
```bash
sudo tcpdump -i eth0 -tttt
# Show human-readable timestamps
```

### Capture and Pipe to Wireshark
```bash
sudo tcpdump -i eth0 -w - | wireshark -k -i -
# Real-time analysis in Wireshark
```

---

## Saving & Reading Captures

### Save to PCAP File
```bash
sudo tcpdump -i eth0 -w capture.pcap
```

### Read from PCAP File
```bash
tcpdump -r capture.pcap
```

### Read with Verbose Output
```bash
tcpdump -r capture.pcap -vv
```

### Read with Specific Filter
```bash
tcpdump -r capture.pcap tcp port 443
```

### Convert PCAP for Analysis
```bash
# Extract HTTP traffic only
tcpdump -r capture.pcap -w http_only.pcap tcp port 80
```

---

## Analyzing Capture Output

### Understanding Default Output

```
14:23:45.123456 192.168.1.100.52341 > 8.8.8.8.53: 12345+ A google.com. (32)
```

| Element | Meaning |
|---------|---------|
| `14:23:45.123456` | Timestamp |
| `192.168.1.100.52341` | Source IP and port |
| `>` | Direction (to) |
| `8.8.8.8.53` | Destination IP and port |
| `12345+` | DNS transaction ID + recursion flag |
| `A` | Query type (A record = IPv4) |
| `google.com.` | Domain being queried |
| `(32)` | Packet size in bytes |

### TCP Flags in Output

| Flag | Meaning |
|------|---------|
| `S` | SYN (connection initiation) |
| `F` | FIN (connection termination) |
| `A` | ACK (acknowledgment) |
| `R` | RST (reset) |
| `P` | PUSH (data available) |
| `W` | Window (flow control) |

**Example**: `S` = SYN packet, `[S.]` = SYN+ACK

---

## Real-World Scenarios

### Monitor All Traffic on Interface
```bash
sudo tcpdump -i eth0 -vv
```

### Capture Suspicious Outbound Traffic
```bash
sudo tcpdump -i eth0 -w suspect.pcap "tcp and dst net !192.168.1.0/24"
# Save non-local traffic
```

### Monitor DNS Queries
```bash
sudo tcpdump -i eth0 -nn "udp port 53" -A
# Show DNS queries with domain names
```

### Capture Traffic to Specific Host
```bash
sudo tcpdump -i eth0 -w host_traffic.pcap host 8.8.8.8
```

### Find TCP Retransmissions
```bash
tcpdump -r capture.pcap "tcp.analysis.retransmission"
# (Note: requires reading into Wireshark or specialized tool)
```

---

## Best Practices

1. **Use Root Privileges**: Always use `sudo` for packet capture
2. **Specify Interface**: Always specify `-i` to avoid confusion
3. **Don't Resolve Names**: Use `-n` on busy networks (faster capture)
4. **Save Captures**: Use `-w` for later analysis in Wireshark
5. **Use Filters**: Reduce noise with precise BPF filters
6. **Monitor Size**: Use `-C` to rotate files and prevent disk filling
7. **Respect Privacy**: Only capture traffic you're authorized to analyze
8. **Verify Packets**: Use `-vv` to confirm filter accuracy before long captures

---

## Key Takeaways

1. Tcpdump is a lightweight, powerful CLI tool for packet capture.
2. Use filters (BPF syntax) to capture only relevant traffic.
3. Save captures to PCAP files with `-w` for later analysis.
4. Combine with Wireshark for detailed GUI analysis.
5. Master boolean operators (and, or, not) for complex filters.
6. Always use appropriate privileges and authorization.
7. Use `-n` and `-nn` to speed up capture on busy networks.
