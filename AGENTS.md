# Правила проекта — Laravel 13 Fullstack

Стек: Laravel 13 · Inertia.js 3 + Vue 3 · Orion (auto-CRUD) · Filament 5 (админка) · Laravel Fortify (auth) · Centrifugo (real-time) · PostgreSQL 17 · FrankenPHP (worker-режим) · Tailwind 4 · Vuzeno (UI, shadcn-vue registry) · Mosaic Design System.

Эти правила обязательны для соблюдения всеми разработчиками и AI-агентами. Они имеют абсолютный приоритет над стилем сгенерированного кода.

---

## Веб-сервер (FrankenPHP worker-режим)

Код пишется совместимым с long-running процессами (FrankenPHP worker-режим):

- **Никакого глобального / статического мутабельного состояния между запросами.** Статические свойства, синглтоны и кэш в памяти (`static $`, `app()->singleton()` с мутабельным состоянием) живут между запросами в worker-режиме. Если значение должно стартовать заново для каждого запроса — сбрасывать его в middleware или использовать `Request::macro`/`Context` (Laravel 13).
- **Никаких `exit()`, `die()`, `dd()` в коде.** Они убивают worker-процесс целиком. Для отладки — `logger()`, `dump()`, Ray.
- **Осторожно с `register_shutdown_function` и глобальными обработчиками ошибок.** В worker-режиме они срабатывают не один раз на процесс, а накапливаются на каждый запрос. Использовать `app()->terminating()` или middleware вместо shutdown-хуков.
- **Не полагаться на то, что процесс завершится после ответа.** Код после `return response()` в контроллере может не выполниться (Laravel `terminate()` вызывается, но сам PHP-процесс остаётся жить). Тяжёлую пост-обработку — только в очередь (`Queue::push`).
- **Не хранить соединения (БД, Redis, HTTP, gRPC) в статических свойствах.** Соединения могут протухнуть между запросами. Клиенты обязаны проверять живость соединения или пересоздаваться.

---

## Философия кода: без поясняющих комментариев

Поясняющих комментариев в коде нет — ни в PHP, ни в Vue/TS. Код самодокументируемый: имя метода, имя переменной и структура выражают намерение без прозы.

- Никаких `//`, `#`, `/* */`, `{{-- --}}` комментариев, объясняющих «что делает строка».
- Docblock — только там, где он несёт типовую информацию, недоступную из сигнатуры (`@param array<string, int>`), а не пересказ названия метода.
- Если фрагмент непонятен без комментария, это сигнал переименовать или выделить метод/composable/Action, а не дописать пояснение.
- Исключение: фиксация внешнего ограничения, которое невозможно вывести из кода (баг в чужой библиотеке, требование закона со ссылкой на норму, обходной путь с датой и причиной).

---

## Размер файла

- Ориентир — до 350 строк, жёсткий потолок — 400.
- Превышение допустимо для Vue SFC и Blade-шаблонов, где разметка не делится без потери связности, а также для сгенерированных файлов (`resources/js/components/ui/**`).
- Класс или компонент, переваливший за 400 строк из-за накопления логики, разбивается: бизнес-логика уходит в `app/Actions`, разметка/логика Vue — в под-компоненты и composables.

---

## Версии и зависимости

Только актуальные версии инструментов, никакого legacy.

- Перед добавлением зависимости проверять последнюю стабильную версию на `packagist.org` / `npmjs.com`, а не копировать из памяти.
- Использовать современные API Laravel 13: атрибуты `#[Fillable]`/`#[Hidden]` вместо свойств, `casts()` методом.
- Никаких пакетов, полная функциональность которых требует платной подписки.
- Deprecation-предупреждения чинятся сразу.

---

## Хелперы и `literal()`

Максимально использовать штатные хелперы и composables Laravel, Inertia, Vue, VueUse и Orion вместо ручных конструкций. Перед написанием цикла или ручного преобразования проверить, нет ли штатного метода.

- В PHP предпочитать: `str()`, `collect()`, `data_get()`, `blank()`, `filled()`, `rescue()`, `throw_if()`, `throw_unless()`, `tap()`, `transform()`, `value()`, `once()`.
- Во Vue предпочитать: Composition API (`computed`, `toRef()`, `toRefs()`, `watch()`, `watchEffect()`) и composables VueUse (`useEventListener()`, `watchDebounced()`, `useDebounceFn()`).
- Собственный helper или composable оправдан только предметной логикой или повторным использованием, а не простой обёрткой над готовым API.

**`literal()` — способ по умолчанию для любой одноразовой именованной ad-hoc структуры везде, где он применим.** Это относится к результатам Actions и сервисов, координации в контроллерах, вложенным Inertia-props и JSON-конвертам:

```php
return literal(findings: $findings, failedChunks: $failed);
```

JSON-ответы:
```php
return response()->json(literal(status: 'ok', data: $data));
```

---

## Возможности PHP 8.5 и Laravel 13

- **Pipe operator `|>` — предпочтительный способ выразить линейную цепочку преобразований одного значения**:
  ```php
  $text |> trim(...) |> mb_strtolower(...) |> $this->normalize(...)
  ```
  Справа от `|>` — callable: функция (`trim(...)`), статический метод (`File::ensureDirectoryExists(...)`), метод объекта (`$this->normalize(...)`), first-class callable или Action.
- `array_first()`, `array_last()`, `array_any()`, `array_all()` вместо `reset()`, `end()` и ручных циклов.
- First-class callable синтаксис `foo(...)` везде вместо строковых имён.
- Property hooks и `readonly` классы для value-объектов.
- **`and`/`or` вместо `&&`/`||` в управляющих guard-конструкциях**:
  `$user = User::find($id) or abort(404);`
- `??=` вместо `if (!isset($x)) { $x = ...; }`, `??` вместо тернарного `isset()`, `?->` вместо ручной проверки на `null`, `?:` вместо `$x ? $x : $y`.
- `match` вместо `switch` для ветвления по значению без fallthrough.

---

## Архитектура и структура

- Бизнес-логика — в `app/Actions/<Домен>/<Действие>.php`, один класс = одно действие. Контроллер/Orion-хук только вызывает Action.
- **Типовой CRUD — через Orion (`Orion::resource()`), не ручные контроллеры.** Ручной контроллер пишется только там, где логика выходит за рамки CRUD (стриминг, запуск сложного анализа, вебхуки).
- **Inertia-контроллеры** отдают страницы (`inertia()` с типизированными props).
- **Orion-эндпоинты** — точечные операции внутри страницы (поиск, пагинация, обновление записи без reload) через TS SDK Orion на клиенте (`$query()/$attributes/$save()`), а не через ручной `fetch`.
- Изоляция команды — таблица `teams`, scope на Eloquent-моделях по `team_id`.
- Vue: страницы — `resources/js/pages/`, компоненты — `resources/js/components/`, примитивы Vuzeno — `resources/js/components/ui/`.
- Админка — Filament, не Inertia-страницы.

### Клиентские запросы
1. CRUD: Orion TS SDK.
2. Данные страницы / chrome: типизированные props через Inertia (`Inertia::optional()`, `router.reload({ only: [...] })`).
3. Мутации без обособленного JSON-ответа: `router.visit()` с Wayfinder-объектом маршрута.
4. Отдельный JSON-эндпоинт: нативные `fetch`, `URL`, `Request`, `Response`.
5. **`axios` в прикладном коде категорически запрещён.**

---

## Скролл (ScrollArea)

Любой скролл (вертикальный список, лента чата, форма, попап) — через `resources/js/components/scroll-area/` (обёртка над `@ark-ui/vue/scroll-area`), не нативный `overflow-y-auto`.
- `ScrollArea.Root orientation="vertical" class="..." content-class="..."`.
- **Обязательно `max-w-full` рядом с `w-full`**: у Vuzeno-заготовки зашит лимит `max-w-[calc(100vw-8rem)]`, который на мобильных устройствах обрезает панель раньше края.
- Не оборачивать в ScrollArea контейнеры с динамически растягиваемой высотой `h-full`, если процентная высота вложенных блоков должна сохраняться.

---

## Современный CSS и Nesting в приложении

Приложение использует Tailwind 4, нативные переменные и расширенные стандарты CSS3/CSS4.

### Нативный CSS Nesting
Никаких SCSS-препроцессоров. В стилях (`app.css`, SFC-блоках `<style>`) используется нативный CSS Nesting:
```css
.chat-bubble {
  position: relative;
  padding: 0.75rem 1rem;
  border-radius: var(--radius-lg);
  background-color: var(--card);

  &:hover {
    background-color: color-mix(in oklab, var(--card) 95%, var(--foreground));
  }

  &[data-mine="true"] {
    background-color: var(--primary);
    color: var(--primary-foreground);

    & .author-label {
      opacity: 0.8;
    }
  }

  @media (min-width: 768px) {
    padding: 1rem 1.25rem;
  }

  @container (min-width: 500px) {
    display: flex;
    align-items: center;
  }
}
```

### Реляционный псевдокласс `:has()`
Использовать `:has()` для контекстной стилизации предков:
```css
/* Выделение контейнера при наличии фокуса в поле ввода */
.search-form:has(input:focus-visible) {
  border-color: var(--ring);
  box-shadow: 0 0 0 2px var(--ring);
}

/* Скрытие или отключение действий при пустом или невалидном состоянии */
.table-container:has(tbody:empty) .table-pagination {
  display: none;
}
```

