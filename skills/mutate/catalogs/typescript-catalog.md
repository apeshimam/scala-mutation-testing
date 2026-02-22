# TypeScript Mutation Catalog

This catalog defines **12 categories** of TypeScript-specific mutations. These supplement the shared catalog (`shared-catalog.md`) with mutations targeting TypeScript idioms and APIs.

---

## Category 1: Optional Chaining

Mutate optional chaining to verify null safety handling.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `a?.b` | `a.b` | Verify optional access is needed (remove safety) |
| `a?.b` | `undefined` | Verify non-null path is tested |
| `a?.b()` | `a.b()` | Verify optional call is needed |
| `a?.b()` | `undefined` | Verify method result matters |
| `a?.[index]` | `a[index]` | Verify optional indexing is needed |
| `a?.b?.c` | `a?.b.c` | Verify intermediate optional matters |

---

## Category 2: Nullish Coalescing

Mutate nullish coalescing to verify null/undefined handling.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `a ?? b` | `a` | Verify fallback is tested |
| `a ?? b` | `b` | Verify primary value is tested |
| `a ?? b` | `a \|\| b` | Verify nullish vs falsy distinction |
| `a ?? b ?? c` | `a ?? c` | Verify middle fallback matters |

---

## Category 3: Type Guards

Mutate type guard expressions to verify type narrowing logic.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `typeof x === "string"` | `typeof x !== "string"` | Verify type check direction |
| `typeof x === "string"` | `typeof x === "number"` | Verify specific type matters |
| `x instanceof T` | `!(x instanceof T)` | Verify instanceof direction |
| `"key" in obj` | `!("key" in obj)` | Verify property check direction |
| Custom type guard `x is T` | Negate the guard body | Verify custom guard logic |
| `Array.isArray(x)` | `!Array.isArray(x)` | Verify array check direction |

---

## Category 4: Discriminated Unions

Mutate discriminated union handling to verify exhaustive matching.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `switch(x.kind) { case "A": bodyA; case "B": bodyB }` | Swap `bodyA` and `bodyB` | Verify branches produce different results |
| `if (x.type === "success") bodyA else bodyB` | Swap `bodyA` and `bodyB` | Verify discriminant controls flow |
| `switch(x.kind) { case "A": ...; case "B": ...; }` | Remove one case | Verify all cases are needed |
| Exhaustive switch with `default` | Remove `default` case | Verify default handling matters |

---

## Category 5: Promise/Async Operations

Mutate async/await and Promise handling.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `await promise` | `promise` | Verify await is needed (remove await) |
| `.then(f)` | `.then(() => undefined)` | Verify then callback matters |
| `.catch(handler)` | Remove `.catch(...)` | Verify error handling is tested |
| `.finally(cleanup)` | Remove `.finally(...)` | Verify cleanup is tested |
| `Promise.all(promises)` | `Promise.race(promises)` | Verify all-vs-race semantics |
| `Promise.race(promises)` | `Promise.all(promises)` | Verify race-vs-all semantics |
| `Promise.allSettled(promises)` | `Promise.all(promises)` | Verify settled-vs-all semantics |
| `async function f()` | `function f()` (remove async) | Verify async is needed |
| `await Promise.all([a, b])` | `await a; await b;` | Verify parallel vs sequential |

---

## Category 6: Enum Mutations

Mutate enum usage to verify correct enum values.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `Enum.MemberA` | `Enum.MemberB` | Verify specific enum member matters |
| `value === Enum.Member` | `value !== Enum.Member` | Verify enum comparison direction |
| Numeric enum value | Different numeric value | Verify enum value matters |
| String enum value | Different string value | Verify enum string matters |

---

## Category 7: Array Operations (TypeScript)

