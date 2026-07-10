# Hashing Basics

## Overview

A **hash function** is a mathematical algorithm that transforms input data of arbitrary size into a fixed-length string of characters (typically hexadecimal). This output is called a **hash value** or **digest**. The key property is that even a single-bit change in the input produces a completely different hash output.

**Key Characteristics:**
- **One-way function**: Cannot reverse a hash to get the original input
- **Deterministic**: Same input always produces the same output
- **Avalanche effect**: Small input changes produce drastically different outputs
- **Fast computation**: Should be computationally efficient

---

## Common Hashing Algorithms

| Algorithm | Output Size | Security Status | Use Case |
|-----------|------------|-----------------|----------|
| MD5 | 128 bits (16 bytes) | ❌ **Deprecated** - Vulnerable to collisions | Legacy systems only |
| SHA-1 | 160 bits (20 bytes) | ❌ **Weak** - Collision attacks exist | Phase out in use |
| SHA-256 | 256 bits (32 bytes) | ✅ **Secure** | General-purpose hashing |
| SHA-512 | 512 bits (64 bytes) | ✅ **Secure** | High-security applications |
| Bcrypt | ~184 bits | ✅ **Secure** | Password hashing (with salt) |
| Scrypt | Variable | ✅ **Very Secure** | Password hashing (memory-hard) |
| Argon2 | Variable | ✅ **Very Secure** | Modern password hashing |
| yescrypt | 256 bits | ✅ **Very Secure** | Modern Linux passwords |

**Practical Commands:**
```bash
md5sum <filename>        # MD5 hash (avoid for security)
sha1sum <filename>       # SHA-1 hash (avoid for security)
sha256sum <filename>     # SHA-256 hash (recommended)
sha512sum <filename>     # SHA-512 hash (high security)
```

### Demonstration: Avalanche Effect

Two files differing by only one bit produce completely different hashes:

**Input:** `T` (0x54) vs `U` (0x55) — differs by 1 bit

```
File 1 (T):
  MD5:    b9ece18c950afbfa6b0fdbfa4ff731d3
  SHA-1:  c2c53d66948214258a26ca9ca845d7ac0c17f8e7
  SHA-256: e632b7095b0bf32c260fa4c539e9fd7b852d0de454e9be26f24d0d6f91d069d3

File 2 (U):
  MD5:    4c614360da93c0a041b22e537de151eb
  SHA-1:  b2c7c0caa10a0cca5ea7d69e54018ae0c0389dd6
  SHA-256: a25513c7e0f6eaa80a3337ee18081b9e2ed09e00af8531c8f7bb2542764027e7
```

---

## Real-Life Applications

### 1. **File Integrity Verification**
Download a large file and verify it matches the source:
- Software vendors publish SHA-256 checksums alongside downloads
- Users run `sha256sum downloaded_file` to verify authenticity
- Any corruption during transfer is immediately detected
- Example: Linux distributions publish checksums for ISO files

### 2. **Detecting Duplicate Files**
- Disk cleanup tools use hashing to find duplicate files efficiently
- Files with identical hashes are identical content
- Faster than byte-by-byte comparison for large files

### 3. **Password Authentication**
- Secure password storage in databases
- Combined with salts to prevent rainbow table attacks
- Bcrypt/Argon2 add computational delay to slow brute-force attacks

### 4. **Data Integrity in Transit**
- HTTPS uses HMACs to verify messages haven't been tampered with
- Blockchain uses hashing for transaction verification
- Version control systems (Git) hash commits for integrity

### 5. **Malware Detection**
- Files are hashed and compared against malware signature databases
- VirusTotal uses file hashes to track malware samples
- Faster than full file scanning for known threats

---

## Secure Password Storage

### Why Not Plaintext?
❌ Database breaches expose all passwords directly  
❌ Insiders with database access see all passwords  
❌ Passwords are easily searchable  

### Why Not Basic Hashing (MD5/SHA)?
❌ **Rainbow Tables**: Pre-computed hash-to-password databases make cracking instant  
❌ **Fast computation**: Billions of hashes can be tested per second  
❌ **No uniqueness**: Same password = same hash for all users  

### The Solution: Salting

A **salt** is random data added to each password before hashing:

```
Without Salt:  password → SHA256 → abc123def456...
               password → SHA256 → abc123def456... (same hash!)

With Salt:     password + salt₁ → SHA256 → xyz789uvw012...
               password + salt₂ → SHA256 → hij345klm678... (different hashes!)
```

