---
# ============================================================================
# Managed section — preserve the document metadata when updating this file
# ============================================================================
id: ADR-0003
type: adr
title: "Use GitHub Actions for CI"
status: proposed
created_at: 2026-11-11
updated_at: 2026-11-12
superseded_by: ""
# ============================================================================
deciders:
  - "@sjgarani"
supersedes: []
related_rfcs: []
tags: [ci, github, automation, security]
---

# ADR-0003: Use GitHub Actions for CI

## Context and Problem Statement

[ADR-0001](0001-link-the-documentation-framework-to-github.md) (rule 7) requires CI to check the shape of
every document and every ADR rule that can be automated, and [ADR-0002](0002-branching-and-merge-workflow.md)
depends on repository settings that need checking too. Those checks need somewhere to run, in a project
whose repository and review workflow already live on GitHub.

What blocks a merge is GitHub's branch protection, not the CI platform. Some plans offer neither branch
protection nor rulesets for private repositories; the API answers "Upgrade to GitHub Pro or make this
repository public to enable this feature.". On those plans, any check can report its result on the pull
request but cannot stop the merge, so we also need to decide what the checks mean in that situation.

## Decision Drivers

- Integration with pull requests and branch protection (required status checks).
- Checks that behave the same on a developer machine and in CI.
- Least-privilege permissions and protection against code from forked pull requests.
- Cost: private repositories spend plan minutes on hosted runners.
- No infrastructure of our own to maintain.

## Considered Options

1. **GitHub Actions on GitHub-hosted runners**
2. **GitHub Actions on self-hosted runners**
3. **Another platform (for example, GitLab CI or Jenkins) integrated with GitHub**

## Pros and Cons of the Options

### Option 1: GitHub Actions, hosted runners

- **Pros**:
  - Native integration with pull requests, status checks, branch protection, and repository secrets.
  - Linux runners ready to use, with nothing to host.
- **Cons**:
  - Minutes are capped by the plan for private repositories.
  - Actions published by others are external code running inside the pipeline (supply-chain risk).

### Option 2: GitHub Actions, self-hosted runners

- **Pros**:
  - No minute limits; local caches survive between runs.
- **Cons**:
  - A server to maintain and secure, and a persistent runner on a repository that accepts outside code is
    a security risk.

### Option 3: Another platform

- **Pros**:
  - Independence from GitHub.
- **Cons**:
  - One more system to run and integrate, with nothing gained for checking documents and repository
    rules.

## Decision Outcome

**Chosen option**: "GitHub Actions on GitHub-hosted runners", because it plugs into pull requests and
branch protection without any infrastructure of our own. Self-hosted runners require a new ADR, should
minute costs or stage durations ever call for them.

Rules:
1. Everything that belongs to CI lives in the `ci/` directory at the project root: the check scripts,
   their shared code, and their tests (`ci/tests/`). The only exception is the workflow files, which
   GitHub requires in `.github/workflows/`; workflows only call the scripts in `ci/`, which run the same
   way locally.
2. Least privilege: every workflow declares `permissions` at the top level, set to `contents: read`. A
   job that needs more declares it on the job itself.
3. Every action from outside the repository is pinned to a full 40-character commit SHA, with the release
   tag in a trailing comment (`uses: <owner>/<action>@<sha> # <tag>`), and never to a moving ref such as
   `@main` or `@v4`.
   This includes `actions/*`. `docker://` images are pinned by digest (`@sha256:`). Local actions (`./`)
   need no pinning.
4. No secret is ever exposed to a pull request from a fork, and `pull_request_target` is forbidden in any
   workflow that checks out code. Steps that need a secret do not run for forks.
5. Every checkout sets `persist-credentials: false`, so the action's default token serves the fetch and
   is not kept. `GITHUB_TOKEN` is passed, as an environment variable, only to the step that calls the
   GitHub API.
6. A required stage is one that must pass before a merge; [SPEC-0001](../specs/0001-ci-pipeline.md)
   (FR-08) lists them. When the plan offers branch protection, the protection on `main` requires a pull
   request and lists the required stages as required status checks
   ([RUNBOOK-0001](../runbooks/0001-configure-the-github-repository.md)). When it does not, the checks
   are informational: they do not block the merge, whoever merges reads the summary job first, and a pull
   request with a failing required stage is not merged.
7. A failure that reaches `main`, whether through a merge with a failing required stage or a direct push,
   is fixed by a new pull request before anything else is merged.
8. CI never sees real user or customer data or production secrets; it uses test data only.

## Consequences

### Positive
- Every pull request gets checked without any server of our own.
- ADR rules are enforced, not just written down.
- The same script answers in CI and locally, which makes failures easier to diagnose.

### Negative / Trade-offs
- CI depends on GitHub.
- Minute usage needs watching; SPEC-0001 separates fast stages, which always run, from slow ones, which
  run only when code changes.
- Without branch protection, nothing technically stops a merge with a failing required stage or a direct
  push to `main`. Rule 6 relies on whoever merges, and the failure only shows up afterwards, in the run
  for the push to `main`.
- Pinning by SHA means updating actions by hand.

## Compliance & Validation (AI & Code Review)

> This section instructs AI coding assistants and human reviewers on how to verify that repository code complies with this architectural decision.

- **Mandatory Patterns**: top-level `permissions` in every workflow; every external action pinned by SHA
  with its version in a comment; `persist-credentials: false` on every checkout; CI scripts, shared code,
  and tests in `ci/`.
- **Forbidden Patterns**: `pull_request_target` together with a checkout; secrets in workflow files;
  actions referenced by a moving ref; merging a pull request with a failing required stage.
- **Automated Verification**: the policy stage of SPEC-0001 (FR-03) inspects the workflows themselves for
  rules 2 to 5 (for rule 4, workflows that use `actions/checkout`). Left to review: rules 1, 6, 7, and 8,
  and, in rule 4, other ways of checking out code.
