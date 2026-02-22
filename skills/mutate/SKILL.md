---
argument-hint: <source-file> [method-name] [--fix] [--diff] [--higher-order]
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

# Semantic Mutation Testing

You are performing **semantic mutation testing** on a source file. Unlike mechanical mutation tools, you use your understanding of the code's *intent* to generate meaningful mutations that test whether the test suite verifies actual behavior.

**Supported languages**: Scala (`.scala`), Python (`.py`), TypeScript (`.ts`, `.tsx`)

**Input**: `$ARGUMENTS`

Parse the input:
- First argument: path to the source file (required)
- Second argument (if not a flag): method/function name to scope mutations to (optional)
- `--fix`: After testing, write suggested tests for surviving mutations and verify they work
- `--diff`: Only mutate lines changed in the current branch vs the base branch
- `--higher-order`: Enable higher-order mutation testing (also auto-enabled when first-order score is 90%+)

If no arguments are provided, ask the user to specify a source file.

---

## Phase 0 — Safety Checks

Before doing anything, verify it's safe to proceed.

### Step 0.0: Detect language

Determine the language from the source file's extension:
- `.scala` → Read the language configuration from `skills/mutate/languages/scala.md`
- `.py` → Read the language configuration from `skills/mutate/languages/python.md`
- `.ts`, `.tsx` → Read the language configuration from `skills/mutate/languages/typescript.md`

If the file extension is not one of the above, stop and inform the user:

> **Unsupported file type**: `<extension>`. Semantic mutation testing currently supports Scala (.scala), Python (.py), and TypeScript (.ts, .tsx).

Store the language configuration for use in all subsequent steps.

### Step 0.1: Verify the source file exists

Use the Read tool to read the file at the specified path. If the file does not exist or is not a supported source file, stop and inform the user.

### Step 0.2: Check for uncommitted changes to the target file

Run:
```bash
git status --porcelain -- <source-file-path>
```

If there is **any output** (the target file has uncommitted changes), **stop immediately** and tell the user:

> **Mutation testing blocked**: `<source-file-path>` has uncommitted changes. Please commit or stash your changes before running mutation testing. This safety check ensures we can always restore your code to its original state.
>
> Run `git stash` or `git commit` and try again.

Do NOT proceed past this point if the target file has uncommitted changes.

### Step 0.3: Store original file content

Read the entire source file and store its complete contents. You will use this to restore the file after every mutation. This is your **safety copy** — you will write this exact content back after each mutation.

### Step 0.4: Detect project structure

Follow the **Project detection** section in the language configuration to detect the project structure, build tool, and any subproject/module organization.

### Step 0.5: Build setup and compilation check

Follow the **Build setup** section in the language configuration to verify the code compiles (or is syntactically valid) and to initialize any build tools needed for the session.

If compilation or syntax validation fails, stop and inform the user that the code must be valid before mutation testing can begin. Keep the timeout guidance from the language configuration.

### Step 0.6: Identify diff scope (--diff mode only)

If `--diff` was specified, identify which lines in the source file were changed relative to the base branch:

```bash
git merge-base HEAD main 2>&1
```

If `main` doesn't exist, try `master`. Store the merge base commit hash, then:

```bash
git diff <merge-base> HEAD -- <source-file-path> 2>&1
```

Parse the diff output to extract the **changed line ranges** (lines added or modified). Store these line ranges — in Phase 1, only generate mutations targeting code within these ranges.

If the file has no changes relative to the base branch, inform the user:

> **No changes detected** in `<source-file-path>` relative to the base branch. Nothing to mutate in --diff mode.

---

## Phase 1 — Locate Test Files & Analyze Source Code

### Step 1.1: Find test files

Extract the class/module/function name from the source file. Use the **Test discovery** patterns from the language configuration to search for test files.

### Step 1.2: Extract test identifiers

Follow the **Test identification** section in the language configuration to extract the test identifiers needed by the test runner (e.g., fully qualified class names for sbt, file paths for pytest/jest).

