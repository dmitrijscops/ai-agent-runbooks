# Capability Matrix - Investigation and Resolution

This is the deployment contract for the **GHCP scenario**, one Copilot Studio agent powered by the
GitHub Copilot harness. Skills are attached only to this agent. No organization-wide skill
installation, ERP integration, or GitHub engineering integration is required.

**Implementation status:** all 17 instruction-only [skill definitions](Skills/README.md) are
included; each capability below links to its own `Skills/<capability-name>/SKILL.md`.
Agent-facing tools remain contracts to implement/configure. Connector definitions, scripts,
and workflow exports are not included. The delivery team must provision and validate the
tools, connections, and backend policies before enabling a capability. Native connector
operation IDs are identified separately.

## Shared User-Context Tool

All deployment profiles include the following read-only supporting tool. It is not an
eighteenth product-action skill and does not enable any ticket/case mutation.

| Agent-facing tool | Connector | Native action / operation ID | Configuration |
|---|---|---|---|
| `GetMyProfile` | Office 365 Users | **Get my profile (V2)** / `MyProfile_V2` | Current user's connection; fixed Select fields: `id,displayName,mail,userPrincipalName,preferredLanguage` |

The V2 action returns the current user of its connection. Bind it to the end-user
connection, not a maker-owned or shared service connection. Do not expose "Get user profile,"
user search, profile updates, or arbitrary HTTP as substitutes. No target-user ID is an input
to this tool. The configured field selection avoids unnecessary directory data and fields
that tenant policy may block.

Call once when user details or a language fallback are needed, then reuse the result across
skills in the same user's conversation. Do not share this cache between users/conversations;
invalidate it on a user/connection change and refresh after an explicit profile refresh request.
Cache profile metadata, not permission decisions. Record/device checks still run in the backend.

Use `displayName` and the configured identity/contact fields to avoid repeatedly asking for
known details. Keep null fields null; `mail` may be absent and `userPrincipalName` is not
automatically a deliverable email address. Do not invent a ServiceNow requester ID, Dynamics
contact, or device ownership from these values; backend mappings remain authoritative.

For response language use: explicit user preference, then current-message language, then
nonempty `preferredLanguage`. If still unclear, ask. Preserve technical identifiers and the
configured backend handoff language. This action does not supply a timezone; never infer it
from language, region, or office location.

On profile failure, report the limitation and stop operations that require the missing
details. Do not impersonate another user or fall back to directory search. Source-authorized
knowledge/catalog reads that do not need profile data can continue. Do not return or copy
the whole profile into a ticket, note, or log.

## Execution Controls

A capability is ON only when its skill, tool, connection, dependencies, and backend policy
are configured and the caller is authorized. Required confirmations must also be satisfied.
Skills guide the agent; they do not enforce access.

- **ON:** attach the skill, expose only its named tools, configure the connection, and allow
  the operation in the backend policy for the approved users/records/devices.
- **OFF:** revoke execution authorization, remove the connector action or disable the MCP
  tool, and remove the skill association. Audit other workflows and APIs for bypass paths.
- Keep shared connections available for capabilities that remain ON. Disabling the entire
  connector is a product-wide action, not a create-versus-update switch.
- Evaluate restrictions at tool execution, including existing sessions. A stale prompt,
  skill, cached tool catalog, or prior approval must not override a revoked capability.
- Reject inconsistent profiles: a required skill/tool/connection is missing, or a dependency
  is OFF. Do not silently enable writes to satisfy a dependency.

The platform supports connector-action selection and individual MCP tool toggles. Skill
management supports adding/removing skills. There is no assumed native atomic switch that
updates all these layers; the runbook coordinates them as a versioned deployment profile.

## Common Prerequisites

| ID | Prerequisite | Owner |
|---|---|---|
| P0 | Copilot Studio environment with GitHub Copilot harness, capacity, authenticated channel, approved DLP/network access, and separate pilot configuration | Power Platform admin |
| U0 | Office 365 Users end-user connection; `GetMyProfile` bound to `MyProfile_V2` with minimal fields; current-user conversation cache and profile failure handling | Power Platform admin |
| S0 | ServiceNow connection and instance; scoped API/table access; requester identity mapping; required incident fields and source-owned lifecycle rules | ServiceNow admin |
| N0 | Nexthink tenant/region endpoint and scoped API credentials; published query/action IDs; verified requester-to-device mapping | Nexthink admin |
| D0 | Dynamics 365 Customer Service environment and Dataverse connection; Case (`incident`, entity set `incidents`) privileges; customer/contact and individual requester mapping | Dynamics admin |
| W0 | Narrow workflow/API policy: fixed operation/schema, caller authorization, field/action allowlists, confirmation, deduplication, audit, and explicit outcomes | Integration/backend team |

