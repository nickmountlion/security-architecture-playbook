# Security Architecture Playbook

**Author:** Facundo "Nick" Montefalcone  
**Role:** Cloud Security Architect | 15+ Years Experience (Fintech, K-12 EdTech, SaaS)  
**License:** [MIT](./LICENSE)

A practitioner-focused, vendor-neutral repository of cloud security architecture patterns, evaluation frameworks, and governance processes designed for high-velocity engineering environments.

---

## Operating Philosophy: How I Work

1. **Evidence First**  
   Security claims require empirical verification. Architectures, risk ratings, and compliance assertions must be backed by reproducible test data, automated gate results, or observable telemetry—never vendor marketing or unverified assertions.

2. **The Written Record Is a Control**  
   If an architectural trade-off or boundary is not written down, it does not exist. Architecture Decision Records (ADRs), threat models, and explicit data-flow boundaries serve as defensive engineering controls, compliance artifacts, and cross-team contracts.

3. **Kaizen Continuous Improvement Loop**  
   Security posture is dynamic. Rather than relying on disruptive annual audits, implement a tight loop: **Plan** (threat hypothesis) -> **Run** (automated gate/canary) -> **Measure** (telemetry) -> **Adjust** (standardize or discard).

---

## Playbook Index

| File | Focus Area | Key Themes | Target Compliance / Standards |
| :--- | :--- | :--- | :--- |
| [API Authentication](./patterns/api-authentication.md) | Machine-to-Machine Auth | OAuth2/OIDC, mTLS, Short Lifetimes, Scopes, Decision Matrix | NIST 800-53, ISO 27001, SOC 2 |
| [SaaS & Vendor Review](./patterns/saas-integration-and-vendor-review.md) | Third-Party Risk | Intake Questionnaire, DFDs, SSO/SCIM, Risk Tiering, Exit Plans | NIST 800-53, ISO 27001, SOC 2 |
| [AI Agent Security](./patterns/ai-agent-security.md) | GenAI & Autonomous Workflows | App Registrations, Least-Privilege Tools, Secrets, HITL, Trust Boundaries | NIST 800-53, ISO 27001, SOC 2, OWASP GenAI |
| [PAM Bakeoff Framework](./patterns/pam-bakeoff.md) | Identity & Access | Evaluation Criteria, Weighted Scorecard, POC Plan, Exit Criteria | NIST 800-53, ISO 27001, SOC 2 |
| [Secure SDLC](./patterns/secure-sdlc.md) | CI/CD & Code Security | Threat Modeling, Automated Gates (SAST/SCA/Secrets), AI Code Governance | NIST 800-53, ISO 27001, SOC 2 |
| [ADR Template & Example](./patterns/architecture-decision-record-template.md) | Architectural Decisions | Standard Template + Production-Grade Sample (JIT Database Access) | NIST 800-53, ISO 27001, SOC 2 |
| [Kaizen Security Loop](./process/kaizen-loop.md) | Process Hardening | Continuous Improvement Applied to AI & Cloud Pipelines, Core Metrics | Deming Cycle, DevSecOps Telemetry |

---

## How to Use This Playbook

- **Architecture Design Reviews:** Use the ADR template and relevant pattern checklist before writing code or signing contracts.
- **Pipeline Implementation:** Enforce the technical constraints defined in `secure-sdlc.md` and `api-authentication.md` directly inside CI/CD linting and build stages.
- **Vendor & SaaS Ingestion:** Require prospective vendors to clear the checklist and data-flow standards in `saas-integration-and-vendor-review.md`.
- **AI Agent Deployment:** Run any autonomous or agentic LLM workflow through the trust boundaries and HITL gates in `ai-agent-security.md`.
- **Tooling Selection:** Use `pam-bakeoff.md` as an objective, weighted methodology when assessing enterprise infrastructure software.

---

## Contribution & Standards

Patterns in this repository are strictly vendor-neutral, emphasizing standard protocols (OIDC, SAML, mTLS, SPIFFE/SPIRE, OpenTelemetry) over proprietary lock-in. Updates must adhere to the Kaizen model: measurable impact, documented justification, and mapped controls.
