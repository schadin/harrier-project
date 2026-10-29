# AGENTS.md — Harrier Project

## Обзор

**Harrier Project** — зонтичный репозиторий проекта **Harrier**: десктопного и
веб-клиента для протокола Matrix, форка [Cinny](https://github.com/cinnyapp/cinny).
Проект ведётся по спецификациям [OpenSpec](https://github.com/Fission-AI/OpenSpec):
сначала фиксируется договорённость о том, что строим, затем — реализация.

- **Формат:** форк. Не претендуем на оригинальность идеи: Cinny — лучший клиент
  для Matrix, заслуга принадлежит авторам оригинала.
- **Лицензия:** AGPL-3.0-only (наследуется от Cinny).
- **Основная ветка:** `harrier` (в подмодуле `desktop/`).

## Состав репозиториев

| Репозиторий | Роль | Подключение |
|-------------|------|-------------|
| `schadin/harrier-project` | Зонтик: спецификации, задачи, координация | этот репозиторий |
| `schadin/harrier-desktop` | Монорепо: клиент React + десктоп-обёртка Tauri v2 (Rust) | подмодуль `desktop/` |
| `schadin/harrier` | Автономный веб-legacy (React + TS + Vite), только критические фиксы | отдельный репозиторий |
| `cinnyapp/cinny` | Upstream веб-клиента; синхронизация прекращена | — |
| `cinnyapp/cinny-desktop` | Upstream десктоп-обёртки | — |

## Структура зонтика

```
harrier-project/
├── desktop/                        # Git-подмодуль: schadin/harrier-desktop
├── openspec/
│   ├── config.yaml                 # Контекст проекта (стек, домены, правила)
│   └── specs/<домен>/spec.md       # Спецификации возможностей
├── .opencode/
│   ├── commands/opsx-*.md          # Slash-команды OpenSpec (/opsx:propose и т.д.)
│   └── skills/openspec-*.md        # Навыки OpenSpec
└── README.md                       # Документация зонтика
```

## Технологический стек

### Десктоп (`desktop/` — Tauri v2 + Rust)
| Компонент | Назначение |
|-----------|------------|
| Tauri 2.x | Основной фреймворк |
| tauri-plugin-localhost | Раздача статики фронтенда на порту 44548 (release) |
| tauri-plugin-window-state | Сохранение позиции/размера окна |
| tauri-plugin-clipboard-manager | Вставка изображений из буфера обмена |
| tauri-plugin-notification | Системные уведомления |
| tauri-plugin-fs / shell / http / process / os / opener / dialog | Общие возможности |
| tauri-plugin-global-shortcut | Глобальные горячие клавиши (desktop only) |

### Веб («schadin/harrier» — автономный веб-legacy, форк Cinny)
React 18, TypeScript, Vite, matrix-js-sdk, vanilla-extract, react-router-dom,
TanStack React Query, i18next, jotai.

### Особенности архитектуры
- Два режима: **dev** (Vite HMR, порт 8080) и **release** (статика из
  `client/dist` через localhost-плагин на порту 44548).
- **Брендинг в исходниках:** брендинг Harrier закоммичен в `desktop/client/` и
  веб-legacy; сборка не требует применения или отката патчей.
- **Миграция настроек:** при первом старте `in.cinny.app` → `io.github.schadin.harrier`
  (копирование без удаления, маркер `migration.success`).
- Версионирование десктопа синхронизировано: npm, `Cargo.toml`, `tauri.conf.json`;
  веб-legacy версионируется независимо.

## Доменная модель

| Домен | Спецификация | Возможности |
|-------|--------------|-------------|
| presence | [spec](openspec/specs/presence/spec.md) | Кастомный статус, пресеты, присутствие, восстановление |
| messaging | [spec](openspec/specs/messaging/spec.md) | Статусы сообщений, маркеры прочтения, вложения |
| spaces | [spec](openspec/specs/spaces/spec.md) | Сортировка комнат в пространствах, навигация |
| notifications | [spec](openspec/specs/notifications/spec.md) | Системные уведомления, превью, E2E-комнаты |
| desktop-integration | [spec](openspec/specs/desktop-integration/spec.md) | Трей, Niri, буфер обмена, миграция |
| appearance | [spec](openspec/specs/appearance/spec.md) | Брендинг, темы Dracula/Alucard, иконка |
| delivery | [spec](openspec/specs/delivery/spec.md) | Сборка релизов, Pages, i18n |
| command-palette | [spec](openspec/specs/command-palette/spec.md) | Быстрый переход и команды, история, непрочитанные чаты |
| dstaskbot | [spec](openspec/specs/dstaskbot/spec.md) | DsTaskBot: команды, карточки задач, сворачивание |

## Спецификации и рабочий процесс (OpenSpec)

- Контекст проекта — `openspec/config.yaml` (стек, домены, инвариант протокола).
- Спецификации доменов — `openspec/specs/<домен>/spec.md` (Requirements + Scenarios
  в формате WHEN/THEN).
- Новые изменения — через `/opsx:propose`: создаётся change (proposal → design →
  tasks), после утверждения реализуется (`/opsx:apply`), по завершении архивируется
  (`/opsx:archive`), что синхронизирует главные спецификации.
- Просмотр всей картины: `openspec view`, `openspec validate --specs`.

## Правила разработки

### Возможности протокола

1. Работаем в рамках возможностей протокола Matrix: фичи строятся только на тех
   механизмах и данных, которые протокол предоставляет (события, read receipts,
   статусы из matrix-js-sdk).
2. Если в протоколе нет механизма для желаемой функции — не реализуем её и не
   изобретаем нестандартные обходные пути.

### Задачи

1. Все доработки ведутся как GitHub Issues в этом репозитории
   (`https://github.com/schadin/harrier-project/issues`). В заголовках и сообщениях
   коммитов задача указывается номером issue на GitHub (`#NN`). В коммитах
   связанных репозиториев (`desktop/`, веб-legacy `schadin/harrier`) для ссылки
   на issue зонтика использовать полный вид `schadin/harrier-project#NN`.
2. Описание задачи содержит цель, план реализации и учёт изменений по ходу работы.
3. После завершения задачи коммит делается только после подтверждения пользователем.
4. Крупные многомесячные работы ведутся как эпик-issue в этом репозитории и
   трекаются через GitHub Project пользователя `schadin` (Board + Roadmap, поле
   `Phase`). Первый эпик — «Harrier → Rust-ядро»
   ([project #1](https://github.com/users/schadin/projects/1),
   эпик [#59](https://github.com/schadin/harrier-project/issues/59)).
5. Гранулярность трекинга: Project — эпик и фазы; issue фазы — отдельный OpenSpec
   change; шаги реализации — `tasks.md` внутри change. Отдельные задачи из
   `tasks.md` в Project не выносятся.
6. На время эпика может действовать feature freeze на затрагиваемый слой: новые
   фичи не начинаются, уже открытые изменения доводятся до конца. Область и срок
   freeze описываются в теле эпика.
7. В release notes обоих репозиториев задачи указываются полными
   Markdown-ссылками на issues зонтика:
   `[#50](https://github.com/schadin/harrier-project/issues/50)`. Короткая запись
   `#50` недопустима — GitHub резолвит её в issues того репозитория, где
   опубликован релиз, а в `schadin/harrier-desktop` и `schadin/harrier` issues не
   ведутся, и ссылка ведёт в пустоту. Внутренние идентификаторы (`TK-00037`)
   не используются.

### Версионирование

1. Версию приложения изменять **только по явной команде пользователя**
   (`npm run bump <version>` в `desktop/`). Самостоятельно поднимать версию нельзя.
2. Процесс выпуска предрелизов и релизов — по
   [docs/RELEASING.md](docs/RELEASING.md): чек-листы, правила оформления release
   notes, создание десктоп-релиза и автономные релизы веб-legacy, откат при сбое.

### Ветки и публикация

1. Основная ветка разработки — `harrier` (в `desktop/`).
2. **Push никогда не выполнять** без явного указания пользователя — все изменения
   остаются локально до подтверждения.

### README.md

1. После завершения доработки спрашивать пользователя, нужно ли добавить описание
   функции в `README.md` (зонтик или `desktop/`).

## Локальная разработка десктоп-клиента

Репозиторий `desktop/` — обычный git-подмодуль; для разработки открывать его как
отдельный репозиторий (`cd desktop`, git-root там). Основные команды:

```sh
cd desktop/client && npm ci
cd ..
npm ci
npm run tauri dev      # dev-режим с HMR
npm run tauri build    # продакшен-сборка
```

## Полезные ссылки

- [schadin/harrier-project — issues](https://github.com/schadin/harrier-project/issues)
- [cinny-desktop releases](https://github.com/schadin/harrier-desktop/releases)
- [Веб-клиент Harrier на Pages](https://schadin.github.io/harrier/)
- [Cinny Web App](https://app.cinny.in/)
- [Tauri v2 Documentation](https://v2.tauri.app/)
- [Matrix protocol](https://matrix.org/)
- [OpenSpec](https://github.com/Fission-AI/OpenSpec)