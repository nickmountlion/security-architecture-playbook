# Pattern: Privileged Access Management (PAM) Bakeoff & Evaluation

## Context
Securing administrative access to cloud control planes, Kubernetes clusters, databases, and production hosts requires a modern Privileged Access Management (PAM) solution. Legacy static-vault solutions introduce credential sprawl, developer friction, and audit gaps. This pattern provides an objective, evidence-based framework for evaluating modern PAM platforms.

---

## Decision & Standard

1. **Evaluation Criteria & Architectural Priorities:**
   - **Zero Standing Privileges (ZSP):** Preference for ephemeral Just-In-Time (JIT) credential issuance and short-lived certificate minting over static password rotation vaults.
   - **Developer Experience & Native Tooling:** Seamless integration with standard CLI tools (`kubectl`, `psql`, `ssh`, cloud CLIs) without requiring heavy proprietary desktop clients.
   - **Session Recording & Auditability:** Keystroke-level logging, tamper-resistant audit logs, and integration with enterprise SIEM/data lakes.
   - **Identity Provider (IdP) Federation:** Native integration with Okta/Entra ID for conditional access, device posture checks, and MFA enforcement.

### Weighted Scorecard Matrix

| Evaluation Dimension | Weight | Critical Capabilities Required | Target Score (1–5) |
| :--- | :--- | :--- | :--- |
| **Just-in-Time Access (JIT)** | 25% | Ephemeral access requests, automated Slack/Teams approvals, automatic TTL revocation | >= 4.5 |
| **Cloud & Multi-Platform Support** | 20% | Native AWS/GCP/Azure IAM role brokering, K8s RBAC, multi-engine database proxies | >= 4.0 |
| **Session Control & Audit** | 20% | Full CLI session recording, structured audit trail export, real-time termination | >= 4.0 |
| **Architecture & Deployment** | 15% | Agentless or lightweight eBPF/proxy, high availability, zero vendor credential access | >= 4.0 |
| **Developer Ergonomics** | 10% | Minimal latency (<100ms overhead), native CLI workflows, transparent SSO logins | >= 4.5 |
| **Enterprise Governance & API** | 10% | Comprehensive Terraform/OpenTofu provider, SCIM support, SIEM streaming | >= 4.0 |

2. **Proof of Concept (POC) Test Plan:**
   - **Test 1 (JIT Elevation):** Request ephemeral access to production database, trigger dual-approval workflow, verify automatic access revocation after 60 minutes.
   - **Test 2 (Credential Injection Defense):** Attempt to extract raw underlying credentials from client memory/terminal during an active session; confirm credentials never touch client machines.
   - **Test 3 (Session Recording & Revocation):** Initiate SSH/kubectl session, execute test commands, trigger immediate administrator kill-switch; verify session drops instantaneously and logs are captured.
   - **Test 4 (High-Availability & Latency):** Run benchmark queries through database proxy under simulated network degradation; measure latency impact against baseline.
   - **Test 5 (Break-Glass Recovery):** Disconnect IdP connection; execute documented offline break-glass procedure to ensure emergency recovery capability.

3. **Exit Criteria (Go/No-Go Gates):**
   - Candidate solution must achieve a weighted score >= 4.0 with zero scores below 3.0 in any security-critical capability.
   - Vendor must successfully pass third-party security assessment and SOC 2 Type II audit review.
   - Clear contract terms for operational SLAs (>= 99.99% availability for access proxy plane).

---

## Controls Mapping

| Framework | Control Reference | Alignment Description |
| :--- | :--- | :--- |
| **NIST SP 800-53 r5** | **AC-2(1), AC-6, IA-2, AU-3, AU-6** | Automated privilege management, least privilege, MFA verification, audit log content and review. |
| **ISO/IEC 27001:2022** | **A.8.2, A.8.5, A.8.18, A.8.24** | Privileged access rights, secure authentication, use of privileged utility programs, cryptography. |
| **SOC 2 (Trust Services Criteria)** | **CC6.1, CC6.2, CC6.3, CC7.2** | Logical access security, user registration/de-registration, least-privilege, security event monitoring. |

---

## Anti-Patterns

- **Selecting Legacy Vaults for Cloud Workloads:** Deploying heavy password-checkout vaults designed for static bare-metal servers into dynamic cloud-native environments.
- **Ignoring Developer UX:** Mandating cumbersome VPN-dependent web consoles that incentivize engineers to establish unmonitored backdoors and static bypass credentials.
- **Vendor-Hosted Master Secrets:** Allowing third-party SaaS vendors to hold customer master encryption keys without zero-knowledge architecture.

---

## Checklist

- [ ] Weighted scorecard agreed upon by Security, DevOps, and Platform teams prior to vendor demos.
- [ ] POC test plan executed in a staging environment mirroring production topologies.
- [ ] Emergency break-glass procedures tested and validated offline.
- [ ] Zero static root credentials remain on developer workstations post-implementation.
- [ ] Audit logs flow directly into enterprise SIEM with tamper-detection enabled.
