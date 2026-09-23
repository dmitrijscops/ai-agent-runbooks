# Resources — IT Service Desk Insights Agent

This folder supports the read-only [Agent Builder scenario](../1.Overview.md).
The [runbook](../3.Runbook.md) includes the starting agent instructions, and
[sample prompts](../4.Sample-prompts.md) define the evaluation cases.

No tenant assets or completed evaluation results are included. The following entries
describe the expected deployment handoff structure, not populated folders:

| Folder / file | Purpose |
|---|---|
| `Instructions/` | Version-controlled copies of the approved Agent Builder instructions |
| `Evaluation/` | The 30-row evaluation set, consistency results, and cross-user permission evidence |
| `Sample-documents/` | Sanitized knowledge articles for a demo ServiceNow instance |
| `Images/` | Sanitized setup screenshots, if needed |

Keep the knowledge inventory, source-scope decision, connector configuration, identity
mapping, citation checks, and positive/negative permission results with the delivery
handoff. Store tenant-sensitive evidence in an approved access-controlled location,
not in this public repository. Never commit credentials.

The capability matrix and 17 action-skill definitions belong only to the separate
[GHCP scenario resources](../../IT-Service-Desk-Insights-Agent-ghcp/0.Resources/README.md);
they are not dependencies of this read-only implementation.
