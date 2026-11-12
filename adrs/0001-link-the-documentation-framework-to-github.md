---
# ============================================================================
# Managed section — preserve the document metadata when updating this file
# ============================================================================
id: ADR-0001
type: adr
title: "Link the documentation framework to GitHub"
status: proposed
created_at: 2026-11-11
updated_at: 2026-11-12
superseded_by: ""
# ============================================================================
deciders:
  - "@sjgarani"
supersedes: []
related_rfcs: []
tags: [documentation, github, traceability]
---

# ADR-0001: Link the documentation framework to GitHub

## Context and Problem Statement

The documentation framework defines five document types (PRD, RFC, ADR, Spec, and Runbook), each with
its own lifecycle, naming rule, and traceability fields. It says that statuses change "through the
project's review workflow" and that documents link to "work items and review records", but it never
names a tool. Until supporting tools exist, every project applies the templates by hand.

For a project hosted on GitHub, that gap means each team settles the same questions again: where the
documents live, what counts as a review, how a pull request points back to the Spec it implements, and
how owners are written down. Left unsettled, those answers tend to scatter across ADRs, Specs, Runbooks,
and AI agent instructions, repeated in several places and sometimes contradicting each other.

This ADR defines the link between the framework and GitHub once, in terms that hold for any project that
adopts doc2gh. The branching and merge workflow lives in [ADR-0002](0002-branching-and-merge-workflow.md),
and the CI platform in [ADR-0003](0003-use-github-actions-for-ci.md).

## Decision Drivers

- Documents and code are reviewed and versioned together, in one history.
- Every status change can be traced to a review.
- Anyone can get from code to its document, and back, using the document ID alone.
- People, AI agents, and CI can all check the rules.
- Nothing beyond what GitHub offers on every plan, including private repositories on the free plan.

## Considered Options

1. **Documents in the repository, pull requests as reviews, and Spec tasks as work items**
2. **Documents in the repository, with GitHub Issues and Projects as mandatory work items**
3. **Documents outside the repository (GitHub wiki or an external tool)**

## Pros and Cons of the Options

### Option 1: Documents in the repository, pull requests as reviews, Spec tasks as work items

- **Pros**:
  - A document and the code it describes change in the same pull request, and one merge records the
    review of both.
  - Git history shows who changed each document's status, when, and in which review.
  - The Task Breakdown section of every Spec is already a checklist that people and AI agents track.
  - CI validates documents the same way it validates code.
- **Cons**:
  - No built-in board for progress; a team that wants one has to build it from the Specs.
  - Requests and bugs that are not yet a document have no home of their own.

### Option 2: Issues and Projects as mandatory work items

- **Pros**:
  - GitHub boards, filters, and notifications for tracking work.
- **Cons**:
  - Two task lists to keep in sync, the Spec's and the issues', and they drift apart.
  - Status lives in two places: the document and the board.

### Option 3: Documents outside the repository

- **Pros**:
  - Edits skip the pull request, which is faster for rough drafts.
- **Cons**:
  - Wiki edits bypass pull requests and CI, so status changes go unreviewed.
  - Documents and code are no longer versioned together.

## Decision Outcome

**Chosen option**: "Documents in the repository, pull requests as reviews, and Spec tasks as work
items", because it puts documents and code through the same review and into the same history without
relying on any paid feature. Issues remain available as an intake channel (rule 3) without duplicating
the task list.

Rules:
1. Documents live in `docs/<type>/`, in the project repository and in the same history as the code, and
   are written in English.
2. The project's review workflow is a pull request to `main` (ADR-0002). New documents, content changes,
   and status changes all go through a pull request, and the merge records the review. No document
   reaches an approval status (`approved`, `accepted`, `ready`, or `active`) without one. The pull
   request that implements a Spec, whether or not it also writes it, carries the status the work ends
   with: `done` when every task and criterion is met, `in-progress` otherwise; a Spec implemented in one
   pull request needs no other pull request for its status. A task is ticked in the pull request that
   brings its code to `main`; an acceptance criterion, once it has been verified the way the Spec's test
   plan describes.
3. Work items are the tasks in each Spec's Task Breakdown section. GitHub Issues may capture requests and
   bugs that have no document yet. Once an issue turns into work, that work is described in a document,
   the issue references the document's ID, and the pull request that finishes the work closes the issue
   (`Closes #<number>` in its description).
4. Every pull request lists in its description the IDs of the documents it creates, changes, or
   implements, along with the requirement or task when there is one (for example, "SPEC-0001, Task 2").
   The title may end with the main ID in parentheses.
