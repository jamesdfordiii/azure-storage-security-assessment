# Azure Storage Security Assessment

## Project Overview

This project documents a hands-on security assessment and hardening exercise performed against a Microsoft Azure Storage account. The objective was to review the resource from a security and governance perspective, identify weaknesses, apply selected remediations, and preserve evidence of the resulting security posture.

The assessment focuses on practical cloud-security controls relevant to identity and access management, data protection, encryption, monitoring, threat protection, and resource governance.

## Environment

- **Cloud platform:** Microsoft Azure
- **Resource:** Azure Storage Account (StorageV2 / General Purpose v2)
- **Resource group:** `RG-SecurityLab`
- **Region:** Southeast Asia
- **Assessment method:** Azure Portal configuration review and manual remediation

## Assessment Areas

| Control Area | Observation | Security Consideration / Action |
| --- | --- | --- |
| Shared Key authorization | Shared Key access was reviewed and disabled | Reduces reliance on storage account keys and supports identity-based authorization |
| Microsoft Entra authorization | Default Entra authorization was reviewed and enabled | Promotes identity-based access instead of shared credentials |
| Blob versioning | Initially disabled | Enabled to improve recovery from unintended modification or deletion |
| Blob/container soft delete | Enabled with 7-day retention | Provides recovery capability for deleted data |
| Encryption at rest | Microsoft-managed keys enabled | Azure Storage data remains encrypted at rest |
| Infrastructure encryption | Disabled | Documented as a defense-in-depth consideration |
| RBAC / IAM | Subscription-level Owner assignment observed | Identified as a least-privilege governance consideration |
| Defender for Storage | Not enabled | Documented as a threat-detection opportunity with potential cost implications |
| Diagnostic logging | Not configured | Identified as a monitoring and audit visibility gap |
| Shared Access Signatures | Shared Key-based SAS unavailable after Shared Key authorization was disabled | Reduces exposure to key-based delegated access |
| Access keys | Keys remain present but Shared Key authorization is disabled | Keys should remain protected and unnecessary key-based authentication avoided |
| Resource lock | Initially absent | Delete lock added to reduce accidental or unauthorized deletion risk |
| Azure Advisor | No active recommendations displayed at verification time | Used as supplementary post-remediation validation |

## Key Findings

### 1. Shared Key Authorization

The storage account initially permitted storage account key access. Shared Key authorization was disabled and the configuration was verified. This moves the resource toward Microsoft Entra ID and Azure RBAC-based authorization rather than long-lived shared credentials.

### 2. Microsoft Entra Authorization

The portal's default Microsoft Entra authorization setting was enabled as part of the hardening process. Identity-based authorization provides stronger accountability and supports role-based access control.

### 3. Data Protection

Blob and container soft delete were enabled with seven-day retention. Blob versioning was initially disabled and was subsequently enabled. Together, these controls improve resilience against accidental deletion and unintended modification.

### 4. Encryption

The storage account uses Microsoft-managed keys for encryption at rest. Infrastructure encryption was observed as disabled. This was recorded as an additional defense-in-depth opportunity rather than changed solely for the lab.

### 5. Identity and Least Privilege

The IAM review showed an Owner role inherited at subscription scope. Owner provides broad administrative capability, so privileged role assignments should be periodically reviewed and reduced where operationally possible.

### 6. Threat Protection

Microsoft Defender for Storage was not enabled. The assessment recorded the available threat-detection and malware-scanning capabilities while recognizing that enabling paid cloud-security services can introduce ongoing cost.

### 7. Monitoring and Logging

Diagnostic settings were not configured for the storage resource. Without diagnostic export, security teams have less centralized visibility into storage operations. In a production environment, relevant logs should be routed to an appropriate monitoring destination such as Log Analytics, Event Hub, or another approved logging platform.

### 8. Resource Protection

No resource lock existed at the beginning of the assessment. A **Delete** lock named `Prevent-Storage-Deletion` was added to the storage account to help prevent accidental or unauthorized deletion of the resource.

## Risk Summary

The assessment demonstrated that a cloud resource can have encryption and basic recovery controls enabled while still presenting security and governance gaps in authentication, privileged access, logging, threat detection, and resource protection.

The highest-value hardening actions in this exercise were reducing Shared Key usage, preferring Entra-based authorization, enabling blob versioning, and adding deletion protection. Monitoring, least-privilege review, and Defender for Storage remain important considerations for a production deployment.

## Evidence

Screenshots collected during the assessment document the baseline posture, findings, remediation steps, and verification states. They are stored in the `screenshots/` directory.

Evidence includes:

- Initial and baseline storage security posture
- Shared Key authorization finding and remediation
- Microsoft Entra authorization configuration
- Blob versioning finding and remediation
- Infrastructure encryption status
- Subscription Owner privilege observation
- Defender for Storage status
- Shared Key/SAS verification
- Access-key protection state
- Diagnostic logging findings
- Missing resource-lock finding
- Delete-lock configuration and verification
- Azure Advisor post-remediation view

## Skills Demonstrated

- Azure Storage security assessment
- Microsoft Entra ID authorization concepts
- Azure RBAC and least-privilege analysis
- Cloud data-protection controls
- Encryption configuration review
- Azure security monitoring assessment
- Microsoft Defender for Cloud awareness
- Azure resource locks and governance
- Security finding documentation
- Risk-based remediation and verification

## Security Notes

This repository contains lab evidence only. Credentials, storage account keys, connection strings, SAS tokens, and other secrets should never be committed to a public repository. Screenshots should be reviewed for sensitive information before publication.

## Project Outcome

The lab produced a documented before-and-after Azure Storage security assessment rather than simply demonstrating resource deployment. It shows the ability to inspect cloud controls, identify risk, distinguish between findings and accepted/unremediated considerations, apply targeted hardening, and verify changes with supporting evidence.
