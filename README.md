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
- Keep project knowledge compiled and current in a wiki maintained by AI agents.
- Maintain traceability between related documents and implementation work.
- Support planning, progress tracking, and review.

## Structure

- [`prds/`](prds/): Product Requirements Documents that define user problems, goals, success metrics, and scope.
- [`rfcs/`](rfcs/): Requests for Comments that invite discussion on larger proposals.
- [`adrs/`](adrs/): Architecture Decision Records that explain important technical choices.
- [`specs/`](specs/): Technical and functional specifications that guide implementation.
- [`runbooks/`](runbooks/): Step-by-step procedures for recurring operations, maintenance, and incident response.
- [`wiki/`](wiki/): LLM Wiki that compiles the project's sources and documents into interlinked pages that an AI agent maintains.

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
| [RUNBOOK-0002](runbooks/0002-review-a-pull-request-against-the-accepted-adrs.md) | Reviewing a pull request against the accepted ADRs, for the rules no CI stage checks. |

## Managing `docs/` in a project

This section is the only place that describes how doc2gh is added to a project and kept up
to date. doc2gh lives in the project's `docs/` directory as a git subtree, because the
SPEC-0001 checks expect the document types in `docs/<type>/`. `docs/LICENSE` is doc2gh's
license: it covers the files that come from doc2gh, not the project.

### Rules

- Adopt and update from doc2gh's `main` branch.
- Always pass `--squash`, so each adoption or update adds a single commit of doc2gh content
  to the project's history.
