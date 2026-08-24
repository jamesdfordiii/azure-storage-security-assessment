# Azure Storage Security Assessment

## Project Overview

This project documents a hands-on security assessment and hardening exercise performed against a Microsoft Azure Storage account. The objective was to review the resource from a security and governance perspective, identify weaknesses, apply selected remediations, and validate the resulting security posture with evidence.

The assessment focuses on practical cloud-security controls across network exposure, identity and access management, data protection, encryption, monitoring, threat protection, and resource governance.

---

## Environment

| Component | Details |
| --- | --- |
| Cloud platform | Microsoft Azure |
| Resource | Azure Storage Account — StorageV2 / General Purpose v2 |
| Resource group | `RG-SecurityLab` |
| Region | Southeast Asia |
| Assessment method | Azure Portal configuration review, risk analysis, remediation, and verification |

![Initial Storage Security Posture](screenshots/01-Initial-Storage-Security-Posture.png)

---

## Executive Findings Summary

| Finding | Risk | Status |
| --- | --- | --- |
| Public network access allowed from all networks | Medium | Remediated |
| Shared Key authorization enabled | Medium | Remediated |
| Microsoft Entra authorization not set as portal default | Low–Medium | Remediated |
| Blob versioning disabled | Medium | Remediated |
| Subscription-level Owner privilege inherited | High | Documented / not changed in lab |
| Infrastructure encryption disabled | Low | Documented |
| Microsoft Defender for Storage disabled | Medium | Recommended / not enabled due to cost |
| Diagnostic logging not configured | Medium | Recommended / Log Analytics not provisioned |
| Resource lock absent | Medium | Remediated |

---

# Assessment & Hardening

## 1. Public Network Exposure

### Finding

The storage account initially allowed public network access from **all networks**. Authentication still protected the storage service, but the endpoint was broadly reachable and exposed a larger network attack surface than necessary.

![Public Network Exposure](screenshots/02-Finding-Public-Network-Exposure.png)

### Remediation

Public access was restricted to **selected networks**, with an approved client IPv4 address added to the storage firewall rules.

![Restricted Network Access](screenshots/03-Remediation-Restricted-Network-Access.png)

**Security result:** Reduced public exposure while preserving required administrative access.

---

## 2. Shared Key Authorization

### Finding

Storage account key access was enabled, allowing Shared Key-based authorization. Shared credentials provide weaker individual accountability than Microsoft Entra ID and Azure RBAC.

![Shared Key Authorization Enabled](screenshots/04-Finding-Shared-Key-Authorization.png)

### Remediation

Storage account key access was disabled.

![Shared Key Disabled](screenshots/05-Remediation-Shared-Key-Disabled.png)

Additional validation on the Shared Access Signature page confirmed that requests authorized with Shared Key, including account SAS, would be denied.

![Shared Key SAS Verification](screenshots/12-Verification-Shared-Key-SAS-Disabled.png)

The Access Keys page also confirmed that the keys remained protected and masked even though Shared Key authorization had been disabled.

![Access Keys Protected](screenshots/13-Verification-Access-Keys-Protected.png)

**Security result:** Reduced reliance on shared credentials and strengthened the identity-based access model.

---

## 3. Microsoft Entra Authorization

The Azure portal was not initially configured to default to Microsoft Entra authorization for storage data access.

The setting was changed to prefer Microsoft Entra authorization and then verified after save.

![Entra Authorization Configuration](screenshots/06-Remediation-Entra-Authorization-PreSave.png)

![Entra Authorization Verified](screenshots/07-Remediation-Entra-Authorization-Verified.png)

**Security result:** Portal access is directed toward identity-based authorization and Azure RBAC rather than shared-account credentials.

---

## 4. Blob Versioning & Recovery

### Finding

Blob and container soft delete were already enabled with seven-day retention, but **blob versioning was disabled**.

![Blob Versioning Disabled](screenshots/08-Finding-Blob-Versioning-Disabled.png)

### Remediation

Blob versioning was enabled and configured to retain versions.

![Blob Versioning Configuration](screenshots/09-Remediation-Blob-Versioning-PreSave.png)

![Blob Versioning Verified](screenshots/10-Remediation-Blob-Versioning-Verified.png)

**Security result:** Improved resilience against unintended overwrite, modification, and deletion scenarios.

---

## 5. Identity & Least Privilege

The IAM review showed an **Owner** role inherited at **subscription scope**.

![Subscription Owner Privilege](screenshots/10-Finding-Subscription-Owner-Privilege.png)

Owner provides broad resource-management and access-assignment capability. In a production environment, permanent subscription-level Owner access should be minimized and replaced with narrower RBAC roles or time-bound privileged access where possible.

This assignment was **documented rather than removed** because it was the administrative account used to perform the lab.

---

## 6. Encryption Review