Every capability uses P0 and the shared U0 tool, invoked only when its profile data is needed.
Product dependencies below list the additional requirements. Reads also need authorization,
field filtering, and bounded queries. A service connection or returned directory profile
never establishes the requester's entitlement by itself.
Credentials remain in managed connections/secrets, not skill text or source control.

## ServiceNow Capabilities

Use the ServiceNow Power Platform connector behind fixed-scope tools. A custom API/workflow
is required when native operation behavior does not meet the contract.

| Skill / capability ID | Agent-facing tools | Connector action / implementation | Additional prerequisites and dependencies |
|---|---|---|---|
| [`sn-knowledge-read`](Skills/sn-knowledge-read/SKILL.md) | `SearchKnowledge`, `GetKnowledgeArticle` | `GetKnowledgeArticles`, `GetKnowledgeArticle` | S0; `sn_km_api` plugin; approved KBs and access checks, including full article content |
| [`sn-catalog-read`](Skills/sn-catalog-read/SKILL.md) | `SearchCatalog`, `GetCatalogItem` | `GetCatalogItems`, `GetCatalogItem` | S0; allowed catalog IDs; read-only, no `OrderItem` |
| [`sn-ticket-read`](Skills/sn-ticket-read/SKILL.md) | `FindIncidents`, `GetIncident` | `GetRecords`, `GetRecord`, fixed to `incident` | S0; caller-permitted record scope and returned fields; no unrestricted encoded queries |
| [`sn-ticket-create`](Skills/sn-ticket-create/SKILL.md) | `CreateIncident` | `CreateRecord`, or custom create-only API/workflow | S0, W0, `sn-ticket-read`; required fields, duplicate check and idempotency; complete initial evidence; no routing fields or subsequent mutations |
| [`sn-ticket-update`](Skills/sn-ticket-update/SKILL.md) | `UpdateIncidentDetails` | Workflow over `UpdateRecord` | S0, W0, `sn-ticket-read`; allowlisted descriptive fields only; excludes notes, routing/owner fields, priority overrides, and lifecycle state |
| [`sn-ticket-note`](Skills/sn-ticket-note/SKILL.md) | `AddInternalNote` | Workflow over `UpdateRecord`, fixed to `work_notes` | S0, W0, `sn-ticket-read`; authorized append-only internal evidence; not public comments or a general field update |
| [`sn-ticket-attach`](Skills/sn-ticket-attach/SKILL.md) | `AttachIncidentEvidence` | Workflow over `UploadAttachmentFile` | S0, W0, `sn-ticket-read`; attachment permission, file limits, redaction, approved target record |
| [`sn-ticket-close`](Skills/sn-ticket-close/SKILL.md) | `CloseOwnIncident` | Dedicated requester-close workflow using the backend's approved self-service transition | S0, W0, `sn-ticket-read`; explicit requester confirmation, authoritative requester match and eligible current state |

**Creation limitation:** native `CreateRecord` documents that full description content can be
ignored. Validate the actual payload. If the investigation cannot be persisted at creation,
use a supported custom create operation; never hide an `UpdateRecord`, note, or attachment
inside a create-only capability. If the create contract still cannot be satisfied, block
creation and provide the proposed handoff for manual use.

`sn-ticket-close` does not depend on `sn-ticket-update`: its workflow is a separate, constrained
requester action, not general write access. It must not allow an operator to resolve another
person's incident. Do not hard-code ServiceNow state numbers; the source admin supplies the
eligible requester transition and required fields.

## Nexthink Capabilities

Use a custom connector or narrowly scoped MCP adapter over documented APIs. A first-party
Nexthink MCP server is not assumed. NQL executes published, parameterized query IDs, not
arbitrary model-authored queries.

