# Mutation Testing Report

**File**: `src/services/user-service.ts`
**Language**: TypeScript
**Date**: 2026-02-15
**Mutations tested**: 17
**Test identifiers**: `src/services/__tests__/user-service.test.ts`, `src/services/__tests__/user-validation.test.ts`
**Mode**: standard
**Flags**: none

---

## Score Summary

| Metric | Value |
|--------|-------|
| **Mutation Score (adjusted)** | **11 / 14 (79%)** |
| Mutation Score (raw) | 11 / 16 (69%) |
| Killed | 11 |
| Survived | 3 |
| Equivalent | 2 |
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
| 2 | Optional Chaining | `UserService.getProfile:35` | `user?.profile` → `user.profile` | 🟡 **SURVIVED** |
| 3 | Optional Chaining | `UserService.getProfile:35` | `user?.profile` → `undefined` | ✅ **KILLED** |
| 4 | Nullish Coalescing | `UserService.getDisplayName:42` | `name ?? "Anonymous"` → `"Anonymous"` | 🟡 **SURVIVED** |
| 5 | Nullish Coalescing | `UserService.getDisplayName:42` | `name ?? "Anonymous"` → `name \|\| "Anonymous"` | ⚪ **EQUIVALENT** |
| 6 | String | `UserService.normalizeEmail:50` | `.toLowerCase()` → `.toUpperCase()` | ✅ **KILLED** |
| 7 | String | `UserService.normalizeEmail:51` | `.trim()` → remove | ✅ **KILLED** |
| 8 | Boolean | `UserService.isActive:55` | `return true` → `return false` | ✅ **KILLED** |
| 9 | Array | `UserService.activeUsers:60` | `.filter(u => u.isActive)` → `.filter(() => true)` | ✅ **KILLED** |
| 10 | Array | `UserService.activeUsers:61` | `.sort((a, b) => a.lastName.localeCompare(b.lastName))` → remove `.sort(...)` | ✅ **KILLED** |
| 11 | Array | `UserService.activeUsers:62` | `.slice(0, limit)` → remove `.slice(...)` | ⚪ **EQUIVALENT** |
| 12 | Type Guard | `UserService.processInput:70` | `typeof input === "string"` → `typeof input !== "string"` | ✅ **KILLED** |
| 13 | Promise/Async | `UserService.fetchUser:78` | `await this.repository.findById(id)` → `this.repository.findById(id)` | 🟡 **SURVIVED** |
| 14 | Promise/Async | `UserService.fetchUser:80` | `.catch(() => null)` → remove `.catch(...)` | ✅ **KILLED** |
| 15 | Object | `UserService.updateUser:88` | `{ ...user, ...updates }` → `{ ...user }` | ✅ **KILLED** |
| 16 | Arithmetic | `UserService.calculateDiscount:95` | `price * discount` → `price / discount` | ✅ **KILLED** |
| 17 | Template Literal | `UserService.formatGreeting:100` | `` `Hello, ${name}!` `` → `""` | ⚠️ **INCOMPILABLE** |

Result key: ✅ KILLED · 🟡 SURVIVED · ⚪ EQUIVALENT · ⚠️ INCOMPILABLE · ⏱️ TIMEOUT

---

## Equivalent Mutation Analysis

### Mutation #5: `name ?? "Anonymous"` → `name || "Anonymous"`

**What was changed**: Replaced nullish coalescing (`??`) with logical OR (`||`).
**Why it's equivalent**: Analysis of the codebase shows `name` is typed as `string | null | undefined`. It is never set to `""` (empty string) or `0` or other falsy-but-non-nullish values. Since `??` and `||` only differ for falsy non-nullish values, and no such values are possible here, the mutation is equivalent in this context.

### Mutation #11: Remove `.slice(0, limit)` from `activeUsers`

**What was changed**: Removed the `.slice(0, limit)` call from the `activeUsers` method.
**Why it's equivalent**: All test cases pass `limit` as `undefined` or a value larger than the test data size. `Array.slice(0, undefined)` returns the full array, making this mutation produce identical results in all tested scenarios. This is a test data gap rather than true code equivalence.

---

## Surviving Mutation Analysis

The following mutations survived, indicating gaps in test coverage:

### Mutation #2: `user?.profile` → `user.profile` (remove optional chaining)

**Category**: Optional Chaining
**Location**: `UserService.getProfile:35`
**What was changed**: Removed the optional chaining operator, changing `user?.profile` to `user.profile`. This will throw a TypeError if `user` is null or undefined.
**Why it matters**: No test covers the case where `user` is null or undefined. If `getProfile` is called with a null user, the code would crash at runtime. The optional chaining is a safety measure that is untested.

**Suggested test case**:
```typescript
it("should return undefined when user is null", () => {
  const result = userService.getProfile(null);

  expect(result).toBeUndefined();
});

it("should return undefined when user is undefined", () => {
  const result = userService.getProfile(undefined);

  expect(result).toBeUndefined();
});
```

---

### Mutation #4: `name ?? "Anonymous"` → always return `"Anonymous"`

**Category**: Nullish Coalescing
**Location**: `UserService.getDisplayName:42`
**What was changed**: Replaced `name ?? "Anonymous"` with just `"Anonymous"`, ignoring the user's actual display name.
**Why it matters**: No test verifies that `getDisplayName` returns the user's actual name when one exists. Tests only cover the fallback case where name is null/undefined.

