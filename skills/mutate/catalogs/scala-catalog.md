# Scala Mutation Catalog

This catalog defines **9 categories** of Scala-specific mutations. These supplement the shared catalog (`shared-catalog.md`) with mutations targeting Scala idioms and APIs.

---

## Category 1: Option Semantics

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

## Category 2: Either Semantics

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

## Category 3: Try Semantics

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

## Category 4: Collection Operations

Mutate Scala collection operations to verify data processing logic.

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

## Category 5: Pattern Matching

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

## Category 6: For Comprehensions

Mutate for-comprehension guards and generators.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `for { x <- xs if guard } yield expr` | `for { x <- xs } yield expr` | Verify guard filters results |
| `for { x <- xs if guard } yield expr` | `for { x <- xs if !guard } yield expr` | Verify guard direction matters |
| `for { x <- xs; y <- ys } yield expr` | `for { x <- xs } yield expr` | Verify second generator matters |
| `for { x <- xs } yield f(x)` | `for { x <- xs } yield x` | Verify yield transformation matters |

---

## Category 7: Higher-Order Functions

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

## Category 8: Implicits and Givens

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

## Category 9: Case Class Copy

Mutate case class copy calls to verify field updates are tested.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `obj.copy(field = newValue)` | `obj` | Verify copy update matters |
| `obj.copy(field = newValue)` | `obj.copy(field = differentValue)` | Verify specific value matters |
| `obj.copy(a = x, b = y)` | `obj.copy(a = x)` | Verify all fields are updated |
| `obj.copy(a = x, b = y)` | `obj.copy(b = y)` | Verify first field update matters |
