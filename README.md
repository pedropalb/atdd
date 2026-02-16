# ATDD — Acceptance Test Driven Development for Claude Code

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet)](https://github.com/swingerman/atdd)
[![Version](https://img.shields.io/badge/version-0.1.0-green)](https://github.com/swingerman/atdd)

A [Claude Code](https://code.claude.com) plugin that enforces the **Acceptance Test Driven Development** (ATDD) methodology when building software with AI. Write human-readable Given/When/Then specs before code, generate project-specific test pipelines, and maintain the discipline of two-stream testing.

**Inspired by [Robert C. Martin's](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) acceptance test approach from [empire-2025](https://github.com/unclebob/empire-2025).** The ideas, methodology, and key insights in this plugin come directly from his work and public writings on Spec Driven Design (SDD) and ATDD.

## Why ATDD with AI?

When using AI to write code, two problems emerge:

1. **AI writes code without constraints** — without acceptance tests anchoring behavior, AI can "willy-nilly plop code around" (Uncle Bob's words) and write unit tests that pass but don't verify the right behavior.
2. **Implementation details leak into specs** — AI naturally tries to fill Given/When/Then statements with class names, API endpoints, and database tables instead of domain language.

This plugin solves both problems by enforcing the ATDD workflow and guarding against implementation leakage. The result: **two test streams** (acceptance + unit) that constrain AI development, producing better-structured, more reliable code.

> "The two different streams of tests cause Claude to think much more deeply about the structure of the code. It can't just willy-nilly plop code around and write a unit test for it. It is also constrained by the structure of the acceptance tests."
> — Robert C. Martin

## How It Works

```
1. Write Given/When/Then specs (natural language, domain-only)
                    ↓
2. Generate test pipeline (parser → IR → test generator)
   Pipeline has DEEP knowledge of your codebase internals
                    ↓
3. Run acceptance tests → they FAIL (red)
                    ↓
4. Implement with TDD (unit tests + code) until BOTH streams pass
                    ↓
5. Review specs for implementation leakage
                    ↓
6. Iterate — next feature, back to step 1
```

The generated pipeline is NOT like Cucumber. It's what Uncle Bob calls "a strange hybrid of Cucumber and the test fixtures" — the parser/generator has **deep knowledge of your system's internals** and produces complete, runnable tests. No manual fixture code needed.

## Installation

Add the marketplace and install the plugin:

```shell
/plugin marketplace add swingerman/atdd
/plugin install atdd@swingerman-atdd
```

Or test locally by cloning:

```bash
git clone https://github.com/swingerman/atdd.git
claude --plugin-dir ./atdd
```

## Getting Started

### 1. Start the ATDD workflow

```
/atdd:atdd Add user authentication with email and password
```

Claude will guide you through understanding the feature, then help you write acceptance specs.

### 2. Write your first spec

Create a file in `specs/` at your project root:

```
;===============================================================
; User can register with email and password.
;===============================================================
GIVEN no registered users.

WHEN a user registers with email "bob@example.com" and password "secret123".

THEN there is 1 registered user.
THEN the user "bob@example.com" can log in.
```

### 3. Generate the test pipeline

Claude analyzes your codebase and generates a project-specific parser, IR format, and test generator tailored to your language and test framework (pytest, Jest, JUnit, Go testing, RSpec, etc.).

### 4. Red → Green → Refactor

Run the acceptance tests (they should fail), then implement with TDD until both acceptance tests AND unit tests pass.

### 5. Check for leakage

```
/atdd:spec-check
```

The spec-guardian agent reviews your specs for implementation details that shouldn't be there.

## GWT Spec Format

Specs use an opinionated, human-readable Given/When/Then format:

```
;===============================================================
; Description of the behavior being specified.
;===============================================================
GIVEN [precondition in domain language].

WHEN [action the user/system takes].

THEN [observable outcome].
```

### The Golden Rule: External Observables Only

Specs must describe what the system does, not how it does it:

| Bad (implementation leakage) | Good (domain language) |
|------------------------------|----------------------|
| `GIVEN the UserService has an empty userRepository.` | `GIVEN there are no registered users.` |
| `WHEN a POST request is sent to /api/users with JSON body.` | `WHEN a new user registers with email "bob@example.com".` |
| `THEN the database contains 1 row in the users table.` | `THEN there is 1 registered user.` |

> "Specs will be co-authored by the humans and the AI, but with final approval, ferociously defended, by the humans."
> — Robert C. Martin

## Plugin Components

| Component | Name | Purpose |
|-----------|------|---------|
| Skill | `atdd` | Core 7-step ATDD workflow: specs → pipeline → red/green → iterate |
| Agent | `spec-guardian` | Catches implementation leakage in Given/When/Then statements |
| Agent | `pipeline-builder` | Generates bespoke parser → IR → test generator for your project |
| Command | `/atdd:atdd` | Start the ATDD workflow for a new feature |
| Command | `/atdd:spec-check` | Audit specs for implementation leakage |
| Hook | PreToolUse | Soft warning when writing code without acceptance specs |
| Hook | Stop | Reminder to verify both test streams pass |

## Key Principles

These principles from Uncle Bob's writings are encoded in the plugin:

- **"Just enough specs for this sprint"** — Don't write all specs upfront. Spec the current feature, implement it, iterate. Avoid Big Up-Front Design.
- **"Two test streams constrain development"** — Acceptance tests define WHAT (external behavior), unit tests define HOW (internal structure). Both must pass.
- **"Specs describe only external observables"** — No class names, API endpoints, database tables, or framework terms in specs. Domain language only.
- **"Co-authored by humans and AI, ferociously defended by humans"** — Claude proposes specs, you approve them. Always.
- **"Small steps"** — Whether your sprint is a day, an hour, or a microsecond, the same rules apply.

## What This Is NOT

- **Not Cucumber.** The generated pipeline has deep knowledge of your system's internals. No generic fixture layer that requires manual glue code.
- **Not a template library.** Claude analyzes your project fresh every time and generates a bespoke pipeline for your language, framework, and codebase.
- **Not Big Up-Front Design.** Write just enough specs for the current feature. Iterate.

## Works With Any Language

The pipeline-builder agent analyzes your project and generates the parser/generator in your project's language and test framework. Tested approaches include Python/pytest, TypeScript/Jest, Java/JUnit, Go/testing, Ruby/RSpec, Clojure/Speclj, and more.

## Attribution

This plugin is an implementation of Robert C. Martin's (Uncle Bob) Acceptance Test Driven Development and Spec Driven Design methodology for Claude Code. The approach, insights, and principles come from:

- [empire-2025](https://github.com/unclebob/empire-2025) — Uncle Bob's project where this approach was developed and refined
- His public writings and tweets on ATDD, SDD, and AI-assisted development

This plugin does not contain any code from empire-2025. It adapts the methodology for use as a Claude Code plugin.

## Contributing

Contributions are welcome! Please open an issue or PR on [GitHub](https://github.com/swingerman/atdd).

## License

[MIT](LICENSE)
