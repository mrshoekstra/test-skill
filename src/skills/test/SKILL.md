# /test — Create, Update, and Run Unit Tests

When this skill is invoked, follow the phases below in order. The goal is to leave the test file comprehensively covering the target source file, then run it green.

---

## Phase 0 — Build a project profile (read before writing anything)

Read the following files to understand the project's conventions. Read only what exists — skip missing files silently.

1. `CLAUDE.md`, `GEMINI.md`, `AGENTS.md`, or `CONTRIBUTING.md` — naming rules, error handling conventions, security requirements, explicit test guidance
2. `composer.json`, `package.json`, `pyproject.toml`, `go.mod`, or `Makefile` — test runner, scripts, test dependencies
3. Test runner config: `phpunit.xml`, `jest.config.*`, `vitest.config.*`, `pytest.ini`, `.mocharc.*`, `go.sum`, etc.
4. Two or three existing test files closest to the target in the directory tree — infer assertion style, mock library, factory helper patterns, naming conventions, and fixture approach from real examples

From this, determine:
- **Test runner command** for a single file and for the full suite
- **Source → test path mapping** rule for this project:

  | Language | Common pattern |
  |----------|---------------|
  | PHP      | `src/Foo/Bar.php` → `tests/Unit/Foo/BarTest.php` |
  | TypeScript / JavaScript | `src/foo.ts` → `src/foo.test.ts` or `__tests__/foo.test.ts` |
  | Python   | `src/foo.py` → `tests/test_foo.py` |
  | Go       | `pkg/foo/bar.go` → `pkg/foo/bar_test.go` (same package) |
  | Rust     | inline `#[cfg(test)]` module in the same file |

- **Language-specific boilerplate** (imports, base class, describe blocks, etc.)
- **Project-specific security and validation rules** extracted from `CLAUDE.md`, `GEMINI.md`, `AGENTS.md`, or `CONTRIBUTING.md`

---

## Phase 1 — Identify the target

Resolve the target in this priority order:
1. Explicit argument passed to the skill (e.g. `/test src/Http/Router.php` or `/test RouterTest`)
2. Active IDE selection — use the file containing the selected code
3. Ask the user

Determine both:
- The **source file** to be tested
- The **mirror test file** path according to the project's mapping rule

---

## Phase 2 — Read and analyse the source

Read the full source file. Catalogue every testable unit:
- Public functions / methods / exported symbols / route handlers
- Constructor / initialisation logic and its preconditions
- All guard clauses, validations, and error branches (every `throw`, `raise`, `panic`, non-zero return)
- State transitions and observable side effects
- Any use of external I/O, environment variables, file paths, or secrets

---

## Phase 3 — Update or create the test file

### Mode A — Test file already exists (incremental update)

This is the default on every run after the first. **Never rewrite the whole file.**

1. Read the existing test file in full — every existing test is valid; keep it
2. Mentally diff the source against what is already tested:
   - New public method / function → add tests for it
   - Changed signature or contract → update only the affected tests
   - Removed source method → identify orphaned tests, ask the user before deleting them
   - Missing coverage in any of the six categories below → add the missing tests
3. Append new tests at the end of the file (or inside the appropriate describe/class block). Edit existing tests only where they are demonstrably wrong or stale.

### Mode B — Test file does not exist (full creation)

Create the file at the derived path using the detected boilerplate. Write tests
covering all six categories below.

---

## Phase 4 — Write tests across all six categories

Cover **all six categories** for every source file. If a category is provably
irrelevant for this specific file (e.g. a pure math helper has no security surface),
state that explicitly in the Phase 6 report — never skip silently.

### A. Industry-standard / happy-path tests
- The golden path: correct inputs → correct output / expected side effect
- Every public method / exported function / route handler has at least one passing test
- Verify return values, response codes, state changes, and side effects

### B. Input / output tests
- Boundary values: empty string, zero, `null`/`nil`/`None`, maximum integer, very long strings
- Type-coercion traps specific to the language: `0` vs `false`, `""` vs `null`, `[]` vs `null`, `undefined` vs `null`, etc.
- Every documented valid input shape → correct output shape
- At least one test per distinct equivalence partition (e.g. positive / zero / negative)

### C. Flow / control-path tests
- Every reachable branch: `if`/`else`, `switch`, early returns, ternaries
- Guard clauses: correct behaviour when a precondition is not met
- State machine transitions (if the unit manages internal state)
- Async / callback / event paths where applicable

### D. Error / exception tests
- Every `throw` / `raise` / `panic` reachable from a public entry point
- Assert the correct exception **type** and **message**
- Where the project uses named error-code constants (e.g. `ERR_*`, JSON-RPC codes), assert the code too
- Where the project requires exception chaining, assert that `getPrevious()` / `__cause__` / `Unwrap()` is set
- Non-throwing error paths: verify error return values, `Result` types, HTTP status codes for failure cases

### E. Security tests
Start from any explicit requirements in `CLAUDE.md`, `GEMINI.md`, `AGENTS.md`, or `CONTRIBUTING.md`, then apply these universal checks where the source file has the relevant surface:

| Threat | What to test |
|--------|-------------|
| Injection (SQL / command / template / header) | Untrusted input is rejected or safely escaped before use |
| Path traversal | Paths containing `../` or URL-encoded variants are rejected; `realpath()` / `os.path.realpath()` result is validated against the allowed root |
| SSRF | Caller-supplied URLs that resolve to loopback, link-local, or RFC-1918 ranges are blocked |
| Token / secret hygiene | `\r`, `\n`, `\r\n` stripped from tokens; secrets do not appear in logs or error responses |
| Header injection | Newlines in `Authorization`, `Cookie`, or `Location` values are rejected |
| Input schema validation | Malformed JSON, wrong field types, missing required fields → correct error code; no silent coercion |
| Output escaping | Any value rendered into HTML is escaped (XSS prevention) |
| Auth enforcement | Protected endpoints / methods reject requests with a missing or invalid token |

### F. Edge-case / robustness tests
- Unicode and multibyte strings where string length or encoding matters
- Whitespace-only inputs where empty is rejected
- Duplicate registrations or keys (last-wins? first-wins? error?)
- Repeated / idempotent calls: calling twice should not corrupt state
- Large inputs if the function advertises or implies a size limit
- Locale / timezone sensitivity: prefer tests that pass explicit locale / timezone arguments rather than relying on system defaults

---

## Phase 5 — Run, fix, re-run

**Step 1** — Run the target test file only (fast feedback loop):

| Runner | Single-file command |
|--------|-------------------|
| PHPUnit | `php vendor/bin/phpunit <file>` |
| Jest | `npx jest <file>` |
| Vitest | `npx vitest run <file>` |
| pytest | `python -m pytest <file>` |
| Go | `go test <./package/path/...>` |
| Cargo | `cargo test <module>` |
| (other) | Detect from the project profile built in Phase 0 |

**Step 2** — If the target passes, run the full suite to catch regressions.

**Step 3** — On any failure:
- Read the full failure output
- If the test is wrong (stale expectation, wrong mock setup): fix the test
- If the source is wrong: flag it and ask the user before changing source code
- Re-run until all tests pass

---

## Phase 6 — Report

End with a concise summary covering:
- **Mode used:** created from scratch or incrementally updated
- **Changes made:** number of tests added / modified
- **Category coverage:** which of A–F were covered; which (if any) were skipped and why
- **Run outcome:** total tests, assertions / passes, any skipped or risky tests
- **Noteworthy findings:** security gaps, missing guards, or anything the user should act on
