# Module 20 – Cryptography

> Status: ✅ Reference complete

## 1. Cryptography concepts
From Greek *kryptos* (hidden) + *graphia* (writing) = "the art of secret writing." **Encryption** converts readable plaintext into unreadable ciphertext; only authorized users with the correct key can decrypt.

### Objectives
- **Confidentiality** — accessible only to the authorized.
- **Integrity** — prevents improper/unauthorized changes.
- **Authentication** — the communication/document is genuine.
- **Nonrepudiation** — sender can't deny sending, recipient can't deny receiving.

### Types
- **Symmetric (secret-key)** — one key for encrypt + decrypt; fast, less processing; but needs secure key exchange, hard to manage many keys, no origin assurance, vulnerable to dictionary/brute force.
- **Asymmetric (public-key)** — public key encrypts, private key decrypts; no key-sharing risk, provides non-repudiable digital signatures; but slow, high processing, compromised/lost private key breaks everything, vulnerable to MITM/brute force. Sequence: find recipient's public key → encrypt → recipient decrypts with private key.

### GAK (Government Access to Keys)
Policy requiring individuals/orgs to provide cryptographic keys to government on legal request (court warrant); uses **key escrow** (keys stored with a trusted third party). Raises privacy/breach concerns — a single compromised escrow key could expose vast data.

### Ciphers
Mathematical algorithm converting plaintext ↔ ciphertext. Open-source (public algorithm, secret key) vs closed-source (both secret).
- **Classical** (operate on A–Z, easily broken): **substitution** (replace units — Beale, autokey, Gronsfeld, Hill; "HELLO"→"PSTER"), **transposition** (rearrange letters — rail fence, route, Myszkowski; "CRYPTOGRAPHY"→"AOYCRGPTYRHP").
- **Modern** (withstand attacks; one-way math functions). By key: **symmetric-key** / **asymmetric-key**. By input: **block cipher** (fixed-size blocks, padding; DES/AES/IDEA — most modern ciphers), **stream cipher** (bit-by-bit with keystream; RC4/SEAL).

## 2. Symmetric encryption algorithms
| Algorithm | Type | Key size (bits) | Block (bits) | Application |
|-----------|------|-----------------|--------------|-------------|
| DES | Block | 56 | 64 | Legacy |
| 3DES | Block | 112, 168 | 64 | Financial/payment |
| AES | Block | 128, 192, 256 | 128 | Secure comms, storage, government |
| RC4 | Stream | 40–2048 (variable) | – | HTTPS, WEP/WPA, streaming |
| RC5 | Block | 0–2040 (variable) | 32, 64, 128 | Crypto libraries |
| RC6 | Block | 128, 192, 256 | 128 | AES finalist |
| Blowfish | Block | 32–448 (variable) | 64 | DES replacement, storage |
| Twofish | Block | 128, 192, 256 | 128 | File/disk encryption |
| IDEA | Block | 128 | 64 | PGP email |
| Threefish | Block | 256, 512, 1024 | 256, 512, 1024 | Skein hash / disk |
| Serpent | Block | 128, 192, 256 | 128 | AES finalist |
| Camellia | Block | 128, 192, 256 | 128 | TLS, Japanese standard |
| TEA | Block | 128 | 64 | Lightweight/embedded |
| CAST-128 | Block | 40–128 | 64 | GPG/PGP |
| CAST-256 | Block | 128–256 | 128 | Crypto libraries |
| ChaCha20 | Stream | 256 | – | Modern protocols |
| Salsa20 | Stream | 256 | – | Crypto protocols |

- **DES** — 64-bit blocks (56-bit key + 8 error-detection); ~72 quadrillion keys, now brute-forceable.
- **3DES** — DES ×3 with key bundle K1/K2/K3: encrypt(K1) → decrypt(K2) → encrypt(K3). Three unique keys = strongest; one key repeated = equals DES.
- **AES** — NIST standard, iterated block cipher, 128-bit block, keys 128/192/256. **Pseudocode:**
  ```
  Cipher(byte in[4*Nb], byte out[4*Nb], word w[Nb*(Nr+1)])
    state = in
    AddRoundKey(state, w)
    for round = 1 to Nr-1:
      SubBytes(state); ShiftRows(state); MixColumns(state); AddRoundKey(state, w+round*Nb)
    SubBytes(state); ShiftRows(state); AddRoundKey(state, w+Nr*Nb)
    out = state
  ```