- Each adoption or update gets a pull request of its own, from a `chore/` branch, holding only
  the doc2gh commits and the adjustments they force on the project's documents, in separate
  commits ([ADR-0002](adrs/0002-branching-and-merge-workflow.md), rules 2 and 6). The adoption
  pull request also accepts the documents and adds the AI agent instruction file
  ([First adoption](#first-adoption)).
- Merge that pull request with a merge commit (ADR-0002, rule 3) and never rebase its branch:
  `git subtree pull` relies on the commits the previous pull created, and rewriting them breaks
  the next update.
- Record the branch in the merge commit subject and the URL in a `Source:` line, as the
  commands below do, so the next update knows where to pull from.

### First adoption

Adoption takes two pull requests, and the person adopting doc2gh only has to merge them
([ADR-0001](adrs/0001-link-the-documentation-framework-to-github.md), rule 8). It needs `gh`
authenticated (`gh auth status`) as an admin of the repository, and `main` on GitHub with at
least one commit. A new project starts with `git init -b main` and a first commit (a
`README.md`, for example), then creates the repository with
`gh repo create <owner>/<repo> --private --source . --push` (or `--public`); that first commit
is the only direct push to `main` (ADR-0002, rule 8).

1. Bring doc2gh in, on a local branch:

   ```bash
   git switch main
   git pull --ff-only origin main
   git switch -c chore/adopt-docs-framework
   git subtree add --prefix=docs https://github.com/sjgarani/doc2gh.git main --squash \
     -m $'Add the documentation framework (main)\n\nSource: https://github.com/sjgarani/doc2gh.git'
   ```

2. Apply the merge settings with Steps 1 to 3 of
   [RUNBOOK-0001](runbooks/0001-configure-the-github-repository.md), now in `docs/runbooks/`, so
   the adoption pull request can only be merged with a merge commit. The values that Step 1
   records go into the adoption pull request's description, for the rollback. Step 4 and the
   Runbook's Verification wait for the implementation pull request (step 5 below).
3. Add two commits on the same branch:
   - **Accept the documents**: ADR-0001 to ADR-0003 go to `accepted` and SPEC-0001 to `ready`.
     In those four documents and in both Runbooks, `deciders` and `owners` hold the adopting
     person's login (`gh api user --jq .login`) unless they named other people, `updated_at`
     is today's date, and `created_at` keeps doc2gh's date. The Runbooks stay `draft`.
   - **Add the AI agent instruction file** ([AI agent instructions](#ai-agent-instructions)).
4. Open the adoption pull request, replacing the two placeholders with the outputs of
   RUNBOOK-0001, Step 1:

   ```bash
   git push -u origin chore/adopt-docs-framework
   gh pr create --base main --title "Adopt the documentation framework" --body-file - <<'EOF'
   Adds doc2gh (main) to docs/, accepts ADR-0001 to ADR-0003, moves SPEC-0001 to ready, updates
   the front matter of RUNBOOK-0001 and RUNBOOK-0002, and adds the AI agent instruction file
   (ADR-0001, rule 8).

   RUNBOOK-0001, Steps 1 to 3, ran before this pull request. For the rollback, the settings
   before Step 2 were `<merge settings JSON>`, and the protection of `main` answered
   <protection JSON | Branch not protected | the "Upgrade to GitHub Pro" message>.
   EOF
   ```

   The person reviews it and merges it with a merge commit
   (`gh pr merge <pr-number> --merge --delete-branch`). That merge approves the documents.
5. From a `ci/ci-pipeline` branch cut from the updated `main`, open the implementation pull
   request as a draft (`gh pr create --draft --title "Build the CI pipeline (SPEC-0001)"`). It
   builds [SPEC-0001](specs/0001-ci-pipeline.md), protects `main` once its first run passes
   (SPEC-0001, Task 5), runs the Verification of RUNBOOK-0001, and verifies every criterion in
   its own runs. Each task is ticked in the commit that completes it. Once the work is done, the
   pull request is reviewed with [RUNBOOK-0002](runbooks/0002-review-a-pull-request-against-the-accepted-adrs.md),
   by an agent that did not write it when the tool allows, and the "must fix" findings are fixed
   in new commits. The last commit ticks the acceptance criteria and moves SPEC-0001 to `done` and
   RUNBOOK-0001 and RUNBOOK-0002 to `active`; once that commit's run passes, the pull request
   leaves draft (`gh pr ready <pr-number>`), and the person merges it the same way. Its
   description lists each task and criterion with its evidence, the result of RUNBOOK-0001's
   Verification, the NFR-01 measurements (SPEC-0001, Task 6), and a link to the review
   (ADR-0001, rule 4).

After the second merge, adoption is complete: every document that ships with doc2gh is final
(the ADRs `accepted`, SPEC-0001 `done`, and both Runbooks `active`), and no pull request is
pending (ADR-0001, rule 8).

`git subtree` refuses to run with uncommitted changes. If it reports
`prefix 'docs' already exists`, the project already has a `docs/` directory that is not a
subtree; migrate it first, as a separate piece of work.

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
- documents that ship with doc2gh: keep the project's front matter and take doc2gh's new text,
  then re-apply the project's own changes to the body that the new text does not already cover.
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
commits, pushes, and opening or reverting pull requests need a person's confirmation. A request
to adopt or update doc2gh is that confirmation for every step of the procedure, except a
conflict that does not fit the rules above. A person merges every pull request.

## AI agent instructions

An AI agent asked to adopt doc2gh carries out [First adoption](#first-adoption) to the end. It
stops only:

- to ask for the owner and visibility of the GitHub repository, when it has to create one;
- when a pull request is ready for review, for the person to review and merge it: the adoption
  pull request as soon as it is open, the implementation pull request once it leaves draft. The
  agent follows the adoption pull request (`gh pr view <pr-number> --json state`) and goes on to
  the implementation pull request as soon as it is merged, without waiting for a new request.
  An agent that cannot keep running ends its turn with the pull request's link and, when it
  resumes, checks that the state is `MERGED` before going on;
- at the stop conditions of the documents it follows.

The agent writes the project's instruction file itself (the file its `/init` command would
write, such as `AGENTS.md` or `CLAUDE.md`) and puts it in the adoption pull request. Ahead of any
other guidance, the file records:

1. Approving the documentation is the first step. While any of ADR-0001 to ADR-0003 is
   `proposed` or SPEC-0001 is `draft`, the only work is the pull request that approves them
   ([ADR-0001](adrs/0001-link-the-documentation-framework-to-github.md), rule 8). Every later
   piece of work also starts from its document, following the
   [Recommended Workflow](#recommended-workflow).
2. The agent prepares pull requests; a person reviews and merges them, and the merge is the
   approval (ADR-0001, rule 2).

The file points to `docs/README.md` (this file) and cites documents by ID instead of copying
their rules (ADR-0001, rule 6). It records no status, ticked task, remote, or anything else that
changes as the work moves on, so it never needs a pull request of its own to stay correct.

## Recommended Workflow

1. Write a PRD when defining a product capability, user problem, and success metrics.
2. Write an RFC when a technical change needs broad discussion.
3. Once the proposal is approved, record lasting architectural choices as ADRs.
4. Turn the work into Specs with clear, verifiable acceptance criteria.
5. When a Spec introduces or changes an operational procedure, write or update
   the matching Runbook and link the two documents.
6. Link each document to its upstream documents, work items, and review records.
7. Keep statuses current as work progresses.
8. Ingest new sources and finished documents into the Wiki so project knowledge stays current.

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
