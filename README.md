# AI301 Open Source contributions

Open source project for CodePath AI 301:

------------------------------------------------------------------------------------------
Current Contribution #2
------------------------------------------------------------------------------------------

Contribution [2]: [fix: test_json.py is not being run]

**Contribution Number:** 2  
**Student:** Ruby Khatoon  
**Issue:** (https://github.com/ossf/cve-bin-tool/issues/5689) 
**Status:** Phase I | Sent a message to claim the issue if not being worked in a polite way as shown below| still waiting for update as of 07/11/2026| Planning to proceed with fix and share the PR soon and see if there will be any update from the maintainer.
**Status:** - I had an update on my contribution 1. I had hear back on a conflict with merging and I resolved this week. The details are avialable bottom section of the Phase4 of the contribution 1.

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
Previous Contribution #1
------------------------------------------------------------------------------------------

# Contribution [1]: [Add sortBy and filter to User.checkouts]

**Contribution Number:** 1  
**Student:** Ruby Khatoon  
**Issue:** https://github.com/saleor/saleor/issues/12520  
**Status:** Phase IV — Complete | PR Submitted | Tests Written | Maintainer Feedback Addressed | Respond to maintainer feedback | Push revisions | Merging Conflict Resolved | Awaiting Final Approval
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
