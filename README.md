# ATDD — Acceptance Test Driven Development for Gemini CLI

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Gemini CLI Extension](https://img.shields.io/badge/Gemini%20CLI-Extension-blueviolet)](https://github.com/swingerman/atdd)

A [Gemini CLI](https://geminicli.com) extension that enforces the **Acceptance Test Driven Development** (ATDD) methodology when building software with AI. Write human-readable Given/When/Then specs before code, generate project-specific test pipelines, and maintain the discipline of two-stream testing.

**Inspired by [Robert C. Martin's](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) acceptance test approach from [empire-2025](https://github.com/unclebob/empire-2025).** The ideas, methodology, and key insights in this extension come directly from his work and public writings on Spec Driven Design (SDD) and ATDD.

## Why ATDD with AI?

When using AI to write code, two problems emerge:

1. **AI writes code without constraints** — without acceptance tests anchoring behavior, AI can "willy-nilly plop code around" (Uncle Bob's words) and write unit tests that pass but don't verify the right behavior.
2. **Implementation details leak into specs** — AI naturally tries to fill Given/When/Then statements with class names, API endpoints, and database tables instead of domain language.

This extension solves both problems by enforcing the ATDD workflow and guarding against implementation leakage. The result: **two test streams** (acceptance + unit) that constrain AI development, producing better-structured, more reliable code.

> "The two different streams of tests cause the AI to think much more deeply about the structure of the code. It can't just willy-nilly plop code around and write a unit test for it. It is also constrained by the structure of the acceptance tests."
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

Install the extension using the Gemini CLI:

```bash
gemini extensions install https://github.com/swingerman/atdd.git
```

Or install from a local directory:

```bash
gemini extensions link .
```

## Getting Started

### 1. Start the ATDD workflow

```
/atdd Add user authentication with email and password
```

Gemini will guide you through understanding the feature, then help you write acceptance specs.

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

Gemini analyzes your codebase and generates a project-specific parser, IR format, and test generator tailored to your language and test framework (pytest, Jest, JUnit, Go testing, RSpec, etc.).

### 4. Red → Green → Refactor

Run the acceptance tests (they should fail), then implement with TDD until both acceptance tests AND unit tests pass.

### 5. Check for leakage

```
/spec-check
```

The `spec-guardian` skill reviews your specs for implementation details that shouldn't be there.

## Team-Based ATDD Workflow

For larger features, you can orchestrate an agent team that follows the ATDD workflow. The team lead coordinates specialist agents — each with a clear role and strict instructions.

> **Tip:** The extension includes an `atdd-team` skill that automates team setup and orchestration. Just say "build [feature] with a team" and the skill handles role creation, phase sequencing, and prompt generation.

### Works With Existing Teams

Already running an agent team? The `atdd-team` skill detects active teams and gives you three options:

- **Extend** — Add ATDD roles (spec-writer, implementer, reviewer) to your existing team. Roles that already exist by name are skipped.
- **Replace** — Shut down the current team and create a fresh ATDD team.
- **New team** — Create a separate ATDD team alongside your existing one.

This means you can spin up a team for any purpose, then later say "add ATDD to my team" or "extend my team with ATDD" and the spec-writer, implementer, and reviewer roles join your existing teammates without disrupting ongoing work.

### Team Roles

| Role | Agent Type | Responsibility |
|------|-----------|----------------|
| **Team Lead** | You (or a `general-purpose` agent) | Orchestrates workflow, reviews specs, approves all work, enforces discipline |
| **Spec Writer** | `agent` | Writes Given/When/Then specs from feature requirements using domain language only |
| **Implementer** | `agent` | Builds code using TDD — unit tests first, then implementation — until both test streams pass |
| **Spec Guardian** | `agent` (read-heavy) | Reviews specs for implementation leakage before and after implementation |

### Setting Up the Team

**Prompt to create the team:**

```
Create a team called "atdd-feature" with the following teammates:

1. "spec-writer" (agent) — Writes Given/When/Then acceptance
   test specs. Has the atdd extension installed. Must follow the atdd
   skill strictly.

2. "implementer" (agent) — Implements features using TDD.
   Writes unit tests first, then code, until both acceptance tests and
   unit tests pass.

3. "reviewer" (agent) — Reviews specs for implementation
   leakage and reviews code for quality. Uses spec-check.
```

### Step-by-Step: Team Lead Instructions

#### Phase 1 — Spec Writing

Send to **spec-writer**:

```
We're implementing [feature description].

Follow the ATDD workflow from the atdd extension. Your job:

1. Read the existing codebase to understand the domain language
   (how does the app refer to users, orders, sessions, etc.?)
2. Write Given/When/Then specs in specs/[feature-name].txt
3. Use ONLY external observables — no class names, no API endpoints,
   no database tables, no framework terms
4. Each spec file starts with a semicolon comment block describing
   the behavior
5. Use periods at the end of each statement
6. Send me the specs for review before proceeding

CRITICAL: If you're unsure whether a term is domain language or
implementation language, ask me. Do NOT guess.

Example of what I expect:

;===============================================================
; User can register with email and password.
;===============================================================
GIVEN no registered users.

WHEN a user registers with email "bob@example.com" and password "secret123".

THEN there is 1 registered user.
THEN the user "bob@example.com" can log in.
```

#### Phase 2 — Spec Review

Send to **reviewer**:

```
Review the specs in specs/[feature-name].txt for implementation leakage.

Flag ANY of these:
- Class names, function names, method names
- Database tables, columns, queries
- API endpoints, HTTP methods, status codes
- Framework terms (controller, service, repository, middleware)
- Internal state or data structures
- File paths or module names

For each violation, show the bad line and propose a rewrite using
domain language only.

Also check:
- Is each spec testing ONE behavior?
- Are the GIVEN/WHEN/THEN statements clear to a non-developer?
- Could these specs work for a different implementation language?

Send me your review.
```

#### Phase 3 — Pipeline Generation

As team lead, do this yourself or instruct:

```
Generate the test pipeline for this project. Analyze:
- Language and test framework in use
- Project structure and existing test patterns
- The specs in specs/[feature-name].txt

Create the 3-stage pipeline:
1. Parser — reads specs/*.txt, outputs IR to acceptance-pipeline/ir/
2. Generator — reads IR, produces runnable tests in generated-acceptance-tests/
3. Runner script — run-acceptance-tests.sh

The generator must have DEEP knowledge of the codebase internals.
This is NOT Cucumber. Generated tests should call directly into the
system — no manual fixture glue.

Run the acceptance tests after generation. They MUST fail (red).
If they pass, either the behavior already exists or the generator
isn't testing the right thing.
```

#### Phase 4 — Implementation

Send to **implementer**:

```
The acceptance specs are in specs/[feature-name].txt.
The test pipeline is set up — run ./run-acceptance-tests.sh to execute.

Implement the feature using TDD:

1. Run acceptance tests first — confirm they FAIL
2. Pick the simplest failing acceptance test
3. Write a unit test for the smallest piece needed
4. Write minimal code to make the unit test pass
5. Refactor
6. Repeat 3-5 until that acceptance test passes
7. Move to the next failing acceptance test
8. Continue until ALL acceptance tests AND unit tests pass

RULES:
- Never modify spec files (specs/*.txt) — they are the contract
- Never modify generated test files — only regenerate via the pipeline
- If a spec seems wrong or ambiguous, STOP and ask me
- Run both test streams before reporting done:
  ./run-acceptance-tests.sh  (acceptance tests)
  [project test command]      (unit tests)
- Send me the results when both streams are green
```

#### Phase 5 — Post-Implementation Review

Send to **reviewer**:

```
Implementation is complete. Do two reviews:

1. SPEC REVIEW: Run spec-check on specs/[feature-name].txt
   Check if any implementation details leaked into specs during
   development. Propose cleanups if found.

2. CODE REVIEW: Review the implementation for:
   - Test quality (are unit tests testing the right things?)
   - Code structure (does it match what the specs describe?)
   - Missing edge cases (any specs that should be added?)

Send me both reviews.
```

### Tips for Team Leads

- **Never let implementers modify specs.** Specs are YOUR contract. If an implementer says a spec is wrong, review it yourself before authorizing changes.
- **Run spec-check twice** — once before implementation (catch leakage from spec writing) and once after (catch leakage from implementation pressure).
- **Keep specs portable.** Ask yourself: "Could these specs generate the same feature in a different language?" If not, there's leakage.
- **Scope tightly.** Each team cycle should cover one feature. Don't spec the whole system — spec what you're building now.
- **Verify both streams.** Before accepting the implementer's work, run both the acceptance tests and unit tests yourself.

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

## Extension Components

| Component | Name | Purpose |
|-----------|------|---------|
| Skill | `atdd` | Core 7-step ATDD workflow: specs → pipeline → red/green → iterate |
| Skill | `atdd-team` | Orchestrates an agent team for ATDD — handles team setup, role assignment, and phase sequencing |
| Skill | `spec-guardian` | Catches implementation leakage in Given/When/Then statements |
| Skill | `pipeline-builder` | Generates bespoke parser → IR → test generator for your project |
| Command | `/atdd` | Start the ATDD workflow for a new feature |
| Command | `/spec-check` | Audit specs for implementation leakage |
| Hook | BeforeTool | Soft warning when writing code without acceptance specs |
| Hook | SessionEnd | Reminder to verify both test streams pass |

## Key Principles

These principles from Uncle Bob's writings are encoded in the extension:

- **"Just enough specs for this sprint"** — Don't write all specs upfront. Spec the current feature, implement it, iterate. Avoid Big Up-Front Design.
- **"Two test streams constrain development"** — Acceptance tests define WHAT (external behavior), unit tests define HOW (internal structure). Both must pass.
- **"Specs describe only external observables"** — No class names, API endpoints, database tables, or framework terms in specs. Domain language only.
- **"Co-authored by humans and AI, ferociously defended by humans"** — Gemini proposes specs, you approve them. Always.
- **"Small steps"** — Whether your sprint is a day, an hour, or a microsecond, the same rules apply.

## What This Is NOT

- **Not Cucumber.** The generated pipeline has deep knowledge of your system's internals. No generic fixture layer that requires manual glue code.
- **Not a template library.** Gemini analyzes your project fresh every time and generates a bespoke pipeline for your language, framework, and codebase.
- **Not Big Up-Front Design.** Write just enough specs for the current feature. Iterate.

## Works With Any Language

The pipeline-builder skill analyzes your project and generates the parser/generator in your project's language and test framework. Tested approaches include Python/pytest, TypeScript/Jest, Java/JUnit, Go/testing, Ruby/RSpec, Clojure/Speclj, and more.

## Attribution

This extension is an implementation of Robert C. Martin's (Uncle Bob) Acceptance Test Driven Development and Spec Driven Design methodology for Gemini CLI. The approach, insights, and principles come from:

- [empire-2025](https://github.com/unclebob/empire-2025) — Uncle Bob's project where this approach was developed and refined
- His public writings and tweets on ATDD, SDD, and AI-assisted development

This extension does not contain any code from empire-2025. It adapts the methodology for use as a Gemini CLI extension.

Grateful acknowledgment to [swingerman](https://github.com/swingerman/atdd), the original author of the Claude Code plugin this extension is based on.

## Contributing

Contributions are welcome! Please open an issue or PR on [GitHub](https://github.com/swingerman/atdd).

## License

[MIT](LICENSE)
