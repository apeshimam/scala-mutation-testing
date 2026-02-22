# Mutation Testing Report Template

Use this template structure to generate the final mutation testing report. Replace the placeholder descriptions with actual data from the mutation testing run. See the `examples/` directory for completed examples.

---

## Report Structure

The report should contain these sections in order:

### Header

```markdown
# Mutation Testing Report

**File**: `<source file path>`
**Language**: <Scala | Python | TypeScript>
**Date**: <today's date>
**Mutations tested**: <total attempted>
**Test identifiers**: <comma-separated list of test classes/files>
**Mode**: <standard | diff-scoped | method-scoped>
**Flags**: <--fix, --diff, --higher-order, or none>
```

### Score Summary

A table with:
- **Mutation Score (adjusted)**: killed / (killed + survived), excluding equivalent mutants — this is the primary metric
- **Mutation Score (raw)**: killed / (killed + survived + equivalent)
- Killed, Survived, Equivalent, Incompilable, and Timed Out counts

Include the quality rating (see Field Definitions below) and the rating scale table. The quality rating is based on the **adjusted** score.

### Detailed Results

A table with one row per mutation:

| # | Category | Location | Mutation | Result |
|---|----------|----------|----------|--------|

Each result should use the emoji prefix: ✅ KILLED, 🟡 SURVIVED, ⚪ EQUIVALENT, ⚠️ INCOMPILABLE, ⏱️ TIMEOUT.

### Equivalent Mutation Analysis

If any mutations were classified as equivalent, include a section explaining each:

For each equivalent mutation:
- **What was changed**: Describe the specific edit
- **Why it's equivalent**: Explain why the mutation does not change observable behavior

If no mutations were equivalent, omit this section.

### Surviving Mutation Analysis

For each surviving (non-equivalent) mutation, include a subsection with:
- **Category** and **Location**
- **What was changed**: Describe the specific edit
- **Why it matters**: Explain what behavior is untested
- **Suggested test case**: A complete, runnable test in the project's test framework style

If all mutations were killed or equivalent, state: "All mutations were killed (or equivalent)! The test suite thoroughly verifies the behavior of this code."

### Higher-Order Mutation Results (if applicable)

Only include this section if higher-order mutations were tested. Include:

A results table:

| # | Mutations Combined | Location | Description | Result |
|---|-------------------|----------|-------------|--------|

A **Higher-Order Score**: killed / (killed + survived) for higher-order mutations only.

For each surviving higher-order mutation, explain:
- Which two mutations were combined
- Why the combination survived (what interaction between behaviors is untested)
- A suggested test that would detect the combined fault

### Category Breakdown

A summary table showing per-category counts:

| Category | Attempted | Killed | Survived | Equivalent | Kill Rate |
|----------|-----------|--------|----------|------------|-----------|

Include a **Total** row. "Attempted" counts only compilable mutations. Kill Rate = killed / (killed + survived), excluding equivalents.

### Auto-Fix Results (if --fix was used)

Only include if `--fix` mode was used. Show a table:

| Mutation | Test Written | Passes Original | Kills Mutant | Status |
|----------|-------------|-----------------|--------------|--------|

Summary: how many tests were added, how many were removed (failed verification), and the new adjusted mutation score.

### Overmocking Analysis

Only include this section if overmocking indicators were detected in Step 1.2.1. If no mocking was detected or no indicators triggered, omit this section entirely.

**Per-test-file summary table**:

| Test File | Framework | Mock Setups | Assertions | Ratio | Severity | Indicators |
|-----------|-----------|-------------|------------|-------|----------|------------|

For each row:
- **Framework**: The mocking framework detected (e.g., mockito-scala, unittest.mock, jest.mock, etc.)
- **Ratio**: Mock setups / assertions (e.g., "2.5:1")
- **Severity**: None, Low, Moderate, or High
- **Indicators**: Comma-separated list of triggered indicators (e.g., "High mock ratio, Excessive wildcards")

**Overall overmocking severity**: Report the highest severity across all test files.

**Indicator details**: For each triggered indicator, include:
- **What was detected**: The specific pattern found, with file and line references
- **Why it matters**: How this pattern can undermine mutation testing results
- **Suggestion**: A concrete action to reduce overmocking

**Correlation with surviving mutations**: If any surviving mutations target code paths that are heavily mocked in test files, call this out explicitly:

> ⚠️ **Overmocking correlation**: Mutation #N survived in `ClassName.method` — this code path is mocked in `TestFile` (N mock setups). The mutation may have survived because mocks short-circuit execution before reaching the mutated code. Consider replacing the mock with a real instance for this test.

### Recommendations

A numbered list (1 = highest priority) of actionable improvements. Focus on:
1. Surviving mutations representing the most critical business logic gaps
2. Entire untested categories
3. Test cases that would kill multiple surviving mutations at once
4. Structural test improvements
5. Higher-order mutation gaps (if applicable)
6. Overmocking concerns — e.g., "Replace mocks with real instances where possible", "Add outcome assertions alongside interaction verifications", "Use specific argument matchers instead of wildcards"

End with: `*Generated by semantic mutation testing with Claude*`

---

## Field Definitions

### Quality Rating
Calculate based on **adjusted** kill rate (killed / (killed + survived)):
- **90-100%**: "**Strong** — Test suite thoroughly verifies the behavioral intent of this code."
- **75-89%**: "**Adequate** — Good behavioral coverage with some gaps to address."
- **60-74%**: "**Weak** — Significant behavioral gaps. The surviving mutations represent untested business logic."
- **Below 60%**: "**Critical** — Tests do not adequately verify the code's behavior. Major test improvements needed."

Note: Equivalent, incompilable, and timed-out mutations are excluded from the adjusted kill rate.

### Result Emoji Mapping
- `KILLED` → ✅
- `SURVIVED` → 🟡
- `EQUIVALENT` → ⚪
- `INCOMPILABLE` → ⚠️
- `TIMEOUT` → ⏱️

### Location Format
Use `ClassName.methodName:lineNumber` format, e.g., `UserService.validateEmail:42` or `user_service.validate_email:42`.

### Suggested Test Code
Write complete, runnable test cases in the project's test framework style. Use the same style as the project's existing tests. Include:
- A descriptive test name
- Setup/given section
- Action/when section
- Assertion/then section
