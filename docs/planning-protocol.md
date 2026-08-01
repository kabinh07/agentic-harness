# Planning protocol (shared by all planning-phase commands)

`/agentic-harness:brd`, `:srs`, `:design`, `:features`, `:adr`, and `:epics`
all follow this protocol instead of restating it. `/agentic-harness:plan`
follows it for its own cross-stage bookkeeping. If a stage command's own
file says something more specific, the stage file wins — this is the
default, not an override.

## Why a shared protocol

Six stage commands doing the same four things (calibrating, asking,
gating, recording) should do them identically, or a teammate reading
`planning/BRD.md` today and `planning/FEATURES.md` next week hits a
different convention each time. This file is the one place those
conventions live.

## Document format

Every generated artifact (BRD, SRS, and — adapted per artifact — FEATURES,
DESIGN, ADRs, EPICS) is a standalone document a stakeholder can open cold,
not a prose memo. Structure:

- **Header table** — Document title, Version, Date, Author, Based on (the
  predecessor artifact + its version, where applicable), Status
  (`Draft` → `For review` → `Approved`).
- **Numbered sections**, each with a clear heading — no unstructured
  prose walls.
- **Requirements as tables**, one row per requirement:
  `| ID | Requirement | Priority |` (or `| ID | Category | Requirement |`
  for NFRs). Never a bulleted prose list once a section is requirement
  -bearing — tables are what a reader scans and what a later stage parses.
- **Module-prefixed IDs**: `FR-<MODULE>-##` / `NFR-<CATEGORY>-##`, not a
  flat incrementing number. Modules are **derived fresh from this
  project's actual business objectives/scope every time** — never assumed
  from a template project. Don't force everything into one bucket, and
  don't invent a module with only one requirement in it if it obviously
  belongs with a neighbor.
  - Conventional software often modularizes around `AUTH`, `USER`,
    `NOTIF`, `SEARCH`, `SETTINGS` — fine when that's genuinely what the
    system is.
  - An AI/agentic system usually modularizes differently and should not
    be forced into that shape: e.g. `GEN`/`META` (prompt→agent or
    prompt→output generation), `MODEL` (inference/model config,
    provider/backend selection), `RUNTIME` (execution/orchestration
    engine), `TOOL`/`CONN` (tool or connector integration), `RAG`/`KNOW`
    (retrieval/knowledge grounding), `GUARD` (safety, guardrails,
    human-approval gates), `EVAL`/`OBS` (evaluation, observability,
    tracing), `MEM` (memory/context management) — alongside whatever
    conventional modules (auth, users, notifications) the system also
    genuinely needs around that AI core.
  - The point is fit, not a fixed catalog from either list — read what
    this specific BRD's objectives actually describe and name modules
    after that, mixing conventional and AI-shaped modules freely when the
    system is both (most agentic products are).
  - A module name may legitimately be technical-sounding (`MODEL`,
    `RUNTIME`) — that's just a category label. The **requirement text**
    filed under it must still clear the BRD vs SRS altitude bar below;
    the module name doesn't grant an exception.
- **Priority** uses **M** (Must-have) / **S** (Should-have) / **C**
  (Could-have) — not "high/medium/low," not must/should/could spelled out
  in the table (the letter is the convention).
- **IDs are inherited, not reinvented, downstream.** SRS elaborates the
  same `FR-<MODULE>-##` IDs the BRD introduced for that module, continuing
  each module's own sequence for SRS-only additions. A requirement's ID
  never changes as it moves from BRD → SRS → traceability matrix — that
  stability is the entire point of numbering it.
- **Business rules / error handling** — a short italic note under a
  module's requirement table for the non-obvious operational detail (edge
  cases, error phrasing, rate limits) that doesn't deserve its own
  requirement row.
- **Assumptions & Constraints** — background conditions the plan operates
  under, stated as accepted fact once confirmed with the user (not hedged
  with a "may be wrong" marker — if it's still uncertain, it's an Open
  Item, not an assumption; see below).
- **Open Items (TBD)** — a single numbered appendix/section per document
  for genuinely unresolved decisions, each referencing the section it
  affects (e.g. "3. Attachment size/type limits (§6.4)."). This replaces
  scattered inline uncertainty markers — one place a reviewer checks for
  "what's still open," not several.
- **Traceability matrix** (SRS onward) — `| Upstream requirement | Covered by |`
  as its own appendix, not folded into prose.
- Closing line: `*End of document — Draft vX.X. Open items: <1-line summary
  or "none">.*`

## BRD vs SRS altitude

The single most common way a BRD goes wrong: it reads like a spec. A BRD
requirement describes a **business capability or outcome** — what the
business/user needs and why, in language a non-technical stakeholder
reads cold. An SRS requirement, same ID, one altitude lower, describes
**how the system satisfies that capability** — data model, specific
mechanisms, technology choices, exact thresholds, acceptance criteria.

