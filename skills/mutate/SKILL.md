---
argument-hint: <scala-source-file> [method-name]
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

# Semantic Mutation Testing for Scala

You are performing **semantic mutation testing** on a Scala source file. Unlike mechanical mutation tools, you use your understanding of the code's *intent* to generate meaningful mutations that test whether the test suite verifies actual behavior.

**Input**: `$ARGUMENTS`

Parse the input:
- First argument: path to the Scala source file (required)
- Second argument: method name to scope mutations to (optional)

If no arguments are provided, ask the user to specify a Scala source file.

---

## Phase 0 — Safety Checks

Before doing anything, verify it's safe to proceed.

### Step 0.1: Verify the source file exists

Use the Read tool to read the file at the specified path. If the file does not exist or is not a `.scala` file, stop and inform the user.

### Step 0.2: Check for uncommitted changes

Run:
```bash
git status --porcelain
```

If there is **any output** (uncommitted changes exist), **stop immediately** and tell the user:

> **Mutation testing blocked**: You have uncommitted changes. Please commit or stash your changes before running mutation testing. This safety check ensures we can always restore your code to its original state.
>
> Run `git stash` or `git commit` and try again.

Do NOT proceed past this point if git status is dirty.

### Step 0.3: Store original file content

Read the entire source file and store its complete contents. You will use this to restore the file after every mutation. This is your **safety copy** — you will write this exact content back after each mutation.

### Step 0.4: Verify compilation

Run:
```bash
sbt compile 2>&1
```

Use a 120-second timeout. If compilation fails, stop and inform the user that the code must compile before mutation testing can begin.

---

## Phase 1 — Analyze Source Code & Build Mutation Plan

### Step 1.1: Understand the code

Read the source file carefully. Identify:
- The class/object name and package
- All methods, especially public methods with business logic
- Types used (Option, Either, Try, collections, case classes)
- Pattern matching expressions
- Error handling (try/catch, recover)
- For comprehensions
- Implicit parameters and given instances

If a **method name** was specified as the second argument, scope your analysis to only that method.

### Step 1.2: Select mutations

Refer to the mutation catalog at `skills/mutate/mutation-catalog.md` for the full list of mutation operators organized by category.

Select mutations following these principles:
- **Prioritize by impact**: Choose mutations most likely to reveal untested behavior
- **Diversify categories**: Cover multiple categories rather than exhausting one
- **Target business logic**: Focus on methods that implement domain rules, not boilerplate
- **Skip trivial code**: Don't mutate logging, toString, hashCode, or pure boilerplate
- **One at a time**: Each mutation is independent — never combine mutations

Cap the mutation plan at **25 mutations maximum**. If more candidates exist, select the 25 most impactful.

### Step 1.3: Present mutation plan

Present the plan as a numbered table for user confirmation:

```
## Mutation Plan for `ClassName`

| # | Category | Location | Mutation Description |
|---|----------|----------|---------------------|
| 1 | Option | `method:line` | `Some(x)` → `None` |
| 2 | Comparison | `method:line` | `age >= 18` → `age > 18` |
| ... | ... | ... | ... |

**Total**: N mutations across M categories

Shall I proceed with this mutation plan?
```

**Wait for user confirmation before proceeding to Phase 2.**

---

## Phase 2 — Locate Test Files

### Step 2.1: Convention-based search

Extract the class/object name (e.g., `UserService`) from the source file. Search for test files using common Scala conventions:

Use Glob to search for:
- `**/UserServiceSpec.scala`
- `**/UserServiceTest.scala`
- `**/UserServiceSuite.scala`

### Step 2.2: Grep-based search

Also search for test files that reference the class by name:

Use Grep to search in `src/test/scala/` for:
- The class name (e.g., `UserService`)
- Import statements referencing the class

### Step 2.3: Extract test class names

For each discovered test file, read it and extract the **fully qualified class name** (package + class name). You will use these with `sbt testOnly`.

If no test files are found, stop and inform the user:

> **No test files found** for `ClassName`. Mutation testing requires existing tests to run against. Please write tests first.

### Step 2.4: Verify tests pass

Run the discovered tests to confirm they pass before mutation testing begins:

