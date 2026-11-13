---
# ============================================================================
# Managed section — preserve the document metadata when updating this file
# ============================================================================
id: RUNBOOK-0002
type: runbook
title: "Review a pull request against the accepted ADRs"
status: draft
created_at: 2026-11-12
updated_at: 2026-11-13
last_verified_at: ""
# ============================================================================
owners:
  - "@sjgarani"
related_specs: [SPEC-0001]
related_adrs: [ADR-0001, ADR-0002, ADR-0003]
tags: [review, pull-request, compliance]
---

# RUNBOOK-0002: Review a pull request against the accepted ADRs

## Purpose and Triggers

- **Purpose**: before a person merges a pull request, report every rule of an `accepted` ADR that the
  pull request breaks and that no CI stage checks, each with its location and fix, in a review comment
  on the pull request. The review informs the person; the merge stays theirs and remains the approval
  ([ADR-0001](../adrs/0001-link-the-documentation-framework-to-github.md), rule 2).
- **Triggers**:
  - a person asks for a review of a pull request;
  - the pull request that implements a Spec is about to leave draft, including the implementation pull
    request of adoption (ADR-0001, rule 8). Its last commit (the ticked criteria and the final statuses)
    and its description are already in place, so the review covers the head that will be merged;
  - new commits arrive on a pull request that was already reviewed. The new review covers the whole pull
    request again, at the new head, and says what became of the earlier findings.

## Prerequisites

- **Access & Permissions**: read access to the repository; a local clone with `origin` pointing to it.
- **Tools**: `git` and `gh`, authenticated (`gh auth status`).
- **Approvals**: none to read. The request for the review covers posting it (Step 5).
- **Credentials**: only the `gh` login.

Run the commands inside the clone. `<n>` is the pull request number and `<head>` the full SHA of its
head commit; `gh` fills in `{owner}` and `{repo}` from the clone's `origin`.

The review is worth most when it runs in a context that did not write the pull request, such as a new
agent session or a subagent, so it does not inherit the author's reasoning. When that is not possible,
the review says so. A review triggered by new commits may run in the context that did the previous
review.

## Risk and Impact

