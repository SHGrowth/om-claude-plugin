# Gap Analysis — Batch Mode

Multi-document engagement scoping. Turns a *folder* of client materials
(transcripts, spec docs, requirement dumps) into an evidence-backed Epic/Story
tree where every story carries a grounded verdict and an atomic-commit effort,
then derives a summary + prioritized backlog.

**This is om-cto's fourth mode, not a separate skill.** Advisory mode answers a
*single* question interactively and statelessly. Batch mode is the same gap
analysis at engagement scale: a directory in → a persisted, resumable backlog
out. It inherits Advisory's currency (atomic commits) and its Output Contract.
Its source-of-evidence rule started as Advisory's but has since diverged
(I038 — batch mode fans out across dozens of stories, so it earns a validated
local checkout Advisory's single-question shape doesn't need); folding batch
mode in here, rather than shipping a top-level `gap-analysis` skill, is
deliberate (om-superpowers surface-budget rule; the v1.16.0 `om-orchestrate`
deletion is the precedent). Source spec: `agents-master/improvements/I019.md`.

## When this mode applies

- A *directory of client docs* (or several pasted documents) → **batch mode** (this file).
- A *single capability question* ("does OM do X?") → **Advisory** (`references/advisory.md`), unchanged.

If the user points at one folder and asks "what would we build on top of OM,"
that's batch mode.

## Why the whole pipeline lives in om-cto, not a PM handoff

The intake → Epic/Story-tree half *looks* like `om-product-manager` territory.
It isn't: PM turns vague intent into well-formed stories with success criteria
and makes **no claim about the platform**. This tree is *subordinate to the
verdict* — it exists only as the addressing scheme for "does OM already provide
this," and every node carries an atomic-commit effort under this mode's gate.
The currency, the Output Contract, and the grounding discipline are
gap-analysis invariants; fold the tree-building out to PM and you fork those
across a skill boundary. So the tree is built here. (Clean
handoffs still hold in both directions: a PM story tree can be *handed in* as
intake docs; this backlog can feed PM for refinement.)

---

## Three phases, and why `/clear` between 1 and 2

The phases have different cognitive shapes:

- **Phase 1 — Scoping** is *input-heavy*: loads transcripts/specs (tens of thousands of tokens) → a slim structured MD.
- **Phase 2 — Verification** is *codebase-heavy*: walks OM via parallel subagents reading a validated local checkout, plus a one-time `gh` fetch for the Upstream-pipeline signal. Carrying phase 1's raw inputs into phase 2 wastes context better spent on grounding.
- **Phase 3 — Synthesis** reads only the now-filled MD → derived artifacts.

Between phase 1 and phase 2 the user runs `/clear` so phase 2 starts clean with
only the structured MD. Phases 2 and 3 share context.

The MD's `status: pending | done | needs-review` per story is what makes the run
**resumable**: an interrupted phase 2 resumes by re-invoking — done stories are
skipped, no flags needed.

---

## Phase 1 — Scoping (docs → Epic/Story tree)

**Goal**: read all client materials, produce one structured MD with an Epic →
Story tree where every story has an empty gap-analysis placeholder.

1. **Project slug** — kebab-case (`dentalos`, `bolttech-b2b`). Drives output filenames. Ask if unclear.
2. **Read all inputs.** Walk the input directory. Track each fact's source — you cite it in the story `**Source**` field.
3. **Extract requirements** across inputs: explicit feature requests, pain points in transcripts, integrations, compliance/multi-tenancy/GDPR/audit needs, reporting.
4. **Group into Epics** (4–10). A coherent area of value. Not one giant epic, not fifty tiny ones.
5. **Break each Epic into Stories** small enough for a single subagent to verify in one pass (1–3 acceptance criteria, one bounded capability). User/role perspective where possible.
6. **Suggest priority and dependencies.** P0 (blocking foundation), P1 (core), P2 (nice-to-have). Dependencies reference Story IDs.
7. **Write the MD** to `./gap-analysis/<project>.md` (create the dir). Template below.
8. **Hand off to Phase 1.5 (completeness gate), still in this session.** Phase 1.5 (populate the per-epic `#### Coverage` blocks via `om-product-manager`, then `bin/gap-checklist-gate`) runs *before* `/clear` — it edits the MD interactively. Only after the gate returns 0:

   > Phase 1 + completeness gate complete. Saved `<full-path>`.
   > Next: run `/clear`, then re-invoke om-cto with: `Run gap-analysis batch phase 2 on <full-path>`

**Do not start phase 2 in the same session, and do not `/clear` until `bin/gap-checklist-gate` returns 0.**

### MD tree template

```markdown
---
project: <slug>
generated: <ISO date>
sources:
  - inputs/requirements.md
  - inputs/transcript-2026-04-15.txt
phase: 1-scoped
total_epics: <n>
total_stories: <n>
coverage_categories:
  - error-path
  - permission-abuse
  - concurrency
  - nfr-multitenancy
  - nfr-gdpr
  - nfr-audit
---
<!-- coverage_categories is the completeness checklist bin/gap-checklist-gate enforces per epic.
     Extend it per domain when a run needs more (e.g. clinical-data-retention).
     Do NOT put inline `#` comments on the list or on coverage lines — the gate parses
     those literally. Use HTML comments on their own line, like this one. -->

# Gap Analysis — <Project Display Name>

> This file is the source of truth across all three phases.

## Epic 1: <Epic name>
**Goal**: <one-sentence outcome>
**Business value**: <why it matters to the client>

#### Coverage
<!-- Populated in Phase 1.5 by om-product-manager; checked by bin/gap-checklist-gate before Phase 2.
     Each category is EITHER a real story ref (`Story <id>`) OR `out-of-scope: <reason>` — never blank. -->
- error-path: Story 1.2
- permission-abuse: out-of-scope: <reason the client confirmed>
- concurrency: Story 1.4
- nfr-multitenancy: Story 1.3
- nfr-gdpr: out-of-scope: <reason>
- nfr-audit: Story 1.5

### Story 1.1: <Story title>
- **Description**: <as a [role], I want [capability], so that [outcome]>
- **Acceptance criteria**:
  - [ ] <criterion>
