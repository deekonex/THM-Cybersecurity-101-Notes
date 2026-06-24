# Networking — Secure Protocols

## Introduction

Many legacy network protocols transmit data in plaintext, leaving communications vulnerable to eavesdropping, tampering, and impersonation. Secure protocols add cryptographic protections for:

- Confidentiality: prevent eavesdroppers from reading data
- Integrity: detect or prevent modification of data in transit
- Authenticity: verify the identity of the peers

Common risks when protocols are unsecured include leaked credentials, modified transactions, and connection to fraudulent services.

---

## Secure Versions of Common Protocols

| Plaintext | Secure Version | Typical Port |
|----------:|:---------------|:------------:|
| HTTP      | HTTPS (HTTP over TLS) | 443 |
| POP3      | POP3S | 995 |
| SMTP      | SMTPS / STARTTLS (SMTP with TLS) | 465 / 587 |
| IMAP      | IMAPS | 993 |
| TELNET    | SSH (replacement for remote shell) | 22 |

The "S" usually stands for Secure. Many protocols support opportunistic TLS (STARTTLS) where the connection is upgraded from plaintext to encrypted.

---

## TLS (Transport Layer Security)

### Overview

TLS is the standard cryptographic layer used to secure traffic between clients and servers. It provides:

- Confidentiality (encryption)
- Integrity (MACs / AEAD)
- Authenticity (certificates)

TLS operates at the Transport layer and is used by many application protocols (HTTP, SMTP, IMAP, etc.). TLS evolved from SSL; modern deployments should use TLS 1.2 or TLS 1.3 (preferably 1.3).

### Why TLS matters

Without TLS, data like passwords, session cookies, and personal information can be intercepted or modified. TLS encrypts traffic so passive sniffers cannot read content and active attackers cannot trivially tamper with messages.

### TLS 1.3 highlights (modern behaviour)

- Simplified handshake with fewer round trips (faster connection setup)
- Mandatory forward secrecy (uses ephemeral Diffie-Hellman like ECDHE)
- Deprecated insecure algorithms and legacy features (e.g., RSA key exchange removed)
- Uses AEAD ciphers (AES-GCM, ChaCha20-Poly1305) for combined encryption+integrity

### TLS Handshake (high-level, TLS 1.3)

1. ClientHello: client proposes TLS version, cipher suites, supported extensions (SNI, ALPN), and ephemeral key share.
2. ServerHello: server picks parameters and returns ephemeral key share.
3. (Optional) Encrypted extensions, certificate, CertificateVerify: server proves identity and signs handshake.
4. Both sides derive shared keys via Diffie-Hellman and begin encrypted application data.

Session resumption and 0-RTT are supported in TLS 1.3 to speed up repeat connections but must be used carefully due to replay risks.

---

## Certificates and PKI

### Purpose

X.509 certificates bind a public key to an identity (domain or organization) and are typically issued by Certificate Authorities (CAs) the client trusts.

### Certificate lifecycle

- Generate a private key
- Create CSR (Certificate Signing Request)
- Submit CSR to CA
- CA validates identity (varies by validation level) and issues certificate
- Install certificate and private key on server

### Types of certificates

