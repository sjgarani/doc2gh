---
# ============================================================================
# Managed section — preserve the document metadata when updating this file
# ============================================================================
id: RUNBOOK-0001
type: runbook
title: "Configure the GitHub repository"
status: draft
created_at: 2026-11-11
updated_at: 2026-11-12
last_verified_at: ""
# ============================================================================
owners:
  - "@sjgarani"
related_specs: [SPEC-0001]
related_adrs: [ADR-0002, ADR-0003]
tags: [github, settings, branch-protection]
---

# RUNBOOK-0001: Configure the GitHub repository

## Purpose and Triggers

- **Purpose**: bring the repository's merge settings in line with
  [ADR-0002](../adrs/0002-branching-and-merge-workflow.md) (rules 3 and 4) and, where the plan allows it,
  protect `main` as [ADR-0003](../adrs/0003-use-github-actions-for-ci.md) (rule 6) requires: pull requests
  mandatory and the required stages from [SPEC-0001](../specs/0001-ci-pipeline.md) as required status
  checks.
- **Triggers**:
  - a repository is adopting doc2gh: Steps 1 to 3 before its first pull request, so that pull request
    can only be merged with a merge commit, and Step 4 during the pull request that implements SPEC-0001
    (Task 5);
  - the policy stage reports a settings mismatch, or a local run with an admin's credentials prints
    "repository settings not checked" (SPEC-0001, FR-04); the CI token usually cannot read the settings,
    so that note in CI is expected;
  - a plan or visibility change makes branch protection available;
  - a new stage has become required (SPEC-0001, FR-08).

## Prerequisites

- **Access & Permissions**: admin role on the repository.
- **Tools**: `gh`, authenticated (`gh auth status`) with the `repo` scope.
- **Approvals**: agreement from the repository owner before Steps 2 and 4, which change the rules for
  every collaborator. One agreement, given before Step 2, covers both steps of the same execution;
  during adoption, the request to adopt doc2gh is that agreement.
- **Credentials**: only the `gh` login. Never paste tokens into commands, logs, or pull requests.

In the commands, `<owner>/<repo>` stands for the project repository.

## Risk and Impact

- **Service Impact**: no service is affected. After Step 2, open pull requests can only be merged with a
  merge commit. After Step 4, nobody, admins included, can push directly to `main` or merge while a
  required check is failing.
