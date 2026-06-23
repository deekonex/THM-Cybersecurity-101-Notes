# Networking — Core Protocols

## Quick ports

- DNS: UDP/TCP 53 (UDP for queries; TCP for zone transfer and large responses)
- HTTP: TCP 80
- HTTPS: TCP 443 (TLS)
- FTP (control): TCP 21; data: TCP 20 (active) or ephemeral ports (passive)
- SFTP (SSH File Transfer): TCP 22 (not FTP over SSH)
- SMTP (MTA): TCP 25; Submission: TCP 587 (STARTTLS + auth); SMTPS (implicit TLS): TCP 465
- POP3: TCP 110; POP3S (TLS): TCP 995
- IMAP: TCP 143; IMAPS (TLS): TCP 993
- WHOIS: TCP 43 (query to registrar)

---

## DNS (Domain Name System)

### Purpose

Resolve domain names to IP addresses and provide service metadata (MX, TXT, SRV).

### Common record types

- A (IPv4), AAAA (IPv6)
- CNAME (alias)
- MX (mail exchange)
- NS (name server)
- TXT (text; SPF/DKIM/DMARC)
- PTR (reverse DNS)
- SOA (zone authority)
- SRV (service locator)

### Notes & security

- DNS uses UDP 53 for most queries; TCP 53 for zone transfers (AXFR) and large responses
- DNS is unauthenticated by default; DNSSEC adds cryptographic signatures for integrity and origin validation
- Consider query privacy (DNS over HTTPS/TLS) to prevent on-path observers

### Tools / examples

```
# Query A record
$ dig example.com A

# Query MX records
$ dig example.com MX

# Use a specific resolver
$ dig @8.8.8.8 example.com ANY
```

---

## WHOIS

### Purpose

Lookup registration metadata (registrant, registrar, creation/expiry). Uses WHOIS servers on TCP port 43.

### Limitations

- Many fields are redacted (privacy/GDPR)
- Information depends on registrar-provided data and may be out-of-date

```
$ whois example.com
```

---

## HTTP / HTTPS

### Purpose

Application-layer request/response protocol for web pages and APIs.

### Important methods

GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS

### Common headers

Host, User-Agent, Accept, Content-Type, Authorization, Cookie, Set-Cookie, Location

### Status codes

1xx informational, 2xx success, 3xx redirect, 4xx client error, 5xx server error

### HTTPS and TLS

- HTTPS = HTTP over TLS (confidentiality, integrity, authentication)
- Use TLS 1.2+; prefer TLS 1.3
- Validate certificate chains and hostnames (SNI)
- Use HSTS, secure cookies, and ciphersuites with forward secrecy

### Quick examples

```
# Show response headers
$ curl -I https://example.com

# Download a page
$ curl -o page.html http://10.10.10.5/flag.html

# Manual raw request (telnet/netcat to port 80)
$ printf 'GET /path HTTP/1.1\r\nHost: example.com\r\n\r\n' | nc 10.10.10.5 80
```

---

## FTP / FTPS / SFTP

### FTP (legacy)

- Control: TCP 21; data: separate connection (active uses TCP 20 or server-initiated connection; passive uses ephemeral ports)
- Credentials and data may be plaintext unless protected
- Common commands: USER, PASS, LIST, RETR, STOR, PASV

### FTPS

- FTP over TLS
- Explicit (AUTH TLS) or implicit (different port)

### SFTP

- File transfer over SSH (port 22)
- Different protocol from FTP; preferred secure option

### Examples

```
$ ftp 10.10.10.5
# username: anonymous

# SFTP
$ sftp user@10.10.10.5
```

---

## SMTP (sending email)

### Purpose

Server-to-server mail transfer and client submission.

### Ports

- 25 (MTA-to-MTA)
- 587 (submission; STARTTLS + auth)
- 465 (legacy implicit TLS)

### Key commands

HELO/EHLO, MAIL FROM:, RCPT TO:, DATA, STARTTLS, QUIT

### Deliverability and authentication

- Mail routing relies on MX records
- Use SPF (DNS TXT), DKIM (signed headers/mail body), and DMARC (policy) to improve trust and reduce spoofing

---

## POP3 vs IMAP

### POP3

- Download-and-optional-delete model
- Ports: 110 (plaintext), 995 (POP3S TLS)
- Commands: USER, PASS, STAT, LIST, RETR, DELE, QUIT

### IMAP

- Synchronizing model; messages stay on server, supports folders and flags
- Ports: 143 (IMAP), 993 (IMAPS TLS)
- Commands: LOGIN, SELECT, FETCH, STORE, COPY, MOVE, LOGOUT

### Security note

- Plaintext protocols expose credentials—use STARTTLS or implicit TLS; prefer modern auth (OAuth2) where possible

---

## Packet capture observations

- Plaintext protocols reveal credentials and content (HTTP non-TLS, FTP, SMTP without STARTTLS, POP3)
- Use tshark/wireshark to inspect captures

```
$ tshark -r capture.pcap -Y "http or ftp or smtp" -V
```

---

## Practical tips & checks

- Prefer secure options: HTTPS, SFTP, IMAPS/POP3S, SMTP submission on 587 with STARTTLS
- For email: configure SPF, DKIM, DMARC
- For DNS troubleshooting: dig +short; check NS delegation and TTLs
- When testing services: nmap -sV -p- target; curl, wget, telnet/nc, appropriate client tools

---

## References & quick commands

```
# DNS
$ dig example.com A
$ nslookup -type=MX example.com

# WHOIS
$ whois example.com

# Web
$ curl -I https://example.com
$ wget http://10.10.10.5/flag.html

# FTP / SFTP
$ ftp 10.10.10.5
$ sftp user@10.10.10.5

# Email
$ telnet 10.10.10.5 25

# Scanning
$ nmap -sS -sV -p- 10.10.10.5
```

---

## Key takeaways

1. Use encrypted alternatives whenever possible to protect credentials and content.
2. DNS is unauthenticated—use DNSSEC and private resolvers for stronger guarantees.
3. Email trust comes from SPF/DKIM/DMARC plus proper server configuration.
4. Know the default ports and common client commands for quick testing and triage.
