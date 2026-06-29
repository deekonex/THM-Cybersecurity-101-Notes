# Networking — Network Traffic Analysis

## Introduction

Network Traffic Analysis (NTA) is the practice of capturing, examining, and interpreting data packets flowing across a network to identify patterns, anomalies, and security threats. It builds on packet analysis fundamentals to understand network behavior and detect indicators of compromise (IOCs).

⚠️ **Important**: NTA requires context and experience. False positives are common; verify findings against baseline and normal traffic patterns.

---

## Analysis Objectives

| Objective | Purpose |
|-----------|---------|
| **Baseline Establishment** | Document normal traffic patterns for comparison |
| **Anomaly Detection** | Identify deviations from established baselines |
| **Threat Hunting** | Proactively search for signs of compromise or malicious activity |
| **Incident Response** | Analyze captured traffic to understand attack scope and methods |
| **Performance Troubleshooting** | Identify network bottlenecks and inefficient communication patterns |
| **Protocol Compliance** | Verify protocols are used correctly and securely |

---

## Traffic Patterns & Baselines

### Normal vs. Anomalous Traffic

| Characteristic | Normal | Anomalous |
|---|---|---|
| **Data Volume** | Consistent, predictable peaks/troughs | Sudden spikes, unusual nighttime activity |
| **Protocols** | Expected services (HTTP, DNS, SMTP) | Unusual or rare protocols |
| **Destinations** | Known internal/partner networks | External IPs never seen before |
| **Packet Size** | Consistent with protocol norms | Oversized packets or unusual fragmentation |
| **Flow Duration** | Typical connection lengths | Unusually long or short sessions |
| **Port Usage** | Services on expected ports | Services on non-standard ports |

### Establishing Baselines

1. **Capture during normal operations** (24-48 hours minimum)
2. **Document protocol distribution** (% HTTP, DNS, SMTP, etc.)
3. **Note expected external communication** (CDNs, APIs, partners)
4. **Record typical user behavior patterns** (peak hours, file transfer volumes)
5. **Identify regular automated tasks** (backups, log rotation, scheduled jobs)

---

## Common Anomalies to Investigate

### Suspicious Outbound Traffic

- **Beaconing**: Regular connections to external IPs at fixed intervals (C2 communication)
- **Data Exfiltration**: Unusual volume of outbound traffic to external networks
- **Tunneling**: Encapsulation of protocols within unexpected carriers (e.g., HTTP tunneling)
- **DNS Queries**: Unusual domains, DNS tunneling, or excessive query volumes

### Suspicious Inbound Traffic

- **Port Scanning**: Multiple connection attempts to different ports from single source
- **Exploitation Attempts**: Malformed packets or protocol violations targeting services
- **Brute Force**: Repeated connection attempts to authentication services (SSH, RDP, SMTP)

### Protocol Anomalies

- **Fragmentation**: Excessive packet fragmentation (may indicate evasion attempts)
- **TTL Anomalies**: TTL values inconsistent with expected routing
- **Checksum Errors**: Invalid checksums (protocol implementation issues or tampering)
- **TCP Flags**: Unusual flag combinations (e.g., SYN+FIN, ACK with no prior SYN)

---

## Analysis Techniques

### Follow-the-Stream Analysis

1. Identify suspicious packet in **Packet List**
2. Use **Analyse → Follow Stream** to view complete conversation
3. Look for:
   - Command/control communications
   - Credentials transmitted in plaintext
   - Unusual payloads or binary data
   - Protocol violations

### Protocol-Level Analysis

| Protocol | What to Check | Suspicious Indicators |
|----------|---------------|----------------------|
| **HTTP** | Methods, paths, headers, User-Agent | Unusual User-Agents, POST to suspicious URLs, unexpected status codes |
| **DNS** | Queries, response codes, TTL values | Typosquatting, DNS tunneling, NXDOMAIN responses |
| **SMTP** | Sender, recipient, attachment types | Unusual recipients, executable attachments, forwarding loops |
| **SSL/TLS** | Certificate details, handshake | Self-signed certs, cert mismatches, downgrade attempts |
| **FTP** | Credentials, transferred files | Plaintext credentials, suspicious filenames, unusual transfer patterns |

### Statistical Analysis

- **Packet Size Distribution**: Detect anomalous packet sizes (e.g., oversized DNS responses indicating data exfiltration)
- **Inter-Arrival Times**: Identify regular beaconing or irregular communication patterns
- **Port Usage**: Document services running on expected vs. unexpected ports
- **Connection Duration**: Flag unusually long or short sessions

