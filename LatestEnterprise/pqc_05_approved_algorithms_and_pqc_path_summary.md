# Approved Algorithms and PQC Path Summary (DOC-05)

**Date:** 16 November 2026  
**Audience:** Commercial enterprise (regulated financial services, not a national-security system)  
**Status:** Normative for algorithm selection (Standards Doc #2, section 9.5)  
**Companions:** Policy v1.1 (DOC-01), Technical Standards v1.5 (DOC-02), Secrets Management Implementation Guide v1.1 (DOC-03), Cryptography Management Implementation Guide v1.1 (DOC-04)

---

## 1. Who approves what

| Body | Role | What it means for us |
|---|---|---|
| **NIST** | Approves algorithms (FIPS / SP 800 series) and sets the retirement timeline (IR 8547, SP 800-131A) | The algorithm baseline. Everything in section 2 traces to a NIST publication unless flagged. |
| **CISA** | Does **not** approve algorithms. Points to NIST standards and publishes migration guidance: the *Quantum-Readiness* factsheet (with NSA and NIST), inventory guidance, and the PQC product-category list (January 2026) | The migration playbook: inventory, prioritise by HNDL exposure, buy PQC-capable products. |
| **NSA CNSA 2.0** | Mandatory suite for US national-security systems (ML-KEM-1024, ML-DSA-87, AES-256, SHA-384/512) | **Not required for us.** Useful as a ceiling for the few assets that need the highest margin. |

Commercial default is **NIST security category 3** (ML-KEM-768, ML-DSA-65), deployed as **hybrid** with a classical algorithm. CNSA 2.0 goes to category 5 and pure PQC; we do not need to follow that.

---

## 2. Use now, by use case

| Use case | Use now | NIST basis | Avoid |
|---|---|---|---|
| Data at rest (files, DB, objects) | **AES-256-GCM** | FIPS 197, SP 800-38D | DES, 3DES, RC4, AES-ECB |
| Disk / volume encryption | **AES-256-XTS** | SP 800-38E | AES-CBC without integrity |
| Key wrapping (DEK under KEK) | **AES-256-KW / KWP**, or AES-256-GCM inside KMS/HSM | SP 800-38F | RSA PKCS#1 v1.5 |
| Web / API / mTLS transport (TLS) | **TLS 1.3** with hybrid key exchange **`X25519MLKEM768`** and suite **`TLS_AES_256_GCM_SHA384`**. Full priority order in section 3.1 | SP 800-52 Rev. 2, FIPS 203, SP 800-227, SP 800-56C Rev. 2, RFC 10024 | SSL, TLS 1.0/1.1, CBC and RSA-key-transport suites, static DH, FFDH < 3072 |
| Server / client certificates | **ECDSA P-256 or P-384**; RSA ≥ 3072 with PSS where ECC is not supported | FIPS 186-5 | SHA-1 certs, RSA < 2048, DSA |
| Application signatures, tokens (JWT etc.) | **ECDSA P-256/P-384** or **Ed25519** | FIPS 186-5 | RSA PKCS#1 v1.5 for new designs, `alg=none`, HS256 with shared weak secrets |
| Code and firmware signing | ECDSA P-384 today; **start ML-DSA-65/87 or LMS/XMSS now** for anything with a long field life | FIPS 186-5, FIPS 204, SP 800-208 | SHA-1, MD5 |
| Hashing | **SHA-256** minimum, **SHA-384** preferred; SHA-3 allowed | FIPS 180-4, FIPS 202 | MD5, SHA-1 |
| Message authentication | **HMAC-SHA-256 / HMAC-SHA-384** | FIPS 198-1 | HMAC-MD5, HMAC-SHA-1 |
| Key derivation | **HKDF** / SP 800-108 KDFs | SP 800-56C Rev. 2, SP 800-108 Rev. 1 | Ad-hoc hash-based derivation |
| Password storage | **PBKDF2-HMAC-SHA-256** (high iteration count) inside FIPS boundaries; **Argon2id** elsewhere | SP 800-132, SP 800-63B | Unsalted hashes, MD5, reversible encryption |
| Random numbers | HSM / OS **DRBG** | SP 800-90A/B/C | Application-level or time-seeded PRNGs |
| SSH | **Ed25519** user/host keys with short-lived SSH certificates; KEX **`mlkem768x25519-sha256`** | FIPS 186-5, FIPS 203 | DSA, RSA < 3072, static long-lived keys |
| IPsec / VPN | IKEv2, AES-256-GCM, ECDH P-384 **plus** ML-KEM additional key exchange (RFC 9370), or a post-quantum pre-shared key (RFC 8784) as an interim | SP 800-77 Rev. 1 | IKEv1, DH groups < 3072-bit |
| Cryptographic modules | **FIPS 140-3** validated HSM / KMS for Tier 1 and Tier 2 keys | FIPS 140-3 | Software keys for roots and CA keys |

Notes on items that are sound for a commercial organisation but **not** NIST-approved on their own:

- **ChaCha20-Poly1305** and **Argon2id** are not NIST-approved. Fine outside a FIPS boundary; keep AES-GCM and PBKDF2 where FIPS validation is claimed.
- **X25519** is not in SP 800-56A. Inside `X25519MLKEM768` the handshake is still acceptable to NIST because the ML-KEM half is approved (SP 800-56C Rev. 2 / SP 800-227).
- **AES-128** remains NIST-approved, including against quantum attack. AES-256 is our baseline for margin and regulator optics, not because AES-128 is broken.
- **RSA-2048** is 112-bit strength and is the first to go. Do not issue new RSA-2048 keys.

---

## 3. TLS 1.3 and hybrid key exchange against HNDL

**The threat.** In harvest-now-decrypt-later (HNDL), an adversary records encrypted traffic today and decrypts it once a quantum computer can break the ECDH or RSA key exchange. Only the **key exchange** matters for HNDL; a forged signature cannot be applied retroactively. So confidentiality of data in transit is the part that is already at risk today, and it is the first thing to fix.

**Why TLS 1.3 is the unlock.**

| Point | Detail |
|---|---|
| Negotiated groups | TLS 1.3 negotiates key exchange through `supported_groups` / `key_share`, separately from the cipher suite. A hybrid group is just a new group ID; no new cipher suites, no certificate change. |
| Standardised | **RFC 10024** (August 2026) defines `X25519MLKEM768`, `SecP256r1MLKEM768`, `SecP384r1MLKEM1024`. |
| Hybrid safety | The session key derives from both the ECDH secret and the ML-KEM secret. An attacker must break both, so a flaw in ML-KEM leaves us no worse than today. |
| TLS 1.2 is a dead end | The IETF has frozen TLS 1.2; it will not get PQC key exchange. Any endpoint stuck on TLS 1.2 stays HNDL-exposed. |
| Already deployed | Default or available in Chrome, Firefox, Edge, Safari, OpenSSL 3.5+, Go 1.24+, BoringSSL, AWS-LC, Cloudflare, and the major cloud provider endpoints. |
| Cost | About 1.1 KB extra in ClientHello and ServerHello each, negligible CPU. The real risk is old middleboxes and load balancers that mishandle a large ClientHello; test those. |
| Signatures unchanged | Certificates stay ECDSA/RSA for now. That is acceptable because authentication is not HNDL-exposed. |

### 3.1 TLS 1.3 profile and priority order

"Web transport" and "TLS key exchange" are the same protocol. A TLS 1.3 connection negotiates two settings independently, so there are two priority lists, not one. Hybrid is a **key exchange group**; `TLS_AES_*` are **cipher suites** that encrypt the data once the key is agreed. A hybrid connection uses one entry from each list, for example `X25519MLKEM768` together with `TLS_AES_256_GCM_SHA384`.

| Priority | Key exchange group | Use |
|---|---|---|
| 1 | `X25519MLKEM768` | Preferred. Hybrid, protects against HNDL |
| 2 | `SecP256r1MLKEM768` | Hybrid, where both components must be NIST-approved (FIPS boundary) |
| 3 | `SecP384r1MLKEM1024` | Hybrid, optional, for longest-lived confidentiality requirements |
| 4 | `X25519` | Classical fallback for peers without hybrid support |
| 5 | `secp256r1`, then `secp384r1` | Classical fallback |

| Priority | Cipher suite | Use |
|---|---|---|
| 1 | `TLS_AES_256_GCM_SHA384` | Preferred |
| 2 | `TLS_AES_128_GCM_SHA256` | Approved; mandatory-to-implement for interoperability |
| 3 | `TLS_CHACHA20_POLY1305_SHA256` | Optional, outside FIPS boundaries only |

Servers enforce their own order. Log every connection that lands on a classical fallback group so the remaining non-hybrid peers can be found and fixed. TLS 1.2 is by time-boxed exception only and can never be hybrid.

**Action:** enable `X25519MLKEM768` (preferred) with classical fallback on every TLS 1.3 terminator now: edge/CDN, load balancers, API gateways, service mesh, and outbound clients. This is a configuration change on current software, not a migration project.

---

## 4. PQC path by use case

| Use case | Today (classical) | Transition (hybrid, do now or next) | Target | Standard | Readiness |
|---|---|---|---|---|---|
| TLS key exchange | ECDHE X25519 / P-256 / P-384 | **X25519MLKEM768** | ML-KEM-768 hybrid; ML-KEM-1024 for longest-lived data | FIPS 203, RFC 10024 | **Ready, deploy now** |
| SSH key exchange | curve25519-sha256 | **mlkem768x25519-sha256** (or sntrup761x25519) | Same | FIPS 203 | **Ready** (OpenSSH 9.9+, default in 10.0) |
| IPsec / IKEv2 | ECDH P-384 | ECDH + ML-KEM via RFC 9370; RFC 8784 PPK as stop-gap | ML-KEM-768/1024 hybrid | FIPS 203 | Vendor dependent, pilot |
| Key transport / BYOK import wrap | RSA-OAEP 3072/4096 | Wrap over hybrid TLS channel; shorten wrap-key lifetime | ML-KEM-768/1024 encapsulation | FIPS 203, SP 800-227 | KMS/HSM dependent |
| Email / document encryption (S/MIME, CMS) | RSA-OAEP, ECDH | Hybrid KEM in CMS | ML-KEM-768 | FIPS 203 | Early, track vendors |
| Internal PKI certificates | ECDSA P-256/P-384 | Stand up a parallel ML-DSA hierarchy; composite or dual certs in pilot | **ML-DSA-65** (leaf), **ML-DSA-87** (root / issuing CA) | FIPS 204 | Pilot now, HSM firmware dependent |
| Public web certificates | ECDSA P-256 / RSA | None available from public CAs yet | ML-DSA, or Merkle Tree Certificates | FIPS 204 | **Wait**, driven by CA/B Forum and browsers |
| Code signing | ECDSA P-384 | Dual-sign: classical + ML-DSA | ML-DSA-65/87 | FIPS 204 | Tooling emerging, pilot |
| Firmware / boot signing, long-lived roots | ECDSA / RSA | Dual-sign | **LMS / XMSS** (stateful, needs HSM state control) or **SLH-DSA** (stateless, larger signatures) | SP 800-208, FIPS 205 | Ready, **prioritise**: devices outlive the transition |
| JWT / API tokens, app signatures | ES256/ES384, EdDSA | Keep classical, short token lifetimes | ML-DSA-44/65 | FIPS 204 | Low urgency, no HNDL exposure |
| Data at rest | AES-256-GCM | No change | **AES-256-GCM** | FIPS 197 | **Done**; fix the key-wrap path above |
| Hash / MAC / KDF | SHA-256/384, HMAC, HKDF | No change | SHA-384 preferred | FIPS 180-4 | **Done** |
| Password hashing | Argon2id / PBKDF2 | No change | Same | SP 800-132 | **Done** |

**Parameter choice for a commercial organisation**

| Algorithm | Default | Step up when |
|---|---|---|
| ML-KEM (FIPS 203) | **ML-KEM-768** | ML-KEM-1024 for data needing confidentiality beyond roughly 15 years |
| ML-DSA (FIPS 204) | **ML-DSA-65** | ML-DSA-87 for root CAs and long-lived signing roots |
| SLH-DSA (FIPS 205) | Roots and firmware only | Where a conservative hash-based fallback is wanted |
| LMS / XMSS (SP 800-208) | Firmware and boot signing | Only with HSM-enforced state management |
| FN-DSA (FIPS 206, Falcon) | **Do not adopt yet** | Still in development at NIST |
| HQC | **Do not adopt yet** | Selected March 2025 as backup KEM; standard still in development |

---

## 5. Timeline anchors

| Date | Event | Source |
|---|---|---|
| Aug 2024 | FIPS 203, 204, 205 final | NIST |
| Jan 2026 | CISA / NSA PQC product-category list published | CISA |
| Aug 2026 | RFC 10024 hybrid ECDHE-MLKEM for TLS 1.3 | IETF |
| Oct 2026 | IR 8547 final. Reported to deprecate **all** quantum-vulnerable public-key algorithms after 2030, at every key size (the 2024 draft only deprecated 112-bit strength then). Confirm against the NIST text before relying on per-strength detail. | NIST |
| **2029** | Enterprise target for priority migration (externally exposed, identity-bearing, long-retention systems) | Standards v1.5 |
| Mar 2029 | Public TLS certificate maximum lifetime falls to 47 days; automation is a prerequisite for crypto-agility | CA/B Forum |
| **After 2030** | RSA, ECDSA, EdDSA, ECDH, DH **deprecated** | NIST IR 8547 |
| 2030 / 2031 | US federal high-value systems: PQC key establishment by end 2030, signatures by end 2031. Federal only; a useful benchmark. | Reported, EO / OMB |
| **After 2035** | Quantum-vulnerable public-key algorithms **disallowed** | NIST IR 8547 |

---

## 6. Consistency review of the document set (16 November 2026)

The findings below were raised against DOC-01 to DOC-04 and have been corrected in those documents.

| # | Finding | Resolution |
|---|---|---|
| 1 | Roadmap deferred hybrid ML-KEM to a 2028 production pilot | DOC-02 and DOC-04 timelines now enable hybrid TLS and SSH key exchange in 2026; signatures and PKI stay on 2027-2029 |
| 2 | HNDL not named; key exchange and signatures given equal urgency | DOC-01 principle 6 and DOC-02 PQC section now put key establishment first |
| 3 | PQC tables named families only | Parameter sets added (ML-KEM-768, ML-DSA-65, step-up rules) in DOC-02 and DOC-04 |
| 4 | RSA / ECDSA / EdDSA / ECDH labelled "legacy only" while section 5.4 mandated them | Reworded as approved now, deprecated after 2030 |
| 5 | "AES-256" with no mode; no TLS suite or group list | DOC-02 sections 5.3, 5.4, 7.3 updated; TLS profile added as section 9.2.1 |
| 6 | TLS 1.3 minimum applied only to secret-retrieval traffic | TLS 1.3 is now the enterprise default; TLS 1.2 by exception |
| 7 | LMS / XMSS and HQC absent; FN-DSA without status | Added to the DOC-02 and DOC-04 algorithm tables |
| 8 | No NIST PQC or CISA references | Added to DOC-02 sections 2, 20.1, 20.4 and DOC-04 references |
| 9 | BYOK import used RSA-OAEP wrapping with no PQC note | Migration requirement added in DOC-02; note added in DOC-04 section 4.2 |
| 10 | DOC-04 example defects (Alibaba CLI for Huawei Cloud, key deletion shown as rotation, SoftHSM with hard-coded PIN, P-256 for Tier 1, invalid AWS JSON) | Examples rewritten; Huawei KooCLI commands are illustrative and need validation against the current CLI reference |
| 11 | Guides cited control IDs (PR-KMS-01, SEC-LC-05 and similar) that do not exist in DOC-02 | Replaced with DOC-02 section references |
| 12 | Version drift: DOC-02 cited as v1.4 and DOC-04 as v1.0 in several places; DOC-01 change history stopped at 1.0 | Aligned to v1.5 and v1.1; DOC-01 history row added; DOC-05 added to all internal document tables |
| 13 | Broken table-of-contents anchors in DOC-02, DOC-03, DOC-04; DOC-02 appendix lettered both A and B; doubled citation links; template artefacts (`{{ }}`, doubled backslashes) in code blocks | Corrected |
| 14 | DOC-01 purpose treated HKMA guidance as an obligation, contradicting its own precedence rule | Reworded: SFC primary, HKMA informative benchmark |
| 15 | DOC-03 used citation code R21 for the OCCICS Code of Practice; DOC-02 uses R22 | DOC-03 renumbered to R22 |

**Still open**

- DOC-02 has no control-ID catalogue. If control IDs are wanted for exception tracking, define them in DOC-02 first.
- DOC-01 to DOC-04 each carry two reference sections (a numbered list and a trailing table) that overlap. They now agree but should be merged at the next revision.
- File names and version numbers were left unchanged. The algorithm changes in DOC-02 are material and would normally warrant a v1.6 at the next formal approval.

---

## 7. Sources

- NIST FIPS 203 (ML-KEM), FIPS 204 (ML-DSA), FIPS 205 (SLH-DSA): <https://csrc.nist.gov/projects/post-quantum-cryptography>
- NIST IR 8547, Transition to Post-Quantum Cryptography Standards: <https://nvlpubs.nist.gov/nistpubs/ir/2026/NIST.IR.8547.pdf>
- Commentary on the final IR 8547: <https://postquantum.com/security-pqc/nist-ir-8547-final/> and <https://labs.cloudsecurityalliance.org/research/csa-research-note-nist-ir8547-pqc-transition-20261030-csa-st/>
- NIST SP 800-52 Rev. 2 (TLS), SP 800-131A (transitions), SP 800-227 (KEMs), SP 800-208 (LMS / XMSS), SP 800-56C Rev. 2 (hybrid key derivation): <https://csrc.nist.gov/publications/sp800>
- RFC 10024 / `draft-ietf-tls-ecdhe-mlkem`: <https://datatracker.ietf.org/doc/draft-ietf-tls-ecdhe-mlkem/>
- CISA PQC product categories announcement: <https://content.govdelivery.com/accounts/USDHSCISA/bulletins/405c246>
- OpenSSH post-quantum status: <https://www.openssh.org/pq.html>