**Salt Properties:**
- ✅ Should be random
- ✅ Should be unique per user
- ✅ Should be 16+ bytes
- ✅ Doesn't need to be secret (stored with hash)
- ✅ Makes rainbow tables impractical

### Best Practices for Password Storage

1. **Choose a secure algorithm**: Argon2 > Scrypt > Bcrypt > PBKDF2
2. **Use automatic salting**: Modern algorithms (Bcrypt, Scrypt, Argon2) handle this automatically
3. **Use cost parameters**: These slow down hashing to resist brute-force attacks
4. **Never use deprecated algorithms**: Avoid MD5, SHA-1

**Example (Bcrypt with OpenSSL):**
```bash
openssl passwd -6  # SHA-512 with salt
# or
openssl passwd -5  # SHA-256 with salt
```

**Storage Format (Linux Shadow File):**
```
$prefix$options$salt$hash
```

---

## Recognizing Hash Types

### Linux Password Hashes (/etc/shadow)

Stored format: `$prefix$options$salt$hash`

| Prefix | Algorithm | Strength | Example Use |
|--------|-----------|----------|-------------|
| `$y$` | yescrypt | ⭐⭐⭐⭐⭐ Strongest | Modern systems |
| `$2b$` | Bcrypt | ⭐⭐⭐⭐ Strong | Common |
| `$6$` | SHA-512 | ⭐⭐⭐ Medium | Older systems |
| `$5$` | SHA-256 | ⭐⭐ Weak | Legacy |
| `$1$` | MD5 | ⭐ Very Weak | Very old systems |

**Example Breakdown:**
```
strategos:$y$j9T$76UzfgEM5PnymhQ7TlJey1$/OOSg64dhfF.TigVPdzqiFang6uZA4QA1pzzegKdVm4:...

$y$ → yescrypt algorithm
j9T → algorithm cost parameter
76UzfgEM5PnymhQ7TlJey1 → salt (22 chars = 128 bits)
/OOSg64... → final hash value
```

### Windows Password Hashes

- **Algorithm**: NTLM (based on MD4, weaker than modern standards)
- **Storage**: Security Accounts Manager (SAM)
- **Format**: Visually identical to MD4/MD5 hashes
- **Extraction**: Tools like `mimikatz` bypass OS protections
- **Hash types**: NT hashes and LM hashes (LM is older/weaker)

### Hash Type Identification

**Typical Hash Sizes:**
- MD5: 32 hex characters (128 bits)
- SHA-1: 40 hex characters (160 bits)
- SHA-256: 64 hex characters (256 bits)
- SHA-512: 128 hex characters (512 bits)
- NTLM: 32 hex characters (indistinguishable from MD5—use context!)

**Context Clues:**
- Found in Linux `/etc/shadow`? → Check prefix (`$1$`, `$5$`, `$6$`, `$y$`, etc.)
- From Windows SAM? → Likely NTLM
- From web application database? → More likely MD5 than NTLM
- With database format like `$2a$10$...`? → Bcrypt

---

## Password Cracking Techniques

### Attack Methods

1. **Dictionary Attack**: Try common passwords from wordlists (rockyou.txt, etc.)
2. **Brute Force**: Try all possible character combinations (slow without GPU)
3. **Rainbow Tables**: Use pre-computed hash lookup tables (prevented by salting)
4. **Hybrid Attack**: Combine dictionary with character modifications (e.g., `password123!`)

### Tools

**Hashcat** (GPU-accelerated, very fast):
```bash
# Basic syntax
hashcat -m <hash_type> -a <attack_mode> <hashfile> <wordlist>

# -m specifies hash type (1400=SHA-256, 3200=Bcrypt, etc.)
# -a specifies attack (0=dict, 1=combo, 3=brute-force, 6=hybrid)

# Examples:
hashcat -m 1400 -a 0 hash.txt rockyou.txt     # SHA-256 dictionary
hashcat -m 3200 -a 0 hash.txt rockyou.txt     # Bcrypt dictionary
hashcat -m 1800 -a 0 hash.txt rockyou.txt     # SHA-512 dictionary
```

**John the Ripper** (Multi-platform):
```bash
john --format=bcrypt --wordlist=rockyou.txt hash.txt
john --format=sha256crypt hash.txt
```

**Online Tools** (Fast but less private):
- CrackStation.net — Large rainbow tables
- Hashes.com — MD5, SHA1, SHA256 cracking

### Why Modern Algorithms Resist Cracking