- **RC4** — variable-key stream cipher (random permutation, 8–16 ops/byte); SSL/web traffic.
- **RC5** — Rivest; variable block (32/64/128), key up to 2040, rounds 0–255; key expansion + integer addition + XOR + data-dependent rotation.
- **RC6** — from RC5; adds integer multiplication (diffusion) + four 4-bit registers (AES 128-bit block).
- **Blowfish** — 16-round Feistel, 64-bit blocks, keys 32–448; key expansion (P-array + S-boxes, repeat 521×).
- **Twofish** — Bruce Schneier, 128-bit Feistel, key up to 256; DES-replacement finalist.
- **Threefish** — Skein (SHA-3 contest); tweakable, block=key (256/512/1024), ARX only (no S-boxes → anti cache-timing), 72/72/80 rounds.
- **Serpent** — 128-bit blocks, 32 rounds, parallel S-boxes; slower than Rijndael but strong.
- **TEA** — 1994 Feistel, 64 rounds, 64-bit blocks, 128-bit key, delta constant (golden ratio), modular add/sub + XOR.
- **CAST-128/256** — Feistel (12/16 rounds), masking + rotation keys, four S-boxes; GPG/PGP.
- **GOST (Magma)** — Russian, 64-bit block, 256-bit key, 32-round Feistel; modern version Kuznyechik (128-bit).
- **Camellia** — 128-bit blocks, Feistel + S-boxes + key whitening; TLS; as secure as AES.

## 3. Asymmetric encryption algorithms
| Algorithm | Key size | Application |
|-----------|----------|-------------|
| RSA | Variable | Encryption, signatures, key exchange |
| DSA | Variable | Digital signatures |
| Diffie-Hellman | Variable | Key exchange |
| ECC | 160–521 | Encryption, signatures, key exchange |
| ElGamal | Variable | Encryption, key exchange |

- **DSA** — NIST, part of DSS (FIPS 186); 320-bit signature, keys 512–1024. Signature gen uses private key; verify uses public key. Algorithm: select prime q (2^159<q<2^160), prime p (q divides p-1), generator α, random d, y=α^d mod p → public (p,q,α,y), private d. Sign: r=(α^k mod p) mod q, s=k⁻¹{h(m)+dr} mod q → (r,s). Verify: w=s⁻¹, u1=w·h(m), u2=rw, v=(α^u1·y^u2 mod p) mod q, accept if v=r.
- **RSA** — Rivest/Shamir/Adleman; two large primes a,b → c=ab (modulus), e relatively prime to (a-1)(b-1), f such that (ef-1) divisible by (a-1)(b-1). Public (c,e), private (c,f). Security = factoring c is hard. **Key gen:** generate primes p,q → n=pq, φ=(p-1)(q-1), choose e with gcd(e,φ)=1, compute d≡e⁻¹ mod φ → public (n,e), private d (destroy p,q). **RSA digital envelope:** encrypt message with random DES key → encrypt DES key with recipient's RSA public key → send both (combines DES speed + RSA key management).
  - **Worked example:** P=61, Q=53, PQ=3233, E=17, D=2753. `encrypt(123)=123^17 mod 3233=855`; `decrypt(855)=855^2753 mod 3233=123`.
- **Diffie-Hellman** — Diffie/Hellman 1976; parameters p (prime) + g (generator). Alice picks a, Bob picks b → public g^a mod p / g^b mod p → exchange → shared key g^ab=g^ba=k. No authentication (MITM-vulnerable), but provides forward secrecy in TLS ephemeral modes.
- **ECC** — elliptic curves; strong security with small keys. Equivalence: ECC 160–223 ≈ RSA 1024; ECC 256–383 ≈ RSA 3072; ECC 512+ ≈ RSA 15360. Ideal for mobile/IoT.
- **YAK** — public-key Authenticated Key Exchange (needs PKI); HMQV variant with ZKP; exchange X=g^x + KP{x} and Y=g^y + KP{y} → shared session key k=H(g^((x+a)(y+b))). Provides private-key security, full forward secrecy, session-key security (lacks joint key control).

