# Technical & Functional Specifications (Specs)

This directory stores the project's **Specifications (Specs)**. A Spec is an executable blueprint that guides developers and **AI agents** through the implementation of features, modules, or refactorings, with implementation work and review records.

---

## Lifecycle & Statuses

Change a Spec's status through the project's review workflow.

| Status | Description |
| :--- | :--- |
| `draft` | Specification is being authored; requirements or design are still incomplete. |
| `ready` | Specification is reviewed and ready to be implemented by a developer or AI agent. |
| `in-progress` | Implementation is actively underway. |
| `done` | All acceptance criteria and tasks have been implemented, tested, and merged. |
| `archived` | Historical specification or superseded by a newer Spec. |

---

## Naming Convention

Files follow a 4-digit sequential number in `kebab-case`:

```text
NNNN-feature-or-module-name.md
```

**Examples:**
- `0001-user-registration-endpoint.md`
- `0002-webhook-retry-worker.md`

---

## Creating and Updating Specs

Start with `template.md` and assign the next available ID. Complete the
specification, link it to the relevant work items and review records, and keep
its status current throughout implementation. Workflow automation may be
added later.

---

## Traceability and AI Agent Guidance

1. **Single Source of Truth for AI Agents**:
   - The **AI Agent Context** section specifies affected directories/files, mandatory ADRs to follow, and test/verification commands.
   - The **Task Breakdown** uses task-list syntax (`- [ ]`) so developers and AI agents can track incremental implementation progress.
2. **Traceability**:
   - Every Spec in `ready` or `in-progress` status should reference its upstream documents (`related_prds`, `related_rfcs`, `related_adrs`) and review records.
