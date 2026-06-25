**CodeAI301FirstProject**

My First open source project for CodePath AI 301

Contribution [1]: [Add sortBy and filter to User.checkouts]

Contribution Number: 1

 Student: Ruby Khatoon
 
 Issue: https://github.com/saleor/saleor/issues/12520
 
 Status: Phase III — Complete | PR Submitted | Tests Written | Ready for Review | Phase IV - In progress

**Why I Chose This Issue**
I chose this issue because I immediately understood the confusion it creates for developers: the global checkouts query supports filtering and sorting, but the User.checkouts field does not. This inconsistency makes the API harder to use and breaks the expectation that similar fields behave similarly. Since I've been learning how GraphQL schemas are structured and how resolvers work, this felt like a great opportunity to apply that knowledge in a real project.
This issue also appealed to me as a first contribution because it is small enough to be approachable, yet meaningful enough to improve the clarity and usability of the API. By resolving this ambiguity, I can help make the developer experience more consistent while learning more about Saleor's schema design patterns, filter/sort utilities, and resolver logic.

**Understanding the Issue**

**Problem Description**

The root-level checkouts query in the GraphQL API already supports both sortBy and filter arguments. However, the User.checkouts field does not expose these same arguments. This creates an inconsistency in the API: developers can sort and filter checkouts globally, but cannot apply the same operations when querying checkouts for a specific user.
The underlying queryset logic appears reusable, so adding these arguments should be straightforward and backward-compatible.

**Expected Behavior**

Extend the User.checkouts field to accept the same filter and sortBy arguments used by the root-level checkouts query.
Update the resolver to pass these arguments into the existing checkout queryset logic.
Add tests to confirm sorting and filtering work correctly on User.checkouts.
Update the GraphQL schema file to reflect the new arguments.

**Current Behavior**

The checkouts root query (in graphql/checkout/schema.py) already implements filtering and sorting through existing filtersets and sorters. The User.checkouts field (in graphql/account/types.py) currently exposes pagination arguments but does not accept filter or sortBy.

**Affected Components**

The API files are in saleor/graphql/ directory. Key files for this issue:
saleor/graphql/account/types.py — where User.checkouts is defined
saleor/graphql/checkout/schema.py — reference for how root checkouts implements filter/sort
saleor/graphql/checkout/filters.py — CheckoutFilterInput (reusable)
saleor/graphql/checkout/sorters.py — CheckoutSortingInput (reusable)

**Reproduction Process**

**Environment Setup**

Prerequisites installed:
libmagic and uv via Homebrew
Python 3.12 via uv python install 3.12
Services started via Docker:
cd .devcontainer
docker compose up db dashboard cache mailpit

Dependencies and hooks:
uv sync
uv run pre-commit install
cp .env.example .env
uv run poe migrate
uv run poe populatedb
uv run poe start

Server runs at http://localhost:8000/graphql/
Steps to Reproduce
Step 1 — Verify root checkouts query accepts sortBy and filter (works):
{
  checkouts(
    first: 5,
    sortBy: { field: CREATION_DATE, direction: ASC }
  ) {
    edges { node { id created } }
  }
}

✅ Returns results successfully.
Step 2 — Verify User.checkouts does NOT accept these arguments (the bug):
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

❌ Error: Unknown argument "sortBy" on field "User.checkouts" — bug confirmed.

**Step 3 — Confirm in source code:**

In saleor/graphql/checkout/schema.py, the root checkouts uses FilterConnectionField with filter and sort_by arguments.
In saleor/graphql/account/types.py, User.checkouts uses a plain ConnectionField with no filter or sort_by — this is the root cause.

**Reproduction Evidence**

Commit showing reproduction: [Link to be added]
My findings: The fix requires changing ConnectionField to FilterConnectionField on User.checkouts and wiring in the existing CheckoutFilterInput and CheckoutSortingInput classes that are already defined and used by the root query.

Solution Approach

**Analysis**
The root cause is in saleor/graphql/account/types.py. The User.checkouts field uses a plain ConnectionField, which only supports pagination. The root checkouts query uses FilterConnectionField, which additionally accepts filter and sort_by arguments. All the necessary filter and sorter classes already exist in saleor/graphql/checkout/ — they simply haven't been wired up to the User type.

**Proposed Solution**

Change User.checkouts from ConnectionField to FilterConnectionField and pass it the existing CheckoutFilterInput and CheckoutSortingInput. Verify the resolver handles the kwargs correctly, rebuild the schema, and add tests.

**Implementation Plan**

**Using UMPIRE framework:**
Understand: User.checkouts is missing sortBy and filter arguments that the root checkouts query already has. The fix is to bring parity by reusing existing filter/sort infrastructure.
Match: The root checkouts query in schema.py is the direct pattern to follow. CheckoutFilterInput and CheckoutSortingInput are already defined and can be imported directly.

**Plan:**

