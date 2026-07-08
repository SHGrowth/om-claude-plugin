# Review handoff — gap-pipeline-crosscheck (v1.23.0, uncommitted)

You are reviewing an uncommitted change set in `/Users/maciejgren/Documents/om-superpowers`
(git repo, branch `main`, nothing staged or committed — run `git status` / `git diff` to see it).
Your job: find real defects the previous review missed, and verify its fixes landed without
regressions. Review only — do not edit any repo file. Report findings with a concrete repro
(command + expected vs got) or an exact quote; an empty findings list is a valid result.

## What was built and why

A new structural gate for om-cto's gap-analysis batch mode: `bin/gap-pipeline-crosscheck`.
It runs once per batch at the Phase 2→3 transition and cross-checks every `done` story's
`**Upstream pipeline**` field against the run's one-time PR/spec snapshot. Motivation: on the
real ESA run (2026-07-08, 94 stories), Story 3.1 ("Manual KSeF e-invoice export…") failed to
cite `official-modules PR #29` ("…full KSeF feature set integration") despite "KSeF" in both
titles; 18 isolated re-trials never reproduced the miss, so it is rare per-call variance that
recurs at batch scale — the fix is a deterministic orchestrator-side pass, not a prompt change.
Source plan (with frontmatter noting the implementation deviations):
`/Users/maciejgren/Documents/gap-analysis/esa-local-grounding/upstream-pipeline-crosscheck-plan.md`

## Change set

- NEW  `bin/gap-pipeline-crosscheck` — bash wrapper + one awk program (BSD awk / bash 3.2 target)
- NEW  `docs/specs/fixtures/gap-pipeline-crosscheck/{snapshot.md,flagged.md,satisfied.md}`
- EDIT `skills/om-cto/references/gap-analysis-batch.md` — Phase 2 step 7 inserted (old 7 → 8),
       residual-failure paragraph extended, Cross-refs row added, acceptance test 13 added
- EDIT `CHANGELOG.md` — new 1.23.0 entry at top (includes a "Review — fifteen issues" section)
- EDIT `package.json`, `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` — 1.23.0

## Algorithm (as shipped — the script header documents it in full)

Tokenization reuses `bin/gap-validate-finding`'s S012 logic (lowercase, non-alnum split, same
stopword list, length ≥ 4, first-5-char stem) plus one declared addition: an English
function-word stoplist. A stem contributes to a (story, candidate) score only when it is under
the snapshot-frequency cutoff (default max(5, 5% of candidates)) AND central to the story (in
its title, or 2+ occurrences in title+criteria). A pair clears the threshold on ≥2 contributing
stems (`--min-distinctive`, default 2), or on one stem that is unique in the whole snapshot
(df == 1) AND whose original token carries a capital/digit beyond its first character (the
proper-noun gate: `KSeF`, `DataTable` yes; `Fix`, `Magento` no). A story is satisfied when its
citation RESOLVES to any real snapshot entry; it is flagged when it cites nothing while
above-threshold candidates exist (missing) or cites a value found nowhere in the snapshot
(phantom — checked for every done story, even with zero candidates). Report on stdout, summary
on stderr; exit 0 clean / 1 flagged / 2 usage-or-wrong-file (fail closed). `needs-review`
stories are skipped. The whole awk runs under `LC_ALL=C` (multibyte-crash fix).

Three deliberate deviations from the source plan, each empirically forced and recorded in the
script header + CHANGELOG: (1) the plan's snapshot-side-only distinctiveness flagged 74/89 real
stories (~45 candidates each) — replaced by centrality + the proper-noun single-stem path;
(2) a story-side rarity filter was tried and REJECTED because it killed "ksef" itself;
(3) the plan's literal "cited must be above-threshold" satisfaction rule false-flagged all
three stories correctly citing PR #29 — replaced by any-resolving-citation-satisfies.

## What was already reviewed — do not just repeat it

A 5-lens adversarial review (awk correctness, plan fidelity, parsing robustness, doc
consistency, design attack) with per-finding refutation confirmed 15 issues; all were fixed:
phantom-citation hole (now checked unconditionally), CRLF fail-open (both files; `\r` stripped
at the top of the awk program), macOS awk fatal on Polish diacritics (`LC_ALL=C`), proper-noun
gate false premise on title-case PR titles (now requires capital/digit beyond char 1),
`--max-title-freq 0` and empty flag values (now exit 2), unstable tie sort (now insertion
sort), and five stale-doc/wrong-number items. The CHANGELOG 1.23.0 "Review" section lists them.
Highest-value work for you: (a) verify each fix is actually present and correct in the code as
it stands now, (b) hunt for defects INTRODUCED by those fixes, (c) attack angles the first
review did not cover.