| Skill / capability ID | Agent-facing tool | API / implementation | Additional prerequisites and dependencies |
|---|---|---|---|
| [`nx-device-read`](Skills/nx-device-read/SKILL.md) | `FindUserDevices` | Predefined query through `/api/v2/nql/execute` | N0; authenticated requester mapping; bounded permitted-device results and correct Collector ID |
| [`nx-diagnostics-read`](Skills/nx-diagnostics-read/SKILL.md) | `GetDeviceDiagnostics` | Approved parameterized NQL queries through `/api/v2/nql/execute` | N0, `nx-device-read`; device/time-window constraints; query IDs for agreed connectivity, performance, or crash evidence |
| [`nx-execution-read`](Skills/nx-execution-read/SKILL.md) | `GetExecutionResult` | Predefined NQL query over execution results | N0, `nx-device-read`; validate request-to-device ownership; report observation time and per-target outcome |
| [`nx-diagnostics-run`](Skills/nx-diagnostics-run/SKILL.md) | `CollectDeviceDiagnostics` | `/api/v1/act/execute`, data-collection action allowlist | N0, W0, `nx-device-read`, `nx-diagnostics-read`, `nx-execution-read`; API-enabled actions and required approval |
| [`nx-remediation-run`](Skills/nx-remediation-run/SKILL.md) | `RunApprovedFix` | `/api/v1/act/execute`, separate remediation allowlist | N0, W0, `nx-device-read`, `nx-diagnostics-read`, `nx-execution-read`; approved fix IDs/parameters, target-specific confirmation and recovery plan |

Reading stored telemetry and running a data-collection script are different capabilities.
Both diagnostic execution and remediation call the same Remote Actions endpoint, so their
backend action-ID allowlists must remain distinct. An API endpoint toggle alone cannot
separate them.

Nexthink uses dedicated service API credentials. The integration must enforce requester/device
authorization. Query-management permissions are for the admin who publishes queries, not
automatically for the runtime identity.

Keep the original `requestId`. Accepted is not completed: an offline device can execute later
or the request can expire. Report pending, completed, failed, expired, or unknown explicitly.
Do not submit the same action again just because a bounded poll did not observe completion.
When disabling execution, identify accepted jobs already queued in Nexthink; removing the tool
does not cancel them. Follow backend procedures and report their outstanding status.

## Dynamics 365 Customer Service Capabilities

Use the Microsoft Dataverse connector, fixed to the approved environment and Case table.
This is Customer Service case management, not Dynamics ERP.

| Skill / capability ID | Agent-facing tools | Connector action / implementation | Additional prerequisites and dependencies |
|---|---|---|---|
| [`d365-case-read`](Skills/d365-case-read/SKILL.md) | `FindCases`, `GetCase` | `ListRecords`, `GetItem` | D0; permitted record/field scope; requester-authorized customer/contact lookup where needed |
| [`d365-case-create`](Skills/d365-case-create/SKILL.md) | `CreateCase` | Workflow over `CreateRecord` ("Add a new row") | D0, W0, `d365-case-read`; required customer/contact fields, duplicate check, correlation key, and source-ticket reference if present |
| [`d365-case-update`](Skills/d365-case-update/SKILL.md) | `UpdateCaseDetails` | Workflow over `UpdateOnlyRecord` ("Update a row") | D0, W0, `d365-case-read`; allowlisted descriptive fields only; no owner, queue, routing, or lifecycle fields |
| [`d365-case-close`](Skills/d365-case-close/SKILL.md) | `CloseOwnCase` | Requester-authorized workflow invoking the supported closure operation, such as `CloseIncident` | D0, W0, `d365-case-read`; explicit confirmation, verified individual requester, current-state checks and required resolution data |

Do not expose `UpdateRecord` (Dataverse upsert), generic bound/unbound action execution, or
arbitrary table selection. A wrapper may use a fixed business action internally but cannot
let the model choose another action.

`d365-case-create` does not invoke routing and does not depend on a ServiceNow write capability.
If its input references a ServiceNow incident, read that record through `sn-ticket-read` and
transfer only authorized evidence. Adding a backlink later requires the separately enabled
`sn-ticket-note`; it is not part of creating the Dynamics case.

`d365-case-close` is a separate permission from descriptive updates. Having write access,
being a case owner, or belonging to the same customer account does not establish requester
identity. If the deployment lacks an authoritative individual requester relationship, leave
closure OFF.

## Hard Limitations

- **Routing and assignment belong to backend teams.** There are no routing skills/tools.
  Neither create nor update accepts model-selected owners, queues, assignees, or assignment
  groups. Backend defaults/rules may assign records independently.
