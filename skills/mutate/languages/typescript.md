# TypeScript Language Configuration

This file defines TypeScript-specific settings for the mutation testing framework. SKILL.md references these sections by name.

---

## File Extensions

- `.ts`
- `.tsx`

---

## Project Detection

Check for project configuration files:

- `package.json` — Node.js project manifest (required)
- `tsconfig.json` — TypeScript configuration

**Read `package.json`** to detect:
- **Test framework** from `devDependencies`: `jest`, `vitest`, `mocha`, `@jest/core`, `ts-jest`, `@vitest/runner`
- **Test scripts** from `scripts.test`: inspect the command to confirm which runner is used

**Detect package manager** from lockfile:
1. `package-lock.json` → npm (`npx`)
2. `yarn.lock` → yarn (`yarn`)
3. `pnpm-lock.yaml` → pnpm (`pnpm exec`)
4. `bun.lockb` or `bun.lock` → bun (`bunx`)
5. None found → default to npm (`npx`)

---

## Build Setup

Verify TypeScript compiles successfully:

```bash
npx tsc --noEmit 2>&1
```

Use a **120-second timeout**. If compilation fails, stop and inform the user that the code must compile before mutation testing can begin.

**Note**: For projects using vitest, jest with ts-jest, or jest with esbuild/swc transforms, TypeScript compilation is handled by the test runner. If `tsc --noEmit` fails but tests pass (Step 1.3), you may proceed — the test runner has its own TypeScript handling. In this case, skip the `tsc` check and rely on test runner compilation errors to detect incompilable mutations.

---

## Test Discovery

Extract the module/class name (e.g., `UserService` from `user-service.ts`) from the source file. Search for test files using common TypeScript/JavaScript conventions.

Use Glob to search for:
- `**/<name>.test.ts`
- `**/<name>.spec.ts`
- `**/<name>.test.tsx`
- `**/<name>.spec.tsx`

Also search in `__tests__/`, `test/`, `tests/` directories.

Use Grep to search for:
- `import.*from.*<module>` or `require.*<module>`
- The class/function name (e.g., `UserService`)

---

## Test Identification

Test runners use **file paths** as test identifiers. Extract the test file paths relative to the project root.

For Jest, you can use `--testPathPattern` for more targeted runs:
- `npx jest --testPathPattern="user-service"` — match test files by pattern

For Vitest:
- `npx vitest run user-service.test.ts` — run specific test file

---

## Test Execution

Detect the test runner and use the appropriate command:

**Jest**:
```bash
npx jest <test_file1> <test_file2> --no-coverage 2>&1
```

**Vitest**:
```bash
npx vitest run <test_file1> <test_file2> 2>&1
```

**Mocha**:
```bash
npx mocha <test_file1> <test_file2> 2>&1
```

Adapt the command prefix based on the detected package manager (e.g., `yarn jest`, `pnpm exec jest`, `bunx vitest`).

Use `--no-coverage` with Jest to speed up mutation runs.

Use a **120-second timeout**.

**Interpreting results**: Look for `FAIL` or `✕` in Jest output, `FAIL` in Vitest output, or `failing` in Mocha output for killed mutations. `PASS` / `✓` / `passing` indicates the mutation survived. TypeScript compilation errors or `SyntaxError` indicate incompilable mutations.

---

## Build Teardown

None needed. TypeScript test runners do not have persistent build servers to shut down.

---

## Mocking Frameworks

### Framework Detection

Look for imports and usage patterns of:

- **Jest**: `jest.mock(`, `jest.spyOn(`, `jest.fn()`, `jest.requireMock`, `jest.createMockFromModule`
- **Vitest**: `vi.mock(`, `vi.spyOn(`, `vi.fn()`, `vi.hoisted`
- **Sinon**: `sinon.stub(`, `sinon.mock(`, `sinon.spy(`, `sinon.fake(`

If no mocking framework is detected, record "None" and skip mocking analysis.

### Mock Setup Patterns

Count occurrences of: `mockReturnValue`, `mockResolvedValue`, `mockRejectedValue`, `mockImplementation`, `jest.fn(`, `vi.fn(`, `.returns(`, `.resolves(`, `.rejects(`, `sinon.stub(`, `jest.mock(`, `vi.mock(`

### Assertion Patterns

Count occurrences of: `expect(`, `toBe`, `toEqual`, `toStrictEqual`, `toThrow`, `toHaveBeenCalled`, `toHaveBeenCalledWith`, `toHaveBeenCalledTimes`, `assert.equal`, `assert.strictEqual`, `assert.deepEqual`, `should.equal`, `assert.ok`

### Wildcard Matchers

Patterns indicating overly broad matching: `expect.any(`, `expect.anything()`, `sinon.match.any`, `expect.objectContaining`

---

## Code Analysis Focus

When analyzing the source file, pay special attention to these TypeScript-specific features (in addition to universal constructs like conditionals, loops, error handling, and function calls):

- **Optional chaining** — `a?.b`, `a?.b()`, `a?.[index]`
- **Nullish coalescing** — `a ?? b`, `a ??= b`
- **Type guards** — `is` return type, `typeof`, `instanceof`, `in` operator
- **Discriminated unions** — `switch(x.kind)`, `if (x.type === "...")`
- **Generics** — type parameters affecting behavior
- **Async/await** — `async` functions, `await` expressions, Promise chains
- **Promises** — `.then()`, `.catch()`, `.finally()`, `Promise.all`, `Promise.race`, `Promise.allSettled`
- **Enums** — `enum` values and exhaustive switches
- **Interfaces** — structural typing, optional properties
- **Type assertions** — `as T`, `as unknown as T`
- **Template literal types** — tagged templates, template expressions
- **Mapped/conditional types** — `Partial`, `Required`, `Pick`, `Omit`, `Record`
- **Logical assignment** — `??=`, `||=`, `&&=`
- **Optional parameters** — `param?: Type`, default values

---

## Test Style Conventions

TypeScript projects typically use one of these test styles:

- **Jest/Vitest style** — `describe`/`it`/`expect` blocks, `beforeEach`/`afterEach` hooks
- **Mocha + Chai style** — `describe`/`it` with `expect`/`should`/`assert` from Chai

When generating suggested test cases or auto-fix tests:
- Match the existing test file's style
- Use the same assertion library (`expect` for Jest/Vitest, Chai for Mocha)
- Follow the project's mock and helper patterns
- Use the same import conventions (ESM vs CommonJS)
- Respect TypeScript typing in test code
