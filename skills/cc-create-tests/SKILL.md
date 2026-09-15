---
name: cc-create-tests
description: >-
  Create, structure, and write high-resilience tests under package-level tests/unit|integration|e2e.
  Enforces 1-to-1 file symmetry, anti-colocation, error assertion hygiene, zero-any type safety,
  and in-memory fixture patterns. Use when writing tests, adding a test script, choosing where
  tests belong, or when the user asks for tests / TDD / coverage.
---

# cc-create-tests

Authoritative guide for creating, structuring, and maintaining resilient test suites across packages and monorepos.

Apply this skill whenever creating, relocating, or authoring tests. It enforces clean architecture seams, strict isolation, and robust assertions.

---

## 1. Directory Layout & Anti-Colocation

Every package or app gets **one** top-level `tests/` directory at the package root.

### Structure

```
<package-or-app>/
  src/
    domain/
      engine.ts
    services/
      ingest.ts
  tests/
    unit/           # Pure, isolated, deterministic logic — no real network, DB, or browser (<10ms)
    integration/    # Crosses module seams, interacts with DB/KV, uses in-memory doubles or real I/O
    e2e/            # Full system/user flow (browser, live HTTP servers, end-to-end runs)
    contract/       # (Optional) API schema compatibility and third-party payload contracts
```

### Hard Layout Rules

1. **Strict Anti-Colocation**: Never place `*.test.ts`, `*.spec.ts`, or `__tests__/` inside `src/`, `lib/`, or alongside implementation code. The `src/` directory is strictly for production code.
2. **Type by Directory, Not Filename**: The folder hierarchy designates the test level (`tests/unit/`, `tests/integration/`). Do not bury test types in filename noise like `engine.unit.test.ts`.
3. **Dedicated Package Root**: Each package in a monorepo owns its own `tests/` folder (e.g., `packages/deye-client/tests/`, `apps/web/tests/`). Do not pool unit tests into a detached root directory unless testing workspace-level tooling.

---

## 2. File Mirroring & Naming Symmetry

Unit tests must mirror source files with mathematical predictability.

1. **One Unit Test File Per Code File**: Each source code file maps to exactly one unit test file in the corresponding `tests/unit/` directory.
   - `src/analytics/engine.ts` → `tests/unit/engine.test.ts` (or `tests/unit/analytics/engine.test.ts`)
   - `src/domain/plantSession.ts` → `tests/unit/plantSession.test.ts`
2. **Identical Suffix Naming**: Suffix must consistently be `.test.ts` / `.test.tsx` (or `.spec.ts` if established by repo conventions).
3. **No Fragmentation**: Never scatter unit tests across multiple files testing the same unit (e.g. avoid `engine-math.test.ts` and `engine-cases.test.ts`; keep them organized via `describe` blocks inside the single canonical test file).

---

## 3. Test Seams & Fixture Architecture (Anti-Monkey-Patching)

Fragile tests depend on brittle monkey-patching (`vi.mock()`, `jest.spyOn()` deep module mocking). Resilient tests depend on clean seams.

1. **In-Memory Fixtures Over Deep Mocking**:
   - For I/O interfaces (API clients, databases, key-value stores), design a domain port/interface and provide a deterministic In-Memory Fixture (e.g., `InMemoryFixtureAdapter`, `createMockEnv()`).
   - Tests execute in milliseconds without flaky timers, race conditions, or unmocked edge cases.
2. **Avoid Global Module Hijacking**:
   - Do not mock standard Node/browser built-ins globally unless testing an explicit error pathway.
   - If a function cannot be tested without deep `vi.mock()`, the code has a design smell: extract pure logic into pure domain functions or inject the dependency.

---

## 4. Assertion Hygiene & Error Verification

Brittle assertions fail during trivial refactors, translations, or error formatting changes.

1. **Never Match Error Message Strings**:
   - ❌ **Forbidden**: `expect(() => fn()).toThrow("Station not found or invalid id")`
   - ❌ **Forbidden**: `expect(() => fn()).toThrow(/invalid id/)`
   - ✅ **Required**: Assert the specific error class or structured property:
     ```ts
     // Class assertion
     expect(() => fn()).toThrow(StationNotFoundError);

     // Property & structure assertion
     try {
       fn();
       expect.unreachable();
     } catch (err: unknown) {
       expect(err).toBeInstanceOf(AppError);
       if (err instanceof AppError) {
         expect(err.code).toBe("STATION_NOT_FOUND");
         expect(err.statusCode).toBe(404);
       }
     }
     ```
2. **Explicit Null & Undefined Checks**:
   - When extracting elements from arrays or nullable objects, assert presence before accessing:
     ```ts
     const firstItem = response.items?.[0];
     expect(firstItem).toBeDefined();
     if (!firstItem) throw new Error("Expected firstItem to be present");
     expect(firstItem.id).toBe("station-1");
     ```

---

## 5. Strict Type Safety (Zero `any`)

Tests must adhere to the exact same type rigor as production code.

1. **Prohibit `any`**:
   - Do not use `as any` or `any` parameters in mocks, test fixtures, or assertions.
   - Use `unknown`, `Record<string, unknown>`, or proper type narrowing (`instanceof`, `typeof`, schema validation).
2. **Partial Test Data**:
   - When constructing complex payloads for tests, use TypeScript utility types (`Partial<T>`, `DeepPartial<T>`) or dedicated test-data factories / builders, rather than casting unverified shapes with `as unknown as FullType` wherever possible.

---

## 6. Monorepo Script Wiring & Execution

1. **Package Script Contract**: Every package must expose a `test` script in its `package.json`:
   ```json
   "scripts": {
     "test": "vitest run"
   }
   ```
2. **Granular Scripts (When needed)**:
   - `"test:unit": "vitest run tests/unit"`
   - `"test:integration": "vitest run tests/integration"`
3. **Workspace Inclusion**: Root test runners (e.g. `vitest.config.ts`) must configure includes across all packages without hardcoded paths:
   ```ts
   include: [
     "packages/*/tests/**/*.test.ts",
     "apps/*/tests/**/*.test.ts",
     "tests/**/*.test.ts",
   ]
   ```
4. **Performance Gate**: Pure unit test suites should execute in < 500ms total. If unit tests exceed this, check for unmocked async timers, disk writes, or network calls.

---

## 7. Execution Workflow

When tasked with writing tests:
1. **Verify Authorization**: Confirm the user requested tests or invoked `/cc-create-tests`.
2. **Inspect Source Seams**: Check if the target module relies on external I/O. If yes, check for an existing in-memory fixture before resorting to mocks.
3. **Pick the Layer**:
   - Pure function / domain engine → `tests/unit/<module>.test.ts`
   - Multi-module orchestration / DB / KV / HTTP routes → `tests/integration/<feature>.test.ts`
   - Full browser / runtime flow → `tests/e2e/<flow>.test.ts`
4. **Write Tests with Strict Types & Clean Assertions**:
   - No `any`.
   - No string-matching in `.toThrow()`.
   - 1:1 filename symmetry for unit tests.
5. **Execute & Verify**: Run `pnpm --filter <pkg> test` and ensure all tests pass cleanly.
