# Scala Semantic Mutation Testing

A Claude Code plugin that performs **semantic mutation testing** on Scala source files. Unlike mechanical mutation tools (e.g., Stryker4s), this leverages Claude's code understanding to generate semantically meaningful mutations — testing whether your test suite actually verifies the *intent* of your code, not just its syntax.

## Requirements

- **sbt** build tool (1.4+ recommended for `--client` incremental compilation)
- **ScalaTest** test framework
- Clean git working tree (uncommitted changes to the target file will block the run)

## Installation

```bash
claude --plugin-dir /path/to/scala-mutation-testing
```

## Usage

```
/mutate src/main/scala/com/example/MyService.scala
```

Scope to a single method:

```
/mutate src/main/scala/com/example/MyService.scala validateEmail
```

### Flags

| Flag | Description |
|------|-------------|
| `--diff` | Only mutate lines changed in the current branch vs main |
| `--fix` | Write and verify tests for surviving mutations |
| `--higher-order` | Enable higher-order mutations (also auto-enabled at 90%+ score) |

Flags can be combined:

```
/mutate src/main/scala/com/example/MyService.scala --diff --fix
```

## What it does

1. **Safety checks** — verifies clean git state, stores original file, starts sbt server
2. **Locates tests** — finds test files by convention (`*Spec`, `*Test`, `*Suite`) and grep
3. **Analyzes your code** — identifies mutation targets across 18 categories using semantic clustering
4. **Applies mutations one at a time** — edits source, runs `sbt testOnly` (incremental), restores original
5. **Detects equivalent mutants** — analyzes survivors to identify mutations that don't change behavior
6. **Higher-order mutations** — optionally combines killed mutations to find deeper gaps
7. **Generates a report** — adjusted score, surviving mutation analysis, suggested test cases
8. **Auto-fix** (`--fix`) — writes suggested tests, verifies they pass on original and kill mutants

## Key Features

### Semantic Clustering
Mutations are grouped by the behavioral property they test. One representative is picked per cluster, maximizing diversity and avoiding redundant mutations that would produce the same test outcome.

### Equivalent Mutant Detection
Surviving mutations are analyzed to determine if they actually change observable behavior. Equivalent mutants are excluded from the kill rate, giving you a more accurate picture of your test suite's strength.

### Diff-Scoped Mode (`--diff`)
Only mutates lines changed in the current branch vs the base branch. Ideal for CI/PR workflows — test the code you actually changed.

### Higher-Order Mutations (`--higher-order`)
Combines two first-order mutations simultaneously to find compensating pairs where two bugs cancel each other out. Auto-enabled when first-order score is 90%+.

### Auto-Fix Mode (`--fix`)
For each surviving mutation, writes a test case into the test file, then verifies it:
1. Passes on the original code
2. Fails when the mutation is applied

Tests that fail verification are automatically removed.

### Incremental Compilation
Uses `sbt --client` to keep the sbt server running across mutations. Zinc tracks file-level dependencies, so each mutation only recompiles the single changed file.

## Mutation Categories

| # | Category | Examples |
|---|----------|----------|
| 1 | Arithmetic | `+` ↔ `-`, `*` ↔ `/` |
| 2 | Comparison | `<` ↔ `<=`, `==` ↔ `!=` |
| 3 | Logical | `&&` ↔ `\|\|`, negate `!` |
| 4 | Boolean literals | `true` ↔ `false` |
| 5 | Return values | Replace with zero/empty/negated |
| 6 | Option | `Some(x)` → `None`, `getOrElse` → always default |
| 7 | Either | `Right` ↔ `Left` |
| 8 | Try | `Success` → `Failure`, remove `.recover` |
| 9 | Collections | `filter` ↔ `filterNot`, `exists` ↔ `forall` |
| 10 | Pattern matching | Swap branches, remove/negate guards |
| 11 | For comprehensions | Remove/negate guards |
| 12 | Higher-order functions | Replace with identity/constant |
| 13 | Implicits/givens | Remove implicits, reverse Ordering |
| 14 | Case class copy | Skip copy, drop field updates |
| 15 | String | Empty strings, swap case transforms |
| 16 | Numeric literals | Off-by-one, negate, zero substitution |
| 17 | Exception handling | Remove catch/finally, broaden catch |
| 18 | Higher-order (combined) | Compensating mutation pairs |

## Safety

- Blocks on uncommitted changes to the target file
- Full file restore (Write, not reverse-Edit) after every mutation
- Post-run integrity verification with automatic restore on discrepancy
- Never modifies test files (except in `--fix` mode, which only adds tests)
- Always uses `sbt testOnly`, never `sbt test`
- 25 first-order mutation cap, 10 higher-order cap, 120s timeout per sbt command
- sbt server shut down after run completes

## License

MIT