If no test files are found, stop and inform the user:

> **No test files found** for `ClassName`. Mutation testing requires existing tests to run against. Please write tests first.

### Step 1.2.1: Analyze test mocking patterns

For each discovered test file, scan for mocking framework usage and overmocking indicators. This analysis informs the final report — it does not block mutation testing.

**Detect mocking framework**: Follow the **Mocking frameworks** section in the language configuration to identify which mocking framework is in use and detect its patterns.

If no mocking framework is detected, record "None" and skip to Step 1.3.

**Count mocks and assertions**: Use the mock setup patterns and assertion patterns from the language configuration.

**Check for overmocking indicators**:

1. **Mocking the SUT** — The class under mutation test is itself mocked (e.g., `mock[UserService]` / `Mock(spec=UserService)` / `jest.mock('./UserService')` when testing `UserService`). This is the strongest signal that tests are not exercising real code paths.

2. **High mock-to-assertion ratio** — More than 2x as many mock setups as assertions in a test file. Indicates tests spend more effort configuring behavior than verifying outcomes.

3. **Excessive wildcard matchers** — More than half of mock setups use wildcard matchers (per the language configuration) instead of specific expected values. Indicates tests don't verify correct arguments are passed.

4. **Verify-only tests** — Test cases that only verify interactions (e.g., `verify()`, `assert_called_with`, `toHaveBeenCalled`) without any outcome assertions. Tests that only check "was this called?" without "did it produce the right result?" miss behavioral changes.

5. **Deep mock chains** — Mock setups that traverse multiple levels of indirection. Indicates tests are coupled to internal object graph structure.

**Record per-test-file summary**:
- File path
- Mocking framework detected
- Mock setup count
- Assertion count
- Mock-to-assertion ratio
- Indicators triggered (list of which indicators fired, with specific evidence)

**Assign severity**:
- **None**: No mocking detected, or mocking present with no indicators triggered
- **Low**: One minor indicator triggered (e.g., slightly high mock ratio, a few wildcard matchers)
- **Moderate**: Multiple indicators triggered, or mock-to-assertion ratio above 3x
- **High**: Mocking the SUT detected, or verify-only tests with no outcome assertions

Store all results for use in Phase 5 report generation.

### Step 1.3: Verify tests pass

Run the discovered tests using the **Test execution** command from the language configuration to confirm they pass before mutation testing begins.

Use a 120-second timeout. If tests fail, stop and inform the user that tests must pass before mutation testing can begin.

### Step 1.4: Understand the code

Read the source file carefully. Identify:
- The class/module/function name and package/module path
- All methods/functions, especially public ones with business logic
- Focus on the features listed in the **Code analysis focus** section of the language configuration, in addition to universal constructs (conditionals, loops, error handling, function calls)

If a **method/function name** was specified as the second argument, scope your analysis to only that method/function.

If **--diff mode** is active, focus your analysis on the changed line ranges identified in Step 0.6. You may still read surrounding context to understand the code, but only generate mutations for code within the changed ranges.

### Step 1.5: Select mutations with semantic clustering

Refer to `catalogs/shared-catalog.md` and the language-specific catalog (`catalogs/<lang>-catalog.md`) for the full list of mutation operators organized by category.

**Candidate identification**: Identify all possible mutation candidates in the target scope.

**Semantic clustering**: Group candidates that test the same behavioral property into clusters. For example:
- Inverting a filter and replacing a filter with always-true both test whether the filter predicate matters — they belong to the same cluster
- `x >= 18` → `x > 18` and `x >= 18` → `x >= 19` both test the boundary at 18 — same cluster
- Two mutations on different variables or different functions test different behaviors — different clusters

**Selection from clusters**: Pick **one representative mutation per cluster** — the one most likely to survive if behavior is untested. This maximizes the diversity of behavioral properties tested while avoiding redundant mutations that would produce the same test outcome.

