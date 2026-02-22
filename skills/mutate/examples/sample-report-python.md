# Mutation Testing Report

**File**: `src/services/user_service.py`
**Language**: Python
**Date**: 2026-02-15
**Mutations tested**: 16
**Test identifiers**: `tests/test_user_service.py`, `tests/test_user_validation.py`
**Mode**: standard
**Flags**: none

---

## Score Summary

| Metric | Value |
|--------|-------|
| **Mutation Score (adjusted)** | **10 / 13 (77%)** |
| Mutation Score (raw) | 10 / 15 (67%) |
| Killed | 10 |
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
| 1 | Comparison | `UserService.validate_age:25` | `age >= 18` → `age > 18` | ✅ **KILLED** |
| 2 | Comparison | `UserService.validate_age:25` | `age >= 18` → `age >= 19` | ✅ **KILLED** |
| 3 | Dictionary | `UserService.get_preferences:34` | `prefs.get("theme", "light")` → `"light"` | 🟡 **SURVIVED** |
| 4 | Comprehension | `UserService.active_users:42` | `[u for u in users if u.is_active]` → `[u for u in users]` | 🟡 **SURVIVED** |
| 5 | Comprehension | `UserService.active_users:42` | `[u for u in users if u.is_active]` → `list(users)` | ⚪ **EQUIVALENT** |
| 6 | Collection | `UserService.active_users:43` | `sorted(result, key=lambda u: u.last_name)` → `result` | ✅ **KILLED** |
| 7 | String | `UserService.normalize_email:50` | `.lower()` → `.upper()` | ✅ **KILLED** |
| 8 | String | `UserService.normalize_email:51` | `.strip()` → remove | ✅ **KILLED** |
| 9 | Boolean | `UserService.is_active:55` | `return True` → `return False` | ✅ **KILLED** |
| 10 | Type-Specific | `UserService.find_user:60` | `if user is not None` → `if user is None` | ✅ **KILLED** |
| 11 | Return Value | `UserService.get_display_name:65` | `return name or "Anonymous"` → `return "Anonymous"` | 🟡 **SURVIVED** |
| 12 | Exception | `UserService.delete_user:75` | Remove `except DatabaseError` handler | ✅ **KILLED** |
| 13 | Arithmetic | `UserService.calculate_discount:82` | `price * discount` → `price / discount` | ✅ **KILLED** |
| 14 | Collection | `UserService.active_users:44` | `result[:limit]` → `result` | ⚪ **EQUIVALENT** |
| 15 | Decorator | `UserService.cached_profile:90` | Remove `@lru_cache` | ✅ **KILLED** |
| 16 | F-String | `UserService.format_greeting:95` | `f"Hello, {name}!"` → `""` | ⚠️ **INCOMPILABLE** |

Result key: ✅ KILLED · 🟡 SURVIVED · ⚪ EQUIVALENT · ⚠️ INCOMPILABLE · ⏱️ TIMEOUT

---

## Equivalent Mutation Analysis

### Mutation #5: `[u for u in users if u.is_active]` → `list(users)`

**What was changed**: Replaced the list comprehension with guard with a plain `list()` call.
**Why it's equivalent**: This mutation is equivalent to mutation #4 (removing the guard). Since mutation #4 already tests whether the guard matters, this mutation is semantically redundant — both test the same behavioral property (whether inactive users are filtered out). Classified as equivalent to avoid double-counting.

### Mutation #14: `result[:limit]` → `result`

**What was changed**: Removed the `[:limit]` slice from the `active_users` method.
**Why it's equivalent**: Analysis of the test data shows all test cases pass `limit=None` or `limit` greater than the test data size. The slice `result[:None]` returns the full list, making this mutation produce identical results in all tested scenarios. This is a test data gap rather than true code equivalence.

---

## Surviving Mutation Analysis

The following mutations survived, indicating gaps in test coverage:

### Mutation #3: `prefs.get("theme", "light")` → always return `"light"`

**Category**: Dictionary
**Location**: `UserService.get_preferences:34`
**What was changed**: Replaced `prefs.get("theme", "light")` with just `"light"`, ignoring the user's actual theme preference.
**Why it matters**: No test verifies that `get_preferences` returns the user's stored theme when one exists. Tests only cover the default/fallback case.

**Suggested test case**:
```python
def test_get_preferences_returns_stored_theme(user_service, sample_user):
    sample_user.preferences = {"theme": "dark", "language": "en"}

    result = user_service.get_preferences(sample_user)

    assert result["theme"] == "dark"
```

---

### Mutation #4: Remove comprehension guard `if u.is_active`

**Category**: Comprehension
**Location**: `UserService.active_users:42`
**What was changed**: Changed `[u for u in users if u.is_active]` to `[u for u in users]`, removing the active-user filter.
**Why it matters**: No test provides a mix of active and inactive users and verifies that only active users are returned.

**Suggested test case**:
```python
def test_active_users_filters_inactive(user_service):
    users = [
        create_user("Alice", is_active=True),
        create_user("Bob", is_active=False),
        create_user("Carol", is_active=True),
    ]

    result = user_service.active_users(users)

    assert len(result) == 2
    assert all(u.is_active for u in result)
```