```bash
sbt "testOnly fully.qualified.TestClass1 fully.qualified.TestClass2" 2>&1
```

Use a 120-second timeout. If tests fail, stop and inform the user that tests must pass before mutation testing can begin.

---

## Phase 3 — Apply & Test Loop

For each mutation in the plan, execute this cycle. **Always restore the original file after each mutation, regardless of outcome.**

### Step 3.1: Apply mutation

Use the **Edit** tool to apply the mutation to the source file. Make exactly one change.

### Step 3.2: Compile check

Run:
```bash
sbt compile 2>&1
```

With a 120-second timeout. If compilation fails:
- Record this mutation as **INCOMPILABLE**
- Skip to Step 3.4 (restore)

### Step 3.3: Run tests

Run:
```bash
sbt "testOnly fully.qualified.TestClass1 fully.qualified.TestClass2" 2>&1
```

With a 120-second timeout. Interpret results:

- **Tests FAIL** → Mutation is **KILLED** (good — tests detected the change)
- **Tests PASS** → Mutation **SURVIVED** (bad — tests missed the change)
- **Timeout** → Record as **TIMEOUT**

### Step 3.4: Restore original file (MANDATORY)

**This step is non-negotiable.** After EVERY mutation, restore the file to its exact original content using the **Write** tool with the complete original file content stored in Phase 0.

Do NOT use Edit to try to reverse the mutation. Always do a full file Write with the stored original content.

### Step 3.5: Report progress

After each mutation, output a one-line progress update:

```
[3/18] ✅ KILLED — Option: `findUser:35` Some(x) → None
[4/18] 🟡 SURVIVED — Collection: `activeUsers:61` .sortBy(_.lastName) → removed
```

---

## Phase 4 — Generate Report

After all mutations have been tested, generate a comprehensive report.

### Step 4.1: Calculate metrics

- **Killed**: Mutations where tests failed (detected the change)
- **Survived**: Mutations where tests passed (missed the change)
- **Incompilable**: Mutations that didn't compile
- **Timed out**: Mutations that exceeded the timeout

**Mutation Score** = Killed / (Killed + Survived) × 100%

(Incompilable and timed-out mutations are excluded from the score.)

### Step 4.2: Determine quality rating

| Score | Rating |
|-------|--------|
| 90–100% | **Strong** — Test suite thoroughly verifies the behavioral intent of this code. |
| 75–89% | **Adequate** — Good behavioral coverage with some gaps to address. |
| 60–74% | **Weak** — Significant behavioral gaps. The surviving mutations represent untested business logic. |
| Below 60% | **Critical** — Tests do not adequately verify the code's behavior. Major test improvements needed. |

### Step 4.3: Generate the report

Follow the report template at `skills/mutate/report-template.md` and the example at `skills/mutate/examples/sample-report.md`.

For each **surviving mutation**, provide:
1. What was changed and where
2. Why it matters (what behavior is untested)
3. A complete, runnable suggested test case in ScalaTest style matching the project's existing test patterns

### Step 4.4: Output the report

Output the complete report directly in the conversation. Do NOT write it to a file unless the user asks.

---

## Phase 5 — Final Safety Verification

### Step 5.1: Verify file integrity

Read the source file one final time and compare its content to the original stored in Phase 0.

### Step 5.2: Restore if needed

If there is ANY discrepancy — even a single character — immediately restore the file using Write with the stored original content. Then inform the user:

> **Safety check**: Detected file discrepancy after mutation testing. The file has been restored to its original state.

### Step 5.3: Confirm clean state

If the file matches the original, confirm:

> **File integrity verified**: `source_file_path` matches its original content. No mutations remain.

---

## Critical Rules

1. **NEVER modify test files** — only the source file under test
2. **NEVER run `sbt test`** — always use `sbt testOnly` with specific test classes
3. **ALWAYS restore after every mutation** — use full Write, not reverse Edit
4. **ALWAYS use 120-second timeouts** for sbt commands
5. **Cap at 25 mutations** — select the most impactful ones if more exist
6. **Stop on dirty git state** — do not proceed if there are uncommitted changes
7. **Wait for user confirmation** after presenting the mutation plan before testing
8. **One mutation at a time** — never apply multiple mutations simultaneously
