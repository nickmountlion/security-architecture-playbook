# Pattern: Security Architecture Decision Record (ADR) Standard & Template

## Context
Architectural decisions with security implications are frequently lost in meeting notes, chat threads, or ephemeral tickets. Without an auditable decision record, engineering teams inadvertently reintroduce vulnerabilities or fail compliance audits when rationale cannot be defended. An ADR operates as both an engineering contract and a compliance control.

---

## Decision & Standard: The Security ADR Framework

Every material architectural change affecting authentication, authorization, cryptographic posture, network boundaries, or data privacy must produce a committed Markdown ADR in `docs/adr/`.

### ADR Standard Template (Copy-Paste)

```markdown
# ADR-[NUMBER]: [Title: Action-Oriented Decision]

- **Status:** [Proposed | Accepted | Superseded | Deprecated]
- **Date:** [YYYY-MM-DD]
- **Authors:** [Name / Role]
- **Reviewers:** [Security Architect, Lead Engineer]

## Context & Problem Statement
[Describe the technical context, threat model, regulatory drivers, or operational friction requiring a decision.]

## Decision
[State the chosen solution clearly and unambiguously. Detail protocol, identity, cryptographic, and network boundaries.]

## Consequences
- **Positive:** [Security improvements, operational gains, developer velocity impacts.]
- **Negative / Trade-offs:** [Operational overhead, license costs, migration effort.]

## Controls & Compliance Mapping
- NIST 800-53: [e.g., AC-2, AC-6] | ISO 27001: [e.g., A.8.2] | SOC 2: [e.g., CC6.1]
```

---

## Filled Production Example: ADR-0042

### ADR-0042: Ephemeral Just-in-Time (JIT) Cloud Database Access
- **Status:** Accepted
- **Date:** 2026-03-15
- **Authors:** Security Architecture Team
- **Reviewers:** Data Platform Lead, Head of Infrastructure

#### Context & Problem Statement
Engineers currently hold persistent personal database credentials for production RDS instances. This creates standing privilege risks, complicates offboarding, and fails SOC 2 CC6.1 audits. We need a zero-standing-privilege model that does not impede emergency debugging.

#### Decision
Replace all static production database credentials with an ephemeral Just-in-Time (JIT) access proxy. Engineers will authenticate via enterprise OIDC with WebAuthn/FIDO2 MFA to request time-bound (max 60 minutes) credentials. All read/write queries will pass through an audited proxy logging structured SQL statements to the central SIEM.

#### Consequences
- **Positive:** Eliminates static database passwords on developer workstations; automates audit logging for compliance; enforces MFA at database connection time.
- **Negative:** Introduces proxy hop latency (~4ms); engineers must trigger CLI elevation before connecting.

#### Controls & Compliance Mapping
- **NIST SP 800-53 r5:** AC-2(1), AC-6, AU-2, IA-2
- **ISO/IEC 27001:2022:** A.8.2, A.8.5, A.8.18
- **SOC 2 Type II:** CC6.1, CC6.3, CC7.2

---

## Anti-Patterns

- **Post-Hoc ADR Generation:** Writing ADRs months after implementation solely to satisfy auditor evidence collection.
- **Vendor Propaganda as Rationale:** Citing vendor marketing claims rather than empirical threat models and measurable architectural trade-offs.
- **Unaddressed Negative Consequences:** Leaving the trade-offs section blank to make a decision appear risk-free.

---

## Checklist

- [ ] ADR stored under version control in a standardized directory (`docs/adr/`).
- [ ] Context clearly defines the threat model or compliance requirement.
- [ ] Decision specifies exact protocols, identity boundaries, and operational constraints.
- [ ] Consequences document both benefits and trade-offs.
- [ ] Relevant security framework controls (NIST, ISO, SOC 2) are mapped explicitly.
- [ ] Required cross-functional stakeholders signed off before merging.
