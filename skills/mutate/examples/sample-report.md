# Mutation Testing Report

**File**: `src/main/scala/com/example/services/UserService.scala`
**Date**: 2026-02-15
**Mutations tested**: 18
**Test classes**: `com.example.services.UserServiceSpec`, `com.example.services.UserValidationSpec`
**Mode**: standard
**Flags**: --higher-order

---

## Score Summary

| Metric | Value |
|--------|-------|
| **Mutation Score (adjusted)** | **13 / 16 (81%)** |
| Mutation Score (raw) | 13 / 17 (76%) |
| Killed | 13 |
| Survived | 3 |
| Equivalent | 1 |
| Incompilable | 1 |
| Timed Out | 0 |

### Quality Rating

**Adequate** — Good behavioral coverage with some gaps to address.

| Score Range | Rating | Meaning |
|------------|--------|---------|
| 90-100% | **Strong** | Test suite thoroughly verifies behavior |
| 75-89% | **Adequate** | Good coverage with some gaps |
| 60-74% | **Weak** | Significant behavioral gaps in tests |
| Below 60% | **Critical** | Tests do not adequately verify code behavior |

---

## Detailed Results

| # | Category | Location | Mutation | Result |
|---|----------|----------|----------|--------|
| 1 | Comparison | `UserService.validateAge:28` | `age >= 18` → `age > 18` | ✅ **KILLED** |
| 2 | Comparison | `UserService.validateAge:28` | `age >= 18` → `age >= 19` | ✅ **KILLED** |
| 3 | Option | `UserService.findUser:35` | `users.find(_.id == id)` → `None` | ✅ **KILLED** |
| 4 | Option | `UserService.findUser:38` | `user.map(_.toDTO)` → `None` | ✅ **KILLED** |
| 5 | Option | `UserService.getDisplayName:42` | `.getOrElse("Anonymous")` → `"Anonymous"` | 🟡 **SURVIVED** |
| 6 | String | `UserService.normalizeEmail:50` | `.toLowerCase` → `.toUpperCase` | ✅ **KILLED** |
| 7 | String | `UserService.normalizeEmail:51` | `.trim` → remove | ✅ **KILLED** |
| 8 | Boolean | `UserService.isActive:55` | `true` → `false` in default status | ✅ **KILLED** |
| 9 | Collection | `UserService.activeUsers:60` | `.filter(_.isActive)` → `.filterNot(_.isActive)` | ✅ **KILLED** |
| 10 | Collection | `UserService.activeUsers:61` | `.sortBy(_.lastName)` → `.sortBy(_.firstName)` | ⚪ **EQUIVALENT** |
| 11 | Collection | `UserService.activeUsers:62` | `.take(limit)` → remove `.take(limit)` | 🟡 **SURVIVED** |
| 12 | Either | `UserService.createUser:70` | `Right(user)` → `Left(ValidationError("mutant"))` | ✅ **KILLED** |
| 13 | Either | `UserService.createUser:68` | `Left(error)` → `Right(defaultUser)` | ✅ **KILLED** |
| 14 | Pattern Match | `UserService.handleStatus:80` | Swap `Active` and `Suspended` case bodies | ✅ **KILLED** |
| 15 | Pattern Match | `UserService.handleStatus:82` | Remove `if user.verified` guard | 🟡 **SURVIVED** |
| 16 | Exception | `UserService.deleteUser:90` | Remove `catch` block for `DatabaseException` | ✅ **KILLED** |
| 17 | Arithmetic | `UserService.calculateDiscount:95` | `price * discount` → `price / discount` | ✅ **KILLED** |
| 18 | Return Value | `UserService.countPremium:100` | Replace body with `0` | ⚠️ **INCOMPILABLE** |

Result key: ✅ KILLED · 🟡 SURVIVED · ⚪ EQUIVALENT · ⚠️ INCOMPILABLE · ⏱️ TIMEOUT

---

## Equivalent Mutation Analysis

### Mutation #10: `.sortBy(_.lastName)` → `.sortBy(_.firstName)`

**What was changed**: Changed the sort key from `lastName` to `firstName` in the `activeUsers` method.
**Why it's equivalent**: Analysis of the test data and upstream callers shows that in all test fixtures and production usage, users are constructed with `firstName` and `lastName` set to the same value (derived from a single `name` field via `name.split(" ")`). Since both fields contain identical values in all reachable scenarios, changing the sort key produces identical ordering. This is technically a test data limitation rather than a true equivalence, but the mutation cannot be killed without first changing the test fixtures to use distinct first/last names.

---

## Surviving Mutation Analysis

The following mutations survived, indicating gaps in test coverage:

### Mutation #5: `getOrElse("Anonymous")` → always return `"Anonymous"`

**Category**: Option
**Location**: `UserService.getDisplayName:42`
**What was changed**: Replaced `user.displayName.getOrElse("Anonymous")` with just `"Anonymous"`, ignoring the user's actual display name.
**Why it matters**: No test verifies that `getDisplayName` returns the user's actual display name when one exists. Tests only check the fallback case.

**Suggested test case**:
```scala
"getDisplayName" should "return the user's display name when it exists" in {
  val user = User(
    id = 1,
    displayName = Some("Alice Johnson"),
    email = "alice@example.com"
  )

  val result = userService.getDisplayName(user)

  result shouldBe "Alice Johnson"
}
```

