# 🧪 `/test` Skill <sub><sup>for AI & LLMs</sup></sub> <picture><img alt="Markdown" src="https://img.shields.io/badge/Markdown-083fa1"></picture>

<!-- DESCRIPTION START -->
An automated AI agent skill operating strictly within the quality assurance domain, designed to meticulously create, incrementally update, and execute unit tests for your codebase. The primary objective is to guarantee that the target source file is comprehensively covered and that the resulting test suite inevitably runs green.
<!-- DESCRIPTION END -->

## 📑 Index

- ✨ [Features](#-features)
- 🚀 [Usage](#-usage)
- ❤️ [Support](#️-support)
- ⚖️ [License](#️-license)

## ✨ Features

* **Quality Assurance Domain Mastery**: Explicitly engineered to uphold QA standards across multiple programming environments by enforcing strict test coverage and regressions checks.
* **Intelligent Profiling**: Automatically infers testing frameworks, naming conventions, and mock strategies by silently scanning existing configuration files and tests.
* **Multi-Language Support**: Seamlessly maps source files to testing paths across PHP, TypeScript, JavaScript, Python, Go, and Rust ecosystems.
* **Incremental Updates**: Safely appends new coverage to existing test files without overwriting or destroying previously established assertions.
* **Six-Dimensional Coverage**: Enforces strict code coverage across happy paths, input/output boundaries, control flows, error exceptions, security vulnerabilities, and architectural edge cases.
* **Automated Run-Fix Loops**: Rapidly executes single-file tests and recursively patches failures or incorrect mock setups until the entire suite passes.

## 🚀 Usage

Invoke the skill by passing a specific file or relying on active IDE selection. The architectural execution flows strictly through the following six phases:

### Phase 0: Build a Project Profile

Before writing any code, the skill reads environmental constraints to establish a baseline.
* It analyzes rules from `CLAUDE.md`, `GEMINI.md`, `AGENTS.md`, or `CONTRIBUTING.md`.
* It inspects manifest files like `composer.json`, `package.json`, `pyproject.toml`, or `go.mod` to determine runner dependencies.
* It reviews runner configurations (`phpunit.xml`, `jest.config.*`, etc.) and adjacent test files to deduce assertion styles and boilerplate formatting.

### Phase 1: Identify the Target

The system precisely locates the file requiring tests using a strict priority queue.
* It first checks for explicit arguments passed directly to the skill (e.g., `/test src/Http/Router.php`).
* It falls back to scanning the active IDE selection to capture the relevant codebase.
* It dynamically derives the mirror test file path based on language-specific mapping rules.

### Phase 2: Read and Analyse the Source

The skill catalogues every functional unit within the target file.
* It maps public methods, constructor initializations, and state transitions.
* It isolates guard clauses, validations, and all reachable error branches (such as `throw`, `raise`, or `panic`).
* It flags any integration with external I/O, environment variables, or secrets.

### Phase 3: Update or Create the Test File

* **Mode A (Incremental Update)**: Mentally diffs the source against existing tests, adding coverage for newly implemented methods or changed signatures.
* **Mode B (Full Creation)**: Scaffolds a complete testing suite at the derived path using the exact boilerplate detected during Phase 0.

> [!IMPORTANT]  
> When operating in Mode A (Incremental Update), the skill will never rewrite the whole file. It retains all existing valid tests and only appends missing coverage.

### Phase 4: Write Tests Across Six Categories

The generation engine systematically covers six distinct architectural categories.
- **A. Industry-Standard / Happy-Path**: Verifies correct inputs yield expected outputs and observable side effects.
- **B. Input / Output Tests**: Stresses boundary values, type-coercion traps, and shape constraints.
- **C. Flow / Control-Path Tests**: Validates reachable branches, guard clauses, and internal state machine transitions.
- **D. Error / Exception Tests**: Asserts correct exception types, internal messages, and chained error codes.
- **E. Security Tests**: Audits injection surfaces, path traversals, SSRF risks, and token hygiene.
- **F. Edge-Case / Robustness Tests**: Checks unicode boundaries, idempotent calls, locale sensitivities, and large input limits.

### Phase 5 & 6: Run, Fix, Re-run, and Report

The skill executes a fast feedback loop utilizing the detected test runner.
- It executes the single target file first.
- On success, it triggers the full test suite to guarantee zero regressions.
- On failure, it analyzes the terminal output to patch stale expectations or flag faulty source code.
- Finally, it delivers a concise summary detailing test increments, category coverage metrics, and actionable security findings.

## ❤️ Support

If this project saved you some time, please consider giving it a ⭐ **Star** on GitHub; it helps others discover the repository!

If you would like to support my work further, please check out the **Sponsor this project** section on this repository page. Even a small contribution makes a big difference!

## ⚖️ License

Distributed under the [MIT License](LICENSE.md).