- **Service Impact**: none.
- **Data Risk**: none; the procedure only reads and posts a comment.
- **Irreversible Steps**: Step 5; a submitted review cannot be deleted, only edited (see
  [Rollback](#rollback)).
- **Maintenance Window**: any time the pull request is open.

## Procedure

Execute the steps in order. If a result differs from the **Expected result**, stop and follow **On failure**. Omit **Command** for manual actions.

### Step 1: Read the pull request

- **Command**:
  ```bash
  gh pr view <n> --json state,isDraft,baseRefName,headRefName,headRefOid,title,body,files
  gh pr diff <n>
  git fetch origin main "pull/<n>/head"
  git log --format='%h %s%n%b' origin/main..<head>
  gh api 'repos/{owner}/{repo}/issues/<n>/events' --paginate \
    --jq '.[] | select(.event == "head_ref_force_pushed") | .created_at'
  ```
- **Expected result**: `state` is `OPEN` and `baseRefName` is `main`. Note `headRefOid` as `<head>`; the
  files at the head are read with `git show <head>:<path>`. `git log` gives every commit's full subject
  and body (`gh pr view --json commits` cuts long subjects). The last command prints one line per force
  push to the pull request's branch, and nothing when there was none.
- **On failure**: a closed or merged pull request is not reviewed; stop. A base other than `main` is
  itself a finding (ADR-0002, rule 2); go on.

### Step 2: Collect the rules that apply

- **Command**:
  ```bash
  for f in $(git ls-tree --name-only origin/main docs/adrs/ | grep -E '/[0-9]{4}-[^/]+\.md$'); do
    git show "origin/main:$f" | grep -q '^status: accepted$' && echo "$f"
  done
  ```
- **Expected result**: the `accepted` ADRs on `main`. Read each one's rules (**Decision Outcome**) and
  its **Compliance & Validation** section, whose **Automated Verification** says which rules a CI stage
  checks and which are left to review; the review covers the latter. A pull request that changes an ADR is reviewed against
  the version on `main`. Also read every document whose ID the description lists or whose
  file the pull request changes, and the framework files whose rules apply: `docs/README.md`,
  the `README.md` of each document type the pull request touches, and
  `docs/wiki/README.md` when it changes the wiki.
- **On failure**: no accepted ADR on `main` means there is nothing to review against; post a review
  that says so (Step 5).

### Step 3: Check the CI result

- **Command**:
  ```bash
  gh pr checks <n>
  gh run list --commit <head> --json databaseId,event,conclusion
  gh api 'repos/{owner}/{repo}' \
    --jq '{allow_merge_commit, allow_squash_merge, allow_rebase_merge, delete_branch_on_merge}'
  ```
- **Expected result**: the `pull_request` run for `<head>` concluded `success`, so every required stage
  (the job IDs in the summary's `--required` list in `.github/workflows/ci.yml` at `<head>`) passed on
  it; `gh run view <run-id>` shows them. Those stages cover what each ADR's **Automated Verification**
  lists, so the review does not repeat those checks. When the policy stage printed
  `repository settings not checked`, the last command reads the settings of ADR-0002, rules 3 and 4,
  with the reviewer's own login; if it cannot read them either, the review says they were not checked.
  Checks from other apps, such as a secret scanner, are outside this review. `gh pr checks` exits
  non-zero while a check fails or is pending.
- **On failure**: a required stage that failed, is pending, or did not run on `<head>` is a "must fix"
  finding (ADR-0003, forbidden patterns); SPEC-0001, Technical Design, section 3 ("A run that fails")
  says how to fix it. Go on with Step 4, and also check by hand the rules that stage would have
  covered.

### Step 4: Check the rules left to review

- **Manual action**: check each rule from Step 2 against everything the pull request is made of: its
  title and description, its branch name, every commit's subject and body, the force pushes, the diff,
  and the files at `<head>`. For a pull request that implements a Spec, compare its ticked tasks and criteria and its
  final status with what the diff and the CI runs actually show (ADR-0001, rule 2):
  - repeat on `<head>` any local run the description gives as evidence (such as SPEC-0001, AC-08) when
    the tools allow, and say so when it was not repeated;
  - check a claim about another repository with `gh api`, such as an action pinned to its tag's SHA
    (`gh api repos/<owner>/<action>/commits/<tag> --jq .sha`, SPEC-0001, Task 4);
  - a decision the description attributes to a person is taken as recorded there (Escalation).

  Two rules concern `main` rather than the pull request. A commit on `main`'s first-parent line that is
  not a merge commit, other than the first one, is a direct push (ADR-0002, rule 8); report it as a
  note. ADR-0003, rule 7, applies to the pull request while the latest `push` run on `main` fails:
  ```bash
  git log --first-parent --no-merges --format='%h %ad %s' --date=short origin/main
  gh run list --branch main --event push --limit 1 --json conclusion,headSha
  ```
- **Expected result**: a list of findings. Each one names the rule ("ADR-0002, rule 7"; for a rule that
  lives in a framework file, the file and section, such as "`docs/adrs/README.md`, AI Agent Guidance"),
  the location (`<path>` or `<path>:<line>`, both at the head named in the review; `branch <name>`; a
  commit's short SHA; "title"; or "description"), what breaks the rule, and the fix. Something no rule
  backs, such as a style preference, is not a finding. Each finding is:
  - **must fix**: the pull request breaks the rule and can be fixed with new commits or an edit to its
    title or description;
  - **note**: the fix would break another rule (a pushed commit cannot be reworded without rewriting
    history, which ADR-0002, rule 5 forbids), or it is unclear whether the rule applies. A note of the
    second kind is escalated ([Escalation](#escalation)).
- **On failure**: when two rules contradict each other, or the pull request changes a rule of an
  accepted ADR without a new ADR that supersedes it, record a "must fix" finding and see
  [Escalation](#escalation).

### Step 5: Post the review

- **Command**:
  ```bash
  gh pr review <n> --comment --body-file - <<'EOF'
  ## Review against the accepted ADRs (RUNBOOK-0002)

  Head: <head, full SHA>. ADRs: <IDs from Step 2>. CI: <result from Step 3>.
  Reviewer: <a context that did not write the pull request | the context of the previous review | the author's own context>.

  ### Earlier findings
  - <link to the earlier review>: <finding>: <fixed in <short SHA> | still open | no longer applies, because ...>.

  ### Must fix
  - <rule>: <location>: <what breaks the rule>. Fix: <fix>.

  ### Notes
  - <rule>: <location>: <observation>.

  ### Checked without findings
  <rules checked that hold, by ID and number>
  EOF
  ```
  Leave out **Earlier findings** on a first review. Write "None." under any other empty heading. Mark a
  finding that needs escalation as "escalated" and mention
  its contacts (`@login`, see [Escalation](#escalation)) in it, unless the contact wrote the pull
  request.
- **Expected result**: the review appears on the pull request with the state `COMMENTED`. The review
  never approves or requests changes: GitHub refuses both on the author's own pull request, and the
  decision belongs to the person who merges.
- **On failure**: an API error; retry once, then see [Escalation](#escalation).

## Verification

- The last review on the pull request is this one, on the pull request's current head:
  ```bash
  gh api --paginate 'repos/{owner}/{repo}/pulls/<n>/reviews' \
    --jq '.[] | "\(.state) \(.commit_id) \(.html_url)"' | tail -n 1
  gh pr view <n> --json headRefOid --jq .headRefOid
  ```
  `state` is `COMMENTED`, `commit_id` equals the head, and `html_url` is the review's link, the one to
  report or cite. If the head moved, the new commits need a review of their own (see **Triggers**).

## Rollback

Revert each state-changing step in reverse order:

1. Revert Step 5: a submitted review cannot be deleted. Correct its body instead:
   `gh api -X PUT 'repos/{owner}/{repo}/pulls/<n>/reviews/<review-id>' -F body=@<file>`, with the ID
   from `gh api --paginate 'repos/{owner}/{repo}/pulls/<n>/reviews' --jq '.[].id' | tail -n 1`.

## Escalation

- **Contacts**: the `deciders` of the ADR concerned; otherwise, this Runbook's `owners`. When every
  contact wrote the pull request, the person who merges it decides.
- **How**: post the review anyway (Step 5), with the finding marked "escalated", and say so to whoever
  asked for the review. The review does not wait for the contacts' answer. The author records the
  answer in the pull request's description, with the login of the person who gave it and the date,
  and the next review takes it from there.
- **When to Escalate**: two rules that contradict each other; a pull request that changes a rule of an
  accepted ADR without superseding it; a doubt about whether a rule applies that the documents do not
  settle; an API error that persists after one retry.

## AI Agent Execution Rules

> This section tells AI agents which steps they may execute and when they must stop.

- **Safe to run without confirmation**: Steps 1 to 4 and [Verification](#verification), which only
  read.
- **Requires human confirmation**: Step 5 and the rollback, which post on the pull request. The request
  for the review is that confirmation; during adoption, the request to adopt doc2gh is.
- **Never**: approve, request changes, merge, or push to the pull request's branch. Findings go into the
  review; whoever wrote the pull request fixes them in it.
- **First execution**: an agent carries out this Runbook while it is still `draft`; that execution is
  what makes it `active` ([ADR-0001](../adrs/0001-link-the-documentation-framework-to-github.md), rule
  8).
- **Stop conditions**: the pull request is closed or merged; its head changes during the review (start
  again from Step 1); any API error after one retry.