---

## Identifying Lateral Movement

### Intra-Network Communication

- Monitor traffic between internal subnets (often overlooked)
- Look for:
  - Unusual host-to-host connections
  - Scanning activity from compromised hosts
  - Services on unexpected internal IPs
  - Mass authentication attempts across multiple systems

### Red Flags

- Source IP sending SYN packets to many different destinations (reconnaissance)
- RDP traffic between non-management systems
- Kerberos errors from unusual sources
- SMB traffic to non-fileserver systems

---

## Detecting Data Exfiltration

### Volume-Based Detection

- **Outbound vs. Inbound**: Compare upload/download ratios
- **Threshold Analysis**: Flag traffic exceeding historical averages
- **Protocol Analysis**: Identify protocols used for data movement (HTTP PUT, FTP, DNS, ICMP)

### Content-Based Indicators

- Compressed files transferred externally (unusual patterns)
- Archive formats (.zip, .rar, .tar.gz) sent to external IPs
- Database dumps or configuration files in outbound traffic
- Repeated connections to cloud storage services

### Time-Based Patterns

- Exfiltration during off-hours (avoiding detection)
- Regular intervals suggesting automated data stealing
- Correlation with known off-hours backups or maintenance

---

## Filtering Techniques for Analysis

### Protocol-Specific Filters

```
tcp.port == 22          # SSH connections
ip.src == 192.168.1.100 # All traffic from host
dns.qry.name contains "suspicious"  # DNS queries
http.request.method == "POST"       # HTTP POST requests
ssl.handshake.type == 1  # TLS ClientHello packets
```

### Combine Filters for Precision

```
ip.dst == 8.8.8.8 and tcp.port == 443  # HTTPS to Google DNS
tcp.flags.syn == 1 and tcp.flags.ack == 0  # TCP SYN (connection initiation)
dns.resp.type == 1 and dns.a == 0.0.0.0  # DNS A records pointing to 0.0.0.0
```

---

## Tools & Integration

### Complementary Tools

- **Zeek/Bro**: Automated traffic analysis and signature detection
- **Suricata**: IDS/IPS with rule-based detection
- **Arkime**: Full-packet capture and search
- **yara**: Pattern matching for malicious signatures
- **Statistics**: Wireshark built-in protocol hierarchy, conversations, endpoints

### Export Data for Further Analysis

- **Export packets** to PCAP for external tools
- **Export objects** to examine transferred files
- **Generate statistics** for reporting and correlation

---

## Practical Workflow

### Step 1: Capture Baseline
1. Record 24-48 hours of normal traffic
2. Document protocol distribution via **Statistics → Protocol Hierarchy**
3. Note typical data volumes and communication patterns

### Step 2: Implement Monitoring
1. Set up ongoing packet capture (with retention policy)
2. Generate daily/weekly traffic summaries
3. Compare against baseline for anomalies

### Step 3: Investigate Anomalies
1. Apply filters to isolate suspicious traffic
2. Use **Follow Stream** to examine conversations
3. Cross-reference IPs and domains against threat feeds
4. Check Expert Information for protocol errors

### Step 4: Document Findings
1. Capture relevant packets with comments
2. Export suspicious streams for further analysis
3. Generate report with timeline and IOCs (Indicators of Compromise)

---

## Best Practices

1. **Know Your Baseline**: Cannot identify anomalies without understanding normal traffic
2. **Use Multiple Perspectives**: Analyze by protocol, host, port, and time
3. **Correlate with Logs**: Match network traffic with endpoint and application logs
4. **Minimize Noise**: Focus on protocols and destinations relevant to your environment
5. **Document Assumptions**: Record what you consider "normal" for context and auditability
6. **Preserve Evidence**: Archive unusual captures with metadata and context
7. **Stay Updated**: Keep threat feed knowledge current (known C2 IPs, malicious domains)

---

## Key Takeaways

1. Network Traffic Analysis requires baseline knowledge and context.
2. Establish normal traffic patterns before hunting for anomalies.
3. Use display filters to focus on relevant protocols and flows.
4. Follow-the-Stream analysis reveals intent and payload of suspicious conversations.
5. Lateral movement, exfiltration, and beaconing are key indicators to monitor.
6. Combine automated tools with manual inspection for comprehensive analysis.
7. Document findings thoroughly for incident response and future correlation.
