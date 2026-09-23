# Resources — IT Service Desk Insights Agent (GHCP)

Start with [Capability-matrix.md](Capability-matrix.md). It maps each agent-scoped
skill to its tool, connector/API operation, prerequisites, dependencies, and ON/OFF behavior.

These resources belong only to the [GHCP scenario](../1.Overview.md), hosted inside
Copilot Studio. The separate [Agent Builder scenario](../../IT-Service-Desk-Insights-Agent/1.Overview.md)
does not use these skills or action tools.

**Included:** documentation, proposed tool contracts, and all 17 instruction-only
[skill definitions](Skills/README.md), one `SKILL.md` per capability. **Not included:**
scripts, connector definitions, deployed workflows, tenant credentials, or completed
evaluation results. Implement/configure and approve the backend components before enabling
actions; uploading a skill does not create tools or grant permissions.

Keep tenant-specific deployment assets in an approved access-controlled location. Only the
capability matrix and Skills folder below are populated; the other entries describe the
expected deployment handoff structure.

| Folder / file | Purpose |
|---|---|
| `Capability-matrix.md` | Included capability catalog, dependencies, profiles, limitations, and capability-card fields |
| `Instructions/` | Versioned GHCP agent instructions |
| [Skills/](Skills/README.md) | Runtime-focused definitions with matching capability names, tool usage, procedures, and failure behavior; setup/dependencies remain in the matrix |
| `Profiles/` | Approved capability selections, actual component IDs, and completed capability cards |
| `Workflows/` | Narrow operation definitions, policy checks, approval contracts, and requester-close workflows |
| `Connections/` | Non-secret connection references, field mappings, API/query/action IDs, and renewal owners |
| `Evaluation/` | Knowledge and capability ON/OFF results; routing exclusions; requester and non-requester closure cases |
| `Sample-documents/` | Sanitized KB articles and incident/diagnostic fixtures, not production personal data |
| `Images/` | Sanitized setup screenshots, if needed |

## Required Deployment Evidence

- Office 365 Users `GetMyProfile` uses each end user's connection, selects minimal fields,
  and is reused only within that user's conversation; profile errors and language fallback
  do not trigger impersonation or broaden source access.
- Approved scope: no ERP, GitHub engineering integration, or agent routing/assignment.
- Separate permission decisions for creation, descriptive updates, notes, attachments, endpoint
  execution, and requester closure. A create-only profile contains no hidden subsequent writes.
- Backend-owned routing behavior, including which creation fields the agent cannot set.
- Authoritative requester mappings for ServiceNow and Dynamics; requester-close lifecycle/state
  rules and proof that read/write privileges alone cannot close someone else's record.
- Positive and negative authorization outcomes, including disabled-action bypass attempts.
- Full initial description persisted by the selected ServiceNow create operation.
- Asynchronous Nexthink handling, partial cross-system handoff recovery, and audit correlation.
- Product/API entitlement and preview acceptance, deployment owner, support owner, and rollback.

Store secrets in managed connections or an approved secret store, never these files. Redact
record contents and diagnostics before adding examples to the public repository.
