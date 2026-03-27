# Enterprise Cryptographic Standard

**Classification:** Internal — Restricted  
**Version:** 1.2  
**Date:** March 2026  
**Owner:** Information Security Architecture  
**Supports:** [Enterprise Cryptographic Policy](1-Enterprise-Cryptographic-Policy-v1_2.md)

---

## Table of Contents

- [1. Purpose](#1-purpose)
- [2. Standard hierarchy](#2-standard-hierarchy)
- [3. Policy-to-control mapping](#3-policy-to-control-mapping)
- [4. Control domains](#4-control-domains)
- [5. Algorithm standard](#5-algorithm-standard)
- [6. Key lifecycle standard](#6-key-lifecycle-standard)
- [7. Identity and secret-access standard](#7-identity-and-secret-access-standard)
- [8. Monitoring and incident standard](#8-monitoring-and-incident-standard)
- [9. Quantum-safe and crypto-agility standard](#9-quantum-safe-and-crypto-agility-standard)
- [10. Cloud and shared-responsibility alignment](#10-cloud-and-shared-responsibility-alignment)
- [11. Document relationships](#11-document-relationships)
- [12. References](#12-references)

---

## 1. Purpose

This document defines the enterprise **standard** requirements that implement the high-level requirements in the [Enterprise Cryptographic Policy](1-Enterprise-Cryptographic-Policy-v1_2.md).

It provides:
- formal control requirements;
- traceability to policy, NIST CSF 2.0, and CSA CCM;
- lower-level mappings to NIST and OWASP references;
- measurable requirements for audit, architecture, and implementation review.

## 2. Standard hierarchy

| Layer | Source | Use |
| :-- | :-- | :-- |
| Policy | [Enterprise Cryptographic Policy](1-Enterprise-Cryptographic-Policy-v1_2.md) | Defines high-level mandatory outcomes |
| Governance overlay | [NIST CSF 2.0](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf) | Internal cyber governance taxonomy |
| Cloud overlay | [CSA CCM](https://cloudsecurityalliance.org/research/cloud-controls-matrix) | Cloud control and shared-responsibility mapping |
| Detailed control sources | [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final), [NIST SP 800-57 Part 1 Rev. 5](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final), [NIST SP 800-175B Rev. 1](https://csrc.nist.gov/pubs/sp/800/175/b/r1/final), [FIPS 203](https://csrc.nist.gov/pubs/fips/203/final), [FIPS 204](https://csrc.nist.gov/pubs/fips/204/final), [FIPS 205](https://csrc.nist.gov/pubs/fips/205/final), [OWASP Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html) | Detailed control and engineering guidance |
| Engineering detail | [Enterprise Cryptographic Implementation Guide](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md) | Platform patterns and implementation approaches |

## 3. Policy-to-control mapping

| Policy ID | Standard domain | Example lower-level references |
| :-- | :-- | :-- |
| POLICY-CI-001 | Governance, architecture, tiering | NIST SP 800-53 PL, RA, CA, CM |
| POLICY-CI-003 | Governance, roles, approvals | NIST SP 800-53 PM, PL, RA |
| POLICY-CI-004 | Key lifecycle, IAM, logging, IR | NIST SP 800-57; NIST SP 800-53 SC, AC, AU, IR, CP |
| POLICY-CI-005 | Risk, audit, logging, incident response | NIST SP 800-53 RA, AU, IR, CP, CA |
| POLICY-SFC-001 | Transport protection | NIST SP 800-53 SC; OWASP transport and storage guidance |
| POLICY-SFC-002 | MFA and access control | NIST SP 800-53 IA, AC |
| POLICY-SFC-003 | Password protection | OWASP password and storage guidance |
| POLICY-CRY-001 / 002 | Algorithm standard | NIST SP 800-175B; enterprise algorithm catalogue |
| POLICY-CRY-003 / 004 | Key inventory and custody | NIST SP 800-57 |
| POLICY-CRY-005 / 006 / 007 / 010 | Secret retrieval, design review, inventory, crypto-agility | NIST SP 800-53 AC, IA, SC; FIPS 203/204/205 |
| POLICY-PQC-001 to 006 | PQC and crypto-agility | FIPS 203/204/205; NIST SP 800-57; transition planning controls |

## 4. Control domains

### 4.1 GRC — Governance, Risk, and Compliance

| Requirement ID | Requirement |
| :-- | :-- |
| STD-GRC-001 | All production cryptographic implementations SHALL complete approved design documentation before deployment. |
| STD-GRC-002 | Critical System and CCS designs SHALL record business criticality, trust boundaries, dependencies, policy mappings, recovery assumptions, and algorithm-transition considerations. |
| STD-GRC-003 | Exceptions SHALL include rationale, compensating controls, expiry date, and risk acceptance. |
| STD-GRC-004 | Inventory and design records SHALL be reviewable for audit and regulatory evidence. |
| STD-GRC-005 | Deprecated algorithm and protocol use SHALL be tracked as a managed remediation item until retired. |
| STD-GRC-006 | The cryptographic inventory SHALL support fleet-wide change planning for algorithms, protocols, ciphers, key sizes, certificate profiles, and trust dependencies. |

### 4.2 CEK — Cryptography, Encryption, and Key Management

| Requirement ID | Requirement |
| :-- | :-- |
| STD-CEK-001 | Approved enterprise algorithms SHALL be used for new systems unless an exception is approved. |
| STD-CEK-002 | Forbidden algorithms and protocols SHALL NOT be used in production. |
| STD-CEK-003 | Envelope encryption SHALL be the default pattern for application-managed protected data. |
| STD-CEK-004 | Root keys, CA keys, high-value signing keys, and equivalent crown-jewel material SHALL use the strongest approved custody model. |
| STD-CEK-005 | Production keys, certificates, managed secrets, and material cryptographic settings SHALL be recorded in inventory. |
| STD-CEK-006 | New asymmetric designs SHALL document crypto-agility and PQC transition considerations. |
| STD-CEK-007 | Chinese algorithms such as SM2, SM3, and SM4 MAY be used where justified by interoperability or business need and documented in the design record. |
| STD-CEK-008 | Systems with long-lived confidentiality or signature requirements SHALL be assessed for hybrid and PQC migration priority. |

### 4.3 IAM — Identity and Access for Cryptographic Services

| Requirement ID | Requirement |
| :-- | :-- |
| STD-IAM-001 | Administrative access to HSM, KMS, vault, and CA platforms SHALL use enterprise identity and MFA. |
| STD-IAM-002 | Workloads SHALL authenticate using managed identity, federation, mTLS, or equivalent short-lived identity. |
| STD-IAM-003 | Long-lived embedded secrets SHALL NOT be used where approved short-lived identity patterns exist. |
| STD-IAM-004 | Secrets SHALL be retrieved at runtime from approved systems of record. |

### 4.4 DSP — Data Security and Protection

| Requirement ID | Requirement |
| :-- | :-- |
| STD-DSP-001 | Sensitive information in internet and private extranet channels SHALL use strong encryption in transit. |
| STD-DSP-002 | TLS 1.3 SHALL be the default for new services unless an exception is approved. |
| STD-DSP-003 | Passwords SHALL be stored using approved memory-hard hashing controls. |
| STD-DSP-004 | Signing, certificate, and integrity controls SHALL be used where business flows require authenticity or non-repudiation. |

### 4.5 LOG / IR — Logging, Monitoring, Incident Response, Recovery

| Requirement ID | Requirement |
| :-- | :-- |
| STD-LOG-001 | Cryptographic control-plane and secret-access events SHALL be logged and centrally reviewed. |
| STD-LOG-002 | Critical Systems and CCS SHALL have alerting for suspicious access, abnormal decrypt or sign activity, and trust-boundary changes. |
| STD-IR-001 | Key compromise, certificate compromise, and secret exposure SHALL trigger defined incident-response procedures. |
| STD-IR-002 | Recovery procedures SHALL include revocation, rotation, re-encryption where needed, and trust re-establishment. |

## 5. Algorithm standard

### 5.1 Algorithm selection principles

- The enterprise standard prefers algorithms and key sizes that provide stronger long-term assurance for critical infrastructure, regulated exchange services, and future crypto-agility.
- RSA-2048 is not treated as immediately broken, but it is not the preferred baseline for new enterprise designs because the enterprise wants stronger long-term assurance and cleaner transition toward post-quantum and hybrid models.
- RSA-2048 MAY remain acceptable by approved exception for current interoperability, legacy integration, or ecosystem constraints, provided the usage is inventoried, time-bounded where possible, and included in transition planning.
- ECDSA P-384, X25519, ECDHE P-384, AES-256-GCM, and Argon2id remain the preferred current classical baselines unless a more specific approved profile applies.

### 5.2 Approved, exception, and forbidden algorithms

| Use case | Preferred order | Permitted by exception | China-compatible option | PQC / agility note | Forbidden |
| :-- | :-- | :-- | :-- | :-- | :-- |
| Symmetric encryption | AES-256-GCM, then ChaCha20-Poly1305 where justified | AES-128-GCM for constrained compatibility only | SM4-GCM / SM4-CCM where justified | Keep metadata and abstraction stable | DES, 3DES, RC4, AES-ECB, unauthenticated CBC |
| Key exchange | X25519, then ECDHE P-384 | Legacy compatibility only | SM2 key exchange where justified | Design for ML-KEM-768 transition | Static DH, weak curves, RSA key transport |
| Public-key encryption / key wrapping | Envelope encryption with KMS-managed DEKs and KEKs | RSA-OAEP 2048 or stronger for legacy interoperability only | SM2 encryption where justified | Prefer KEM-style architecture | RSA PKCS#1 v1.5 encryption, RSA below 2048 bits |
| Signatures | ECDSA P-384, then Ed25519 where supported | RSA-PSS 2048 or 4096 for legacy compatibility only | SM2 + SM3 where justified | Plan for ML-DSA-65 and selected SLH-DSA use | DSA, SHA-1 signatures, RSA PKCS#1 v1.5 signatures |
| Hashing | SHA-384, then SHA-256 minimum | SHA-3 where justified | SM3 where justified | Record hash algorithm identifiers | MD5, SHA-1 |
| Password storage | Argon2id | PBKDF2-SHA256 where constrained and approved | No separate baseline | Review parameters periodically | Unsalted hashes, reversible storage, MD5, SHA-1-based password verification |
| TLS certificates and server auth | ECDSA P-384 certificates preferred; TLS 1.3 baseline | RSA-2048 or RSA-4096 certificates and TLS 1.2 by approved exception | SM2 certificates where justified | Keep PKI agile for hybrid transition and future certificate profile change | SHA-1-signed certs, RSA below 2048 bits, SSLv3, TLS 1.0, TLS 1.1 |

### 5.3 RSA-2048 rationale

RSA-2048 is acceptable only as an exception path because the enterprise baseline is deliberately set toward stronger long-term assurance and away from new lock-in to older classical-only designs. For regulated critical environments, the enterprise wants new systems to prefer elliptic-curve baselines and preserve a cleaner path to hybrid and PQC transition.

Where RSA-2048 remains necessary, teams SHALL record the reason, affected interfaces, dependent counterparties, certificate or signing scope, and target review or retirement date in inventory and design records.

## 6. Key lifecycle standard

| Lifecycle stage | Standard requirement |
| :-- | :-- |
| Design | Define use case, owner, classification, algorithm, key source, dependencies, and PQC impact |
| Generation | Use approved entropy and approved platform generation methods |
| Registration | Record inventory and link to design record |
| Activation | Enforce role-based access and approved usage policy |
| Rotation | Define and execute cryptoperiod-based rotation |
| Revocation / Disablement | Maintain emergency disablement and replacement procedures |
| Recovery | Maintain tested backup or trust-restoration procedures where applicable |
| Destruction | Destroy or retire material under controlled evidence-based procedures |

## 7. Identity and secret-access standard

| Area | Standard requirement |
| :-- | :-- |
| User authentication | 2FA or stronger controls for regulated and privileged access |
| Workload authentication | Managed identity, federation, workload identity, or mTLS |
| Secret delivery | Runtime retrieval only from approved sources |
| Administrative control | MFA, role separation, review, and logging |

## 8. Monitoring and incident standard

| Area | Standard requirement |
| :-- | :-- |
| Logging | Log create, rotate, disable, decrypt, sign, secret-read, and administrative events |
| Monitoring | Alert on suspicious patterns, failure states, unexpected principals, deprecated algorithm presence, and trust changes |
| Incident handling | Include cryptographic assets in compromise procedures and escalation paths |
| Recovery | Validate restoration of trust, dependency updates, and evidence retention |

## 9. Quantum-safe and crypto-agility standard

### 9.1 Planning baseline

| Area | Standard requirement |
| :-- | :-- |
| Crypto-agility | New asymmetric designs SHALL avoid hard-coded algorithm assumptions in business logic. |
| Long-lived confidentiality | Prioritise migration planning where compromise impact remains material into the quantum era. |
| Long-lived signatures | Plan for signature and verification transition where artefacts require long-term trust. |
| Algorithm metadata | Preserve versioning, algorithm identifiers, abstraction boundaries, and deployable preference ordering. |
| Inventory for change | Maintain inventory of protocols, certificates, key types, cipher suites, trust stores, signature usages, and counterparties to support future change campaigns. |
| Transition categories | Track each major usage as preferred, permitted by exception, deprecated, or forbidden. |

### 9.2 Migration direction

| Purpose | Current preferred baseline | Exception path | Transition direction | Target planning reference |
| :-- | :-- | :-- | :-- | :-- |
| Key establishment | X25519 / ECDHE P-384 | Legacy interoperability only | Hybrid classical + PQC | ML-KEM-768 |
| Signatures | ECDSA P-384 / Ed25519 | RSA-PSS 2048 or 4096 where approved | Hybrid classical + PQC | ML-DSA-65 |
| Selected high-assurance verification | Classical signing where required | Legacy validation path | Evaluate hybrid or parallel verification | SLH-DSA where justified |
| TLS certificates | ECDSA P-384 certificates | RSA-2048 or RSA-4096 by approved exception | Prepare for future hybrid or updated certificate profiles | Enterprise PKI roadmap |
| Symmetric encryption | AES-256-GCM | AES-128-GCM only where justified | No immediate PQC change required | AES-256-GCM |

### 9.3 Remediation categories

| Category | Treatment |
| :-- | :-- |
| Forbidden now | Must not be used in production |
| Deprecated | Allowed only for managed transition with approved remediation plan |
| Permitted by exception | Requires documented justification, inventory visibility, and review expiry |
| Preferred baseline | Use by default in new systems and major redesigns |

### 9.4 Inventory minimums for crypto-agility

The inventory used for crypto-agility SHALL capture at least the following where applicable:
- service or application name;
- business criticality and tier;
- protocol and version;
- certificate profile and issuing CA;
- key type and size;
- signing or encryption algorithm;
- cipher suites or equivalent negotiated settings;
- counterparty or dependency;
- exception status;
- planned target configuration.

## 10. Cloud and shared-responsibility alignment

The enterprise uses [CSA CCM](https://cloudsecurityalliance.org/research/cloud-controls-matrix) to align cloud obligations.

| Service model | Enterprise expectation |
| :-- | :-- |
| SaaS | Verify provider controls, contractual capability, logging visibility, encryption options, and tenant protections |
| PaaS | Use provider capabilities securely, enable customer-managed options where available, and protect workload identities and secrets |
| IaaS | Apply the full standard to workloads, platforms, secrets, certificates, and runtime controls under customer control |

## 11. Document relationships

This standard implements the policy and is implemented in turn by the engineering guidance in the [Enterprise Cryptographic Implementation Guide](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md).

## 12. References

| Ref ID | Reference | Link |
| :-- | :-- | :-- |
| S-R1 | Enterprise Cryptographic Policy | [1-Enterprise-Cryptographic-Policy-v1_2.md](1-Enterprise-Cryptographic-Policy-v1_2.md) |
| S-R2 | NIST CSF 2.0 | [https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf) |
| S-R3 | CSA Cloud Controls Matrix | [https://cloudsecurityalliance.org/research/cloud-controls-matrix](https://cloudsecurityalliance.org/research/cloud-controls-matrix) |
| S-R4 | NIST SP 800-53 Rev. 5 | [https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) |
| S-R5 | NIST SP 800-57 Part 1 Rev. 5 | [https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final) |
| S-R6 | NIST SP 800-175B Rev. 1 | [https://csrc.nist.gov/pubs/sp/800/175/b/r1/final](https://csrc.nist.gov/pubs/sp/800/175/b/r1/final) |
| S-R7 | FIPS 203 | [https://csrc.nist.gov/pubs/fips/203/final](https://csrc.nist.gov/pubs/fips/203/final) |
| S-R8 | FIPS 204 | [https://csrc.nist.gov/pubs/fips/204/final](https://csrc.nist.gov/pubs/fips/204/final) |
| S-R9 | FIPS 205 | [https://csrc.nist.gov/pubs/fips/205/final](https://csrc.nist.gov/pubs/fips/205/final) |
| S-R10 | OWASP Cryptographic Storage Cheat Sheet | [https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html) |
| S-R11 | Enterprise Cryptographic Implementation Guide | [3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md) |
| S-R12 | Enterprise Cryptography and Key/Secret Management Document Plan | [4-Enterprise-Cryptography-and-Key-Secret-Management-Document-Plan-v1_0.md](4-Enterprise-Cryptography-and-Key-Secret-Management-Document-Plan-v1_0.md) |
