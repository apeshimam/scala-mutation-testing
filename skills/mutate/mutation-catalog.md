# Scala Semantic Mutation Catalog

This catalog defines 17 categories of semantically meaningful mutations for Scala code. Unlike mechanical AST transformations, these mutations target the **intent** of the code — testing whether the test suite verifies behavior, not just syntax.

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
| `val flag = true` | `val flag = false` | Verify flag controls behavior |

**Selection guidance**: Only mutate booleans that influence control flow or return values. Skip boolean literals in annotations, configurations, or test setup.

---

## Category 5: Return Value Mutations

Replace method body expressions with boundary/zero/empty values.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `def foo: Int = expr` | Replace body with `0` | Verify non-zero result matters |
| `def foo: Int = expr` | Replace body with `-1` | Verify specific value matters |
| `def foo: Int = expr` | Replace body with `expr + 1` | Verify exact value matters |
| `def foo: String = expr` | Replace body with `""` | Verify non-empty result matters |
| `def foo: Boolean = expr` | Replace body with `!expr` | Verify boolean result matters |
| `def foo: List[A] = expr` | Replace body with `Nil` | Verify non-empty collection matters |
| `def foo: A = expr` | Negate/invert the expression | Verify the result is used |

---

## Category 6: Option Semantics

Mutate Option handling — a critical Scala idiom for null safety.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `Some(x)` | `None` | Verify presence is handled |
| `None` | `Some(defaultValue)` | Verify absence is handled |
| `option.isDefined` | `option.isEmpty` | Verify presence check direction |
| `option.isEmpty` | `option.isDefined` | Verify emptiness check direction |
| `option.getOrElse(default)` | `default` | Verify Some path is tested |
| `option.getOrElse(default)` | `option.get` | Verify None path is tested |
| `option.map(f)` | `None` | Verify transformation matters |
| `option.flatMap(f)` | `None` | Verify chained computation matters |
| `option.fold(z)(f)` | `z` | Verify Some branch is tested |
| `option.orElse(alternative)` | `option` | Verify fallback matters |
| `option.orElse(alternative)` | `alternative` | Verify primary matters |
| `option.filter(p)` | `option` | Verify filter condition matters |
| `option.filter(p)` | `None` | Verify filter allows some values |
| `option.contains(x)` | `!option.contains(x)` | Verify containment check direction |

---

## Category 7: Either Semantics

Mutate Either handling for error/success path coverage.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `Right(x)` | `Left(errorValue)` | Verify success path is tested |
| `Left(x)` | `Right(defaultValue)` | Verify error path is tested |
| `either.isRight` | `either.isLeft` | Verify success/failure check direction |
| `either.isLeft` | `either.isRight` | Verify failure/success check direction |
| `either.map(f)` | `either` | Verify Right transformation matters |
| `either.leftMap(f)` | `either` | Verify Left transformation matters |
| `either.flatMap(f)` | `Left(errorValue)` | Verify chained success path |
| `either.getOrElse(default)` | `default` | Verify Right value is used |
| `either.fold(fa, fb)` | `fa(leftValue)` | Verify Right branch is tested |
| `either.fold(fa, fb)` | `fb(rightValue)` | Verify Left branch is tested |
| `either.swap` | `either` | Verify swap is intentional |

---

## Category 8: Try Semantics

Mutate Try handling for error recovery coverage.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `Success(x)` | `Failure(new RuntimeException("mutant"))` | Verify success path is tested |
| `Failure(e)` | `Success(defaultValue)` | Verify failure path is tested |
| `Try(expr)` | `Failure(new RuntimeException("mutant"))` | Verify Try wrapping matters |
| `tryVal.recover { ... }` | `tryVal` | Verify recovery is tested |
| `tryVal.recoverWith { ... }` | `tryVal` | Verify recovery chain matters |
| `tryVal.getOrElse(default)` | `default` | Verify Success path is tested |
| `tryVal.toOption` | `None` | Verify conversion preserves value |
| `tryVal.isSuccess` | `tryVal.isFailure` | Verify success check direction |
| `tryVal.map(f)` | `tryVal` | Verify transformation matters |

