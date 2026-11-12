---
# ============================================================================
# Managed section — preserve the document metadata when updating this file
# ============================================================================
id: ADR-0002
type: adr
title: "Branching and merge workflow"
status: proposed
created_at: 2026-11-11
updated_at: 2026-11-12
superseded_by: ""
# ============================================================================
deciders:
  - "@sjgarani"
supersedes: []
related_rfcs: []
tags: [git, github, workflow]
---

# ADR-0002: Branching and merge workflow

## Context and Problem Statement

[ADR-0001](0001-link-the-documentation-framework-to-github.md) makes a pull request to `main` the review
workflow for both documents and code. It does not say where branches start, how they are merged, or how
the repository should be configured to support that.

How a pull request is merged matters too. GitHub's "Squash and merge" and "Rebase and merge" rewrite the
pull request's commits, so what lands on `main` is not what was reviewed, and any commit SHA cited during
the review, or recorded by tools that rely on Git history, no longer points at anything on `main`.

Keeping more than one long-lived branch (`main` plus `devel`, for instance) causes other problems: every
merge means fast-forwarding the other branch by hand, and a branch cut from the integration branch drags
unpublished commits into its pull request.

## Decision Drivers

- Commits reach `main` exactly as they were reviewed, which means merge commits only.
- Little or no manual syncing between branches.
- A single branch for CI to check and, where the plan allows, to protect
  ([ADR-0003](0003-use-github-actions-for-ci.md)).
- Pull requests that show only their own changes.
- Repository settings that prevent the wrong kind of merge rather than merely discourage it.

## Considered Options

1. **`main` as the only long-lived branch; topic branches start from `main` and come back through a pull
   request merged with a merge commit**
2. **`devel` as an integration branch**: topic branches start from and return to `devel`, which is merged
   into `main` from time to time
3. **Full Git Flow**: `devel`, `release/*`, and `hotfix/*`

## Pros and Cons of the Options

### Option 1: `main` only

- **Pros**:
  - No syncing between long-lived branches.
  - CI and branch protection focus on a single branch.
  - Every pull request shows only its own changes.
- **Cons**:
  - Every merge lands straight on the main branch; without branch protection, it is up to whoever merges
    to check the results (ADR-0003, rule 6).
  - Long-running work stays on a topic branch that has to be kept up to date with `main`.

### Option 2: `devel` as integration branch

- **Pros**:
  - Separates what has been integrated from what has been released.
- **Cons**:
  - Two levels of pull requests, and `devel` has to be synced with `main` at every release.
  - Branches cut from `devel` carry unreleased commits into pull requests against `main`.

### Option 3: Full Git Flow

- **Pros**:
  - A well-known model for products that maintain several versions in parallel.
- **Cons**:
  - More branches and ceremony than most projects need; a project that does ship parallel versions can
    adopt it in its own ADR.

## Decision Outcome

**Chosen option**: "`main` as the only long-lived branch, with topic branches and merge commits", because
it removes manual syncing, keeps CI and protection on one branch, and lands every commit on `main`
exactly as it was reviewed.

Rules:
1. `main` is the only long-lived branch.
2. Every change reaches `main` through a pull request from a branch cut from an up-to-date `main` and
   named `<type>/<short-topic>`, where `<short-topic>` is lowercase words joined by hyphens. Types:
   `docs` (documentation only), `feat` (a new capability, with its documents and tests), `fix` (a bug
   fix), `ci` (workflows and check scripts, including a project stage added on its own), and `chore` (maintenance, such as updating dependencies or
   anything brought in from another repository).
3. Pull requests are always merged with a merge commit. Repository settings: `allow_merge_commit` on,
   `allow_squash_merge` and `allow_rebase_merge` off.
4. Topic branches are deleted after the merge. Repository setting: `delete_branch_on_merge` on
   ("Automatically delete head branches").
5. To bring `main` into a topic branch, run `git merge main`. Never rebase a branch that has already been
   pushed.
6. Each update of something brought in from another repository gets a pull request of its own. Besides
   the update itself, that pull request carries only the adjustments the update forces on the project, in
   separate commits. The pull request that first brings doc2gh in also carries what
   [ADR-0001](0001-link-the-documentation-framework-to-github.md), rule 8, puts in it.
7. Commit messages and pull request titles and descriptions are written in English. A commit subject is
   imperative and at most 72 characters; the body, when there is one, explains why. Documentation follows
   the language of the adopted framework (ADR-0001, rule 1).
8. Pushing directly to `main` is forbidden. The only exception is the first commit of a new repository,
   which creates `main` so that pull requests have a base. When the GitHub plan offers branch protection,
   the protection on `main` requires a pull request (ADR-0003, rule 6).
9. Versioning and releases (tags, GitHub releases, deployments) are left to each project's own ADR.

[RUNBOOK-0001](../runbooks/0001-configure-the-github-repository.md) applies rules 3, 4, and 8 on GitHub
and checks them.

## Consequences

### Positive
- No more manual syncing between long-lived branches.
- CI and protection apply to a single branch.
- One merge commit per pull request, with the reviewed commits preserved underneath it.
- With the settings from rules 3 and 4, GitHub does not even offer the wrong merge button.

### Negative / Trade-offs
- With no intermediate branch, a bad merge lands directly on `main`, and the fix is a new pull request
  (ADR-0003, rule 7).
- Merge commits make the history less linear than squashing would.

## Compliance & Validation (AI & Code Review)

> This section instructs AI coding assistants and human reviewers on how to verify that repository code complies with this architectural decision.

- **Mandatory Patterns**: topic branches cut from `main` and named `<type>/<short-topic>`; pull requests
  to `main`; merge commits; updates of anything brought in from another repository in their own pull requests; commits and pull
  requests in English.
- **Forbidden Patterns**: squash or rebase merges; rebasing a branch that has already been pushed; other
  long-lived branches; direct pushes to `main`.
- **Automated Verification**: the policy stage of [SPEC-0001](../specs/0001-ci-pipeline.md) checks branch
  names on pull requests (FR-10) and reads the settings from rules 3 and 4 through the GitHub API (FR-04).
  RUNBOOK-0001 checks the protection required by rule 8; commit language and the remaining rules are left
  to review.