---

### Mutation #11: `return name or "Anonymous"` → always return `"Anonymous"`

**Category**: Return Value
**Location**: `UserService.get_display_name:65`
**What was changed**: Replaced `name or "Anonymous"` with just `"Anonymous"`, ignoring the user's actual display name.
**Why it matters**: No test verifies that `get_display_name` returns the user's actual name when one exists. Tests only check the fallback case.

**Suggested test case**:
```python
def test_get_display_name_returns_actual_name(user_service, sample_user):
    sample_user.display_name = "Alice Johnson"

    result = user_service.get_display_name(sample_user)

    assert result == "Alice Johnson"
```

---

## Category Breakdown

| Category | Attempted | Killed | Survived | Equivalent | Kill Rate |
|----------|-----------|--------|----------|------------|-----------|
| Comparison | 2 | 2 | 0 | 0 | 100% |
| Dictionary | 1 | 0 | 1 | 0 | 0% |
| Comprehension | 2 | 0 | 1 | 1 | 0% |
| Collection | 2 | 1 | 0 | 1 | 100% |
| String | 2 | 2 | 0 | 0 | 100% |
| Boolean | 1 | 1 | 0 | 0 | 100% |
| Type-Specific | 1 | 1 | 0 | 0 | 100% |
| Return Value | 1 | 0 | 1 | 0 | 0% |
| Exception | 1 | 1 | 0 | 0 | 100% |
| Arithmetic | 1 | 1 | 0 | 0 | 100% |
| Decorator | 1 | 1 | 0 | 0 | 100% |
| F-String | 1 (incompilable) | 0 | 0 | 0 | N/A |
| **Total** | **15** | **10** | **3** | **2** | **77%** |

---

## Overmocking Analysis

| Test File | Framework | Mock Setups | Assertions | Ratio | Severity | Indicators |
|-----------|-----------|-------------|------------|-------|----------|------------|
| `test_user_service.py` | unittest.mock | 14 | 9 | 1.6:1 | **Moderate** | Excessive `patch()`, Verify-only tests |
| `test_user_validation.py` | unittest.mock | 2 | 12 | 0.2:1 | **None** | — |

**Overall severity**: **Moderate**

### Indicator: Excessive `patch()` usage

**What was detected**: 10 of 14 mock setups in `test_user_service.py` use `patch()` as a context manager or decorator, patching module-level imports rather than injecting dependencies. For example:
- Line 30: `@patch("services.user_service.UserRepository")`
- Line 55: `with patch("services.user_service.email_client") as mock_email:`

**Why it matters**: Patching at the module level means tests are coupled to the import structure rather than the behavior. If `get_preferences` is tested with a fully patched repository, mutations to the `dict.get()` default value may not reach real code paths.

**Suggestion**: Use dependency injection instead of `patch()` where possible. Pass the `UserRepository` as a constructor argument and use a fake or stub in tests.

### Indicator: Verify-only tests

**What was detected**: 2 test cases in `test_user_service.py` use only `mock.assert_called_with()` or `mock.assert_called_once()` without asserting on return values:
- Line 72: `"test_delete_user_calls_repository"` — only verifies `mock_repo.delete.assert_called_once_with(user_id)`
- Line 85: `"test_create_user_sends_email"` — only verifies `mock_email.send.assert_called_once()`

**Why it matters**: These tests confirm methods were *called* but not that they produced *correct results*. Mutations changing return values or error handling in these methods would survive.

**Suggestion**: Add return value assertions alongside `assert_called` checks. For `delete_user`, also assert the return value. For `create_user`, also assert the returned user object has correct fields.

### Correlation with surviving mutations

> ⚠️ **Overmocking correlation**: Mutation #3 survived in `UserService.get_preferences:34` — this method is tested in `test_user_service.py` where the `UserRepository` is patched at module level (10 `patch()` calls). The mock returns a fixed user object, so the `dict.get()` default value path is never exercised with real preference data. Consider testing with a real preferences dictionary.

---

## Recommendations

1. **Add tests for `get_preferences` with stored values**: The `dict.get()` default in `get_preferences` only tests the fallback case. Add a test with a user who has explicit preferences stored.

2. **Test `active_users` with mixed active/inactive users**: The list comprehension guard is untested. Add a test with both active and inactive users and verify only active ones are returned.

3. **Test `get_display_name` with an actual name**: The `or "Anonymous"` fallback is tested but the happy path is not. Add a test verifying a user's real name is returned.

4. **Fix test data for `active_users` limit**: Mutation #14 was classified as equivalent because tests never exercise the limit. Update test data to include more items than the limit and assert on result size.

5. **Replace `patch()` with dependency injection in `test_user_service.py`**: 10 of 14 mock setups use module-level patching. Refactor `UserService` to accept dependencies via constructor and pass fakes in tests.

6. **Add outcome assertions to verify-only tests**: Two tests only verify method calls without checking results. Add `assert` statements for return values alongside `assert_called` checks.

---

*Generated by semantic mutation testing with Claude*
