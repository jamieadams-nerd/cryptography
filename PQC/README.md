# Post-Quantum Cryptography (PQC)

## Overview
Post-Quantum Cryptography (PQC) is about creating encryption methods that **cannot be broken by quantum computers**.  
Today, most internet security—like banking, emails, and online shopping—relies on cryptography methods such as **RSA** and **ECC**. These methods are safe against classical computers but could be broken by large quantum computers.  

PQC develops new algorithms that remain secure even in a world with quantum computers.

---

## Why It’s Needed
1. **Classical encryption** (RSA, ECC) relies on problems like factoring large numbers or solving discrete logs.  
2. **Quantum computers** can solve these problems very fast using **Shor’s algorithm**.  
3. If large quantum computers exist, all current encryption could be broken.  
4. PQC protects sensitive data **before quantum computers arrive**.

---

## History
- **1970s–1990s:** RSA, Diffie-Hellman, ECC are developed and become standard for secure communication.  
- **1994:** Peter Shor invents **Shor’s algorithm**, showing quantum computers can efficiently break RSA and ECC.  
- **2000s:** Research into quantum-safe cryptography begins, exploring lattice-based, hash-based, code-based, multivariate, and isogeny-based cryptography.  
- **2016–2017:** NIST (National Institute of Standards and Technology) starts formal **PQC standardization process**.  
- **2022–2024:** NIST selects several PQC algorithms for standardization, with final standards expected around **2025–2026**.

---

## Technical Explanation (Simplified)
Post-Quantum Cryptography relies on math problems that **quantum computers cannot solve efficiently**. Main approaches include:

### 1. Lattice-based cryptography
- Uses points in a high-dimensional grid (lattice).  
- Security relies on problems like **Learning With Errors (LWE)**.  
- Example algorithms: **Kyber** (key exchange), **Dilithium** (digital signatures).

### 2. Code-based cryptography
- Uses error-correcting codes.  
- Hard to decode without a secret key.  
- Example algorithm: **Classic McEliece**.

### 3. Hash-based cryptography
- Uses hash functions for digital signatures.  
- Very secure, but signatures can be large.  
- Example algorithm: **SPHINCS+**.

### 4. Multivariate quadratic equations
- Based on solving complex polynomial systems over finite fields.  
- Example: **Rainbow signature scheme**.

### 5. Isogeny-based cryptography
- Uses math from elliptic curves in a way quantum computers struggle with.  
- Example: **SIKE** (withdrawn from NIST due to vulnerabilities).

---

## Timelines
- **Now (2025):** NIST is finalizing PQC standards. Early adoption in testing environments is possible.  
- **Next 5–10 years:** Gradual deployment in sensitive areas like government, banking, and cloud services.  
- **10+ years:** PQC likely becomes standard for most digital communications, replacing RSA/ECC.

---

## Key Takeaways
- PQC is **future-proof encryption** against quantum attacks.  
- It uses **new math problems** quantum computers cannot solve efficiently.  
- NIST is leading standardization, with some algorithms already selected.  
- Transition is gradual but urgent; sensitive data today could be compromised by future quantum computers.
