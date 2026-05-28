# Cryptography Notes — Task 1

## Overview
Cryptography is the practice of securing information by transforming it into an unreadable format. It is the backbone of modern cybersecurity — protecting data in transit, at rest, and during authentication.

---

## Core Concepts

| Term | Definition |
|------|-----------|
| **Plaintext** | Original readable data |
| **Ciphertext** | Encrypted, unreadable data |
| **Encryption** | Converting plaintext → ciphertext |
| **Decryption** | Converting ciphertext → plaintext |
| **Key** | Secret value used in encryption/decryption |
| **Cipher** | Algorithm used to encrypt/decrypt |
| **Hash** | One-way fixed-length output from input data |

---

## Types of Cryptography

### 1. Symmetric Encryption
- **Same key** used for both encryption and decryption
- Fast, efficient — good for large data
- Key distribution is a challenge

**Common Algorithms:**
| Algorithm | Key Size | Notes |
|-----------|----------|-------|
| AES | 128, 192, 256-bit | Industry standard, very secure |
| DES | 56-bit | Outdated, broken |
| 3DES | 112/168-bit | DES applied 3 times, deprecated |
| Blowfish | 32–448-bit | Fast, older alternative |
| RC4 | Variable | Stream cipher, now considered weak |

```
Encrypt: plaintext + key → ciphertext
Decrypt: ciphertext + key → plaintext
```

---

### 2. Asymmetric Encryption (Public Key Cryptography)
- Uses a **key pair**: Public key (encrypt) + Private key (decrypt)
- Slower than symmetric, but solves key distribution
- Used in SSL/TLS, SSH, digital signatures

**Common Algorithms:**
| Algorithm | Key Size | Use Case |
|-----------|----------|----------|
| RSA | 2048, 4096-bit | Encryption, digital signatures |
| ECC | 256-bit | Efficient, used in TLS, Bitcoin |
| DSA | 1024–3072-bit | Digital signatures only |
| Diffie-Hellman | Variable | Key exchange protocol |
| ElGamal | Variable | Encryption and signatures |

```
Encrypt: plaintext + recipient's PUBLIC key → ciphertext
Decrypt: ciphertext + recipient's PRIVATE key → plaintext
```

---

### 3. Hashing
- **One-way function** — cannot be reversed
- Same input always produces same output (deterministic)
- Used for: password storage, file integrity, digital signatures

**Common Hash Algorithms:**
| Algorithm | Output Size | Status |
|-----------|-------------|--------|
| MD5 | 128-bit (32 hex) | Broken — collision vulnerable |
| SHA-1 | 160-bit (40 hex) | Deprecated |
| SHA-256 | 256-bit (64 hex) | Secure, widely used |
| SHA-512 | 512-bit (128 hex) | More secure, slower |
| bcrypt | Variable | Password hashing with salt |
| Argon2 | Variable | Modern password hashing |

```bash
echo -n "hello" | md5sum
echo -n "hello" | sha256sum
```

---

## Digital Signatures
- Proves **authenticity** and **integrity** of a message
- Sender signs with their **private key**
- Receiver verifies with sender's **public key**

**Process:**
1. Hash the message
2. Encrypt hash with private key → signature
3. Recipient decrypts signature with public key
4. Compare decrypted hash with message hash

---

## PKI (Public Key Infrastructure)
- Framework for managing digital certificates and keys
- **Certificate Authority (CA)**: Trusted entity that issues certificates
- **Digital Certificate**: Binds a public key to an identity (X.509 format)
- **Certificate Chain**: Root CA → Intermediate CA → End-entity cert

---

## SSL/TLS
- **SSL** (Secure Sockets Layer) — deprecated
- **TLS** (Transport Layer Security) — current standard (TLS 1.2, 1.3)
- Provides: Encryption, Authentication, Integrity

**TLS Handshake (simplified):**
1. Client Hello (supported cipher suites)
2. Server Hello + Certificate
3. Key Exchange (Diffie-Hellman)
4. Session keys derived
5. Encrypted communication begins

---

## Common Encoding Schemes (Not Encryption)

| Scheme | Purpose | Example |
|--------|---------|---------|
| Base64 | Binary-to-text encoding | `SGVsbG8=` |
| Hex | Binary as hexadecimal | `48656c6c6f` |
| URL Encoding | Safe characters in URLs | `Hello%20World` |
| ASCII | Character encoding | `A = 65` |

> **Note**: Encoding is NOT encryption — it's reversible without a key.

---

## Classic Ciphers (Historical)

| Cipher | Type | Description |
|--------|------|-------------|
| Caesar | Substitution | Shift letters by N positions |
| Vigenère | Polyalphabetic | Uses a keyword for shifting |
| ROT13 | Substitution | Caesar cipher with shift of 13 |
| XOR | Bitwise | XOR each byte with key |
| Atbash | Substitution | Reverse alphabet (A↔Z) |

```
Caesar cipher (shift 3): HELLO → KHOOR
ROT13: HELLO → URYYB
```

---

## Key Concepts in Practice

- **Salt**: Random data added to passwords before hashing (prevents rainbow table attacks)
- **IV (Initialization Vector)**: Random value used with block ciphers to ensure unique ciphertext
- **HMAC**: Hash-based Message Authentication Code — combines hashing with a secret key
- **KDF (Key Derivation Function)**: Derives cryptographic keys from passwords (PBKDF2, bcrypt, Argon2)
- **Perfect Forward Secrecy (PFS)**: New session keys generated per session — past sessions stay secure even if key is compromised

---

## Cryptographic Attacks

| Attack | Description |
|--------|-------------|
| Brute Force | Try all possible keys |
| Dictionary Attack | Try common passwords/keys |
| Rainbow Table | Precomputed hash lookup |
| Man-in-the-Middle | Intercept key exchange |
| Replay Attack | Resend captured valid messages |
| Birthday Attack | Exploit hash collision probability |
| Padding Oracle | Exploit padding validation in block ciphers |

---

## Tools

```bash
openssl genrsa -out key.pem 2048          # Generate RSA key
openssl req -new -x509 -key key.pem       # Generate self-signed cert
openssl enc -aes-256-cbc -in file.txt     # Encrypt with AES
gpg --gen-key                             # Generate GPG key pair
gpg --encrypt --recipient user file.txt  # Encrypt file
hashcat -m 0 hash.txt wordlist.txt        # Crack MD5 hash
john --wordlist=rockyou.txt hash.txt      # Password cracking
```
