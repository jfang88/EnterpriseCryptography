# Cryptography Management Implementation Guide v1.1

**Organization Type:** Critical Financial Services Provider  
**Document Type:** Implementation Guide  
**Version:** 1.1  
**Classification:** Internal Restricted  
**Review Cycle:** Annual  
**Companion Standards:** Secrets, Keys, and Cryptography Technical Standards v1.5

**Status:** Approved Implementation Guide

---

## Table of Contents

1. [Purpose and Scope](#1-purpose-and-scope)
2. [Implementation Model](#2-implementation-model)
3. [Platform Mapping](#3-platform-mapping)
4. [Huawei Cloud KMS](#4-huawei-cloud-kms)
5. [AWS KMS](#5-aws-kms)
6. [VMware vHSM](#6-vmware-vhsm)
7. [CloudHSM / Dedicated HSM](#7-cloudhsm--dedicated-hsm)
8. [Shared Responsibility Matrices](#8-shared-responsibility-matrices)
9. [Key Custody Models](#9-key-custody-models)
10. [Migration and Onboarding](#10-migration-and-onboarding)
11. [Troubleshooting and Operations](#11-troubleshooting-and-operations)

**Appendices:**
A. [API Examples and Configuration](#appendix-a-api-examples-and-configuration)
B. [Control Mapping to Standards Doc #2](#appendix-b-control-mapping-to-standards-doc-2)

---

## 1. Purpose and Scope

This guide provides platform-specific implementation patterns for **cryptographic key management only** (generation, protection, usage, rotation of DEKs/KEKs/signing keys).

**Secrets management** (API keys/service credentials) is covered in the companion **Secrets Management Implementation Guide**.

**Reference:** Secrets, Keys, and Cryptography Technical Standards v1.5 (Standards Doc #2) for lifecycle requirements and section references, and DOC-05 (Approved Algorithms and PQC Path Summary) for the approved algorithm baseline.

### 1.1 Objectives

- Provide tested key management patterns for each platform.
- Map platform capabilities to Standards Doc #2 sections (§5 key requirements, §9 encryption, §12 rotation).
- Document BYOK/CMK/HYOK custody models.
- Enable consistent cryptographic key handling across hybrid infrastructure.

### 1.2 Platforms Covered

| Platform | Key Service | Custody Model | Status |
|----------|-------------|---------------|--------|
| Huawei Cloud | KMS | CMK/BYOK | Primary private cloud |
| AWS | KMS | CMK/BYOK | Public cloud primary |
| VMware | vHSM | vHSM cluster | On-prem/VMware |
| Multi-cloud | CloudHSM | Dedicated HSM | High-value keys |

---

## 2. Implementation Model

### 2.1 Core Principles

All implementations must satisfy Standards Doc #2:

- **Key Generation**: CSPRNG/HSM-backed
- **Custody**: Enterprise-controlled (CMK/BYOK/HYOK)
- **Usage**: API-only (no export for high-value)
- **Rotation**: Cryptoperiod-based
- **Audit**: Key policy/usage events to SIEM

### 2.2 Custody Decision Framework

| Sensitivity | Recommended Custody |
|-------------|---------------------|
| Tier 1 (roots/signing) | Dedicated HSM/vHSM |
| Tier 2 (KEKs) | CloudHSM/KMS CMK |
| Tier 3 (DEKs) | KMS CMK (BYOK where key origin must be enterprise-controlled) |
| Tier 4 | Provider-managed (exception) |

---

## 3. Platform Mapping

| Capability | Huawei KMS | AWS KMS | vHSM | CloudHSM |
|------------|------------|---------|------|----------|
| Key Types | Symmetric | Symmetric/Asym | PKCS#11 | PKCS#11 |
| BYOK Import | Yes | Yes | Yes | Yes |
| Rotation | Manual/API | Automatic | Manual | Manual |
| Audit | Cloud logs | CloudTrail | vHSM logs | HSM logs |
| FIPS | Yes | FIPS mode | FIPS validated | FIPS validated |

---

## 4. Huawei Cloud KMS

### 4.1 Overview

Huawei KMS provides customer-managed keys (CMK) for data protection and CSMS secret encryption.

**Maps to Standards Doc #2:**
- Custody: CMK → §5.5 (key hierarchy) and §9.1 (encryption at rest)
- Rotation: API → §12.2 (rotation schedule)
- Audit: Cloud logs → §16.1 (events that must be logged)

### 4.2 Core Patterns

**Pattern 1: CMK Creation**
Examples use the Huawei Cloud CLI (KooCLI, `hcloud`). They are illustrative; validate operation and parameter names against the current KooCLI KMS reference before use.

```bash
hcloud KMS CreateKey \
  --cli-region={region} \
  --key_alias="payments-kek" \
  --key_spec=AES_256 \
  --key_usage=ENCRYPT_DECRYPT
```

**Pattern 2: BYOK Import**
```bash
# 1. Obtain the wrapping public key and import token
hcloud KMS CreateParametersForImport \
  --cli-region={region} \
  --key_id={cmk-id} \
  --wrapping_algorithm=RSAES_OAEP_SHA_256

# 2. Import the key material wrapped under that public key
hcloud KMS ImportKeyMaterial \
  --cli-region={region} \
  --key_id={cmk-id} \
  --import_token={import-token} \
  --encrypted_key_material={wrapped-material}
```

RSA-OAEP import wrapping is quantum-vulnerable. Perform the import over a hybrid TLS 1.3 channel where the endpoint supports it, use the import token once, and track vendor support for ML-KEM based import (Standards Doc #2, PQC migration requirements).

### 4.3 Integration Points

- **CSMS Secrets**: Secrets encrypted by KMS CMK
- **CCE/Object Storage**: Runtime encryption
- **Logs**: Cloud Log Service → SIEM

---

## 5. AWS KMS

### 5.1 Overview

AWS KMS for symmetric/asymmetric keys with automatic rotation and CloudTrail audit.

### 5.2 Core Patterns

**Pattern 1: CMK w/ Rotation**
```bash
aws kms create-key \
  --key-usage ENCRYPT_DECRYPT \
  --key-spec SYMMETRIC_DEFAULT \
  --description "Payments KEK"

aws kms enable-key-rotation \
  --key-id {key-id} \
  --rotation-period-in-days 365
```

**Pattern 2: BYOK**
```bash
aws kms import-key-material \
  --key-id {key-id} \
  --import-token fileb://import-token.bin \
  --encrypted-key-material fileb://wrapped.bin \
  --expiration-model KEY_MATERIAL_EXPIRES \
  --expiration-time 2026-12-31T00:00:00Z
```

**Multi-Region Replication:**
```bash
aws kms replicate-key \
  --key-id arn:...:primary-region \
  --replica-region us-west-2
```

---

## 6. VMware vHSM

### 6.1 Overview

VMware-validated Hardware Security Module for vSphere/NSX with PKCS#11 interface.

### 6.2 Core Patterns

**vHSM Cluster Setup:**
```bash
# HSM partition (token) initialisation, performed under dual control
pkcs11-tool --module /opt/vhsm/libvhsm_pkcs11.so \
  --init-token --label "vHSM-Prod" --so-pin "$VHSM_SO_PIN"
```

**PKCS#11 Integration:**
```
# App config. The user PIN is retrieved from the secrets platform at runtime;
# it is never written to configuration files.
library=/opt/vhsm/libvhsm_pkcs11.so
slot=0
```

**Key Generation:**
```bash
pkcs11-tool --module /opt/vhsm/libvhsm_pkcs11.so \
  --login --pin "$VHSM_USER_PIN" \
  --keypairgen --key-type EC:secp384r1 \
  --label "TLS-Signing-Key"
```

---

## 7. CloudHSM / Dedicated HSM

### 7.1 Overview

Dedicated HSM partitions (AWS CloudHSM, Azure Dedicated HSM) for Tier 1 keys.

### 7.2 Core Patterns

**AWS CloudHSM Cluster:**
```bash
aws cloudhsmv2 create-cluster \
  --subnet-ids subnet-123 subnet-456 \
  --hsm-type hsm1.medium
```

**PKCS#11 Key Gen:**
```bash
pkcs11-tool --module /opt/cloudhsm/lib/libcloudhsm_pkcs11.so \
  --login --pin "$CLOUDHSM_CU_CREDENTIAL" \
  --keypairgen --key-type EC:secp384r1 \
  --label "Tier1-Signing-Key"
```

Tier 1 keys use P-384 in line with Standards Doc #2 section 5.4.

---

## 8. Shared Responsibility Matrices

### 8.1 Huawei KMS

| Control | Enterprise | Huawei |
|---------|------------|---------|
| Key Policy | CMK config | KMS service |
| Key Material | BYOK custody | Generate/rotate |
| Audit Logs | SIEM integration | Raw Cloud Logs |

### 8.2 AWS KMS

| Control | Enterprise | AWS |
|---------|------------|---------|
| Rotation Policy | Enable/disable | Automatic execution |
| Grants/Policies | IAM/KMS policy | Enforcement |
| CloudTrail | Subscription | Log delivery |

---

## 9. Key Custody Models

| Model | Use Case | Platforms | Standards Ref |
|-------|----------|-----------|---------------|
| **CMK** | Tier 2 KEKs and Tier 3 DEKs | All cloud KMS | §5.3, §9.1 |
| **BYOK** | Import existing | KMS w/ wrapping | §5.5, §9.1 |
| **HYOK** | Export prohibited | vHSM/CloudHSM | §5.4, §9.4 |
| **Provider-Managed** | Tier 4 only | Exception | §9.1; Policy §7 |

---

## 10. Migration and Onboarding

1. **Inventory existing keys** (type/location/custody)
2. **Map to custody model** above
3. **Test key operations** (encrypt/decrypt/rotate)
4. **Cutover** w/ dual-key validation
5. **Update dependents** (CSMS policies, app config)

---

## 11. Troubleshooting and Operations

**Common Issues:**
| Issue | Platform | Resolution |
|-------|----------|------------|
| KMS quota exceeded | Huawei/AWS | Request increase |
| vHSM cluster fail | VMware | Backup/restore partition |
| BYOK unwrap fail | All | Check wrapping algo/key size |

**Monitoring:**
- Key policy changes
- Rotation failures
- Grant/usage anomalies

---

# Appendix A: API Examples and Configuration

**Huawei KMS Enable Key Rotation:**
```bash
hcloud KMS EnableKeyRotation \
  --cli-region={region} \
  --key_id={cmk-id}

hcloud KMS UpdateKeyRotationInterval \
  --cli-region={region} \
  --key_id={cmk-id} \
  --rotation_interval=365
```

**AWS KMS Grant Example:**
```json
{
  "GranteePrincipal": "arn:aws:iam::account:role/app-role",
  "Operations": ["Decrypt", "GenerateDataKey"]
}
```

# Appendix B: Control Mapping to Standards Doc #2

| Platform Pattern | Standards Doc #2 Section |
|------------------|--------------------------|
| Huawei KMS CMK | §5.5, §9.1 (custody) |
| AWS Auto Rotation | §12.2 (rotation) |
| vHSM PKCS#11 | §9.4 (HSM-backed operations) |
| CloudHSM Partition | §5.4, §9.1 (Tier 1 custody) |

---

*References Standards Doc #2 for full technical requirements.*

## Algorithm Positioning

The approved algorithm baseline and the per-use-case PQC path are defined in DOC-05; the table below is a summary aligned to Standards Doc #2.

| Algorithm | PQC? | Recommended status |
|---|---|---|
| ML-KEM (FIPS 203) | Yes | Preferred KEM. ML-KEM-768 by default, deployed as a hybrid with ECDHE; enable now where supported |
| ML-DSA (FIPS 204) | Yes | Preferred PQC signature. ML-DSA-65 by default; ML-DSA-87 for root CAs and long-lived roots |
| SLH-DSA (FIPS 205) | Yes | Alternative PQC signature for roots, firmware, and long-horizon cases |
| LMS / XMSS (SP 800-208) | Yes | Firmware and boot signing only, with HSM-enforced state management |
| FN-DSA, HQC | Yes, standards in development | Monitor; do not adopt yet |
| RSA | No | Approved now (3072-bit minimum, no new RSA-2048); deprecated by NIST after 2030; replaced through the PQC programme |
| ECDSA / EdDSA | No | Approved now; deprecated by NIST after 2030; replaced through the PQC programme |
| ECDH | No | Approved as the classical half of a hybrid or as a fallback; hybridize with ML-KEM in key-establishment paths |
| AES-256 | No, remains secure | Symmetric baseline (GCM for data, XTS for disks, KW / KWP for key wrapping); no PQC change required |
| SHA-256 / SHA-384 / SHA-512 | No, remains secure | SHA-256 minimum, SHA-384 preferred; no PQC change required |

## Post-Quantum Roadmap

### Recommended timeline

PQC migration is run as a separate programme (Standards Doc #2, Post-Quantum Cryptography Transition). The phases below are the programme's indicative plan for KMS, HSM, and PKI platforms, not obligations on platform teams. Outside the programme, teams only enable hybrid key exchange where their current software already supports it (DOC-05, section 3).

| Period | Programme activity |
|---|---|
| 2026 (now) | Enable hybrid key establishment (`X25519MLKEM768` for TLS 1.3, `mlkem768x25519-sha256` for SSH) wherever supported. Complete cryptographic inventory for RSA, ECDSA, EdDSA, ECDH, SSH, X.509, TLS, VPN, code signing, and KMS/HSM dependencies |
| 2027 | Extend hybrid key establishment to remaining TLS, VPN, and key-transport paths. Enable crypto-agile designs, capture algorithm metadata, and pilot ML-DSA in non-production PKI and code signing |
| 2028 | Pilot ML-DSA or supported PQC signatures in production for selected trust chains; begin firmware and long-lived root migration |
| 2029 | Target priority production migration for externally exposed, identity-bearing, and long-retention systems, with documented exceptions for unsupported platforms |

### Platform recommendations

- **Huawei KMS / cloud KMS services:** Track vendor support for PQC key establishment, certificate, and signing workflows; do not assume current KMS APIs alone deliver full PQC trust-chain migration.
- **AWS KMS:** Monitor and adopt supported PQC or hybrid capabilities as exposed through AWS cryptographic services, TLS endpoints, or related key-management integrations.
- **vHSM / CloudHSM:** Use where enterprise-controlled key custody or signature workflows need higher assurance during phased migration; validate library, firmware, PKCS#11, and application compatibility early.
- **PKI and code signing:** Expected to be the largest PQC programme workstreams, because authentication and signature trust chains take longest to replace.

### Implementation guidance

- Prefer **hybrid** constructions during transition where platform support exists. For TLS follow the version, group, and cipher suite priority order in DOC-05 section 2.2.
- Avoid hard-coded assumptions that a key type, certificate, or signature algorithm will remain classical.
- Store algorithm identifiers, parameter sets, certificate profile versions, and ciphertext metadata so data and trust objects remain interpretable after migration.
- Update operational runbooks for fallback, rollback, logging, performance validation, and interop testing.

## References

### Internal Documents

| Doc Ref | Document | Version |
|---|---|---|
| DOC-01 | Secrets, Keys, and Cryptography Policy | v1.1 |
| DOC-02 | Secrets, Keys, and Cryptography Technical Standards | v1.5 |
| DOC-03 | Secrets Management Implementation Guide | v1.1 |
| DOC-04 | Cryptography Management Implementation Guide | v1.1 |
| DOC-05 | Approved Algorithms and PQC Path Summary | v0.3 (draft for approval) |

### External Sources

| Source ID | Reference | URL |
|---|---|---|
| EXT-01 | AWS Key Management Service Documentation | [https://docs.aws.amazon.com/kms/](https://docs.aws.amazon.com/kms/) |
| EXT-02 | AWS CloudHSM Documentation | [https://docs.aws.amazon.com/cloudhsm/](https://docs.aws.amazon.com/cloudhsm/) |
| EXT-03 | Huawei Cloud KMS for Encryption | [https://support.huaweicloud.com/intl/en-us/usermanual-dew/dew_01_0094.html](https://support.huaweicloud.com/intl/en-us/usermanual-dew/dew_01_0094.html) |
| EXT-04 | NIST SP 800-57 Part 1 Rev. 5 | [https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final) |
| EXT-11 | NIST Post-Quantum Cryptography (FIPS 203, 204, 205) | [https://csrc.nist.gov/projects/post-quantum-cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography) |
| EXT-12 | NIST IR 8547 | [https://nvlpubs.nist.gov/nistpubs/ir/2026/NIST.IR.8547.pdf](https://nvlpubs.nist.gov/nistpubs/ir/2026/NIST.IR.8547.pdf) |
| EXT-13 | Huawei Cloud KooCLI | [https://support.huaweicloud.com/intl/en-us/productdesc-hcli/hcli_01.html](https://support.huaweicloud.com/intl/en-us/productdesc-hcli/hcli_01.html) |
| EXT-05 | CSA Cloud Controls Matrix (CCM) | [https://cloudsecurityalliance.org/research/cloud-controls-matrix](https://cloudsecurityalliance.org/research/cloud-controls-matrix) |
| EXT-06 | Google cryptography migration timeline | [https://blog.google/innovation-and-ai/technology/safety-security/cryptography-migration-timeline/](https://blog.google/innovation-and-ai/technology/safety-security/cryptography-migration-timeline/) |
| EXT-07 | Google Cloud PQC overview | [https://cloud.google.com/security/resources/post-quantum-cryptography](https://cloud.google.com/security/resources/post-quantum-cryptography) |
| EXT-08 | Cloudflare PQC roadmap | [https://blog.cloudflare.com/post-quantum-roadmap/](https://blog.cloudflare.com/post-quantum-roadmap/) |
| EXT-09 | Cloudflare PQC docs | [https://developers.cloudflare.com/ssl/post-quantum-cryptography/](https://developers.cloudflare.com/ssl/post-quantum-cryptography/) |
| EXT-10 | Microsoft quantum-safe progress update | [https://www.microsoft.com/en-us/security/blog/2025/08/20/quantum-safe-security-progress-towards-next-generation-cryptography/](https://www.microsoft.com/en-us/security/blog/2025/08/20/quantum-safe-security-progress-towards-next-generation-cryptography/) |