5. Owners (`deciders`, `owners`) and escalation contacts are GitHub users, written as `@login`.
6. Documents refer to each other by ID: in the `related_*`, `supersedes`, and `superseded_by` fields, and
   with a relative link in the text. Each definition is written once, in the document that owns it;
   other documents cite it by ID and rule or requirement number (for example, "ADR-0003, rule 6")
   instead of restating it. When the type's front matter has no field for a reference (a Spec that
   depends on another Spec, for example), the citation lives in the text only.
7. CI checks the shape of every document (file name, `id`, fields, status, owners, references, and
   links) in the documentation stage of [SPEC-0001](../specs/0001-ci-pipeline.md). ADR rules that can be
   automated become checks, listed in SPEC-0001, and each violation names the ADR it comes from; each
   ADR's compliance section says which of its rules are left to review, and
   [RUNBOOK-0002](../runbooks/0002-review-a-pull-request-against-the-accepted-adrs.md) is how that review is done.
8. The documents that ship with doc2gh (ADR-0001 to ADR-0003, SPEC-0001, RUNBOOK-0001, and
   RUNBOOK-0002) become the project's own documents on adoption, which takes two pull requests:
   - the adoption pull request brings doc2gh in, accepts the ADRs, moves SPEC-0001 to `ready`, puts the
     project's people in `deciders` and `owners`, and adds the project's AI agent instruction file. Its
     merge approves them, and no other work starts before it;
   - the implementation pull request builds SPEC-0001, finishes the first execution of RUNBOOK-0001,
     and is the first pull request reviewed with RUNBOOK-0002. Every task and criterion of SPEC-0001
     is verified in it, so it leaves SPEC-0001 `done` (rule 2) and both Runbooks `active`.

   After the second merge, every one of these documents is final.
   An AI agent may carry out the first execution of either Runbook while it is still `draft`,
   following its AI Agent Execution Rules, since that execution is what makes it `active`. From then
   on, the project keeps these documents current through pull requests, like any other document.

How the framework maps to GitHub:

| Framework concept | On GitHub | Defined in |
| :--- | :--- | :--- |
| Where documents live | `docs/` in the project repository | Rule 1 |
| Review workflow and status changes | Pull request to `main`, merged with a merge commit | Rule 2; ADR-0002 |
| Work item | Spec task; optional issue that references the ID | Rule 3 |
| Review record | Pull request that lists the IDs | Rule 4 |
| Review of the rules left to review | Review comment on the pull request | Rule 7; RUNBOOK-0002 |
| Owners and contacts | `@login` user | Rule 5 |
| Document and rule checks | GitHub Actions workflow | Rule 7; ADR-0003; SPEC-0001 |
| Merge settings and `main` protection | Repository settings and branch protection | ADR-0002; ADR-0003; RUNBOOK-0001 |
| Documents that ship with doc2gh | The project's own documents, adopted in two pull requests | Rule 8 |

## Consequences

### Positive
- Every project that adopts doc2gh starts with the review workflow, traceability, and checks already
  defined, instead of writing them from scratch.
- Git history answers who approved each decision and in which pull request.
- AI agents find each rule in exactly one document and cite it by ID.

### Negative / Trade-offs
- There is no mandatory progress board; progress is read from the Specs or from a board the project
  builds itself.
- doc2gh's own documents take the first numbers (ADR-0001 to ADR-0003, SPEC-0001, RUNBOOK-0001, and
  RUNBOOK-0002). A project that already uses those numbers has to renumber its documents when it adopts
  doc2gh, and a new doc2gh document may later take a number the project already uses.
- Project documents share `docs/` with the framework files, and the project edits the documents that
  ship with doc2gh (rule 8), so a new doc2gh version may bring conflicts to resolve.

## Compliance & Validation (AI & Code Review)

> This section instructs AI coding assistants and human reviewers on how to verify that repository code complies with this architectural decision.

- **Mandatory Patterns**: documents in `docs/<type>/`, created from the type's `template.md`, in
  English, with the next free ID; pull requests that list the IDs of the documents involved; owners
  written as `@login`; cross-references by ID.
- **Forbidden Patterns**: changing a document's status outside a pull request; project documents outside
  `docs/` or in the wiki; the same definition written in two documents.
- **Automated Verification**: the documentation stage of SPEC-0001 (FR-02) checks owners (rule 5), that
  references exist (rule 6), and the shape of every document (rule 7). Left to review: rules 1, 2, 3, 4,
  and 8, and, in rule 6, that each definition is written once.