**Litmus test:** if a requirement names a database table/column, a
specific library/protocol/vendor product, an algorithm, a file, a config
key, or an HTTP status code — that's SRS-altitude detail. It doesn't
belong in the BRD, even filed under an ID the BRD table introduces as a
placeholder row. When in doubt, read the requirement aloud to someone
outside engineering — if a term in it would make them ask "what's that,"
it's probably too low.

This applies to every requirement-bearing document, not the BRD alone —
Business Objectives, Scope, Background, and Assumptions & Constraints are
BRD-altitude too. A "no cloud AI provider" or "single-server deployment"
style constraint is fine at BRD level (it's a real business/procurement
decision, like naming an approved vendor) — naming the specific
database/ORM/queue/library that implements it is not.

| BRD-level (business) | SRS-level (technical) |
|---|---|
| "The product shall keep every organization's data completely separate from every other's." | "Every query against agents/runs/documents tables filters by `tenant_id`; cross-tenant reads/writes are impossible through the API." |
| "Connector credentials shall be protected so they can't be read in usable form if the underlying storage is compromised." | "Gmail OAuth tokens and SQL connection strings are stored via a secret-store abstraction; app code never reads/writes raw credentials from/to the database directly." |
| "Only the Owner role may change organization-wide settings." | "Only the Owner role can modify tenant-level settings (retention window, removing Admins, deleting the org), enforced via a role-check decorator on the settings router." |

`/agentic-harness:brd` writes the left column; `/agentic-harness:srs`
writes the right column under the same ID. A BRD reviewer who hits the
right column's level of detail should push back — that's a signal the
document needs a rewrite at the correct altitude, not a nitpick.

## Versioning

No artifact is ever silently overwritten. Every artifact's header table
carries a `Version` field (starts `0.1 (Draft)`). Before writing any change
to an artifact that already exists on disk:

1. Copy the current file verbatim to
   `planning/versions/<ARTIFACT>_v<current-version>.md` (e.g.
   `planning/versions/BRD_v0.1.md`) — the archive is the full document as
   it stood, not a diff.
2. Bump the new file's `Version`:
   - A revision within the same review cycle (user asked for changes
     before approving) → minor bump, `+0.1`.
   - Approval → same version number, `Status` changes to `Approved`
     (still archive the pre-approval draft per step 1 first).
   - A material amendment *after* approval (an earlier artifact changed,
     or — like a scope expansion mid-session — the user adds something
     substantial) → minor bump, `Status` reverts to `For review` until
     re-approved. Note in the new version what changed and why, one line,
     under Open Items or a short "Changelog" note if the change is
     non-trivial.
3. Never delete a version file. `planning/versions/` is an append-only
   history — if it's getting large, that's a sign of a lot of iteration,
   not a problem to clean up mid-project.

`planning/README.md`'s stage table carries a Version column alongside
Status/Approved, always matching the live artifact's header table.

## Calibration (once per planning session)

Before the question ladder starts (first thing `/agentic-harness:brd` does
on a fresh planning run, or `/agentic-harness:plan`'s Phase 2 if driving
the whole walk), establish two dials — don't re-ask per stage, reuse for
the rest of the session:

- **Knowledge level** (`planning.knowledge_level`): **new** (lacks core
  vocabulary/model for this domain) / **working** (understands basics,
  can discuss tradeoffs) / **expert** (knows the domain deeply, wants
  sharp critique). Default **working** if unanswered.
- **Pressure level** (`planning.pressure_level`): **light** (just clarify
  goals/constraints/missing context, stop once top ambiguities resolve) /
  **standard** (challenge assumptions and tradeoffs, keep going until the
  path is concrete) / **hard** (probe failure modes, edge cases,
  incentives, reversibility; name weak reasoning directly). Default
  **standard** if unanswered.

