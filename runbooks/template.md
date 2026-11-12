---
# ============================================================================
# Managed section — preserve the document metadata when updating this file
# ============================================================================
id: RUNBOOK-0000
type: runbook
title: "Action-oriented title of the operational procedure"
status: draft
created_at: YYYY-MM-DD
updated_at: YYYY-MM-DD
last_verified_at: ""
# ============================================================================
owners:
  - "@owner"
related_specs: []
related_adrs: []
tags: []
---

# RUNBOOK-0000: Action-oriented title of the operational procedure

## Purpose and Triggers

- **Purpose**: State the outcome this procedure produces.
- **Triggers**: List what calls for this procedure, such as alert names, observed symptoms, scheduled maintenance or requests.

## Prerequisites

- **Access & Permissions**: List the systems, roles or environments the operator needs.
- **Tools**: List the required CLIs, scripts or consoles, with minimum versions if they matter.
- **Approvals**: List the change requests or sign-offs needed before starting.
- **Credentials**: Say where to obtain them, such as a secret store path or the credential owner. Never write secret values in this document.

## Risk and Impact

- **Service Impact**: Describe expected downtime, degraded performance or user-visible effects.
- **Data Risk**: Describe the data that may be modified, deleted or exposed.
- **Irreversible Steps**: List the steps that cannot be rolled back, for example Step N.
- **Maintenance Window**: Say when this procedure may and may not be executed.

## Procedure

Execute the steps in order. If a result differs from the **Expected result**, stop and follow **On failure**. Omit **Command** for manual actions.

### Step 1: Short description of the action

- **Command**:
  ```bash
  command --with-arguments
  ```
- **Expected result**: Describe what the operator should observe when the step succeeds.
- **On failure**: Go to [Rollback](#rollback) or [Escalation](#escalation).

### Step 2: Short description of the action

- **Command**:
  ```bash
  command --with-arguments
  ```
- **Expected result**: Describe what the operator should observe when the step succeeds.
- **On failure**: Go to [Rollback](#rollback) or [Escalation](#escalation).

## Verification

- List the checks that confirm the procedure succeeded, such as commands, dashboards or health endpoints, with the expected result of each.

## Rollback

Revert each state-changing step in reverse order:

1. Revert Step 2: command or action.
2. Revert Step 1: command or action.

## Escalation

- **Contacts**: Name the team, on-call rotation or owner to contact.
- **When to Escalate**: List the conditions that require stopping and escalating, for example a failed rollback or impact beyond what **Risk and Impact** describes.

## AI Agent Execution Rules

> This section tells AI agents which steps they may execute and when they must stop.

- **Safe to run without confirmation**: [e.g., Step 1, a read-only status check].
- **Requires human confirmation**: [e.g., Step 2, which restarts the service]. Every state-changing, irreversible or outward-facing step belongs here.
- **Stop conditions**: [e.g., Any unexpected result, a missing prerequisite, or impact beyond what **Risk and Impact** describes].