## How to verify (expected results as of handoff)

```bash
cd /Users/maciejgren/Documents/om-superpowers
F=docs/specs/fixtures/gap-pipeline-crosscheck

bin/gap-pipeline-crosscheck $F/flagged.md $F/snapshot.md
# exit 1; summary: "4 done stories checked against 13 snapshot candidates … 2 flagged"
# FLAG [3.1] cites: none — ranked: 1. PR #109 [manua, expor]  2. official-modules PR #29 [ksef]
# FLAG [5.2] cites: official-modules PR #77 (open) — NOT FOUND … (phantom), no candidate list

bin/gap-pipeline-crosscheck $F/satisfied.md $F/snapshot.md
# exit 0 — 3.1 cites PR #29 (passes despite higher-scored uncited PR #109);
# 5.2 cites real-but-irrelevant PR #108 (documents the accepted any-resolving-citation limit)

# Binding case against REAL artifacts (snapshot: /tmp/gap-upstream-pipeline-esa-local-grounding.md,
# MD: /Users/maciejgren/Documents/gap-analysis/esa-local-grounding/esa-local-grounding.md).
# Regenerate the reverted copy (Story 3.1's citation → none) if the scratch copy is gone:
awk '/^### Story 3\.1:/{in31=1} in31 && /^- \*\*Upstream pipeline\*\*: official-modules PR #29 \(open\)$/{sub(/official-modules PR #29 \(open\)/,"none"); in31=0} {print}' \
  /Users/maciejgren/Documents/gap-analysis/esa-local-grounding/esa-local-grounding.md > /tmp/esa-rev31.md
bin/gap-pipeline-crosscheck /tmp/esa-rev31.md /tmp/gap-upstream-pipeline-esa-local-grounding.md
# exit 1, 37 flagged; FLAG [3.1] lists exactly one candidate: official-modules PR #29 [ksef]
bin/gap-pipeline-crosscheck /Users/maciejgren/Documents/gap-analysis/esa-local-grounding/esa-local-grounding.md /tmp/gap-upstream-pipeline-esa-local-grounding.md
# exit 1, 36 flagged, and NO "FLAG [3.1]" row — delta vs reverted is exactly Story 3.1

# Edge behaviors: no args / missing file / swapped inputs / --min-distinctive x /
# --max-title-freq 0 / --max-title-freq= → all exit 2 with a concrete message.
# CRLF copies of both fixtures → identical results to LF. A story+title containing "Kraków"
# → matches, no awk crash. Tied candidates → snapshot order preserved.
bash scripts/check-version-sync.sh   # → OK at 1.23.0
```

## Suggested fresh attack surface

- The awk program end to end (the file is ~450 lines, roughly half header comments): parsing
  state machine (`instory`/`incrit` resets on `##`/`###`, `####` stays inside a story),
  FILENAME==ARGV[1] file guard, df/proper marking at candidate parse time, the scoring loop's
  array hygiene across stories, the resolve-vs-flag control flow (the phantom path was
  restructured late — look for paths where `resolved`/`nh`/`cited_kind` interact wrongly).
- Spec-citation resolution: exact basename equality, case-insensitive, with a missing `.md`
  extension as the one tolerated variation. (A second-pass review already caught and fixed a
  substring-containment false resolve — `…outbox.md.bak` passing as real; that repro now
  exits 1 as a phantom.) Try to construct a remaining false resolve or a false phantom (a
  legitimate spec citation shape that fails to resolve).
- `LC_ALL=C` side effects: accented letters become token separators; confirm both corpora
  tokenize identically and nothing else regressed (e.g. `tolower` on non-ASCII in the
  spec-resolution comparison).
- Doc-vs-code truth: every number and behavioral claim in the script header, CHANGELOG 1.23.0,
  and gap-analysis-batch.md step 7 / test 13 should reproduce. The repo has a standing rule
  (see memory/CHANGELOG history) that CHANGELOG claims must reconcile against the actual diff.
- Skill-doc integrity: step renumbering (old step 7 → 8) — search the file and the wider repo
  for references to batch-mode "step 7"/"step 8" that now point at the wrong step.

## Constraints for your report

Rank findings most-severe first. For each: file, one-sentence defect, concrete
failure scenario (inputs → wrong output), and the repro command. Distinguish "defect" from
"documented accepted limit" — the script header's Scope honesty section and step 7's scope
paragraph list the accepted ones (lexical false positives, synonym misses, any-resolving-
citation satisfaction). Re-litigating those without new evidence is noise.
