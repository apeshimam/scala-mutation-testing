# Scala Language Configuration

This file defines Scala-specific settings for the mutation testing framework. SKILL.md references these sections by name.

---

## File Extensions

- `.scala`

---

## Project Detection

Determine if this is a multi-module sbt project by checking for the target file's location relative to any subproject directories. Run:

```bash
sbt projects 2>&1
```

If there are multiple projects, identify which subproject contains the source file based on its path (e.g., a file in `modules/core/src/...` likely belongs to the `core` project). Store the subproject name — you will prefix sbt commands with it (e.g., `core/testOnly ...` instead of `testOnly ...`).

If there is only one project (or the root project), no prefix is needed.

---

## Build Setup

Start the sbt server and verify compilation. The server keeps zinc's incremental compilation state in memory, so subsequent mutations only recompile the single changed file rather than cold-starting a new JVM each time.

First, check if `sbt --client` (thin client) is available:

```bash
sbt --client "compile" 2>&1
```

Use a **180-second timeout** (first invocation starts the server). If this succeeds, **use `sbt --client` for all subsequent sbt commands** in this session. The thin client connects to the running server, avoiding JVM startup overhead and enabling incremental compilation.

If `--client` fails (e.g., older sbt version), fall back to plain `sbt` commands:

```bash
sbt compile 2>&1
```

Store which mode is being used (client vs plain) for the rest of the session. For multi-module projects, prefix commands with the subproject name.

If compilation fails in either mode, stop and inform the user that the code must compile before mutation testing can begin.

---

## Test Discovery

Extract the class/object name (e.g., `UserService`) from the source file. Search for test files using common Scala conventions.

Use Glob to search for:
- `**/<ClassName>Spec.scala`
- `**/<ClassName>Test.scala`
- `**/<ClassName>Suite.scala`

Also use Grep to search in `src/test/scala/` (or the appropriate test directory for multi-module projects) for:
- The class name (e.g., `UserService`)
- Import statements referencing the class

---

## Test Identification

For each discovered test file, read it and extract the **fully qualified class name** (package + class name). You will use these with `sbt testOnly`.

Example: A test file with `package com.example.services` and `class UserServiceSpec` yields `com.example.services.UserServiceSpec`.

---

## Test Execution

Run tests using sbt's `testOnly` command with the fully qualified test class names:

```bash
sbt --client "testOnly fully.qualified.TestClass1 fully.qualified.TestClass2" 2>&1
```

For multi-module projects, prefix with the subproject name:

```bash
sbt --client "<subproject>/testOnly fully.qualified.TestClass1 fully.qualified.TestClass2" 2>&1
```

If client mode is unavailable, use plain `sbt` instead of `sbt --client`.

Use a **120-second timeout**.

**Note**: `sbt testOnly` compiles incrementally before running tests, so a separate compile step is unnecessary. With `--client`, only the mutated file is recompiled (zinc tracks file-level dependencies). Look for `Compilation failed` or `error:` in the output to identify incompilable mutations.

---

## Build Teardown

If client mode was used, shut down the sbt server to free resources:

```bash
sbt --client shutdown 2>&1
```

This is optional but good practice. The server will also shut down automatically after an idle timeout.

---

## Mocking Frameworks

### Framework Detection

Look for imports and usage patterns of:

- **mockito-scala**: `org.mockito.scalatest`, `org.mockito.MockitoSugar`, `when(...).thenReturn(...)`, `doReturn(...).when(...)`
- **ScalaMock**: `org.scalamock`, `.mock[T]`, `.stub[T]`, `.expects(...)`, `.returning(...)`
- **Mockito Java**: `org.mockito.Mockito`, `Mockito.mock(...)`, `Mockito.when(...)`, `Mockito.verify(...)`

If no mocking framework is detected, record "None" and skip mocking analysis.

### Mock Setup Patterns

Count occurrences of: `when(`, `doReturn(`, `.expects(`, `.returning(`, `mock[`, `Mockito.mock(`

### Assertion Patterns

Count occurrences of: `shouldBe`, `shouldEqual`, `should be`, `should equal`, `must be`, `mustBe`, `assert(`, `assertEquals`, `shouldNot`, `mustNot`

### Wildcard Matchers

Patterns indicating overly broad matching: `any`, `any[T]`, `*[T]`, `*`, `eqTo(any)`

---

## Code Analysis Focus

When analyzing the source file, pay special attention to these Scala-specific features (in addition to universal constructs like conditionals, loops, error handling, and function calls):

- **Option** — `Some`, `None`, `getOrElse`, `map`, `flatMap`, `fold`, `orElse`, `filter`, `isDefined`, `isEmpty`
- **Either** — `Right`, `Left`, `map`, `flatMap`, `fold`, `getOrElse`, `isRight`, `isLeft`, `swap`
- **Try** — `Success`, `Failure`, `recover`, `recoverWith`, `getOrElse`, `toOption`, `map`
- **Pattern matching** — `match`/`case` expressions, guards, exhaustiveness
- **For comprehensions** — generators, guards, yield transformations
- **Implicit parameters / given instances** — `implicit`, `given`, `using`, type class instances
- **Case classes** — `.copy()` calls with field overrides
- **Sealed traits / enums** — exhaustive matching, ADT patterns
- **Type classes** — implicit/given instances that affect behavior
- **Collection operations** — `filter`, `map`, `flatMap`, `foldLeft`, `sortBy`, `exists`, `forall`, `find`, etc.
- **Higher-order functions** — function arguments to collection methods, callbacks

---

## Test Style Conventions

Scala projects typically use **ScalaTest** with one of these styles:
- **FlatSpec** — `"subject" should "behavior" in { ... }`
- **WordSpec** — `"subject" should { "behavior" in { ... } }`
- **FunSuite** / **AnyFunSuite** — `test("description") { ... }`
- **FreeSpec** — `"subject" - { "behavior" in { ... } }`

When generating suggested test cases or auto-fix tests:
- Match the existing test file's style
- Use ScalaTest matchers (`shouldBe`, `should have size`, etc.)
- Follow the project's import conventions
- Use the same fixture/helper patterns present in existing tests
