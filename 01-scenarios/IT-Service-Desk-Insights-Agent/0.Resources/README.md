# Resources — IT Service Desk Insights Agent

Start with [Capability-matrix.md](Capability-matrix.md) for Path B. It maps each agent-scoped
skill to its tool, connector/API operation, prerequisites, dependencies, and ON/OFF behavior.

**Included:** documentation and proposed operation contracts. **Not included:** executable
skill bundles, connector definitions, deployed workflows, tenant credentials, or completed
evaluation results. Build and approve those deployment artifacts before enabling actions.

Keep tenant-specific deployment assets in an approved access-controlled location. The following
folders describe the expected handoff structure; they are not pre-populated in this repository.

| Folder / file | Purpose |
|---|---|
| `Capability-matrix.md` | Included capability catalog, dependencies, profiles, limitations, and capability-card fields |
| `Instructions/` | Versioned Path A or Path B agent instructions |
| `Skills/` | Implemented skill packages to attach only to this agent; no organization-wide installation |
| `Profiles/` | Approved capability selections, actual component IDs, and completed capability cards |
| `Workflows/` | Narrow operation definitions, policy checks, approval contracts, and requester-close workflows |
| `Connections/` | Non-secret connection references, field mappings, API/query/action IDs, and renewal owners |
| `Evaluation/` | Knowledge and capability ON/OFF results; routing exclusions; requester and non-requester closure cases |
| `Sample-documents/` | Sanitized KB articles and incident/diagnostic fixtures, not production personal data |
| `Images/` | Sanitized setup screenshots, if needed |

## Required Deployment Evidence

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
