---
name: jargoud-laravel
description: >-
  Applies Laravel application architecture for app code: Actions, Spatie Data DTOs,
  Integrations, Observers, HTTP-agnostic domain layers, and Pest testing. Use when
  creating or editing files under app/ (controllers, actions, data, integrations,
  observers, jobs, Filament) or when the user mentions Laravel structure, actions,
  DTOs, FormRequest validation, or third-party API clients.
license: MIT
metadata:
  author: Jargoud
---

# Laravel architecture

Apply with the `jargoud-php-coding-quality` skill, `php.mdc`, `AGENTS.md`, and other activated Laravel skills.

## When to activate

- Activate when working on `app/**/*.php` (actions, data, integrations, observers, HTTP layer, jobs).
- Activate for Laravel use cases, Spatie Data DTOs, external API integrations, model observers, or Pest tests.
- Activate when the user asks about folder placement, action pattern, or HTTP validation vs DTOs.

## Actions (`app/Actions/`)

Use the **action pattern** for application use cases (create, save, sync orchestration, etc.).

- Namespace: `App\Actions\{Domain}\{Verb}{Entity}Action` (e.g. `App\Actions\Article\SaveArticleAction`).
- **One public entry point**: `execute(...)` — no other public methods for the use case.
- Use `protected`/`private` methods for internal steps; inject other actions via the constructor.
- Prefer `readonly` classes when the action has no mutable state.
- Call actions from controllers, jobs, Filament pages, commands, or other actions — do not embed the same orchestration inline.

```php
readonly class SaveArticleAction
{
    public function __construct(
        protected SaveTargetAttributesAction $saveAttributesAction,
    ) {}

    public function execute(Article $article, array $attributes): void
    {
        // orchestration…
    }
}
```

## Data transfer objects

Use **Spatie Laravel Data** (`extends Data`) instead of untyped arrays for structured input/output.

| Scope | Location | Example |
|-------|----------|---------|
| Application / domain | `app/Data/{Context}/` | `App\Data\Auth\Sso\SsoUserData` |
| Action-specific payloads | `app/Data/Actions/{Action}/` | nested under the action when only it consumes them |
| External API payloads | `app/Integrations/{Vendor}/Data/` | `InboundArticleData`, nested subfolders per API shape |

- Map external keys with `#[MapInputName]`, collections with `#[DataCollectionOf]`, custom casts under `Data/Casts/`.
- Pass DTOs into `execute()` — avoid `array<string, mixed>` at action boundaries when the shape is known.

### HTTP validation vs DTOs

**Validate at the HTTP boundary first**, then map to a DTO — do not skip request validation.

| Entry point | Validate with | Then |
|-------------|---------------|------|
| API / HTTP controllers | `FormRequest` in `app/Http/Requests/` (extend `ApiRequest` for API) | `SomeData::from($request->validated(...))` |
| Filament / Livewire | form schema rules, `->rules()`, or dedicated request classes | map to DTO or typed args before calling an action |
| Jobs / CLI / internal callers | caller already has trusted types | construct or `::from()` the DTO directly |

- **FormRequest** owns HTTP-specific rules (required fields, formats, nested keys).
- **Spatie Data** owns structure, mapping, and casts — not a substitute for skipping `FormRequest` on inbound HTTP.
- Do not read `request()` inside a DTO factory; pass validated arrays or models into `::from()` / `::fromModel()` from the controller or job.

```php
public function store(ArticleRequest $request): Response
{
    $data = InboundArticleData::from($request->validated(ArticleRequest::ARTICLE_ATL));

    dispatch(new SyncArticleJob($data));

    return response(status: Response::HTTP_NO_CONTENT);
}
```

## HTTP-agnostic domain code

Actions, DTOs, observers, and integration classes must **not** depend on the HTTP layer.

**Forbidden** inside `app/Actions/`, `app/Data/`, `app/Observers/`, and `app/Integrations/`:

- Global helpers: `request()`, `auth()`, `session()`, etc.
- Facades: `Request`, `Auth`, and any facade that hides HTTP or the current user/session.

**Required instead:**

- Pass inputs as method arguments (models, DTOs, primitives).
- Inject services via the constructor (`CreateProductAction`, clients, caches).
- Read configuration once in the constructor from `config()` — avoid `env()` outside config files and avoid scattering `Config::` calls.

Infrastructure facades (`DB`, `Log`, `Cache`, `Http`) are acceptable when they match existing project usage; prefer injection when it improves testability.

## External integrations (`app/Integrations/`)

All third-party API clients and integration-specific logic live here — **not** in `app/Services` or generic helpers.

Per vendor, mirror existing layout:

```
app/Integrations/{Vendor}/
  {Vendor}Client.php      # HTTP/SDK calls, auth, low-level API
  Actions/                # integration use cases (execute + DTOs in)
  Data/                   # request/response DTOs (Spatie Data)
    Casts/
    Concerns/
    Interfaces/
```

- **Client**: config from `config/services.php` / `config/*.php`, no `env()` outside config.
- **Actions** (`Integrations/{Vendor}/Actions/`): `execute()` accepting integration DTOs; map to/from domain models via `app/Actions/` when needed.
- **Data**: vendor-specific shapes only; do not leak integration DTOs into Filament forms or unrelated domains.

Shared clients at `app/Integrations/{Name}Client.php` are allowed when there is no vendor subfolder (e.g. `MicrosoftGraphClient`).

## Observers (`app/Observers/`)

Model lifecycle side effects (cache, jobs, related updates) belong in **observer classes**, not in models.

- Class: `App\Observers\{Model}Observer`.
- Register on the model with `#[ObservedBy({Model}Observer::class)]`.
- Keep models focused on relationships, scopes, casts, and small domain methods — avoid `booted()` event chains for side effects (existing exceptions: only touch when already in scope).

## Where to put code (avoid catch-alls)

Do **not** add new logic under generic buckets such as `app/Support/` or `app/Services/` (the latter should stay unused).

| Need | Place instead |
|------|----------------|
| Use case / workflow | `app/Actions/{Domain}/` |
| External API | `app/Integrations/{Vendor}/` |
| Structured data | `app/Data/` or `Integrations/.../Data/` |
| Eloquent side effects | `app/Observers/` |
| HTTP layer | Form requests, controllers, Filament resources |
| Async work | `app/Jobs/` (delegate to actions) |
| Domain-specific attributes/UI | existing `app/Support/Attributes/` only — do not extend `Support/` for unrelated features |

When introducing a new concern, create a **named domain folder** under `app/` rather than a shared junk drawer.

## Tests (Pest)

This project uses **[Pest](https://pestphp.com/)** v4 — not PHPUnit-style test classes.

- New tests: `php artisan make:test --pest {Name}` (feature by default; `--unit` for unit tests).
- Syntax: `test()` / `it()` / `describe()`, `expect()`, datasets; use `pestphp/pest-plugin-livewire` for Filament/Livewire.
- Run: `php artisan test --compact` with a file path or `--filter=` — never generate `extends TestCase` boilerplate unless matching an existing PHPUnit file in the same directory.
- Follow examples and conventions in `tests/Pest.php` and sibling tests.

## Scope

Apply to **new** code and files you are already changing; do not reorganize legacy `app/Support/` unless explicitly requested.
