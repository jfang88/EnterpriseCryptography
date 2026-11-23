# Approved Algorithms and PQC Path Summary (DOC-05)

**Document Type:** Technical Standard (algorithm baseline)  
**Version:** 0.3  
**Status:** Draft for approval. Once approved it is normative for algorithm selection (Standards Doc #2, section 9.5)  
**Date:** 23 November 2026  
**Owner:** Security Architecture  
**Approval:** Security Steering Committee / CISO  
**Classification:** Internal Restricted  
**Review Cycle:** Annual or on material change  
**Audience:** Architects, engineers, IT operations, and compliance in a commercial enterprise (regulated financial services, not a national-security system)  
**Companions:** Policy v1.1 (DOC-01), Technical Standards v1.5 (DOC-02), Secrets Management Implementation Guide v1.1 (DOC-03), Cryptography Management Implementation Guide v1.1 (DOC-04)

---

## 1. At a glance

- **Section 2 is the approved list.** Use it for every new design, build, and configuration change.
- **Each use case is ranked.** Configure the 1st choice wherever the platform supports it. The 2nd choice is approved without an exception. A fallback needs a documented reason (interoperability or platform limitation) in the system's design record or cryptographic inventory entry; it does not need an exception. Anything under "Do not use" needs a formal exception under Policy section 7.
- **Symmetric encryption, hashing, and password storage are already quantum-safe.** Nothing changes there.
- **Post-quantum cryptography (PQC) asks for two things today**, both configuration changes on current software: hybrid key exchange on TLS 1.3 and on SSH, where the software already supports it. See section 3.
- **Everything else in PQC is planning only.** Nobody is expected to replace certificates, PKI, code signing, VPNs, or KMS/HSM key types today. The one item to plan now is firmware signing for new device designs (section 3). That migration is a separate programme; Part C is there to help teams plan for it.

| If you are | Read |
|---|---|
| Engineer or IT operations | Section 2 (what to configure), section 3 (the two PQC settings) |
| Architect | Sections 2 and 3, then Part C for planning |
| Compliance or audit | Section 2 "Do not use" columns, section 4 (traceability to NIST) |

---

# Part A. What to use

## 2. Approved algorithms, in priority order

### 2.1 Data protection

| Use case | 1st choice | 2nd choice | Fallback (documented reason) | Do not use |
|---|---|---|---|---|
| Data at rest (files, DB, objects) | **AES-256-GCM** | AES-256-CCM | AES-128-GCM | DES, 3DES, RC4, AES-ECB, unauthenticated CBC |
| Disk / volume encryption | **AES-256-XTS** | None | AES-128-XTS (platform default only) | AES-CBC without integrity |
| Key wrapping (DEK under KEK) | **AES-256-KW / KWP** | AES-256-GCM inside KMS / HSM | RSA-OAEP 3072+ (key import and transport only) | RSA PKCS#1 v1.5 |
| Key custody | **FIPS 140-3 validated HSM** (Tier 1 keys) | FIPS 140-3 validated cloud KMS (Tier 2 and Tier 3 keys) | Provider-managed keys (Tier 4 only) | Software-held keys for roots and CA keys |

### 2.2 Data in transit

TLS, SSH, and IPsec each negotiate several settings independently, so each setting has its own row and its own order. A connection uses one entry from each row. For example, hybrid TLS uses the group `X25519MLKEM768` together with the suite `TLS_AES_256_GCM_SHA384`.

| Use case | 1st choice | 2nd choice | Fallback (documented reason) | Do not use |
|---|---|---|---|---|
| TLS protocol version | **TLS 1.3** | None | TLS 1.2, for systems that cannot yet support TLS 1.3, using only the TLS 1.2 suites below | SSL, TLS 1.0, TLS 1.1 |
| TLS 1.3 key exchange group | **`X25519MLKEM768`** (hybrid) | `SecP256r1MLKEM768` (hybrid, all-NIST components); `SecP384r1MLKEM1024` for longest-lived data | Classical `X25519`, then `secp256r1`, then `secp384r1`, for peers or platforms without hybrid support | RSA key transport, static DH, FFDH < 3072 |
| TLS 1.3 cipher suite | **`TLS_AES_256_GCM_SHA384`** | `TLS_AES_128_GCM_SHA256` | `TLS_CHACHA20_POLY1305_SHA256` (outside FIPS boundaries only) | CBC-mode and non-AEAD suites |
| TLS 1.2 cipher suite (TLS 1.2 fallback only) | **`TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384`**, `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384` | `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256`, `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` | `TLS_ECDHE_*_WITH_CHACHA20_POLY1305_SHA256` (outside FIPS boundaries only) | RSA key exchange (`TLS_RSA_*`), `DHE` suites, CBC-mode suites, 3DES, RC4, NULL and EXPORT suites |
| SSH key exchange | **`mlkem768x25519-sha256`** (hybrid) | `sntrup761x25519-sha512` (hybrid) | `curve25519-sha256`, then `ecdh-sha2-nistp384`, for hosts without hybrid support | `diffie-hellman-group1/14-sha1`, DH groups < 3072-bit |
| SSH cipher | **`aes256-gcm@openssh.com`** | `aes128-gcm@openssh.com` | `chacha20-poly1305@openssh.com` (outside FIPS boundaries); `aes256-ctr` with ETM MAC | CBC ciphers, `arcfour`, `hmac-md5`, `hmac-sha1` |
| SSH identity (user / host keys) | **Ed25519** with short-lived SSH certificates | ECDSA P-384 | RSA-4096 for legacy devices, max 90 days | DSA, RSA < 3072, static long-lived keys |
| IPsec / VPN key exchange | **IKEv2 with ECDH P-384** | IKEv2 with ECDH P-256 | None | IKEv1, DH groups < 3072-bit |
| IPsec / VPN encryption | **AES-256-GCM** | AES-128-GCM | AES-256-CBC with HMAC-SHA-384 | 3DES, AES-CBC with HMAC-SHA-1 or MD5 |

Servers enforce their own preference order. Where a TLS or SSH endpoint supports hybrid key exchange, a classical fallback is for the peer's benefit only; where the platform can log the negotiated group, log those connections so the remaining non-hybrid peers can be found.

**Inside a FIPS boundary:** `X25519` and `curve25519-sha256` are not NIST-approved on their own. Where FIPS validation is claimed, the classical fallback order is `secp256r1` then `secp384r1` for TLS, and `ecdh-sha2-nistp256` then `ecdh-sha2-nistp384` for SSH.

### 2.3 Signatures and certificates

| Use case | 1st choice | 2nd choice | Fallback (documented reason) | Do not use |
|---|---|---|---|---|
| Server / client certificates | **ECDSA P-384** | ECDSA P-256 | RSA 3072+ with PSS where ECC is unsupported | SHA-1 certificates, new RSA keys < 3072, DSA |
| Application signatures, tokens (JWT etc.) | **ECDSA P-384** (ES384) | ECDSA P-256 (ES256) or Ed25519 | RSA-PSS 3072+ (PS256 / PS384) | RSA PKCS#1 v1.5 for new designs, `alg=none`, HS256 with shared weak secrets |
| Code signing | **ECDSA P-384** | ECDSA P-256 | RSA-PSS 3072+ | SHA-1, MD5, RSA PKCS#1 v1.5 |
| Firmware / boot signing | **ECDSA P-384** | RSA-PSS 3072+ | None | SHA-1, MD5, RSA < 3072 |

**Certificate operations**

- **Public TLS certificates must be renewed automatically** (ACME or a certificate-management platform). The maximum lifetime of a public TLS certificate is 200 days from 15 March 2026, 100 days from 15 March 2027, and 47 days from 15 March 2029. Manual renewal will not keep up.
- **Client certificates for mutual TLS must come from a private CA.** Public CAs are removing the client-authentication extended key usage (clientAuth EKU) from public TLS certificates; the Chrome Root Program deadline is 15 March 2027, and some CAs stop earlier (DigiCert on 1 March 2027). Systems that present a public TLS certificate as a client certificate will fail at the first renewal after their CA's cut-off.

### 2.4 Hashing, passwords, keys, and randomness

| Use case | 1st choice | 2nd choice | Fallback (documented reason) | Do not use |
|---|---|---|---|---|
| Hashing | **SHA-384** | SHA-256 (minimum), SHA-512 | SHA-3 family where a protocol requires it | MD5, SHA-1 |
| Message authentication | **HMAC-SHA-384** | HMAC-SHA-256 | KMAC256 | HMAC-MD5, HMAC-SHA-1 |
| Key derivation | **HKDF-SHA-384** | HKDF-SHA-256 | SP 800-108 counter-mode KDF | Ad-hoc hash-based derivation |
| Password storage, outside FIPS boundaries | **Argon2id**, minimum 19 MiB memory, 2 iterations, parallelism 1 | PBKDF2-HMAC-SHA-256, minimum 600,000 iterations | scrypt, minimum N=2^17, r=8, p=1 | Unsalted hashes, MD5, SHA-family hashes without a work factor, reversible encryption |
| Password storage, inside FIPS boundaries | **PBKDF2-HMAC-SHA-256**, minimum 600,000 iterations | PBKDF2-HMAC-SHA-512, minimum 210,000 iterations | None | Argon2id and scrypt (not FIPS-approved); unsalted hashes, hashes without a work factor, reversible encryption |
| Key generation | **HSM or KMS** key generation (keys never leave the module) | FIPS 140-3 validated software module | None | Keys generated in application code for Tier 1 and Tier 2 |
| Random numbers in application code | **OS generator through the standard library**: `secrets` (Python), `crypto/rand` (Go), `SecureRandom` (Java), `crypto.randomBytes` / `crypto.getRandomValues` (Node.js), `RandomNumberGenerator` (.NET) | `getrandom()` or `/dev/urandom` directly | None | `random`, `Math.random`, `rand()`, `java.util.Random`, or any time-seeded generator |

---

## 3. Post-quantum: what applies today

| Area | Status | What is expected |
|---|---|---|
| TLS 1.3 key exchange | **Available now** | Enable `X25519MLKEM768` where the software already supports it (current browsers, OpenSSL 3.5+, Go 1.24+, major cloud and CDN endpoints). Keep classical groups as fallback. No certificate change. |
| SSH key exchange | **Available now** | Enable `mlkem768x25519-sha256` on OpenSSH 9.9 or later. Keep `curve25519-sha256` as fallback. No key change. |
| Symmetric encryption, hashing, MAC, KDF, password storage | **No change needed** | Section 2 choices are already quantum-safe. |
| New device designs whose firmware must stay trusted beyond 2035 | **Plan now** | Evaluate LMS / XMSS or SLH-DSA firmware signing at design time. Boot ROM and secure-boot code cannot be changed after shipment, so this is the one planning item with a real deadline. Existing devices: planning only. |
| Certificates and PKI, code signing, firmware signing (other than above), tokens, IPsec / VPN, key import and BYOK wrapping, email encryption | **Planning only** | No change expected now. Keep using section 2. Product and ecosystem support is not yet general. |

Enabling hybrid key exchange is a configuration change, not a migration. If a platform does not support it yet, staying on the classical fallback is compliant; no exception is needed and no platform has to be replaced for this reason alone.

**What "planning only" asks of teams**

- Record where RSA, ECDSA, EdDSA, and ECDH are used in the cryptographic inventory.
- In new designs, do not hard-code algorithms, key sizes, or signature and certificate sizes. Carry an algorithm identifier so they can be changed later.
- Ask vendors for their ML-KEM and ML-DSA roadmap at procurement and renewal.
- Use Part C to see where each use case is heading.

The PQC migration itself is run as a separate programme. Its outline timeline is in Standards Doc #2 (Post-Quantum Cryptography Transition).

---

## 4. Traceability to NIST

| Area | NIST basis |
|---|---|
| AES and modes | FIPS 197; SP 800-38C (CCM), 800-38D (GCM), 800-38E (XTS), 800-38F (key wrap) |
| TLS configuration | SP 800-52 Rev. 2 |
| Hybrid key exchange | FIPS 203 (ML-KEM); SP 800-227; SP 800-56C Rev. 2; RFC 10024 |
| Signatures | FIPS 186-5 (ECDSA, EdDSA, RSA-PSS) |
| RSA key transport | SP 800-56B |
| Hashing and MAC | FIPS 180-4, FIPS 202, FIPS 198-1, SP 800-185 |
| Key derivation | SP 800-56C Rev. 2, SP 800-108 Rev. 1 |
| Password storage | SP 800-132, SP 800-63B (minimum parameters in section 2.4 follow the OWASP Password Storage Cheat Sheet) |
| Random numbers and key generation | SP 800-90A / B / C |
| IPsec | SP 800-77 Rev. 1 |
| Cryptographic modules | FIPS 140-3 |
| Algorithm retirement | SP 800-131A, NIST IR 8547 |

Items in section 2 that are acceptable for a commercial organisation but are **not** NIST-approved on their own:

- **ChaCha20-Poly1305** and **Argon2id** are not NIST-approved. Fine outside a FIPS boundary; keep AES-GCM and PBKDF2 where FIPS validation is claimed.
- **X25519** is not in SP 800-56A. Inside `X25519MLKEM768` the handshake is still acceptable to NIST because the ML-KEM half is approved (SP 800-56C Rev. 2 / SP 800-227). On its own, as a classical fallback, it is not; see the FIPS boundary note in section 2.2.
- **AES-128** remains NIST-approved, including against quantum attack. AES-256 is our baseline for margin and regulator optics, not because AES-128 is broken.
- **RSA-2048** is approved until NIST deprecation after 2030, like every other quantum-vulnerable algorithm, but at 112-bit strength it has no margin. Do not issue new RSA-2048 keys; existing keys can run to their normal renewal.
- **scrypt** and the OpenSSH **sntrup761** hybrid are not NIST-approved; both are acceptable outside FIPS boundaries.
- **P-384 over P-256** and **SHA-384 over SHA-256** are enterprise preferences for margin. P-256 and SHA-256 remain fully NIST-approved.

---

# Part B. Rationale

## 5. Who approves what

| Body | Role | What it means for us |
|---|---|---|
| **NIST** | Approves algorithms (FIPS / SP 800 series) and sets the retirement timeline (IR 8547, SP 800-131A) | The algorithm baseline. Everything in section 2 traces to a NIST publication unless flagged. |
| **CISA** | Does **not** approve algorithms. Points to NIST standards and publishes migration guidance: the *Quantum-Readiness* factsheet (with NSA and NIST), inventory guidance, and the PQC product-category list (January 2026) | The migration playbook: inventory, prioritise by HNDL exposure, buy PQC-capable products. |
| **NSA CNSA 2.0** | Mandatory suite for US national-security systems (ML-KEM-1024, ML-DSA-87, AES-256, SHA-384/512) | **Not required for us.** Useful as a ceiling for the few assets that need the highest margin. |

Commercial default is **NIST security category 3** (ML-KEM-768, ML-DSA-65), deployed as **hybrid** with a classical algorithm. CNSA 2.0 goes to category 5 and pure PQC; we do not need to follow that.

## 6. Why hybrid key exchange now

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

**Action:** where the software already supports it, enable `X25519MLKEM768` (preferred) with classical fallback on TLS 1.3 terminators: edge/CDN, load balancers, API gateways, service mesh, and outbound clients. Many current platforms already enable it by default. This is a configuration change on current software, not a migration project.

---

# Part C. PQC planning reference

This part is for planning. It does not create requirements beyond section 3. Where the availability column says anything other than "Available now" or "Done", no change is expected today.

## 7. PQC path by use case

| Use case | Today (classical) | Transition step | Target | Standard | Availability |
|---|---|---|---|---|---|
| TLS key exchange | ECDHE X25519 / P-256 / P-384 | **X25519MLKEM768** | ML-KEM-768 hybrid; ML-KEM-1024 for longest-lived data | FIPS 203, RFC 10024 | **Available now** (see section 3) |
| SSH key exchange | curve25519-sha256 | **mlkem768x25519-sha256** (or sntrup761x25519-sha512) | Same | FIPS 203 | **Available now** (OpenSSH 9.9+, default in 10.0) |
| IPsec / IKEv2 | ECDH P-384 | ECDH + ML-KEM via RFC 9370; RFC 8784 PPK as stop-gap | ML-KEM-768/1024 hybrid | FIPS 203 | Vendor dependent |
| Key transport / BYOK import wrap | RSA-OAEP 3072/4096 | Wrap over hybrid TLS channel; shorten wrap-key lifetime | ML-KEM-768/1024 encapsulation | FIPS 203, SP 800-227 | KMS/HSM dependent |
| Email / document encryption (S/MIME, CMS) | RSA-OAEP, ECDH | Hybrid KEM in CMS | ML-KEM-768 | FIPS 203 | Early, track vendors |
| Internal PKI certificates | ECDSA P-256/P-384 | Stand up a parallel ML-DSA hierarchy; composite or dual certs in pilot | **ML-DSA-65** (leaf), **ML-DSA-87** (root / issuing CA) | FIPS 204 | Programme pilot; HSM firmware dependent |
| Public web certificates | ECDSA P-256 / RSA | None available from public CAs yet | ML-DSA, or Merkle Tree Certificates | FIPS 204 | **Wait**, driven by CA/B Forum and browsers |
| Code signing | ECDSA P-384 | Dual-sign: classical + ML-DSA | ML-DSA-65/87 | FIPS 204 | Tooling emerging |
| Firmware / boot signing, long-lived roots | ECDSA / RSA | Dual-sign | **LMS / XMSS** (stateful, needs HSM state control) or **SLH-DSA** (stateless, larger signatures) | SP 800-208, FIPS 205 | Available now; evaluate for new device designs (section 3), planning only for existing devices |
| JWT / API tokens, app signatures | ES256/ES384, EdDSA | Keep classical, short token lifetimes | ML-DSA-65 | FIPS 204 | Low urgency, no HNDL exposure |
| Data at rest | AES-256-GCM | No change | **AES-256-GCM** | FIPS 197 | **Done**; key wrapping is covered by the key-transport row |
| Hash / MAC / KDF | SHA-256/384, HMAC, HKDF | No change | SHA-384 preferred | FIPS 180-4 | **Done** |
| Password hashing | Argon2id / PBKDF2 | No change | Same | SP 800-132 | **Done** |

## 8. PQC parameter choices

| Algorithm | Default | Step up when |
|---|---|---|
| ML-KEM (FIPS 203) | **ML-KEM-768** | ML-KEM-1024 for data needing confidentiality beyond roughly 15 years |
| ML-DSA (FIPS 204) | **ML-DSA-65** | ML-DSA-87 for root CAs and long-lived signing roots |
| SLH-DSA (FIPS 205) | Roots and firmware only | Where a conservative hash-based fallback is wanted |
| LMS / XMSS (SP 800-208) | Firmware and boot signing | Only with HSM-enforced state management |
| FN-DSA (FIPS 206, Falcon) | **Do not adopt yet** | Still in development at NIST |
| HQC | **Do not adopt yet** | Selected March 2025 as backup KEM; standard still in development |

## 9. Timeline anchors

| Date | Event | Source |
|---|---|---|
| Aug 2024 | FIPS 203, 204, 205 final | NIST |
| Jan 2026 | CISA / NSA PQC product-category list published | CISA |
| 15 Mar 2026 | Public TLS certificate maximum lifetime 200 days | CA/B Forum SC-081 |
| 1 Mar 2027 | DigiCert stops including clientAuth EKU in public TLS certificates (other CAs vary) | DigiCert |
| 15 Mar 2027 | Public TLS certificate maximum lifetime 100 days; Chrome Root Program deadline for removing clientAuth EKU from public TLS certificates | CA/B Forum SC-081; Chrome Root Program |
| Aug 2026 | RFC 10024 hybrid ECDHE-MLKEM for TLS 1.3 | IETF |
| Oct 2026 | IR 8547 final. Reported to deprecate **all** quantum-vulnerable public-key algorithms after 2030, at every key size (the 2024 draft only deprecated 112-bit strength then). **Unverified, open issue 1.** | NIST (via commentary) |
| **2029** | PQC programme target for priority systems (externally exposed, identity-bearing, long-retention) | Standards v1.5 |
| 15 Mar 2029 | Public TLS certificate maximum lifetime 47 days; domain validation reuse 10 days | CA/B Forum SC-081 |
| **After 2030** | RSA, ECDSA, EdDSA, ECDH, DH **deprecated** | NIST IR 8547 |
| 2030 / 2031 | US federal high-value systems: PQC key establishment by end 2030, signatures by end 2031. Federal only; a useful benchmark. **Unverified, open issue 1.** | Reported, EO / OMB |
| **After 2035** | Quantum-vulnerable public-key algorithms **disallowed** | NIST IR 8547 |

---

# Part D. Document control

## 10. Change history

| Version | Date | Change |
|---|---|---|
| 0.1 | 16 Nov 2026 | Initial summary: approved algorithms by use case, TLS 1.3 hybrid key exchange, PQC path. Consistency review of DOC-01 to DOC-04 (versions, cross-references, examples, PQC wording) |
| 0.2 | 16 Nov 2026 | Choices ranked per use case; restructured so the approved list comes first and rationale and PQC planning follow |
| 0.3 | 23 Nov 2026 | Version control added; PQC migration framed as a separate programme across the document set; TLS 1.2 made an approved fallback with a suite list; FIPS fallback note; password, random number, and key generation rows; certificate lifetime and clientAuth changes; review fixes |

## 11. Open issues

| # | Issue | Owner | Status |
|---|---|---|---|
| 1 | The NIST IR 8547 final (October 2026) details and the US federal 2030/2031 dates in section 9 come from commentary, not the primary texts. Verify against NIST IR 8547 and the executive order / OMB memo, then remove the "Unverified" flags or delete the rows. | Security Architecture | **Open, before approval** |
| 2 | DOC-02 has no control-ID catalogue. If control IDs are wanted for exception tracking, define them in DOC-02 first. | Security Architecture | Open |
| 3 | DOC-01 to DOC-04 each carry two overlapping reference sections (a numbered list and a trailing table). Merge at the next revision. | Document owners | Open |
| 4 | File names and versions of DOC-01 to DOC-04 are unchanged. The DOC-02 algorithm and PQC changes are material and would normally warrant v1.6 at the next formal approval. | Security Architecture | Open |

## 12. Sources

- NIST FIPS 203 (ML-KEM), FIPS 204 (ML-DSA), FIPS 205 (SLH-DSA): <https://csrc.nist.gov/projects/post-quantum-cryptography>
- NIST IR 8547, Transition to Post-Quantum Cryptography Standards: <https://nvlpubs.nist.gov/nistpubs/ir/2026/NIST.IR.8547.pdf>
- Commentary on the final IR 8547 (see open issue 1): <https://postquantum.com/security-pqc/nist-ir-8547-final/> and <https://labs.cloudsecurityalliance.org/research/csa-research-note-nist-ir8547-pqc-transition-20261030-csa-st/>
- NIST SP 800-52 Rev. 2 (TLS), SP 800-131A (transitions), SP 800-227 (KEMs), SP 800-208 (LMS / XMSS), SP 800-56C Rev. 2 (hybrid key derivation): <https://csrc.nist.gov/publications/sp800>
- RFC 10024 / `draft-ietf-tls-ecdhe-mlkem`: <https://datatracker.ietf.org/doc/draft-ietf-tls-ecdhe-mlkem/>
- CISA PQC product categories announcement: <https://content.govdelivery.com/accounts/USDHSCISA/bulletins/405c246>
- OpenSSH post-quantum status: <https://www.openssh.org/pq.html>
- CA/Browser Forum Ballot SC-081v3 (certificate lifetimes): <https://cabforum.org/2025/04/11/ballot-sc081v3-introduce-schedule-of-reducing-validity-and-data-reuse-periods/>
- Google Chrome Root Program, clientAuth deprecation: <https://developers.google.com/public-key-infrastructure/updates/may2025-clientauth>
- DigiCert, sunsetting clientAuth EKU: <https://knowledge.digicert.com/alerts/sunsetting-client-authentication-eku-from-digicert-public-tls-certificates>
- OWASP Password Storage Cheat Sheet: <https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html>
