---
name: jargoud-php-coding-quality
description: >-
  Applies cross-cutting coding principles, quality bar, implementation discipline,
  and validation for PHP and Laravel work. Use on every implementation, bug fix,
  refactor, or code review when scope, design, minimal diffs, SOLID/KISS/YAGNI/DRY,
  security, typing, tests, and formatter/linter checks matter—even if the user
  does not explicitly mention coding quality.
license: MIT
metadata:
  author: Jargoud
---

# Coding quality

Apply on every implementation, bug fix, and refactor.

## When to activate

- Activate for any implementation, bug fix, refactor, or code review.
- Activate when the user asks for minimal diffs, quality bar, or disciplined validation.
- Activate for PHP and Laravel application code unless the user explicitly limits scope.

## Rule priority

| Layer | Purpose |
|-------|---------|
| **Stack / tooling rules** | Language, framework, versions, formatters, MCP (e.g. `AGENTS.md`, generated skills) |
| **Language / framework rules** | e.g. `php.mdc` (`**/*.php`), `laravel.mdc` (`app/**/*.php`) |
| **This skill** | Cross-cutting principles and quality bar |

When rules conflict: follow **stack rules** for ecosystem specifics; follow **this skill** for scope, design, and verification discipline. Do not duplicate stack documentation here.

---

## 1. Scope (highest priority)

- Deliver **only** what was requested — no drive-by refactors, features, or cleanup unless explicitly asked.
- **Minimal diff**: simplest correct solution; smaller changes are easier to review and revert.
- **MECE**: each change has one clear purpose; avoid partial fixes that leave duplicate or ambiguous paths.

## 2. Design principles

- **KISS**: favor readability; do not over-engineer.
- **YAGNI**: no speculative features or flexibility.
- **DRY**: extract shared logic; do not copy similar code across files.
- **SOLID**: small focused units; extend via abstractions when they reduce complexity; depend on interfaces when justified — prefer **framework idioms** over custom abstraction layers.

## 3. Code quality

- **Complexity**: keep **cognitive** load low — short methods, max 2–3 nesting levels, well-named helpers. On paths where data volume can grow, mind **algorithmic** cost: prefer **O(n)** or **O(n log n)**; justify nested loops or repeated scans; use indexes, pagination/chunking, and appropriate data structures for lookups — no formal Big-O write-ups on trivial code.
- **Bounded execution**: loops and recursion need a provable exit or an explicit cap (max iterations, max depth); set timeouts on external I/O; paginate or chunk large datasets — do not load unbounded data into memory.
- **Comments**: explain **why**, not **what**; avoid self-evident or redundant comments (including JSDoc/PHPDoc that only restate the code).
- **Dead code**: remove unused code, unreachable paths, and commented-out blocks.
- **Typing**: type parameters and returns; on **new** code follow language-specific rules (e.g. `php.mdc`) — do not bulk-modernize legacy files without explicit approval.
- **Security**: no hardcoded secrets; use framework validation, ORM/query builders, and output escaping — avoid raw SQL/HTML in application code.
- **Errors**: specific exceptions; no empty `catch` blocks. If you catch to recover or fallback, **log** with context (`Log::error($message, ['exception' => $e, ...])`) — never swallow failures silently.
- **Style**: follow project style guide and run the configured formatter/linter on touched files.

## 4. Files and structure

- **One primary responsibility** per file; split files beyond **~300 LOC** when they harm readability or testability.
- **Documentation**: add or update docs only when the user or project rules require it.

## 5. Environments and data

- **Multi-environment**: behavior configurable via env vars and project config — no hardcoded hosts, paths, or credentials.
- **Runtime data**: mocks, stubs, and seed fakes belong in **tests** only — not in production code paths.
- **Data changes**: migrations and scripts must be safe and reversible by design; no destructive production changes without explicit approval.

## 6. Before writing new code

- Search the codebase and project docs for existing patterns, helpers, and dependencies.
- Extend current abstractions before adding new ones; **remove** replaced implementations.
- Prefer project tooling (MCP, CLI, test harness) over one-off scripts unless the user requests them.

## 7. Validation (zero tolerance)

Skip only when the user explicitly asks.

1. **Build / analyze** — changed code compiles; fix new issues in touched files.
2. **Tests** — add or update **Pest** tests for changed behavior; run the **minimum** relevant suite (`php artisan test --compact` with file or filter — see `laravel.mdc`).
3. **Format / lint** — project formatter/linter on modified files.
4. **Dependency order** — data layer → application tests → API/integration → UI only if affected.
5. **Finish** — impact analysis on affected areas; compare `git diff` to the request; no failing tests or warnings left in changed scope.
