# Реорганизация репозиториев (фаза −1 переноса ядра на Rust)

## Why

Проект начинает перенос ядра клиента на Rust
([#59](https://github.com/schadin/harrier-project/issues/59)). Синк с upstream
Cinny прекращается окончательно — а значит, исчезает единственная причина, по
которой клиент жил git-подмодулем, а брендинг накатывался патчем на время
сборки. Текущая схема (подмодуль + `rebrand.mjs` + Pages-деплой, клонирующий
десктопный репозиторий ради патча) — мёртвый груз и источник хрупких связей.
Прежде чем строить ядро, репозитории нужно привести в состояние, где веб и
десктоп автономны.

## What Changes

- **Desktop становится монорепо**: клиент из подмодуля `desktop/cinny/`
  переезжает в дерево `harrier-desktop` с сохранением истории (subtree merge).
  Подмодуль упраздняется. **BREAKING**: `schadin/harrier` больше не
  используется как источник клиента для десктоп-сборки.
- **Брендинг коммитится навсегда**: `0002-rebrand-harrier.patch` применяется к
  коду клиента, ассеты переносятся в репозиторий; `scripts/rebrand.mjs`,
  патч и связанные шаги `beforeBuildCommand`/`beforeDevCommand` удаляются.
- **Веб-legacy автономен**: `schadin/harrier` продолжает деплоиться на Pages
  (`schadin.github.io/harrier/` сохраняется); `pages-deploy.yml` очищается от
  клонирования `harrier-desktop`. Netlify/PR-preview workflows остаются как
  есть.
- **Релизный процесс упрощается**: один релизный цикл в `harrier-desktop`
  вместо синхронных релизов двух репозиториев; `update-version.mjs` больше не
  двигает подмодуль. **BREAKING**: теги веб-legacy и десктопа перестают быть
  синхронными.
- **Обновление документации**: `docs/RELEASING.md`, `AGENTS.md` зонтика,
  `openspec/config.yaml` (репозиторная карта).

## Capabilities

### New Capabilities

(нет)

### Modified Capabilities

- `delivery`: публикация Pages больше не зависит от десктопного репозитория;
  релиз публикуется в одном репозитории вместо двух; синхронное
  версионирование подмодуля упраздняется.
- `appearance`: ребрендинг выполняется постоянным коммитом брендинга в код
  клиента, а не применением/откатом патча при сборке.

## Impact

- **Код**: `desktop/` (весь клиент переезжает в дерево, удаление
  `scripts/rebrand.mjs`, `scripts/update-version.mjs`, `0002-rebrand-harrier.patch`,
  правки `tauri.conf.json`, `package.json`); `schadin/harrier` (коммит
  брендинга, очистка `pages-deploy.yml`).
- **CI/CD**: `pages-deploy.yml` (веб), `tauri.yml`/`test.yml`/`archive.yml`
  (desktop, без submodules).
- **Спецификации**: `openspec/specs/delivery/spec.md`,
  `openspec/specs/appearance/spec.md`.
- **Документация**: `docs/RELEASING.md`, `AGENTS.md` (зонтик и desktop),
  `openspec/config.yaml`.
- **Не затрагивается**: Rust-код `src-tauri` (трей, миграция настроек),
  поведение приложения для пользователей, Pages-URL веб-версии.
- **Предусловие для**: фазы 0 (контракт ядра) эпика #59.
