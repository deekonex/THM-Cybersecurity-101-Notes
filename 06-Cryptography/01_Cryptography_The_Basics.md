# Cryptography: The Basics

## Overview

Cryptography is the practice and study of techniques for secure communication in the presence of adversaries. It ensures that no third parties can read, alter, or forge the exchanged data, and that we can verify we're communicating with the intended recipient.

### The Three Pillars of Cryptography
- **Confidentiality**: Only authorized parties can read the message
- **Integrity**: The message hasn't been altered during transmission
- **Authenticity**: We can verify the message came from the claimed sender

---

## Why Cryptography Matters

Cryptography is the foundation of digital trust and is used ubiquitously in modern life:

### Common Real-World Applications
- **Web Browsing**: HTTPS encrypts data between your browser and websites
- **Authentication**: Login credentials are encrypted when sent to servers
- **SSH/Remote Access**: Encrypted tunnels prevent eavesdropping on terminal sessions
- **Online Banking**: Certificate verification confirms you're connected to your actual bank
- **File Integrity**: Hash functions verify downloaded files haven't been corrupted or tampered with
- **Password Storage**: Systems hash (not encrypt) passwords so they can't be recovered even if databases are breached

### Regulatory Requirements
Organizations handling sensitive data must comply with standards:
- **PCI DSS**: Credit card data must be encrypted at rest and in transit
- **HIPAA/HITECH** (USA): Medical records protection requirements
- **GDPR** (EU): General data protection and privacy regulations
- **DPA** (UK): Data protection standards

---

## Key Cryptographic Terms

| Term | Definition |
|------|-----------|
| **Plaintext** | Original, readable message or data before encryption (documents, images, any binary data) |
| **Ciphertext** | Scrambled, unreadable version after encryption; should reveal no information about the plaintext except approximate size |
| **Cipher** | Algorithm or mathematical method to convert plaintext ↔ ciphertext |
| **Key** | String of bits used by the cipher to encrypt/decrypt data; must remain secret (except public keys in asymmetric encryption) |
| **Encryption** | Process of converting plaintext → ciphertext using a cipher and key |
| **Decryption** | Process of converting ciphertext → plaintext using a cipher and key |

**Important Principle**: The cipher is typically public knowledge; security depends on the key remaining secret.

---

## Historical Ciphers

### Caesar Cipher (1st Century BCE)

The simplest historical cipher: shift each letter by a fixed number.

**Example**:
- Plaintext: `TRYHACKME`
- Key: 3 (right shift)
- Ciphertext: `WUBKDFNPH`

Each letter shifts 3 positions in the alphabet, wrapping from Z back to A.

**Decryption**: Shift left by 3 to recover the original message.

**Why It's Insecure**: Only 25 possible keys exist (shifting by 26 = no change). An attacker can try all possibilities in seconds. By modern standards, this is completely broken.

### Other Historical Ciphers
- **Vigenère Cipher** (16th century): Multiple Caesar shifts using a repeating key
- **Enigma Machine** (WWII): Electromechanical rotor-based substitution cipher
- **One-Time Pad** (Cold War): Theoretically unbreakable, but requires a key as long as the plaintext

---

## Modern Encryption: Two Main Types

### 1. Symmetric Encryption

**Definition**: Same key is used to encrypt AND decrypt the data.

```
Alice ---encrypt with key K---> Ciphertext ---decrypt with key K---> Bob
          (sends to Bob)                        (Bob's side)
```

**Requirements**:
- The key must be kept absolutely secret
- Both parties must securely exchange the key before communication
- Managing keys becomes difficult with many recipients

**Real-World Challenge**: How do you securely share the password for an encrypted email with a colleague? You can't email it (anyone with mailbox access gets both the file and password). You'd need to meet in person or use a separate secure channel.

**Common Symmetric Ciphers**:
- **DES** (1977): 56-bit key → Broken by brute force in <24 hours (1999)
- **3DES**: DES applied 3 times; 168-bit key (112-bit effective security) → Deprecated in 2019
- **AES** (2001): 128, 192, or 256-bit keys → Current standard, considered secure

| Cipher | Key Size | Status | Notes |
|--------|----------|--------|-------|
| DES | 56 bits | Broken | Outdated, shouldn't be used |
| 3DES | 168 bits (112-bit effective) | Deprecated | Found in legacy systems only |
| AES | 128/192/256 bits | Standard | Modern go-to cipher |

