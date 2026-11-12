# Product Requirements Documents (PRDs)

This directory stores the project's **Product Requirements Documents (PRDs)**. A PRD defines what capability is being built and why from a user and business perspective—capturing the problem, goals, success metrics, user scenarios, and scope boundaries before technical design and implementation.

---

## Lifecycle & Statuses

Change a PRD's status through the project's review workflow.

| Status | Description |
| :--- | :--- |
| `draft` | Initial product requirements being drafted by an author or AI agent. |
| `review` | Open for product, design, and engineering review. |
| `approved` | Scope and success metrics accepted; ready to spawn RFCs and Specs. |
| `delivered` | All derived RFCs and Specs have been implemented and released. |
| `archived` | Historical, declined, or superseded PRD preserved for context. |

---

## Naming Convention

Files follow a 4-digit sequential number in `kebab-case`:

```text
NNNN-product-initiative-title.md
```

**Examples:**
- `0001-self-service-sso-onboarding.md`
- `0002-usage-based-billing-alerts.md`

---

## Creating and Updating PRDs

Start with `template.md` and assign the next available ID. Define the problem, success metrics, in-scope requirements, and explicit non-goals. Submit the PRD for review, then update its status as product validation and delivery progress. Workflow automation may be added later.

---

## Review and AI Agent Guidance

1. **Review**:
   - Submit new PRDs for review.
   - Resolve items listed under **Open Questions** during review.
2. **Spawning RFCs and Specs**:
   - Once a PRD transitions to `approved`, use **Adoption Plan & Follow-up Artifacts** to create the corresponding **RFCs** (`rfcs/`) and **Specs** (`specs/`).
3. **Instructions for AI Agents**:
   - When asked to draft or review a PRD, help clarify the user problem, measurable metrics under **Goals and Success Metrics**, key **User Scenarios**, and explicit **Out-of-Scope (Non-Goals)** boundaries.
   - When drafting an RFC or Spec from an `approved` PRD, reference it in `related_prds` and respect its scope and requirements.
