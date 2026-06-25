# Networking — Wireshark: The Basics

## Introduction

Wireshark is a free, open-source packet analyser for capturing and dissecting live network traffic or reading packet captures (PCAP files). It provides detailed visibility into network communications at the byte level, making it essential for troubleshooting, learning protocol internals, and detecting security anomalies.

⚠️ **Important**: Wireshark is an analysis tool, not an intrusion detection system. Detection and interpretation depend entirely on the analyst's skill.

---

## Core GUI Sections

| Section | Purpose |
|---------|---------|
| **Packet List** | Displays summary of all captured packets (No., Time, Source, Destination, Protocol, Length, Info) |
| **Packet Details** | Hierarchical breakdown of selected packet across OSI layers |
| **Packet Bytes** | Raw hexadecimal and ASCII representation of packet data |
| **Display Filter Bar** | Text field for entering filter expressions |
| **Status Bar** | Shows total packets captured, displayed count, capture interface, and profile name |

---

## Working with Packet Captures

### Opening Files

- **File → Open**: Browse and open a PCAP file
- **Drag-and-drop**: Drop PCAP directly onto Wireshark window
- **Double-click**: Open PCAP files associated with Wireshark

### Merging Captures

- **File → Merge**: Combine two or more PCAP files
- Creates a new combined file (save before working to avoid losing data)

### File Properties

Access via **Statistics → Capture File Properties** or clicking the PCAP icon on the status bar:

- File hash (SHA256, MD5)
- Capture duration and start time
- Capture interface and filter used
- Total packet count and file size

---

## Packet Colourization

### Visual Organization

Packets are colour-coded by protocol or custom rules for quick visual scanning.

- **Permanent Rules**: Saved in profile via **View → Coloring Rules**
- **Temporary Colourization**: Right-click packet → **Colourise Conversation** (highlights related packets without filtering)
- **Toggle On/Off**: **View → Colourise Packet List**
- **Reset Colourization**: **View → Colourise Conversation → Reset Colourisation**

---

## Packet Dissection (OSI Layers)

Click on a packet in the **Packet List** to expand layers in the **Packet Details** pane. Each layer reveals protocol-specific information:

| OSI Layer | Information |
|-----------|-------------|
| **Frame (L1)** | Frame number, length, arrival time, encapsulation type |
| **Ethernet / MAC (L2)** | Source and destination MAC addresses |
| **IP (L3)** | Source/destination IPv4/IPv6 addresses, TTL, flags |
| **TCP / UDP (L4)** | Protocol, source/destination ports, sequence numbers, flags, payload size, retransmissions, checksum errors |
| **Application (L5+)** | HTTP, FTP, DNS, SMTP — methods, headers, status codes, payloads |

**Tip**: Click any field in Packet Details to highlight its corresponding bytes in the Packet Bytes pane.

---

## Packet Navigation & Search

### Go to Packet

- **Go → Go to Packet** (Ctrl+G): Jump directly to a specific packet number

### Find Packets

- **Edit → Find Packet** (Ctrl+F): Search by:
  - Display filter
  - Hex value
  - String (case-sensitive or insensitive)
  - Regular expression
- **Scope**: Choose where to search (Packet List, Packet Details, or Packet Bytes)

### Mark Packets

- **Right-click → Mark Packet** or use **Edit** menu
- Marked packets appear with black background
- **Note**: Marks are session-only and lost when file is closed

### Packet Comments

- **Right-click → Packet Comment**: Add or remove notes on individual packets
- Comments persist in the PCAP file and survive across sessions

### Export Packets

- **File → Export Specified Packets**: Save a subset (marked packets, displayed packets, range, etc.) to a new PCAP file

### Export Objects (Extracted Files)

- **File → Export Objects**: Choose protocol (HTTP, SMB, TFTP, etc.)
- Extracts files transferred over the network (images, documents, scripts, etc.)

---

## Time Display Formats

**View → Time Display Format** allows switching between:

- Seconds since beginning of capture
- UTC time
- Date and time of day
- Relative time (time offset from first packet)

Choose the format most useful for your analysis.

---

## Expert Information

**Analyse → Expert Information** (or click status bar icon) displays protocol warnings, errors, and informational notes.

- **Severities**: Error, Warning, Note, Chat (informational)
- **Groups**: Checksum errors, sequence issues, response codes, malformed packets, etc.

Use this to quickly identify protocol anomalies and potential network issues.

---

## Display Filtering

Display filters are applied to already-captured traffic (unlike capture filters used during sniffing). They allow you to focus on specific packets.

### Filter Application Methods

