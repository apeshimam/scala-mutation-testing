# Shared Mutation Catalog

This catalog defines **8 categories** of semantically meaningful mutations that apply to all supported languages. Unlike mechanical AST transformations, these mutations target the **intent** of the code — testing whether the test suite verifies behavior, not just syntax.

Language-specific mutations are defined in separate catalogs (`scala-catalog.md`, `python-catalog.md`, `typescript-catalog.md`).

When selecting mutations, prioritize those that:
- Change observable behavior (return values, side effects, error handling)
- Target business logic rather than boilerplate
- Could represent realistic bugs a developer might introduce

---

## Category 1: Arithmetic Operators

Mutate arithmetic operations to verify calculations are tested.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `a + b` | `a - b` | Verify addition is intentional |
| `a - b` | `a + b` | Verify subtraction is intentional |
| `a * b` | `a / b` | Verify multiplication is intentional |
| `a / b` | `a * b` | Verify division is intentional |
| `a % b` | `a / b` | Verify modulo is intentional |
| `a + b` | `a` | Verify second operand matters |
| `-a` | `a` | Verify negation is intentional |

---

## Category 2: Comparison Operators

Mutate comparisons to verify boundary conditions are tested.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `a < b` | `a <= b` | Test boundary exclusion |
| `a <= b` | `a < b` | Test boundary inclusion |
| `a > b` | `a >= b` | Test boundary exclusion |
| `a >= b` | `a > b` | Test boundary inclusion |
| `a == b` | `a != b` | Verify equality check matters |
| `a != b` | `a == b` | Verify inequality check matters |
| `a < b` | `a > b` | Verify direction of comparison |
| `a < b` | `false` | Verify comparison isn't dead code |
| `a == b` | `true` | Verify equality check isn't always true |

---

## Category 3: Logical Operators

Mutate boolean logic to verify conditions are correctly composed.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `a && b` | `a \|\| b` | Verify AND vs OR matters |
| `a \|\| b` | `a && b` | Verify OR vs AND matters |
| `a && b` | `a` | Verify second condition matters |
| `a && b` | `b` | Verify first condition matters |
| `a \|\| b` | `a` | Verify fallback condition matters |
| `!a` | `a` | Verify negation is intentional |
| `a && b` | `true` | Verify conjunction isn't dead code |
| `a \|\| b` | `false` | Verify disjunction isn't dead code |

---

## Category 4: Boolean Literals

Mutate boolean constants to verify branching is tested.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `true` | `false` | Verify true value is intentional |
| `false` | `true` | Verify false value is intentional |
| `flag = true` | `flag = false` | Verify flag controls behavior |

**Selection guidance**: Only mutate booleans that influence control flow or return values. Skip boolean literals in annotations, configurations, or test setup.

---

## Category 5: Return Value Mutations

Replace method/function body expressions with boundary/zero/empty values.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| Return int expression | Replace with `0` | Verify non-zero result matters |
| Return int expression | Replace with `-1` | Verify specific value matters |
| Return int expression | Replace with `expr + 1` | Verify exact value matters |
| Return string expression | Replace with `""` | Verify non-empty result matters |
| Return boolean expression | Negate the expression | Verify boolean result matters |
| Return list/array expression | Replace with empty collection | Verify non-empty collection matters |
| Return expression | Negate/invert the expression | Verify the result is used |

---

## Category 6: String Operations

Mutate string handling to verify text processing.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `"literal string"` | `""` | Verify non-empty string matters |
| String interpolation/template | `""` | Verify interpolation result matters |
| `str.toUpperCase()` | `str.toLowerCase()` | Verify case transformation matters |
| `str.toLowerCase()` | `str.toUpperCase()` | Verify case transformation matters |
| `str.trim()` | `str` | Verify trimming matters |
| Substring extraction | `str` | Verify substring extraction matters |
| Starts-with check | Negate the check | Verify prefix check direction |
| Ends-with check | Negate the check | Verify suffix check direction |
| Contains check | Negate the check | Verify containment check direction |
| String replace | `str` (skip replacement) | Verify replacement matters |
| String split | Single-element result | Verify splitting matters |
| String concatenation | First operand only | Verify concatenation matters |