In saleor/graphql/account/types.py: change checkouts = ConnectionField(...) to checkouts = FilterConnectionField(...) and add filter=CheckoutFilterInput(...) and sort_by=CheckoutSortingInput(...) arguments.
Verify or update resolve_checkouts to correctly handle the additional kwargs passed by FilterConnectionField.
Run uv run poe build-schema to regenerate schema.graphql.
Create saleor/graphql/checkout/tests/queries/test_user_checkouts.py with tests for sort, filter, and combined behavior.
Run the test suite and fix any issues.
Add a changelog entry under Unreleased > Other changes.
Implement: [Add sortBy and filter to User.checkouts - closes #12520 by rubysnewjourney · Pull Request #19363 · saleor/saleor ]

**Review:**

[ ] Follows FilterConnectionField pattern used elsewhere in the codebase
[ ] Field descriptions end in a period (Saleor style)
[ ] schema.graphql rebuilt and committed
[ ] Pre-commit hooks pass
[ ] No existing tests broken
Evaluate: Manually verify in the GraphQL playground that User.checkouts accepts sortBy and filter and returns correctly sorted/filtered results. Confirm all new tests pass and no regressions in existing checkout/account test suites.

**Testing Strategy**

**Unit Tests**
[ ] Test case 1: User.checkouts returns all checkouts with no arguments (regression/baseline)
[ ] Test case 2: User.checkouts with sortBy: CREATION_DATE ASC returns results in ascending order
[ ] Test case 3: User.checkouts with sortBy: CREATION_DATE DESC returns results in descending order
[ ] Test case 4: User.checkouts with a channel filter returns only matching checkouts
[ ] Test case 5: Combined filter + sortBy works correctly

**Integration Tests**

[x] Non-staff user querying their own User.checkouts with filter/sort works and only shows their own data
[x] All 28 existing test_me.py tests continue to pass — no regressions

**Manual Testing**

Tested in the GraphQL playground at http://localhost:8000/graphql/ using the reproduction steps. Confirmed:
Root checkouts query with sortBy works ✅
User.checkouts without sortBy — confirmed bug ❌
User.checkouts with sortBy after fix — works ✅
User.checkouts with filter after fix — works ✅
customers { checkouts(sortBy: ...) } staff query — works ✅

**Implementation Notes**

**Week 1 Progress (Phase I — Complete)**

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
Code Changes
Files modified: saleor/graphql/account/types.py, saleor/graphql/account/tests/queries/test_me.py
Key commits: https://github.com/rubysnewjourney/saleor/commit/0aacdcb8faa0cc20272de908ba0a0acc47eb4d93
Approach decisions: Reused existing CheckoutFilterInput, CheckoutSortingInput, and FilterConnectionField — no new classes created. Added ADDED_IN_324 labels to both new arguments following Saleor's versioning convention. Updated resolver to convert dataloader list to queryset with get_database_connection_name for correct replica DB access. Windows-specific workarounds (rlimit.py, pyproject.toml) kept off the PR branch and saved on windows-dev-setup branch instead.

**Pull Request**

PR Link: https://github.com/saleor/saleor/pull/19363
PR Description Draft:
Extends User.checkouts to accept the same filter and sortBy arguments as the root checkouts query, resolving the API inconsistency reported in #12520. Changes ConnectionField to FilterConnectionField in saleor/graphql/account/types.py and wires in the existing CheckoutFilterInput and CheckoutSortingInput. Adds tests for sort, filter, and combined behavior.
Maintainer Feedback: [To be added]
Status: Awaiting review

**Learnings & Reflections**

Technical Skills Gained
How Saleor structures GraphQL types, filters, and sorters across its app modules
The difference between ConnectionField and FilterConnectionField in Graphene-Django
How Saleor's dataloader pattern works — dataloaders return Python lists, not Django querysets
How to correctly convert a dataloader list to a queryset using get_database_connection_name for replica DB access
How to set up a Django + GraphQL project locally on Windows using uv and Docker
How to write pytest tests following Saleor's given/when/then pattern
Open source contribution workflow: fork, branch, rebase, squash, PR, draft mode
Challenges Overcome
Windows environment setup — 4 Windows-specific blockers resolved: C++ Build Tools for pywatchman, memray Linux-only package excluded via platform marker, AppLocker blocking poe bypassed via direct Python module invocation, and resource Unix-only module patched with sys.platform guard
Indentation errors — Notepad mixed tabs and spaces when editing Python files; resolved by using VS Code and Python scripts to fix indentation precisely
Dataloader list vs queryset — filter_connection_queryset expects a Django queryset but the dataloader returns a Python list; fixed by converting list to queryset with correct replica DB connection
Keeping Windows workarounds off the PR — learned to use separate branches and git restore to keep local dev changes out of the clean contribution branch
Interactive rebase on Windows — navigated Git editor issues on Windows using Notepad as the configured Git editor
What I'd Do Differently Next Time
Set up on Linux/WSL2 from the start to avoid Windows-specific issues
Read the full resolver chain before writing the fix — understanding the dataloader pattern upfront would have saved debugging time
Write tests earlier in the process to catch the list-vs-queryset issue sooner
Sync with upstream more frequently throughout the process

**Resources Used**
Saleor Contributing Guide
Saleor GraphQL API Reference — User
Saleor GraphQL API Reference — Checkouts Query
Issue #12520
PR #19363
Windows dev setup branch
Microsoft C++ Build Tools — required for building C extensions on Windows
Django ORM documentation — understanding querysets vs lists
Graphene-Django FilterConnectionField — filter field pattern used in the fix