- Domain Validated (DV): CA validates control of domain (fast, automated, e.g., Let's Encrypt)
- Organization Validated (OV): CA checks identity of organization (more assurance)
- Extended Validation (EV): strict validation and green bar in older browsers (less common today)
- Self-signed: signed by owner — not trusted publicly and should only be used for testing or internal use
- Wildcard: covers *.example.com
- SAN (Subject Alternative Name): covers multiple domains in one certificate

### Certificate considerations

- Use SANs over deprecated CN-only checks.
- Prefer short-lived certificates and automation (Let's Encrypt + Certbot) for renewal.
- Use OCSP stapling to provide revocation status without extra client network requests.
- Enforce HSTS (HTTP Strict Transport Security) to reduce downgrade attacks on browsers.

---

## Mutual TLS (mTLS)

Mutual TLS requires both client and server present certificates. Common uses:

- API authentication between services
- Secure administrative access

mTLS provides strong authentication but increases complexity (certificate distribution, rotation).

---

## Common TLS Tools & Commands

- Test a TLS connection and show certificate chain:

```bash
openssl s_client -connect example.com:443 -servername example.com
```

- Show only certificate chain and exit quickly:

```bash
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -text
```

- Curl with verbose TLS output:

```bash
curl -vI https://example.com/
```

- Check supported protocols/ciphers (modern OpenSSL):

```bash
openssl ciphers -v 'ALL:!aNULL'  # local supported cipher list
```

- Use sslyze or testssl.sh (not included here) or Qualys SSL Labs online scanner for external grading.

---

## TLS in Packet Capture (Wireshark / tshark)

- Encrypted TLS packets appear as "Application Data" with the TLS record layer; contents are not readable.
- If you have the server's private key (only for RSA key exchange or older non-PFS modes), you can decrypt captures by configuring Wireshark with the key. TLS 1.3 with ECDHE provides Perfect Forward Secrecy (PFS) and cannot be decrypted with the server private key alone.

- Example display filter to show TLS handshakes:

```
ssl.handshake or tls.handshake
```

- To decrypt TLS 1.2 non-PFS with Wireshark, add the server private key under Preferences → Protocols → TLS; for TLS 1.3, use session key logging (SSLKEYLOGFILE env var with a client like Firefox/Chrome) and import that file into Wireshark.

---

## Attacks & Historical Vulnerabilities (and mitigations)

- SSLv2/SSLv3 & weak ciphers: disable old protocol versions; use TLS 1.2+ only
- POODLE: exploited SSLv3 fallback — mitigate by disabling SSLv3 and enforcing secure TLS
- BEAST/CRIME/BREACH: attacks on older TLS/HTTP compression or block-cipher modes — mitigate with modern cipher selection, disabled compression
- Heartbleed (OpenSSL CVE-2014-0160): memory leak in OpenSSL's heartbeat extension — patch and rotate keys
- Downgrade attacks: use HSTS and properly configured servers to avoid protocol downgrade
- Man-in-the-Middle via forged certificates: use Certificate Transparency, pinning (carefully), and rely on trusted CAs

Modern mitigations: prefer TLS 1.3, ECDHE for PFS, AEAD ciphers, HSTS, OCSP stapling, and strong server configs.

---

## Practical Examples (real-life, hands-on)

### 1) Quick certificate check with OpenSSL

```bash
# Inspect cert chain and cipher negotiated
openssl s_client -connect example.com:443 -servername example.com
```

Interpretation: look for the certificate chain, verify certificate validity dates, and note the negotiated cipher (e.g., TLSv1.3, TLS_AES_128_GCM_SHA256).

### 2) Forcing TLS versions and ciphers with curl

```bash
# Force TLS 1.2
curl --tlsv1.2 -v https://example.com/

# Show negotiated TLS info
curl -vI https://example.com/ 2>&1 | sed -n '1,120p'
```

This helps test whether a server still accepts TLS 1.0/1.1 (which it shouldn’t).

### 3) Create a self-signed cert (local testing)

```bash
# Generate private key
openssl genpkey -algorithm RSA -out server.key -pkeyopt rsa_keygen_bits:2048

# Create self-signed cert valid 365 days
openssl req -new -x509 -key server.key -out server.crt -days 365 -subj "/CN=local.test"
```

Serve with a simple Python HTTPS server (testing only):

```bash
# Python 3.8+ example (not for production)
python -m http.server 8443 --bind 127.0.0.1 --directory . \
  --certfile server.crt --keyfile server.key
```

Clients will warn about an untrusted cert — expected for self-signed.

### 4) Analyze a captured TLS handshake (Wireshark)

- Look for ClientHello/ServerHello and check chosen cipher suite
- Verify certificate message and chain
- Check for Server Name Indication (SNI) in ClientHello if hosting multiple domains

---

## Server Configuration Examples (nginx snippets)

Use modern recommended defaults; example for TLS 1.2+ and 1.3 support:

```nginx
server {
  listen 443 ssl;
  server_name example.com;

  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

  # Protocols
  ssl_protocols TLSv1.2 TLSv1.3;

  # Prefer server ciphers
  ssl_prefer_server_ciphers on;

  # Cipher suites (example; adjust to your OpenSSL version)
  ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:...';

  # Security headers
  add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

  # OCSP stapling
  ssl_stapling on;
  ssl_stapling_verify on;
}
```

Always test config with a staging environment and tools like `openssl s_client` and online scanners.

---

## Checking and Hardening

- Run an external scan (Qualys SSL Labs) for a public service
- Use `openssl s_client`, `sslyze`, or `testssl.sh` for in-depth analysis
- Enforce HSTS and implement secure cookies (Secure, HttpOnly, SameSite)
- Rotate keys periodically and use automated renewals (Certbot / ACME)
- Monitor Certificate Transparency logs for unexpected certificates

---

## Key Takeaways

1. TLS is the standard way to secure application protocols and should be used everywhere sensitive data is transmitted.
2. Use TLS 1.3 where possible, with ECDHE for forward secrecy and AEAD ciphers for integrity.
3. Certificates issued by trusted CAs provide authenticity; self-signed certs should only be used in test/internal contexts.
4. Harden server configurations, enable HSTS and OCSP stapling, and automate certificate renewals.
5. Test and monitor your TLS deployment with tools like openssl, sslyze, and external scanners.