### 2. Asymmetric Encryption (Public Key Cryptography)

**Definition**: Uses a pair of mathematically linked keys:
- **Public Key**: Shared freely; used to encrypt
- **Private Key**: Kept secret; used to decrypt

```
Anyone ---encrypt with Bob's public key---> Ciphertext ---decrypt with Bob's private key---> Bob
```

**Advantages**:
- No need to secretly share keys beforehand
- Scalable to many participants
- Enables digital signatures (proving sender identity)

**Disadvantages**:
- Significantly slower than symmetric encryption
- Requires larger keys for equivalent security

**Key Size Comparison** (for equivalent security levels):
- **RSA**: 2048-bit minimum (3072/4096 bits recommended)
- **Diffie-Hellman**: 2048-bit minimum (3072/4096 bits recommended)
- **ECC** (Elliptic Curve): 256-bit provides security equivalent to 3072-bit RSA

**Mathematical Foundation**: Asymmetric encryption relies on "one-way" mathematical problems that are:
- Easy to compute in one direction
- Computationally infeasible to reverse (would take millions of years with current technology)

**Common Asymmetric Ciphers**:
- **RSA**: Factoring large numbers
- **Diffie-Hellman**: Discrete logarithm problem
- **ECC**: Elliptic curve discrete logarithm problem

---

## Basic Mathematical Operations in Cryptography

### XOR (Exclusive OR) Operation

**Definition**: Binary logical operation comparing two bits.

**Truth Table**:
| A | B | A ⊕ B |
|---|---|-------|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

Returns 1 if bits are different, 0 if the same.

**Example**: 1010 ⊕ 1100 = 0110

**Key Properties** (why XOR is useful in cryptography):
- **Self-inverse**: A ⊕ A = 0
- **Identity**: A ⊕ 0 = A
- **Commutative**: A ⊕ B = B ⊕ A
- **Associative**: (A ⊕ B) ⊕ C = A ⊕ (B ⊕ C)

**Cryptographic Application**: Simple symmetric cipher
```
Encryption: Ciphertext (C) = Plaintext (P) ⊕ Key (K)
Decryption: P = C ⊕ K = (P ⊕ K) ⊕ K = P ⊕ (K ⊕ K) = P ⊕ 0 = P
```

**Limitation**: Requires a secret key as long as the plaintext itself (impractical).

### Modulo Operation

**Definition**: The remainder when dividing X by Y, written as X mod Y or X % Y.

**Examples**:
- 25 mod 5 = 0 (25 ÷ 5 = 5 remainder 0)
- 23 mod 6 = 5 (23 ÷ 6 = 3 remainder 5)
- 23 mod 7 = 2 (23 ÷ 7 = 3 remainder 2)

**Properties**:
- Result is always non-negative
- Result is always less than the divisor
- For any integer a and positive integer n: 0 ≤ (a mod n) ≤ n-1
- **NOT reversible**: x mod 5 = 4 has infinite solutions (4, 9, 14, 19, ...)

**Cryptographic Use**: Forms the basis for many algorithms including RSA and Diffie-Hellman.

---

## Cryptography Terminology Reference

- **Alice and Bob**: Standard fictional characters in cryptography examples representing two parties
- **Private Key**: Secret key that must be guarded (used in both symmetric and asymmetric encryption)
- **Public Key**: Non-secret key that can be freely distributed (asymmetric encryption only)

---

## Summary

| Aspect | Symmetric | Asymmetric |
|--------|-----------|-----------|
| Keys | One shared key | Public + private key pair |
| Speed | Fast | Slower |
| Key Exchange | Requires secure channel | Not required |
| Use Cases | Bulk data encryption | Key exchange, digital signatures, authentication |
| Key Size | Smaller (128-256 bits) | Larger (2048-4096 bits) |

Cryptography is essential to digital security and is everywhere in modern communication—from banking to browsing. Understanding these fundamentals provides the foundation for exploring advanced cryptographic systems like public key cryptography and digital signatures.

---

## Next Steps
- Study Public Key Cryptography Basics for asymmetric systems in detail
- Learn about hashing and its role in data integrity and authentication