**General principles**:
- **Diversify categories**: Cover multiple categories rather than exhausting one
- **Target business logic**: Focus on methods/functions that implement domain rules, not boilerplate
- **Skip trivial code**: Don't mutate logging, toString/`__str__`, hashCode/`__hash__`, or pure boilerplate
- **One at a time**: Each mutation is independent — never combine mutations

Cap the mutation plan at **25 mutations maximum**. If more clusters exist, select the 25 most impactful.

### Step 1.6: Present mutation plan

Present the plan as a numbered table for user confirmation:

```
## Mutation Plan for `ClassName`

**Test identifiers**: <test classes/files to run>
**Mode**: [standard | diff-scoped | method-scoped]

| # | Category | Location | Mutation Description | Cluster |
|---|----------|----------|---------------------|---------|
| 1 | ... | `method:line` | Description | Cluster name |
| 2 | ... | `method:line` | Description | Cluster name |
| ... | ... | ... | ... | ... |

**Total**: N mutations across M categories (K clusters)

Shall I proceed with this mutation plan?
```

**Wait for user confirmation before proceeding to Phase 2.**

---

## Phase 2 — Apply & Test Loop

For each mutation in the plan, execute this cycle. **Always restore the original file after each mutation, regardless of outcome.**

### Step 2.1: Apply mutation

Use the **Edit** tool to apply the mutation to the source file. Make exactly one change.

### Step 2.2: Run tests

Run the test execution command from the language configuration with a 120-second timeout. Interpret results:

- **Compilation/syntax error in output** → Record as **INCOMPILABLE**, skip to Step 2.3
- **Tests FAIL** → Mutation is **KILLED** (good — tests detected the change)
- **Tests PASS** → Mutation **SURVIVED** (bad — tests missed the change)
- **Timeout** → Record as **TIMEOUT**

### Step 2.3: Restore original file (MANDATORY)

**This step is non-negotiable.** After EVERY mutation, restore the file to its exact original content using the **Write** tool with the complete original file content stored in Phase 0.

Do NOT use Edit to try to reverse the mutation. Always do a full file Write with the stored original content.

### Step 2.4: Report progress

After each mutation, output a one-line progress update:

```
[3/18] ✅ KILLED — Option: `findUser:35` Some(x) → None
[4/18] 🟡 SURVIVED — Collection: `activeUsers:61` .sortBy(_.lastName) → removed
```

---

## Phase 3 — Equivalent Mutant Detection

After the mutation loop completes, analyze each **surviving** mutation to determine if it is semantically equivalent to the original code.

### Step 3.1: Analyze each survivor

For each mutation that SURVIVED, carefully reason about whether the mutation actually changes observable behavior:

**A mutation is EQUIVALENT if**:
- The mutated code produces the same output for all possible inputs (e.g., reordering independent side-effect-free statements)
- The mutation is in dead code that can never be reached
- Type constraints or domain invariants make the original and mutant behave identically (e.g., `x >= 0` → `x > 0` when `x` is always a positive int due to upstream validation)
- The mutation changes internal representation but not the public API contract (e.g., swapping equivalent collection implementations)

**A mutation is NOT equivalent if**:
- There exists any valid input that would produce a different output
- The mutation changes error handling behavior, even for edge cases
- The mutation affects performance characteristics that tests could observe (e.g., timeout-based tests)

### Step 3.2: Document equivalence reasoning

For each survivor, record your analysis:
- **SURVIVED**: The mutation genuinely changes behavior that is untested
- **EQUIVALENT**: The mutation does not change observable behavior (with explanation)

### Step 3.3: Recalculate metrics

Update the mutation score to exclude equivalent mutants:

**Adjusted Mutation Score** = Killed / (Killed + Survived - Equivalent) x 100%

Report both the raw and adjusted scores in the final report.

---

## Phase 4 — Higher-Order Mutations (optional)