The user can change either dial mid-session ("softer," "harder," "teach
more," "skip basics") — update the config field immediately when they do.

**If knowledge is new:** define the one missing concept in 2-4 sentences
before asking about it, avoid unexplained jargon, ask fewer branching
questions, focus on goals/constraints/first principles before tradeoffs.

**If knowledge is expert:** skip basics entirely, ask sharper
counterfactuals, probe hidden costs/adverse incentives/long-term
maintenance, ask what evidence would change their mind.

**If pressure is light:** keep questions clarifying, stop as soon as the
top ambiguities for that section are resolved — don't manufacture more
questions to fill the ladder.

**If pressure is hard:** be direct, name weak reasoning when you see it,
ask about the unpleasant edge case instead of the comfortable one, demand
something observable for "done," still ask one focused question (or one
batched `AskUserQuestion` call) at a time.

## Questioning

- Work a **question ladder**, not a flat checklist — move through goal
  fit, constraint reality, option pressure, execution/scope, failure
  modes, validation, and reversibility, in that order, stopping early once
  the artifact section is clear enough (see stopping conditions below).
  Each stage command's own file maps this ladder onto its specific
  question bank — the ladder is the shape, the stage file is the content.
- Every question needs a recommended answer and, when it's not obvious
  why, a one-sentence rationale — this is what `AskUserQuestion`'s first
  option (labeled "Recommended") plus its description field are for. The
  user is picking or correcting, not brainstorming from a blank page.
- Batch related questions into one `AskUserQuestion` call — it caps at 4
  questions per call, so group by ladder rung or by topic, not one call
  per sub-point.
- If the answer can be found by reading the codebase (via
  `agentic-harness:codebase-analyst`), existing artifacts, or config,
  inspect those first instead of asking — see Never fabricate below on
  presenting inferences.
- If an answer implies an immediate follow-up (e.g. "yes, multi-tenant"
  implies "so what isolates tenants?"), ask that follow-up before moving
  on — don't let something one question away from being answered become
  an Open Item instead.
- Don't over-ask a **new**-knowledge user on domain basics, and don't
  under-ask an **expert** — see Calibration above.

### Stopping conditions

Stop the ladder for a given artifact/section once any of these is true:

- Goal, constraints, chosen approach, and a way to validate it are all
  clear enough to write the requirement/section.
- The user says stop, or to move on.
- What's missing can only come from external research or code exploration
  the user can't do live — capture it as an Open Item instead of stalling
  the session on it.
- The user's knowledge gap is blocking useful questioning — switch to a
  brief explanation of the missing concept, then re-offer the next
  question rather than repeating it unexplained.

## Never fabricate

- No detail invented to fill a gap, ever — not a plausible-sounding
  success metric, not a guessed persona, not an assumed NFR.
- If a question genuinely has no answer yet (user defers, doesn't know,
  says "decide later"), the item goes into that artifact's **Open Items
  (TBD)** section (numbered, with a section back-reference) — never
  invented, never silently dropped — and is mirrored into
  `planning/README.md`'s Open Items log with a pointer back to the
  artifact.
- A confirmed background condition (the user actually answered, it's just
  a constraint rather than a requirement) goes into **Assumptions &
  Constraints** instead, stated plainly — it's not "unvalidated" once the
  user has confirmed it, so don't hedge it with an uncertainty marker.
- Inference from an existing codebase (via `agentic-harness:codebase-analyst`)
  is not fabrication — it's evidence-based — but it must be presented to the
  user as an inference to confirm or correct, never asserted silently as
  fact in the final artifact. Once confirmed, it moves to a normal
  requirement/Assumptions entry, no longer flagged as inferred.

## Approval gate

Every stage ends the same way:

1. Present a concise summary of what the artifact says (not the whole file
   dumped back at the user — the shape and key points, plus current
   Version/Status from its header table).
2. Ask: approve as-is / revise (what to change) / skip this stage for now.
3. **Approve** → archive-then-write per Versioning above, `Status:
   Approved`, set `planning.stages.<stage>.status: approved` and
   `approved_on: <today>` in `planning/project.config.yaml`.
4. **Revise** → archive-then-write with a version bump per Versioning,
   re-present, loop.
5. **Skip** → only valid where the stage itself allows it (currently only
   `design`, gated on `planning.has_ui`) — set `status: skipped`, write
   nothing, and don't re-ask on future `/agentic-harness:plan` runs unless
   the user explicitly revisits it (`/agentic-harness:plan design`).

A later stage that requires an earlier one to be `approved` (see each
stage's own gate) refuses to run against a `draft` predecessor — tell the
user which stage is blocking and why, don't proceed anyway without the
user overriding explicitly.

## Bookkeeping

After every stage — approved, revised, or skipped — update, as one atomic
step:
- `planning/project.config.yaml`'s `planning.stages.<stage>`.
- `planning/README.md`'s stage status table (incl. Version column), Open
  Items log, and Assumptions & Constraints summary.

Never let these drift out of sync with each other or with the artifact
files themselves — `planning/README.md` is what a teammate reads first, so
it must always reflect reality without them having to open every artifact.

## Traceability

Every artifact after the BRD should be able to point back up the chain:
SRS requirement → BRD requirement (same ID, elaborated); Feature → SRS
requirement(s); ADR → Feature/SRS requirement(s) it enables; Epic/Story/
Task → Feature → SRS requirement → BRD requirement. A stage command that
finds an item with no upstream trace reports it as an orphan rather than
silently keeping or dropping it — see each stage's own orphan-check step
and the SRS's Appendix A traceability matrix.
