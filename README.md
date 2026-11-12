# doc2gh

Reusable documentation framework for software projects, wired into GitHub. It provides a
consistent way to plan, record, and track software work, and it defines how that work flows
through GitHub: pull requests as the review workflow, GitHub Actions as the checks, and the
repository settings that keep both honest.

Add this repository to another project with `git subtree` to keep the shared
documentation structure and practices in one place and sync them into each
project that uses them.

## Goals

- Provide a consistent structure for technical documentation.
- Capture product requirements, user problems, and success metrics before technical design.
- Record architectural decisions and their consequences.
- Gather feedback on proposed changes before implementation.
- Turn requirements and decisions into actionable specifications.
- Document operational procedures that people and AI agents can execute safely.
- Maintain traceability between related documents and implementation work.
- Support planning, progress tracking, and review.

## Structure

- [`prds/`](prds/): Product Requirements Documents that define user problems, goals, success metrics, and scope.
- [`rfcs/`](rfcs/): Requests for Comments that invite discussion on larger proposals.
- [`adrs/`](adrs/): Architecture Decision Records that explain important technical choices.
- [`specs/`](specs/): Technical and functional specifications that guide implementation.
- [`runbooks/`](runbooks/): Step-by-step procedures for recurring operations, maintenance, and incident response.

Each directory includes a `README.md` and a starter template. Keep document
IDs, status, and links to related work up to date.

## GitHub integration

doc2gh ships with the documents that tie the framework to GitHub. A project that adopts
doc2gh starts with them as its first ADRs, Spec, and Runbooks; from then on they are the
project's own documents, which it accepts, implements, and keeps up to date
([ADR-0001](adrs/0001-link-the-documentation-framework-to-github.md), rule 8):

| Document | What it settles |
| :--- | :--- |
| [ADR-0001](adrs/0001-link-the-documentation-framework-to-github.md) | Where documents live, pull requests as the review workflow, Spec tasks as work items, `@login` owners, and cross-references by ID. |
| [ADR-0002](adrs/0002-branching-and-merge-workflow.md) | `main` as the only long-lived branch, branch naming, merge commits only, and the matching repository settings. |
| [ADR-0003](adrs/0003-use-github-actions-for-ci.md) | GitHub Actions on hosted runners, everything CI-related in `ci/`, workflow security rules, required checks, and what happens without branch protection. |
| [SPEC-0001](specs/0001-ci-pipeline.md) | The CI pipeline: documentation, policy, and summary stages, plus extension points for project stages. |
| [RUNBOOK-0001](runbooks/0001-configure-the-github-repository.md) | Applying the merge settings and protecting `main`. |
| [RUNBOOK-0002](runbooks/0002-diagnose-and-fix-ci-pipeline-failures.md) | Tracing a CI failure back to its rule and fixing it. |

## Managing `docs/` in a project

This section is the only place that describes how doc2gh is added to a project and kept up
to date. doc2gh lives in the project's `docs/` directory as a git subtree, because the
SPEC-0001 checks expect the document types in `docs/<type>/`.

### Rules

- Pick the branch for your language, `main` for English or `pt_BR` for Brazilian Portuguese,
  and keep using it for every update.
- Always pass `--squash`, so each adoption or update adds a single commit of doc2gh content
  to the project's history.
- Each adoption or update gets a pull request of its own, from a `chore/` branch, holding only
  the doc2gh commits and the adjustments they force on the project's documents, in separate
  commits ([ADR-0002](adrs/0002-branching-and-merge-workflow.md), rules 2 and 6).
- Merge that pull request with a merge commit (ADR-0002, rule 3) and never rebase its branch:
  `git subtree pull` relies on the commits the previous pull created, and rewriting them breaks
  the next update.
- Record the branch in the merge commit subject and the URL in a `Source:` line, as the
  commands below do, so the next update knows where to pull from.

### First adoption

```bash
git switch main
git pull --ff-only origin main        # skip in a repository without an origin remote
git switch -c chore/add-docs-framework
git subtree add --prefix=docs https://github.com/sjgarani/doc2gh.git main --squash \
  -m $'Add the documentation framework (main)\n\nSource: https://github.com/sjgarani/doc2gh.git'
git push -u origin chore/add-docs-framework
gh pr create --base main --title "Add the documentation framework" \
  --body "Adds doc2gh (main) to docs/: ADR-0001 to ADR-0003, SPEC-0001, RUNBOOK-0001, RUNBOOK-0002."
gh pr merge <pr-number> --merge --delete-branch
```

