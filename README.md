# A/OS PolicyGates Checklist (v0.1 · draft)

Minimal checklist of *PolicyGates* to approve/hold/block AI actions in enterprise environments.
Each gate states: **ID**, **Scope**, **Rule**, **Rationale**, **Risk**, **Evidence fields**.

## Gates (starter set)

1. **high-risk-write** — *block unless human sign-off*
Scope: prod systems, financial records, legal docs
Rule: block by default; require admin approval + ticket ID
Evidence: trace_id, actor, target, approval_id

2. **finance-approve** — *hold unless controller approves*
Scope: invoices, payments, vendor changes
Rule: hold; release on approval by controller role
Evidence: trace_id, policy_enforced=true, approver_role, approval_ts

3. **pii-exfil** — *block*
Scope: PII/PHI export outside allowlist domains
Rule: block; raise incident
Evidence: trace_id, fields_redacted, incident_id

4. **prod-credentials** — *block*
Scope: secrets, tokens, keys
Rule: block; require break-glass procedure
Evidence: trace_id, secret_type, break_glass_id (if any)

5. **external-call** — *hold unless destination is allowlisted*
Scope: HTTP/API calls
Rule: hold; auto-approve if dest ∈ allowlist
Evidence: trace_id, dest_host, allowlist_hit

6. **data-deletion** — *hold (two-person rule)*
Scope: delete/update destructive ops
Rule: require two approvals (requester ≠ approver)
Evidence: trace_id, approver_1, approver_2

7. **vendor-onboard** — *approve when checks complete*
Scope: onboarding flows
Rule: approve if KYC/Tax/Contract flags = pass
Evidence: trace_id, checks_passed=true, checklist_ref

8. **telemetry-allow** — *approve (read-only)*
Scope: metrics/logs queries
Rule: approve if read-only + within quota
Evidence: trace_id, read_only=true, quota_remaining

---

### Evidence & exports
- DecisionTrace fields: `trace_id, episode_id, actor, action, target, policy_enforced, result, evidence_uris`.
- Export formats: NDJSON + OpenTelemetry JSON (see repos: *aos-decision-trace-schema* and *aos-evidence-export-packs*).

### Contributing
Open an issue or PR to add gates per domain (FIN/SEC/OPS/LEGAL).

---

Status: **draft v0.1** · Backward-compatible increments will follow (v0.1.x).
