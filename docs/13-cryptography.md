---
layout: default
title: "Module 13 — Cryptography"
nav_order: 14
---

# Module 13 — Cryptography
{: .no_toc }

Cryptography is the backbone of information security. For ethical hackers, understanding cryptography is not about becoming a mathematician — it is about recognizing when cryptographic implementations are weak, misconfigured, or outright broken. This module covers the theory, tools, attacks, and practical skills you need to assess cryptographic security during penetration tests.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Cryptography Fundamentals

### What Is Cryptography and Why It Matters

Cryptography is the science of securing information by transforming it into an unreadable format that can only be reversed by authorized parties. The word comes from the Greek *kryptos* (hidden) and *graphein* (to write).

For ethical hackers, cryptography matters because:

- **Data protection** — Encryption protects data at rest and in transit. Weak encryption means exposed data.
- **Authentication** — Digital signatures and certificates verify identity. Flawed implementations enable impersonation.
- **Integrity** — Hashing ensures data has not been tampered with. Broken hash functions mean undetectable modifications.
- **Non-repudiation** — Cryptographic signatures prove origin. Poor key management undermines accountability.
- **Attack surface** — Every cryptographic implementation is a potential vulnerability. Misconfigurations, deprecated algorithms, and poor key management are among the most common findings in penetration tests.

### Core Terminology

| Term | Definition |
|------|-----------|
| **Plaintext** | The original, readable data before encryption |
| **Ciphertext** | The encrypted, unreadable output after encryption |
| **Key** | A secret value used by the algorithm to encrypt or decrypt data |
| **Algorithm (Cipher)** | The mathematical procedure that performs encryption/decryption |
| **Key Space** | The total number of possible keys (determines brute-force difficulty) |
| **Nonce** | A number used once — prevents replay attacks and ensures unique ciphertexts |
| **Initialization Vector (IV)** | A random value used to ensure identical plaintexts produce different ciphertexts |
| **Salt** | Random data added to input before hashing — prevents precomputation attacks |

### Encryption vs Encoding vs Hashing vs Obfuscation

These four concepts are frequently confused. Understanding the differences is critical.

| Property | Encryption | Encoding | Hashing | Obfuscation |
|----------|-----------|----------|---------|-------------|
| **Purpose** | Confidentiality | Data representation | Integrity verification | Make code harder to read |
| **Reversible?** | Yes, with key | Yes, no key needed | No (one-way) | Yes, with effort |
| **Key required?** | Yes | No | No | No |
| **Security goal?** | Yes | No | Yes | Weak/No |
| **Example** | AES, RSA | Base64, URL encoding | SHA-256, bcrypt | JavaScript minification |

{: .warning }
> **Common Mistake**: Base64 is NOT encryption. It is encoding. Anyone can decode Base64 without a key. Finding Base64-encoded credentials during a pentest is a finding — the developer likely thought they were "encrypting" data.

```bash
# Base64 is NOT encryption — anyone can decode it
echo -n "admin:password123" | base64
# Output: YWRtaW46cGFzc3dvcmQxMjM=

echo "YWRtaW46cGFzc3dvcmQxMjM=" | base64 -d
# Output: admin:password123
```

### Kerckhoffs's Principle

> "A cryptosystem should be secure even if everything about the system, except the key, is public knowledge."
> — Auguste Kerckhoffs, 1883

This principle is foundational to modern cryptography. It means:

- The security of a system must depend **solely on the secrecy of the key**, not on the secrecy of the algorithm.
- Algorithms should be published, peer-reviewed, and battle-tested.
- AES, RSA, SHA-256 — their specifications are fully public. Their security comes from the mathematical difficulty of breaking them without the key.

Shannon's maxim restates this more bluntly: **"The enemy knows the system."**

### Security Through Obscurity (and Why It Fails)

Security through obscurity is the practice of keeping a system secure by hiding how it works rather than by using proven cryptographic methods.

**Why it fails:**

- Reverse engineering is always possible — attackers have unlimited time and tools.
- Proprietary algorithms rarely survive scrutiny. History is littered with broken "secret" ciphers.
- Once the algorithm leaks (and it always does), the entire system collapses.
- Open algorithms benefit from thousands of researchers trying to break them.

**Real-world examples of obscurity failures:**

| System | What Happened |
|--------|--------------|
| **DVD CSS** | "Secret" encryption broken in days after reverse engineering |
| **GSM A5/1** | Proprietary cipher cracked, enabling cell phone eavesdropping |
| **Mifare Classic** | NXP's secret cipher broken, compromising millions of transit cards |
| **WEP** | Proprietary RC4 implementation with fundamental design flaws |

{: .tip }
> During a pentest, finding custom or proprietary encryption is almost always a vulnerability. Report it and recommend replacing it with standard, well-vetted algorithms.

---

## Symmetric Encryption

### How Symmetric Encryption Works

Symmetric encryption uses the **same key** for both encryption and decryption. Both parties must share this secret key through a secure channel before communicating.

```
Plaintext + Key → [Encryption Algorithm] → Ciphertext
Ciphertext + Key → [Decryption Algorithm] → Plaintext
```

**Advantages:**
- Fast — orders of magnitude faster than asymmetric encryption
- Efficient for large volumes of data
- Simple key management for single-user scenarios

**Disadvantages:**
- Key distribution problem — how do you securely share the key?
- Key management scales poorly — N users need N(N-1)/2 keys for pairwise communication
- No non-repudiation — both parties have the same key

### Block Ciphers vs Stream Ciphers

| Property | Block Cipher | Stream Cipher |
|----------|-------------|---------------|
| **Operation** | Encrypts fixed-size blocks (e.g., 128 bits) | Encrypts one bit/byte at a time |
| **Speed** | Slightly slower | Generally faster |
| **Use case** | File encryption, disk encryption | Real-time communication, streaming |
| **Error propagation** | One corrupted block may affect others | Errors affect only corrupted bits |
| **Examples** | AES, DES, Blowfish | ChaCha20, RC4, Salsa20 |

### AES (Advanced Encryption Standard)

AES is the gold standard for symmetric encryption. It was selected by NIST in 2001 after a public competition, replacing DES. The winning algorithm was Rijndael, designed by Joan Daemen and Vincent Rijmen.

**Key properties:**
- Block size: **128 bits** (always)
- Key sizes: **128, 192, or 256 bits**
- Structure: Substitution-Permutation Network (SPN)
- Rounds: 10 (AES-128), 12 (AES-192), 14 (AES-256)

| Variant | Key Size | Rounds | Security Level | Use Case |
|---------|----------|--------|----------------|----------|
| **AES-128** | 128 bits | 10 | Very high | General purpose, good performance |
| **AES-192** | 192 bits | 12 | Higher | Government classified (SECRET) |
| **AES-256** | 256 bits | 14 | Highest | Top Secret, long-term security |