- **Data Risk**: none; the procedure only changes settings.
- **Irreversible Steps**: none; [Rollback](#rollback) restores the values recorded in Step 1.
- **Maintenance Window**: any time nobody is in the middle of merging a pull request.

## Procedure

Execute the steps in order. If a result differs from the **Expected result**, stop and follow **On failure**. Omit **Command** for manual actions.

### Step 1: Record the current settings

- **Command**:
  ```bash
  gh api repos/<owner>/<repo> \
    --jq '{visibility, allow_merge_commit, allow_squash_merge, allow_rebase_merge, delete_branch_on_merge}'
  gh api repos/<owner>/<repo>/branches/main/protection
  ```
- **Expected result**: all five fields filled in; keep them for the rollback (during adoption, in the
  adoption pull request's description). The second call returns
  the current protection, `Branch not protected` (HTTP 404), or "Upgrade to GitHub Pro or make this
  repository public to enable this feature." (HTTP 403); note which one.
- **On failure**: missing or `null` merge fields mean you lack admin rights; see
  [Escalation](#escalation).

### Step 2: Apply the merge settings

- **Command**:
  ```bash
  gh api -X PATCH repos/<owner>/<repo> \
    -F allow_merge_commit=true -F allow_squash_merge=false \
    -F allow_rebase_merge=false -F delete_branch_on_merge=true \
    --jq '{allow_merge_commit, allow_squash_merge, allow_rebase_merge, delete_branch_on_merge}'
  ```
- **Expected result**: `allow_merge_commit` and `delete_branch_on_merge` are `true`;
  `allow_squash_merge` and `allow_rebase_merge` are `false`.
- **On failure**: HTTP 403 or 404 means missing permissions; see [Escalation](#escalation).

### Step 3: Check whether the plan offers branch protection

- **Manual action**: look at the second response from Step 1.
- **Expected result**: one of two answers, both valid:
  - an existing protection or `Branch not protected`: the plan supports protection. Continue with
    Step 4 once the required stages have reported on an open pull request; a protection that requires
    checks no run reports blocks every merge, so during adoption Step 4 waits for the first passing
    run of the pull request that implements SPEC-0001;
  - "Upgrade to GitHub Pro or make this repository public to enable this feature.": the plan offers
    no protection for this repository. Checks stay informational (ADR-0003, rule 6); skip Step 4 and go
    to [Verification](#verification).

  During adoption, Verification runs in the pull request that implements SPEC-0001, where
  `ci/check-policy` exists.
- **On failure**: any other answer; see [Escalation](#escalation).

### Step 4: Protect `main`

- **Command**: set `contexts` to the display names of the required stages (SPEC-0001, Technical Design,
  section 2), plus any stages the project has made required.
  ```bash
  gh api -X PUT repos/<owner>/<repo>/branches/main/protection --input - <<'EOF'
  {
    "required_status_checks": {"strict": false, "contexts": ["Documentation", "Policy", "Summary"]},
    "enforce_admins": true,
    "required_pull_request_reviews": {"required_approving_review_count": 0},
    "restrictions": null,
    "required_linear_history": false,
    "allow_force_pushes": false,
    "allow_deletions": false
  }
  EOF
  ```
- **Expected result**: the protection JSON, with the given `contexts`, `enforce_admins.enabled` set to
  `true`, and `required_linear_history.enabled` set to `false` (the history has merge commits, per
  ADR-0002).
- **On failure**: HTTP 422 means the body is invalid; check it and retry once. If it still fails, see
  [Escalation](#escalation).

## Verification

- The merge settings match Step 2 and, with protection, the `contexts` match Step 4 and `admins` is
  `true`:
  ```bash
  gh api repos/<owner>/<repo> \
    --jq '{allow_merge_commit, allow_squash_merge, allow_rebase_merge, delete_branch_on_merge}'
  gh api repos/<owner>/<repo>/branches/main/protection \
    --jq '{contexts: .required_status_checks.contexts, admins: .enforce_admins.enabled}'
  ```
- On an open pull request, "Create a merge commit" is the only merge option and, with protection, the
  required stages are marked "Required".
- Locally, with an admin's credentials, `GITHUB_TOKEN=$(gh auth token) ci/check-policy`
  reports no `[ADR-0002]` violation and no `repository settings not checked` note.

## Rollback

Revert each state-changing step in reverse order:

1. Revert Step 4: if `main` had no protection before, run
   `gh api -X DELETE repos/<owner>/<repo>/branches/main/protection`; otherwise, repeat Step 4 with the JSON saved in
   Step 1.
2. Revert Step 2: repeat Step 2 with the values saved in Step 1.

## Escalation

- **Contacts**: the repository owner (the `owners` field).
- **When to Escalate**: no admin role; no branch protection on the plan for a project that needs merges
  blocked (a plan or visibility decision; not during adoption, where checks simply stay informational,
  as ADR-0003, rule 6, says); an API error that persists after one retry; a failed rollback.

## AI Agent Execution Rules

> This section tells AI agents which steps they may execute and when they must stop.

- **Safe to run without confirmation**: Steps 1 and 3 and [Verification](#verification), which only read.
- **Requires human confirmation**: Steps 2 and 4 and the rollback, which change repository rules for
  everyone. One confirmation before Step 2 covers Steps 2 and 4 of the same execution (see
  **Approvals**).
- **First execution**: during adoption, an agent carries out this Runbook while it is still `draft`;
  that execution is what makes it `active`
  ([ADR-0001](../adrs/0001-link-the-documentation-framework-to-github.md), rule 8).
- **Stop conditions**: missing merge fields in Step 1; any HTTP response other than the ones listed; a
  pull request being merged while Step 2 or Step 4 runs; Step 4 before the required stages have
  reported on an open pull request (Step 3).
