# Public Key Cryptography Basics

## Overview
Public (asymmetric) key cryptography uses a key pair: a public key that can be shared widely and a private key that is kept secret. It enables authentication, integrity, and in some cases confidentiality, and is commonly used alongside symmetric cryptography to combine performance and security.

Key security goals:
- Authentication: confirm identity of a party.
- Integrity: ensure data has not been altered.
- Confidentiality: prevent unauthorized reading.
- Non-repudiation: provide evidence a specific party created a message (with caveats).

Asymmetric systems are slower than symmetric ones; real-world protocols typically use asymmetric crypto to establish or protect symmetric session keys, then use symmetric algorithms for bulk data encryption (hybrid encryption).

---

## Common real-world uses (examples)
- HTTPS/TLS: certificates + asymmetric crypto to authenticate servers and negotiate session keys; TLS often uses ECDHE for forward secrecy.
- SSH: public-key authentication to log into servers (ed25519 or RSA keys).
- Email encryption/signing: PGP/GPG for end-to-end message confidentiality and signatures.
- Code and package signing: ensure integrity and origin of software releases.
- VPNs and secure messaging: key exchange and signatures to establish secure channels.

---

## Key concepts and short analogies
- Lock-and-box analogy: a recipient’s public key is like a lock anyone can close; only the recipient’s private key (the only matching key) can open it. This illustrates confidentiality (encrypt with public key) but omits authenticity — signatures are required to prove the sender.
- Hybrid approach: use asymmetric crypto once to securely exchange a symmetric key, then use fast symmetric encryption for the session.

---

## RSA (brief)
- Based on the difficulty of factoring large integers: public key (n, e), private key (n, d), where n = p * q (p, q primes).
- Typical use: encrypt small pieces of data or, more commonly, encrypt a symmetric key or verify signatures.
- Practical notes:
  - Use modern key sizes (at least 2048 bits; 3072+ or ECC recommended for longer-term security).
  - Don’t roll your own RSA padding—use standardized schemes (e.g., RSA-OAEP for encryption, RSASSA-PSS for signatures).
  - Common attacks target poor randomness, small primes, improper padding (e.g., Bleichenbacher attacks against PKCS#1 v1.5), or low public exponent misuse.

---

## Diffie–Hellman key exchange (DH)
- Purpose: two parties derive a shared secret over an insecure channel without transmitting the secret.
- Core idea: each side computes g^a mod p and g^b mod p, exchange those values, then both compute (g^b)^a = (g^a)^b which equals g^(ab) mod p.
- Variants: classic DH over finite fields, and Elliptic Curve Diffie–Hellman (ECDH).
- Important: DH by itself does not authenticate parties — unauthenticated DH is vulnerable to active MITM attacks. Use authenticated variants (e.g., signed DH, certificates, or authenticated key agreement like TLS's ECDHE with server certificates).
- Forward secrecy: ephemeral DH (using fresh DH keys per session) provides forward secrecy — compromise of long-term keys does not reveal past session keys.

---

## Digital Signatures & Certificates
- Digital signatures: create a signature with a private key; anyone with the corresponding public key can verify the signature, providing authenticity and integrity.
- Certificates: bind a public key to an identity (domain, organization, person) and are issued by Certificate Authorities (CAs) forming a chain of trust.
- Key certificate points:
  - Fields: subject, issuer, public key, validity period, Subject Alternative Name (SAN), usage constraints.
  - Revocation: CRL and OCSP exist but have operational limitations; short-lived certificates and automatic renewal (e.g., Let’s Encrypt) mitigate revocation problems.
  - Pinning and strict validation (e.g., certificate transparency) improve trust.

---

## SSH keys
- Common, secure choices today: ed25519 (recommended), RSA (2048/3072+), ECDSA (use carefully). Avoid DSA.
- Generate locally: `ssh-keygen -t ed25519`.
- Protect private keys: filesystem permissions (600), optional passphrase, hardware tokens (e.g., YubiKey).
- Server trust: add the public key to `~/.ssh/authorized_keys` on the server; do not copy private keys to servers.

---

## PGP / GPG
- OpenPGP provides public-key encryption and signing for files and email.
- Web of trust vs CA model: PGP historically uses a web-of-trust model (users sign each others’ keys) rather than centralized CAs.
- Practical tips: set reasonable expiry, back up private keys securely, use passphrases and hardware tokens when available, prefer modern curves (ed25519) if supported.

---

## Practical security considerations (what matters most)
- Use well-reviewed libraries and standards (TLS, OpenSSH, OpenPGP implementations) rather than building primitives yourself.
- Prefer modern algorithms and parameters: ed25519/curve25519 for signatures/key agreement or RSA 3072+ if RSA is required.
- Ensure strong randomness sources (OS CSPRNG). Weak randomness breaks keys.
- Employ forward secrecy (ephemeral ECDHE) for session-based protocols.
- Protect private keys: use secure storage, hardware tokens, KMS, and least-privilege access.
- Understand and use proper padding, modes, and authenticated encryption (e.g., AES-GCM) for symmetric crypto.

---

## Short glossary
- Asymmetric: public/private key pair cryptography.
- Symmetric: single shared key used for encrypt/decrypt (fast).
- Hybrid encryption: use asymmetric crypto to securely transfer a symmetric key, then use symmetric crypto for bulk data.
- Forward secrecy: past sessions remain secure even if long-term keys are later compromised.

---

## Further reading / references
- RFCs and standards: RFC 8017 (PKCS#1), TLS 1.3 specification, OpenPGP RFC 4880.
- Practical guides: OWASP crypto recommendations, NIST guidance on key sizes and algorithms.

