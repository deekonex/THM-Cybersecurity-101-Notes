# John the Ripper - The Basics

## Overview

John the Ripper is a powerful, open-source password cracking tool used to identify weak passwords through hash cracking. It supports multiple hash formats and can work with various file types including password-protected archives and SSH keys.

## Core Concepts

### What is John the Ripper?

John the Ripper is a password cracking utility that attempts to determine plaintext passwords from encrypted hashes using various attack methods. It's widely used in penetration testing, forensics, and security auditing.

### Popular Versions

- **Community Version**: Free, standard John the Ripper
- **Jumbo John**: Extended version with support for additional hash types and features (most popular for comprehensive penetration testing)

### Common Wordlists

**RockYou Wordlist**: Derived from the 2009 rockyou.com breach, containing millions of commonly used passwords. This wordlist is essential for dictionary attacks and is one of the most effective tools for cracking real-world passwords.

## Basic Syntax

```bash
john --format=[format] --wordlist=[path to wordlist] [path to file]
```

**Parameters:**
- `--format=`: Specifies the hash format (e.g., md5, sha1, sha256, nt)
- `--wordlist=`: Path to dictionary file for wordlist mode
- `[path to file]`: File containing the hash to crack

## Hash Types and Cracking Methods

### Common Hash Formats

| Hash Type | Example Format | Cracking Command |
|-----------|----------------|------------------|
| MD5 | `--format=md5` | `john --format=md5 --wordlist=rockyou.txt hash.txt` |
| SHA1 | `--format=sha1` | `john --format=sha1 --wordlist=rockyou.txt hash.txt` |
| SHA256 | `--format=sha256` | `john --format=sha256 --wordlist=rockyou.txt hash.txt` |
| Whirlpool | `--format=whirlpool` | `john --format=whirlpool --wordlist=rockyou.txt hash.txt` |

### Real-Life Application Example: Web Application Breach

Imagine a web application database is compromised containing user passwords hashed with MD5. An attacker extracts these hashes and uses John with the RockYou wordlist to crack them:

```bash
john --format=md5 --wordlist=/usr/share/wordlists/rockyou.txt stolen_hashes.txt
```

Within minutes to hours (depending on password complexity), common passwords are cracked. This demonstrates why proper password hashing (bcrypt, scrypt, Argon2) and salting are critical.

## Windows Authentication Hashes

### NTLM (NT LAN Manager)

Windows systems store passwords as NTLM hashes. These are particularly vulnerable to rainbow table and dictionary attacks.

**Cracking NTLM hashes:**

```bash
john --format=nt --wordlist=rockyou.txt ntlm_hash.txt
```

### Real-Life Application: Internal Network Penetration Test

During a penetration test, you dump the SAM database from a Windows machine and extract NTLM hashes:

```bash
john --format=nt --wordlist=rockyou.txt sam_hashes.txt
```

Common administrative passwords like "P@ssw0rd" or "Company2024" are often cracked quickly, highlighting the importance of strong password policies.

## Linux Shadow File Hashes

### Understanding /etc/shadow

Linux systems store password hashes in `/etc/shadow`. Cracking these requires both `/etc/passwd` and `/etc/shadow` files.

### Combining passwd and shadow

Use the `unshadow` utility to merge the files:

```bash
unshadow /etc/passwd /etc/shadow > combined.txt
```

This creates a format John can process.

### Cracking Combined Hashes

```bash
john --wordlist=rockyou.txt combined.txt
```

John automatically detects the hash format (usually SHA512 on modern Linux systems).

### Real-Life Application: Post-Compromise Analysis

After gaining initial access to a Linux server, you extract the passwd and shadow files to crack the root password:

```bash
# On attacker machine
unshadow passwd shadow > unshadow.txt
john --wordlist=rockyou.txt unshadow.txt
```

Successfully cracking the root password allows for privilege escalation and lateral movement.

## Single Crack Mode

### Using Username Context

Many systems generate passwords based on usernames. Single Crack Mode exploits this by mangling the username to generate password candidates.

**Syntax:**
```bash
john --single --format=[format] [file with username:hash]
```

### Hash Format for Single Crack Mode

Prepare hashes in the format:
```
username:hash
```

Example:
```
Joker:7bf6d9bb82bed1302f331fc6b816aada
```

### Cracking Command

```bash
john --single --format=md5 joker_hash.txt
```

### Real-Life Application: Social Engineering Insights

During a security assessment, you discover users often create passwords based on their names:
- Username: "Sarah" → Password: "Sarah2024" or "sarah!"
- Username: "Admin" → Password: "Admin123" or "AdminPass"

Single Crack Mode automatically generates these variations by mangling the username.

### How Mangling Works

Common mangling patterns:
- Capitalize first letter: `joker` → `Joker`
- Add numbers: `joker` → `joker1`, `joker2024`
- Add special characters: `joker` → `joker!`, `joker@`
- Reverse: `joker` → `rekoj`

## Custom Rules

### What Are Custom Rules?

Custom rules allow you to exploit **password complexity predictability**—the patterns users follow when creating "complex" passwords.