Azure Storage encryption at rest was enabled using Microsoft-managed keys. **Infrastructure encryption** was disabled.

![Infrastructure Encryption Disabled](screenshots/10-Finding-Infrastructure-Encryption-Disabled.png)

This was treated as a **low-risk defense-in-depth observation**, not evidence that the storage account was unencrypted. Infrastructure encryption can provide an additional encryption layer where organizational requirements justify it.

---

## 7. Microsoft Defender for Storage

Microsoft Defender for Storage was not enabled.

![Defender for Storage Disabled](screenshots/11-Finding-Defender-for-Storage-Disabled.png)

Defender for Storage can provide additional threat-detection and malware-scanning capabilities. The control was documented but not enabled because the Azure portal indicated an ongoing paid cost for the service.

**Recommendation:** Enable Defender for Storage where the workload's threat model and business requirements justify the additional monitoring cost.

---

## 8. Diagnostic Logging

### Finding

Azure diagnostic settings were disabled for the storage account services, creating a monitoring and investigation visibility gap.

![Diagnostic Logging Disabled](screenshots/14-Finding-Diagnostic-Logging-Disabled.png)

The Blob service also showed no diagnostic settings configured.

![Blob Diagnostic Logging Disabled](screenshots/13-Finding-Blob-Diagnostic-Logging-Disabled.png)

The remediation path was investigated by preparing Storage Read, Storage Write, Storage Delete, and Transaction telemetry for export. A Log Analytics workspace was not available in the subscription, so the change was not implemented solely for this lab.

**Recommendation:** Route appropriate Azure Storage diagnostics to Log Analytics, Event Hub, or another approved centralized monitoring/SIEM platform in production.

---

## 9. Resource Deletion Protection

### Finding

The storage account initially had **no resource locks**.

![No Resource Lock](screenshots/14-Finding-No-Resource-Lock.png)

### Remediation

A Delete lock named `Prevent-Storage-Deletion` was configured.

![Delete Lock Configured](screenshots/15-Remediation-Delete-Lock-Configured.png)

The applied lock was then verified on the resource.

![Delete Lock Enabled](screenshots/16-Post-Remediation-Delete-Lock-Enabled.png)

**Security result:** Added an additional safeguard against accidental or unauthorized deletion of the storage account resource.

---

## 10. Azure Advisor Verification

Azure Advisor displayed no active recommendations for the storage account at the time of verification. Azure noted that recommendation status can take time to refresh, so Advisor was treated as supplementary evidence rather than the primary validation source.

![Azure Advisor](screenshots/17-Advisor-No-Active-Recommendations.png)

---

# Controls That Passed Review

Several controls were already configured securely and were not changed simply to create additional findings:

- Secure transfer required
- Minimum TLS version 1.2
- Blob anonymous access disabled
- Blob soft delete enabled
- Container soft delete enabled
- Encryption at rest enabled with Microsoft-managed keys
- Access keys remained masked and protected

This distinction was important to the assessment: **not every setting should be classified as a vulnerability, and not every available security feature is mandatory for every workload.**

---

# Remediation Summary

```text
Baseline Configuration Review
        ↓
Identify Security & Governance Gaps
        ↓
Assess Risk and Operational Context
        ↓
Apply Targeted Remediations
        ↓
Verify Resulting Configuration
        ↓
Document Unimplemented Recommendations
```

Implemented changes included:

- Restricted storage network access to selected networks/IPs
- Disabled Shared Key authorization
- Enabled Microsoft Entra authorization as the portal default
- Enabled blob versioning
- Added a Delete resource lock

Documented but not implemented:

- Reduction of inherited subscription-level Owner access
- Infrastructure encryption
- Microsoft Defender for Storage
- Centralized diagnostic export to Log Analytics

---

# Skills Demonstrated

- Azure Storage security assessment
- Azure networking and storage firewall review
- Microsoft Entra ID authorization concepts
- Azure RBAC and least-privilege analysis
- Shared Key and SAS security analysis
- Cloud data-protection controls
- Blob versioning and soft-delete analysis
- Encryption configuration review
- Azure diagnostic settings assessment
- Microsoft Defender for Cloud awareness
- Azure resource locks and governance
- Risk classification and remediation prioritization
- Security control validation
- Cloud security documentation

---

## Security Notes

This repository contains lab evidence only. Credentials, storage account keys, connection strings, SAS tokens, and other secrets were not intentionally exposed or committed. Screenshots were captured with secrets masked.

## Project Outcome

This project demonstrates a complete Azure security-assessment workflow rather than simply deploying a cloud resource. The storage account was reviewed across identity, network, data protection, encryption, monitoring, threat detection, and governance controls. Findings were evaluated in context, selected controls were remediated and verified, and recommendations that were inappropriate or unnecessary to implement in the lab were documented rather than forced.