---

## Category 9: Collection Operations

Mutate collection operations to verify data processing logic.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `list.filter(p)` | `list.filterNot(p)` | Verify filter direction matters |
| `list.filterNot(p)` | `list.filter(p)` | Verify negated filter matters |
| `list.exists(p)` | `list.forall(p)` | Verify existential vs universal |
| `list.forall(p)` | `list.exists(p)` | Verify universal vs existential |
| `list.headOption` | `None` | Verify first element is used |
| `list.lastOption` | `None` | Verify last element is used |
| `list.isEmpty` | `list.nonEmpty` | Verify emptiness check direction |
| `list.nonEmpty` | `list.isEmpty` | Verify non-emptiness check direction |
| `list.map(f)` | `list` | Verify transformation matters |
| `list.flatMap(f)` | `Nil` | Verify flat-mapped result matters |
| `list.foldLeft(z)(op)` | `z` | Verify fold body executes |
| `list.take(n)` | `list` | Verify take limit matters |
| `list.drop(n)` | `list` | Verify drop offset matters |
| `list.sorted` | `list` | Verify sorting matters |
| `list.sortBy(f)` | `list` | Verify sort key matters |
| `list.distinct` | `list` | Verify deduplication matters |
| `list.reverse` | `list` | Verify order matters |
| `list.zip(other)` | `list.map(x => (x, x))` | Verify pairing matters |
| `list.find(p)` | `None` | Verify find result is used |
| `list.contains(x)` | `!list.contains(x)` | Verify containment check direction |
| `list.size` | `0` | Verify size is used |
| `list ++ other` | `list` | Verify concatenation matters |
| `list ++ other` | `other` | Verify first list matters |

---

## Category 10: Pattern Matching

Mutate match expressions to verify branch coverage.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `case A => exprA; case B => exprB` | `case A => exprB; case B => exprA` | Verify branches produce different results |
| `case x if guard => expr` | `case x => expr` | Verify guard condition matters |
| `case x if guard => expr` | `case x if !guard => expr` | Verify guard direction matters |
| `case Some(x) => a; case None => b` | `case Some(x) => b; case None => a` | Verify Option matching matters |
| `case Right(x) => a; case Left(e) => b` | `case Right(x) => b; case Left(e) => a` | Verify Either matching matters |
| `case head :: tail => expr` | `case Nil => defaultExpr` | Verify list decomposition matters |
| `case _ => default` | Remove default case (if other cases exist) | Verify exhaustiveness matters |
| Multi-case with `\|` | Remove one alternative | Verify all alternatives needed |

---

## Category 11: For Comprehensions

Mutate for-comprehension guards and generators.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `for { x <- xs if guard } yield expr` | `for { x <- xs } yield expr` | Verify guard filters results |
| `for { x <- xs if guard } yield expr` | `for { x <- xs if !guard } yield expr` | Verify guard direction matters |
| `for { x <- xs; y <- ys } yield expr` | `for { x <- xs } yield expr` | Verify second generator matters |
| `for { x <- xs } yield f(x)` | `for { x <- xs } yield x` | Verify yield transformation matters |

---

## Category 12: Higher-Order Functions

Mutate function arguments to verify callbacks are meaningful.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `list.map(f)` | `list.map(identity)` | Verify transformation function matters |
| `list.map(f)` | `list.map(_ => constantValue)` | Verify function isn't constant |
| `list.filter(p)` | `list.filter(_ => true)` | Verify predicate isn't always true |
| `list.filter(p)` | `list.filter(_ => false)` | Verify predicate isn't always false |
| `list.sortBy(f)` | `list.sortBy(_ => 0)` | Verify sort key differentiates |
| `list.groupBy(f)` | `list.groupBy(_ => "constant")` | Verify grouping function matters |
| `list.reduce(op)` | Replace op with `(a, _) => a` | Verify reduction uses both args |
| `list.reduce(op)` | Replace op with `(_, b) => b` | Verify reduction uses both args |

---

## Category 13: Implicits and Givens