### Common Password Patterns

Most users follow predictable patterns:
- Capitalize first letter + add number: `Password123`
- Word + year: `Summer2024`
- Word + special character: `Pass@word`

### Custom Rule Syntax

**Basic Rule Components:**
- `A` - Append character
- `[A-Z]` - Capital letters A-Z
- `[0-9]` - Digits 0-9
- `[!@#$]` - Special characters

**Example Rules:**

```
# Add capital letters to end
Az"[A-Z]"

# Add numbers to end
Az"[0-9]"

# Capitalize first letter and add number
Cl AZ"[0-9]"
```

### Applying Custom Rules

```bash
john --wordlist=rockyou.txt --rules=THMRules hash.txt
```

**Flag:** `--rule=RuleName`

### Real-Life Application: Corporate Environment

In a corporate network, password policy requires:
- Minimum 8 characters
- One uppercase letter
- One number

Users predictably generate: `Rockyou1`, `Welcome2024`, `Company99`

A custom rule can automate this pattern:

```
Cl AZ"[0-9]"
```

This rule capitalizes the first letter (C) and appends (A) a digit (Z"[0-9]"), dramatically improving crack rates.

## Cracking Archive Files

### Password-Protected ZIP Files

**Extract hash from ZIP:**

```bash
zip2john secure.zip > zip_hash.txt
```

**Crack the hash:**

```bash
john --wordlist=rockyou.txt zip_hash.txt
```

### Real-Life Application: Incident Response

During forensic analysis, you find a password-protected archive on a suspect's computer:

```bash
zip2john suspicious_archive.zip > hash.txt
john --wordlist=rockyou.txt hash.txt
```

The archive is likely protected with a weak password found in common wordlists.

### Password-Protected RAR Archives

**Extract hash from RAR:**

```bash
rar2john secure.rar > rar_hash.txt
```

**Crack the hash:**

```bash
john --wordlist=rockyou.txt rar_hash.txt
```

**Key Difference:** RAR uses different encryption; use `rar2john` instead of `zip2john`.

## SSH Private Key Cracking

### Understanding SSH Key Encryption

SSH private keys (id_rsa files) can be encrypted with a passphrase for additional security. John can crack these passphrases.

### Extract Hash from SSH Key

```bash
ssh2john id_rsa > ssh_hash.txt
```

**Note:** `ssh2john` is included with Jumbo John.

### Crack the Passphrase

```bash
john --wordlist=rockyou.txt ssh_hash.txt
```

### Real-Life Application: Lateral Movement

After compromising a user account, you find their SSH private key:

```bash
find / -name "id_rsa" 2>/dev/null
```

If the key has a weak passphrase:

```bash
ssh2john ~/.ssh/id_rsa > hash.txt
john --wordlist=rockyou.txt hash.txt
```

Once cracked, you use the passphrase to access the key and move laterally to other systems via SSH.

## Attack Methodology Summary

1. **Identify hash type** - Use hash-identifier tools or online services (hashes.com)
2. **Extract hash** - Use appropriate tool (zip2john, rar2john, ssh2john, unshadow)
3. **Select wordlist** - RockYou is industry standard; consider custom wordlists for specific contexts
4. **Choose mode** - Wordlist, single crack, or custom rules based on target
5. **Execute John** - Run with correct format and parameters
6. **Monitor progress** - John shows cracking progress and displays cracked passwords

## Security Best Practices (Defense)

### For System Administrators

- **Use strong hashing algorithms**: bcrypt, scrypt, or Argon2 instead of MD5/SHA1
- **Implement salting**: Prevents rainbow table attacks
- **Enforce password policies**: Minimum length, complexity requirements
- **Rate limiting**: Limit login attempts to prevent brute force
- **Multi-factor authentication**: Add additional security layer beyond passwords

### Why Weak Hashes Fail

- **MD5/SHA1**: Fast to compute; designed for speed, not security
- **No salt**: Rainbow tables contain pre-computed hashes for all common passwords
- **No stretching**: Allows rapid password guessing

**Modern approach:**
```bash
# Weak (vulnerable to John)
MD5(password) → 5f4dcc3b5aa765d61d8327deb882cf99

# Strong (resistant to cracking)
bcrypt(password, salt, rounds=12) → $2b$12$...
```

## Common Mistakes to Avoid

1. **Wrong hash format** - Verify format before cracking; test with known passwords first
2. **Insufficient wordlist** - RockYou covers 90%+ of weak passwords; custom lists for specific targets
3. **Ignoring case sensitivity** - Some formats are case-sensitive; verify output
4. **Not using GPU acceleration** - If available, use Hashcat instead of CPU-bound John for large-scale cracking
5. **Assuming all hashes are crackable** - Very strong passwords won't crack; time is the limiting factor

## Tools and Resources

- **Hash Identifier**: Identify unknown hash types
- **Hashes.com**: Online hash lookup and cracking service
- **Hashcat**: GPU-accelerated alternative for large-scale operations
- **Custom wordlists**: Combine RockYou with domain-specific terms for better results