- **Source**: <source-file>:<location or quote>
- **Priority**: P0 | P1 | P2
- **Dependencies**: <story IDs, or "none">
- **Status**: pending

#### Gap analysis
<!-- Filled by phase 2 via the gate. Do not edit by hand. -->
- **Verdict**: ⚪ not yet analyzed
- **Evidence**:
- **Grounding query**:
- **Grounding source**:
- **Gaps**:
- **Effort**:
- **Suggested implementation path**:
- **Upstream pipeline**:
- **Investigated**:
```

Story IDs (`Epic.Story`, e.g. `2.4`) are stable — never renumber after phase 1.

---

## Phase 1.5 — Completeness gate (before `/clear` → Phase 2)

**Why this exists.** Phase 2 verifies *whatever stories the tree contains*. A
happy-path-only tree — "user books an appointment", nothing about double-booking,
cancellation, permission denial, concurrency, GDPR, audit — yields a confident,
complete-*looking* backlog that silently omits the hard 20%. Phase 1 only
*mentions* NFRs, and a mention does **not** bind (`feedback_text_channel_does_not_bind`,
N=17). So completeness is enforced **structurally**, the I019 gate one layer up:
the same "structural check outside the model loop binds; prose inside it drifts."

**The check is `bin/gap-checklist-gate`, not a prose reminder.** Every epic must
address each category in the MD's `coverage_categories` list, satisfied one of
two ways: a real `Story <id>` reference **or** `out-of-scope: <reason>`. Blank,
reasonless, or a reference to a story not in the MD all fail.

**Steps:**

1. **Populate, via `om-product-manager` (delegate — do not author story-critique here).** For each epic, PM proposes the missing negative-path / NFR stories *or* the explicit `out-of-scope: <reason>` for each unaddressed category, and writes them into the epic's `#### Coverage` block (adding the stories to the tree where needed). PM *populates*; it is not what binds.
2. **Run the gate:**
   ```bash
   bin/gap-checklist-gate ./gap-analysis/<project>.md
   ```
   - **exit 0** → every epic covers every category. Proceed to `/clear` → Phase 2.
   - **exit 1** → at least one epic has a category in neither state (the gate prints which epic + which category). **Do not start Phase 2.** Go back to step 1 for the flagged epics.
3. **Only on exit 0**, hand off to Phase 2.

**Known limitation (scope it honestly, mirror `gap-validate-finding`).** A green
checklist means *these declared dimensions are addressed*, **not** *the tree is
complete*. A dimension not on the list (i18n, data-migration, observability)
passes untouched — that is the price of decidability. You cannot enumerate every
missing story, but you *can* decide "does each epic have a negative-path story or
an explicit N/A per fixed category." Extend `coverage_categories` per domain when
a run needs more; do not replace the gate with prose.

---

## Phase 2 — Verification (the gated batch loop)

**Goal**: for every `status: pending` story, dispatch a read-only subagent to
investigate OM, then **gate** its findings before writing them into the MD.