Mutate implicit behavior to verify implicit dependencies.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `(implicit ord: Ordering[T])` | `Ordering[T].reverse` | Verify ordering direction matters |
| `implicit val ec: ExecutionContext` | Remove/change to global EC | Verify specific EC matters |
| `implicitly[TypeClass[T]]` | Provide alternative instance | Verify specific instance matters |
| `given Ordering[T] = ...` | `given Ordering[T] = ... .reverse` | Verify given ordering direction |
| `using` parameter | Remove or substitute | Verify context parameter is used |

**Selection guidance**: Only mutate implicits that affect observable behavior. Skip implicits used purely for type inference or compilation.

---

## Category 14: Case Class Copy

Mutate case class copy calls to verify field updates are tested.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `obj.copy(field = newValue)` | `obj` | Verify copy update matters |
| `obj.copy(field = newValue)` | `obj.copy(field = differentValue)` | Verify specific value matters |
| `obj.copy(a = x, b = y)` | `obj.copy(a = x)` | Verify all fields are updated |
| `obj.copy(a = x, b = y)` | `obj.copy(b = y)` | Verify first field update matters |

---

## Category 15: String Operations

Mutate string handling to verify text processing.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `"literal string"` | `""` | Verify non-empty string matters |
| `s"interpolated $var"` | `""` | Verify interpolation result matters |
| `str.toUpperCase` | `str.toLowerCase` | Verify case transformation matters |
| `str.toLowerCase` | `str.toUpperCase` | Verify case transformation matters |
| `str.trim` | `str` | Verify trimming matters |
| `str.strip` | `str` | Verify stripping matters |
| `str.substring(a, b)` | `str` | Verify substring extraction matters |
| `str.startsWith(prefix)` | `!str.startsWith(prefix)` | Verify prefix check direction |
| `str.endsWith(suffix)` | `!str.endsWith(suffix)` | Verify suffix check direction |
| `str.contains(sub)` | `!str.contains(sub)` | Verify containment check direction |
| `str.replace(a, b)` | `str` | Verify replacement matters |
| `str.split(delim)` | `Array(str)` | Verify splitting matters |
| `str + other` | `str` | Verify concatenation matters |

---

## Category 16: Numeric Literals

Mutate numeric constants to verify boundary conditions.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `n` (positive Int) | `n + 1` | Off-by-one: verify exact value |
| `n` (positive Int) | `n - 1` | Off-by-one: verify exact value |
| `0` | `1` | Verify zero is intentional |
| `1` | `0` | Verify one is intentional |
| `1` | `-1` | Verify sign matters |
| `n` | `-n` | Verify sign matters |
| `n` | `0` | Verify non-zero matters |
| `Int.MaxValue` | `0` | Verify max boundary matters |
| `0.0` | `1.0` | Verify zero double matters |
| `timeout = n` | `timeout = 0` | Verify timeout value matters |

**Selection guidance**: Focus on literals that represent business logic boundaries (limits, thresholds, indices, sizes). Skip literals in logging, string formatting, or UI constants.

---

## Category 17: Exception Handling

Mutate error handling to verify exception paths are tested.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `try { ... } catch { case e: Ex => handler }` | `try { ... }` (remove catch) | Verify exception handling is tested |
| `catch { case e: SpecificEx => ... }` | `catch { case e: Exception => ... }` | Verify specific catch matters |
| `catch { case e: Ex => recovery }` | `catch { case e: Ex => throw e }` | Verify recovery logic is tested |
| `finally { cleanup }` | Remove finally block | Verify cleanup is tested |
| `throw new Exception(msg)` | Remove throw (return default) | Verify exception is expected |
| `throw new SpecificEx(msg)` | `throw new RuntimeException(msg)` | Verify exception type matters |

---

## Category 18: Higher-Order Mutations

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
| Option + Pattern match | `Some(x)` → `None` AND swap case branches | None value hits the swapped branch, producing original behavior |
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
7. **Target business logic**: Focus on methods that implement domain rules
8. **Consider method scope**: If user specified a method, only mutate within that method
9. **Use semantic clustering**: Group mutations that test the same behavioral property, pick one per cluster
10. **Respect diff scope**: If `--diff` mode is active, only mutate within changed line ranges