### Контейнерные запросы (Container Queries)
Виджеты, чаты, сайдбары и карточки сущностей адаптируются под свой родительский контейнер (`@container`), обеспечивая независимость от размера окна браузера:
```css
.entity-grid-item {
  container-type: inline-size;
}

.entity-card {
  @container (min-width: 400px) {
    grid-template-columns: auto 1fr;
  }
}
```

### Цветовые пространства, `color-mix()` и `light-dark()`
- Токены тем задаются в `app.css`.
- Динамические оттенки, кольца фокуса и прозрачности рассчитываются через `color-mix(in oklab, ...)`:
  `shadow-[0_0_0_3px_color-mix(in_oklab,var(--foreground)_10%,transparent)]`
- Темизация поверхностей поддерживает `light-dark()`:
  `--surface: light-dark(#ffffff, #101012);`

### CSS Subgrid
В карточках списков (Orion/Inertia списки) использовать `subgrid` для выравнивания заголовков, описания и кнопок действий между рядами.

---

## Дизайн-система (Mosaic)

Визуальный контракт — Mosaic (Clerk).
- **Brand-кнопки используют градиент, остальные контролы — плоскую заливку.** `default`-кнопка идёт от `primary` к `primary-dk`. Inset-shadow на интерактивных элементах запрещён.
- **Радиус — по ролям, не по размеру.** `--radius: 0.5rem` разворачивается в `--radius-sm/md/lg/xl` = `4/6/8/12px`.
  `rounded-md` — control (кнопка, инпут sm/md), `rounded-lg` — крупный инпут, `rounded-xl` — контейнер (Card).
- **Фокус разный для кнопок и полей.** У кнопки — цветное кольцо (`focus-visible:ring-2 ring-ring ring-offset-2`), у инпута — нейтральный halo:
  `focus-visible:border-muted-foreground focus-visible:shadow-[0_0_0_3px_color-mix(in_oklab,var(--foreground)_10%,transparent)]`.
- **Размеры контролов**:
  - `sm`: h-7 / px-2.5 (кнопка), h-7 / px-3 (инпут)
  - `md`: h-8 / px-3 (стандартные кнопки и инпуты)
  - `lg`: h-9 / px-3 (крупные контролы)
- **Хром приложения (шапка, сайдбар)**: брендовый акцент, не поверхность контента. Токены `--header`, `--header-foreground`, `--header-border` не зависят от темы контента.

---

## Real-time (Centrifugo) и Межсервисное взаимодействие

- **Real-time**: Паблиш событий — через gRPC API Centrifugo (`CentrifugoApi`, вендоренный `proto/centrifugo/v1/api.proto`), не HTTP API и не Laravel Broadcasting.
  Событие на клиенте только триггерит повторный запрос/инвалидацию через Orion TS SDK.
- **RPC по умолчанию**: Бэкенд-сервисы взаимодействуют по gRPC с типизированным контрактом в `proto/`. Клиенты сервисов биндятся как `scoped` (не `singleton`), чтобы gRPC-канал не перетекал между worker-запросами.
- **Health Check**:
  - gRPC: реализация `grpc.health.v1.Health` (`Check`/`Watch`).
  - HTTP: `GET /health` -> JSON `{"status": "ok"}`.
  - Контейнер приложения не должен требовать `service_healthy` от внешних микросервисов для своего старта.

---

## Auth (Laravel Fortify)

- Авторизация и регистрация через Laravel Fortify.
- Кастомный `CreateNewUser` создаёт пользователя и привязанную заявку в `registration_requests`.
- Доступ гейтится через `EnsureRegistrationApproved` middleware: до одобрения админом в Filament — редирект на `/pending-approval`.
- Обязательное подтверждение Email до одобрения заявки.

---

## i18n

**Никакого raw пользовательского текста в коде.**
- Сервер: YAML-файлы в `lang/<locale>/*.yaml` (`YamlFileLoader`), RU и EN.
- Vue SFC: локальный блок `<i18n lang="yaml">` рядом с шаблоном. Не выносить разовый текст в общий каталог.
- Общие клиентские переводы: `resources/js/locales/{ru,en}.yaml`.
- Форматирование дат, чисел и плюрализация — средствами Vue I18n / `Intl`.
- Любой новый ключ сразу добавляется для RU и EN с идентичной структурой.

---

## Гейты проверки

Перед коммитом обязаны проходить:

```bash
./vendor/bin/pint --test
composer run types:check
php artisan test
pnpm oxlint resources/js vite.config.ts
pnpm oxfmt resources/js vite.config.ts --check
pnpm run types:check
pnpm run test
```