#### Apply as Filter
- **Right-click field → Apply as Filter**: Selected value becomes active filter
- Non-matching packets are hidden immediately

#### Conversation Filter
- **Right-click packet → Conversation Filter**: Shows only packets with matching IP addresses + ports (entire session)
- Automatically applies a display filter

#### Colourise Conversation
- **Right-click → Colourise Conversation**: Highlights related packets without filtering them out
- Useful for visual inspection without hiding data

#### Prepare as Filter
- **Right-click field → Prepare as Filter**: Adds query to filter bar without applying it
- Allows building complex filters using AND/OR logic before executing

#### Apply as Column
- **Right-click field → Apply as Column**: Adds new column to Packet List
- Enables quick scanning of specific values across many packets

### Common Display Filter Examples

| Goal | Filter Query |
|------|--------------|
| Show only HTTP traffic | `http` |
| Show only DNS traffic | `dns` |
| Show traffic to/from specific IP | `ip.addr == 192.168.1.1` |
| Show only TCP port 80 | `tcp.port == 80` |
| Show only HTTPS (TLS) | `ssl or tls` |
| Show packets with TCP errors | `tcp.analysis.flags` |
| Show specific TCP stream | `tcp.stream eq 0` |

---

## Stream Reconstruction

### Follow Stream

**Analyse → Follow TCP/UDP/HTTP Stream** (or right-click packet):

- Reconstructs complete application-layer conversation
- Client traffic appears in **red**, server traffic in **blue**
- Automatically applies display filter to isolate that stream
- Available for TCP, UDP, and HTTP streams

**Use Cases**:
- Viewing complete HTTP request/response exchanges
- Inspecting FTP or Telnet sessions (plaintext protocols)
- Analyzing DNS queries and responses

---

## TLS/SSL in Packet Capture

### Viewing Encrypted Traffic

- Encrypted TLS packets appear as "Application Data" in the TLS record layer
- Contents are not readable without decryption
- Server private key or session keys are required to decrypt

### Display Filter for TLS Handshakes

```
ssl.handshake or tls.handshake
```

### Decryption (Advanced)

- **TLS 1.2 (non-PFS)**: If you have the server's private key, configure it under **Preferences → Protocols → TLS**
- **TLS 1.3**: Uses Perfect Forward Secrecy (ECDHE) by default, making historical decryption impossible; use SSLKEYLOGFILE environment variable with compatible clients (Firefox, Chromium)

---

## Practical Workflow Examples

### Example 1: Quick Protocol Overview

1. Open PCAP or start live capture
2. Scan the **Packet List** for protocol distribution (HTTP, DNS, TCP, UDP)
3. Use colour-coding for quick visual patterns
4. Open **Statistics → Protocol Hierarchy** for a breakdown

### Example 2: Investigate Suspicious Traffic

1. Apply display filter: `ip.dst == 192.168.1.100`
2. Review packet types and destinations
3. Right-click suspicious packet → **Apply as Filter** to narrow down
4. Use **Follow Stream** for HTTP or DNS queries

### Example 3: Analyze a Conversation

1. Apply filter: `tcp.port == 443`
2. Find first handshake packet (SYN flag)
3. Right-click → **Conversation Filter** to isolate that session
4. Review certificate exchange in **Packet Details**

### Example 4: Extract Files

1. Apply filter: `http.request`
2. View HTTP requests in Packet List
3. **File → Export Objects → HTTP** to save transferred files
4. Check extracted files for content/malware (with caution)

---

## Best Practices

1. **Filter Before Analyzing**: Use display filters to reduce noise; start broad and narrow down
2. **Understand Context**: Know what traffic should look like normally before hunting anomalies
3. **Use Colours Wisely**: Apply consistent colourization rules for protocols or traffic types
4. **Annotate Important Packets**: Add comments to mark suspicious or significant packets
5. **Archive Captures**: Save important PCAP files with metadata for future reference and compliance
6. **Expert Info First**: Always check **Analyse → Expert Information** for protocol errors and warnings
7. **Verify Before Exporting**: Review exported objects/files before using them (potential security risk)

---

## Key Takeaways

1. Wireshark is a powerful tool for network analysis, troubleshooting, and learning.
2. Master the GUI sections (Packet List, Details, Bytes) to efficiently navigate captures.
3. Display filters are essential for focusing on relevant traffic without manual searching.
4. Follow Stream and Expert Info are key features for understanding conversations and protocol anomalies.
5. Use colour-coding and comments to organize complex analyses.
6. Always remember: Wireshark shows the data, but interpretation requires knowledge and caution.