## 4. Hash (message digest) functions
One-way functions converting any data → fixed-length digest; tiny input change → totally different output; irreversible; for integrity, signatures, key derivation.

| Algorithm | Output (bits) | Block (bits) | Rounds | Security (bits) | Application |
|-----------|---------------|--------------|--------|-----------------|-------------|
| MD2 | 128 | 128 | 18 | 128 | Legacy/checksum |
| MD4 | 128 | 512 | 48 | 64 | Obsolete |
| MD5 | 128 | 512 | 64 | 64 | File verification, checksum |
| MD6 | 224–512 | 512 | Variable | 128–256 | Data integrity (Merkle tree) |
| SHA-0 | 160 | 512 | 80 | 0 | Obsolete |
| SHA-1 | 160 | 512 | 80 | 80 | Legacy, TLS (deprecated 2010) |
| SHA-2 | 224–512 | 512, 1024 | 64, 80 | 112–256 | Signatures, SSL |
| SHA-3 | 224–512 | 1088, 576 | Variable | 112–256 | Next-gen (sponge) |
| RIPEMD-160 | 160 | 512 | 160 | 80 | Data integrity |
| WHIRLPOOL | 512 | 512 | 10 | 256 | Secure hashing |
| Tiger | 192 | 512 | 24 | 192 | High-speed/checksum |
| BLAKE2 | 256, 512 | 512, 1024 | 10–14 | 128–256 | High-speed hashing |
| BLAKE3 | 256 | 512 | Variable | 128 | High-performance |

- **MD5/MD6** — 128-bit fingerprint; MD5 vulnerable to collisions; MD6 uses Merkle tree. Minimally different inputs → radically different hashes (`md5sum`).
- **SHA** — NIST, FIPS PUB 180. **SHA-0** (withdrawn), **SHA-1** (160-bit, deprecated 2010), **SHA-2** (SHA-256 32-bit words / SHA-512 64-bit; truncated SHA-224/384), **SHA-3** (sponge construction).
- **RIPEMD-160** — 160-bit, 80-step compression ×2.
- **HMAC** — secret key + hash (SHA/MD5), two-stage; resistant to length-extension attacks.
- **GOST hash** — GOST R 34.11-94, 256-bit output.
- **Multilayer hashing** (nested/recursive) — apply hash repeatedly (CyberChef); harder to reverse. Tools: QuickHash-GUI, MD5 Calculator, HashMyFiles.

## 5. Hardware encryption & advanced techniques
- **TPM** — motherboard crypto-processor; stores keys, full-disk encryption, platform integrity, password storage.
- **HSM** — external crypto device; manage/generate/store keys (useful for symmetric keys >256-bit); TCP/IP. Examples: Thales Luna, nShield, Utimaco.
- **USB encryption** — onboard encryption for USB devices (Kingston Ironkey, diskAshur Pro).
- **Hard drive encryption** — needs TPM/HSM (no on-device keyboard); 256-bit AES.
- **Quantum cryptography (QKD)** — uses photons with spin/polarization (horizontal/forward-slash=0, vertical/backslash=1); any interception distorts polarization → detectable (anti-MITM).
- **Homomorphic encryption** — compute on encrypted data without decrypting; only key holder decrypts the result (secure cloud outsourcing).
- **Post-quantum (quantum-resistant)** — lattice/hash/code-based, secure against quantum attacks.
- **Lightweight cryptography** — for low-power IoT/RFID; minimal resources, quantum-safe focus.

### Cipher modes of operation
- **ECB** — each block encrypted independently; identical plaintext → identical ciphertext (patterns visible, insecure).
- **CBC** — each block XORed with IV / previous ciphertext before encryption; error propagation.
- **CFB** — IV encrypted, XORed with plaintext; ciphertext feeds next block.
- **CTR (Counter)** — turns block cipher into stream cipher (XOR with encrypted counter); no error propagation but needs counter sync.

