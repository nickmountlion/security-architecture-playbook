# Pattern: Secure Software Development Life Cycle (SSDLC) & CI/CD Security Gates

## Context
Accelerated deployment cadences and the widespread adoption of AI coding assistants require automated, non-blocking guardrails embedded directly into the developer workflow. Security cannot be a manual end-of-quarter milestone; it must function as continuous policy-as-code within CI/CD.

---

## Decision & Standard

1. **Threat Modeling Gates:**
   - Mandatory lightweight threat model (STRIDE / PASTA) triggered during the design phase for: new public endpoints, authentication/authorization changes, sensitive data schema changes, or external third-party integrations.
   - Threat models are stored as versioned Markdown files in the repository alongside code.

2. **Automated CI/CD Scanning Gates:**
   - **Pre-Commit / Pre-Push:** Local hook scanning for high-entropy secrets and API tokens.
   - **Secrets Detection:** Pipeline failure if unencrypted secrets/keys are detected. Zero-tolerance policy; exemptions require time-bound cryptographic suppression hashes.
   - **Software Composition Analysis (SCA):** Automated blocking on open-source dependencies with known vulnerabilities (CVSS >= 7.0 / High or Critical) that possess active exploit vectors.
   - **Static Application Security Testing (SAST):** Automated incremental scanning on pull requests. Blocking on confirmed injection flaws (SQLi, SSRF, XSS) and insecure cryptographic primitives.
   - **Container / Artifact Security:** Base images scanned and signed (e.g., Cosign/Sigstore) with mandatory Software Bill of Materials (SBOM) generation (CycloneDX or SPDX).

3. **Peer Review & Branch Protection:**
   - Main branches protected: Direct pushes disabled, minimum 2 peer approvals required.
   - Mandatory code ownership (`CODEOWNERS`) routing for security-critical paths (e.g., `/auth`, `/crypto`, `/iam`, `.github/workflows`).
   - All commits must be cryptographically signed (GPG or SSH keys).

4. **AI-Assisted Code Controls:**
   - AI-generated code is subject to identical scrutiny and gates as human-written code.
   - Automated linters enforce license compatibility scans to prevent accidental ingestion of viral open-source licenses (GPL/AGPL) into proprietary codebases.
   - Hallucinated package detection: Package manager lockfiles must be verified against official package registries (e.g., npm, PyPI) to prevent dependency-confusion and package-hallucination squatting attacks.

---

## Controls Mapping

| Framework | Control Reference | Alignment Description |
| :--- | :--- | :--- |
| **NIST SP 800-53 r5** | **SA-8, SA-11, SI-2, SI-10** | Security engineering principles, developer testing, flaw remediation, information input validation. |
| **ISO/IEC 27001:2022** | **A.8.25, A.8.28, A.8.29, A.8.30** | Secure development lifecycle, secure coding, security testing in development, outsourced development. |
| **SOC 2 (Trust Services Criteria)** | **CC7.1, CC8.1** | Vulnerability management infrastructure, change management authorization, testing, and deployment. |

---

## Anti-Patterns

- **Informational-Only Security Alerts:** Configuring SAST and SCA tools to run asynchronously as non-blocking warnings that developers inevitably ignore.
- **Bypassing Reviews for AI Code:** Treating AI-generated pull requests as pre-verified or self-auditing.
- **Ignoring Transitive Dependencies:** Scanning direct dependencies while ignoring vulnerable sub-dependencies in lockfiles.
- **Late-Stage Penetration Testing as Sole Gate:** Relying solely on an annual penetration test right before product release.

---

## Checklist

- [ ] Branch protection rules require signed commits, CI green status, and CODEOWNERS approvals.
- [ ] Pre-commit hooks and CI pipelines block hardcoded secrets.
- [ ] SCA scans block dependencies with CVSS >= 7.0 with verified exploitable paths.
- [ ] SAST runs incrementally on PRs with zero critical vulnerabilities allowed into main.
- [ ] Container images are built from verified minimal base images and cryptographically signed.
- [ ] SBOM is automatically generated during release builds.
