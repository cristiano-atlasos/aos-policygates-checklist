# A/OS PolicyGates Checklist (v0.1 · draft)

Goal: enforce policy **before** action; record evidence **after**. Use this checklist to wire gates that approve / hold / block AI-driven operations in enterprise environments.

## Core Gates (approve / hold / block)

- **Identity & Roles**
- Purpose: ensure only authorized actors can trigger actions.
- Rules: role ∈ {viewer, operator, admin}; mfa_required=true for high-risk.
- Example: `deny if actor.role != "operator" and action in ["write","delete"]`.

- **Approval Chains**
- Purpose: require human sign-off for sensitive actions.
- Rules: 1-2 stage approvals; quorum; expiry.
- Example: `hold if amount > 10_000 until finance.approver_signed`.

- **Scope / Data Domain**
- Purpose: restrict actions by dataset, tenant, region.
- Rules: dataset tags; tenant_id; LGPD region constraints.
- Example: `block if dataset.tag == "PII_sensitive" and env=="prod" and redaction!=true`.

- **Action Type**
- Purpose: gate mutating actions vs. reads.
- Rules: write/delete/external-call require stricter checks.
- Example: `block if action=="external_call" and provider not in allowlist`.

- **Environment (sandbox → prod)**
- Purpose: progressive rollout with safe defaults.
- Rules: sandbox-only for new policies; prod requires canary + metrics.
- Example: `hold if policy.version is new and env=="prod" until canary_ok`.

- **Risk Class**
- Purpose: tighten controls as risk increases.
- Rules: low/medium/high mapped from heuristic or model score.
- Example: `block if risk=="high"; hold if risk=="medium"`.

- **Budget / Rate Limits**
- Purpose: keep volume, cost e erros dentro de SLO/SLAs.
- Rules: QPS, monthly cap, error_budget.
- Example: `hold if monthly_decisions > budget.monthly_cap`.

- **Compliance (LGPD/PCI/HIPAA)**
- Purpose: garantir base legal, minimização e retenção.
- Rules: purpose_limitation; retention_tag; DSR readiness.
- Example: `block if purpose_not_declared or retention_tag missing`.

- **Telemetry & Evidence**
- Purpose: só aprovar se tracing/evidence estiverem ativos.
- Rules: otel_enabled; trace_fields complete.
- Example: `hold if decision_trace.missing_fields > 0`.

- **High-Risk Safeguards**
- Purpose: evitar classes perigosas (pagamentos, acesso, segurança).
- Rules: explicit allow + emergency stop.
- Example: `block "user_delete" unless allow_flag and approver_signed`.

## Operational Controls

- **Emergency STOP:** global switch → `block all high-risk`.
- **Read-only Mode:** fallback seguro durante incidentes.
- **Drift & Replay:** simular, reproduzir e auditar decisões.
- **Change Log:** toda edição de política gera commit + razão + autor.
- **Testing:** unit tests por gate + canary em prod (shadow/hold-first).

## Evidence (post-action)

Emit **DecisionTrace** com: `trace_id, actor, action, target, policy_gate, result, latency_ms, inputs_redacted, isolation_key, evidence_uris, schema_version`.
Exports: **NDJSON** (SIEM/GRC) e **OpenTelemetry JSON**.

---

Status: **draft v0.1** · Backward-compatible increments will follow (v0.1.x).
