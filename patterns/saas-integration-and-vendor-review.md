# Pattern: SaaS Integration & Third-Party Vendor Security Review

## Context
Third-party SaaS applications accelerate product delivery but introduce significant supply-chain, regulatory (e.g., FERPA, GLBA, GDPR), and data-exfiltration risks. Organizations require a standardized, defensible review process to evaluate, integrate, and decommission vendor software without impeding business agility.

---

## Decision & Standard

1. **Risk Tiering Matrix:**
   - **Tier 1 (Critical):** SaaS processes Customer PII/NPI, financial records, student records, credentials, or connects directly to production data stores. Requires comprehensive review, SOC 2 Type II / ISO 27001 audit report, annual re-assessment, and executive sign-off.
   - **Tier 2 (Moderate):** Internal business data, employee PII, or non-production code. Requires questionnaire review, standard SOC 2 Type II, and automated identity provisioning.
   - **Tier 3 (Low):** Publicly available data, ephemeral collaboration tools with no sensitive integrations. Lightweight questionnaire review.

2. **Security Intake Questionnaire (Core Gates):**
   - Independent verification: Unqualified SOC 2 Type II report (within past 12 months) covering Security, Availability, and Confidentiality.
   - Encryption: TLS 1.3 (minimum TLS 1.2) in transit; AES-256 at rest with customer-managed key (CMEK) option for Tier 1.
   - Sub-processor notification SLA: Minimum 30-day advance notice before third-party data handoffs.
   - Vulnerability disclosure & incident response: Written contractual commitment to report breaches within **48 hours** (24 hours for Tier 1).

3. **Identity & Access Management Integration:**
   - Mandatory SAML 2.0 or OIDC enterprise Single Sign-On (SSO) enforced with enterprise Multi-Factor Authentication (MFA).
   - Automated user provisioning and deprovisioning via **SCIM 2.0**. No unmanaged local accounts allowed.
   - Role-Based Access Control (RBAC) mapping directory groups directly to vendor roles.

4. **Data-Flow Architecture & Exit Planning:**
   - Architecture review requires an approved Data-Flow Diagram (DFD) prior to network whitelisting or credential issuance.
   - **Contractual Exit Plan:** Vendor must provide verifiable bulk data export in standard formats (e.g., JSON, Parquet, CSV) and deliver a formal **Certificate of Data Destruction** within 30 days of contract termination.

---

## Controls Mapping

| Framework | Control Reference | Alignment Description |
| :--- | :--- | :--- |
| **NIST SP 800-53 r5** | **SA-9, SR-3, SR-5, PS-7** | External system services, supply chain risk management, acquisition controls, third-party personnel security. |
| **ISO/IEC 27001:2022** | **A.5.19, A.5.20, A.5.21, A.5.22** | Information security in supplier relationships, addressing security within supplier agreements, ICT supply chain. |
| **SOC 2 (Trust Services Criteria)** | **CC9.2, CC6.2, CC6.3** | Assessment and management of vendor risks, external system boundary access controls. |

---

## Anti-Patterns

- **Marketing-Driven Assessments:** Accepting self-attested vendor marketing sheets or security whitepapers instead of third-party audit reports.
- **SSO Exemptions ("Just a Few Admin Accounts"):** Permitting vendor-native username/password logins for administrative roles, bypassing corporate MFA and offboarding controls.
- **Undefined Offboarding Procedures:** Terminating contracts without confirming complete cryptographic erasure of tenant data and backups.
- **Unrestricted API Integration:** Granting broad OAuth tenant-wide permissions (e.g., Google Workspace or Microsoft 365 tenant scopes) without granular scoping.

---

## Checklist

- [ ] Vendor completed intake security questionnaire and provided current SOC 2 Type II report.
- [ ] Tier 1/2 data flows are mapped on a DFD showing ingress, egress, persistence, and encryption boundaries.
- [ ] Enterprise SSO (SAML 2.0 / OIDC) and SCIM provisioning are configured and validated.
- [ ] Vendor does not retain static API keys with permanent administrative rights.
- [ ] Contract includes mandatory 48-hour incident notification SLA and data deletion SLA upon termination.
- [ ] Security architecture review logged as an auditable compliance record.
