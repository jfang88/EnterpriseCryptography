# Enterprise Cryptography and Key/Secret Management Document Plan

**Classification:** Internal — Restricted  
**Version:** 1.0  
**Date:** March 2026  
**Owner:** Information Security Architecture

---

## Table of Contents

- [1. Purpose](#1-purpose)
- [2. Overlap reduction approach](#2-overlap-reduction-approach)
- [3. Recommended document structure](#3-recommended-document-structure)
- [4. Two-set model](#4-two-set-model)
- [5. Ownership model](#5-ownership-model)
- [6. Cross-reference model](#6-cross-reference-model)
- [7. Build plan](#7-build-plan)
- [8. References](#8-references)

---

## 1. Purpose

This plan explains how to reduce overlap between enterprise **cryptography** and enterprise **key/secret management** documentation while preserving clear ownership, traceability, and implementation guidance.

It assumes the enterprise may maintain either:
- one combined cryptographic management suite; or
- two related but distinct document suites, one for enterprise cryptography and one for enterprise key and secret management.

## 2. Overlap reduction approach

The main way to reduce overlap is to define **one topic owner per concept**.

| Topic | Primary owner document set | Secondary references only |
| :-- | :-- | :-- |
| Approved algorithms, banned algorithms, deprecated algorithms, PQC direction | Enterprise Cryptography | Key/secret documents reference only |
| Key lifecycle and key custody for cryptographic keys | Enterprise Cryptography | Key/secret documents reference only where operationally needed |
| Vault usage, runtime secret retrieval, rotation workflows, application secret delivery | Key/Secret Management | Cryptography documents reference only |
| Certificates, signing, PKI, trust anchors | Enterprise Cryptography | Key/secret documents reference only |
| Password storage and authentication secret handling | Shared, but cryptography owns algorithms and key/secret management owns delivery and lifecycle operations | Mutual cross-reference |
| Cloud KMS and HSM governance | Enterprise Cryptography | Key/secret management references service-consumption patterns |
| Secret-store integrations, ESO, CSI, CyberArk application delivery | Key/Secret Management | Cryptography documents reference approved pattern |

### Core rule

- Put **what algorithm, why, and when it is allowed** in the cryptography set.
- Put **how secrets are delivered, rotated, injected, brokered, and operated** in the key/secret set.
- Use references instead of duplicated prose whenever possible.

## 3. Recommended document structure

### Option A — One combined suite

1. Policy
2. Standard
3. Implementation Guide
4. Appendices / examples

Use this when the same team owns cryptography, KMS, vaults, certificates, and secret delivery.

### Option B — Two linked suites

#### Suite 1: Enterprise Cryptography
1. Cryptographic Policy
2. Cryptographic Standard
3. Cryptographic Implementation Guide

#### Suite 2: Enterprise Key and Secret Management
1. Key and Secret Management Policy
2. Key and Secret Management Standard
3. Key and Secret Management Implementation Guide

Use this when cryptography governance and key/secret platform operations have different owners or when the secret-management operating model needs much deeper operational content.

## 4. Two-set model

### 4.1 Enterprise Cryptography suite should own

- Regulatory and policy drivers for encryption, signing, certificates, and crypto-agility.
- Approved algorithms, banned algorithms, deprecated algorithms, and China-compatible algorithm positioning.
- PQC planning, crypto-agility, trust services, certificate policy, code signing, and long-lived confidentiality strategy.
- Key lifecycle rules for cryptographic keys.

### 4.2 Enterprise Key and Secret Management suite should own

- KMS and vault operating model.
- Secret delivery patterns for applications, VMs, Kubernetes, CI/CD, and administrators.
- Rotation operations, credential brokering, dynamic secrets, lease management, and break-glass procedures.
- Service onboarding, platform ownership, recovery runbooks, and platform SLOs for KMS, vault, or PAM-related systems.

### 4.3 Shared areas

| Shared area | Recommended owner | Reference approach |
| :-- | :-- | :-- |
| Password storage algorithm | Cryptography | Key/secret guide references cryptography standard |
| Password reset and secret rotation operations | Key/Secret Management | Cryptography policy references operational control |
| KMS for envelope encryption | Cryptography | Key/secret guide references operational service pattern |
| Secret retrieval from vault | Key/Secret Management | Cryptography standard points to approved delivery patterns |
| Inventory and evidence model | Shared governance | Maintain one common appendix or control catalog |

## 5. Ownership model

| Role | Cryptography suite | Key/secret suite |
| :-- | :-- | :-- |
| Information Security Architecture | Owns policy, algorithm standard, PQC direction, trust model | Co-owns platform control requirements |
| Cloud / Platform Security | Contributes platform patterns and service controls | Co-owns platform implementation and operations |
| IAM / Vault / PAM team | Consulted | Primary owner |
| PKI / Trust Services team | Primary contributor | Consulted |
| SOC / IR | Shared | Shared |
| Audit / Compliance | Shared | Shared |

## 6. Cross-reference model

Use a simple reference pattern across both suites.

| ID prefix | Use |
| :-- | :-- |
| CRY-POL-* | Cryptography policy requirements |
| CRY-STD-* | Cryptography standard requirements |
| CRY-IMPL-* | Cryptography implementation references |
| KSM-POL-* | Key and secret management policy requirements |
| KSM-STD-* | Key and secret management standard requirements |
| KSM-IMPL-* | Key and secret management implementation references |
| XREF-* | Shared cross-reference rows or common evidence mappings |

Maintain one shared traceability appendix or CSV that maps:
- policy ID;
- standard ID;
- implementation section;
- owner;
- evidence;
- related regulatory driver.

## 7. Build plan

### Phase 1 — Stabilise the cryptography suite

Create and approve:
1. [Enterprise Cryptographic Policy](1-Enterprise-Cryptographic-Policy-v1_1.md)
2. [Enterprise Cryptographic Standard](2-Enterprise-Cryptographic-Standard-v1_1.md)
3. [Enterprise Cryptographic Implementation Guide](3-Enterprise-Cryptographic-Implementation-Guide-v1_1.md)

### Phase 2 — Create the key and secret management suite

Build:
1. Enterprise Key and Secret Management Policy
2. Enterprise Key and Secret Management Standard
3. Enterprise Key and Secret Management Implementation Guide

### Phase 3 — Create shared appendices

Maintain shared appendices for:
- inventory schema;
- traceability matrix;
- reference catalog;
- evidence catalog;
- platform ownership map.

### Phase 4 — Remove duplicated content

For each duplicated section:
1. Choose the primary owner suite.
2. Keep the detailed content only there.
3. Replace the duplicate section in the other suite with a short summary and a cross-reference link.
4. Update the traceability matrix.

## 8. References

| Ref ID | Reference | Link |
| :-- | :-- | :-- |
| D-R1 | Enterprise Cryptographic Policy | [1-Enterprise-Cryptographic-Policy-v1_1.md](1-Enterprise-Cryptographic-Policy-v1_1.md) |
| D-R2 | Enterprise Cryptographic Standard | [2-Enterprise-Cryptographic-Standard-v1_1.md](2-Enterprise-Cryptographic-Standard-v1_1.md) |
| D-R3 | Enterprise Cryptographic Implementation Guide | [3-Enterprise-Cryptographic-Implementation-Guide-v1_1.md](3-Enterprise-Cryptographic-Implementation-Guide-v1_1.md) |
| D-R4 | NIST CSF 2.0 | [https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf) |
| D-R5 | CSA Cloud Controls Matrix | [https://cloudsecurityalliance.org/research/cloud-controls-matrix](https://cloudsecurityalliance.org/research/cloud-controls-matrix) |
