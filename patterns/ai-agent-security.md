# Pattern: AI Agent Security Architecture & Governance

## Context
Autonomous AI agents and LLMs equipped with tool-use (function calling), code execution, and database access introduce non-deterministic operational risks. Traditional perimeter defenses fail when agents ingest untrusted external context and possess execution capabilities across production environments.

---

## Decision & Standard

1. **Identity & Application Registration:**
   - Every AI agent must possess a unique, dedicated identity (Service Principal / Workload Identity). Shared "AI System" accounts are prohibited.
   - Credentials must be short-lived, rotated via cloud IAM federation (e.g., OIDC workload identity), and never passed directly into LLM prompts.

2. **Least-Privilege Tool Scopes:**
   - Tools exposed to agents must enforce strict, parameterized function schemas with boundary-checked inputs.
   - Prohibit raw shell execution or arbitrary SQL execution tools. Tools must wrap narrow, single-purpose endpoints (e.g., `lookup_order(order_id)` vs `run_sql_query(query)`).
   - Read-only tools must use read-only database replicas and network policies isolating them from state modification.

3. **Threat Model: Prompt Injection & Tool Abuse:**
   - **Indirect Prompt Injection:** Untrusted inputs (web pages, customer emails, documents) must be quarantined and marked with explicit structural delimiters (e.g., XML tags). Instructions must never treat parsed payload strings as executable directives.
   - **Tool Abuse / Confused Deputy:** The agent engine must enforce authorization on tool invocation: the agent's identity must have permission to act, and the end-user on whose behalf the agent acts must also possess valid delegated rights.

4. **Human-in-the-Loop (HITL) Gates:**
   - Irreversible or high-impact actions (financial transactions, data deletion, privilege grants, external communication dispatch) require synchronous, out-of-band human approval before execution.

5. **Cross-Agent Communication & Trust Boundaries:**
   - Multi-agent message boards or peer-to-peer protocols must treat message payloads as untrusted inputs.
   - Messages between agents must be signed, authenticated, and schema-validated before execution.

6. **Comprehensive Audit Logging:**
   - Audit logs must record: Agent ID, invoking user ID, timestamp, full system prompt hash, user input hash, tool call invoked, validated parameters, and tool return status. Raw sensitive user data must be masked or tokenized prior to long-term SIEM storage.

---

## Controls Mapping

| Framework | Control Reference | Alignment Description |
| :--- | :--- | :--- |
| **NIST SP 800-53 r5** | **AC-3, AC-6, AU-2, SI-4, SC-7** | Access enforcement, least privilege, audit record generation, information system monitoring, boundary protection. |
| **ISO/IEC 27001:2022** | **A.8.2, A.8.3, A.8.15, A.8.20** | Privileged access, information access restriction, logging, network security controls. |
| **SOC 2 (Trust Services Criteria)** | **CC6.1, CC6.3, CC6.8, CC7.2** | Logical access security, unauthorized action prevention, malicious input defense, monitoring. |

---

## Anti-Patterns

- **Universal Agent God-Mode:** Provisioning an agent with broad administrative API keys or unchecked bash terminal access.
- **Context-Window Secret Injection:** Storing database passwords or API keys directly in the system prompt.
- **Blind Auto-Approval:** Relying solely on LLM self-evaluations to authorize high-risk external state changes.
- **Unbounded Agent-to-Agent Trust:** Allowing subagents to execute received peer instructions without parameter sanitation.

---

## Checklist

- [ ] Each agent possesses a unique Service Principal with minimal scoped permissions.
- [ ] Tool parameters are strictly typed, sanitized, and validated against regex/allow-lists.
- [ ] High-risk tool calls (destructive/financial/IAM changes) enforce a blocking Human-in-the-Loop gate.
- [ ] Prompts clearly delineate untrusted third-party inputs using system delimiters.
- [ ] Ephemeral credentials are used; zero static secrets reside in prompt contexts.
- [ ] Structured audit logs capture all tool invocations and user attribution in the central SIEM.