**Preconditions (all three, in order):**
1. `bin/gap-orientation-preflight` returns 0 — it validates and hard-freshens
   **two** local checkouts, printing exactly two stdout lines on success:
   - **Line 1, `<REPO_ROOT>` (required).** A clean upstream `open-mercato/open-mercato`
     on `develop` (not `main` — `develop` sits hundreds of commits ahead of
     `main`'s last release; grounding against `main` alone silently undercounts
     what OM already has), not a fork or feature-branch WIP (e.g. the
     bug-triage checkout conventionally parked at `~/Documents/OM`), and just
     fetched-and-fast-forwarded. This checkout carries **three** jobs: code
     reading for orientation, merged-code verdict grounding (`bin/gap-validate-finding`
     greps it), and the routing AGENTS.md files themselves — vendored
     `om-reference/` is no longer a separate copy for this mode (I038).
   - **Line 2, `<OFFICIAL_MODULES_ROOT>` (best-effort).** The same validation
     against `open-mercato/official-modules@develop` — a shipped capability
     there is real `✅`/`🟡` evidence, not just an open-PR signal (I038; this
     was the main gap in I038's first implementation, which only ever
     snapshotted official-modules' open PRs, never its actual shipped code).
     Prints the literal string `UNAVAILABLE` if this checkout couldn't be
     validated/provisioned — that alone never fails the whole preflight (a
     smaller, auxiliary repo being briefly unreachable shouldn't block the
     entire run); `bin/gap-validate-finding` then correctly fails only the
     specific findings that actually needed it.

   Orienting or grounding against fork/WIP code primes false `✅`/`🟡` (I037);
   grounding against a stale checkout reopens the exact TagsInput-drift
   failure I019 closed, one layer earlier (I038) — which is why this
   preflight fetches and fast-forwards (and asserts `HEAD` actually equals the
   remote tip, not just that a fast-forward was *possible*) rather than just
   noting staleness.
2. `bin/gap-grounding-preflight` returns 0 — the narrower channel that's still
   live-`gh`-bound (open PRs on both `open-mercato/open-mercato` and
   `official-modules`, plus planned specs) is reachable. This channel feeds
   the **Upstream pipeline** field only — it never grounds a `✅`/`🟡`/`❌`
   verdict, so its preflight is a plain reachability check, not I036's
   original control-term search (I038).
3. `bin/gap-checklist-gate` returned 0 in Phase 1.5. Do not enter Phase 2 on a
   tree that has not passed the completeness gate.

### Architecture: orchestrator + subagents + gate

- **You are the orchestrator.** You parse the MD, dispatch the investigation subagents in one message, **validate each returned block through `bin/gap-validate-finding`**, edit the MD, update statuses. You do **not** explore the codebase yourself.
- **Subagents** are read-only investigators (one story each). They do not edit files. They propose the grounding query; they do not need to be trusted on whether they ran it.
- **The gate** (`bin/gap-validate-finding`) is the structural surface *outside each subagent's model loop*. This is load-bearing: prose rules in a subagent prompt do **not** bind it against fabrication-shape failures (`feedback_text_channel_does_not_bind`, N=17 incl. S012; I018). `bin/claude-validated` cannot help here — it wraps `claude -p`, and these subagents run via the **Task tool**, which it does not intercept. So the binding check lives in the orchestrator's parse step. The gate must verify the *goal* (a story-grounded verdict), not a *proxy* (a query↔verdict agreement a strawman query satisfies) — S012 is exactly a gate that went green on the proxy; the `--story` token guard re-ties it to the goal.

### Steps

0. **Preflights — the first action in Phase 2.** Run `bin/gap-orientation-preflight` and `bin/gap-grounding-preflight`. Both must return 0 before dispatching anything:
   - **`gap-orientation-preflight`**: **exit 0** → stdout is exactly two lines: line 1 is the validated, just-fetched `open-mercato/open-mercato@develop` checkout path — **capture it as `<REPO_ROOT>`**, pass it to every subagent (code-orientation) and to every `bin/gap-validate-finding` call (`--repo-root`, merged-code grounding); line 2 is the `official-modules@develop` checkout path, or the literal string `UNAVAILABLE` — **capture it as `<OFFICIAL_MODULES_ROOT>`** and pass it to subagents + `bin/gap-validate-finding` (`--official-modules-root`) whenever it isn't `UNAVAILABLE`. **exit 1** → the **core** checkout (line 1) is missing (and not auto-managed), not a git repo, not upstream OM, on the wrong branch (most commonly: pointed at a fork/feature-branch working checkout, e.g. bug-triage WIP conventionally parked at `~/Documents/OM`), or has a dirty working tree the preflight refused to fast-forward; **stop — dispatch nothing**, and **relay the preflight's stderr to the user verbatim** — it names the concrete fix. A bad **official-modules** checkout alone never triggers exit 1 — it only downgrades line 2 to `UNAVAILABLE` (with a WARN on stderr worth relaying too, since it means official-modules-sourced findings will need review). **exit 2** → a transient clone/fetch failure on the **core** checkout (rate limit, timeout); wait ~60s and re-run (I037/I038).
   - **`gap-grounding-preflight`**: **exit 0** → the PR/official-modules channel is reachable, proceed. **exit 1** → dead (gh unauthed / no repo access / a repo renamed); **stop — dispatch nothing**, and **relay the preflight's stderr to the user verbatim** — it names the concrete fix (`gh auth login`, repo access). Do not proceed until a re-run returns 0. **exit 2** → transient (rate limit); wait ~60s and re-run the preflight (I036/I038).
1. **Fetch the Upstream-pipeline snapshot once, for the whole run — never per-story.** The orchestrator is the sole `gh` caller for this channel, exactly once, regardless of story count (this is what keeps the fan-out in step 3 safe — see "Why the snapshot is fetched once" below):
   ```bash
   {
     echo "## Open PRs — open-mercato/open-mercato"
     gh pr list --repo open-mercato/open-mercato --state open --limit 100 \
       --json number,title --jq '.[] | "- PR #\(.number): \(.title)"'
     echo
     echo "## Open PRs — open-mercato/official-modules"
     gh pr list --repo open-mercato/official-modules --state open --limit 100 \
       --json number,title --jq '.[] | "- official-modules PR #\(.number): \(.title)"'
     echo
     echo "## Planned specs — .ai/specs/ (open-mercato/open-mercato@develop)"
     gh api "repos/open-mercato/open-mercato/contents/.ai/specs?ref=develop" --jq '.[].name' | sed 's/^/- /'
   } > /tmp/gap-upstream-pipeline-<project>.md
   ```
   Pass this file's path to every subagent as `<PIPELINE_SNAPSHOT>` in the prompt template below. Subagents match their story's domain against it themselves (semantic judgment, not a mechanical grep) — they never call `gh`.
2. **Load the MD.** Parse frontmatter + tree. List `status: pending` stories. Set `phase: 2-verifying`.
3. **Dispatch all `pending` investigation subagents in one Task-tool message.** No hand-counted batching — the Task tool already bounds its own concurrency, so the old fixed-size grouping was a self-imposed cap that bought nothing. Each subagent gets one story (the prompt template below) and returns a findings block in the schema below.
4. **Gate the returned blocks — one at a time — before writing each.** For each, write the story's title + acceptance criteria to a temp file and pass it with `--story`, pass `<REPO_ROOT>` with `--repo-root`, and pass `<OFFICIAL_MODULES_ROOT>` with `--official-modules-root` whenever it isn't `UNAVAILABLE`; the gate **requires** the story to ground a `❌ Missing` (without it, it cannot prove the grounding query references the story rather than a strawman — the S012 self-confirm guard), and **always re-runs every grounded verdict** — there is no shape-trust exemption for any `Grounding source` (I038 closes I019 §88 hole 3; a local search costs nothing, unlike the rate-limited `gh` call I019 was rationing when it shape-trusted a `live`-sourced positive):
   ```bash
   printf '%s\n' "$STORY_TITLE_AND_CRITERIA" > /tmp/story-<id>.txt
   GATE_ARGS=(--repo-root "$REPO_ROOT")
   [ "$OFFICIAL_MODULES_ROOT" != "UNAVAILABLE" ] && GATE_ARGS+=(--official-modules-root "$OFFICIAL_MODULES_ROOT")
   printf '%s\n' "$BLOCK" | bin/gap-validate-finding <story-id> "${GATE_ARGS[@]}" --story /tmp/story-<id>.txt
   ```
   - **exit 0 (PASS)** → write the block into the story's `#### Gap analysis`, flip `**Status**` to `done`, set `**Investigated**`.
   - **exit 1 (FAIL)** → do **not** write. Mark `**Status**: needs-review`, re-dispatch *once* with a reinforced prompt (echo the gate's stderr reason into the retry). If it fails again, leave `needs-review` and move on. (Covers: malformed block, verdict contradicted by the local re-run, a degenerate/strawman grounding query, an unrecognized `Grounding source`, a missing/invalid `--repo-root` or `--official-modules-root`, or an unrecognized **Upstream pipeline** shape.)
   - No exit 2 for this gate (I038) — grounding is a local `git grep`/`rg` re-run against an already-validated checkout, not a rate-limited network call, so there is no transient case to re-queue.
5. **Report progress** briefly: "X/Y done, Z needs-review."
6. **Resumability**: a story already `done` at phase-2 start is skipped.
7. **Cross-check the Upstream-pipeline citations — once, at the Phase 2→3 transition.** When all stories are `done` or `needs-review`, run the pipeline cross-check against the same snapshot file step 1 wrote:
   ```bash
   bin/gap-pipeline-crosscheck ./gap-analysis/<project>.md /tmp/gap-upstream-pipeline-<project>.md
   ```
   Why this exists: the field's *content* was the only per-story output decided purely by each subagent's isolated judgment against a one-line PR title — `bin/gap-validate-finding` checks its *shape* only, by design (non-verdict-altering, I036/I038). Observed on a real client engagement run (2026-07-08): a manual-KSeF-e-invoice-export story failed to cite `official-modules PR #29` ("…full KSeF feature set integration") despite "KSeF" appearing verbatim in both titles, while two sibling stories cited it correctly; 18 isolated re-trials of the identical prompt never reproduced the miss. Rare per-call variance multiplied by ~94 independent calls per run keeps producing a handful of misses regardless of prompt quality — so the backstop is deterministic and orchestrator-side (the same principle that grounds verdicts), never a "try harder" prompt rewrite (the S012 anti-pattern).
   - **exit 0** → every `done` story either cites a value that resolves to a real snapshot entry, or cites nothing while having no above-threshold candidates. Proceed to step 8.
   - **exit 1** → the stdout report lists each flagged story — a **missing** citation (cites nothing while above-threshold candidates exist; the full ranked candidate list follows, each annotated with the shared stems that matched) or a **phantom** one (cites a well-formed value found nowhere in the snapshot — explicitly labeled; since subagents read only this snapshot, a legitimate citation must exist in it). Review each row — this is the semantic judgment the script deliberately does not make: hand-edit the story's `**Upstream pipeline**` field where a flag is real; dismiss it where the lexical overlap is coincidental (a legitimate outcome — note dismissals briefly when reporting Phase 2 results, do not loop until exit 0). **Never auto-apply; never touch Verdict, Evidence, Effort, or Gaps.**
   - **exit 2** → wrong file or bad invocation (the script fails closed rather than passing vacuously); fix and re-run.

   Scope honesty (mirror the other gates' recorded limits): lexical overlap is a consistency backstop, not a relevance oracle — a flag can be a false positive, and a true match phrased entirely in synonyms can still be missed. This closes the *consistency* gap (94 isolated guesses vs one deterministic pass over the same data), not the *semantic-relevance* gap. `needs-review` stories are skipped — their field was never legitimately populated, so flagging it would be noise. A story whose citation resolves to *any* real snapshot entry is satisfied: one true match is enough, it is never required to enumerate every overlapping PR, and the script has no standing to second-guess *which* entry a subagent chose.
8. **When all stories are `done` or `needs-review` and the cross-check report has been reviewed**: set `phase: 3-synthesizing`, flow into Phase 3.

### Why the snapshot is fetched once — not per-story (the rate-limit discipline, carried forward)

I019 made the orchestrator the single canonical `gh` caller for grounding because GitHub's code-search endpoint allows ~30 req/min plus an undocumented secondary "abuse" limiter that trips on bursty parallel access — and step 3 dispatches *all* pending subagents in one message, so anything they called directly would race that limiter. I038 removes `gh` from grounding entirely (it's a local `git grep` against `<REPO_ROOT>` now, no rate limit, no `sleep`), but the **same fan-out hazard reappears** for the new Upstream-pipeline signal if it were fetched per-story instead of once: 40 parallel subagents each calling `gh pr list` would trip the exact limiter I019 built the single-caller rule to avoid. Fetching the snapshot once in step 1, before any subagent is dispatched, sidesteps this entirely — subagents read a static file, they never call `gh`.

**Validation (step 4) stays sequential, one story at a time — but for a plainer reason now.** With grounding local and the pipeline snapshot pre-fetched, there is no shared external resource left to protect; sequential processing here is a simplicity default (read a block, validate it, write it, move to the next — keeping the resumable per-story `**Status**` field consistent one story at a time), not a rate-limit requirement. If you parallelize this loop, nothing external breaks — it just isn't necessary to.

### Currency: atomic commits, never T-shirt sizes

Effort is an **atomic-commit score 0–5** per `references/atomic-commits.md` —
the same unit Advisory uses, so the two modes' backlogs reconcile. Do **not**
use XS/S/M/L/XL; the gate rejects T-shirt sizes (`bin/gap-validate-finding`
check 4). Score meaning (full table in `atomic-commits.md`):

| Score | Meaning |
|---|---|
| 0 | Platform does it, zero commits |
| 1 | 1 commit: config/seed only |
| 2 | 1–2 commits: small gap |
| 3 | 2–3 commits: medium gap |
| 4 | 3–5 commits: large gap |
| 5 | 5+ commits or external dependency |

**FLAG** any story whose plan touches `core-module` or `official-module` scope —
those carry upstream dependencies (see `atomic-commits.md` §Scope column).

### Source-of-evidence rule (verdict-conditional)

Vendored `om-reference/` is a daily snapshot of the plugin's own bundled copy —
answers "what did OM look like at sync time," not "what does OM provide," and
is 35 markdown files with **0 lines of source**: a Task Router plus per-module
conventions, not a code mirror. om-superpowers has already shipped a wrong
verdict from trusting a stale copy as evidence (the TagsInput drift bug, README
v1.13.0). This mode does not use that vendored copy at all — I038 collapsed it
away for gap-analysis specifically: the validated `<REPO_ROOT>` checkout
(`bin/gap-orientation-preflight`) already contains the *real* `AGENTS.md` files,
kept fresh by the same fetch-and-fast-forward that grounds verdicts, so reading
routing guidance from a second, separately-vendored copy would just be a
redundant, potentially-staler duplicate. (Other skills' use of the plugin's
vendored `om-reference/` — `om-ds-guardian`, `om-product-manager`, `om-ux`,
other om-cto modes — is untouched; they have no checkout precondition to
collapse onto.)

| Use | Allowed source |
|---|---|
| **Routing** — which module/guide to look at, where a feature would live | `<REPO_ROOT>/AGENTS.md` and the relevant module's own `AGENTS.md`, read directly from the validated checkout |
| **Code-level orientation** — read real entities/routes/UI to sanity-check a `✅`/`🟡` | `<REPO_ROOT>` (core) or `<OFFICIAL_MODULES_ROOT>` (official-modules), whichever the story's domain points to |
| **`✅`/`🟡` verdict evidence — core capability** | a local `git grep`/`rg` hit in `<REPO_ROOT>`, re-run by `bin/gap-validate-finding` (I038 — replaces the pre-I038 live `gh search code` hit) |
| **`✅`/`🟡` verdict evidence — shipped as an official module** | a local `git grep`/`rg` hit in `<OFFICIAL_MODULES_ROOT>`, re-run the same way (`Grounding source: official-modules`) — a real, merged official module is genuine coverage, not merely the Upstream-pipeline PR signal below (I038; this was the main gap I038's first pass missed — it only ever snapshotted official-modules' open PRs, never its shipped code) |
| **`❌ Missing` verdict evidence** | a local search in the relevant checkout returning no match, re-run and confirmed by the gate, **always**. Never "I didn't see it" without the re-run. |
| **Upstream pipeline** (an *open, unmerged* PR on either repo / a planned spec — supplementary, never verdict-altering) | the one-time snapshot the orchestrator fetches via `gh` in Phase 2 step 1 — never a per-story `gh` call (I038) |
| **Auditing the local app's own code** (impl phase) | local Glob/Grep, only here |

In one line: **the two validated checkouts (`<REPO_ROOT>`, `<OFFICIAL_MODULES_ROOT>`)
carry routing, orientation, AND every verdict's grounding, unconditionally;
`gh` narrows to one orchestrator-only bulk fetch for the Upstream-pipeline
signal, never a per-story call (I038).** `bin/gap-orientation-preflight`
enforces the checkout boundary — upstream `develop`, not a fork/feature-branch,
fetched and fast-forwarded before Phase 2 dispatches anything.
`bin/gap-validate-finding` enforces the grounding boundary — it re-runs the
cited query as a local search rather than trusting the subagent's pasted
result, for **every** grounded verdict, not just some (I038 closes I019 §88
hole 3 — see below).

### What the gate does NOT do (scope it honestly — I019 §88)

The gate is an asymmetric **falsifier**, not a truth oracle. Record these as
known gaps; do not let a green run read as "everything verified":

1. **Falsifier, not confirmer.** A search hit proves a string matches — not that the matched code satisfies the story's acceptance criteria. A `✅` pointing at real-but-irrelevant code passes clean. Semantic judgment stays with the subagent. *This is the one residual semantic hole.*
2. **Strawman queries — CLOSED on the `❌` path (review #2 / S012).** The gate re-runs *the query the subagent named*, so a fully-unrelated strawman (`zzqxnonexistentmodule12345`) could once self-confirm a false `❌`. It no longer can: the gate requires the story (`--story`) and rejects any `❌` whose grounding query shares no noun token with the story title/criteria. The same token check guards every positive when the story is supplied, regardless of `Grounding source` (I038 — this used to be phrased as "vendored positives" back when `vendored` was a real, distinct source; now every source is checked the same way). What it still can't catch is a *plausibly-related-but-too-narrow* query (`"AppointmentScheduler"` when the module is `modules/scheduling`) — the token overlaps, so it passes, but the search misses. That narrower case collapses into hole 1 (semantic relevance), not a free strawman.
3. **`develop` can contain unreleased code (I038).** A verdict grounded in either checkout proves the capability exists on OM's `develop` branch — not that it has shipped in a tagged release. The gate does not distinguish "merged last year" from "merged an hour ago and not yet released"; `bin/gap-orientation-preflight` surfaces this at the run level (how many commits `develop` sits ahead of `main`, per repo) rather than per-finding — coarse but honest.

**CLOSED — shape-trust (was hole 3, pre-I038).** Every grounded verdict is now re-run unconditionally, regardless of its `Grounding source`. The old exemption (a `live`/`checkout`-sourced `✅`/`🟡` skipped the re-run) existed only to ration GitHub's rate-limited search API; a local `git grep`/`rg` re-run has no such cost, so I038 removes the exemption instead of carrying it forward. `Grounding source` is now provenance only (which checkout backed the claim), never a bypass.

Net: the gate **falsifies ungrounded/stale `❌ Missing`, strawman-grounded `❌`,
and every unre-run `✅/🟡`** — the TagsInput failure mode, the S012
self-confirm, and (as of I038) the shape-trust hole, all closed. What remains
is semantic relevance (hole 1) and release-boundary honesty (hole 3, informational).

A separate failure — a **stale or wrong checkout**, where a checkout is a
fork, a feature branch, or simply behind its remote — is not a per-finding
hole this gate can see: it trusts whatever `--repo-root`/`--official-modules-root`
it's handed. It is closed one step earlier by `bin/gap-orientation-preflight`
(I037/I038), the Phase-2 precondition above, which validates each checkout's
remote and branch and hard-fetches it fresh (asserting `HEAD` actually equals
the remote tip, not just that a fast-forward was possible) before Phase 2
dispatches anything. Another separate failure — a **dead Upstream-pipeline
channel**, where `gh` can't reach the PR/official-modules repos — cannot
silently corrupt a verdict (that field is never verdict-altering) but would
silently under-report the pipeline signal; it is closed by the re-scoped
`bin/gap-grounding-preflight` (I036/I038). A third — a subagent that reads a
live channel but fails to *cite* the matching entry in its **Upstream
pipeline** field (rare per-call variance, observed once across ~94 calls on
a real engagement run) — is caught after the fact by `bin/gap-pipeline-crosscheck` at
the Phase 2→3 transition (step 7), which deterministically re-scores every
`done` story against every snapshot title and reports uncited matches for
human review, never auto-fixing.

### Subagent prompt template

Fill `<STORY_ID>`, `<STORY_BLOCK>`, `<REPO_ROOT>` (stdout line 1 of `bin/gap-orientation-preflight` in Step 0 — a validated upstream `develop` checkout of `open-mercato/open-mercato`, never `~/Documents/OM` or any other unvalidated local repo), `<OFFICIAL_MODULES_ROOT>` (stdout line 2 — the same, for `official-modules`; may be `UNAVAILABLE`, see below), `<PIPELINE_SNAPSHOT>` (the file written in Step 1).

```
You are a read-only Open Mercato codebase investigator in a gap analysis.
Verify whether ONE story is already implemented in Open Mercato — either in
core or as an official module — and return a structured findings block.
Investigate only the story below.

## The story
<STORY_BLOCK>

## How to investigate
1. Route with `<REPO_ROOT>/AGENTS.md` (Task Router) — which module owns this, for routing ONLY.
2. For code-level orientation — read the real entities/routes/UI — use the validated checkout at `<REPO_ROOT>`. It has already passed `bin/gap-orientation-preflight`, so it is upstream `develop`, not a fork/feature-branch; never substitute any other local checkout.
3. If the capability could plausibly ship as a separate official module rather than core (integrations, carriers, niche verticals), ALSO check `<OFFICIAL_MODULES_ROOT>` (skip this step if it's `UNAVAILABLE`) — a real, merged module there is genuine coverage, not just an in-progress signal.
4. For the verdict, search whichever checkout you found evidence in, locally (Grep/Glob), for domain nouns and synonyms. Only merged code counts — in either repo.
5. Check `<PIPELINE_SNAPSHOT>` (already fetched once for this whole run) for an *open, unmerged* PR or planned spec matching this story's domain — use judgment, not exact string matching. Report it in **Upstream pipeline**; it never changes your **Verdict** (a merged official module is a `✅`/`🟡` per step 3+4 above, not an Upstream-pipeline note). Never call `gh` yourself — this file is the only source for this field.
6. Name the single most decisive local search term in **Grounding query** — the orchestrator will RE-RUN it as a local search against whichever checkout you name in **Grounding source**, to verify your verdict — so pick the term that actually decides it (e.g. the module path `modules/<x>` or `packages/<official-module-name>`), not a vague word. Every verdict is re-run, with no exception for any source.

Tools: Read, Glob, Grep (scoped to `<REPO_ROOT>`, `<OFFICIAL_MODULES_ROOT>`, and `<PIPELINE_SNAPSHOT>`). Never Edit/Write. Never call `gh` — every `gh` call for this run is centralized in the orchestrator (Step 0 preflights + the one-time Step 1 snapshot).

## Output — return ONLY this block, no preamble:
- **Verdict**: ✅ Implemented | 🟡 Partial | ❌ Missing | ⚠️ Unclear
- **Evidence**:
  - `<repo-relative path>`: <role it plays>
- **Grounding query**: `<the single local search term that decides this verdict>`
- **Grounding source**: checkout | official-modules   <!-- 'checkout' = you searched <REPO_ROOT>; 'official-modules' = you searched <OFFICIAL_MODULES_ROOT>. No third option — there is no vendored copy to fall back to; every claim must be searched in one of these two checkouts. -->
- **Gaps**:
  - <specific missing piece, or "none">
- **Effort**: <atomic-commit score 0–5; see scoring — NEVER XS/S/M/L/XL>
- **Suggested implementation path**:
  - <steps referencing existing OM patterns>
- **Upstream pipeline**: none | PR #<n> (open) | official-modules PR #<n> (open) | spec: <path> (planned, unbuilt)

Rules the orchestrator's gate enforces (your block is rejected and re-dispatched if violated):
- Effort is a number 0–5, never a T-shirt size.
- No percentage without an N/M fraction. No hedges (approximately/around/roughly). No persona names.
- **Grounding source** must be exactly `checkout` or `official-modules` — the orchestrator re-runs your query against the checkout your source names, so an unrecognized value (including `vendored`) is rejected outright, not defaulted.
- Every verdict (not just ❌ Missing) MUST name the search term that decides it; the orchestrator re-runs it against the named checkout, no exceptions.
- **Upstream pipeline** is required (use `none` if you found nothing) and must match one of the four recognized shapes above.
```

---

## Phase 3 — Synthesis (MD → summary + backlog)

**Goal**: read the now-complete MD, produce two derived artifacts. Never
re-derive findings from memory — always read the MD (this is what keeps it
audit-friendly).

1. **Aggregate stats**: total epics/stories, verdict distribution, atomic-commit effort totals by verdict and by priority, stories blocked by dependencies, and `needs-review` count (surfaced separately, never folded into the distribution).
2. **Produce `<project>-summary.md`** (template below).
3. **Produce `<project>-backlog.md`** (template below) — a sequenced delivery plan; every item links its Story ID(s).
4. **Update source MD frontmatter**: `phase: complete`.
5. **Present all three files.** Phase 3 is idempotent — re-run freely.

### Summary template — `<project>-summary.md`

```markdown
---
project: <slug>
generated: <ISO date>
source: <project>.md
type: gap-analysis-summary
---

# Gap Analysis Summary — <Project Display Name>

## Executive summary
<2–3 plain-language paragraphs: how much already exists, the biggest risks, the recommended start. May be read by a client stakeholder.>

## Coverage at a glance
| Verdict | Stories | Share | Effort (commits) |
|---|---|---|---|
| ✅ Implemented | <n> | <n>/<total> | — |
| 🟡 Partial | <n> | <n>/<total> | <sum> |
| ❌ Missing | <n> | <n>/<total> | <sum> |
| ⚠️ Unclear | <n> | <n>/<total> | <sum> |

<!-- Share is N/total, never a bare percentage — the Output Contract forbids it. -->
**Total effort to close gaps**: <sum> atomic commits across <k> stories.

## Coverage by epic
| Epic | Implemented | Partial | Missing | Unclear | Notes |
|---|---|---|---|---|---|
| 1. <name> | <n> | <n> | <n> | <n> | <one-line takeaway> |

## Top 5 risks
Each cites a Story ID, why it matters, and what unblocks it. Prefer P0+❌, then P0+🟡, then P1+❌ with downstream deps. Don't pad to 5.

## Recommended sequencing
<Which epics first and why, referencing dependencies. Concrete.>

## What's already strong
<What OM already covers — frames the engagement positively.>

## Open questions
<⚠️ Unclear stories and scoping assumptions the client should confirm.>
```

### Backlog template — `<project>-backlog.md`

```markdown
---
project: <slug>
generated: <ISO date>
source: <project>.md
type: gap-analysis-backlog
---

# Implementation Backlog — <Project Display Name>

> Grouped into delivery phases by dependency order. Effort tags are atomic-commit scores (same currency as the gap analysis).

## Phase A — Foundation
<Stories whose absence blocks everything else. Usually P0 + ❌/🟡, no deps.>

### A.1 — <Item title>
- **Stories**: <1.1, 1.2>
- **Verdict context**: ❌ Missing
- **Effort**: <0–5>
- **Scope flag**: <app | core-module | official-module | n8n | external — FLAG core/official>
- **Dependencies**: none
- **Outcome**: <what's true when done>
- **Implementation notes**: <condensed suggested path from the MD>

## Phase B — Core capabilities
## Phase C — Differentiation & polish
## Out of scope (for now)
## Cross-cutting work

## Effort roll-up
| Phase | Items | Effort (commits) |
|---|---|---|
| A — Foundation | <n> | <sum> |
| B — Core | <n> | <sum> |
| C — Polish | <n> | <sum> |
| Cross-cutting | <n> | <sum> |
| **Total** | **<n>** | **<sum>** |
```

### Cross-check before writing files

- Every Story ID in the backlog exists in the source MD.
- Stories across phases (A+B+C+out-of-scope) = total minus `✅ Implemented` needing no work.
- Effort totals in the summary equal the backlog roll-up.

If anything doesn't add up, surface it as a footnote — never silently fudge.

---

## Acceptance tests (the ship bar — I019 §Verification)

The mode ships only if these pass. Tests 3 and 4 are binding.

1. **Path/load**: invoke batch mode on a 2-doc fixture; an MD tree is produced (no missing-file errors — everything is in this one reference).
2. **Currency**: `grep -E 'XS|XL'` over the backlog → zero hits. Effort is 0–5.
3. **Gate (binding), two halves**:
   - *Form*: a `❌ Missing` block with no citation → gate refuses to write (`needs-review`).
   - *Grounding*: a `❌ Missing` block carrying a well-formed query **for a capability that genuinely exists in `<REPO_ROOT>`** → the orchestrator re-runs the query as a local search, sees hits, rejects the block. (Verified: `bin/gap-validate-finding` returns exit 1 here — e.g. query `currencies` against the real `develop` checkout. This is *not* a shape check — the query is actually re-run.)
4. **Staleness (the binding one) — assert the checkout is fresh before grounding, not after.** Seed `<REPO_ROOT>` N commits behind `origin/develop` (a capability landed upstream since the checkout was last fetched) and re-run `bin/gap-orientation-preflight`: it fast-forwards the checkout to the new tip **before** returning `<REPO_ROOT>`, so a subsequent grounding re-run sees the capability rather than missing it. Assert on the **checkout's own HEAD after the preflight**, not on any single subagent's return — the preflight, not the subagent, is what's supposed to close this gap.
5. **Currency-regression**: `grep -E '\b(XS|XL|[0-9]+%)\b'` over backlog + summary → zero hits.
6. **Completeness gate is structural, not prose (I024):** `bin/gap-checklist-gate docs/specs/fixtures/gap-checklist/happy-path-only.md` → **exit 1** naming the unaddressed categories; `…/complete.md` → **exit 0**. A `#### Coverage` category satisfied by a story ref to a story not in the MD, or an `out-of-scope:` with no reason, also fails — the gate checks the *goal*, not a presence proxy.
7. **No-batch (I023):** the implementation brief's Part-1 acceptance grep (for the retired fixed-size-batch phrasings) returns zero hits over this reference; the Upstream-pipeline snapshot fetch is a single per-run call, never per-story, preserving the single-caller discipline I019 established for a different channel.
8. **Grounding preflight (I036/I038) — the pipeline-channel-dead guard.** `bin/gap-grounding-preflight` → **exit 0** when both `open-mercato/open-mercato` and `open-mercato/official-modules` are reachable via `gh api`. `gh` absent from PATH → exit 1. A repo that's renamed/inaccessible → exit 1, naming the fix. This channel only feeds the non-verdict **Upstream pipeline** field — Phase 2 must not start on a non-zero preflight, but a failure here can never corrupt a `✅`/`🟡`/`❌` verdict.
9. **Orientation preflight (I037/I038) — the fork/feature-branch AND staleness guard, for BOTH checkouts.** `bin/gap-orientation-preflight` → **exit 0** against a clean upstream `develop` checkout, stdout = exactly two lines (core path, official-modules path), and each checkout is fetched-and-fast-forwarded with `HEAD` asserted to equal the remote tip (verified: seeding either checkout behind its remote and re-running fast-forwards it, not just a note). Pointed at a fork's feature-branch checkout (the real, observed bug: `matgren/open-mercato @ feat/*`) → **exit 1**, naming the branch and the fix — this is the binding case. A checkout with `origin=<fork>` and a *different* remote matching upstream → correctly grounds against the matched remote, not `origin` (verified against a realistic fork-with-shared-history repro, not just a synthetic one). A checkout one commit *ahead* of its remote (local-only commits) → **exit 1**, not silently accepted (`merge --ff-only` alone is a no-op in this case; the explicit `HEAD`-equality check catches it). A git dir with no `open-mercato/open-mercato` remote → exit 1. Missing path with no existing dedicated cache → auto-clones one; a conventional candidate path (`~/Documents/open-mercato` by default) that's *already* a valid, on-branch, clean checkout → reused instead of cloning fresh; a candidate on the wrong branch (e.g. `main`) → left untouched, falls back to the dedicated cache. A dirty working tree → exit 1 rather than risking a destructive fast-forward. Official-modules unreachable → core checkout still succeeds, line 2 reads `UNAVAILABLE`, exit 0 (best-effort, never blocks the whole run).
10. **Every grounded verdict is re-run, no exceptions (I038 closes I019 §88 hole 3).** A `✅`/`🟡` declaring `Grounding source: checkout` (or `official-modules`) with zero real hits in the named checkout → **exit 1**, rejected — this is the fix for a real gap in I038's first pass, which still shape-trusted these sources the way pre-I038 shape-trusted a `live`-sourced positive. A genuine hit still passes. An unrecognized `Grounding source` value (typo, or a value from before this fix) → **exit 1**, rejected outright rather than silently defaulting to the core checkout.
11. **Official-modules IS real coverage evidence, not just a pipeline signal (I038, the main gap in the first pass).** A story matching a real, merged official-modules package (verified: `packages/carrier-inpost` in the actual checkout) with `Grounding source: official-modules` → **exit 0**, grounded against `<OFFICIAL_MODULES_ROOT>`, not `<REPO_ROOT>`. A fabricated official-modules claim (zero hits) → **exit 1**. An official-modules-sourced finding with no `--official-modules-root` supplied (e.g. because line 2 was `UNAVAILABLE`) → **exit 1** with a clear "no --official-modules-root was provided" message, never silently mis-grounded against the core checkout instead.
12. **Local-grep grounding never inflates the Upstream-pipeline signal into a verdict (I038).** A finding whose **Grounding query** has zero hits but whose **Upstream pipeline** names an open PR → the verdict must still be `❌ Missing`, never silently upgraded; `bin/gap-validate-finding`'s shape check on **Upstream pipeline** is independent of its verdict-grounding logic and never overrides it. Missing **Upstream pipeline** field entirely → gate rejects (required, `none` is a valid value, and single-digit PR numbers like `PR #9 (open)` are correctly accepted — a shape-check regression once rejected these while accepting non-numeric junk like `PR #1abc2 (open)`, since fixed). `--repo-root` missing or pointed at a non-git path when grounding is required → gate rejects with the concrete fix.
13. **Pipeline-citation cross-check (Phase 2→3).** `bin/gap-pipeline-crosscheck` fixtures in `docs/specs/fixtures/gap-pipeline-crosscheck/`: a `done` story sharing a snapshot-unique proper-noun stem (`KSeF`) with an open PR while citing `none` → **exit 1**, the report names the story, its cited value, and the ranked candidates with their shared stems. This is the binding case — verified against the *real* engagement run with the missed story's manual fix reverted: the report flags it with exactly `official-modules PR #29` as its sole candidate. The same story citing that PR → **exit 0**, even though a second, *higher-scored* candidate (the fixture's PR #109, two shared stems) remains uncited — one true match satisfies, a story is never flagged for not also citing another candidate. A `done` story citing a well-formed value found nowhere in the snapshot (fixture Story 5.2, `official-modules PR #77 (open)`) → **exit 1**, labeled a phantom citation — checked for every done story, even one with zero lexical candidates; the same story citing any real snapshot entry passes (the accepted any-resolving-citation limit, recorded in step 7). Spec citations resolve by exact basename equality, never substring — a cited path that merely contains a real snapshot filename (`…outbox.md.bak`) is a phantom; a citation missing only its `.md` extension is the one tolerated variation. `needs-review` stories → skipped entirely, never flagged over their unpopulated placeholder. A stem in more than max(5, 5% of snapshot titles) candidates (`invoice`, in 7 of 13 fixture titles) → non-distinctive, never matches. A snapshot-unique but lowercase stem (`number`) → excluded from the single-stem path by the proper-noun gate; only ≥2-stem overlaps or internally-cased/numbered unique tokens (`KSeF`, `DataTable`) match on one stem. Swapped or wrong input files → **exit 2**, fail-closed, never a vacuous pass; CRLF line endings and multibyte (Polish-diacritic) tokens are handled, not silently misparsed. The script only ever reads — it never edits the MD and is structurally incapable of altering a Verdict, Evidence, or Effort value.

If tests 3 and 4 pass, the stale-absence hole closes (the TagsInput failure mode)
and the false-`❌` half of the I018 fabrication hole closes with it. The
fabrication hole is **narrowed, not sealed** — the residual holes above
(narrow-query, semantic-relevance, and `develop`'s release-boundary honesty)
are accepted tradeoffs, recorded here so they are not mistaken for closed.
Shape-trust (the former hole 3) is CLOSED, not merely narrowed, per test 10.

## Cross-refs

- `bin/gap-validate-finding` — the verdict-layer gate (Phase 2); grounds every verdict via a local `git grep`/`rg` re-run against `--repo-root` or `--official-modules-root` (whichever `Grounding source` names), not `gh`, and not exempt for any source (I038).
- `bin/gap-checklist-gate` — the intake-layer completeness gate (Phase 1.5); fixtures in `docs/specs/fixtures/gap-checklist/`.
- `bin/gap-orientation-preflight` — the checkout-layer preflight (Phase 2 precondition); validates and hard-freshens local upstream `develop` checkouts of BOTH `open-mercato/open-mercato` (required) and `official-modules` (best-effort), auto-provisioning or reusing a qualifying conventional path — these checkouts carry routing, orientation, AND every verdict's grounding (I037/I038).
- `bin/gap-grounding-preflight` — the pipeline-channel preflight (Phase 2 precondition); fails fast if `open-mercato/open-mercato` or `official-modules` is unreachable via `gh`, before any subagent runs (I036/I038).
- `bin/gap-pipeline-crosscheck` — the pipeline-citation cross-check (Phase 2→3 transition, `done` stories only); deterministically re-scores every story against every snapshot PR/spec title (S012 stemming + within-snapshot IDF + story-centrality + a proper-noun gate on single-stem matches) and flags a story only when it cites nothing while above-threshold candidates exist, or cites a value found nowhere in the snapshot (a phantom) — never auto-fixes, never alters a verdict, never requires a story to cite more than one true match. Fixtures in `docs/specs/fixtures/gap-pipeline-crosscheck/`.
- `references/atomic-commits.md` — the inherited currency + scope flags.
- `references/advisory.md` §Output Contract — the contract this gate enforces structurally; line 99's vendored-`OR` is tightened here (candidate I020 would tighten advisory itself).
- `bin/claude-validated` (I018) — the source of the five form checks (relocated into the orchestrator parse step, not the `claude -p` wrapper).
