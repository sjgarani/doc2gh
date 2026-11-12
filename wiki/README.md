# LLM Wiki

This directory holds the project's **LLM Wiki**, a persistent, interlinked set of Markdown pages that an AI agent writes and maintains from the project's sources. It follows Andrej Karpathy's [LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). The agent compiles knowledge into the wiki once and keeps it current as new sources arrive, so it does not have to rediscover that knowledge from raw documents for every question. Humans curate sources, ask questions, and review. The agent writes the summaries and cross-references and keeps the pages consistent.

This README is the wiki's **schema**: the structure, conventions, and workflows the agent follows.

---

## Layers

| Layer | Path | Owner | Rule |
| :--- | :--- | :--- | :--- |
| Raw sources | `raw/` | Humans | Immutable. Humans choose what goes in, and an agent may copy in, unchanged, a file a human hands it. Agents never modify, rename, or delete them. |
| Wiki | `pages/`, `index.md`, `log.md` | AI agent, reviewed by humans | The agent creates and updates pages and keeps the index and log in sync. |
| Schema | This README and the project's agent instructions | Humans and AI agent | Co-evolved as the wiki grows. |

The project's PRDs, RFCs, ADRs, Specs, Runbooks, and code are also sources. The agent reads them where they live and does not copy them into `raw/`.

Wiki changes go through the project's review workflow like any other document change. Each request from a human (an ingest, a query filed back, or a lint pass) is reviewed as one change, with one commit per ingested source.

---

## Layout

```text
wiki/
├── README.md     # Schema (this file)
├── template.md   # Starter template for pages
├── index.md      # Catalog of every page, grouped by category
├── log.md        # Append-only record of ingests, queries, and lint passes
├── raw/          # Immutable source documents
└── pages/        # Agent-maintained pages
```

The first ingest creates `index.md`, `log.md`, `raw/`, and `pages/`. Store large binary sources according to the project's large-file policy (for example, Git LFS).

---

## Page Categories

Each page has exactly one `category`, set in its frontmatter:

| Category | Holds |
| :--- | :--- |
| `overview` | The big picture of the project or one domain. Entry point that links its entities and concepts. |
| `entity` | A concrete thing: system, component, service, device, team, vendor, or tool. |
| `concept` | An idea, mechanism, protocol, or pattern. |
| `comparison` | Side-by-side analysis of alternatives, usually as a table. |
| `synthesis` | Cross-source analysis or a query answer filed back into the wiki. States the question it answers. |
| `source` | Summary of one ingested raw source: key takeaways and links to the pages it informed. The knowledge itself belongs in entity and concept pages. |

---

## Naming Convention

Pages live in `pages/` and are named in `kebab-case` after their subject, without numeric IDs. The file name is the link target. Raw sources use a `kebab-case` slug of the document title with the original extension, and a source page reuses that slug:

```text
pages/subject-name.md
raw/document-title.pdf  →  pages/document-title.md
```

**Examples:**
- `pages/payment-gateway.md` (`entity`)
- `pages/idempotency-keys.md` (`concept`)
- `raw/payment-provider-api-guide.pdf` → `pages/payment-provider-api-guide.md` (`source`)

---

## Writing Pages

Start each page from `template.md`. Set `category`, `title`, and `created_at`, then update `updated_at` on every change.

- **One subject per page**: Split a page that covers two subjects, and merge pages that cover the same one.
- **Current truth**: Rewrite a page so it states what is true now. Git keeps the history, so pages carry no changelog.
- **Citations**: Cite every non-obvious claim inline with a link to its source, and list each source in `sources`:
  - `raw/<file>`: a file in this wiki's `raw/` directory.
  - `PRD-NNNN`, `RFC-NNNN`, `ADR-NNNN`, `SPEC-NNNN`, `RUNBOOK-NNNN`: a project document.
  - `<repository>@<commit>`: code read at a given commit, with the repository as its hosting platform names it (for example, `owner/name`) and the commit as an abbreviated hash of at least 7 characters.
  - `<person>, YYYY-MM-DD`: a fact a human stated during an operation, such as an answer in Ingest step 2, when no document records it.
  - A URL, only when the content cannot be kept in `raw/`.
- **Links**: Use relative Markdown links: `[Name](other-page.md)` between pages, `[Title](../raw/file.pdf)` to sources, `[ADR-NNNN](../../adrs/NNNN-title.md)` to project documents, and the path from the page to a code file, naming the commit in the text, such as ``([`src/app.py`](../../../src/app.py) at 3de0829)``. They render on Git hosting platforms and in Markdown editors such as Obsidian.
- **Contradictions**: When a newer or more authoritative source settles a conflict, rewrite the page to match it and cite it. Otherwise, keep both claims with their sources in a callout that Lint can find:
  ```markdown
  > [!WARNING]
  > **Contradiction**: [Source A](../raw/a.pdf) states X; [Source B](../raw/b.pdf) states Y.
  ```
- **Compliance gaps**: A source that prescribes how things should be (a guide, a standard, a requirement) and a source that shows how they are (code, a measurement) do not contradict each other when they differ. Record the gap, citing both, on the page about the subject or in a `synthesis` page, without a callout. Closing the gap is a requirement change, so it goes to a PRD or a Spec.
- **Authority**: `approved` PRDs, `accepted` ADRs, Specs in `ready` or a later status, and `active` Runbooks are authoritative for what they cover. A wiki page summarizes and links them and never overrides them: it holds what the document does not, such as its context, rationale, and trade-offs in brief, how it relates to other subjects, and pointers to its rules by number, never copies of the rules. If a source contradicts an `accepted` ADR, flag the contradiction; changing the decision requires a new ADR.
- **Durable knowledge**: Record mechanisms and facts that stay true. Transient status, such as open tickets, current priorities, and one-off measurements, belongs in the work tracker.

