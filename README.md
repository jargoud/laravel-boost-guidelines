# Laravel Boost Guidelines

Opinionated Laravel & PHP coding guidelines as AI skills, compatible with [Laravel Boost](https://laravel.com/docs/master/boost).

## Installation

```bash
composer require jargoud/laravel-boost-guidelines --dev
php artisan boost:install
```

Select **jargoud/laravel-boost-guidelines** from the list and they'll be installed automatically.

## What's Included

Boost installs a core guideline (`core.blade.php`) that instructs assistants to always activate the skills below when working on Laravel or PHP code.

| Skill | Description |
|-------|-------------|
| `jargoud-php-coding-quality` | Cross-cutting principles, quality bar, scope discipline, and validation workflow |
| `jargoud-php` | PHP 8+ strict typing and modern idioms (`strict_types`, promotion, `readonly`, enums) |
| `jargoud-laravel` | Laravel architecture under `app/` (Actions, DTOs, Integrations, Observers, Pest) |

## Keeping Up to Date

```bash
composer update jargoud/laravel-boost-guidelines
php artisan boost:update
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for recent changes.

## Contributing

Contributions are welcome. Please open an issue or pull request on [GitHub](https://github.com/jargoud/laravel-boost-guidelines).

## Security Vulnerabilities

Please review [the security policy](https://github.com/jargoud/laravel-boost-guidelines/security/policy) on how to report security vulnerabilities.

## Credits

- [Jérémy ARGOUD](https://github.com/jargoud)
- Inspired by [spatie/guidelines-skills](https://github.com/spatie/guidelines-skills)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
