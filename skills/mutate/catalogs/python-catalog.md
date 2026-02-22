# Python Mutation Catalog

This catalog defines **13 categories** of Python-specific mutations. These supplement the shared catalog (`shared-catalog.md`) with mutations targeting Python idioms and APIs.

---

## Category 1: Dictionary Operations

Mutate dictionary handling to verify key/value logic.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `dict.get(k, default)` | `default` | Verify key-present path is tested |
| `dict[k]` | `dict.get(k)` | Verify KeyError matters (None vs exception) |
| `dict.pop(k)` | `del dict[k]` | Verify return value of pop is used |
| `dict.pop(k, default)` | `default` | Verify key-present path is tested |
| `dict.update(other)` | Remove the call | Verify update has an effect |
| `k in dict` | `k not in dict` | Verify membership check direction |
| `dict.setdefault(k, v)` | `dict[k]` | Verify default-setting behavior matters |
| `dict.keys()` | `dict.values()` | Verify keys vs values matters |
| `dict.items()` | `dict.keys()` | Verify iteration over pairs matters |

---

## Category 2: List/Dict/Set Comprehensions

Mutate comprehension expressions to verify filtering and transformation logic.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `[x for x in xs if cond]` | `[x for x in xs]` | Verify guard filters results (remove guard) |
| `[x for x in xs if cond]` | `[x for x in xs if not cond]` | Verify guard direction matters |
| `[f(x) for x in xs]` | `list(xs)` | Verify transformation function matters |
| `[f(x) for x in xs]` | `[x for x in xs]` | Verify transform isn't identity |
| `{k: v for k, v in items}` | `{}` | Verify comprehension result matters |
| `{k: v for k, v in items if cond}` | `{k: v for k, v in items}` | Verify dict comprehension guard |
| `{x for x in xs if cond}` | `{x for x in xs}` | Verify set comprehension guard |
| Generator expression | List comprehension | Verify lazy evaluation matters |

---

## Category 3: Context Managers

Mutate `with` statement handling to verify resource management.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `with open(f) as fh: body` | Remove body (replace with `pass`) | Verify context manager body has effect |
| `with resource as r: body` | `body` without `with` wrapping | Verify resource cleanup matters |
| `with A() as a, B() as b: body` | `with A() as a: body` | Verify second context manager matters |

**Selection guidance**: Focus on context managers that manage I/O, locks, database transactions, or temporary state. Skip trivial context managers.

---

## Category 4: Decorators

Mutate decorator usage to verify decorator behavior.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `@decorator` on function | Remove the decorator | Verify decorator has an effect |
| `@decorator_a` then `@decorator_b` | Swap decorator order | Verify decorator ordering matters |
| `@staticmethod` | `@classmethod` | Verify method type matters |
| `@classmethod` | `@staticmethod` | Verify class access matters |
| `@property` | Remove decorator | Verify property access pattern matters |
| `@cache` / `@lru_cache` | Remove decorator | Verify caching behavior is tested |

**Selection guidance**: Only mutate decorators that affect runtime behavior. Skip decorators used purely for documentation or type checking.

---

## Category 5: Generators/Iterators

Mutate generator behavior to verify lazy evaluation and yielding logic.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `yield x` | `return x` | Verify generator protocol matters |
| `yield from xs` | `yield xs` | Verify delegation vs single yield |
| `yield x` | `yield None` | Verify yielded value matters |
| Generator function | Return list equivalent | Verify lazy evaluation matters |
| `next(iterator)` | `None` | Verify next value is used |
| `next(iterator, default)` | `default` | Verify iterator-has-values path |

---

## Category 6: Default/Mutable Arguments

Mutate default argument values to verify they matter.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `def f(x=[])` | `def f(x=None)` | Verify mutable default behavior |
| `def f(x=None)` | `def f(x=[])` | Verify None-check logic |
| `def f(x=0)` | `def f(x=1)` | Verify default value matters |
| `def f(x="")` | `def f(x="default")` | Verify empty default matters |
| `def f(x=True)` | `def f(x=False)` | Verify boolean default matters |

---

## Category 7: Type-Specific Operations

