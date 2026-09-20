# dotconfig (Ветка `laravel`)

Специализированная конфигурация для высокопроизводительных проектов на **Laravel 13 + PHP 8.5 + FrankenPHP (worker mode) + Inertia.js 3 / Vue 3 + Tailwind 4**.

Базируется на проверенной архитектуре `lexai`.

## Содержимое ветки

- `.oxfmtrc.json` — конфигурация `oxfmt` (без точек с запятой, одинарные кавычки).
- `.oxlintrc.json` — конфигурация `oxlint` (плагины: typescript, unicorn, vue).
- `pint.json` — стандарты форматирования PHP для Laravel Pint.
- `phpstan.neon` — статический анализ типов Larastan (Level 7).
- `.editorconfig` — стандартизированные отступы для PHP, Vue, YAML, JSON.
- `CLAUDE.md` и `AGENTS.md` — исчерпывающие инструкции для разработчиков и AI-агентов:
  - Long-running worker-режим FrankenPHP (управление памятью, запрет утечек, безопасное завершение).
  - Нулевая терпимость к лишним комментариям, лимиты длины файлов (≤ 350–400 строк).
  - Паттерн `literal()` по умолчанию для именованных структур и ответов.
  - Синтаксис PHP 8.5: pipe operator `|>`, first-class callables, `readonly`, `and`/`or` guards.
  - **Современный CSS и нативный Nesting**: нативный W3C CSS Nesting, `:has()`, Container Queries, `color-mix`, `light-dark`, `@layer`, `subgrid`.
  - Дизайн-система Mosaic, специфика Ark UI ScrollArea (`max-w-full`), Fortify auth, Centrifugo gRPC, строгие YAML i18n правила и гейты качества.

## Быстрый старт для Laravel проекта

Скопировать файлы конфигурации в корень нового или существующего проекта:

```bash
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/.oxfmtrc.json -o .oxfmtrc.json
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/.oxlintrc.json -o .oxlintrc.json
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/pint.json -o pint.json
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/phpstan.neon -o phpstan.neon
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/.editorconfig -o .editorconfig
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/CLAUDE.md -o CLAUDE.md
curl -sSL https://raw.githubusercontent.com/e1berd/dotconfig/laravel/AGENTS.md -o AGENTS.md
```
