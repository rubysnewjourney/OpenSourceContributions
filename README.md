# AI301 Open Source contributions

Open source project for CodePath AI 301:
------------------------------------------------------------------------------------------
Current Contribution #4 - Documentation Only
------------------------------------------------------------------------------------------

**Contribution Number**: 4
**Student**: Ruby Khatoon

**Issue**: Add migration guide from standalone MCP servers to relay (valtors/relay #30)

**Status**: Phase 1 & 2 Completed | Guide and evidence finalized | Ready to branch, commit, and open PR

## High-Level Project Summary

Relay is a Go-based, single-binary MCP (Model Context Protocol) server built
by Tamish Mhatre (`valtors/relay`, MIT licensed). It bundles 40 tools across
7 categories (File, Image, PDF, Text, Data, Web, Workflow) into one binary,
so users don't need to install and configure multiple standalone MCP servers
separately. Installable via `npx userelay tui` or `relay init`, which
auto-detects the user's editor (Claude Desktop, Cursor, VS Code) and writes
the config.

## Why I Chose This Issue

Labeled `good first issue` + `documentation` — no code changes required, just
a new markdown file (`docs/migrate-to-relay.md`). Good fit for internalizing
the process and building confidence with the contribution workflow (claiming
an issue, working with a maintainer, opening a PR). This gives exposure
working on various types of issues like bugs, features, and documentation.

## Understanding the Issue

### Problem Description
No guide currently exists to help users migrating from multiple standalone
MCP servers (e.g. a memory server, a fetch server, a screenshot server) over
to Relay's single-binary approach.

### Expected Behavior
A new `docs/migrate-to-relay.md` file showing side-by-side config comparisons
(multiple servers → single Relay entry), a mapping of old standalone-server
tool names to Relay's actual tool names, and before/after Claude Desktop
config examples.

### Current Behavior
No migration documentation exists. Users have to manually figure out the
config change and tool-name equivalents themselves.

### Affected Components
Documentation only (`docs/` directory). No source code changes.

## Reproduction Process

(Docs-issue equivalent of reproduction: verifying the issue's technical
claims against the actual codebase before writing anything.)

### Environment Setup
- **Prerequisites installed:** Git, Go (`go version` → `go1.26.5 windows/amd64`)
- **Services started via Docker:** None — this repo does not use Docker for
  this contribution.
- **Dependencies and hooks:** Built the binary locally with
  `go build -o relay.exe .` in `C:\Users\rubys\Projects\AI301\relay` to
  validate tool names/behavior directly, rather than relying solely on
  source-code review.

### Steps Taken
1. Read the issue (#30) in full, noting three specific tool-name claims to
   verify: "relay memory tools," "relay fetchURL tool," "relay screenshot
   tool."
2. Read `docs/FIRST_PR.md` — confirmed the repo's norm is to comment on an
   issue to claim it (no formal "please assign" process beyond that).
3. Identified the maintainer (`tamish560`) as the sole visible contributor,
   who also authored the issue.
4. Posted a claiming comment on the issue.
5. Forked the repo to `rubysnewjourney/relay` and cloned it locally to
   `C:\Users\rubys\Projects\AI301\relay`.
6. Attempted `go run . tools --json` to get the live tool registry —
   discovered Go was not installed locally.
7. Used `dir tools` to inspect the `tools/` directory structure instead,
   confirming no `memory.go` file exists among the 15 tool-category files
   present.
8. Read `tools/registrations.go` directly (the actual tool-registration
   source) to get the definitive, authoritative tool list.
9. Posted findings as a clarifying comment on the issue, citing
   `tools/registrations.go`; maintainer confirmed all three discrepancies
   and provided the definitive 40-tool list, then assigned the issue.
10. Drafted `docs/migrate-to-relay.md` using only maintainer-confirmed tool
    names, organized into per-category tables with before/after config
    examples.
11. Installed Go locally and built the binary (`go build -o relay.exe .`).
12. Ran `.\relay.exe tools --json` against the live binary and diffed all
    40 entries against the guide's tables — full match on names, categories,
    and per-category counts.
13. Ran `.\relay.exe --help` and found its "Tool categories" listing stale
    across every category (not just the known `web_screenshot` reference),
    confirming `tools --json` / `registrations.go` as the only reliable
    sources.
14. Traced the `ANTHROPIC_API_KEY` gating claim to `doctor.go:327-330`, then
    confirmed it live by running `.\relay.exe doctor` with the key unset.
15. Corrected the `pdf_info` table row in the guide to include `creator` and
    `page dimensions`, per the live tool output's fuller description.

### Reproduction Evidence

**Phase 1 — Initial source-code investigation**
`tools/registrations.go` shows 7 registration functions
(`registerFileTools`, `registerImageTools`, `registerPDFTools`,
`registerDataTools`, `registerTextTools`, `registerWebTools`,
`registerWorkflowTools`), registering exactly 40 tools total — matching the
README's advertised count. No memory-related registration exists anywhere
in the file. The `web` category registers only `web_fetch` and `web_status`
— no screenshot tool.

This directly contradicted three claims in the original issue text:

| Issue claimed | Source shows |
|---|---|
| "relay memory tools" | Does not exist — no memory category/tool anywhere |
| "relay fetchURL tool" | Actual name is `web_fetch` |
| "relay screenshot tool" | Does not exist — `web` category only has `web_fetch`, `web_status` |

Posted these findings as a clarifying comment on the issue, citing the exact
file (`tools/registrations.go`) and asking whether to drop the memory/
screenshot references or document them differently. Maintainer confirmed all
three findings and provided the definitive 40-tool list.

**Phase 2 — Live validation against the built binary**

***Environment Setup (completed)***
- Go installed and verified: `go version` → `go1.26.5 windows/amd64`
- Built the binary from the fork: `go build -o relay.exe .` in `C:\Users\rubys\Projects\AI301\relay`

***Live tool registry validation***
Ran `.\relay.exe tools --json` against the built binary and diffed all 40 entries
against the guide's tool tables — full match on names, categories, and per-category
counts (file 7, image 7, pdf 6, data 4, text 6, web 2, workflow 8), matching the
maintainer's confirmed breakdown.

***`--help` output found unreliable***
Ran `.\relay.exe --help` and found its "Tool categories" listing is stale across
every category, not just the previously-known `web_screenshot` reference (e.g. it
lists 4 image tools where 7 actually exist, and lists tool names like `text_extract`,
`data_yaml` that don't exist in the real registry at all). Confirms `tools --json` /
`registrations.go` as the only reliable sources.

***`ANTHROPIC_API_KEY` gating confirmed***
- Source: `doctor.go:327-330` — checks `os.Getenv("ANTHROPIC_API_KEY")` and reports
  workflow tools as gated on it.
- Live confirmation: ran `.\relay.exe doctor` with the key unset, which output
  `! ANTHROPIC_API_KEY / not set; workflow tools require this key` — matching source
  and matching the guide's claim.

***Other doctor findings (informational, not blocking)***
- `process lock`: `relay.exe` running during doctor check can cause `EPERM` on
  Windows reinstalls until stopped.
- `npm wrapper`: not found in PATH — relevant only if referencing the `npx userelay`
  install path rather than a locally built binary.

## Solution Approach

### Analysis
The issue's tool-name claims appear to be stale relative to the current
codebase (likely written before some tools were renamed or before the
memory/screenshot features were descoped). Rather than assume, verified
directly against source before writing any guide content, since incorrect
tool names in a published migration guide would actively mislead users and
require a maintainer correction later.

### Proposed Solution
Maintainer confirmed all three findings in his reply:
1. Memory tools don't exist — drop from the guide entirely.
2. Fetch tool is named `web_fetch`, not `fetchURL` — use the correct name.
3. Screenshot tool doesn't exist; the `web_screenshot` reference in
   `main.go`'s `printUsage()` help text is a separate, stale bug he'll
   clean up independently.

He provided the full, definitive 40-tool list across all 7 categories to
document.

### Implementation Plan
**Plan:**
- Draft `docs/migrate-to-relay.md` covering: why-consolidate rationale,
  before/after config examples, and a full tool-mapping table per category
  (file, image, pdf, data, text, web, workflow), using only maintainer-
  confirmed tool names. — **Done.**
- Validate the guide against a locally running instance of Relay before
  opening a PR. — **Done**, see Reproduction Evidence Phase 2.
- Create a feature branch (`docs/migrate-to-relay`) on the fork, commit
  `docs/migrate-to-relay.md`, and push. — **Not yet started.**
- Open PR referencing issue #30 once pushed. — **Not yet started.**

**Implement:** [Branch not yet created — pending next step]

**Review:**
-

**Evaluate:**


## Testing Strategy

### Unit Tests
N/A — documentation-only contribution, no code changes.

### Integration Tests
Ran the built binary directly rather than relying on source review alone:
- `.\relay.exe tools --json` — confirmed all 40 tool names, categories, and
  counts in the guide match live output exactly.
- `.\relay.exe --help` — used to cross-check the maintainer's "stale help
  text" comment; found the staleness extends to every category, not just
  the `web_screenshot` reference originally flagged.
- `.\relay.exe doctor` — confirmed the `ANTHROPIC_API_KEY` gating claim
  live, matching the `doctor.go` source check.

## Implementation Notes

The guide is organized as one table per tool category (file, image, pdf,
data, text, web, workflow), each listing tool name and a one-line
description, so a reader can scan for their specific use case without
reading the whole document. Before/after `claude_desktop_config.json`
snippets are shown twice — once generically for "any standalone server,"
and once specifically for the fetch-server case, since that's the most
common single-purpose server in circulation. Workflow tools are called out
separately at the end since they're not a like-for-like replacement for any
standalone server, to avoid implying a false migration mapping there. A
footnote clarifies that the tool list was verified against
`tools/registrations.go` and confirmed with the maintainer, and that
`relay --help`'s own category listing is not a reliable source, so future
readers/contributors don't repeat the same mistake the issue text made.

## Maintainer Feedback

Full comment from `tamish560` (issue #30):

> Hey @rubysnewjourney, thanks for digging in. Good catches all around.
> You're right on all three points:
> 1. memory tools - don't exist. No memory category is registered. Drop it
>    from the guide.
> 2. fetchURL - the actual tool is `web_fetch` under the `web` category.
>    Use that name.
> 3. screenshot - no `web_screenshot` tool is registered. The `web` category
>    only has `web_fetch` and `web_status`. The `web_screenshot` reference
>    in `main.go` printUsage is stale and will be cleaned up separately.
>
> For the guide, document what's actually shipped: file (7), image (6),
> pdf (5), data (4), text (6), web (2), workflow (8). That's 40 tools
> across 7 categories. Go ahead and start on `docs/migrate-to-relay.md`.
> Assigning you now.

(Note: his category counts for image/pdf have small typos — 6/5 listed but
7/6 tools actually named — the tool names themselves match
`registrations.go` exactly, and this was independently re-confirmed against
the live binary in Phase 2.)

## Pull Request

**PR Link:** Not yet opened.

**PR Description:** To be drafted once the branch is pushed.

**Copy of maintainer feedback:** See above.

### Code Changes
N/A — documentation only.

## Learnings & Reflections

### Challenges Overcome

- **Conflicting sources of truth.** The original issue text, the README, and the
  binary's own `--help` output all disagreed with each other on tool names and
  counts. Rather than trust any single source, I worked through them in order of
  reliability: issue text → source code (`tools/registrations.go`) → maintainer
  confirmation → live binary output (`tools --json`) → runtime behavior
  (`relay doctor`). Each layer either confirmed or corrected the one before it,
  which is what caught that `--help`'s "Tool categories" listing was stale across
  every category, not just the one screenshot reference flagged in the issue.

- **No local Go environment initially.** Rather than block on the install, I did
  the first round of verification by reading `tools/registrations.go` directly,
  which let me raise the three tool-name discrepancies to the maintainer before
  I'd even built the binary. Once Go was installed, I repeated the verification
  against the live binary to confirm the source-level findings held up in practice.

- **Validating a claim with no direct CLI path.** There was no `relay call <tool>`
  command to directly test `run_workflow`'s `ANTHROPIC_API_KEY` gating. Instead of
  hand-rolling an MCP JSON-RPC session, I traced the check in `doctor.go` and then
  confirmed it live with `relay doctor`, which is a faster and more reliable path
  to the same evidence.

### What I'd Do Differently Next Time

- Run `relay doctor` and `relay tools --json` as a first step in any Relay
  contribution, before reading source or `--help` — it would have surfaced the
  `--help` staleness and the API key gating immediately, instead of as something
  found while chasing another question.

- Post the source-vs-issue discrepancies to the maintainer earlier in the process,
  as soon as the first mismatch was found, rather than batching all three findings
  into a single comment.

## Resources Used

- Issue: https://github.com/valtors/relay/issues/30
- Repo: https://github.com/valtors/relay
- Fork: https://github.com/rubysnewjourney/relay
- `docs/FIRST_PR.md` (contribution norms — comment to claim an issue)
- `docs/ADDING_A_TOOL.md` (tool categories and registration conventions)
- `tools/registrations.go` (authoritative tool registry — source of truth)
- `main.go` (printUsage help text — found to contain a stale tool reference)
- `doctor.go` (source of the `ANTHROPIC_API_KEY` gating check)
- Official reference servers: https://github.com/modelcontextprotocol/servers
  (used to verify real "before" config examples for `mcp-server-fetch` and
  `@modelcontextprotocol/server-memory`)

------------------------------------------------------------------------------------------
Current Contribution #3 - Bug fix
------------------------------------------------------------------------------------------

**Contribution Number:** *3*

**Student:** Ruby Khatoon

**Issue:** [DataQualityPipeline.warnings leaks state between run() calls · Issue #1 · AshayK003/XadaptiveEDA](https://github.com/AshayK003/XadaptiveEDA/issues/1)

**Status:** Phase1 completed | Phase2 completed | Phase3 completed | Phase 4 compeletd | PR #13 submitted | MERGED — PR #13 merged and Issue #1 closed on first review, no change requests

---

## High-Level Project Summary

XadaptiveEDA is an adaptive exploratory data analysis tool built with Streamlit. It provides intelligent analysis recommendations that learn from user preferences, Plotly visualizations, LLM-powered insights, and natural language queries over uploaded datasets. The component I worked on, `DataQualityPipeline` (in `data_quality.py`), sits between data loading and profiling: it normalizes, validates, and reports on data quality — producing a `QualityReport` with warnings about issues like constant columns, sparse columns, mixed types, and duplicate rows.

## Why I Chose This Issue

- Labeled `bug` + `good first issue` and well specified: the maintainer pointed to the exact line (`data_quality.py`, `__init__`) and even suggested the fix.
- It matched my process: small enough to reproduce and verify end-to-end, but real enough to matter — a user uploading a second dataset would see an incorrect quality report contaminated by the first dataset's warnings.
- It turned out to be a stronger learning opportunity than it looked: my investigation revealed the suggested one-line fix was incomplete (see Solution Approach), which led to a scope discussion with the maintainer and an expanded, approved fix.

---

## Understanding the Issue

### Problem Description

`DataQualityPipeline.__init__` sets `self.warnings: list[str] = []` once, but `run()` never resets it. Calling `run()` multiple times on the same pipeline instance carries warnings from earlier datasets into later reports.

During investigation I found a second, related defect the issue did not mention: at the end of `run()`, `report.warnings = self.warnings` assigns the **same list object** to the report rather than a copy. Every report returned by the same pipeline instance shares one list — so a report returned from an *earlier* `run()` call silently keeps mutating as later runs append warnings.

### Expected Behavior

- Each `run()` call starts with an empty warnings list.
- Two consecutive runs on the same instance do not accumulate warnings.
- A returned `QualityReport` is an independent snapshot — later runs must not retroactively change it.

### Current Behavior (before fix)

- Run 2's report contained run 1's warnings even when run 2's data was clean.
- `report1.warnings is report2.warnings` evaluated to `True` — both reports pointed at the identical list object, so `report1` mutated after it had already been returned.

### Affected Components

- `data_quality.py` → `DataQualityPipeline.__init__` (line ~54) and `DataQualityPipeline.run()` (reset missing at top; aliasing at the `report.warnings` assignment near the end)
- Consumers: `cleanse()` convenience function and `DataProcessor.cleanse` (wraps the pipeline); the Streamlit app's per-upload quality report display
- `test_data_quality.py` (new Test 13 added)

---

## Reproduction Process

### Environment Setup

**Prerequisites installed:**
- Python 3.14.4 (repo requires 3.10+)
- Git for Windows, PowerShell

**Services started via Docker:** N/A — the project has no Docker; it runs directly via Python/Streamlit.

**Dependencies and hooks:**
- Cloned **my fork** (not upstream, since I would be pushing branches): `git clone https://github.com/rubysnewjourney/XadaptiveEDA.git`
- Added `upstream` remote pointing at `AshayK003/XadaptiveEDA` and verified with `git remote -v`
- Created and activated a virtual environment (`python -m venv venv`); resolved a Windows PowerShell execution-policy block with `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` (process-scoped only — resets when the terminal closes)
- `pip install -r requirements.txt` — verified via exit code `0` and `python -c "import pandas, numpy; print('OK')"`
- **Deliberately skipped:** `streamlit run app.py` (the fix is testable via the plain-script test suite; the UI is not involved) and the optional Ollama/LLM `.env` setup (unrelated to the data quality pipeline). Scoped the environment to exactly what the task required.
- Synced with upstream before branching: `git fetch upstream` + `git rev-list --count main..upstream/main` → `0` (already current)

### Steps to Reproduce

1. Create a branch: `git checkout -b feature/fix-warnings-state-leak` (naming per the repo README's `feature/...` convention — this repo has no separate CONTRIBUTING.md; guidelines live in the README).
2. Write `repro_issue_1.py` (kept untracked — evidence artifact, not part of the fix):
   - Instantiate **one** `DataQualityPipeline`.
   - Run 1: a DataFrame designed to trigger warnings (`{'a': [1, 1, 1]}` — constant column + duplicate rows). This matters: the issue's steps don't specify data, but the leak is only *visible* if run 1 actually produces warnings.
   - Run 2: clean data (`{'b': [1, 2, 3]}`) that should produce zero warnings.
   - Check 1: does run 2's report contain run 1's warnings?
   - Check 2: `report1.warnings is report2.warnings` — do the reports share one list object?
3. Run `python repro_issue_1.py`.

### Reproduction Evidence

**Before fix:**

```
Run 1 warnings: ['1 constant column(s): a', 'Found 2 duplicate row(s)']
Run 2 warnings: ['1 constant column(s): a', 'Found 2 duplicate row(s)']
BUG 1 (state leak):      REPRODUCED
BUG 2 (report aliasing): REPRODUCED
```

Baseline test suite before any change: **12/12 passing** (`python test_data_quality.py`).

---

## Solution Approach

### Analysis

Reading `data_quality.py` in full confirmed `self.warnings` is the **only** instance attribute set in `__init__` — so no other state leaks. But the file also revealed why the bug is worse than reported: `report.warnings = self.warnings` aliases the shared list into every report. The recommended reset alone would stop accumulation going forward, but would not protect a caller holding a reference to an earlier report.

Reading `test_data_quality.py` explained why the bug shipped unnoticed: every one of the 12 existing tests calls the `cleanse()` convenience function, which constructs a **fresh** `DataQualityPipeline` per call. The suite is structurally incapable of exercising instance reuse — the exact scenario that triggers the bug.

### Proposed Solution

Two changes, both discussed with and approved by the maintainer before implementation:

1. `self.warnings = []` as the first statement of `run()` — placed **above** the `if df.empty:` early return, so even the empty-dataframe path starts clean.
2. `report.warnings = list(self.warnings)` — a shallow copy is sufficient because the list contains immutable strings; each report becomes an independent snapshot.

### Implementation Plan

**Plan:**
1. Sync fork with upstream; branch `feature/fix-warnings-state-leak`
2. Reproduce both defects with a repeatable script; capture "before" evidence
3. Apply both fixes; validate via `git diff` before retesting
4. Re-run the identical repro script; capture "after" evidence
5. Re-run full `test_data_quality.py`; add Test 13 matching existing conventions
6. Grep all other test suites for `data_quality` imports; run any that depend on it
7. Commit (imperative style per repo README), push to fork, open PR, tag maintainer

**Implement:**
- Branch: `feature/fix-warnings-state-leak`
- Commit: `fab3e7d` — "Fix warnings state leak and report aliasing in DataQualityPipeline" (2 files changed, 16 insertions, 2 deletions)
- PR: https://github.com/AshayK003/XadaptiveEDA/pull/13

**Review:**
- Merged on first review with no change requests. Maintainer's review: the fix pattern was called "textbook" (reset before computation, snapshot on assign) and Test 13 was noted as cleanly covering both the leak and aliasing cases.

**Evaluate:**
- Same reproduction script, before vs. after — both defects gone, run 1 behavior unchanged
- 13/13 tests passing; zero regressions in dependent suites

### Testing Strategy

**Unit Tests**
- New **Test 13** in `test_data_quality.py`, matching the file's existing plain `print`/`assert` script style (no pytest — "follow existing code style" per README). Critically, it instantiates `DataQualityPipeline()` directly and reuses it across two `run()` calls — the only structure that can catch this bug. Asserts:
  1. Run 1 produces warnings (guards against a silently useless test)
  2. Run 2's report has zero warnings (reset works)
  3. `report1.warnings is not report2.warnings` (aliasing fixed)
  4. `report1.warnings` is unchanged after run 2 (earlier reports no longer retroactively mutated)
- Full suite after fix: **13/13 passing**.

**Integration Tests**
- No CI in the repo. Verified downstream consumers via `Select-String` across all other test files for `data_quality|DataQualityPipeline|cleanse`: only `test_phase1.py` (imports `cleanse`, `QualityReport`; exercises `DataProcessor.cleanse` as a regression check) and `test_phase2.py` (imports `QualityReport`) touch the changed module. Ran both — **all passing**, including the `DataProcessor.cleanse: OK` regression check. Remaining suites confirmed not to import the module. (Manual UI check via Streamlit — uploading two datasets back-to-back — noted as an optional further integration step.)

**After-fix reproduction evidence:**

```
Run 1 warnings: ['1 constant column(s): a', 'Found 2 duplicate row(s)']
Run 2 warnings: []
BUG 1 (state leak):      not present
BUG 2 (report aliasing): not present
```

(Run 1 still producing its own warnings confirms the fix removed the leakage without breaking warning collection.)

---

## Implementation Notes

- The diff was validated with `git diff` **before** retesting — exactly two hunks, both in `data_quality.py`, correct placement and indentation confirmed against a checklist.
- Files were staged explicitly by name (`git add data_quality.py test_data_quality.py`) rather than `git add .`, keeping the untracked `repro_issue_1.py` evidence script out of the commit — per the repo's "keep PRs focused" guideline.
- Commit message uses this repo's imperative, no-prefix style (README example: `Add amazing feature`) — deliberately different from the `feat:` conventional-commits style my previous project (mini-agent-cli) used. Convention is per-repo, not universal.
- Verified no `PULL_REQUEST_TEMPLATE.md` exists (checked root, `docs/`, and hidden `.github/` — the three locations GitHub supports) before writing a freeform PR description. This is the first PR on the repository.

## Maintainer Feedback

**PR Link:** https://github.com/AshayK003/XadaptiveEDA/pull/13

**PR Description:** Structured as Fixes #1 → What was happening → Changes → Testing → review tag. The `Fixes #1` linkage worked as intended — Issue #1 was automatically closed when the PR merged.

**Copy of the maintainer feedback:**

1. On my claim comment: *"Assigned! Go ahead — your plan looks solid. Let me know if you hit anything unexpected."*
2. On my scope-check comment (raising the aliasing finding): *"Good catch — that's a real gap and closely related to the original issue. Since fixing `self.warnings = []` at the start of `run()` would make the aliasing less visible but wouldn't protect a caller holding onto a report reference, both fixes belong in the same PR. Go ahead and include both: 1. `self.warnings = []` at the start of `run()` 2. `report.warnings = list(self.warnings)` (copy, not alias). Makes the fix complete. Tag me when the PR is up and I'll review it."*
3. PR review (merge): *"Reviewed and merged. The two-line fix is textbook — reset before computation, snapshot on assign. Test 13 covers both the leak and aliasing cases cleanly. Good work. Thanks for the contribution!"* — Pull request successfully merged and closed.

## Code Changes

```diff
diff --git a/data_quality.py b/data_quality.py
@@ -55,6 +55,7 @@ class DataQualityPipeline:
     def run(self, df: pd.DataFrame, skip_name_normalization: bool = False) -> tuple[pd.DataFrame, QualityReport]:
         """Run full pipeline. Returns (cleaned_df, QualityReport)."""
+        self.warnings = []
         if df.empty:
             report = QualityReport(warnings=["Empty dataset — no processing applied"])
             return df, report
@@ -104,7 +105,7 @@ class DataQualityPipeline:
         self._compute_quality_metrics(df, report)
         self._collect_warnings(report)
-        report.warnings = self.warnings
+        report.warnings = list(self.warnings)
         return df, report
```

Plus Test 13 in `test_data_quality.py` (reuses a single pipeline instance across two `run()` calls; four asserts covering the leak, the aliasing, and both guard conditions).

---

## Learnings & Reflections

- **The issue description is a starting point, not the full spec.** The suggested one-line fix was correct but incomplete — reading the whole `run()` method surfaced the aliasing defect. Verifying against source code rather than trusting the write-up changed the scope of the fix.
- **Test suites can have structural blind spots.** All 12 existing tests went through `cleanse()`, which creates a fresh pipeline per call — so the suite *couldn't* catch an instance-reuse bug no matter how many tests were added in that style. The new test had to deliberately break from the file's dominant pattern (while matching its style) to cover the gap.
- **Ask before expanding scope — it pays off.** Instead of silently bundling the second fix or shipping it with an after-the-fact explanation, I raised it on the issue and asked whether it belonged in the same PR. The maintainer not only approved but supplied the strongest justification himself (a reset alone "wouldn't protect a caller holding onto a report reference").
- **A repeatable reproduction script beats an interactive shell.** Running the identical script before and after the fix produced a clean, apples-to-apples evidence pair.
- **Check downstream consumers, don't assume.** A quick `Select-String` across the other six test files found two suites importing from the changed module — both then run and confirmed green, including a dedicated `DataProcessor.cleanse` regression check I wouldn't have known existed otherwise.

### Challenges Overcome

- **Windows PowerShell execution policy** blocked venv activation; resolved with a process-scoped bypass (no system-wide security change).
- **Reproduction design subtlety:** the issue's repro steps don't specify input data, but the leak is only observable if run 1 actually generates warnings — the repro had to construct warning-triggering data deliberately.
- **Convention differences between repos:** branch naming (`feature/` vs `feat/`) and commit style differ from my previous contribution's repo; each was verified against *this* repo's README rather than assumed.

### What I'd Do Differently Next Time

- Check for sibling test suites that import the target module at the **start** of the investigation, not just before opening the PR — it would have informed the testing plan earlier.
- Preview PR description Markdown before submitting: `__init__` rendered as bold "init" because unescaped double underscores are Markdown bold syntax (should have been backticked).

## Resources Used

- Issue #1 and its comment thread (maintainer's fix recommendation and scope-approval)
- Repo README (Contributing section: fork/branch/PR workflow, `feature/...` naming, code style, "add tests" and "keep PRs focused" guidelines; Quick Start for environment setup)
- Source files read in full before changing anything: `data_quality.py`, `test_data_quality.py`
- `git` (remotes, fetch/rev-list upstream sync, diff validation, explicit staging), PowerShell (`Set-ExecutionPolicy -Scope Process`, `Select-String` for dependency grep, `Get-ChildItem -Force` for PR-template check)
- GitHub docs (PR template locations: root, `docs/`, `.github/` — used to verify none exists in this repo)
- Python interactive shell + `repro_issue_1.py` for reproduction and identity (`is`) verification

------------------------------------------------------------------------------------------
Current Contribution #2 - New Feature
------------------------------------------------------------------------------------------

**Contribution Number**: 2

**Student**: Ruby Khatoon

**Issue**: [feat: add deleteFile tool #14](https://github.com/HoussemEddineChaouch/mini-agent-cli/issues/14)

**Status**: Phase1 completed | Phase2 Completed | Phase 3 Completed | Phase 4 completed | PR submitted, maintainer feedback addressed | **Merged Completed**

## High-Level Project Summary

`mini-agent-cli` is a lightweight CLI AI agent built in Node.js that uses Google Gemini through a simple tool-calling loop. It can already read files, write files, list directories, fetch URLs, and run shell commands — each one wired up as a "tool" the LLM can call when it decides it needs to. This contribution adds a `deleteFile` tool, so the agent can finally delete a file, not just read/write/list them.

## Why I Chose This Issue

It was labeled `good first issue` and nobody had picked it up yet. I liked that it fit neatly into a pattern the codebase already had going — one function in `functions.js`, one entry in `tools.js` — so it felt like a first manageable contribution to this repo, but still one where I'd actually have to understand how the pieces fit together (the tool registry, the agent loop, the error handling) instead of just tweaking a string somewhere.

---

## Understanding the Issue

### Problem Description
The agent could read, write, and list files, and pull content from URLs — but had no way to delete a file. Basic gap in file management.

### Expected Behavior
A `deleteFile` tool the agent can call with a path, which deletes the file and returns a clear message either way — success, or "that file doesn't exist" — matching how the other tools already behave.

### Current Behavior
Nothing. There was no delete option at all; you'd have to leave the agent entirely to remove a file.

### Affected Components
- `src/functions.js` — where the actual tool logic lives
- `src/tools.js` — the registry that tells Gemini what tools exist and how to call them
- `src/agent.js` — has a hardcoded list of examples that teaches Gemini how to format each tool call
- `README.md` — the tools table in the docs

---

## Reproduction Process

### Environment Setup

**Prerequisites installed:**
- Node.js (v24.15.0) and npm
- Git

**Services started via Docker:** None needed — it's just a local Node CLI, no Docker involved.

**Dependencies and hooks:**
```bash
npm install
```
Had to run this as `cmd /c "npm install"` instead of directly, because PowerShell's execution policy blocked `npm.ps1` from running on my machine (Windows).

### Steps to Reproduce
This one was a feature request, not a bug, so there wasn't really anything broken to reproduce. Instead I spent time getting the app running locally and actually reading the existing code before touching anything:
1. Cloned the repo, ran `npm install`.
2. Read through `functions.js`, `tools.js`, and `agent.js` to figure out the pattern other tools followed — how each function handles errors, what it returns, and how `agent.js` calls `tool.func(...)` with its own try/catch wrapped around it.
3. Checked `package.json` and confirmed there's no test framework set up in this project at all.

### Reproduction Evidence
Not applicable here since it's a feature, not a bug — the closest thing to "evidence" was just confirming the existing tools' behavior firsthand before writing anything new.

---

## Solution Approach

### Analysis
I read exactly how `writeFile` handles errors — it only catches `ENOENT` and lets everything else throw. At first I wasn't sure if that was too minimal, but then I found that `agent.js` already wraps every tool call in its own try/catch and turns any thrown error into a friendly "Tool failed: ..." message. So the two-layer design is intentional: each tool only needs to handle its one expected failure case, and the agent handles the rest.

### Proposed Solution
Add a `deleteFile(path)` function using `fs.unlinkSync`, built to match `writeFile` almost exactly — plain success string, friendly message for `ENOENT`, everything else thrown — plus register it in `tools.js` the same way the other tools are registered.

### Implementation Plan

**Plan:**
1. Write `deleteFile` in `functions.js`, register it in `tools.js`.
2. Test it locally with a quick throwaway script — no framework, no API key needed.
3. Branch, commit, and open a PR following `CONTRIBUTING.md`.

**Implement:**
- Branch: `feat/add-delete-file-tool`
- First commit: [`0b1aace`](https://github.com/rubysnewjourney/mini-agent-cli/commit/0b1aaceb29b79fe69a3c5e3734c5abe0c915aa91) — `feat: add deleteFile tool`
- Follow-up commit: `docs: add deleteFile example to agent.js and README tools table`
- Fix commit: `fix: restore runCommand formatting and reorder module.exports`

**Review:**

* Opened PR #30 against `HoussemEddineChaouch:main`
* Round 1 — another contributor, `@abusnitsky`, suggested I follow the repo's actual PR template instead of the custom description I'd written; the maintainer agreed
* Round 2 — maintainer asked for two more things: a `deleteFile` example in `agent.js`, and a row in the README's tools table

**Evaluate:**
- Both rounds of feedback are addressed and pushed. Maintainer reviewed and merged the PR.

---

## Testing Strategy

### Unit Tests
No test framework exists in this repo, so I wrote a small standalone Node script that called `deleteFile` directly, no API key or agent loop involved:
- **Success case:** made a temp file, deleted it, checked I got back `"File deleted successfully"` and that the file was actually gone.
- **Missing file:** ran it against a path that didn't exist — got the friendly "does not exist" message back instead of a crash.
- **Directory case:** ran it against a folder on purpose, just to see what happens. It threw `EPERM` (that's the Windows behavior — on Linux/macOS it'd be `EISDIR` instead), which is exactly what I wanted since I'd decided *not* to special-case directories and just let the agent's existing error handling catch it.

### Integration Tests
Didn't get to run the agent end-to-end with Gemini since I don't have a `GEMINI_API_KEY` yet. I was upfront about that in the PR rather than pretending it was tested.

---

## Implementation Notes

- Stuck closely to the existing style rather than doing my own thing — same error handling pattern as `writeFile`, same description style in `tools.js`.
- Deliberately didn't add a testing framework even though the repo has none — that felt like a bigger decision than this issue called for, so I left it out.
- Partway through review, another PR (`runCommand`, from a different contributor) got merged into `main` before mine, which caused a real merge conflict in `functions.js`. I merged `main` into my branch and resolved it by hand rather than force-pushing over their work.
- While resolving that conflict, my editor quietly reformatted some blank lines inside the unrelated `runCommand` function and I accidentally left `deleteFile` out of order in the exports line. Caught both by diffing carefully afterward and fixed them in a separate follow-up commit so the final PR diff stayed clean.

---

## Maintainer Feedback

### Pull Request

**PR Link:** [#30 — feat: add deleteFile tool](https://github.com/HoussemEddineChaouch/mini-agent-cli/pull/30)

**PR Description** (rewritten to follow the repo's `PULL_REQUEST_TEMPLATE.md`):
> **What does this PR do?**
> Adds a `deleteFile` tool that removes a file from disk via `fs.unlinkSync`, matching the existing `readFile`/`writeFile`/`listDir` pattern in `functions.js` and `tools.js`. Returns "File deleted successfully" on success, a friendly message for missing files (ENOENT), and lets other errors (e.g. attempting to delete a directory) propagate up to be caught by `agent.js`'s existing error handling — consistent with `writeFile`'s approach.
>
> **Type of change:** New feature / tool
>
> **How to test it:** Ran a standalone script (no framework, no API key needed) calling `deleteFile` directly from `functions.js` — success case, missing-file case, and directory case (confirms `EPERM` on Windows, surfaced via `agent.js`'s existing try/catch).
>
> **Related issues:** Closes #14

**What the maintainer actually said:**
> First, `@abusnitsky` (another contributor) chimed in suggesting I follow the repo's PR template. The maintainer agreed:
>
> *"Hey @rubysnewjourney, great work! 🙌 Could you update the PR description to follow the repo's PR template? (What does this PR do / Type of change / How to test it) Then I'll review and merge!"*
>
> After I updated it, he came back with two more asks:
>
> *"Hey @rubysnewjourney, great work! One thing, agent.js has an examples list that teaches Gemini how to call each tool. deleteFile needs one too otherwise Gemini might not use it correctly. Also add deleteFile to the README tools table and fill in the PR description template. Fix those and this is ready to merge!"*
> **Outcome**: PR #30 was merged by the maintainer, closing issue #14.

### Code Changes
- `src/functions.js` — added `deleteFile(path)`
- `src/tools.js` — registered `deleteFile`
- `src/agent.js` — added a `deleteFile` example to the Examples list
- `README.md` — added `deleteFile` to the tools table
- One merge commit to reconcile the conflict with `runCommand`, plus a small fix commit cleaning up formatting/ordering that got messed up during that merge

---

## Learnings & Reflections

### Challenges Overcome
- PowerShell blocked `npm install` outright — got around it with `cmd /c "npm install"` instead of changing system-wide settings.
- Realized partway through that `origin` was pointed at the original repo, not my fork — forked it, added a second remote (`fork`), and pushed there instead. Caught this before it caused a failed push, not after.
- A text editor silently saved my test script as `test-deleteFile.js.txt` instead of `.js`, which gave a confusing "module not found" error until I actually listed the directory and saw the real filename.
- Copied a PR description template from someone else's already-merged PR at first, which turned out to be the wrong move — went and found the repo's actual `PULL_REQUEST_TEMPLATE.md` file instead once a reviewer pointed it out.
- Had to resolve a real merge conflict when another tool (`runCommand`) got merged into `main` while my PR was still open — first actual merge conflict I've walked through carefully rather than just accepting one side.
- Spent longer than I'd like admitting on a git diff command that kept showing "no changes" even after I'd clearly edited the file — turned out I was comparing two commits instead of my actual uncommitted changes. Switching the command is what finally showed the truth.

### What I'd Do Differently Next Time
- Check for a PR template file right at the start, before writing any PR description, instead of finding out the hard way after a reviewer flags it. Eventhough I had read it before and knew I should follow the expected format but this was a total miss and realized after submitting it and I should have changed it right after.
- After resolving any merge conflict, diff the whole file again — not just the part that conflicted — to catch anything my editor quietly changed on the side.
- If a command keeps giving me the same "nothing changed" result more than once, stop and try a different way of checking instead of running it again hoping for something different.

---

## Resources Used
- [Issue #14](https://github.com/HoussemEddineChaouch/mini-agent-cli/issues/14)
- [Pull Request #30](https://github.com/HoussemEddineChaouch/mini-agent-cli/pull/30)
- The repo's own `README.md`, `CONTRIBUTING.md`, and `.github/PULL_REQUEST_TEMPLATE.md`
- `src/functions.js`, `src/tools.js`, `src/agent.js` — read directly rather than assumed
- Node.js docs and a few GitHub issues on how `fs.unlinkSync` behaves differently on Windows (`EPERM`) vs. POSIX (`EISDIR`) when you try to delete a directory
- Git docs on merging, remotes, and forks

**Lighting Talk presentation to the Class AI301 on 07/08/2026:**
I also got a chance to do the lighting talk presentation to the class 07/08/2026. 
I reviewed my presenttion deck with teh faculty member and also sent her the practice audio for the feedback. 

Message for the open Issue: @alex-ter -Hello there, I noticed you've been assigned this for a couple of weeks — are you still actively working on it? No pressure at all, just wanted to check before doing any investigation myself. Happy to collaborate or wait if you're close to a PR. Thanks for your time.

**Background about my open source issue:**
## CVE Binary Tool — High Level

**What it is:**
A free, open source security tool that scans software to find known vulnerabilities. You point it at a binary, a folder, or a software bill of materials (SBOM), and it tells you which components have known CVEs (Common Vulnerabilities and Exposures) — essentially a security audit tool for your software supply chain.

**Who uses it:**
Developers and security teams who want to catch vulnerable dependencies before they ship, typically as part of a CI/CD pipeline. It's backed by the Open Source Security Foundation (OpenSSF) and used by enterprise Linux environments.

**How it works at a glance:**
It downloads vulnerability data daily from sources like NVD (National Vulnerability Database), RedHat, OSV, and GitLab Advisory Database, then cross-references that data against whatever you're scanning.

---

## Issue #5689 — What We're Trying to Fix

**The short version:**
There's a test file called `test_json.py` that is supposed to run as part of the project's automated test suite, but it never actually runs due to a misconfiguration — and even if it did run, it would fail for reasons unrelated to the tool itself.

**How it got broken:**
Two separate, unrelated code changes at different points in time each added a condition that must be true for the tests to execute — one requiring `LONG_TESTS=1` and another requiring `EXTERNAL_SYSTEM=1`. Neither change was wrong on its own, but together they created a combination that no CI workflow actually sets simultaneously, so the tests silently get skipped every single run without anyone noticing.

**Why it matters:**
A test that never runs gives a false sense of security — you think you have coverage but you don't. It's also confusing for contributors who see the file and assume it's part of the active test suite when it effectively isn't.

**What the test actually does:**
It validates cached NVD tar.gz data files against their schema and metadata — essentially checking that the downloaded vulnerability data is well-formed. However, NVD has had widespread data corruption issues for a while, meaning even if the test ran, it would produce failures caused by NVD's own bad data rather than any actual bug in the tool.

**The two possible fixes being discussed:**
- Fix the guard conditions so the test actually runs in the `linux-mayfail` CI job that was specifically designed for flaky, network-dependent tests
- Remove `test_json.py` entirely since the underlying NVD data it validates is too unreliable to test against meaningfully

The maintainer and a core contributor have both leaned toward removal, but the primary maintainer (`terriko`) hasn't formally signed off yet — which is why confirming direction before writing any code is the right first step.



------------------------------------------------------------------------------------------
Previous Contribution #1 - Bug fix
------------------------------------------------------------------------------------------

# Contribution [1]: [Add sortBy and filter to User.checkouts]

**Contribution Number:** 1  
**Student:** Ruby Khatoon  
**Issue:** https://github.com/saleor/saleor/issues/12520  
**Status:** Phase IV — Complete | PR Submitted | Tests Written | Maintainer Feedback Addressed | Respond to maintainer feedback | Push revisions | Merging Conflict Resolved | Awaiting Final Approval

**Latest Status:** - I had an update on my contribution 1. I had hear back on a conflict with merging and I resolved this week. The details are avialable bottom section of the Phase4 of the contribution 1.

**Background about my open source issue:**

**Saleor — High-Level Project Summary**

Repository: https://github.com/saleor/saleor
 Stars: 22.9k | Forks: 6k | Language: Python 99.2%
 License: BSD-3-Clause (open source, free to use)

**What Is Saleor?**
Saleor is an open source, headless, API-only e-commerce platform built for developers. Rather than a traditional all-in-one shop platform (like Shopify or WooCommerce), Saleor exposes everything through a GraphQL API — the frontend, mobile app, or dashboard connects to it however the developer chooses.
Think of it like this:
Traditional e-commerce = the store, the backend, and the checkout are all bundled together
Saleor = just the backend and API — developers build their own storefront on top of it using any technology they want

**Core Technology Stack**
Layer
Technology
Language
Python 3.12
Web framework
Django
API
GraphQL (via Graphene-Django)
Database
PostgreSQL
Cache / queues
Redis (Valkey)
Task runner
Celery
Package manager
uv
Testing
pytest

**Key Design Principles**
1 — GraphQL only Every single interaction with Saleor — reading products, placing orders, managing customers — goes through a GraphQL API. There is no REST API, no fragmentation.
2 — Headless and API-only Saleor has no built-in storefront. The frontend is completely decoupled — developers build it with whatever stack they prefer (Next.js, React, mobile apps, etc.).
3 — Technology agnostic No plugin architecture that locks you into Python or Django. Extensions are built as independent apps that communicate via webhooks — they can be written in any language.
4 — Native multichannel Every aspect of the store — pricing, currencies, stock, products — can be configured per channel. A single Saleor instance can run multiple stores in different countries with different currencies simultaneously.
5 — Open source, single version No paid "enterprise edition" with hidden features. Everything is in the public repo under BSD-3-Clause license.

**What Saleor Can Do (Feature Set)**
**Area**    **  Capabilities  **
Products      Rich content model, variants, attributes, categories, collections 
Orders        Flexible order model, split payments, multi-warehouse, returns
Checkout      Advanced tax and payment options, discounts, promotions
Customers     Order history, preferences, addresses
Payments      Multi-gateway, extensible payment API, any payment method
Promotions    Sales, vouchers, cart rules, gift cards
Channels      Per-channel pricing, currency, stock, and product visibility
Content       CMS for product and marketing content
Translations  Fully translatable catalog
Apps          Extend via webhooks, metadata, iframe dashboard extensions
Dashboard     Separate admin UI repo (saleor/saleor-dashboard)

**How Saleor Is Structured (Code)**

**All application code lives in the saleor/ directory, organized by domain:**

saleor/
├── graphql/           ← All GraphQL schema, types, resolvers, filters, sorters
│   ├── account/       ← User, customer, staff queries & mutations
│   ├── checkout/      ← Checkout queries & mutations  ← YOUR CONTRIBUTION IS HERE
│   ├── order/         ← Order queries & mutations
│   ├── product/       ← Product catalog
│   └── ...
├── account/           ← Django models for users
├── checkout/          ← Django models for checkouts
├── order/             ← Django models for orders
├── payment/           ← Payment logic
└── ...

**Each app follows a consistent pattern:**

schema.py — GraphQL queries and mutations

types.py — GraphQL type definitions

filters.py — filter input classes

sorters.py — sorting input classes

dataloaders.py — efficient batch data loading

tests/ — pytest test suite

**The Saleor Ecosystem**
Saleor Core (this repo) is just one piece of a larger ecosystem:

**Component          Repo                        Purpose**
Saleor Core      saleor/saleor                The GraphQL API backend (this repo)
Dashboard        saleor/saleor-dashboard      React admin UI for store management
Storefront       saleor/storefront            Example Next.js storefront
Platform         saleor/saleor-platform       Docker Compose to run all components together
CLI              saleor/saleor-cli            Command line tool for developers
Docs             saleor/saleor-docs           Documentation site

**Where Your Contribution Fits**
Your PR #19363 touches saleor/graphql/account/types.py — right at the heart of the GraphQL layer. Specifically:
The User type definition in the GraphQL schema
The checkouts field on the User type
The resolver that fetches checkouts for a user
By adding sortBy and filter to User.checkouts, you improved API consistency across the entire platform — making the developer experience more predictable for anyone building on top of Saleor's GraphQL API.

**Community & Scale**
22,900+ GitHub stars — one of the most popular open source e-commerce projects

6,000+ forks — widely adopted and extended

22,464 commits — actively maintained since 2013

186 open issues — healthy backlog of improvements

Active Discord community at saleor.io/discord

Saleor Cloud — hosted version for teams that don't want to self-host


## Why I Chose This Issue

I chose this issue because I immediately understood the confusion it creates for developers: the global checkouts query supports filtering and sorting, but the User.checkouts field does not. This inconsistency makes the API harder to use and breaks the expectation that similar fields behave similarly. Since I've been learning how GraphQL schemas are structured and how resolvers work, this felt like a great opportunity to apply that knowledge in a real project.

This issue also appealed to me as a first contribution because it is small enough to be approachable, yet meaningful enough to improve the clarity and usability of the API. By resolving this ambiguity, I can help make the developer experience more consistent while learning more about Saleor's schema design patterns, filter/sort utilities, and resolver logic.

---

## Understanding the Issue

### Problem Description

The root-level `checkouts` query in the GraphQL API already supports both `sortBy` and `filter` arguments. However, the `User.checkouts` field does not expose these same arguments. This creates an inconsistency in the API: developers can sort and filter checkouts globally, but cannot apply the same operations when querying checkouts for a specific user.

The underlying queryset logic appears reusable, so adding these arguments should be straightforward and backward-compatible.

### Expected Behavior

- Extend the `User.checkouts` field to accept the same `filter` and `sortBy` arguments used by the root-level `checkouts` query.
- Update the resolver to pass these arguments into the existing checkout queryset logic.
- Add tests to confirm sorting and filtering work correctly on `User.checkouts`.
- Update the GraphQL schema file to reflect the new arguments.

### Current Behavior

The `checkouts` root query (in `graphql/checkout/schema.py`) already implements filtering and sorting through existing filtersets and sorters. The `User.checkouts` field (in `graphql/account/types.py`) currently exposes pagination arguments but does not accept `filter` or `sortBy`.

### Affected Components

The API files are in `saleor/graphql/` directory. Key files for this issue:

- `saleor/graphql/account/types.py` — where `User.checkouts` is defined
- `saleor/graphql/checkout/schema.py` — reference for how root `checkouts` implements filter/sort
- `saleor/graphql/checkout/filters.py` — `CheckoutFilterInput` (reusable)
- `saleor/graphql/checkout/sorters.py` — `CheckoutSortingInput` (reusable)

---

## Reproduction Process

### Environment Setup

**Prerequisites installed:**
- `libmagic` and `uv` via Homebrew
- Python 3.12 via `uv python install 3.12`

**Services started via Docker:**
```bash
cd .devcontainer
docker compose up db dashboard cache mailpit
```

**Dependencies and hooks:**
```bash
uv sync
uv run pre-commit install
cp .env.example .env
uv run poe migrate
uv run poe populatedb
uv run poe start
```

Server runs at `http://localhost:8000/graphql/`

### Steps to Reproduce

**Step 1 — Verify root `checkouts` query accepts sortBy and filter (works):**
```graphql
{
  checkouts(
    first: 5,
    sortBy: { field: CREATION_DATE, direction: ASC }
  ) {
    edges { node { id created } }
  }
}
```
✅ Returns results successfully.

**Step 2 — Verify `User.checkouts` does NOT accept these arguments (the bug):**
```graphql
{
  user(id: "VXNlcjox") {
    checkouts(
      first: 5,
      sortBy: { field: CREATION_DATE, direction: ASC }
    ) {
      edges { node { id } }
    }
  }
}
```
❌ Error: `Unknown argument "sortBy" on field "User.checkouts"` — bug confirmed.

**Step 3 — Confirm in source code:**

In `saleor/graphql/checkout/schema.py`, the root `checkouts` uses `FilterConnectionField` with `filter` and `sort_by` arguments.

In `saleor/graphql/account/types.py`, `User.checkouts` uses a plain `ConnectionField` with no `filter` or `sort_by` — this is the root cause.

### Reproduction Evidence

- **Commit showing reproduction:** [Link to be added]
- **My findings:** The fix requires changing `ConnectionField` to `FilterConnectionField` on `User.checkouts` and wiring in the existing `CheckoutFilterInput` and `CheckoutSortingInput` classes that are already defined and used by the root query.

---

## Solution Approach

### Analysis

The root cause is in `saleor/graphql/account/types.py`. The `User.checkouts` field uses a plain `ConnectionField`, which only supports pagination. The root `checkouts` query uses `FilterConnectionField`, which additionally accepts `filter` and `sort_by` arguments. All the necessary filter and sorter classes already exist in `saleor/graphql/checkout/` — they simply haven't been wired up to the `User` type.

### Proposed Solution

Change `User.checkouts` from `ConnectionField` to `FilterConnectionField` and pass it the existing `CheckoutFilterInput` and `CheckoutSortingInput`. Verify the resolver handles the kwargs correctly, rebuild the schema, and add tests.

### Implementation Plan

Using UMPIRE framework:

**Understand:** `User.checkouts` is missing `sortBy` and `filter` arguments that the root `checkouts` query already has. The fix is to bring parity by reusing existing filter/sort infrastructure.

**Match:** The root `checkouts` query in `schema.py` is the direct pattern to follow. `CheckoutFilterInput` and `CheckoutSortingInput` are already defined and can be imported directly.

**Plan:**

1. In `saleor/graphql/account/types.py`: change `checkouts = ConnectionField(...)` to `checkouts = FilterConnectionField(...)` and add `filter=CheckoutFilterInput(...)` and `sort_by=CheckoutSortingInput(...)` arguments.
2. Verify or update `resolve_checkouts` to correctly handle the additional kwargs passed by `FilterConnectionField`.
3. Run `uv run poe build-schema` to regenerate `schema.graphql`.
4. Create `saleor/graphql/checkout/tests/queries/test_user_checkouts.py` with tests for sort, filter, and combined behavior.
5. Run the test suite and fix any issues.
6. Add a changelog entry under `Unreleased > Other changes`.

**Implement:** [Branch/commit links to be added as work progresses]

**Review:**
- [ ] Follows `FilterConnectionField` pattern used elsewhere in the codebase
- [ ] Field descriptions end in a period (Saleor style)
- [ ] `schema.graphql` rebuilt and committed
- [ ] Pre-commit hooks pass
- [ ] No existing tests broken

**Evaluate:** Manually verify in the GraphQL playground that `User.checkouts` accepts `sortBy` and `filter` and returns correctly sorted/filtered results. Confirm all new tests pass and no regressions in existing checkout/account test suites.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: `User.checkouts` returns all checkouts with no arguments (regression/baseline)
- [ ] Test case 2: `User.checkouts` with `sortBy: CREATION_DATE ASC` returns results in ascending order
- [ ] Test case 3: `User.checkouts` with `sortBy: CREATION_DATE DESC` returns results in descending order
- [ ] Test case 4: `User.checkouts` with a channel filter returns only matching checkouts
- [ ] Test case 5: Combined filter + sortBy works correctly

### Integration Tests

- [x] Non-staff user querying their own `User.checkouts` with filter/sort works and only shows their own data
- [x] All 28 existing `test_me.py` tests continue to pass — no regressions

### Manual Testing

Tested in the GraphQL playground at `http://localhost:8000/graphql/` using the reproduction steps. Confirmed:
- Root `checkouts` query with `sortBy` works ✅
- `User.checkouts` without `sortBy` — confirmed bug ❌
- `User.checkouts` with `sortBy` after fix — works ✅
- `User.checkouts` with `filter` after fix — works ✅
- `customers { checkouts(sortBy: ...) }` staff query — works ✅

---

## Implementation Notes

### Week 1 Progress (Phase I — Complete)

Selected issue #12520, forked the saleor/saleor repository, set up GitHub, and created this Contribution README. Identified the affected files and understood the problem scope.

**Week 2 Progress (Phase II — Complete)**

Set up local development environment on Windows using uv, Docker, and Saleor's poe task runner. Encountered and resolved 4 Windows-specific blockers (C++ Build Tools for pywatchman, memray Linux-only package, AppLocker blocking poe, and missing resource module). Windows workarounds saved separately on the windows-dev-setup branch at https://github.com/rubysnewjourney/saleor/tree/windows-dev-setup.
Reproduced the bug in the GraphQL playground — confirmed User.checkouts rejects sortBy with Unknown argument error while the root checkouts query accepts it fine.

**Week 3 Progress (Phase III — Complete)**

Implemented the fix in saleor/graphql/account/types.py:

Added imports for CheckoutFilterInput, CheckoutSortingInput, and ADDED_IN_324

Changed User.checkouts from ConnectionField to FilterConnectionField

Added sort_by and filter arguments with ADDED_IN_324 version labels

Updated resolve_checkouts resolver to convert dataloader list result to a Django queryset using get_database_connection_name for replica DB access, then applied filter_connection_queryset so filter and sort arguments are actually executed against the database

Wrote 4 new tests in saleor/graphql/account/tests/queries/test_me.py:

test_me_checkouts_sort_by_creation_date_asc — verifies ASC sort order

test_me_checkouts_sort_by_creation_date_desc — verifies DESC sort order

test_me_checkouts_filter_by_channel — verifies channel filter returns correct subset

test_me_checkouts_sort_and_filter_combined — verifies both arguments work together

All 32 tests in test_me.py pass including the 4 new ones. No regressions.

Synced branch with latest upstream before each push via git fetch + git rebase. Submitted PR #19363 and marked ready for review.

**Code Changes**

**Files modified**: saleor/graphql/account/types.py, saleor/graphql/account/tests/queries/test_me.py

**Key commits:** https://github.com/rubysnewjourney/saleor/commit/0aacdcb8faa0cc20272de908ba0a0acc47eb4d93

**Approach decisions**: Reused existing CheckoutFilterInput, CheckoutSortingInput, and FilterConnectionField — no new classes created. Added ADDED_IN_324 labels to both new arguments following Saleor's versioning convention. Updated resolver to convert dataloader list to queryset with get_database_connection_name for correct replica DB access. Windows-specific workarounds (rlimit.py, pyproject.toml) kept off the PR branch and saved on windows-dev-setup branch instead.

### Week 4 Progress (Maintainer Feedback — Complete)

## Pull Request

**PR Link:** https://github.com/saleor/saleor/pull/19363

**PR Description Draft:**
Extends `User.checkouts` to accept the same `filter` and `sortBy` arguments as the root `checkouts` query, resolving the API inconsistency reported in #12520. Changes `ConnectionField` to `FilterConnectionField` in `saleor/graphql/account/types.py` and wires in the existing `CheckoutFilterInput` and `CheckoutSortingInput`. Adds tests for sort, filter, and combined behavior.

**Maintainer Feedback:** 
Received review feedback from `wcislo-saleor` with 4 requested changes. Investigated each point by reading the reference test `test_sort_order_by_rank_without_search` and tracing the `validate_and_apply_search_rank_sorting` function in `saleor/graphql/core/utils/__init__.py`.

**Here is the copy of the maintainer feedbacks:**

Marcin Wcisło <notifications@github.com> Unsubscribe
Thu, Jun 25, 4:19 AM (1 day ago)
to saleor/saleor, me, Author

@wcislo-saleor requested changes on this pull request.

On saleor/graphql/account/tests/queries/test_me.py:

Please refer to test saleor/graphql/order/tests/queries/test_order_with_sort.py:test_sort_order_by_rank_without_search. If query with such variable would be sent to query User's checkout it would have crashed.

In saleor/graphql/account/types.py:

> @@ -561,8 +565,18 @@ def return_checkout_ids(checkouts):
     @staticmethod
     def resolve_checkouts(root: models.User, info: ResolveInfo, **kwargs):
         def _resolve_checkouts(checkouts):
+            from ...checkout.models import Checkout
+
+            if isinstance(checkouts, list):
It seems like dataloaders from which data comes in to this function are always returning list therefore this always will be a list. This code could be simplified.

In saleor/graphql/account/types.py:

> @@ -561,8 +565,18 @@ def return_checkout_ids(checkouts):
     @staticmethod
     def resolve_checkouts(root: models.User, info: ResolveInfo, **kwargs):
         def _resolve_checkouts(checkouts):
+            from ...checkout.models import Checkout
+
+            if isinstance(checkouts, list):
+                checkout_ids = [c.pk for c in checkouts]
+                checkouts = Checkout.objects.using(
checkouts list of Checkout objects is currently unconditionally ingored and new query set gets evaluated. This is not necessary when sorting nor filtering isn't happening.

In saleor/graphql/account/types.py:

> @@ -368,13 +370,15 @@ class User(ModelObjectType[models.User]):
             description="Slug of a channel for which the data should be returned."
         ),
     )
-    checkouts = ConnectionField(
+    checkouts = FilterConnectionField(
GraphQL schema has to be regenerated after this change.

—
Reply to this email directly, view it on GitHub, or unsubscribe.
Triage notifications, keep track of coding agent tasks and review pull requests on the go with GitHub Mobile for iOS and Android. Download it today!
You are receiving this because you authored the thread.

**I responded back as below:**

Ruby Khatoon <rubysnewjourney@gmail.com>
Thu, Jun 25, 7:57 PM (1 day ago)
to saleor/saleor, saleor/saleor, Author

Thank you for your prompt response and for the detailed review!

I'll address all four points:

1. I'll add a test covering the RANK sort without search case, following the pattern in `test_sort_order_by_rank_without_search`.
2. I'll remove the `isinstance` check since dataloaders always return a list.
3. I'll add a guard so the list-to-queryset conversion only happens when `filter` or `sort_by` kwargs are actually present.
4. I'll regenerate `schema.graphql` and include it in the commit.

Please let me know if there are any gaps in understanding your feedback to address the points brought forth.  Once I hear back from you, I will make the changes , test them and update the details and push an update accordingly. 

Thanks again for your help and support. I'm looking forward to hearing from you.
Ruby

**Maintainer responded back as :**
Marcin Wcisło <notifications@github.com>
6:45 AM (16 hours ago)
to Mention, saleor/saleor, me


wcislo-saleor
 left a comment 
(saleor/saleor#19363)
@rubysnewjourney

It sounds fine.

—
Reply to this email directly, view it on GitHub, or unsubscribe.
Triage notifications, keep track of coding agent tasks and review pull requests on the go with GitHub Mobile for iOS and Android. Download it today!
You are receiving this because you were mentioned.

**Changes made in response to feedback:**

1. **Added RANK test** — `test_me_checkouts_sort_by_rank_without_search` added to `test_me.py`. Confirmed the exact error message `"Sorting by RANK is available only when using a search filter."` and used `ignore_errors=True` pattern matching the order tests.

2. **Removed `isinstance` check** — Simplified the resolver since dataloaders always return a list. Removed the conditional guard.

3. **Added conditional queryset conversion** — List-to-queryset conversion now only happens when `sort_by` or `filter` kwargs are present, avoiding unnecessary DB queries for plain pagination requests.

4. **Added `validate_and_apply_search_rank_sorting`** — Added the RANK validation call at the top of `resolve_checkouts`, matching the pattern used in the root `checkouts` resolver in `schema.py`. Also added `CheckoutSortField` and `validate_and_apply_search_rank_sorting` imports.

5. **Regenerated `schema.graphql`** — Ran `poe build-schema` and committed the updated schema file showing new `sortBy` and `filter` arguments on `User.checkouts`.

All 33 tests in `test_me.py` pass. Fixes pushed in updated commit `f513311`. PR is open and awaiting re-review.

### Code Changes

- **Files modified:** `saleor/graphql/account/types.py`, `saleor/graphql/account/tests/queries/test_me.py`, `saleor/graphql/schema.graphql`
- **Key commits:** https://github.com/rubysnewjourney/saleor/commit/f513311c49e5abbf6d3342e35746fde33e423b06
- **Approach decisions:** Reused existing `CheckoutFilterInput`, `CheckoutSortingInput`, and `FilterConnectionField` — no new classes created. Added `ADDED_IN_324` labels to both new arguments. Updated resolver to conditionally convert dataloader list to queryset only when filter/sort args present. Added `validate_and_apply_search_rank_sorting` to handle RANK sort edge case. Regenerated `schema.graphql`. Windows-specific workarounds (`rlimit.py`, `pyproject.toml`) kept off the PR branch and saved on `windows-dev-setup` branch.

**Status:** Awaiting review
**Week 6 follow up**

I followed up with a friendly reminder
I received an automated update on merge conflict and the merge conflict is resolved. 

Below is the updat so far post the conflict merge resolution
**previously done:**
All 4 maintainer feedback points
Addressed 5 tests passing
All 33 test_me.py tests pass
schema.graphql regenerated 
**Latest changes:**
Merge conflict resolved 
FixedBranch synced with latest upstream 
On 697b9ec6f Comment posted to prompt re-review

I posted an update to the maintainers both on Github and also the discord channel
as shown below:
**Discord:** Hello @wcislo-saleor & @stmpn — resolved the merge conflict from the recent ADDED_IN cleanup commit (af4ddf5). Branch is now up to date with main and ready for re-review! This is for the PR - https://github.com/saleor/saleor/pull/19363

**GitHub:** Hello @wcislo-saleor & @stmpn — resolved the merge conflict from the recent ADDED_IN cleanup commit (af4ddf5). Branch is now up to date with main and ready for re-review!

## Learnings & Reflections

### Technical Skills Gained

- How Saleor structures GraphQL types, filters, and sorters across its app modules
- The difference between `ConnectionField` and `FilterConnectionField` in Graphene-Django
- How Saleor's dataloader pattern works — dataloaders return Python lists, not Django querysets
- How to correctly convert a dataloader list to a queryset using `get_database_connection_name` for replica DB access
- How to conditionally apply queryset conversion only when needed to avoid unnecessary DB queries
- How `validate_and_apply_search_rank_sorting` works and why RANK sorting requires a search query
- How to set up a Django + GraphQL project locally on Windows using `uv` and Docker
- How to write pytest tests following Saleor's given/when/then pattern including testing expected errors with `ignore_errors=True`
- How to regenerate `schema.graphql` after GraphQL field changes
- Open source contribution workflow: fork, branch, rebase, squash, PR, draft mode, responding to maintainer feedback

### Challenges Overcome

- **Windows environment setup** — 4 Windows-specific blockers resolved: C++ Build Tools for `pywatchman`, `memray` Linux-only package excluded via platform marker, AppLocker blocking `poe` bypassed via direct Python module invocation, and `resource` Unix-only module patched with `sys.platform` guard
- **Indentation errors** — Notepad mixed tabs and spaces when editing Python files; resolved by using VS Code and Python scripts to fix indentation precisely
- **Dataloader list vs queryset** — `filter_connection_queryset` expects a Django queryset but the dataloader returns a Python list; fixed by converting list to queryset with correct replica DB connection
- **Unconditional queryset conversion** — First implementation always converted list to queryset even without filter/sort; maintainer pointed this out; fixed with conditional guard
- **Missing RANK validation** — First implementation didn't call `validate_and_apply_search_rank_sorting`; maintainer pointed this out via reference test; fixed by adding the call to the resolver
- **`schema.graphql` not regenerated** — Forgot to run `poe build-schema` after changing field type; maintainer flagged it; fixed by regenerating and committing
- **Keeping Windows workarounds off the PR** — Learned to use separate branches and `git restore` to keep local dev changes out of the clean contribution branch
- **Interactive rebase on Windows** — Navigated Git editor issues on Windows using Notepad as the configured Git editor

### What I'd Do Differently Next Time

- Set up on Linux/WSL2 from the start to avoid Windows-specific issues
- Read the full resolver chain before writing the fix — understanding the dataloader pattern upfront would have saved debugging time
- Study existing similar implementations (e.g. root `checkouts` resolver in `schema.py`) more thoroughly before writing the fix — would have caught the missing `validate_and_apply_search_rank_sorting` call and `schema.graphql` regeneration earlier
- Write tests earlier in the process to catch issues sooner
- Sync with upstream more frequently throughout the process
- Check all related files (schema, sorters, filters) before submitting to ensure nothing is missed

---

## Resources Used

- [Saleor Contributing Guide](https://github.com/saleor/saleor/blob/main/CONTRIBUTING.md)
- [Saleor GraphQL API Reference — User](https://docs.saleor.io/docs/3.x/api-reference/users/objects/user)
- [Saleor GraphQL API Reference — Checkouts Query](https://docs.saleor.io/docs/3.x/api-reference/checkout/queries/checkouts)
- [Issue #12520](https://github.com/saleor/saleor/issues/12520)
- [PR #19363](https://github.com/saleor/saleor/pull/19363)
- [Windows dev setup branch](https://github.com/rubysnewjourney/saleor/tree/windows-dev-setup)
- [Microsoft C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) — required for building C extensions on Windows
- [Django ORM documentation](https://docs.djangoproject.com/en/5.2/topics/db/queries/) — understanding querysets vs lists
- [Graphene-Django FilterConnectionField](https://docs.graphene-python.org/projects/django/en/latest/filtering/) — filter field pattern used in the fix
