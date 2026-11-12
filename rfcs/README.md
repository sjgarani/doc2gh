# Requests for Comments (RFCs)

This directory stores the project's **Requests for Comments (RFCs)**. An RFC is used to propose major architectural changes, complex new features, public API/contract modifications, or cross-cutting technical initiatives that benefit from collaborative review prior to implementation.

---

## Lifecycle & Statuses

Change an RFC's status through the project's review workflow.

| Status | Description |
| :--- | :--- |
| `draft` | Initial proposal being drafted by an author or AI agent. |
| `review` | Open for discussion and feedback. |
| `approved` | Proposal accepted; ready to spawn ADRs and executable Specs. |
| `rejected` | Proposal declined (preserved in the repository for historical context). |
| `implemented` | All derived Specs and implementation tasks have been completed. |

---

## Naming Convention

Files follow a 4-digit sequential number in `kebab-case`:

```text
NNNN-proposal-title.md
```

**Examples:**
- `0001-oauth2-authentication-flow.md`
- `0002-async-event-dispatch-pipeline.md`

---

## Creating and Updating RFCs

Start with `template.md` and assign the next available ID. Explain the
proposal, alternatives, and unresolved questions. Submit the RFC for review to invite
discussion, then update the status when the group reaches a decision. Workflow
automation may be added later.

---

## Review and AI Agent Guidance

1. **Review**:
   - Submit new RFCs for review.
   - Resolve items listed under **Open Questions** during review.
2. **Spawning ADRs, Specs, and Runbooks**:
   - Once an RFC transitions to `approved`, extract binding architectural decisions into **ADRs** (`adrs/`), break down the implementation into **Specs** (`specs/`), and identify operational procedures in **Runbooks** (`runbooks/`).
3. **Instructions for AI Agents**:
   - When asked to draft an RFC, inspect the existing codebase, `accepted` ADRs, and any related PRD (`related_prds`) to identify affected components, compatibility risks, and viable alternatives.
   - When an RFC is `approved`, use its **Adoption Plan & Follow-up Artifacts** checklist to create the corresponding ADRs, Specs, and Runbooks.
