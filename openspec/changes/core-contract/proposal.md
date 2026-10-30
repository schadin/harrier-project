# Контракт ядра @harrier/core (фаза 0 переноса ядра на Rust)

## Why

React-код Harrier напрямую зависит от `matrix-js-sdk`: после реорганизации репозиториев в `client/src` обнаружены 232 файла с такими импортами, а объекты SDK используются как общий язык состояния и UI. Это мешает заменить движок на Rust без переписывания клиента. По решению эпика [#59](https://github.com/schadin/harrier-project/issues/59) сначала нужен стабильный API-first контракт с JS-адаптером и проверкой feature parity.

## What Changes

- Создать внутренний TypeScript-пакет `@harrier/core`: типы, view-модели и единый интерфейс ядра для auth, rooms, timeline, send, crypto, media, presence, notifications и опциональных calls.
- Реализовать адаптер поверх `matrix-js-sdk`, включая преобразование событий и состояния SDK в неизменяемые view-модели и перевод команд ядра в вызовы SDK.
- Заменить `MatrixClientProvider` на провайдер ядра и перевести все точки касания клиента на контракт; UI больше не получает `MatrixClient`, `Room`, `MatrixEvent` и другие типы SDK.
- Добавить контрактный тест-сьют с фикстурами sync, пригодный для JS- и будущей Rust-реализации, а также покрыть критичные миграции существующего поведения.
- Добавить ESLint-запрет прямых импортов `matrix-js-sdk` и его subpath-модулей вне адаптера.
- Сохранить текущее поведение JS-движка, включая крипто-UX, звонки и threads; переключение на Rust и допустимые в нём временные регрессы в этот change не входят.

## Capabilities

### New Capabilities

- `core`: стабильный контракт движка, доменные view-модели, команды и подписки, заменяемость реализаций и проверка их поведения контрактными тестами.

### Modified Capabilities

(нет: пользовательские требования существующих доменов сохраняются; меняется их реализация поверх нового контракта)

## Impact

- **Код**: `desktop/client/packages/core/`, `desktop/client/packages/core-matrix-js/`, `desktop/client/packages/core-contract-tests/`, `desktop/client/src/client/`, провайдеры, hooks, state, features и components клиента; 232 файла с прямыми импортами SDK.
- **Зависимости и сборка**: npm workspaces в `desktop/client`, экспорты `@harrier/core`, TypeScript/Vite/Vitest-конфигурация; версия приложения не изменяется.
- **Качество**: новые контрактные фикстуры и тесты, ESLint-гейт, существующие lint/typecheck/test/build и сборка Tauri.
- **Не затрагивается**: веб-legacy `schadin/harrier`, Rust-код `src-tauri`, формат Matrix-событий и пользовательские данные.
- **Предусловие для**: фазы 1 — Rust-реализации read-only ядра за тем же контрактом.
