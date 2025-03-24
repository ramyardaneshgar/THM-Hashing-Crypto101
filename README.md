# THM-Hashing-Crypto101
Writeup for TryHackMe Hashing Lab - Hash identification, password cracking, and integrity validation using Hashcat, John the Ripper, sha1sum, and Splunk.

By  Ramyar Daneshgar 

---

## **Task 1 – Key Terms**

### Summary:
This section tested basic terminology recognition. I confirmed that **Base64** is a data representation format — not cryptographic — and should never be confused with encryption or hashing.

- **Plaintext:** Data in original form.
- **Encoding:** Reversible transformation for interoperability.
- **Hash:** Fixed-length output derived from variable input.
- **Brute force:** Exhaustive password search.
- **Cryptanalysis:** Mathematical attack on cryptographic algorithms.

 **Answer:** `encoding`

---

## **Task 2 – What Is a Hash Function?**

### Analysis:
A hash function maps arbitrary-length input to a fixed-length digest. I validated the following cryptographic properties:

- **Determinism:** Same input always yields same output.
- **Collision resistance:** Computationally infeasible to find two inputs with the same output.
- **Avalanche effect:** Minor changes to input produce drastically different outputs.
- **Pre-image resistance:** Infeasible to reconstruct original input from hash.
- **Second pre-image resistance:** Infeasible to find a different input with the same hash.

I referenced the MD5 specification and determined its output size is 128 bits, or 16 bytes. I confirmed that with only 8 bits of output space, a hash function would yield 2^8 = 256 possible outputs, illustrating the **pigeonhole principle** and why collisions are inevitable in large data sets.

 Output size: `16`  
 Collisions avoidable: `Nay`  
 8-bit hash space: `256`

---

## **Task 3 – Uses for Hashing**

### Real-World Applications:
- **Password Storage:** Hashing with salt and slow algorithms (bcrypt, Argon2).
- **Integrity Checking:** Hashes used to verify data at rest or in transit (e.g., ISO downloads).
- **Fingerprinting:** Malware signatures, forensics, duplicate detection.

I examined historical failures:
- **RockYou:** Plaintext storage, harvested for wordlists like `rockyou.txt`.
- **Adobe:** Encrypted passwords, but using insecure ECB mode, compromising confidentiality.
- **LinkedIn:** Stored passwords using SHA1, fast and GPU-crackable.

### Rainbow Tables:
Rainbow tables precompute unsalted hashes to invert them rapidly. I manually mapped:

 `d0199f51d2728db6011945145a1b607a` → `basketball`  
 `5b31f93c09ad1d065c0491b764d04933` → `tryhackme`  
  Password encryption?: `Nay`

---

## **Task 4 – Recognizing Password Hashes**

### Detection Approach:
Hash recognition depends on:
- **Prefix (e.g., $6$ = SHA512crypt)**
- **Length (e.g., 32 chars = MD5/NTLM)**
- **Context (e.g., Windows = NTLM, Linux = SHA512crypt)**

I validated hash types using [Hashcat’s example hash library](https://hashcat.net/wiki/doku.php?id=example_hashes) and file path conventions (`/etc/shadow`, `SAM` on Windows).

 SHA512crypt rounds: `5000`  
 Citrix hash: `1765058016a22f1b4e076dccd1c3df4e8e5c0839ccded98ea`  
 NTLM hash length: `32`

---

## **Task 5 – Password Cracking**

### Methodology:
I used **John the Ripper** and **Hashcat** with `rockyou.txt` to perform brute force and dictionary-based attacks. The hashing algorithm dictated the cracking method.

---

### Bcrypt (`$2a$`)
```bash
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
 Output: `85208520`

---

### SHA256 (64-char hex)
```bash
hashcat -m 1400 -a 0 hash.txt rockyou.txt
```
 Output: `halloween`

---

### SHA512crypt (`$6$`)
```bash
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
 Output: `spaceman`

---

### MD5 (32-char hex)
```bash
hashcat -m 0 -a 0 hash.txt rockyou.txt
```
 Output: `funforyou`

---

## **Task 6 – Hashing for Integrity Checking**

### File Integrity:
I validated that cryptographic hashes (e.g., SHA1, SHA256) are used to confirm file integrity. I computed hashes using:

```bash
sha1sum kali-2019.4-amd64.iso
```

 Expected SHA1: `186c5227e24ceb60deb711f1bdc34ad9f4718ff9`

---

### HMAC (Hash-Based Message Authentication Code):
An **HMAC** combines a hash function with a secret key to provide authenticity and integrity. Used in VPNs, TLS, and API signing. TryHackMe's VPN uses `HMAC-SHA512`.

To identify the **Hashcat mode** for cracking an HMAC where `key = password`, I referred to Hashcat’s wiki.

 HMAC-SHA512 (key = $pass): `1750`

---

## **Conclusion and Operational Relevance**

This room reinforces the critical importance of properly implementing cryptographic primitives. Hashing is not encryption, and while both serve different goals, their misuse results in systemic vulnerabilities. From a security engineering standpoint, the following practices must be observed:

- Never store passwords in plaintext or encrypted form. Use **strong, salted hash algorithms** such as bcrypt or Argon2.
- Always verify file downloads using published cryptographic hashes.
- Understand the limitations of legacy algorithms like MD5 and SHA1, which have been proven to allow **collision attacks**.
- Incorporate HMACs for **message authentication and tamper detection**.
- Use GPU-accelerated cracking tools like **Hashcat** for adversarial testing in controlled environments, ensuring password policies are defensible under brute force threat models.

