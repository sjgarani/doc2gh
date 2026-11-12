# Runbooks

This directory stores the project's **Runbooks**. A Runbook is a step-by-step operational procedure that a developer, an operator or an **AI agent** can execute. Runbooks cover recurring operations, routine maintenance and incident response, such as rotating credentials, restoring a backup or recovering a failed service.

---

## Lifecycle & Statuses

Change a Runbook's status through the project's review workflow.

| Status | Description |
| :--- | :--- |
| `draft` | Procedure is being written or has never been executed end to end. |
| `active` | Procedure has been verified (`last_verified_at` is set) and can be executed. |
| `deprecated` | Procedure no longer applies because the system or operation was retired or replaced. The file is kept for historical context. |

---

## Naming Convention

Files follow a 4-digit sequential number in `kebab-case`:

```text
NNNN-action-or-scenario.md
```

**Examples:**
- `0001-rotate-database-credentials.md`
- `0002-restore-service-from-backup.md`

---

## Creating and Updating Runbooks

Start with `template.md` and assign the next available ID. Describe when to run the procedure,
its prerequisites and its risks. Then write each step with its command, expected result and
failure handling. Never store secrets in a Runbook. Point to the secret store or the credential
owner instead.

Link the Runbook to the Specs and ADRs it derives from (`related_specs`, `related_adrs`). Submit the Runbook for review, and move it to
`active` only after a successful end-to-end execution, recorded in `last_verified_at`.

When the underlying system changes, update the steps and `updated_at`. Change
`last_verified_at` only after the next successful execution. Workflow automation may be added
later.

---

## AI Agent Guidance

1. **Instructions for AI Agents**:
   - Execute only `active` Runbooks. Treat `draft` Runbooks as unverified, and never execute `deprecated` ones.
   - Follow the **AI Agent Execution Rules** section. Run the steps listed under **Safe to run without confirmation** directly. Ask for human confirmation before any step listed under **Requires human confirmation**. Stop at any of the **Stop conditions**.
   - When a step's result differs from its **Expected result**, follow its **On failure** instructions instead of continuing.
   - When asked to draft a Runbook, inspect the related Specs, ADRs and code to identify the commands, risks and rollback steps.
