# Architecture Decision Records (ADRs)

This directory contains the project's **Architecture Decision Records (ADRs)**. An ADR captures a significant architectural decision, the context and problem it addresses, the options evaluated, and the resulting consequences.

---

## Lifecycle & Statuses

Change an ADR's status through the project's review workflow.

| Status | Description |
| :--- | :--- |
| `proposed` | Decision is under review. |
| `accepted` | Decision is approved and active. Developers and AI agents must follow its rules. |
| `deprecated` | Decision is no longer relevant because the underlying system or feature was retired. |
| `superseded` | Decision has been replaced by a newer ADR (requires `--superseded-by ADR-NNNN`). |

---

## Naming Convention

Files follow a 4-digit sequential number in `kebab-case`:

```text
NNNN-short-decision-title.md
```

**Examples:**
- `0001-adopt-doc2gh-documentation-architecture.md`
- `0002-use-postgresql-as-primary-database.md`

---

## Creating and Updating ADRs

Start with `template.md` and assign the next available ID. Explain the decision,
the alternatives considered, and the consequences. Submit the ADR for review,
then update the status when the decision is settled. Workflow
automation may be added later.

---

## AI Agent Guidance

1. **Instructions for AI Agents**:
   - Read all `accepted` ADRs before proposing or reviewing architectural changes.
   - Enforce all rules listed under **Compliance & Validation (AI & Code Review)** when generating or reviewing code.
   - Never alter the core decision of an `accepted` ADR; instead, create a new ADR and mark the previous one as `superseded`.