`git subtree` refuses to run with uncommitted changes. If it reports
`prefix 'docs' already exists`, the project already has a `docs/` directory that is not a
subtree; migrate it first, as a separate piece of work.

The documents that ship with doc2gh arrive as `proposed` and `draft`. Accepting the ADRs and
implementing SPEC-0001 come next, in pull requests of their own (ADR-0001, rule 8).

### Updating

```bash
git log -1 --grep '^Source: ' --format=%B   # branch (in the subject) and URL of the last update
git switch main
git pull --ff-only origin main        # skip in a repository without an origin remote
git switch -c chore/update-docs-framework
git subtree pull --prefix=docs <url> <branch> --squash \
  -m $'Update the documentation framework (<branch>)\n\nSource: <url>'
git diff --stat HEAD^1 HEAD -- docs/        # what the update brought in, plus conflict fixes
git diff HEAD^1 HEAD -- docs/adrs/          # changes to ADR rules
ci/check-docs
```

Then open the pull request, titled "Update the documentation framework" and listing in its
description the IDs of the documents the update changed (ADR-0001, rule 4), and merge it with
a merge commit, as on adoption. A project adopted before the `Source:` line existed asks the
repository owner for the URL and branch, and records them on that update.

If the pull stops on a conflict, resolve it file by file, then run
`git add docs && git commit --no-edit --cleanup=strip`:

- framework files (`README.md`, `template.md`): take doc2gh's version;
- documents that ship with doc2gh: keep the project's front matter and take doc2gh's new text.
  Carry ticks over by content, not by number: a task or criterion whose text changed goes back
  to `[ ]` until a later pull request checks it again (ADR-0001, rule 2).

If a conflict does not fit these rules, run `git merge --abort`.

Stop and ask the repository owner when:

- the update changes a rule of an ADR the project has accepted; an accepted decision changes
  only through a new ADR that supersedes it;
- `ci/check-docs` reports a duplicated number because a new doc2gh document took a number the
  project already uses; renumber the project's document, and every reference to it, in a pull
  request merged first;
- a conflict does not fit the rules above;
- an update pull request was squash- or rebase-merged by mistake.

### Checking and undoing

- `git log -1 --grep 'git-subtree-dir: docs' --format=%B main` shows
  `git-subtree-split: <sha>`, the doc2gh commit the project is on.
- Before the merge, close the pull request (`gh pr close <pr-number> --delete-branch`). After
  it, revert the merge in a new pull request (`git revert -m 1 <merge-commit>`); a later pull
  of the same doc2gh version will not bring the reverted changes back.

AI agents may run the read-only commands (`git status`, `git log`, `git diff`, and
`ci/check-docs`) on their own. `git subtree add` and `git subtree pull`, conflict resolution,
commits, pushes, and opening, merging, or reverting pull requests need a person's
confirmation.

## Recommended Workflow

1. Write a PRD when defining a product capability, user problem, and success metrics.
2. Write an RFC when a technical change needs broad discussion.
3. Once the proposal is approved, record lasting architectural choices as ADRs.
4. Turn the work into Specs with clear, verifiable acceptance criteria.
5. When a Spec introduces or changes an operational procedure, write or update
   the matching Runbook and link the two documents.
6. Link each document to its upstream documents, work items, and review records.
7. Keep statuses current as work progresses.

Each section's README and template define its statuses, naming rules, and
traceability fields. On GitHub, the review workflow is a pull request to `main`, and every
status change goes through one ([ADR-0001](adrs/0001-link-the-documentation-framework-to-github.md)).

## Project Status

This repository provides the documentation framework and the documents that link it to
GitHub. It ships documents only: each adopting project builds the CI scripts and workflow
from [SPEC-0001](specs/0001-ci-pipeline.md) in its own `ci/` directory. Until it does, follow
the templates and the GitHub documents by hand.

## License

Released under the [MIT License](LICENSE).