{: .note }
> AES-128 is still considered secure against all known attacks, including quantum computing (Grover's algorithm reduces it to 64-bit equivalent, still infeasible). AES-256 provides a comfortable margin for future-proofing.

#### Block Cipher Modes of Operation

A block cipher alone only encrypts a single block. Modes of operation define how to encrypt data larger than one block.

**ECB (Electronic Codebook)**

```
Block 1 + Key → Ciphertext Block 1
Block 2 + Key → Ciphertext Block 2
Block 3 + Key → Ciphertext Block 3
```

- Each block is encrypted independently with the same key.
- **Identical plaintext blocks produce identical ciphertext blocks.**
- This is catastrophically insecure for structured data.

{: .warning }
> **The ECB Penguin**: If you encrypt a bitmap image of the Linux penguin (Tux) with ECB mode, the outline of the penguin is clearly visible in the ciphertext because identical color blocks produce identical encrypted blocks. This famous example demonstrates why ECB must never be used for anything beyond single-block encryption.

**CBC (Cipher Block Chaining)**

```
Ciphertext[i] = Encrypt(Plaintext[i] XOR Ciphertext[i-1])
```

- Each plaintext block is XORed with the previous ciphertext block before encryption.
- Requires an Initialization Vector (IV) for the first block.
- Identical plaintexts produce different ciphertexts (due to chaining).
- Widely used but vulnerable to **padding oracle attacks**.

**CBC Padding Oracle Attack (Concept):**
When a server reveals whether decrypted ciphertext has valid padding, an attacker can iteratively decrypt ciphertext byte by byte without knowing the key. The attacker modifies ciphertext bytes, sends them to the server, and uses the "valid/invalid padding" responses as an oracle.

Tools like `PadBuster` automate this attack:

```bash
# PadBuster — automated padding oracle exploitation
padbuster http://target.com/decrypt?data= ENCRYPTED_DATA 16
```

**CTR (Counter Mode)**

```
Ciphertext[i] = Plaintext[i] XOR Encrypt(Nonce || Counter)
```

- Turns a block cipher into a stream cipher.
- Encrypts a counter value and XORs the result with plaintext.
- Parallelizable — much faster than CBC.
- No padding needed.
- **Critical**: Never reuse a nonce/counter combination with the same key.

**GCM (Galois/Counter Mode)**

- Combines CTR mode with authentication (GMAC).
- Provides both **confidentiality and integrity** (Authenticated Encryption with Associated Data — AEAD).
- The recommended mode for most modern applications.
- Used in TLS 1.2 and TLS 1.3.

{: .tip }
> When assessing applications, look for AES-GCM or AES-CTR. If you find ECB or unauthenticated CBC, that is a reportable vulnerability.

### DES and 3DES (Legacy)

**DES (Data Encryption Standard):**
- Block size: 64 bits, Key size: 56 bits
- Adopted as a US government standard in 1977
- **Completely broken** — can be brute-forced in hours with modern hardware
- NIST withdrew DES in 2005

**3DES (Triple DES):**
- Applies DES three times with two or three different keys
- Effective key length: 112 bits (2-key) or 168 bits (3-key)
- **Deprecated** by NIST in 2023 — disallowed after 2023
- Extremely slow compared to AES
- Vulnerable to the Sweet32 birthday attack on 64-bit blocks

{: .warning }
> Finding DES or 3DES in any system is a critical finding. Both are deprecated and must be replaced with AES.

### ChaCha20-Poly1305

ChaCha20-Poly1305 is a modern Authenticated Encryption with Associated Data (AEAD) construction designed by Daniel J. Bernstein.

- **ChaCha20** — Stream cipher (256-bit key, 96-bit nonce, 32-bit counter)
- **Poly1305** — Message Authentication Code (MAC)
- Combined, they provide both confidentiality and integrity

**Why ChaCha20-Poly1305 matters:**
- Faster than AES on devices without hardware AES instructions (mobile, IoT)
- Resistant to timing attacks by design (constant-time operations)
- Used in TLS 1.3, WireGuard, and Signal Protocol
- Google adopted it for HTTPS on Android devices

### Key Management Challenges

The hardest part of symmetric encryption is not the algorithm — it is managing the keys.

| Challenge | Description |
|-----------|-------------|
| **Key distribution** | How do you securely share a symmetric key? |
| **Key storage** | Where do you store keys so they are not compromised? |
| **Key rotation** | How often should keys be changed? |
| **Key revocation** | How do you invalidate a compromised key? |
| **Key escrow** | Should a third party hold a copy of the key? |
| **Key lifecycle** | Generation, distribution, use, rotation, archival, destruction |

### Practical: Encrypting Files with OpenSSL

```bash
# Encrypt a file with AES-256-CBC
openssl enc -aes-256-cbc -salt -in plaintext.txt -out encrypted.bin -pbkdf2
# You will be prompted for a password

# Decrypt the file
openssl enc -aes-256-cbc -d -in encrypted.bin -out decrypted.txt -pbkdf2

# Encrypt with AES-256-GCM (authenticated encryption)
openssl enc -aes-256-gcm -salt -in plaintext.txt -out encrypted.bin -pbkdf2

# Encrypt with a specific key and IV (for scripting — not recommended for passwords)
openssl enc -aes-256-cbc -in plaintext.txt -out encrypted.bin \
  -K 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef \
  -iv 0123456789abcdef0123456789abcdef

# Encode output as base64 for readability
openssl enc -aes-256-cbc -salt -in plaintext.txt -out encrypted.b64 -pbkdf2 -a
```

{: .note }
> The `-pbkdf2` flag tells OpenSSL to use PBKDF2 for key derivation from the password. Without it, OpenSSL uses a weaker (deprecated) key derivation method. Always include `-pbkdf2`.

---

## Asymmetric Encryption

### Public Key / Private Key Concept

Asymmetric encryption uses a **mathematically linked key pair**:

- **Public key** — shared freely, used to encrypt data or verify signatures
- **Private key** — kept secret, used to decrypt data or create signatures

```
Encryption:   Plaintext + Public Key  → Ciphertext
Decryption:   Ciphertext + Private Key → Plaintext

Signing:      Message + Private Key → Signature
Verification: Message + Signature + Public Key → Valid/Invalid
```

The mathematical relationship between the keys means:
- Data encrypted with the public key can **only** be decrypted with the corresponding private key.
- A signature created with the private key can **only** be verified with the corresponding public key.
- **Knowing the public key does not reveal the private key** (this is the core security assumption).

### RSA (Rivest-Shamir-Adleman)

RSA, published in 1977, is the most widely deployed asymmetric algorithm. Its security relies on the difficulty of **factoring large prime numbers**.

**How RSA Works (Simplified):**

1. **Key Generation:**
   - Choose two large prime numbers: *p* and *q*
   - Compute *n = p * q* (modulus)
   - Compute *phi(n) = (p-1)(q-1)*
   - Choose public exponent *e* (commonly 65537)
   - Compute private exponent *d* such that *e * d mod phi(n) = 1*
   - Public key: *(n, e)*, Private key: *(n, d)*

2. **Encryption:** *Ciphertext = Plaintext^e mod n*
3. **Decryption:** *Plaintext = Ciphertext^d mod n*

**RSA Key Sizes:**

| Key Size | Security Level | Status |
|----------|---------------|--------|
| 1024 bits | ~80-bit | **Deprecated** — factorable with modern resources |
| 2048 bits | ~112-bit | Minimum acceptable, standard for most uses |
| 3072 bits | ~128-bit | Recommended for medium-term security |
| 4096 bits | ~140-bit | Strong, recommended for long-term security |

**RSA Weaknesses and Attacks:**

| Attack | Description |
|--------|-------------|
| **Small key factoring** | Keys under 1024 bits can be factored with modern hardware |
| **Textbook RSA** | Raw RSA without padding is vulnerable to mathematical attacks |
| **Bleichenbacher's attack** | PKCS#1 v1.5 padding oracle — practical against many implementations |
| **ROBOT attack** | Return Of Bleichenbacher's Oracle Threat — affected major TLS stacks (2017) |
| **Coppersmith's attack** | Exploits small public exponents or partial key knowledge |
| **Common modulus attack** | Using the same modulus with different exponents leaks plaintext |
| **Wiener's attack** | Exploits small private exponents |
| **Quantum threat** | Shor's algorithm can factor RSA keys — threat from future quantum computers |

{: .warning }
> RSA must always use proper padding (OAEP for encryption, PSS for signatures). Finding textbook RSA or PKCS#1 v1.5 in a system is a significant vulnerability.

### Diffie-Hellman Key Exchange

Diffie-Hellman (DH) allows two parties to establish a shared secret over an insecure channel without ever transmitting the secret itself. Published in 1976, it was the first practical public-key protocol.

**How DH Works (Simplified):**

1. Alice and Bob agree on public parameters: a large prime *p* and generator *g*
2. Alice chooses secret *a*, computes *A = g^a mod p*, sends *A* to Bob
3. Bob chooses secret *b*, computes *B = g^b mod p*, sends *B* to Alice
4. Alice computes shared secret: *s = B^a mod p*
5. Bob computes shared secret: *s = A^b mod p*
6. Both arrive at the same shared secret *s = g^(ab) mod p*

An eavesdropper sees *g*, *p*, *A*, and *B* but cannot compute *s* without solving the **discrete logarithm problem**.

**Perfect Forward Secrecy (PFS):**

When DH is used with **ephemeral keys** (new keys for every session), it provides Perfect Forward Secrecy. This means:

- If a long-term key is compromised in the future, past session keys remain secure.
- Each session has unique keys that are discarded after use.
- PFS is a critical property — look for it when assessing TLS configurations.

{: .tip }
> Cipher suites with `DHE` or `ECDHE` in the name provide Perfect Forward Secrecy. Suites with just `RSA` for key exchange do NOT. TLS 1.3 mandates PFS for all connections.

**ECDH (Elliptic Curve Diffie-Hellman):**

ECDH performs the same key exchange but uses elliptic curve mathematics instead of modular exponentiation. It provides equivalent security with much smaller key sizes and faster computation.

### Elliptic Curve Cryptography (ECC)

ECC is based on the algebraic structure of elliptic curves over finite fields. The security relies on the **Elliptic Curve Discrete Logarithm Problem (ECDLP)**.

**Why ECC:**

| RSA Key Size | ECC Key Size | Security Level |
|-------------|-------------|----------------|
| 1024 bits | 160 bits | 80-bit |
| 2048 bits | 224 bits | 112-bit |
| 3072 bits | 256 bits | 128-bit |
| 7680 bits | 384 bits | 192-bit |
| 15360 bits | 521 bits | 256-bit |

ECC achieves the same security level with dramatically smaller keys, resulting in:
- Faster key generation
- Faster signing and verification
- Smaller certificates and signatures
- Less bandwidth and storage
- Better performance on constrained devices (IoT, mobile)

**Common Curves:**

| Curve | Key Size | Standard | Usage |
|-------|----------|----------|-------|
| **P-256** (secp256r1) | 256 bits | NIST | TLS, most widespread |
| **P-384** (secp384r1) | 384 bits | NIST | Government, high-security |
| **P-521** (secp521r1) | 521 bits | NIST | Maximum NIST security |
| **Curve25519** | 256 bits | Bernstein | Modern, high performance, preferred |
| **secp256k1** | 256 bits | SECG | Bitcoin, blockchain |

{: .note }
> Curve25519 (and its signature variant Ed25519) are increasingly preferred over NIST curves because they were designed to be resistant to implementation errors, have no unexplained constants (eliminating "nothing up my sleeve" concerns about NIST curves), and are faster.

### Digital Signatures

Digital signatures provide three critical security properties:

1. **Authentication** — Proves who created the message
2. **Integrity** — Proves the message was not altered
3. **Non-repudiation** — The signer cannot deny signing

**How Digital Signatures Work:**

```
Signing:
  1. Hash the message → digest
  2. Encrypt digest with private key → signature
  3. Send message + signature

Verification:
  1. Hash the received message → digest
  2. Decrypt signature with public key → original digest
  3. Compare: if digests match → signature is valid
```

**Common Signature Algorithms:**

| Algorithm | Based On | Key Size | Speed | Status |
|-----------|----------|----------|-------|--------|
| **RSA-PSS** | RSA | 2048-4096 bits | Slow | Standard, widely used |
| **ECDSA** | ECC (P-256, etc.) | 256-521 bits | Fast | Widely used in TLS, blockchain |
| **Ed25519** | Curve25519 | 256 bits | Very fast | Modern, recommended, deterministic |
| **Ed448** | Curve448 | 448 bits | Fast | Higher security variant |

{: .tip }
> Ed25519 is deterministic — it does not require a random number during signing. This eliminates an entire class of vulnerabilities (the PlayStation 3 ECDSA hack in 2010 was caused by reusing a random nonce during signing, which leaked the private key).

### Practical: RSA Key Operations with OpenSSL

```bash
# Generate an RSA private key (2048-bit)
openssl genrsa -out private.pem 2048

# Extract the public key
openssl rsa -in private.pem -pubout -out public.pem

# View private key details (shows primes, exponents, modulus)
openssl rsa -in private.pem -text -noout

# Generate a 4096-bit key with AES-256 passphrase protection
openssl genrsa -aes256 -out private_encrypted.pem 4096

# Encrypt a file with the public key
openssl rsautl -encrypt -inkey public.pem -pubin -in secret.txt -out secret.enc
# Note: RSA can only encrypt data smaller than the key size minus padding

# Decrypt with the private key
openssl rsautl -decrypt -inkey private.pem -in secret.enc -out secret_decrypted.txt

# Sign a file
openssl dgst -sha256 -sign private.pem -out signature.bin file.txt

# Verify a signature
openssl dgst -sha256 -verify public.pem -signature signature.bin file.txt

# Generate an ECC key pair (P-256 curve)
openssl ecparam -genkey -name prime256v1 -out ec_private.pem
openssl ec -in ec_private.pem -pubout -out ec_public.pem

# Generate an Ed25519 key pair
openssl genpkey -algorithm Ed25519 -out ed25519_private.pem
openssl pkey -in ed25519_private.pem -pubout -out ed25519_public.pem
```

---

## Hashing

### Properties of Cryptographic Hash Functions

A cryptographic hash function takes arbitrary-length input and produces a fixed-length output (the hash, digest, or fingerprint). Secure hash functions must have these properties:

| Property | Description |
|----------|-------------|
| **One-way (preimage resistance)** | Given a hash *h*, it is computationally infeasible to find any input *m* such that hash(m) = h |
| **Deterministic** | The same input always produces the same hash |
| **Avalanche effect** | Changing a single bit of input changes approximately 50% of the output bits |
| **Collision resistance** | It is infeasible to find two different inputs that produce the same hash |
| **Second preimage resistance** | Given input *m1*, it is infeasible to find a different *m2* such that hash(m1) = hash(m2) |
| **Fixed output length** | Regardless of input size, the output is always the same length |

```bash
# Demonstrating the avalanche effect
echo -n "hello" | sha256sum
# 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824

echo -n "hellp" | sha256sum
# 5f7791746e802d0e11e7a0a3a1b77889ff4d1afe024d55c6b2c36a3e82e531f0
# Completely different output from changing just one character
```

### Common Hash Algorithms

**MD5 (Message Digest 5) — BROKEN**

| Property | Value |
|----------|-------|
| Output | 128 bits (32 hex characters) |
| Status | **Cryptographically broken** |
| Year | 1991 (Ron Rivest) |

- Collision attacks are trivial — two different files with the same MD5 can be generated in seconds.
- Still seen in legacy systems, file checksums (non-security), and unfortunately, password storage.
- Finding MD5-hashed passwords during a pentest is a critical finding.

**SHA-1 (Secure Hash Algorithm 1) — BROKEN**

| Property | Value |
|----------|-------|
| Output | 160 bits (40 hex characters) |
| Status | **Cryptographically broken** |
| Year | 1995 (NSA/NIST) |

- The **SHAttered attack** (2017) demonstrated a practical collision — two different PDF files with the same SHA-1 hash.
- Google and CWI Amsterdam produced the collision using approximately 6,500 years of CPU computation and 110 years of GPU computation.
- Browsers and CAs stopped accepting SHA-1 certificates in 2017.
- Git still uses SHA-1 internally (migrating to SHA-256).

**SHA-2 Family — Currently Secure**

| Variant | Output | Block Size | Status |
|---------|--------|-----------|--------|
| **SHA-224** | 224 bits | 512 bits | Secure (rarely used) |
| **SHA-256** | 256 bits | 512 bits | **Widely used, recommended** |
| **SHA-384** | 384 bits | 1024 bits | Secure |
| **SHA-512** | 512 bits | 1024 bits | Secure, faster on 64-bit systems |

- SHA-256 is the workhorse of modern cryptography — used in TLS, code signing, blockchain, and file integrity verification.
- No practical attacks against any SHA-2 variant.

**SHA-3 (Keccak) — Latest Standard**

| Property | Value |
|----------|-------|
| Output | 224, 256, 384, or 512 bits |
| Structure | Sponge construction (fundamentally different from SHA-2) |
| Year | 2015 (NIST competition winner, designed by Guido Bertoni et al.) |
| Status | Secure, standardized |

- SHA-3 was designed as a backup in case SHA-2 is ever broken.
- Uses a completely different internal structure (sponge construction vs Merkle-Damgard).
- Not widely deployed yet because SHA-2 remains secure.
- Also provides SHAKE128/SHAKE256 — extendable output functions (XOFs).

**BLAKE2 and BLAKE3 — Modern Alternatives**

| Algorithm | Output | Speed | Status |
|-----------|--------|-------|--------|
| **BLAKE2b** | Up to 512 bits | Faster than SHA-256 | Secure, widely used |
| **BLAKE2s** | Up to 256 bits | Optimized for 32-bit | Secure |
| **BLAKE3** | 256 bits | Extremely fast (parallelizable) | Secure, newest |

- BLAKE2 is used in Argon2, WireGuard, and many modern applications.
- BLAKE3 can hash data at speeds limited by memory bandwidth, not CPU.
- Both are excellent choices for non-legacy applications.

### Password Hashing

Password hashing is fundamentally different from general-purpose hashing. A password hash must be:

- **Slow** — deliberately computationally expensive to resist brute force
- **Salted** — unique random salt per password to prevent precomputation
- **Memory-hard** (ideally) — resist GPU/ASIC acceleration

{: .warning }
> **NEVER use MD5, SHA-1, SHA-256, or any general-purpose hash for passwords.** These are designed to be fast — which is exactly what attackers want. A modern GPU can compute billions of SHA-256 hashes per second, making brute force trivial for weak passwords.

**bcrypt**

| Property | Value |
|----------|-------|
| Output | 60 characters (encoded) |
| Salt | 128-bit, automatically generated and embedded |
| Cost factor | Configurable (default 10, each increment doubles time) |
| Max input | 72 bytes |
| Status | **Widely used, well-proven** |

```
$2b$12$LJ3m4ys3Lz/YRMYxKQr0aeJf0P.GMB8XV5qUdK/r9lRsLy0ECwVFG
 ├─┤├┤├────────────────────────┤├──────────────────────────────┤
 alg cost        salt                       hash
```

**scrypt**

| Property | Value |
|----------|-------|
| Design | Memory-hard — requires large amounts of RAM |
| Parameters | N (CPU/memory cost), r (block size), p (parallelism) |
| Advantage | Resists GPU/ASIC attacks due to memory requirements |
| Status | **Good choice, used in some cryptocurrencies** |

**Argon2 — Recommended**

Argon2 won the Password Hashing Competition (PHC) in 2015 and is the recommended algorithm for new applications.

| Variant | Optimized For |
|---------|--------------|
| **Argon2d** | GPU resistance (data-dependent memory access) |
| **Argon2i** | Side-channel resistance (data-independent memory access) |
| **Argon2id** | Hybrid — **recommended variant** |

Parameters: memory cost, time cost (iterations), parallelism degree, salt, hash length.

**PBKDF2 (Password-Based Key Derivation Function 2)**

| Property | Value |
|----------|-------|
| Standard | NIST SP 800-132, RFC 2898 |
| Iterations | Minimum 600,000 recommended (OWASP 2023) |
| Advantage | FIPS compliant, widely supported |
| Disadvantage | Not memory-hard, vulnerable to GPU acceleration |

{: .tip }
> Password hashing recommendation hierarchy: **Argon2id > bcrypt > scrypt > PBKDF2**. If you find passwords stored with MD5, SHA-1, or unsalted SHA-256 during a pentest, this is a critical finding.

### HMAC (Hash-based Message Authentication Code)

HMAC combines a hash function with a secret key to provide both integrity and authentication.

```
HMAC(K, m) = H((K' XOR opad) || H((K' XOR ipad) || m))
```

- Used extensively in API authentication (e.g., AWS Signature V4)
- Used in TLS for record-layer integrity
- Not vulnerable to length extension attacks (unlike raw hash functions)
- Common variants: HMAC-SHA256, HMAC-SHA512

```bash
# Generate HMAC-SHA256
echo -n "message" | openssl dgst -sha256 -hmac "secret_key"

# Verify HMAC (compare output)
echo -n "message" | openssl dgst -sha256 -hmac "secret_key"
```

### Practical: Hash Generation and Identification

```bash
# Generate various hashes
echo -n "password" | md5sum
# 5f4dcc3b5aa765d61d8327deb882cf99

echo -n "password" | sha1sum
# 5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8

echo -n "password" | sha256sum
# 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8

echo -n "password" | sha512sum

# Hash a file
sha256sum important_file.bin
md5sum important_file.bin

# Identify hash type using hashid
hashid '5f4dcc3b5aa765d61d8327deb882cf99'
# Output: MD5, MD4, MD2, NTLM, etc.

hashid '$2b$12$LJ3m4ys3Lz/YRMYxKQr0aeJf0P.GMB8XV5qUdK/r9lRsLy0ECwVFG'
# Output: bcrypt

# hash-identifier (interactive tool in Kali)
hash-identifier

# Using Python hashlib for custom hashing
python3 -c "
import hashlib
print('MD5:   ', hashlib.md5(b'password').hexdigest())
print('SHA1:  ', hashlib.sha1(b'password').hexdigest())
print('SHA256:', hashlib.sha256(b'password').hexdigest())
print('SHA512:', hashlib.sha512(b'password').hexdigest())
"
```

---

## Public Key Infrastructure (PKI)

### Certificate Authorities (CA)

A Certificate Authority (CA) is a trusted entity that issues digital certificates. CAs form the foundation of internet trust.

**CA Hierarchy:**

```
Root CA (offline, air-gapped)
├── Intermediate CA 1
│   ├── End-entity certificate (website)
│   └── End-entity certificate (website)
├── Intermediate CA 2
│   ├── End-entity certificate (email)
│   └── End-entity certificate (code signing)
```

- **Root CAs** are self-signed and pre-installed in operating systems and browsers (the "trust store").
- **Intermediate CAs** are signed by root CAs and sign end-entity certificates.
- This hierarchy allows root CA private keys to remain offline and air-gapped.

**Major CAs:**
- DigiCert, Sectigo (Comodo), GlobalSign, Let's Encrypt (free, automated), IdenTrust, Entrust

### X.509 Certificates

X.509 is the standard format for public key certificates. A certificate binds a public key to an identity.

**Key fields in an X.509 certificate:**

| Field | Description |
|-------|-------------|
| **Version** | v3 (current) |
| **Serial Number** | Unique identifier assigned by the CA |
| **Signature Algorithm** | Algorithm used by CA to sign (e.g., SHA256withRSA) |
| **Issuer** | CA that signed the certificate |
| **Validity** | Not Before / Not After dates |
| **Subject** | Entity the certificate identifies (CN, O, OU, etc.) |
| **Subject Public Key Info** | Public key algorithm and key |
| **Extensions** | Subject Alternative Names (SAN), Key Usage, etc. |
| **Signature** | The CA's digital signature over the certificate |

### Certificate Chains and Trust

Trust is established through a chain:

1. Browser receives the server's certificate.
2. Browser checks: "Is this certificate signed by a trusted CA?"
3. If the signer is an intermediate CA, the browser follows the chain upward.
4. Chain ends at a root CA in the browser's trust store.
5. If the chain is complete and valid, the certificate is trusted.

**Common issues pentesters find:**
- Incomplete certificate chains (missing intermediate certificates)
- Self-signed certificates in production
- Expired certificates
- Certificates with wrong hostnames

### Certificate Signing Requests (CSR)

A CSR is a request sent to a CA to obtain a certificate:

```bash
# Generate a CSR (interactive — prompts for subject details)
openssl req -new -key private.pem -out request.csr

# Generate key and CSR in one command
openssl req -new -newkey rsa:2048 -nodes -keyout key.pem -out request.csr \
  -subj "/C=US/ST=State/L=City/O=Org/CN=example.com"

# View CSR contents
openssl req -in request.csr -text -noout
```

### SSL/TLS Certificates

**Certificate Types by Validation Level:**

| Type | Validation | Visual Indicator | Cost |
|------|-----------|-------------------|------|
| **DV** (Domain Validation) | Domain ownership only | Padlock | Free (Let's Encrypt) |
| **OV** (Organization Validation) | Domain + organization identity | Padlock | $$ |
| **EV** (Extended Validation) | Rigorous identity verification | Padlock (formerly green bar) | $$$ |

**Certificate Types by Scope:**

| Type | Covers | Example |
|------|--------|---------|
| **Single domain** | One exact domain | example.com |
| **Wildcard** | One domain + all subdomains | *.example.com |
| **Multi-domain (SAN)** | Multiple specific domains | example.com, example.org |

### Let's Encrypt and the ACME Protocol

Let's Encrypt is a free, automated, and open Certificate Authority that has revolutionized certificate deployment.

- Issues Domain Validated (DV) certificates at no cost.
- Uses the **ACME protocol** (Automated Certificate Management Environment) for automated issuance and renewal.
- Certificates are valid for 90 days (encourages automation).
- Certbot is the most common ACME client.

```bash
# Issue a certificate with Certbot
sudo certbot certonly --standalone -d example.com

# Renew all certificates
sudo certbot renew

# Test renewal
sudo certbot renew --dry-run
```

### Certificate Transparency Logs

Certificate Transparency (CT) is a system for monitoring and auditing certificate issuance.

- All publicly trusted CAs must log certificates to public CT logs.
- Anyone can search CT logs to find certificates issued for a domain.
- Helps detect misissued or rogue certificates.

```bash
# Search CT logs for certificates issued to a domain
# Using crt.sh (public CT log search)
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq '.[].name_value' | sort -u

# This is an excellent reconnaissance technique for discovering subdomains
```

{: .tip }
> Certificate Transparency logs are a goldmine for reconnaissance. They reveal all subdomains that have had certificates issued, including internal, staging, and development environments that may not be in DNS.

### Certificate Pinning

Certificate pinning restricts which certificates or CAs a client will accept for a specific server, preventing man-in-the-middle attacks even if a CA is compromised.

**Types of pinning:**
- **Certificate pinning** — Pin the exact certificate (must update when certificate rotates)
- **Public key pinning** — Pin the public key (survives certificate renewal if key stays the same)
- **CA pinning** — Pin the CA (allows any certificate from that CA)

**HPKP (HTTP Public Key Pinning):** Deprecated by browsers in 2019 because misconfiguration could permanently lock users out of a site. Replaced by Certificate Transparency and Expect-CT.

**Mobile app pinning:** Still common and recommended for mobile applications. Bypassing pinning is a standard step in mobile pentesting.

```bash
# Bypass SSL pinning on Android with Frida
frida -U -f com.target.app -l ssl_pinning_bypass.js --no-pause

# Bypass with objection (built on Frida)
objection -g com.target.app explore
# Inside objection:
android sslpinning disable
```

### Practical: Certificate Examination

```bash
# Examine a live server's certificate
openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -text -noout

# Check certificate expiration
openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Check the full certificate chain
openssl s_client -connect example.com:443 -showcerts 2>/dev/null

# Generate a self-signed certificate (for testing)
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes \
  -subj "/C=US/ST=State/L=City/O=Test/CN=localhost"

# Verify a certificate against a CA bundle
openssl verify -CAfile ca_bundle.pem certificate.pem

# Check if a certificate matches a private key
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in key.pem | openssl md5
# If the MD5 outputs match, the certificate and key are a pair
```

---

## SSL/TLS

### TLS Handshake Explained

The TLS handshake establishes an encrypted connection between client and server. Here is the TLS 1.2 full handshake, step by step:

**TLS 1.2 Handshake (RSA Key Exchange):**

```
Client                                      Server
  │                                           │
  ├─── ClientHello ──────────────────────────►│  (1)
  │    • TLS version                          │
  │    • Random number (client_random)        │
  │    • Cipher suites supported              │
  │    • Compression methods                  │
  │                                           │
  │◄──────────────────────── ServerHello ─────┤  (2)
  │    • Selected TLS version                 │
  │    • Random number (server_random)        │
  │    • Selected cipher suite                │
  │                                           │
  │◄──────────────────────── Certificate ─────┤  (3)
  │    • Server's X.509 certificate chain     │
  │                                           │
  │◄──────────────────── ServerHelloDone ─────┤  (4)
  │                                           │
  ├─── ClientKeyExchange ────────────────────►│  (5)
  │    • Pre-master secret (encrypted with    │
  │      server's public key)                 │
  │                                           │
  ├─── ChangeCipherSpec ─────────────────────►│  (6)
  │    (switching to encrypted communication) │
  │                                           │
  ├─── Finished ─────────────────────────────►│  (7)
  │    (encrypted verification hash)          │
  │                                           │
  │◄─────────────────── ChangeCipherSpec ─────┤  (8)
  │                                           │
  │◄──────────────────────────── Finished ────┤  (9)
  │                                           │
  │◄═══════ Encrypted Application Data ══════►│  (10)
```

Both sides derive the **master secret** from: pre-master secret + client_random + server_random. Session keys (encryption key, MAC key, IV) are derived from the master secret.

### TLS 1.2 vs TLS 1.3 Differences

TLS 1.3, finalized in 2018 (RFC 8446), represents a major overhaul.

| Feature | TLS 1.2 | TLS 1.3 |
|---------|---------|---------|
| **Handshake round-trips** | 2 RTT | 1 RTT (0-RTT with resumption) |
| **Key exchange** | RSA, DHE, ECDHE | **ECDHE or DHE only** (PFS mandatory) |
| **Cipher suites** | ~37 supported | **5 supported** (simplified) |
| **Symmetric ciphers** | AES-CBC, AES-GCM, RC4, 3DES | **AES-GCM, AES-CCM, ChaCha20-Poly1305 only** |
| **Hash functions** | MD5, SHA-1, SHA-256 | **SHA-256, SHA-384 only** |
| **RSA key exchange** | Supported | **Removed** (no PFS) |
| **Compression** | Supported | **Removed** (CRIME attack) |
| **Renegotiation** | Supported | **Removed** |
| **0-RTT resumption** | No | Yes (with replay risk) |
| **Encrypted handshake** | Partial | Most messages encrypted |

{: .note }
> TLS 1.3 eliminated all known-weak algorithms and features. Finding a server that only supports TLS 1.0 or 1.1 is a finding — both are deprecated by RFC 8996 (2021). TLS 1.2 is acceptable with proper cipher suites. TLS 1.3 is preferred.

### Cipher Suites and Their Components

A cipher suite defines the algorithms used for each aspect of the TLS connection.

**TLS 1.2 cipher suite format:**

```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
 │    │     │        │   │   │    │
 │    │     │        │   │   │    └── PRF hash (key derivation)
 │    │     │        │   │   └─────── AEAD mode
 │    │     │        │   └─────────── Key size
 │    │     │        └─────────────── Symmetric cipher
 │    │     └──────────────────────── Authentication
 │    └────────────────────────────── Key exchange
 └─────────────────────────────────── Protocol
```

**TLS 1.3 cipher suite format (simplified):**

```
TLS_AES_256_GCM_SHA384
```

Key exchange and authentication are negotiated separately in TLS 1.3 (via extensions), so the cipher suite only specifies the symmetric cipher and hash.

**Recommended cipher suites (TLS 1.2):**

```
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
```

### SSL/TLS Attacks

| Attack | Year | Description | Affected |
|--------|------|-------------|----------|
| **BEAST** | 2011 | Exploits predictable IVs in TLS 1.0 CBC mode | TLS 1.0 CBC |
| **CRIME** | 2012 | Exploits TLS compression to leak session cookies | TLS with compression |
| **BREACH** | 2013 | Like CRIME but targets HTTP compression | HTTP compression + TLS |
| **Heartbleed** | 2014 | OpenSSL buffer over-read — leaks server memory including private keys | OpenSSL 1.0.1-1.0.1f |
| **POODLE** | 2014 | Padding oracle attack on SSL 3.0 CBC mode | SSL 3.0 |
| **FREAK** | 2015 | Forces export-grade RSA keys (512-bit) | Export cipher suites |
| **Logjam** | 2015 | Forces export-grade DH keys (512-bit) | Export DH |
| **DROWN** | 2016 | Bleichenbacher attack via SSLv2 to decrypt TLS sessions | Servers with SSLv2 enabled |
| **ROBOT** | 2017 | Return of Bleichenbacher's Oracle Threat — RSA key exchange | RSA key exchange in TLS |
| **ROCA** | 2017 | Weak RSA key generation in Infineon chips | Specific hardware |

**Heartbleed (CVE-2014-0160) — In Detail:**

Heartbleed was a buffer over-read vulnerability in OpenSSL's TLS heartbeat extension. An attacker could request up to 64KB of server memory per heartbeat request, potentially leaking:

- Private keys
- Session cookies
- Passwords
- User data in memory

```bash
# Test for Heartbleed
nmap -p 443 --script ssl-heartbleed target.com

# Using Metasploit
msfconsole -q -x "use auxiliary/scanner/ssl/openssl_heartbleed; set RHOSTS target.com; run"
```

**Downgrade Attacks:**

An attacker forces the client and server to negotiate a weaker protocol version or cipher suite than they would normally use. TLS 1.3 includes a downgrade protection mechanism (a sentinel value in the server random field).

### Testing SSL/TLS

```bash
# sslscan — fast SSL/TLS scanner
sslscan target.com
# Shows: supported protocols, cipher suites, certificate details, vulnerabilities

# testssl.sh — comprehensive TLS testing
testssl.sh target.com
# Tests for all known vulnerabilities, rates the configuration

# testssl.sh with specific checks
testssl.sh --vulnerable target.com          # Check known vulnerabilities
testssl.sh --protocols target.com           # Check supported protocols
testssl.sh --ciphers target.com             # Check supported ciphers
testssl.sh --headers target.com             # Check security headers

# Nmap SSL scripts
nmap --script ssl-enum-ciphers -p 443 target.com
nmap --script ssl-cert -p 443 target.com
nmap --script ssl-heartbleed -p 443 target.com
nmap --script ssl-poodle -p 443 target.com
nmap --script ssl-dh-params -p 443 target.com

# OpenSSL manual testing
# Check if a specific protocol version is supported
openssl s_client -connect target.com:443 -tls1
openssl s_client -connect target.com:443 -tls1_1
openssl s_client -connect target.com:443 -tls1_2
openssl s_client -connect target.com:443 -tls1_3

# Check specific cipher support
openssl s_client -connect target.com:443 -cipher 'RC4-SHA'
openssl s_client -connect target.com:443 -cipher 'NULL'

# SSL Labs (web-based, most thorough)
# https://www.ssllabs.com/ssltest/
```

{: .tip }
> For penetration test reports, use testssl.sh for the most comprehensive output. For quick checks, sslscan is faster. Always test from the command line rather than relying solely on web-based tools, as some internal targets may not be accessible from the internet.

---

## Cryptographic Attacks

### Brute Force (Key Space Exhaustion)

A brute-force attack tries every possible key until the correct one is found.

| Key Size | Possible Keys | Time at 10^12 guesses/sec |
|----------|--------------|---------------------------|
| 40 bits | ~10^12 | Seconds |
| 56 bits (DES) | ~7.2 x 10^16 | Hours |
| 64 bits | ~1.8 x 10^19 | ~213 days |
| 128 bits (AES-128) | ~3.4 x 10^38 | ~10^13 years |
| 256 bits (AES-256) | ~1.2 x 10^77 | Longer than the age of the universe |

{: .note }
> AES-128 and above are immune to brute force with any foreseeable technology. Attacks against AES target implementation flaws, not the algorithm itself.

### Dictionary Attacks

Instead of trying every possible key, dictionary attacks try a list of likely passwords or keys:

```bash
# Dictionary attack against hashes with John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Dictionary attack with Hashcat (GPU-accelerated)
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

### Rainbow Tables and Why Salting Defeats Them

A rainbow table is a precomputed lookup table mapping hashes back to plaintexts. Instead of hashing every possible password at attack time, the hashes are precomputed once and stored.

**How rainbow tables work:**
1. Precompute hash(password) for millions/billions of passwords
2. Store the password-hash pairs in a searchable structure
3. Look up captured hashes to find passwords instantly

**Why salting defeats rainbow tables:**
- A salt is a random value added to each password before hashing: `hash(salt + password)`
- Each user has a unique salt — the same password produces different hashes
- An attacker would need a separate rainbow table for every possible salt value
- With a 128-bit salt, the attacker needs 2^128 separate rainbow tables — completely infeasible

```bash
# Rainbow table generation (for educational purposes)
rtgen md5 loweralpha 1 7 0 3800 33554432 0

# Rainbow table lookup
rcrack /path/to/tables -h 5f4dcc3b5aa765d61d8327deb882cf99
```

### Birthday Attack (Collision Finding)

The birthday attack exploits the birthday paradox: in a group of 23 people, there is a >50% chance two share a birthday.

Applied to hashing:
- For an *n*-bit hash, a collision can be found in approximately **2^(n/2)** operations (not 2^n).
- MD5 (128-bit) has collision resistance of only 2^64 — feasible for modern computers.
- SHA-256 (256-bit) has collision resistance of 2^128 — still infeasible.

This is why hash output lengths must be at least double the desired security level.

### Side-Channel Attacks

Side-channel attacks exploit physical characteristics of the implementation rather than mathematical weaknesses in the algorithm.

| Type | What It Measures | Example |
|------|-----------------|---------|
| **Timing** | Time taken to perform operations | Comparing hash values byte by byte takes variable time |
| **Power analysis** | Electrical power consumption | Simple Power Analysis (SPA), Differential Power Analysis (DPA) |
| **Electromagnetic** | EM emissions during computation | TEMPEST attacks on displays, RSA key extraction |
| **Cache-based** | CPU cache access patterns | Flush+Reload, Prime+Probe attacks |
| **Acoustic** | Sound emitted by hardware | RSA key extraction from laptop sounds (demonstrated by researchers) |

**Mitigations:**
- Constant-time implementations (no data-dependent branches)
- Blinding (randomize intermediate values)
- Hardware countermeasures (noise injection, power regulation)

### Padding Oracle Attacks

A padding oracle attack exploits a system that reveals whether decrypted data has valid padding.

**How it works:**
1. Attacker captures a CBC-mode ciphertext.
2. Attacker modifies a byte in a ciphertext block and sends it to the server.
3. Server responds differently for valid vs invalid padding.
4. Attacker uses these responses to deduce the plaintext byte by byte.

**Tools:**
```bash
# PadBuster — automated padding oracle exploitation
padbuster http://target.com/vulnerable?data= CIPHERTEXT_HERE 16 \
  -encoding 0 -error "Invalid padding"
```

{: .warning }
> Padding oracle attacks are practical and devastating. They can decrypt entire messages and forge valid ciphertexts. CBC mode is particularly susceptible. AEAD modes (GCM, ChaCha20-Poly1305) are immune because they detect any ciphertext modification.

### Length Extension Attacks

Length extension attacks exploit Merkle-Damgard hash functions (MD5, SHA-1, SHA-256) when used for message authentication as `hash(secret || message)`.

**How it works:**
1. Attacker knows `hash(secret || message)` and the length of `secret || message`.
2. Without knowing the secret, the attacker can compute `hash(secret || message || padding || extension)`.
3. This allows appending data to the message with a valid hash.

**Affected:** MD5, SHA-1, SHA-256, SHA-512 when used as `hash(key || message)`.

**Not affected:** HMAC, SHA-3, BLAKE2 (truncated), BLAKE3.

```bash
# HashPump — length extension attack tool
hashpump -s ORIGINAL_HASH -d "original_data" -a "appended_data" -k KEY_LENGTH
```

{: .tip }
> Always use HMAC for message authentication, never raw `hash(key || message)`. HMAC is specifically designed to prevent length extension attacks.

### Man-in-the-Middle (Certificate Spoofing)

An attacker intercepts TLS connections by presenting a forged certificate:

1. Attacker positions themselves between client and server (ARP spoofing, DNS spoofing, etc.)
2. Attacker presents their own certificate to the client
3. If the client does not properly validate the certificate, the connection succeeds
4. Attacker decrypts traffic, optionally re-encrypts and forwards to the real server

**Tools:**
```bash
# mitmproxy — interactive HTTPS proxy
mitmproxy

# Bettercap — MITM framework
bettercap -T target_ip -X --proxy --proxy-https

# sslstrip — downgrade HTTPS to HTTP
sslstrip -l 8080
```

### Downgrade Attacks

Force a connection to use a weaker protocol or cipher:

```bash
# Test for protocol downgrade vulnerability
openssl s_client -connect target.com:443 -ssl3     # Force SSL 3.0
openssl s_client -connect target.com:443 -tls1      # Force TLS 1.0

# Test for EXPORT cipher acceptance
openssl s_client -connect target.com:443 -cipher EXPORT
```

### Cryptanalysis Basics

| Technique | Description |
|-----------|-------------|
| **Differential cryptanalysis** | Studies how differences in input affect differences in output |
| **Linear cryptanalysis** | Finds linear approximations of the cipher's behavior |
| **Algebraic attacks** | Represents the cipher as a system of equations and solves |
| **Related-key attacks** | Exploits mathematical relationships between different keys |
| **Known-plaintext attack** | Attacker has pairs of plaintext and corresponding ciphertext |
| **Chosen-plaintext attack** | Attacker can encrypt arbitrary plaintexts |
| **Chosen-ciphertext attack** | Attacker can decrypt arbitrary ciphertexts |

---

## Steganography

### Hiding Data in Plain Sight

Steganography is the practice of hiding secret data within ordinary, non-secret data. Unlike encryption (which makes data unreadable), steganography makes data invisible.

**Common carriers:**
- **Images** — Modify least significant bits (LSB) of pixel data
- **Audio** — Hide data in inaudible frequency ranges or LSBs of samples
- **Video** — Embed data in individual frames
- **Text** — Zero-width characters, whitespace manipulation, word substitution
- **Network** — Covert channels in protocol headers, timing patterns
- **File systems** — Slack space, alternate data streams (NTFS ADS)

### Steganography Tools

**steghide — Embed and Extract Data in Images/Audio**

```bash
# Embed a secret file in an image
steghide embed -cf cover_image.jpg -ef secret.txt
# Prompts for passphrase

# Embed without passphrase
steghide embed -cf cover_image.jpg -ef secret.txt -p ""

# Extract hidden data
steghide extract -sf stego_image.jpg
# Prompts for passphrase

# Get info about a stego file
steghide info stego_image.jpg

# Supported formats: JPEG, BMP, WAV, AU
```

**stegseek — Fast Steghide Brute-Force**

```bash
# Brute-force steghide passphrase with wordlist
stegseek stego_image.jpg /usr/share/wordlists/rockyou.txt

# Crack without specifying wordlist (uses built-in rockyou)
stegseek stego_image.jpg

# Output: extracted file and passphrase
# stegseek is extremely fast — can try millions of passwords per second
```

**binwalk — Firmware and File Analysis**

```bash
# Scan for embedded files and data
binwalk suspicious_file.png

# Extract embedded files
binwalk -e suspicious_file.png

# Recursive extraction
binwalk -eM suspicious_file.png

# Scan for specific signatures
binwalk -R "\x89PNG" firmware.bin

# Entropy analysis (high entropy = likely encrypted/compressed data)
binwalk -E suspicious_file.png
```

**exiftool — Metadata Analysis**

```bash
# View all metadata
exiftool image.jpg

# Look for suspicious metadata fields
exiftool -a -G1 image.jpg

# Remove all metadata (for sanitizing files before sharing)
exiftool -all= image.jpg

# Check for GPS coordinates (privacy concern)
exiftool -gps:all image.jpg

# Metadata can hide data in comment fields, user data, etc.
```

**zsteg — PNG/BMP Steganography Detection**

```bash
# Scan PNG for hidden data across all channels and bit planes
zsteg image.png

# Check all possible methods
zsteg -a image.png

# Extract data from a specific channel/bit
zsteg -e "b1,rgb,lsb,xy" image.png

# Verbose output
zsteg -v image.png
```

**Stegsolve — Visual Analysis (GUI)**

Stegsolve is a Java application that provides visual analysis of images:
- View individual color planes (R, G, B) at each bit level
- Apply various filters and transformations
- Compare images (XOR, AND, OR)
- Extract data from specific bit planes
- View frame-by-frame for animated GIFs

```bash
# Run Stegsolve
java -jar Stegsolve.jar
```

**Additional Steganography Tools:**

```bash
# OpenStego — digital watermarking and data hiding
openstego embed -mf secret.txt -cf cover.png -sf stego.png

# snow — whitespace steganography (hides data in spaces/tabs)
snow -C -m "hidden message" -p "password" cover.txt stego.txt

# Stegsnow extraction
snow -C -p "password" stego.txt

# Sonic Visualiser — audio steganography analysis
# Spectrogram view can reveal hidden images in audio

# Audacity — manual audio analysis
# View spectrogram for hidden messages in frequency domain
```

### Detection Techniques

| Technique | Purpose | Tools |
|-----------|---------|-------|
| **Visual inspection** | Compare original and stego images side by side | Stegsolve |
| **Statistical analysis** | Detect anomalies in pixel distribution | Chi-square analysis, RS analysis |
| **File format analysis** | Look for appended data after file terminator | binwalk, xxd, hexdump |
| **Metadata inspection** | Check for unusual metadata entries | exiftool |
| **Entropy analysis** | High entropy regions may indicate hidden data | binwalk -E, ent |
| **LSB analysis** | Check least significant bits for patterns | zsteg, Stegsolve |
| **Comparison** | Compare suspicious file to original (if available) | diff, compare (ImageMagick) |

---

## Cryptography in Practice for Pentesters

### Cracking Hashes Found During Engagements

During penetration tests, you will encounter hashes from databases, configuration files, SAM files, and network captures. Here is the workflow:

**Step 1: Identify the hash type**

```bash
# hashid
hashid '5f4dcc3b5aa765d61d8327deb882cf99'
# [+] MD5

hashid '$2b$12$LJ3m4ys3...'
# [+] bcrypt

# hash-identifier (Kali)
hash-identifier
```

**Step 2: Select the correct Hashcat mode**

| Hash Type | Hashcat Mode | Example |
|-----------|-------------|---------|
| MD5 | -m 0 | 5f4dcc3b5aa765d61d8327deb882cf99 |
| SHA-1 | -m 100 | 5baa61e4c9b93f3f... |
| SHA-256 | -m 1400 | 5e884898da28047... |
| SHA-512 | -m 1700 | ... |
| NTLM | -m 1000 | 32ed87bdb5fdc5e9... |
| NetNTLMv2 | -m 5600 | admin::DOMAIN:... |
| bcrypt | -m 3200 | $2b$12$... |
| MD5crypt | -m 500 | $1$... |
| SHA512crypt | -m 1800 | $6$... |
| Kerberos TGS (RC4) | -m 13100 | $krb5tgs$23$*... |
| Kerberos AS-REP | -m 18200 | $krb5asrep$23$... |
| WPA/WPA2 | -m 22000 | (PMKID/EAPOL) |

**Step 3: Crack with Hashcat or John the Ripper**

```bash
# Hashcat — GPU-accelerated hash cracking
# Dictionary attack
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt

# Dictionary + rules (mutations)
hashcat -m 0 -a 0 hashes.txt rockyou.txt -r rules/best64.rule

# Brute force (mask attack)
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a?a?a

# Mask syntax:
# ?l = lowercase  ?u = uppercase  ?d = digit
# ?s = special    ?a = all        ?b = binary

# Show cracked passwords
hashcat -m 0 hashes.txt --show

# John the Ripper — CPU-based (also supports GPU)
john --wordlist=rockyou.txt hashes.txt
john --rules --wordlist=rockyou.txt hashes.txt
john --incremental hashes.txt

# Show cracked passwords
john --show hashes.txt
```

### Identifying and Exploiting Weak Crypto Implementations

**Common findings during pentests:**

| Finding | Severity | Recommendation |
|---------|----------|----------------|
| MD5/SHA-1 password hashing | Critical | Use Argon2id or bcrypt |
| Unsalted password hashes | Critical | Add unique random salt per password |
| ECB mode encryption | High | Use GCM or CTR mode |
| Hardcoded encryption keys | Critical | Use key management service |
| Custom/proprietary encryption | High | Replace with standard algorithms |
| Weak random number generation | High | Use cryptographically secure PRNG |
| SSL 2.0/3.0 or TLS 1.0/1.1 | Medium-High | Upgrade to TLS 1.2+ |
| Self-signed certificates in production | Medium | Use CA-signed certificates |
| Base64 "encryption" | High | Implement actual encryption |
| Static IVs or nonces | High | Use random IVs/nonces per operation |

### SSL/TLS Assessment as Part of Pentests

A standard pentest SSL/TLS assessment checklist:

```bash
# 1. Check supported protocol versions
testssl.sh --protocols target.com

# 2. Check for weak cipher suites
testssl.sh --ciphers target.com

# 3. Check for known vulnerabilities
testssl.sh --vulnerable target.com

# 4. Check certificate validity
testssl.sh --server-defaults target.com

# 5. Check for HSTS and other headers
testssl.sh --headers target.com

# 6. Full comprehensive test
testssl.sh --full target.com

# 7. Save results in multiple formats
testssl.sh --json --html --csv target.com
```

### Encrypted Communications for Secure Reporting

Pentesters handle extremely sensitive data — credentials, vulnerabilities, and network diagrams. Secure communication is essential.

```bash
# GPG/PGP for encrypting pentest reports

# Generate your GPG key pair
gpg --full-generate-key
# Select RSA and RSA, 4096 bits, set expiration

# Export your public key (share with client)
gpg --export --armor your@email.com > your_public_key.asc

# Import client's public key
gpg --import client_public_key.asc

# Encrypt the pentest report for the client
gpg --encrypt --recipient client@company.com pentest_report.pdf
# Creates pentest_report.pdf.gpg

# Encrypt and sign (recommended — proves authenticity)
gpg --encrypt --sign --recipient client@company.com pentest_report.pdf

# Decrypt a received file
gpg --decrypt encrypted_file.gpg > decrypted_file.pdf

# Sign a file (without encrypting)
gpg --sign report.pdf
# Creates report.pdf.gpg (binary signature)

gpg --clearsign report.txt
# Creates report.txt.asc (readable text with inline signature)

gpg --detach-sign report.pdf
# Creates report.pdf.sig (separate signature file)

# Verify a signature
gpg --verify report.pdf.sig report.pdf

# List keys in your keyring
gpg --list-keys
gpg --list-secret-keys

# Encrypt with symmetric key (password-based, no key pair needed)
gpg --symmetric --cipher-algo AES256 sensitive_file.txt
```

{: .warning }
> Always encrypt pentest reports and findings before transmitting them. An unencrypted pentest report in transit is a vulnerability itself — it contains all the information an attacker needs.

---

## Hands-On Exercises

### Exercise 1: Hash Identification and Cracking

**Objective:** Identify hash types and crack them using John the Ripper and Hashcat.

**Setup:**
```bash
# Create a file with various hashes to crack
cat << 'EOF' > hashes_to_crack.txt
5f4dcc3b5aa765d61d8327deb882cf99
5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8
5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8
$1$abc$yzqAxwzHTZ60Jltz0L9Ul.
$6$rounds=5000$saltsalt$HX1ux...
EOF
```

**Tasks:**
1. Identify each hash type using `hashid` or `hash-identifier`.
2. Crack the MD5 hash using John the Ripper with the rockyou wordlist.
3. Crack the SHA-256 hash using Hashcat with a dictionary attack.
4. Attempt a rule-based attack using Hashcat's `best64.rule`.
5. Document which hashes cracked and which did not, and explain why.

**Expected skills practiced:**
- Hash identification
- Tool selection (John vs Hashcat)
- Wordlist and rule-based attacks
- Understanding hash strength differences

---

### Exercise 2: SSL/TLS Assessment of a Test Server

**Objective:** Perform a comprehensive SSL/TLS assessment and generate a findings report.

**Setup:**
```bash
# Use a deliberately vulnerable test server or set up your own
# HackTheBox, TryHackMe, or a local test server with weak TLS configuration

# Install testssl.sh if not already available
git clone --depth 1 https://github.com/drwetter/testssl.sh.git
```

**Tasks:**
1. Run `sslscan` against the target and document the supported protocols and ciphers.
2. Run `testssl.sh --full` and save the output as HTML and JSON.
3. Use `nmap --script ssl-enum-ciphers` to cross-validate findings.
4. Manually test for specific vulnerabilities (Heartbleed, POODLE) using OpenSSL and Nmap scripts.
5. Write a findings summary with severity ratings and remediation recommendations.

**Expected skills practiced:**
- Multi-tool validation of SSL/TLS configurations
- Vulnerability identification and severity rating
- Report writing for cryptographic findings

---

### Exercise 3: Encrypt and Decrypt Files with OpenSSL

**Objective:** Practice symmetric and asymmetric encryption workflows with OpenSSL.

**Tasks:**
1. Create a text file with sensitive content.
2. Encrypt it with AES-256-CBC using a password.
3. Decrypt it and verify the contents match.
4. Generate an RSA key pair (2048-bit).
5. Encrypt a small file with the public key and decrypt with the private key.
6. Sign a file with the private key and verify the signature with the public key.
7. Generate an ECC key pair and repeat the signing/verification.
8. Generate a self-signed certificate and examine its contents.

```bash
# Step 1: Create test file
echo "This is top secret pentest data." > secret.txt

# Step 2: Symmetric encryption
openssl enc -aes-256-cbc -salt -in secret.txt -out secret.enc -pbkdf2

# Step 3: Decrypt and verify
openssl enc -aes-256-cbc -d -in secret.enc -out secret_decrypted.txt -pbkdf2
diff secret.txt secret_decrypted.txt

# Step 4: Generate RSA key pair
openssl genrsa -out rsa_private.pem 2048
openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem

# Step 5: Asymmetric encrypt/decrypt
openssl rsautl -encrypt -inkey rsa_public.pem -pubin -in secret.txt -out secret_rsa.enc
openssl rsautl -decrypt -inkey rsa_private.pem -in secret_rsa.enc -out secret_rsa_dec.txt
diff secret.txt secret_rsa_dec.txt

# Step 6: Sign and verify
openssl dgst -sha256 -sign rsa_private.pem -out file.sig secret.txt
openssl dgst -sha256 -verify rsa_public.pem -signature file.sig secret.txt

# Step 7: ECC operations
openssl ecparam -genkey -name prime256v1 -out ec_private.pem
openssl ec -in ec_private.pem -pubout -out ec_public.pem
openssl dgst -sha256 -sign ec_private.pem -out file_ec.sig secret.txt
openssl dgst -sha256 -verify ec_public.pem -signature file_ec.sig secret.txt

# Step 8: Self-signed certificate
openssl req -x509 -newkey rsa:4096 -keyout cert_key.pem -out cert.pem -days 365 -nodes \
  -subj "/C=US/ST=Test/L=Lab/O=PentestLab/CN=testlab.local"
openssl x509 -in cert.pem -text -noout
```

**Expected skills practiced:**
- Symmetric encryption with AES
- RSA key generation and usage
- Digital signatures
- Certificate generation and examination

---

### Exercise 4: GPG Key Generation and Encrypted Communication

**Objective:** Set up GPG encryption and simulate secure communication between two parties.

**Tasks:**
1. Generate a GPG key pair for yourself.
2. Export your public key.
3. Create a second key pair (simulating a client).
4. Import each other's public keys.
5. Encrypt a pentest report for the "client."
6. "As the client," decrypt the report.
7. Sign a document and verify the signature.
8. Encrypt and sign a file simultaneously.

```bash
# Step 1: Generate your GPG key
gpg --batch --gen-key << 'KEYEOF'
Key-Type: RSA
Key-Length: 4096
Subkey-Type: RSA
Subkey-Length: 4096
Name-Real: Pentester
Name-Email: pentester@lab.local
Expire-Date: 1y
%no-protection
%commit
KEYEOF

# Step 2: Export public key
gpg --export --armor pentester@lab.local > pentester_pub.asc

# Step 3: Generate client key
gpg --batch --gen-key << 'KEYEOF'
Key-Type: RSA
Key-Length: 4096
Subkey-Type: RSA
Subkey-Length: 4096
Name-Real: Client
Name-Email: client@company.local
Expire-Date: 1y
%no-protection
%commit
KEYEOF

# Step 4: Export and import (in real scenario, exchange via secure channel)
gpg --export --armor client@company.local > client_pub.asc
# Keys are already in the same keyring for this lab

# Step 5: Encrypt report for client
echo "PENTEST REPORT: 3 critical vulnerabilities found." > report.txt
gpg --encrypt --recipient client@company.local report.txt

# Step 6: Decrypt as client
gpg --decrypt report.txt.gpg

# Step 7: Sign and verify
gpg --local-user pentester@lab.local --detach-sign report.txt
gpg --verify report.txt.sig report.txt

# Step 8: Encrypt and sign
gpg --encrypt --sign --local-user pentester@lab.local \
  --recipient client@company.local report.txt
```

**Expected skills practiced:**
- GPG key management
- Encrypted file exchange workflow
- Digital signatures with GPG
- Simulating secure pentester-client communication

---

### Exercise 5: Steganography Challenge

**Objective:** Hide data in images and then find hidden data in prepared challenge files.

**Tasks:**
1. Create a cover image and hide a secret message using steghide.
2. Extract the hidden message from your stego image.
3. Use stegseek to brute-force a password-protected stego image.
4. Analyze a suspicious PNG file with zsteg.
5. Use binwalk to search for embedded files in a binary.
6. Check image metadata with exiftool for hidden information.

```bash
# Step 1: Hide data with steghide
echo "FLAG{steganography_master}" > flag.txt
steghide embed -cf cover.jpg -ef flag.txt -p "mysecretpass"

# Step 2: Extract hidden data
steghide extract -sf cover.jpg -p "mysecretpass"
cat flag.txt

# Step 3: Brute-force with stegseek
steghide embed -cf challenge.jpg -ef hidden.txt -p "dragon"
stegseek challenge.jpg /usr/share/wordlists/rockyou.txt

# Step 4: Analyze PNG with zsteg
zsteg -a challenge.png

# Step 5: Analyze with binwalk
binwalk suspicious_file.png
binwalk -e suspicious_file.png

# Step 6: Metadata analysis
exiftool challenge.jpg
# Look for unusual Comment, UserComment, or custom fields
```

**Expected skills practiced:**
- Steganographic embedding and extraction
- Brute-force passphrase cracking for stego
- Multi-tool analysis (steghide, stegseek, zsteg, binwalk, exiftool)
- Systematic approach to stego challenges (CTF preparation)

---

## Knowledge Check

Test your understanding of cryptographic concepts, tools, and their application in penetration testing.

**Question 1:** What is Kerckhoffs's principle, and why is it important for modern cryptography?

<details>
<summary>Show Answer</summary>
Kerckhoffs's principle states that a cryptosystem should be secure even if everything about the system, except the key, is public knowledge. This is important because security that depends on algorithm secrecy (security through obscurity) fails once the algorithm is reverse-engineered or leaked. Modern algorithms like AES and RSA are fully public — their security relies entirely on key secrecy. This principle drives the use of open, peer-reviewed, battle-tested algorithms rather than proprietary ones.
</details>

**Question 2:** Explain why ECB mode is insecure and what mode you would recommend instead.

<details>
<summary>Show Answer</summary>
ECB (Electronic Codebook) mode encrypts each block independently with the same key. This means identical plaintext blocks produce identical ciphertext blocks, revealing patterns in the data. The famous "ECB penguin" example shows that encrypting a bitmap image preserves the outline of the original image. Recommended alternatives are GCM (Galois/Counter Mode) for authenticated encryption — providing both confidentiality and integrity — or CTR (Counter Mode) when authentication is handled separately. GCM is preferred because it is an AEAD cipher that detects tampering.
</details>

**Question 3:** What is the difference between password hashing and general-purpose hashing? Name the recommended algorithm for password hashing.

<details>
<summary>Show Answer</summary>
General-purpose hash functions (SHA-256, BLAKE3) are designed to be fast — ideal for file integrity checks, digital signatures, and data structures. Password hashing requires the opposite: functions that are deliberately slow, memory-intensive, and resistant to GPU/ASIC acceleration. This slowness makes brute-force attacks against stolen password hashes impractical. The recommended algorithm is Argon2id, which won the Password Hashing Competition in 2015. It provides configurable time cost, memory cost, and parallelism. bcrypt is also a well-proven alternative. You should NEVER use MD5, SHA-1, or SHA-256 for password storage.
</details>

**Question 4:** Describe how a padding oracle attack works and what conditions must be present for it to succeed.

<details>
<summary>Show Answer</summary>
A padding oracle attack targets CBC-mode encryption when the server reveals whether decrypted ciphertext has valid padding. The attacker modifies bytes in a ciphertext block and sends the modified ciphertext to the server. Based on the server's response (valid padding vs invalid padding), the attacker can mathematically deduce the plaintext one byte at a time. Conditions required: (1) CBC mode encryption, (2) the server provides distinguishable responses for valid vs invalid padding (the "oracle"), and (3) the attacker can submit modified ciphertexts. Mitigations include using AEAD modes (GCM, ChaCha20-Poly1305) that detect ciphertext modification, or ensuring error messages do not reveal padding validity.
</details>

**Question 5:** What is Perfect Forward Secrecy (PFS), and which TLS cipher suite components provide it?

<details>
<summary>Show Answer</summary>
Perfect Forward Secrecy ensures that if a server's long-term private key is compromised in the future, past encrypted sessions remain secure. This is achieved by using ephemeral (temporary) keys for each session that are discarded after use. Cipher suites containing DHE (Diffie-Hellman Ephemeral) or ECDHE (Elliptic Curve Diffie-Hellman Ephemeral) in their names provide PFS. Cipher suites using static RSA key exchange do NOT provide PFS — if the RSA private key is later compromised, all past sessions encrypted with that key can be decrypted. TLS 1.3 mandates PFS for all connections by only allowing ECDHE and DHE key exchange.
</details>

**Question 6:** You discover a database with passwords hashed as unsalted MD5. Explain the attack workflow to crack them.

<details>
<summary>Show Answer</summary>
The workflow is: (1) Confirm the hash type using hashid or by recognizing the 32-character hex format. (2) First try a rainbow table lookup — since the hashes are unsalted, precomputed rainbow tables will work. Online services like CrackStation can instantly crack common passwords. (3) Run a dictionary attack with Hashcat: <code>hashcat -m 0 -a 0 hashes.txt rockyou.txt</code>. (4) Apply rule-based mutations: <code>hashcat -m 0 -a 0 hashes.txt rockyou.txt -r rules/best64.rule</code>. (5) For remaining uncracked hashes, try larger wordlists or mask attacks. MD5 is extremely fast to compute — a modern GPU can test over 50 billion MD5 hashes per second, making even moderately complex passwords crackable in minutes. Report this as a critical finding with a recommendation to migrate to Argon2id with unique salts.
</details>

**Question 7:** What is the difference between TLS 1.2 and TLS 1.3? Name at least four improvements in TLS 1.3.

<details>
<summary>Show Answer</summary>
TLS 1.3 (RFC 8446) made significant improvements over TLS 1.2: (1) Reduced handshake from 2 round-trips to 1 round-trip (with 0-RTT resumption option). (2) Removed all non-PFS key exchange methods — RSA key exchange is gone; only ECDHE and DHE are allowed, making Perfect Forward Secrecy mandatory. (3) Removed all weak algorithms — no more RC4, 3DES, CBC mode ciphers, MD5, or SHA-1. Only AES-GCM, AES-CCM, and ChaCha20-Poly1305 remain. (4) Simplified cipher suites from approximately 37 to just 5. (5) Encrypted most of the handshake (in TLS 1.2, certificates were sent in plaintext). (6) Removed TLS compression (vulnerable to CRIME attack). (7) Removed renegotiation (attack surface reduction).
</details>

**Question 8:** Explain how Certificate Transparency logs can be used for reconnaissance during a penetration test.

<details>
<summary>Show Answer</summary>
Certificate Transparency (CT) logs are public, append-only logs where CAs must record all issued certificates. Pentesters can query CT logs (via services like crt.sh) to discover all certificates ever issued for a target domain. This reveals: (1) Subdomains — including internal, staging, development, and test environments that may not appear in DNS or be found through brute-forcing. (2) Historical certificates — showing old domains, naming patterns, and infrastructure changes. (3) Certificate details — revealing CA choices, key sizes, and validity periods. The command <code>curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq '.[].name_value' | sort -u</code> returns all subdomains with issued certificates. This is a passive reconnaissance technique that does not touch the target's infrastructure.
</details>

**Question 9:** You find a file hidden inside a JPEG image during a CTF challenge. Describe your systematic approach to extracting it.

<details>
<summary>Show Answer</summary>
Systematic approach: (1) Start with metadata — run <code>exiftool image.jpg</code> to check for hidden data in EXIF/IPTC/XMP fields, comments, or GPS data. (2) Check for appended data — run <code>binwalk image.jpg</code> to detect files embedded after the JPEG end-of-file marker (common hiding technique). If found, extract with <code>binwalk -e image.jpg</code>. (3) Try steghide — run <code>steghide info image.jpg</code> to check if steghide was used. Try extraction without password: <code>steghide extract -sf image.jpg -p ""</code>. If password-protected, brute-force with <code>stegseek image.jpg rockyou.txt</code>. (4) Check LSB encoding — use <code>zsteg image.jpg</code> (if converted to PNG) or Stegsolve to examine individual bit planes. (5) Examine with hex editor — <code>xxd image.jpg | less</code> to look for readable strings or anomalies. (6) Check file size — compare to expected size for an image of those dimensions; significantly oversized files likely contain hidden data. (7) Run <code>strings image.jpg</code> to find readable text embedded in the file.
</details>

**Question 10:** A client asks you to assess whether their application's use of encryption is secure. What specific things would you check?

<details>
<summary>Show Answer</summary>
A comprehensive cryptographic assessment would check: (1) Algorithm selection — are they using modern, standard algorithms (AES-256-GCM, SHA-256+, Argon2id) or deprecated ones (DES, 3DES, MD5, SHA-1, RC4)? (2) Key management — are keys hardcoded in source code, stored in plaintext, or properly managed through a KMS? Are keys rotated regularly? (3) Password storage — are passwords hashed with a proper password hashing function (Argon2id, bcrypt) with unique salts, or are they using fast hashes or reversible encryption? (4) TLS configuration — protocol versions (TLS 1.2+ only), cipher suites (AEAD with PFS), certificate validity, HSTS headers. (5) Random number generation — is the application using cryptographically secure PRNGs (e.g., /dev/urandom, crypto.getRandomValues) or predictable sources (Math.random, timestamps)? (6) Implementation details — correct use of IVs/nonces (unique, random), proper padding, no ECB mode, no custom/proprietary crypto. (7) Key sizes — RSA 2048+ bits, AES 128+ bits, ECC 256+ bits. (8) Data in transit and at rest — is all sensitive data encrypted both when stored and when transmitted? (9) Error handling — do error messages leak cryptographic details (padding oracle, timing oracle)? (10) Compliance — does the implementation meet relevant standards (FIPS 140-2, PCI DSS, HIPAA)?
</details>

---

## Summary

Cryptography is a vast domain, but for ethical hackers, the practical focus is clear:

- **Recognize weak crypto** — Deprecated algorithms, insecure modes, poor key management, and custom implementations are common pentest findings.
- **Crack hashes efficiently** — Master John the Ripper and Hashcat with wordlists, rules, and masks.
- **Assess SSL/TLS thoroughly** — Use testssl.sh, sslscan, and Nmap scripts to identify protocol and cipher weaknesses.
- **Understand the theory** — Knowing why ECB is insecure, why salts matter, or how padding oracles work lets you identify vulnerabilities that scanners miss.
- **Communicate securely** — Use GPG to protect sensitive pentest deliverables.
- **Think like an attacker** — Every cryptographic implementation is an attack surface. Your job is to find where it breaks.

The best cryptographic implementations are invisible — they work correctly, use standard algorithms, manage keys properly, and fail securely. Your job as a pentester is to find the ones that do not.

---

[Next Module: Social Engineering -->](14-social-engineering.md)
