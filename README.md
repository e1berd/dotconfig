# dotconfig

Централизованный репозиторий настроек, конфигураций линтеров/форматтеров и правил для AI-агентов и разработчиков.

## Ветки репозитория

- **`main`**: Базовые универсальные стандарты для современных веб-проектов:
  - `.oxfmtrc.json` — конфигурация ультрабыстрого форматтера `oxfmt`
  - `.oxlintrc.json` — конфигурация линтера `oxlint` (TypeScript, Unicorn, Vue)
  - `.editorconfig` — согласованное форматирование отступов и концов строк
  - `CLAUDE.md` / `AGENTS.md` — строгие правила разработки, отказ от избыточных комментариев, лимиты строк и расширенные стандарты современного CSS (нативный CSS Nesting, `:has()`, Container Queries, `oklch`, `color-mix`, `light-dark`, `@layer`, `subgrid`).

- **`laravel`**: Специализированный пресет для высоконагруженных приложений на **Laravel 13 + PHP 8.5 + FrankenPHP + Inertia.js 3 / Vue 3 + Tailwind 4**:
  - Все конфигурации из `main`
  - `pint.json` — код-стайл PHP от Laravel Pint
  - `phpstan.neon` — статический анализ типов PHPStan / Larastan на уровне 7
  - `CLAUDE.md` / `AGENTS.md` с правилами работы с worker-режимом FrankenPHP, pipe operator `|>`, `literal()`, Orion CRUD, Centrifugo gRPC, Ark UI ScrollArea, дизайном Mosaic и строгими гейтами.

## Использование в проектах

### Подключение как submodule
```bash
git submodule add https://github.com/e1berd/dotconfig.git .dotconfig
```

### Копирование конфигураций
```bash
# Базовый набор (ветка main)
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/main/.oxfmtrc.json -o .oxfmtrc.json
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/main/.oxlintrc.json -o .oxlintrc.json
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/main/.editorconfig -o .editorconfig
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/main/CLAUDE.md -o CLAUDE.md
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/main/AGENTS.md -o AGENTS.md

# Для Laravel-проектов (ветка laravel)
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/pint.json -o pint.json
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/phpstan.neon -o phpstan.neon
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/CLAUDE.md -o CLAUDE.md
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/AGENTS.md -o AGENTS.md
```