Mutate TypeScript array operations to verify data processing logic.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `.filter(p)` | `.filter(() => true)` | Verify predicate matters |
| `.filter(p)` | `.filter(() => false)` | Verify filter preserves some elements |
| `.map(f)` | `.map(x => x)` | Verify transformation matters |
| `.find(p)` | `undefined` | Verify find result is used |
| `.reduce(f, init)` | `init` | Verify reduction body executes |
| `.some(p)` | `.every(p)` | Verify existential vs universal |
| `.every(p)` | `.some(p)` | Verify universal vs existential |
| `.includes(x)` | `!arr.includes(x)` | Verify containment check direction |
| `[...arr]` | `arr` | Verify spread copy matters |
| `.flat()` | Remove `.flat()` | Verify flattening matters |
| `.flatMap(f)` | `.map(f)` | Verify flatMap vs map |
| `.sort(compareFn)` | Remove `.sort(...)` | Verify sorting matters |
| `.slice(a, b)` | Remove `.slice(...)` | Verify slicing matters |
| `.splice(i, n)` | Remove the call | Verify splice has effect |
| `arr.length` | `0` | Verify length is used |

---

## Category 8: Object Operations

Mutate object operations to verify structural handling.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `{...obj, key: value}` | `{...obj}` | Verify key override matters |
| `{...obj}` | `obj` | Verify spread copy matters |
| `{...a, ...b}` | `{...a}` | Verify second spread matters |
| `Object.keys(obj)` | `[]` | Verify keys are used |
| `Object.values(obj)` | `[]` | Verify values are used |
| `Object.entries(obj)` | `[]` | Verify entries are used |
| `Object.assign(target, source)` | `target` | Verify assign has effect |
| `delete obj.key` | Remove the delete | Verify deletion matters |
| `Object.freeze(obj)` | `obj` | Verify freeze is needed |

---

## Category 9: Type Assertions

Mutate type assertions to verify type handling.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `x as T` | `x as unknown as U` | Verify correct type is asserted |
| `x as T` | Remove assertion (`x`) | Verify assertion is needed |
| `<T>x` | `<U>x` | Verify correct type in angle bracket syntax |
| `x!` (non-null assertion) | `x` | Verify non-null assertion is needed |

**Selection guidance**: Only mutate type assertions that guard against runtime errors. Skip assertions used purely for type narrowing in safe contexts.

---

## Category 10: Template Literals

Mutate template literal expressions.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `` `${expr}` `` | `""` | Verify template result matters |
| `` `prefix ${expr} suffix` `` | `` `${expr}` `` | Verify surrounding text matters |
| `` `${expr}` `` | `` `${otherExpr}` `` | Verify correct expression is interpolated |
| Tagged template `` tag`...` `` | Plain template `` `...` `` | Verify tag function matters |

---

## Category 11: Logical Assignment

Mutate logical assignment operators.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `x ??= y` | Remove the statement | Verify nullish assignment matters |
| `x ??= y` | `x = y` | Verify conditional assignment vs always-assign |
| `x \|\|= y` | `x &&= y` | Verify OR-assign vs AND-assign |
| `x &&= y` | `x \|\|= y` | Verify AND-assign vs OR-assign |
| `x \|\|= y` | Remove the statement | Verify falsy assignment matters |

---

## Category 12: Control Flow Patterns (switch/if-else chains)

Mutate switch statements and if-else chains used for pattern-like matching.

| Original | Mutation | Rationale |
|----------|----------|-----------|
| `case "A": bodyA; case "B": bodyB` | Swap `bodyA` and `bodyB` | Verify case bodies differ |
| `case "A": ...; break` | Remove `break` | Verify fall-through behavior |
| `case "A": bodyA; default: bodyD` | Swap `bodyA` and `bodyD` | Verify specific case vs default |
| `if (a) bodyA; else if (b) bodyB` | Swap `bodyA` and `bodyB` | Verify condition-body pairing |
| `if (a) bodyA; else if (b) bodyB; else bodyC` | Remove one branch | Verify all branches needed |