This phase runs if `--higher-order` was specified OR if the first-order mutation score is 90% or higher. Higher-order mutations combine two first-order mutations simultaneously, creating subtler faults that are harder to detect.

### Step 4.1: Select higher-order mutation pairs

From the **killed** first-order mutations, select pairs that:
- Are in the same method/function or closely related methods/functions
- Individually are killed but might survive when combined (compensating mutations)
- Target different categories (e.g., a comparison change + a return value change)

Generate up to **10 higher-order mutations**. Each is a pair of two first-order mutations applied simultaneously.

Refer to the higher-order mutation section in `catalogs/shared-catalog.md` for guidance.

### Step 4.2: Present higher-order plan

```
## Higher-Order Mutation Plan

Your first-order score is X%. Testing with combined mutations to find deeper gaps.

| # | Mutations Combined | Location | Description |
|---|-------------------|----------|-------------|
| H1 | #3 + #7 | `method:lines` | Description of combined mutation |
| ... | ... | ... | ... |

**Total**: N higher-order mutations

Shall I proceed?
```

**Wait for user confirmation.**

### Step 4.3: Apply & test higher-order mutations

For each higher-order mutation:

1. Apply **both** edits using the Edit tool (two separate Edit calls)
2. Run the test execution command (same as Phase 2, Step 2.2)
3. Record result: KILLED / SURVIVED / INCOMPILABLE / TIMEOUT
4. **Restore original file** using full Write (same as Phase 2, Step 2.3)

### Step 4.4: Report progress

```
[H2/10] ✅ KILLED — Higher-order: #3 + #7 (Option + Collection)
[H3/10] 🟡 SURVIVED — Higher-order: #5 + #12 (Option + Either)
```

---

## Phase 5 — Generate Report

After all mutations have been tested, generate a comprehensive report.

### Step 5.1: Calculate metrics

- **Killed**: Mutations where tests failed (detected the change)
- **Survived**: Mutations where tests passed (missed the change), excluding equivalents
- **Equivalent**: Mutations that don't change observable behavior
- **Incompilable**: Mutations that didn't compile
- **Timed out**: Mutations that exceeded the timeout

**Mutation Score (raw)** = Killed / (Killed + Survived + Equivalent) x 100%
**Mutation Score (adjusted)** = Killed / (Killed + Survived) x 100%

The **adjusted score** is the primary metric. Equivalent and incompilable mutations are excluded.

If higher-order mutations were tested, calculate a separate **Higher-Order Score**.

### Step 5.2: Determine quality rating

| Score | Rating |
|-------|--------|
| 90-100% | **Strong** — Test suite thoroughly verifies the behavioral intent of this code. |
| 75-89% | **Adequate** — Good behavioral coverage with some gaps to address. |
| 60-74% | **Weak** — Significant behavioral gaps. The surviving mutations represent untested business logic. |
| Below 60% | **Critical** — Tests do not adequately verify the code's behavior. Major test improvements needed. |

### Step 5.3: Generate the report

Follow the report template at `skills/mutate/report-template.md` and an appropriate example from `skills/mutate/examples/`.

For each **surviving mutation** (non-equivalent), provide:
1. What was changed and where
2. Why it matters (what behavior is untested)
3. A complete, runnable suggested test case in the project's existing test style (see **Test style conventions** in the language configuration)

For each **equivalent mutation**, provide:
1. What was changed
2. Why it's equivalent (the reasoning from Phase 3)

If higher-order mutations were tested, include a separate section with those results.

**Overmocking analysis**: If any overmocking indicators were detected in Step 1.2.1, include the Overmocking Analysis section in the report (see template). Cross-reference surviving mutations with overmocking findings — if a surviving mutation targets code that is heavily mocked in the test files, flag this explicitly in both the Overmocking Analysis section and the Recommendations. This helps the user distinguish between "missing test" and "test exists but is mocked past the mutation point."

### Step 5.4: Output the report

