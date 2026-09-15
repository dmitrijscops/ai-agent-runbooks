# Review Checklist

Use this after authoring or revising a runbook. Apply only relevant items and distinguish
local document checks from checks that require a deployed agent and source systems.

## Scope and Evidence

- The target runtime and host match the user's actual intent.
- Included products, excluded products, users, and autonomy level follow the agreement,
  not assumptions imported from another runbook.
- The architecture contains sources for the records, telemetry, or feedback it claims
  to analyze. Knowledge connectors are not presented as live ticket feeds.
- Native operations and hosting support are verified from current official references.
  Unverified capabilities and custom work are labeled; preview is not silently called GA.
- Included files and missing deployment components are described honestly.
- Review-only requests have not turned into unsolicited implementation.

## Skill Coverage and Structure

- Every matrix capability has exactly one matching skill folder and `SKILL.md`.
- Folder name, YAML `name`, matrix ID, and linked file agree; names are unique,
  lowercase kebab-case and no more than 64 characters.
- YAML front matter has a nonempty name and concise description. Files use UTF-8
  without a BOM; no unresolved template placeholders remain in final definitions.
- Every runtime skill has scope, tools, inputs, procedure, and result/failure guidance.
- Runtime skills assume configuration; deployment prerequisites and connector-setup
  prose remain in the matrix/runbook.
- Descriptions are useful activation cues rather than repetitive authentication claims.
- Each tool list matches its own tools, required read tools, conditional tools, and
  applicable shared context tools. Native API names are not alternative execution paths.
- Read skills do not execute actions; result-tracking skills do not resubmit work.
- Procedures are action-specific and numbered in order.

## Capability and Identity Boundaries

- Product capability profiles are explicit, default omitted actions to OFF, and include
  their mandatory dependencies. Conditional dependencies are documented.
- Skill removal is not treated as permission revocation. Tool exposure, alternate
  workflows, backend checks, and stale sessions are addressed.
- Creation contains the complete initial payload without a hidden subsequent mutation.
- Generic updates cannot bypass restrictions on lifecycle, routing, notes, or ownership.
- Where backend teams own assignment, no model-selected routing fields or routing tool
  appears during creation or subsequent updates.
- Where closure is requester-only, it requires an explicit own-record request and
  source-state validation. General updates can remain OFF; a fix does not close records.
- Linked records are not implicitly mutated or closed together.
- Reads or returned profile fields do not establish ownership or action permission.

## User Context and Language

- Any current-user profile tool runs through the intended user's connection, not a
  maker identity accidentally inherited by a workflow.
- Only necessary profile fields are requested; null values are not fabricated.
- Profile context is reused within one user's conversation, not across users, and
  is refreshed after identity/connection changes or an explicit refresh request.
- Permissions and approvals are not cached as profile metadata.
- Explicit language preference wins, then message language, then profile preference.
  Missing timezone is not inferred from language, region, or office location.
- Existing intake/device details are reused without skipping current backend checks.
- Profile errors stop only dependent work, are reported, and never cause impersonation
  or a broader directory search.

## Operation Outcomes

- No-op descriptive updates avoid writes.
- Confirmation remains bound to the actual target, action, and payload.
- Concurrent changes trigger reevaluation instead of blind overwrites.
- Empty source results are distinguished from access denial and retrieval failure.
- Asynchronous acceptance, completion, failure, expiry, and unknown state are distinct.
- Unknown writes are reconciled before retries; IDs and successful partial steps are
  preserved, avoiding duplicate records or repeated endpoint actions.
- Disabling a tool is not falsely described as cancelling accepted backend work.
- Source IDs, URLs, timestamps, and status labels are preserved without invention.
- Sensitive content is minimized and external text is treated as data, not instructions.

## Documentation and Local Checks

- Overview, architecture, runbook, prompts, resource index, and directly affected
  repository indexes describe the same scope.
- Each skill is linked from the matrix and Skills index; local links/anchors resolve.
- Markdown code fences and tables are valid; line endings are preserved.
- Old "skills not included" claims and duplicate inline skill copies are removed.
- The runbook contains at least one combined scenario and disabled/failed variations.
- Templates are not confused with delivered implementations or live test results.

Use the repository's existing tooling when available. For small local checks, compare
the capability set with skill-directory names, parse the actual front matter, check
declared tool sets, and resolve relative links. Normalize CRLF/LF before parsing.
Do not assert arbitrary words such as "fallback" or "backend" must appear when the same
rule is expressed correctly in different prose; inspect semantics instead.

Live connector checks require a configured environment and explicit authorization.
If none is available, state that the deliverable is the definitions/documentation,
not a deployed or production-validated integration.