- **Closure is requester-side only.** No technician/admin closure of another requester's
  record and no automatic closure after successful diagnostics or remediation.
- An explicit closure request applies to one identified record. Never cascade closure
  across ServiceNow and Dynamics without a separate confirmation and requester check.
- A blocked transition is not permission to force administrative closure. Report blocked or
  pending backend processing honestly; never describe a submitted request as confirmed closed.
- ERP, GitHub engineering support, record deletion, catalog ordering, public-comment writes,
  arbitrary scripts, and unrestricted CRUD are not included.

## Example Deployment Profiles

These are proposed profiles to approve, not pre-enabled features. All include the shared
`GetMyProfile` tool. **Any product capability not listed is OFF.** All routing/assignment
capabilities remain absent in every profile.

| Profile | Enabled capabilities | Intended result |
|---|---|---|
| `knowledge-only` | `sn-knowledge-read`, `sn-catalog-read` | KB/catalog answers without live incident access or mutations |
| `diagnose-create` | `sn-knowledge-read`, `sn-ticket-read`, `sn-ticket-create`, `nx-device-read`, `nx-diagnostics-read` | Investigate stored telemetry and create a complete initial incident; no changes after creation |
| `assisted-handoff` | All `diagnose-create` capabilities plus `sn-ticket-note`, `nx-execution-read`, `nx-diagnostics-run`, `nx-remediation-run`, `d365-case-read`, `d365-case-create` | Approved diagnostics/remediation; optional case handoff to backend-owned routing |
| `requester-close` | `sn-ticket-read`, `sn-ticket-close`, `d365-case-read`, `d365-case-close` | Close only the user's own explicitly confirmed records; no general updates |

Profiles can be combined only after reviewing the resulting union of permissions. For example,
adding `requester-close` to `diagnose-create` allows requester closure, so the result is no longer
a strict create-only profile. For a single-product closure deployment, include only that
product's read/close pair and its prerequisites.

## Capability Card - Required Handoff Fields

Complete one card per enabled capability, including its actual deployed mappings:

| Field | What the offshore delivery team must record |
|---|---|
| Identity | Capability/skill ID, agent ID, version, target environment, profile and ON/OFF state |
| Dependencies | Required capability IDs, actual skill files, tool/workflow IDs, connection references |
| Implementation | Native operation or custom endpoint, fixed table/action/query IDs, input/output schema |
| Authorization | Authenticated requester mapping, execution identity, source permissions, record/device/field scope |
| Approval | Which action/record/device/payload is confirmed, evidence lifetime, and invalidation conditions |
| Prerequisites | Plugin/features, credentials owner, shared profile tool/user binding, query/action IDs, field mappings, eligible closure states |
| Enable/disable | Skill association, exposed actions, execution-policy changes, republishing and session handling |
| Outcomes | Expected success, OFF/denied behavior, pending/unknown response, and safe recovery procedure |
| Operations | Audit location, correlation/idempotency keys, credential renewal, rollback and support owner |

All operation results need an explicit outcome and correlation ID. Writes return the record
ID/URL and actual persisted status; asynchronous tools return their execution request ID.
Do not convert errors into empty successful results. Preserve successful partial steps so
recovery does not duplicate records or endpoint actions.

## References

- [Office 365 Users: Get my profile (V2), selected fields, and limitations](https://learn.microsoft.com/en-us/connectors/office365users/)
- [ServiceNow connector and native operation IDs](https://learn.microsoft.com/en-us/connectors/service-now/)
- [ServiceNow Table API](https://www.servicenow.com/docs/r/api-reference/rest-apis/c_TableAPI.html)
- [Nexthink NQL API](https://docs.nexthink.com/api/nql)
- [Nexthink Remote Actions API](https://docs.nexthink.com/platform/user-guide/remote-actions/setting-up-and-managing-remote-actions/creating-remote-actions/remote-actions-api)
- [Microsoft Dataverse connector and operation IDs](https://learn.microsoft.com/en-us/connectors/commondataserviceforapps/)
- [Dynamics Customer Service Case operations](https://learn.microsoft.com/en-us/dynamics365/developer/reference/entities/incident)
- [Agent skill management](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agents-experience/skills-manage)
- [Agent tool management](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agents-experience/tools-manage)