Output the complete report directly in the conversation. Do NOT write it to a file unless the user asks.

---

## Phase 6 — Auto-Fix: Write & Verify Tests (--fix mode only)

This phase only runs if `--fix` was specified. For each surviving (non-equivalent) mutation, write a test that kills it.

### Step 6.1: Identify target test file

Select the most appropriate test file to add new tests to. Prefer the primary test file discovered in Phase 1 (the file with the most existing tests for this class/module).

Read the test file and note:
- The test style used (see **Test style conventions** in the language configuration)
- Import patterns
- Helper methods / fixtures available
- Where to insert new tests (typically at the end of the file or test class)

### Step 6.2: Write tests for each survivor

For each surviving (non-equivalent) mutation, write a test case that:
- Uses the same style and conventions as the existing tests
- Has a descriptive name referencing the behavior being verified
- Would FAIL when the mutation is applied (i.e., it specifically tests the behavior the mutation changes)
- Would PASS on the original code

Add all new tests to the test file using the **Edit** tool. Group them under a comment indicating they were generated:

```
// Tests generated by semantic mutation testing   (for Scala/TypeScript)
# Tests generated by semantic mutation testing    (for Python)
```

### Step 6.3: Verify tests pass on original code

Run the test execution command from the language configuration.

If any new test fails on the original code, it's a bad test. Remove it and log:
```
⚠️ Removed test for mutation #N — failed on original code
```

### Step 6.4: Verify tests kill mutations

For each surviving mutation that now has a test:

1. Apply the mutation using Edit
2. Run the test execution command
3. Verify the new test FAILS (mutation is now killed)
4. Restore original file using Write
5. If the test still passes on the mutation, remove it and log:
```
⚠️ Removed test for mutation #N — did not detect the mutation
```

### Step 6.5: Restore original source file

After all verification, restore the source file to its original content using Write. The test file retains the new tests.

### Step 6.6: Report results

```
## Auto-Fix Results

| Mutation | Test Written | Passes Original | Kills Mutant | Status |
|----------|-------------|-----------------|--------------|--------|
| #5 | ✅ | ✅ | ✅ | **Added** |
| #10 | ✅ | ✅ | ❌ | **Removed** (did not detect mutation) |
| #11 | ✅ | ❌ | — | **Removed** (failed on original) |

**Tests added**: N of M surviving mutations now have tests
**New mutation score**: X% (was Y%)
```

---

## Phase 7 — Final Safety Verification

### Step 7.1: Verify source file integrity

Read the source file one final time and compare its content to the original stored in Phase 0.

### Step 7.2: Restore if needed

If there is ANY discrepancy — even a single character — immediately restore the file using Write with the stored original content. Then inform the user:

> **Safety check**: Detected file discrepancy after mutation testing. The file has been restored to its original state.

### Step 7.3: Build teardown

Follow the **Build teardown** section in the language configuration to clean up any build servers or resources.

### Step 7.4: Confirm clean state

If the file matches the original, confirm:

> **File integrity verified**: `source_file_path` matches its original content. No mutations remain.

If `--fix` was used, also confirm:

> **Test file updated**: `test_file_path` — N new tests added.

---

## Critical Rules

1. **NEVER modify test files** — only the source file under test (exception: `--fix` mode adds tests to test files)
2. **NEVER run the full test suite** — always use the targeted test command, never run all tests
3. **ALWAYS restore after every mutation** — use full Write, not reverse Edit
4. **ALWAYS use 120-second timeouts** for test execution commands
5. **Cap at 25 first-order mutations** — select the most impactful ones if more exist
6. **Cap at 10 higher-order mutations** — select the most promising pairs
7. **Stop on dirty target file** — do not proceed if the source file has uncommitted changes
8. **Wait for user confirmation** after presenting mutation plans before testing
9. **One mutation at a time** — never apply multiple mutations simultaneously (except higher-order pairs)
10. **Equivalent mutants are not failures** — do not count them against the test suite
