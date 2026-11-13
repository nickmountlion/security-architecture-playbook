# Pattern: Machine-to-Machine API Authentication & Authorization

## Context
Modern cloud architectures rely heavily on service-to-service (east-west) and partner (north-south) API communications across microservices, serverless functions, and third-party platforms. Relying on static API keys or long-lived shared tokens introduces severe credential leakage, credential replay, and attribution blind spots.

---

## Decision & Standard

1. **Authentication Protocol:**
   - **External / Edge Ingress:** OAuth 2.0 / OIDC Client Credentials Grant (`grant_type=client_credentials`) using asymmetric JWTs (RS256 or EdDSA) validated via public JWKS endpoints.
   - **Internal High-Trust (East-West):** Mutual TLS (mTLS) with short-lived X.509 certificates issued by an internal PKI or service mesh (e.g., SPIFFE/SPIRE), combined with bearer identity tokens for request-level authorization.

2. **Token Lifetimes & Scopes:**
   - **Access Tokens:** Strict maximum lifetime of **15 minutes** (900 seconds). Non-refreshable in pure machine-to-machine flows (re-authenticate using client credentials).
   - **Scope Design:** Hierarchical, least-privilege syntax: `<domain>:<resource>:<action>` (e.g., `billing:invoice:read`, `identity:tenant:write`). Wildcards (`*`) are prohibited.
   - **Audience Validation:** Validating the `aud` (Audience) claim is mandatory at every receiving endpoint to prevent cross-service token replay.

3. **Secret Storage & Key Rotation:**
   - Cryptographic keys rotated automatically every **90 days** (maximum).
   - Verification systems must maintain a dual-key grace window (current key + previous key) for seamless rollover.
   - Secrets and private keys must reside in hardware security modules (HSM) or cloud KMS vaults; never injected as plain environment variables in source control.

### Decision Matrix

| Integration Scenario | Primary Mechanism | Secondary Defense | Token TTL | Complexity |
| :--- | :--- | :--- | :--- | :--- |
| Internal Microservice to Microservice | mTLS (SPIFFE/X.509) | Service Identity Header | Session / ephemeral | Medium |
| External Partner Ingress (High Criticality / Fintech) | mTLS + OAuth 2.0 Client Creds | IP Whitelisting / Mutual CA | <= 15 min | High |
| External SaaS / General API Ingress | OAuth 2.0 Client Credentials (JWT) | Asymmetric Signature (JWKS) | <= 15 min | Low |
| Legacy System / Gateway Proxy | Asymmetric Client Assertion (RFC 7523) | Gateway Token Exchange | <= 10 min | Medium |

---

## Controls Mapping

| Framework | Control Reference | Alignment Description |
| :--- | :--- | :--- |
| **NIST SP 800-53 r5** | **AC-3, AC-17, IA-2, IA-5, SC-8** | Access enforcement, remote access auth, service identification, authenticator management, transmission confidentiality. |
| **ISO/IEC 27001:2022** | **A.5.15, A.8.2, A.8.5, A.8.24** | Access control, privileged access rights, secure authentication, use of cryptography. |
| **SOC 2 (Trust Services Criteria)** | **CC6.1, CC6.3, CC6.6, CC6.7** | Logical access security, role-based access points, perimeter and transit encryption. |

---

## Anti-Patterns

- **Static Global API Keys:** Using a single persistent shared string passed in headers (`X-API-Key`) across multiple systems without identity binding.
- **Symmetric HMAC Secrets Shared Widely:** Distributing a single shared secret across services for token verification, compromising the entire network if one service is breached.
- **Missing Audience Validation:** Verifying only signature validity while ignoring `aud`, enabling a compromised downstream service to replay the token elsewhere.
- **Excessive Token TTL:** Issuing machine tokens with lifetimes of days or months to bypass re-authentication logic.

---

## Checklist

- [ ] Private keys and client secrets are stored in a dedicated KMS/vault, never in configuration repositories.
- [ ] Ingress APIs validate signature (`alg`), expiration (`exp`), not-before (`nbf`), issuer (`iss`), and audience (`aud`).
- [ ] Token expiration is configured for <= 15 minutes.
- [ ] Token scopes follow granular `<domain>:<resource>:<action>` formatting.
- [ ] JWKS endpoint supports automated rotation with zero downtime.
- [ ] Internal high-risk transit enforces mTLS with mutual certificate verification.
- [ ] All token issuance and failed authentication attempts emit structured audit logs to the SIEM.