---

## Category 7: Numeric Literals

Mutate numeric constants to verify boundary conditions.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `n` (positive int) | `n + 1` | Off-by-one: verify exact value |
| `n` (positive int) | `n - 1` | Off-by-one: verify exact value |
| `0` | `1` | Verify zero is intentional |
| `1` | `0` | Verify one is intentional |
| `1` | `-1` | Verify sign matters |
| `n` | `-n` | Verify sign matters |
| `n` | `0` | Verify non-zero matters |
| Max value constant | `0` | Verify max boundary matters |
| `0.0` | `1.0` | Verify zero float/double matters |
| `timeout = n` | `timeout = 0` | Verify timeout value matters |

**Selection guidance**: Focus on literals that represent business logic boundaries (limits, thresholds, indices, sizes). Skip literals in logging, string formatting, or UI constants.

---

## Category 8: Exception Handling

Mutate error handling to verify exception paths are tested.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| try/catch block | Remove catch handler | Verify exception handling is tested |
| Catch specific exception | Catch broad exception | Verify specific catch matters |
| Catch with recovery | Catch and re-throw | Verify recovery logic is tested |
| Finally/cleanup block | Remove cleanup | Verify cleanup is tested |
| Throw exception | Remove throw (return default) | Verify exception is expected |
| Throw specific exception | Throw generic exception | Verify exception type matters |

---

## Higher-Order Mutations

Higher-order mutations combine two first-order mutations simultaneously. A "subsuming" higher-order mutant is harder to kill than either constituent mutation alone, revealing deeper test gaps.

### When to Use

- When first-order mutation score is 90%+ (test suite is already strong)
- When `--higher-order` flag is specified
- To find compensating mutation pairs where two bugs cancel each other out

### Pair Selection Strategy

Select pairs where the two mutations might **compensate** — where one mutation's effect could mask the other's:

| Pair Type | Example | Rationale |
|-----------|---------|-----------|
| Comparison + Return value | `>=` → `>` AND return `x` → `x + 1` | Off-by-one in check compensated by off-by-one in result |
| Filter + Size check | `filter(p)` → `filterNot(p)` AND `isEmpty` → `nonEmpty` | Inverted filter compensated by inverted emptiness check |
| Boolean + Logical | `true` → `false` AND `&&` → `\|\|` | Flipped boolean with flipped logic can restore original behavior |
| Arithmetic + Comparison | `a + b` → `a - b` AND `>` → `<` | Sign change compensated by direction change |
| Negation + Branch swap | `!condition` → `condition` AND swap if/else bodies | Double negation restores original behavior |

### Guidelines

- **Cap at 10 higher-order mutations** per run
- Only pair mutations from **killed** first-order mutants (pairing survivors has no diagnostic value)
- Prefer pairs in the **same method** or closely related methods
- Prefer pairs from **different categories** for maximum diversity
- A surviving higher-order mutant means the tests verify each behavior independently but not their interaction

---

## Mutation Selection Guidelines

When building a mutation plan for a file:

1. **Prioritize by impact**: Start with mutations most likely to reveal untested behavior
2. **One mutation at a time**: Never apply multiple mutations simultaneously (except higher-order pairs)
3. **Semantic over syntactic**: Prefer mutations that change meaning over mechanical swaps
4. **Skip trivial code**: Don't mutate logging, toString, hashCode, or pure boilerplate
5. **Cap at 25 first-order mutations**: Select the most impactful 25 if more candidates exist
6. **Diversify categories**: Cover multiple categories rather than exhausting one
7. **Target business logic**: Focus on methods/functions that implement domain rules
8. **Consider method scope**: If user specified a method/function, only mutate within that scope
9. **Use semantic clustering**: Group mutations that test the same behavioral property, pick one per cluster
10. **Respect diff scope**: If `--diff` mode is active, only mutate within changed line ranges
