# Semantic Mutation Testing

A Claude Code plugin that performs **semantic mutation testing** on source files. Unlike mechanical mutation tools, this leverages Claude's code understanding to generate semantically meaningful mutations — testing whether your test suite actually verifies the *intent* of your code, not just its syntax.

## Supported Languages

| Language | Extensions | Build Tool | Test Framework |
|----------|-----------|------------|----------------|
| **Scala** | `.scala` | sbt (1.4+ for `--client`) | ScalaTest |
| **Python** | `.py` | None (any package manager) | pytest, unittest |
| **TypeScript** | `.ts`, `.tsx` | npm/yarn/pnpm/bun | Jest, Vitest, Mocha |

## Requirements

- **Git** — clean working tree required (uncommitted changes to the target file will block the run)
- **Scala**: sbt build tool (1.4+ recommended for incremental compilation via `--client`)
- **Python**: pytest (or unittest); Poetry, Pipenv, uv, or pip for dependency management
- **TypeScript**: `package.json` with a test framework; `tsconfig.json` for compilation checks

## Installation

```bash
claude install-skill /path/to/mutation-testing
```

## Usage

```
/mutate src/main/scala/com/example/MyService.scala
/mutate src/services/user_service.py
/mutate src/services/user-service.ts
```

Scope to a single method/function:

```
/mutate src/main/scala/com/example/MyService.scala validateEmail
/mutate src/services/user_service.py validate_email
/mutate src/services/user-service.ts validateEmail
```

### Flags

| Flag | Description |
|------|-------------|
| `--diff` | Only mutate lines changed in the current branch vs main |
| `--fix` | Write and verify tests for surviving mutations |
| `--higher-order` | Enable higher-order mutations (also auto-enabled at 90%+ score) |

Flags can be combined:

```
/mutate src/services/user_service.py --diff --fix
```

## What It Does

1. **Safety checks** — verifies clean git state, stores original file, initializes build tools
2. **Locates tests** — finds test files by naming convention and import analysis
3. **Analyzes your code** — identifies mutation targets using semantic clustering across shared + language-specific categories
4. **Applies mutations one at a time** — edits source, runs targeted tests, restores original
5. **Detects equivalent mutants** — analyzes survivors to identify mutations that don't change behavior
6. **Higher-order mutations** — optionally combines killed mutations to find deeper gaps
7. **Generates a report** — adjusted score, surviving mutation analysis, suggested test cases
8. **Auto-fix** (`--fix`) — writes suggested tests, verifies they pass on original and kill mutants

## Mutation Categories

### Shared (All Languages)

| # | Category | Examples |
|---|----------|----------|
| 1 | Arithmetic | `+` ↔ `-`, `*` ↔ `/` |
| 2 | Comparison | `<` ↔ `<=`, `==` ↔ `!=` |
| 3 | Logical | `&&` ↔ `\|\|`, negate `!` |
| 4 | Boolean literals | `true` ↔ `false` |
| 5 | Return values | Replace with zero/empty/negated |
| 6 | String operations | Empty strings, swap case transforms |
| 7 | Numeric literals | Off-by-one, negate, zero substitution |
| 8 | Exception handling | Remove catch/finally, broaden catch |

### Scala-Specific (9 categories)

Option semantics, Either semantics, Try semantics, collection operations, pattern matching, for comprehensions, higher-order functions, implicits/givens, case class copy.

### Python-Specific (13 categories)

Dictionary operations, list/dict/set comprehensions, context managers, decorators, generators/iterators, default/mutable arguments, type-specific operations, unpacking/starred, collection operations, pattern matching (3.10+), walrus operator, f-string mutations, property mutations.

### TypeScript-Specific (12 categories)

Optional chaining, nullish coalescing, type guards, discriminated unions, Promise/async operations, enum mutations, array operations, object operations, type assertions, template literals, logical assignment, control flow patterns.

## Key Features

### Semantic Clustering
Mutations are grouped by the behavioral property they test. One representative is picked per cluster, maximizing diversity and avoiding redundant mutations that would produce the same test outcome.

### Equivalent Mutant Detection
Surviving mutations are analyzed to determine if they actually change observable behavior. Equivalent mutants are excluded from the kill rate, giving you a more accurate picture of your test suite's strength.

### Overmocking Detection
Test files are analyzed for mocking anti-patterns (mocking the SUT, high mock-to-assertion ratios, excessive wildcard matchers, verify-only tests). Surviving mutations are cross-referenced with overmocking findings.

### Diff-Scoped Mode (`--diff`)
Only mutates lines changed in the current branch vs the base branch. Ideal for CI/PR workflows — test the code you actually changed.

### Higher-Order Mutations (`--higher-order`)
Combines two first-order mutations simultaneously to find compensating pairs where two bugs cancel each other out. Auto-enabled when first-order score is 90%+.

### Auto-Fix Mode (`--fix`)
For each surviving mutation, writes a test case into the test file, then verifies it:
1. Passes on the original code
2. Fails when the mutation is applied

Tests that fail verification are automatically removed.

### Incremental Compilation (Scala)
Uses `sbt --client` to keep the sbt server running across mutations. Zinc tracks file-level dependencies, so each mutation only recompiles the single changed file.

## Safety

- Blocks on uncommitted changes to the target file
- Full file restore (Write, not reverse-Edit) after every mutation
- Post-run integrity verification with automatic restore on discrepancy
- Never modifies test files (except in `--fix` mode, which only adds tests)
- Always uses targeted test commands, never runs the full test suite
- 25 first-order mutation cap, 10 higher-order cap, 120s timeout per test command
- Build server cleanup after run completes (Scala)

## License

MIT