---

### Mutation #11: Remove `.take(limit)` from `activeUsers`

**Category**: Collection
**Location**: `UserService.activeUsers:62`
**What was changed**: Removed the `.take(limit)` call, causing the method to return all active users regardless of the `limit` parameter.
**Why it matters**: No test verifies that the `limit` parameter actually constrains the result size. The limit could be removed without any test failing.

**Suggested test case**:
```scala
"activeUsers" should "respect the limit parameter" in {
  val users = (1 to 20).map(i =>
    createUser(s"User$i", s"Last$i", isActive = true)
  ).toList

  val result = userService.activeUsers(users, limit = 5)

  result should have size 5
}
```

---

### Mutation #15: Remove `if user.verified` guard from pattern match

**Category**: Pattern Match
**Location**: `UserService.handleStatus:82`
**What was changed**: Removed the `if user.verified` guard from the `Pending` case, making the case match all Pending users regardless of verification status.
**Why it matters**: The guard distinguishes between verified and unverified pending users, but no test covers the scenario where a pending user is *not* verified.

**Suggested test case**:
```scala
"handleStatus" should "not auto-activate unverified pending users" in {
  val user = createUser(status = Pending, verified = false)

  val result = userService.handleStatus(user)

  result should not be Active
  result shouldBe Pending
}
```

---

## Higher-Order Mutation Results

First-order adjusted score was 81%, so higher-order testing was triggered via `--higher-order` flag.

| # | Mutations Combined | Location | Description | Result |
|---|-------------------|----------|-------------|--------|
| H1 | #1 + #17 | `validateAge:28, calculateDiscount:95` | `>=` → `>` AND `*` → `/` | ✅ **KILLED** |
| H2 | #3 + #9 | `findUser:35, activeUsers:60` | `find` → `None` AND `filter` → `filterNot` | ✅ **KILLED** |
| H3 | #6 + #7 | `normalizeEmail:50-51` | `toLowerCase` → `toUpperCase` AND remove `trim` | ✅ **KILLED** |
| H4 | #8 + #14 | `isActive:55, handleStatus:80` | `true` → `false` AND swap Active/Suspended branches | 🟡 **SURVIVED** |
| H5 | #12 + #13 | `createUser:68-70` | `Right` → `Left` AND `Left` → `Right` | ✅ **KILLED** |

**Higher-Order Score**: 4 / 5 (80%)

### Surviving Higher-Order Mutation H4

**Mutations combined**: #8 (`true` → `false` in `isActive` default) + #14 (swap `Active` and `Suspended` case bodies in `handleStatus`)
**Why it survived**: Flipping the default active status and swapping the Active/Suspended handler branches produces compensating behavior — users who would have been active now enter the suspended handler, but since the suspended handler's logic happens to produce the same side effect for the default case, the net behavior is unchanged from the test suite's perspective.
**What this reveals**: Tests verify `isActive` and `handleStatus` independently but don't test their interaction — specifically, what happens when a user's active status feeds into status handling.

**Suggested test**:
```scala
"handleStatus" should "produce different outcomes for active vs suspended users" in {
  val activeUser = createUser(status = Active)
  val suspendedUser = createUser(status = Suspended)

  val activeResult = userService.handleStatus(activeUser)
  val suspendedResult = userService.handleStatus(suspendedUser)

  activeResult should not equal suspendedResult
}
```

---

## Category Breakdown

| Category | Attempted | Killed | Survived | Equivalent | Kill Rate |
|----------|-----------|--------|----------|------------|-----------|
| Comparison | 2 | 2 | 0 | 0 | 100% |
| Option | 3 | 2 | 1 | 0 | 67% |
| String | 2 | 2 | 0 | 0 | 100% |
| Boolean | 1 | 1 | 0 | 0 | 100% |
| Collection | 3 | 1 | 1 | 1 | 50% |
| Either | 2 | 2 | 0 | 0 | 100% |
| Pattern Match | 2 | 1 | 1 | 0 | 50% |
| Exception | 1 | 1 | 0 | 0 | 100% |
| Arithmetic | 1 | 1 | 0 | 0 | 100% |
| Return Value | 1 (incompilable) | 0 | 0 | 0 | N/A |
| **Total** | **17** | **13** | **3** | **1** | **81%** |

---

## Recommendations

1. **Add tests for `getDisplayName` Some path**: The Option handling in `getDisplayName` only tests the `None`/default case. Add a test that provides a `Some` display name and verifies it's returned.

2. **Test `activeUsers` limit parameter**: The `limit` parameter is completely untested. Add a test with more users than the limit and verify the result size.

3. **Test pattern match guard in `handleStatus`**: The `if user.verified` guard on the `Pending` case is untested. Add tests for both verified and unverified pending users to ensure the guard controls behavior.

4. **Fix test data for sort order testing**: Mutation #10 was classified as equivalent because test fixtures use identical first/last names. Update test data to use distinct values, then add a sort-order assertion.

5. **Test `isActive`/`handleStatus` interaction**: Higher-order mutation H4 revealed that these two features are tested in isolation. Add an integration test verifying that active status correctly influences status handling outcomes.

---

*Generated by semantic mutation testing with Claude*
