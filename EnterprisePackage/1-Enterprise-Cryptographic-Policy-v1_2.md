# Enterprise Cryptographic Policy

**Classification:** Internal — Restricted  
**Version:** 1.2  
**Date:** March 2026  
**Owner:** Information Security Architecture  
**Applies to:** All systems, with stricter requirements for Critical Systems and Critical Computer Systems (CCS)

---

## Table of Contents

- [1. Purpose](#1-purpose)
- [2. Policy hierarchy](#2-policy-hierarchy)
- [3. Scope and tiering](#3-scope-and-tiering)
- [4. High-level policy requirements](#4-high-level-policy-requirements)
- [5. Quantum-safe and crypto-agility policy](#5-quantum-safe-and-crypto-agility-policy)
- [6. Governance and traceability](#6-governance-and-traceability)
- [7. Document relationships](#7-document-relationships)
- [8. References](#8-references)

---

## 1. Purpose

This document defines the enterprise **policy** requirements for cryptography, encryption, key management, certificate management, secret management, signing, and cryptographic trust services.

It translates the following into mandatory enterprise policy outcomes:
- Hong Kong critical-infrastructure obligations under Cap. 653;
- SFC expectations for regulated internet and private-extranet access;
- enterprise requirements for approved algorithms, secure key custody, runtime secret retrieval, auditability, resilience, recoverability, and crypto-agility.

This policy is supported by:
- [Enterprise Cryptographic Standard](2-Enterprise-Cryptographic-Standard-v1_2.md)
- [Enterprise Cryptographic Implementation Guide](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md)
- [Enterprise Cryptography and Key/Secret Management Document Plan](4-Enterprise-Cryptography-and-Key-Secret-Management-Document-Plan-v1_0.md)

## 2. Policy hierarchy

| Tier | Source | Role |
| :-- | :-- | :-- |
| Tier 0 | [Cap. 653 Protection of Critical Infrastructures (Computer Systems) Ordinance](https://www.elegislation.gov.hk/hk/cap653) | Legal driver for critical-infrastructure computer-system security |
| Tier 0.5 | [SFC Cybersecurity FAQ](https://www.sfc.hk/en/faqs/intermediaries/supervision/Cybersecurity/Cybersecurity) and related internet-trading guidance | Sector-specific policy expectations for exchange and market-facing services |
| Tier 1 | This Policy | High-level enterprise policy requirements |
| Tier 2 | [NIST CSF 2.0](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf) | Internal cyber governance framework |
| Tier 2 | [CSA Cloud Controls Matrix](https://cloudsecurityalliance.org/research/cloud-controls-matrix) | Cloud alignment and shared-responsibility framework |
| Tier 3 | [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final), [NIST SP 800-57 Part 1 Rev. 5](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final), [NIST SP 800-175B Rev. 1](https://csrc.nist.gov/pubs/sp/800/175/b/r1/final), [OWASP Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html) | Lower-level control and engineering guidance |
| Tier 4 | Platform and provider implementation guidance | Technology-specific implementation detail |

## 3. Scope and tiering

This policy applies to **all systems**.

Requirements intensify by system tier:

| Tier | Description | Policy expectation |
| :-- | :-- | :-- |
| Baseline | All systems | Mandatory minimum cryptographic governance and secure usage |
| Critical System | Systems whose compromise materially affects security, resilience, identity, transaction integrity, or customer protection | Enhanced controls and stronger evidence |
| CCS / Crown Jewel | Critical Computer Systems and equivalent trust-boundary or regulated-service dependencies | Strictest controls, auditability, recovery readiness, and regulator-facing evidence |

## 4. High-level policy requirements

### 4.1 Cap. 653 policy requirements

| Policy ID | Requirement |
| :-- | :-- |
| POLICY-CI-001 | All systems SHALL implement enterprise-approved security controls, with stricter controls for Critical Systems and CCS. |
| POLICY-CI-002 | The organisation SHALL identify systems that are critical to regulated exchange and financial-service operations, including enabling and protective dependencies. |
| POLICY-CI-003 | The organisation SHALL maintain a computer-system security management capability, including clear governance for cryptography, secrets, keys, certificates, and trust services. |
| POLICY-CI-004 | The organisation SHALL maintain and implement a management plan covering cryptographic governance, key custody, identity, logging, incident response, resilience, and recovery. |
| POLICY-CI-005 | Critical Systems and CCS SHALL be subject to periodic risk assessment, audit, drill, incident reporting, and emergency planning, with cryptographic dependencies included in scope. |
| POLICY-CI-006 | Material changes affecting CCS scope, trust boundaries, identity architecture, connectivity, or key-management architecture SHALL be governed and escalated. |
| POLICY-CI-007 | Cryptographic controls SHALL support resilience, backup, restoration, trust continuity, and secure recovery of critical services. |

### 4.2 SFC policy requirements

| Policy ID | Requirement |
| :-- | :-- |
| POLICY-SFC-001 | Internet-facing and private-extranet-accessible regulated services SHALL use strong encryption for sensitive information in transit. |
| POLICY-SFC-002 | Client, participant, member, privileged, and administrative access to regulated services SHALL use 2FA or stronger approved MFA controls. |
| POLICY-SFC-003 | Passwords and equivalent authentication secrets stored by regulated systems SHALL be protected using strong cryptographic protection and approved password-storage controls. |
| POLICY-SFC-004 | Regulated services SHALL implement monitoring for unauthorised access, anomalous login patterns, and suspicious account activity. |
| POLICY-SFC-005 | Regulated services SHALL support security-significant notification and response workflows where required by process, rule, or risk treatment. |
| POLICY-SFC-006 | Regulated services SHALL preserve integrity, availability, auditability, and trusted recovery across internet and private extranet channels. |

### 4.3 Enterprise cryptography policy requirements

| Policy ID | Requirement |
| :-- | :-- |
| POLICY-CRY-001 | Approved enterprise cryptographic algorithms and protocols SHALL be used for all new systems. |
| POLICY-CRY-002 | Forbidden algorithms and protocols SHALL NOT be used in production. |
| POLICY-CRY-003 | Production keys, certificates, managed secrets, and material cryptographic settings SHALL be inventoried and linked to design and ownership records. |
| POLICY-CRY-004 | Root keys, trust anchors, high-value signing keys, and equivalent crown-jewel keys SHALL use the strongest approved custody model. |
| POLICY-CRY-005 | Secrets SHALL be retrieved at runtime from approved vault or secret-management services and SHALL NOT be embedded in code, images, or plaintext configuration. |
| POLICY-CRY-006 | Cryptographic designs SHALL be approved before production deployment. |
| POLICY-CRY-007 | New asymmetric designs SHALL include a crypto-agility or PQC transition path. |
| POLICY-CRY-008 | International cryptographic standards are the primary baseline; Chinese standards SHALL be used where justified by interoperability, jurisdictional, ecosystem, or customer need. |
| POLICY-CRY-009 | Deprecated algorithms and protocols SHALL be tracked and retired according to approved transition plans. |
| POLICY-CRY-010 | The organisation SHALL maintain a cryptographic inventory sufficient to support future algorithm transition, policy updates, and fleet-wide configuration change campaigns. |

## 5. Quantum-safe and crypto-agility policy

The organisation SHALL maintain crypto-agility for new asymmetric designs and long-lived protection use cases.

| Policy ID | Requirement |
| :-- | :-- |
| POLICY-PQC-001 | New asymmetric designs SHALL document a crypto-agility strategy and identify future PQC migration paths. |
| POLICY-PQC-002 | Systems protecting long-lived confidential data or long-lived signatures SHALL be prioritised for hybrid and PQC transition planning. |
| POLICY-PQC-003 | Approved current planning targets SHALL include ML-KEM-768 for key-establishment transition, ML-DSA-65 for signature transition, and SLH-DSA for selected high-assurance or long-lived verification cases. |
| POLICY-PQC-004 | Classical-only assumptions for new long-lived asymmetric designs SHALL NOT be accepted without explicit exception approval. |
| POLICY-PQC-005 | Algorithm metadata, versioning, abstraction, and deployable preference ordering SHALL be preserved so algorithms can be changed without major business-logic rewrites. |
| POLICY-PQC-006 | The cryptographic inventory SHALL identify preferred, permitted-by-exception, deprecated, and forbidden algorithm usage so configuration changes can be planned and executed in a controlled way. |

## 6. Governance and traceability

This policy uses [NIST CSF 2.0](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf) as the internal governance structure and [CSA CCM](https://cloudsecurityalliance.org/research/cloud-controls-matrix) for cloud alignment.

Lower-level control detail is defined in the linked standard and implementation guide.

| Policy area | NIST CSF 2.0 | CSA CCM | Lower-level document |
| :-- | :-- | :-- | :-- |
| Governance and accountability | Govern | GRC | [Enterprise Cryptographic Standard](2-Enterprise-Cryptographic-Standard-v1_2.md) |
| Cryptography and key management | Protect | CEK | [Enterprise Cryptographic Standard](2-Enterprise-Cryptographic-Standard-v1_2.md) |
| Identity and secret access | Protect | IAM | [Enterprise Cryptographic Standard](2-Enterprise-Cryptographic-Standard-v1_2.md), [Implementation Guide](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md) |
| Cloud implementation and shared responsibility | Govern / Protect | IVS, CCC, TVM, CEK | [Enterprise Cryptographic Standard](2-Enterprise-Cryptographic-Standard-v1_2.md), [Implementation Guide](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md) |
| Monitoring, response, and recovery | Detect / Respond / Recover | LOG, SEF, BCR | [Enterprise Cryptographic Standard](2-Enterprise-Cryptographic-Standard-v1_2.md), [Implementation Guide](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md) |
| Quantum-safe planning and crypto-agility | Govern / Protect | CEK | [Enterprise Cryptographic Standard](2-Enterprise-Cryptographic-Standard-v1_2.md), [Implementation Guide](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md) |
| Secure engineering patterns | Protect | DSP, CEK, TVM | [Enterprise Cryptographic Implementation Guide](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md) |

## 7. Document relationships

This policy states **what** the organisation requires.

The supporting documents define:
- [Enterprise Cryptographic Standard](2-Enterprise-Cryptographic-Standard-v1_2.md) — the control requirements, mappings, measurable standards, approved and forbidden algorithms, exception conditions, and PQC planning requirements;
- [Enterprise Cryptographic Implementation Guide](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md) — the platform patterns, engineering guidance, migration planning, inventory expectations, and example implementation approaches;
- [Enterprise Cryptography and Key/Secret Management Document Plan](4-Enterprise-Cryptography-and-Key-Secret-Management-Document-Plan-v1_0.md) — the recommended long-term document structure and overlap-reduction approach.

## 8. References

| Ref ID | Reference | Link |
| :-- | :-- | :-- |
| P-R1 | Cap. 653 Protection of Critical Infrastructures (Computer Systems) Ordinance | [https://www.elegislation.gov.hk/hk/cap653](https://www.elegislation.gov.hk/hk/cap653) |
| P-R2 | Cap. 653 Schedule 1 | [https://www.elegislation.gov.hk/hk/cap653!en/sch1?_lang=en](https://www.elegislation.gov.hk/hk/cap653!en/sch1?_lang=en) |
| P-R3 | Official PCICSO / Cap. 653 summary | [https://www.coms-auth.hk/en/policies_regulations/other/pcicso/index.html](https://www.coms-auth.hk/en/policies_regulations/other/pcicso/index.html) |
| P-R4 | SFC Cybersecurity FAQ | [https://www.sfc.hk/en/faqs/intermediaries/supervision/Cybersecurity/Cybersecurity](https://www.sfc.hk/en/faqs/intermediaries/supervision/Cybersecurity/Cybersecurity) |
| P-R5 | NIST CSF 2.0 | [https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf) |
| P-R6 | CSA Cloud Controls Matrix | [https://cloudsecurityalliance.org/research/cloud-controls-matrix](https://cloudsecurityalliance.org/research/cloud-controls-matrix) |
| P-R7 | Enterprise Cryptographic Standard | [2-Enterprise-Cryptographic-Standard-v1_2.md](2-Enterprise-Cryptographic-Standard-v1_2.md) |
| P-R8 | Enterprise Cryptographic Implementation Guide | [3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md](3-Enterprise-Cryptographic-Implementation-Guide-v1_2.md) |
| P-R9 | Enterprise Cryptography and Key/Secret Management Document Plan | [4-Enterprise-Cryptography-and-Key-Secret-Management-Document-Plan-v1_0.md](4-Enterprise-Cryptography-and-Key-Secret-Management-Document-Plan-v1_0.md) |
