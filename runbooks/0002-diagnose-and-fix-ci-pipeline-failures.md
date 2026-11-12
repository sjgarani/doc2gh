---
# ============================================================================
# Managed section — preserve the document metadata when updating this file
# ============================================================================
id: RUNBOOK-0002
type: runbook
title: "Diagnose and fix CI pipeline failures"
status: draft
created_at: 2026-11-11
updated_at: 2026-11-12
last_verified_at: ""
# ============================================================================
owners:
  - "@sjgarani"
related_specs: [SPEC-0001]
related_adrs: [ADR-0001, ADR-0002, ADR-0003]
tags: [ci, github-actions, troubleshooting]
---

# RUNBOOK-0002: Diagnose and fix CI pipeline failures

## Purpose and Triggers

- **Purpose**: find the stage and the rule that failed in the [SPEC-0001](../specs/0001-ci-pipeline.md)
  pipeline, reproduce the failure locally, and fix it through a pull request.
- **Triggers**:
  - the **Summary** job on a pull request shows a required stage that did not pass;
  - the run for a push to `main`, the scheduled run, or a manual run failed;
  - a stage takes far longer than the NFR-01 targets in SPEC-0001.

## Prerequisites

- **Access & Permissions**: read access to the repository and its GitHub Actions runs; write access to
  re-run jobs and open pull requests.
- **Tools**: `git`, `gh` authenticated (`gh auth status`), and Python 3.12 or later.
- **Approvals**: none to diagnose; fixes follow the
  [ADR-0002](../adrs/0002-branching-and-merge-workflow.md) workflow.
- **Credentials**: only the `gh` login. Never paste tokens into commands, logs, or pull requests.

## Risk and Impact

- **Service Impact**: none; CI deploys nothing. While checks are informational
  ([ADR-0003](../adrs/0003-use-github-actions-for-ci.md), rule 6), a failure does not block the merge, so
  the real risk is merging with a required stage still failing.
- **Data Risk**: only if a real secret shows up in a committed file or in a log; treat that secret as
  leaked.
- **Irreversible Steps**: none in this procedure. Rewriting `main` history to purge a secret is outside
  its scope and needs a decision of its own.
- **Maintenance Window**: any time.

## Procedure

Execute the steps in order. If a result differs from the **Expected result**, stop and follow **On failure**. Omit **Command** for manual actions.

### Step 1: Find the run and the failing stage

- **Command**:
  ```bash
  gh pr checks <pr-number>                              # on a pull request
  gh run list --workflow ci.yml --branch main --limit 5 # on main, scheduled, or manual
  gh run view <run-id>
  ```
- **Expected result**: the name of the failing job (`Documentation`, `Policy`, `Summary`, `Changes`
  where the project has it, or a project stage). The **Summary** job's table lists every stage by job ID (`changes`, `docs`,
  `policy`, and so on), whether it is required, and its result.
