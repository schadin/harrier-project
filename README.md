# Harrier Project

Зонтичный репозиторий проекта **Harrier** — десктопного и веб-клиента для протокола
Matrix, форка [Cinny](https://github.com/cinnyapp/cinny).

> Это форк. Проект не претендует на оригинальность идеи: Cinny — лучший клиент для
> Matrix, и вся заслуга принадлежит авторам оригинального проекта.

## Состав

| Репозиторий | Роль | Доступ |
|-------------|------|--------|
| [`desktop/`](desktop/) (подмодуль → `schadin/harrier-desktop`) | Десктоп-обёртка на Tauri v2 (Rust) | приватный |
| `schadin/harrier` | Форк веб-клиента Cinny (React + TypeScript + Vite) | публичный |
| `cinnyapp/cinny` | Upstream веб-клиента (git-подмодуль внутри `desktop/`) | upstream |
| `cinnyapp/cinny-desktop` | Upstream десктоп-обёртки | upstream |

## Технологии

- **Десктоп:** Tauri v2, Rust, плагины `tauri-plugin-*` (localhost, window-state,
  clipboard-manager, notification, fs, shell, http, process, os, opener, dialog,
  global-shortcut).
- **Веб:** React 18, TypeScript, Vite, `matrix-js-sdk`, vanilla-extract,
  react-router-dom, TanStack React Query, i18next, jotai.
- **Релизы:** Windows (MSI), Linux (deb/AppImage), macOS (DMG). Веб-версия
  публикуется на GitHub Pages.

## Доменная модель

| Домен | Назначение |
|-------|------------|
| [presence](openspec/specs/presence/spec.md) | Пользовательское присутствие и кастомный статус |
| [messaging](openspec/specs/messaging/spec.md) | Статусы сообщений, маркеры прочтения, вложения |
| [spaces](openspec/specs/spaces/spec.md) | Иерархия пространств, сортировка, навигация |
| [notifications](openspec/specs/notifications/spec.md) | Системные уведомления |
| [desktop-integration](openspec/specs/desktop-integration/spec.md) | Трей, Niri, миграция настроек |
| [appearance](openspec/specs/appearance/spec.md) | Брендинг, темы, иконки |
| [delivery](openspec/specs/delivery/spec.md) | Сборка релизов, веб-публикация, i18n |

## Спецификации

Проект ведётся по [OpenSpec](https://github.com/Fission-AI/OpenSpec) — спек-ориентированной
разработке. Спецификации возможностей находятся в [`openspec/specs/`](openspec/specs/),
контекст проекта — в [`openspec/config.yaml`](openspec/config.yaml).

Задачи ведутся как GitHub Issues в этом репозитории. Ссылка на задачу — её номер
на GitHub (`#NN`).

## Инвариант протокола

Фичи строятся только на механизмах и данных, которые предоставляет протокол Matrix
(события, read receipts, presence через `matrix-js-sdk`). Если в протоколе нет
механизма — функция не реализуется и нестандартные обходные пути не изобретаются.

## Лицензия

AGPL-3.0-only (наследуется от Cinny).