### Authenticated encryption (AE)
Combines encryption + MAC (anti chosen-ciphertext). **Encrypt-then-MAC (EtM)** — encrypt, then MAC the ciphertext (most secure). **Encrypt-and-MAC (E&M)** — MAC plaintext + encrypt separately. **MAC-then-Encrypt (MtE)** — MAC plaintext, combine, encrypt. **AEAD** — encrypted payload + unencrypted authenticated header. Tools: BCTextEncoder, CryptoForge, AxCrypt.

## 6. Applications of cryptography

### PKI (Public Key Infrastructure)
Binds public keys to identities via a CA. **Components:** Certificate Management System, Digital Certificates, Validation Authority (VA), Certification Authority (CA), End User, Registration Authority (RA). **Process:** subject applies to RA → RA verifies identity, requests CA → CA issues public-key certificate, updates VA → user signs message with private key + certificate → client verifies with VA → VA validates certificate.
- **CAs:** Comodo, IdenTrust, DigiCert CertCentral, GoDaddy.
- **Signed certificate** (from trusted CA: holder name, serial, expiry, public key, CA's digital signature; verified via VA) vs **self-signed** (created with Adobe Acrobat/Java keytool/Keychain; verified by the user directly).

### Digital signature
Asymmetric: signer hashes the message → encrypts hash with private key = signature. Verifier decrypts signature with sender's public key, compares to a fresh hash of the received message; match = authentic + unaltered.

### SSL / TLS
- **SSL** — RSA public-key encryption; channel security: *private* (encrypted after handshake), *authenticated* (server always, client optional), *reliable* (integrity check). **Handshake:** client hello → server hello (protocol version, session ID, cipher suite, compression) → server certificate + optional key exchange + certificate request → server hello done → client certificate + key-exchange message → change cipher spec + finished (both sides) → application data. Session resumption uses the session ID.
- **TLS** — symmetric for bulk encryption, asymmetric for auth/key exchange, MAC for integrity; RSA 1024/2048; application-protocol independent. **TLS Record Protocol** (fragment/compress/MAC/encrypt), **TLS Handshake Protocol** (authenticate, negotiate keys): Client hello (random + cipher suites) → Server hello (random) → certificate + optional client cert request → server hello done → client sends pre-master secret encrypted with server's public key → both derive master secret + session keys → change cipher spec + client finished → server change cipher spec + server finished → encrypted application data.
- Toolkits: OpenSSL, wolfSSL, Libsodium, Crypto++, PyCryptodome.

### PGP / GPG / Web of Trust
- **PGP** — hybrid (RSA + IDEA + MD5); encrypt/sign emails/files. **Encryption:** compress → create random one-time session key → encrypt plaintext with session key → encrypt session key with recipient's public key → send both. **Decryption:** recover session key with private key → decrypt ciphertext.
- **GPG** — free OpenPGP implementation; hybrid, supports S/MIME, SSH, ECC (ECDSA/ECDH/EdDSA), Libgcrypt. Sign with sender's private key → encrypt with receiver's public key.
- **Web of Trust (WoT)** — decentralized; users verify + sign each other's public keys (no central CA).

### Email, disk encryption & blockchain
- **Email:** S/MIME (Outlook Trust Center), Office 365 Message Encryption (OME), FlowCrypt (Gmail + OpenPGP). Tools: RMail, Mailvelope, Virtru, Proofpoint.
- **Disk encryption:** VeraCrypt (real-time), Rohos (AES-256), BitLocker (TPM-based). Linux: Cryptsetup (dm-crypt/LUKS), Tomb, CryFS. macOS: FileVault (XTS-AES-128, 256-bit key).
- **Blockchain** — distributed ledger of cryptographically linked blocks (data + hash + previous-block hash); genesis block = 0s. Validation = **proof of work**; adding blocks = **crypto mining**. Uses SHA-256 + asymmetric keys. **Variants:** **public** (permissionless, decentralized — Bitcoin/Ethereum, B2C), **private** (central authority — Hyperledger/Ripple, B2B), **federated/consortium** (group-controlled — EWF/R3), **hybrid** (selective public data — IBM Food Trust).

## 7. Cryptanalysis
Study of ciphers/cryptosystems to find vulnerabilities and extract plaintext without the key.

### Methods
- **Linear** (Matsui) — approximate the cipher with linear XOR equations between plaintext/ciphertext/key bits; known-plaintext.
- **Differential** (Biham/Shamir) — examine how input differences affect output differences; chosen-plaintext.
- **Integral** (Knudsen) — fix some bits, vary others through all values; for SPN ciphers.
- **Quantum** — Shor's algorithm (factor RSA/ECDH in polynomial time), Grover's algorithm (faster brute force for AES/SHA). Resources: circuit width/depth, gates, T-gates, T-depth, MAXDEPTH.

### Cryptography attacks
- **Ciphertext-only** — only ciphertexts available.
- **Chosen-plaintext / adaptive chosen-plaintext** — attacker gets ciphertext for chosen plaintexts (adaptive = interactive queries).
- **Known-plaintext** — some plaintext-ciphertext pairs (linear cryptanalysis).
- **Chosen-ciphertext** — get plaintext for chosen ciphertexts (variants: lunchtime/midnight, adaptive).
- **Related-key** — ciphertexts under two closely-related keys.
- **Dictionary** — prebuilt plaintext↔ciphertext table.
- **Rubber-hose** — coercion/torture to extract keys.
- **Chosen-key** — breaks the larger system dependent on the ciphertext.
- **Timing** — measure execution time of modular exponentiation.
- **MITM** — intercept and negotiate crypto parameters (public-key key exchange).

### Code-breaking methodologies
- **Brute force** — try every key; depends on key length + hardware (DES fell to exhaustive search).
- **Frequency analysis** — count letter/pattern frequency (source code especially vulnerable).
- **Trickery and deceit** — social engineering to extract keys.
- **One-time pad** — unbreakable (non-repeating random key = message length); impractical for large messages.

**Brute-force cost table (time to break):**
| Power/Cost | 40-bit | 56-bit | 64-bit | 128-bit |
|-----------|--------|--------|--------|---------|
| $2K (1 PC) | 1.4 min | 73 days | 50 years | 10^20 years |
| $100K (company) | 2 sec | 35 hours | 1 year | 10^19 years |
| $1M (state) | 0.2 sec | 3.5 hours | 37 days | 10^18 years |
> Each additional key bit doubles the brute-force time. (DES 56-bit in 1 sec → AES 128-bit ≈ 149 trillion years.)

### Specific attacks
- **Birthday attack** — brute-force against hashes via the birthday paradox (23 people → >50% share a birthday); collision expected after ~1.17·√(2^n) ≈ 2^64 for MD5's 2^128 space.
- **Brute-forcing VeraCrypt** — extract hash, crack with hashcat/John:
  ```bash
  dd.exe if=<container> of=<hashfile.tc> bs=512 count=1
  hashcat.exe -a 3 -w 1 -m 13721 <hashfile.tc> ?d?d?d?d     # 4-digit numeric
  hashcat.exe -w 1 -m 13721 hash.tc wordlist.txt            # dictionary
  ```
- **Meet-in-the-middle** — against multi-key encryption (double DES); space-time trade-off (birthday-based); encrypt from one end, decrypt from the other to "meet in the middle" (~2^56–2^57 ops vs 2^112).
- **Side-channel** — physical attack monitoring: **power consumption** (SPA/DPA), **electromagnetic field**, **light emission** (CRT/LED), **timing/delay**, **sound (acoustic)**. Depends on implementation, not algorithm. **Mitigations:** fixed-time algorithms, masking/blinding, add noise, pre-charge registers, power-line filtering, hardware isolation (HSM/enclaves), monitoring, randomization, error detection.
- **Hash collision** — find two inputs with the same hash (forge signatures; SHA-1 target).
- **DUHK (Don't Use Hard-coded Keys)** — recover keys when ANSI X9.31 PRNG uses a hard-coded seed (MITM → derive VPN/web session keys).
- **DROWN** — decrypt TLS by abusing SSLv2 servers sharing the same private key (cross-protocol Bleichenbacher padding oracle); MITM to steal keys.
- **Rainbow table** — precomputed hash lookup table (time-memory trade-off).
- **Related-key (WEP example)** — reused IVs + fixed key → related RC4 keys → key recovery.
- **Padding oracle** — probe CBC padding error responses to decrypt byte-by-byte; defend with authenticated encryption (AES-GCM), constant-time generic errors, MAC-then-decrypt.

### Blockchain attacks
- **51% (majority)** — control >50% hash/stake power → double-spend, reverse transactions. Steps: gain 51% → mine a private chain → extend it longer than the public chain → release → network accepts the longer chain.
- **Finney** — pre-mine a block with a self-transaction, pay the victim with the same coins, then broadcast the pre-mined block to reverse the payment (targets zero-confirmation merchants).
- **Eclipse** — isolate a node by surrounding it with malicious peers (fill peer tables → force restart → reconnect only to attacker nodes).
- **Race** — broadcast two conflicting transactions quickly (victim + self); if the self-transaction confirms first, the victim's is invalidated (zero-confirmation targets).
- **DeFi sandwich** — front-run + back-run a large pending DEX transaction (buy before to inflate price, sell after) to profit at the victim's expense.

### Quantum computing risks & attacks
**Risks:** breaking classical crypto (Shor/Grover vs RSA/DSA/ECC), transition challenges, harvest-now-decrypt-later, decrypting SSL/TLS/VPN, quantum-driven exploits, quantum eavesdropping, undermining blockchain, breaking authentication, quantum malware. **Attacks:** quantum cryptanalysis, quantum side-channel, classical-to-quantum transition, harvest-now/decrypt-later, quantum Trojan horse, quantum supply chain, quantum sabotage, fault injection on quantum hardware, quantum DoS, quantum data eavesdropping, quantum bit-flipping, error-correction exploitation, quantum replay.

**Tools:** CrypTool (CT1/CT2/JCT/CTO), RsaCtfTool, Msieve, Cryptol, CryptoSMT. Online MD5 decryption: dcode.fr, md5decrypt.net, hashes.com, cmd5.org.

## 8. Countermeasures
### Defend against cryptographic attacks
Strong keys (256-bit symmetric, 2048-bit+ asymmetric); keep keys secure (HSM, never in code); encrypt stored keys; rotate keys regularly; strong hashing (SHA-256/SHA-3 + unique salt); TLS for communication; avoid weak/custom crypto (use AES); message authentication (Encrypt-then-MAC / GCM); key derivation functions (PBKDF2/bcrypt/Argon2); prepare for quantum (lattice/hash-based).

### Defend against blockchain attacks
Decentralized identifiers (DIDs); zero-knowledge proofs; HSM key storage; multi-signature wallets; real-time transaction monitoring; code audits; formal verification of smart contracts; advanced DDoS protection; secure interoperability protocols; wait for multiple confirmations.

### Defend against quantum computing attacks
Quantum-resistant algorithms (lattice/hash/code-based); hybrid classical+quantum-safe; increase key sizes; rotate keys; Quantum Key Distribution (QKD); quantum-safe VPNs/certificates; quantum-safe encryption at rest; quantum-safe MFA; modular cryptographic systems; HSMs supporting quantum-safe algorithms.

### Key stretching
Strengthen weak keys/passwords (anti brute-force). **PBKDF2** (PKCS #5 v2.01 — applies hash/HMAC + salt to the passphrase). **Bcrypt** (Blowfish-based hashing + salt).

> Core: pick the right primitive — symmetric (AES) for bulk speed, asymmetric (RSA/ECC) for key exchange + signatures, hashes (SHA-256/SHA-3 + salt) for integrity, authenticated encryption (AES-GCM / Encrypt-then-MAC) for both. Attackers break crypto through implementation flaws (side-channel, padding oracle, weak RNG like DUHK) and protocol weaknesses (DROWN, related-key) far more than by breaking the math. Defend with strong keys + HSM key management + rotation + TLS + trusted libraries, and start the migration to post-quantum algorithms before harvest-now-decrypt-later becomes real.