Mutate type checks and identity comparisons.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `isinstance(x, T)` | `not isinstance(x, T)` | Verify type check direction |
| `isinstance(x, (A, B))` | `isinstance(x, A)` | Verify multi-type check breadth |
| `x is None` | `x is not None` | Verify None check direction |
| `x is not None` | `x is None` | Verify non-None check direction |
| `x is y` | `x == y` | Verify identity vs equality matters |
| `x == y` | `x is y` | Verify equality vs identity matters |
| `type(x) is T` | `isinstance(x, T)` | Verify exact type vs inheritance |
| `hasattr(x, "attr")` | `not hasattr(x, "attr")` | Verify attribute check direction |

---

## Category 8: Unpacking/Starred Expressions

Mutate unpacking and starred expressions.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `a, b = pair` | `b, a = pair` | Verify unpacking order matters |
| `first, *rest = items` | `*rest, first = items` | Verify head vs tail matters |
| `f(*args)` | `f(args)` | Verify spreading matters |
| `f(**kwargs)` | `f(kwargs)` | Verify dict spreading matters |
| `{**dict_a, **dict_b}` | `{**dict_a}` | Verify second dict merge matters |

---

## Category 9: Collection Operations (Python)

Mutate Python collection operations to verify data processing logic.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `list.append(x)` | Remove the call | Verify append has an effect |
| `list.extend(xs)` | Remove the call | Verify extend has an effect |
| `list.insert(i, x)` | `list.append(x)` | Verify insertion position matters |
| `sorted(x)` | `x` | Verify sorting matters |
| `sorted(x, key=f)` | `sorted(x)` | Verify sort key matters |
| `sorted(x, reverse=True)` | `sorted(x)` | Verify sort direction matters |
| `reversed(x)` | `x` | Verify reversal matters |
| `filter(f, x)` | `x` | Verify filter has an effect |
| `map(f, x)` | `x` | Verify map transformation matters |
| `any(iterable)` | `all(iterable)` | Verify existential vs universal |
| `all(iterable)` | `any(iterable)` | Verify universal vs existential |
| `len(x) == 0` | `len(x) != 0` | Verify emptiness check direction |
| `len(x)` | `0` | Verify length is used |
| `enumerate(xs)` | `xs` | Verify index is used |
| `zip(a, b)` | `a` | Verify pairing matters |
| `set(xs)` | `xs` | Verify deduplication matters |
| `list(xs)` | `xs` | Verify materialization matters |

---

## Category 10: Pattern Matching (Python 3.10+)

Mutate `match`/`case` statements to verify branch coverage.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `case A: body_a; case B: body_b` | Swap `body_a` and `body_b` | Verify branches produce different results |
| `case pattern if guard:` | `case pattern:` | Verify guard condition matters |
| `case pattern if guard:` | `case pattern if not guard:` | Verify guard direction matters |
| `case [first, *rest]:` | `case []:` | Verify sequence decomposition matters |
| `case {"key": value}:` | `case {}:` | Verify mapping pattern matters |
| `case _: default_body` | Remove default case | Verify exhaustiveness matters |

---

## Category 11: Walrus Operator

Mutate `:=` assignment expressions (Python 3.8+).

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `if (n := len(x)) > 10:` | `if len(x) > 10:` | Verify assigned name is used later |
| `while chunk := f.read(8192):` | `while True:` | Verify walrus controls loop |
| `[y for x in xs if (y := f(x))]` | `[f(x) for x in xs]` | Verify walrus-in-comprehension matters |

---

## Category 12: F-String Mutations

Mutate f-string expressions.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `f"...{expr}..."` | `f"...{other_expr}..."` | Verify correct expression is interpolated |
| `f"...{expr}..."` | `""` | Verify f-string result matters |
| `f"...{expr:.2f}..."` | `f"...{expr}..."` | Verify format spec matters |
| `f"prefix {expr} suffix"` | `f"{expr}"` | Verify surrounding text matters |

---

## Category 13: Property Mutations

Mutate property-decorated methods.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `@property` getter returns `expr` | Return different value | Verify property value matters |
| `@x.setter` with validation | Remove validation | Verify setter validates input |
| `@x.setter` sets `self._x = value` | Skip assignment | Verify setter stores value |
| `@x.deleter` cleanup | Remove cleanup body | Verify deleter has effect |