**Suggested test case**:
```typescript
it("should return the user's display name when it exists", () => {
  const user = createUser({ displayName: "Alice Johnson" });

  const result = userService.getDisplayName(user);

  expect(result).toBe("Alice Johnson");
});
```

---

### Mutation #13: `await this.repository.findById(id)` → remove `await`

**Category**: Promise/Async
**Location**: `UserService.fetchUser:78`
**What was changed**: Removed `await` from `await this.repository.findById(id)`, causing the variable to hold a Promise instead of the resolved value.
**Why it matters**: No test verifies that the returned value is actually a resolved user object rather than a Promise. The test may be asserting on a Promise object that happens to be truthy, rather than the actual user data.

**Suggested test case**:
```typescript
it("should return the resolved user object, not a Promise", async () => {
  mockRepository.findById.mockResolvedValue({
    id: 1,
    name: "Alice",
    email: "alice@example.com",
  });

  const result = await userService.fetchUser(1);

  expect(result).toEqual({
    id: 1,
    name: "Alice",
    email: "alice@example.com",
  });
  expect(result).not.toBeInstanceOf(Promise);
});
```

---

## Category Breakdown

| Category | Attempted | Killed | Survived | Equivalent | Kill Rate |
|----------|-----------|--------|----------|------------|-----------|
| Comparison | 1 | 1 | 0 | 0 | 100% |
| Optional Chaining | 2 | 1 | 1 | 0 | 50% |
| Nullish Coalescing | 2 | 0 | 1 | 1 | 0% |
| String | 2 | 2 | 0 | 0 | 100% |
| Boolean | 1 | 1 | 0 | 0 | 100% |
| Array | 3 | 2 | 0 | 1 | 100% |
| Type Guard | 1 | 1 | 0 | 0 | 100% |
| Promise/Async | 2 | 1 | 1 | 0 | 50% |
| Object | 1 | 1 | 0 | 0 | 100% |
| Arithmetic | 1 | 1 | 0 | 0 | 100% |
| Template Literal | 1 (incompilable) | 0 | 0 | 0 | N/A |
| **Total** | **16** | **11** | **3** | **2** | **79%** |

---

## Overmocking Analysis

| Test File | Framework | Mock Setups | Assertions | Ratio | Severity | Indicators |
|-----------|-----------|-------------|------------|-------|----------|------------|
| `user-service.test.ts` | Jest | 15 | 10 | 1.5:1 | **Moderate** | Excessive `jest.fn()`, Verify-only tests |
| `user-validation.test.ts` | Jest | 2 | 11 | 0.2:1 | **None** | — |

**Overall severity**: **Moderate**

### Indicator: Excessive `jest.fn()` usage

**What was detected**: 11 of 15 mock setups in `user-service.test.ts` create mock functions with `jest.fn()` that use `mockReturnValue` or `mockResolvedValue` with generic values. For example:
- Line 20: `const mockFindById = jest.fn().mockResolvedValue(mockUser)`
- Line 35: `const mockSave = jest.fn().mockResolvedValue(undefined)`

**Why it matters**: Mock functions with fixed return values don't verify that the correct arguments are passed to dependencies. A mutation that changes which user ID is looked up would not be caught.

**Suggestion**: Use `expect(mockFindById).toHaveBeenCalledWith(expectedId)` alongside return value assertions, or use `mockImplementation` to validate arguments.

### Indicator: Verify-only tests

**What was detected**: 2 test cases in `user-service.test.ts` use only `toHaveBeenCalled` / `toHaveBeenCalledWith` without outcome assertions:
- Line 80: `"should call repository.delete when deleting user"` — only checks `expect(mockRepo.delete).toHaveBeenCalledWith(userId)`
- Line 95: `"should send notification on create"` — only checks `expect(mockNotifier.send).toHaveBeenCalled()`

**Why it matters**: These tests confirm interactions occurred but not that they produced correct results. Mutations changing return values, error handling, or business logic would survive.

**Suggestion**: Add `expect(result).toEqual(...)` assertions alongside `toHaveBeenCalled` checks.

### Correlation with surviving mutations

> ⚠️ **Overmocking correlation**: Mutation #13 survived in `UserService.fetchUser:78` — the `await` removal went undetected because `user-service.test.ts` mocks `repository.findById` with `jest.fn().mockResolvedValue(mockUser)`. The test's assertion likely checks truthiness of the result (a Promise is truthy) rather than its actual shape. Consider asserting on specific fields of the returned object.

---

## Recommendations

1. **Test `getProfile` with null/undefined user**: The optional chaining in `getProfile` is untested. Add tests that pass null and undefined to verify the method handles missing users gracefully.

2. **Test `getDisplayName` with an actual name**: The nullish coalescing fallback `"Anonymous"` is tested but the happy path is not. Add a test verifying a user's real name is returned.

3. **Verify `fetchUser` returns resolved data, not a Promise**: The removed `await` was not detected. Add a test that asserts on specific properties of the returned user (e.g., `expect(result.name).toBe("Alice")`) rather than just truthiness.

4. **Fix test data for `activeUsers` limit**: Mutation #11 was classified as equivalent because tests never exercise the limit. Update test data to include more items than the limit and assert on result length.

5. **Replace generic `jest.fn()` with argument-validating mocks**: 11 of 15 mocks use fixed return values without argument checks. Add `toHaveBeenCalledWith` assertions or use `mockImplementation` to validate inputs.

6. **Add outcome assertions to verify-only tests**: Two tests only verify interactions. Add `expect(result)` assertions to check return values alongside `toHaveBeenCalled` checks.

---

*Generated by semantic mutation testing with Claude*