- **Bcrypt, Scrypt, Argon2** include cost parameters
- **One hash attempt takes milliseconds** (vs microseconds for SHA-256)
- **GPU acceleration is limited** by memory/computational constraints
- **Example**: Bcrypt with cost=10 makes brute-forcing impractical

---

## HMACs: Authenticated Hashing

**HMAC** (Keyed-Hash Message Authentication Code) combines a hash function with a secret key to prove both **authenticity** (sender identity) and **integrity** (message unchanged).

### How HMACs Work

```
1. Pad secret key to hash function block size
2. XOR padded key with ipad constant (0x36...)
3. Hash: H((key ⊕ ipad) || message)
4. XOR padded key with opad constant (0x5C...)
5. Hash result again: H((key ⊕ opad) || hash_from_step_3)
6. Final output: HMAC value
```

**Formula:**
```
HMAC(K, M) = H((K ⊕ opad) || H((K ⊕ ipad) || M))
```

### Practical Use

- **API Authentication**: Request signed with HMAC-SHA256 proves sender identity
- **Message Verification**: HTTPS uses HMACs in TLS record authentication
- **Webhook Verification**: GitHub/Stripe sign webhooks with HMACs
- **Session Tokens**: HMAC prevents token tampering

---

## File Integrity Checking

### Checksums in Practice

Large files are distributed with published hashes to verify integrity:

**Example: Fedora Linux**
```
SHA256 (Fedora-Workstation-Live-x86_64-40-1.14.iso) = dd1faca950d1a8c3d169adf2df4c3644ebb62f8aac04c401f2393e521395d613
```

**Verification Process:**
```bash
# Download the ISO
wget https://example.com/Fedora-Workstation-Live-x86_64-40-1.14.iso

# Compute hash
sha256sum Fedora-Workstation-Live-x86_64-40-1.14.iso

# Compare output to published hash
# If they match → file is authentic and unmodified
```

**Benefits:**
- ✅ Detects corruption during download
- ✅ Detects man-in-the-middle tampering
- ✅ Fast verification (seconds vs minutes)

---

## Distinguishing: Hashing vs Encoding vs Encryption

| Property | Hashing | Encoding | Encryption |
|----------|---------|----------|-----------|
| **Reversible?** | ❌ One-way | ✅ Reversible | ✅ Reversible (with key) |
| **Purpose** | Integrity/Authentication | Format compatibility | Confidentiality |
| **Key/Secret?** | No | No | Yes (requires key) |
| **Example** | SHA-256 | Base64, UTF-8 | AES-256, RSA |
| **Security** | Protects against tampering | Does NOT provide security | Protects confidentiality |

**Key Distinction:**
- **Hashing** = Integrity checking (you can't get original data back)
- **Encoding** = Format conversion (anyone can decode)
- **Encryption** = Confidentiality (only key holders can decrypt)

### Example: Base64 Encoding

```bash
# Encoding (reversible, NOT secure)
echo "TryHackMe" | base64
# Output: VHJ5SGFja01lCg==

# Decoding
echo "VHJ5SGFja01lCg==" | base64 -d
# Output: TryHackMe
```

⚠️ **Note**: Base64 is just formatting—anyone can decode it. It provides zero security.

---

## Summary Table: When to Use What

| Scenario | Use | Why |
|----------|-----|-----|
| Verify file didn't corrupt | **SHA-256 Hash** | Detect any bit-level changes |
| Store user passwords | **Argon2/Bcrypt** | Resistant to brute-force attacks |
| Quick authentication verification | **HMAC-SHA256** | Proves message authenticity |
| Detect duplicate files | **Any hash (MD5 OK)** | Speed matters, security less critical |
| API request signing | **HMAC-SHA256** | Prevents tampering and identifies sender |
| Malware detection | **MD5 (legacy) or SHA-256** | Fast lookup in signature databases |
| Data in transit | **HMAC** | Ensures integrity + authenticity |

---

## Key Takeaways

✅ Hash functions are one-way, deterministic, and produce avalanche effects  
✅ Modern algorithms (SHA-256+, Bcrypt, Argon2) are secure; avoid MD5/SHA-1  
✅ **Always salt passwords**—prevents rainbow table attacks  
✅ Use cost parameters (Bcrypt/Argon2) to slow brute-force attacks  
✅ HMACs add authenticity to hashing; useful for message verification  
✅ Hashing ≠ Encryption ≠ Encoding (each serves different purposes)  
✅ Verify file integrity with checksums before trusting downloads  

