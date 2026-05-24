---
name: jargoud-php
description: >-
  Applies PHP 8+ strict typing and modern idioms: declare(strict_types=1),
  constructor property promotion, readonly, backed enums, and typed PHPDoc discipline.
  Use when creating or editing any PHP file (**/*.php), including tests, or when
  the user mentions strict types, type hints, readonly, enums, or PHP style.
license: MIT
metadata:
  author: Jargoud
---

# PHP standards

Apply with the `jargoud-php-coding-quality` skill, `jargoud-laravel` skill (under `app/`), `AGENTS.md`, and other activated PHP/Laravel skills.

## When to activate

- Activate when working on any `**/*.php` file (application code, tests, config PHP files).
- Activate for strict typing, constructor promotion, `readonly`, backed enums, or PHPDoc discipline.
- Activate when the user asks to modernize PHP style on files already in scope.

## Strict typing

- Every **new** PHP file: `declare(strict_types=1);` immediately after `<?php`.
- Do **not** bulk-add strict types to existing files without explicit approval.
- Type every parameter, return type, and property; use `void` and short nullable syntax (`?string`) where appropriate.

## Modern idioms

- **Constructor property promotion** when all constructor-injected properties can be promoted.
- **`readonly`** classes or properties for immutable units (Actions, DTOs, value objects) — not required on Eloquent models, Livewire, or other types the framework expects to mutate.
- **Backed enums** for stable finite sets (statuses, types, roles) instead of magic strings, int constants, or loose associative maps — skip enums for one-off or unstable values (YAGNI).

## PHPDoc

- Do not add docblocks that only repeat information already expressed by type hints (follow Spatie/project conventions).

## Tests

- Under `tests/`, use **Pest** (`test()`, `it()`, `expect()`) — see the `jargoud-laravel` skill for commands and Filament/Livewire testing.

## Scope

- Apply these rules to **new** code and files you are already changing; do not drive-by modernize unrelated legacy code.
