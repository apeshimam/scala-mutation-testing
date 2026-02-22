# Python Language Configuration

This file defines Python-specific settings for the mutation testing framework. SKILL.md references these sections by name.

---

## File Extensions

- `.py`

---

## Project Detection

Check for project configuration files in the repository root and ancestor directories:

- `pyproject.toml` — modern Python projects (Poetry, Flit, Hatch, PDM, or plain setuptools)
- `setup.py` — legacy setuptools projects
- `setup.cfg` — declarative setuptools configuration
- `requirements.txt` — pip-based dependency management
- `Pipfile` — Pipenv projects
- `poetry.lock` — Poetry projects (confirms Poetry is in use)

**Detect package manager** (in priority order):
1. `poetry.lock` exists → use `poetry run` prefix
2. `Pipfile.lock` or `Pipfile` exists → use `pipenv run` prefix
3. `uv.lock` exists or `[tool.uv]` in `pyproject.toml` → use `uv run` prefix
4. Otherwise → use bare commands (assumes activated virtualenv or global install)

**Verify pytest is available**:
```bash
python -m pytest --version 2>&1
```
(Or `poetry run pytest --version`, etc., depending on detected package manager.)

If pytest is not found, check if `unittest` tests are used instead (look for `import unittest` in test files).

**Detect virtual environment**: Check for `.venv/`, `venv/`, or `$VIRTUAL_ENV` environment variable.

---

## Build Setup

Python does not require compilation. Optionally verify syntax:

```bash
python -c "import py_compile; py_compile.compile('<file>', doraise=True)" 2>&1
```

If the file has syntax errors, stop and inform the user that the code must be syntactically valid before mutation testing can begin.

---

## Test Discovery

Extract the module/class name (e.g., `UserService` from `user_service.py`) from the source file. Search for test files using common Python conventions.

Use Glob to search for:
- `**/test_<name>.py`
- `**/<name>_test.py`
- `**/test_<name>*.py`

Also search in `tests/`, `test/` directories.

Use Grep to search for:
- `import <module_name>` or `from <module_name> import`
- `from <package>.<module_name> import`
- The class name (e.g., `UserService`)

---

## Test Identification

pytest uses **file paths** as test identifiers (not fully qualified class names). Extract the test file paths relative to the project root.

Optionally, identify specific test classes or functions using `::` syntax for targeted runs:
- `tests/test_user_service.py::TestUserService` — run a specific test class
- `tests/test_user_service.py::TestUserService::test_create_user` — run a specific test function

For basic usage, file paths are sufficient.

---

## Test Execution

Run tests using pytest with the discovered test file paths:

```bash
python -m pytest <test_file1> <test_file2> -x -v 2>&1
```

With the appropriate package manager prefix if detected:
- `poetry run pytest <test_file1> <test_file2> -x -v 2>&1`
- `pipenv run pytest <test_file1> <test_file2> -x -v 2>&1`
- `uv run pytest <test_file1> <test_file2> -x -v 2>&1`

Use `-x` (stop on first failure) and `-v` (verbose) for clear output.

Use a **120-second timeout**.

**Interpreting results**: Look for `FAILED` or `ERROR` in output for killed mutations. `passed` indicates the mutation survived. Syntax errors or import errors indicate incompilable mutations.

---

## Build Teardown

None needed. Python has no persistent build server to shut down.

---

## Mocking Frameworks

### Framework Detection

Look for imports and usage patterns of:

- **unittest.mock**: `from unittest.mock import`, `Mock()`, `MagicMock()`, `patch(`, `mock_open`
- **pytest-mock**: `mocker.patch`, `mocker.MagicMock`, `mocker.Mock`, `mocker.spy`

If no mocking framework is detected, record "None" and skip mocking analysis.

### Mock Setup Patterns

Count occurrences of: `mock.return_value`, `mock.side_effect`, `patch(`, `MagicMock(`, `Mock(`, `mocker.patch`, `.return_value =`, `.side_effect =`

### Assertion Patterns

Count occurrences of: `assert `, `assertEqual`, `assertTrue`, `assertFalse`, `assertIn`, `assertRaises`, `assertIsNone`, `assertIsNotNone`, `pytest.raises`, `assert.*==`, `assert.*is`

### Wildcard Matchers

Patterns indicating overly broad matching: `ANY`, `mock.ANY`, `unittest.mock.ANY`, `mocker.ANY`

---

## Code Analysis Focus

When analyzing the source file, pay special attention to these Python-specific features (in addition to universal constructs like conditionals, loops, error handling, and function calls):

- **Decorators** — `@staticmethod`, `@classmethod`, `@property`, custom decorators
- **Context managers** — `with` statements, `__enter__`/`__exit__`, `contextlib`
- **Generators/iterators** — `yield`, `yield from`, generator expressions
- **List/dict/set comprehensions** — `[x for x in xs if cond]`, `{k: v for ...}`, `{x for ...}`
- **`*args`/`**kwargs`** — variadic arguments, unpacking
- **Dataclasses** — `@dataclass`, field defaults, `__post_init__`
- **Abstract classes** — `ABC`, `@abstractmethod`
- **Properties** — `@property`, `@x.setter`, `@x.deleter`
- **Type hints** — `Optional`, `Union`, `List`, `Dict`, type guards
- **Walrus operator** — `:=` assignment expressions (Python 3.8+)
- **F-strings** — `f"...{expr}..."`
- **Default mutable arguments** — `def f(x=[])` anti-pattern
- **Dunder methods** — `__str__`, `__repr__`, `__eq__`, `__hash__`, `__len__`, etc.
- **`__slots__`** — class attribute optimization
- **Pattern matching** — `match`/`case` statements (Python 3.10+)

---

## Test Style Conventions

Python projects typically use one of these test styles:

- **pytest style** — plain functions with `assert`, fixtures via `@pytest.fixture`, parametrize via `@pytest.mark.parametrize`
- **unittest style** — class-based tests extending `unittest.TestCase`, `setUp`/`tearDown` methods

When generating suggested test cases or auto-fix tests:
- Match the existing test file's style
- Use the same assertion patterns (plain `assert` for pytest, `self.assertEqual` for unittest)
- Follow the project's fixture and helper patterns
- Use the same import conventions