---

## index.md

`index.md` is the content catalog. Every page appears in it exactly once, under its category, with a link and its one-line summary. A line may end with optional metadata such as the source count. The agent reads it first when answering a query and updates it on every change to `pages/`.

```markdown
# Wiki Index

## Overview

- [Project overview](pages/project-overview.md) — What the system does and how its domains fit together.

## Entities

- [Payment gateway](pages/payment-gateway.md) — Service that authorizes and captures card payments. (3 sources)

## Concepts

## Comparisons

## Syntheses

## Sources

- [Payment provider API guide](pages/payment-provider-api-guide.md) — The provider's REST API, rate limits, and error model.
```

---

## log.md

`log.md` is the chronological record. It starts with a `# Wiki Log` heading. Entries are appended at the end and never edited. Each entry starts with a fixed prefix, `## [YYYY-MM-DD] <operation> | <title>`, where the operation is `ingest`, `query`, or `lint`, followed by the pages the operation created or updated. Every ingest and lint pass gets an entry. A query gets one when it files a page back into the wiki, and an ingest that a query runs gets its own `ingest` entry before the `query` entry. A lint entry's title names what the pass covered (`Whole wiki` or a domain), and an `Open:` line lists what the pass left: data gaps with the sources it proposes, and findings it did not fix.

```markdown
# Wiki Log

## [2026-04-02] ingest | Payment provider API guide

- Created: [payment-provider-api-guide](pages/payment-provider-api-guide.md), [idempotency-keys](pages/idempotency-keys.md)
- Updated: [payment-gateway](pages/payment-gateway.md), [project-overview](pages/project-overview.md)

## [2026-04-09] lint | Whole wiki

- Updated: [payment-gateway](pages/payment-gateway.md)
- Open: retry limits of the provider API (propose the provider's support answer as a source)
```

The fixed prefix keeps the log greppable. For example, `grep "^## \[" log.md | tail -5` lists the last five entries.

---

## Operations

### Ingest

A human adds a file to `raw/`, hands the agent a file to copy there unchanged, or names a project document or code to ingest. Ingest one source at a time:

1. Read the source in full.
2. Discuss the key takeaways with the human, and agree on what matters before writing.
3. Write the `source` page. Skip this step for project documents and code, which already exist in the repository.
4. Update every `overview`, `entity`, and `concept` page the source affects, and create pages for new subjects. One source often touches 10–15 pages.
5. Add cross-links in both directions, and resolve or mark contradictions.
6. Update `index.md`.
7. Append an entry to `log.md`.

### Query

1. Read `index.md`, then the relevant pages.
2. Consult the sources (raw sources, project documents, or code) only when the pages do not answer the question, then ingest what was missing with the full Ingest procedure before answering.
3. Answer with citations that link to wiki pages and sources. The answer can be prose, a table, a comparison, slides, or a chart.
4. File valuable answers back into the wiki as `synthesis` or `comparison` pages, then update `index.md` and `log.md`.

### Lint

Run Lint periodically and after large ingests. Check for and fix:

- **Contradictions** between pages, and unresolved `**Contradiction**` callouts.
- **Stale claims**: claims that newer sources supersede, or that rely on `archived` PRDs, `rejected` RFCs, `superseded` or `deprecated` ADRs, `archived` Specs, or `deprecated` Runbooks.
- **Orphan pages**: pages that no other page links to. Links from `index.md` do not count.
- **Missing pages**: subjects mentioned across several pages that have no page of their own.
- **Missing cross-references** between related pages.
- **Index drift and broken links**: every page appears in `index.md` exactly once, and every link resolves.
- **Frontmatter**: every field of `template.md` is present, `category` is valid, and every cited source is listed in `sources`.
- **Data gaps**: questions the wiki cannot answer yet. Propose sources that would fill them and ask the human before fetching anything; the pass does not wait for the answer.

Record each pass in `log.md`, with what it left open.

---

## Tooling

The wiki is plain Markdown in Git and needs no tools. Markdown editors such as Obsidian can help with browsing, because their graph view shows hub pages and orphans. For large wikis, a local full-text or hybrid search tool helps the agent find pages beyond what `index.md` lists. The mechanical Lint checks (frontmatter fields, `category`, index drift, and broken links) suit a script in the project's CI; the other checks need reading. Wiki tooling may be added later.

---

## AI Agent Guidance

1. **Instructions for AI Agents**:
   - Read this README before any wiki operation, together with the project-specific wiki rules in the project's agent instructions.
   - Never modify, rename, or delete files in `raw/`.
   - Answer from the wiki first (`index.md`, then pages) and fall back to raw sources only for what the wiki lacks.
   - Keep `index.md` and `log.md` in sync with every change to `pages/`. When merging or deleting a page, update every link to it.
   - Do not restate PRDs, RFCs, ADRs, Specs, or Runbooks in wiki pages: summarize and link them. Record new requirements as PRDs, new decisions as ADRs, and new procedures as Runbooks.
2. **Traceability**:
   - Every claim must trace back to a source through an inline citation and the page's `sources` list.
   - The wiki is the most downstream layer: its pages reference project documents through `sources`, and those documents do not link back to the wiki.
3. **Schema Evolution**:
   - This README comes from the documentation framework and is updated with `git subtree pull`, so do not edit it in the consuming project.
   - Record project-specific rules (tag vocabulary, page language, domain boundaries) in the project's agent instructions file (`CLAUDE.md` or `AGENTS.md`), and propose changes there when a convention proves missing or wrong.