- **On failure**: if there is no run for the commit (the workflow never triggered), check that the pull
  request targets `main` and that `.github/workflows/ci.yml` exists on the branch; see
  [Escalation](#escalation).

### Step 2: Read the violations

- **Command**:
  ```bash
  gh run view <run-id> --log-failed
  ```
- **Expected result**: lines formatted as `[<source document>] <location>[:<line>]: <message>`, where the
  location is a file, `repos/<owner>/<repo>`, or `branch <name>` (SPEC-0001, Technical Design,
  section 2). The source document tells you which rule applies. Exit code `1` means a violation; `2`
  means the script itself failed to run.
- **On failure**: a failing test in the **Policy** job's `unittest` step has no such line; go to Step 3
  and use the `ci/tests/` row of Step 4. Otherwise, if no line has that format, the failure is outside
  the scripts (checkout, network, or GitHub infrastructure); go to Step 5.

### Step 3: Reproduce locally

- **Command**:
  ```bash
  git fetch origin && git switch <pr-branch>
  ci/check-docs
  ci/check-policy
  GITHUB_EVENT_NAME=pull_request GITHUB_HEAD_REF=<pr-branch> ci/check-policy --offline  # branch name
  python3 -m unittest discover --start-directory ci/tests
  ```
- **Expected result**: the same violations as in CI (SPEC-0001, AC-08). The branch name is checked only
  on a `pull_request` event (SPEC-0001, FR-10), hence the second `check-policy` run.
- **On failure**: a different result points to local state that differs from the commit (untracked
  files, a submodule on another commit) or to an environment difference; note it and, if nothing
  explains it, see [Escalation](#escalation).

### Step 4: Fix according to the rule

- **Manual action**: fix the problem on the pull request branch, or on a new branch cut from `main` if the
  failure is on `main`, then repeat Step 3. Fixes by source:

| Source | Typical message | Fix |
| :--- | :--- | :--- |
| SPEC-0001 (`check-docs`) | bad file name, `id` not matching the file, invalid status, missing field, H1, unknown reference, broken link | Bring the document in line with its type's `template.md` and `README.md` under `docs/`. |
| SPEC-0001 (`check-policy`, workflows) | step or `with:` written as a flow mapping, file not valid UTF-8 | Write steps and `with:` blocks in block style, in UTF-8. |
| ADR-0003 (`check-policy`, workflows) | `permissions` other than `contents: read`, action not pinned by SHA or without a version comment, `pull_request_target` with a checkout, checkout without `persist-credentials: false` | Set top-level `permissions` to `contents: read`; pin the action to the SHA of its latest tag (`gh api repos/<owner>/<action>/releases/latest --jq .tag_name`, then `gh api repos/<owner>/<action>/commits/<tag> --jq .sha`) with the tag in a comment; drop `pull_request_target`; add `persist-credentials: false`. |
| ADR-0002 (`check-policy`, settings) | merge settings mismatch | Follow [RUNBOOK-0001](0001-configure-the-github-repository.md). |
| ADR-0002 (`check-policy`, branch) | branch name outside `<type>/<short-topic>` | Rename the branch (`gh api -X POST repos/<owner>/<repo>/branches/<branch>/rename -f new_name=<type>/<short-topic>`), which updates the open pull request. |
| `check-policy` (exit `2`) | cannot tell which GitHub repository to read | Locally, set `GITHUB_REPOSITORY=<owner>/<repo>` or run with `--offline`; in CI, GitHub always sets `GITHUB_REPOSITORY`. |
| Project ADR or Spec | a project-specific rule or project stage | Fix as the cited document describes. |
| API error (exit `2`) | GitHub API call failed | Check the rate limit (`gh api rate_limit --jq .resources.core`) and GitHub's status; re-run in Step 5. |
| `Changes` job, where it exists | commits could not be compared | The job runs the slow stages to be safe; check `fetch-depth: 0` on that job's checkout. |
| `ci/tests/` tests | failing test | A bug in the `ci/` scripts: fix the script, not the test, unless the test itself is wrong. |

- **Expected result**: Step 3 reports no violations.
- **On failure**: if the fix means changing a rule from an `accepted` ADR, stop: that change needs a new
  ADR (see [Escalation](#escalation)).

### Step 5: Re-run, if the failure is not in the code

- **Command**:
  ```bash
  gh run rerun <run-id> --failed
  ```
- **Expected result**: the re-run jobs pass. Use this only for failures outside the code (API calls,
  infrastructure), and at most once per failure (SPEC-0001, Technical Design, section 3).
- **On failure**: [Escalation](#escalation).

### Step 6: Publish the fix

- **Manual action**: commit and push to the pull request branch, or open a new pull request to `main` if
  the failure was on `main` (ADR-0002). A failure that reached `main` is fixed before anything else is
  merged (ADR-0003, rule 7).
- **Expected result**: a new run whose **Summary** job ends with "All required stages passed.".
- **On failure**: go back to Step 2 with the new run.

## Verification

- `gh pr checks <pr-number>` (or `gh run list --workflow ci.yml --branch main --limit 1`) shows every job
  passing.
- The **Summary** job's table shows every required stage as passed.
- The run stays within the SPEC-0001 NFR-01 targets
  (`gh run view <run-id> --json createdAt,updatedAt,jobs`).

## Rollback

This procedure only changes the repository through pull requests:

1. A fix that made things worse and is not merged yet: close the pull request or revert the commit on
   the branch (`git revert <commit>`).
2. A merge to `main` that broke CI with no quick fix in sight: open a pull request that reverts the merge
   (`git revert -m 1 <merge-commit>` on a branch cut from `main`) and merge it with a merge commit.

## Escalation

- **Contacts**: the repository owner (the `owners` field).
- **When to Escalate**: a real secret was detected; the fix requires changing a rule from an `accepted`
  ADR; a failure that does not reproduce locally and is not explained by local state; a GitHub
  infrastructure failure that persists after one re-run; a workflow that does not trigger.

## AI Agent Execution Rules

> This section tells AI agents which steps they may execute and when they must stop.

- **Safe to run without confirmation**: Steps 1, 2, and 3 (read-only, plus running the scripts locally),
  and editing files on the local branch to fix the violations from Step 4.
- **Requires human confirmation**: Step 5 (re-running jobs), Step 6 (commit, push, pull request, merge),
  revoking or rotating credentials, any change to `.github/workflows/` or to the rules in
  `ci/`, and the rollback. On the pull request that implements SPEC-0001 during adoption, the request to
  adopt doc2gh is that confirmation for everything except the merge, which a person does.
- **First execution**: an agent may carry out this Runbook while it is still `draft`; the pull request
  that fixes the failure moves it to `active`
  ([ADR-0001](../adrs/0001-link-the-documentation-framework-to-github.md), rule 8).
- **Stop conditions**: a real secret was detected (alert the owner without repeating its value; the
  credential is revoked or rotated before anything else); a violation whose fix would change a rule from
  an `accepted` ADR; a local result that differs from CI with no explanation; a failure that persists
  after one re-run.
