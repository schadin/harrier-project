## Why

Автозапуск при входе в систему заявлен как ожидаемая функция десктопного клиента, но отсутствует: пользователю приходится каждый раз запускать Harrier вручную, хотя клиент постоянно живёт в трее (когда он включён) и работает как основной клиент Matrix.

## What Changes

- В настройках Desktop появляется переключатель «Запускать Harrier при входе в систему» (виден только в десктопной сборке).
- При включении приложение регистрирует автозапуск в системе (Windows — реестр/Startup, Linux — XDG autostart, macOS — LaunchAgent), при выключении — снимает регистрацию.
- Состояние переключателя синхронизируется с реальным состоянием автозапуска в системе: если регистрация не удалась, переключатель возвращается в выключенное положение.
- Значение настройки сохраняется вместе с остальными настройками клиента.

## Capabilities

### New Capabilities

- _(нет)_

### Modified Capabilities

- `desktop-integration`: добавляется требование «Автозапуск при входе в систему» — переключатель в настройках Desktop управляет регистрацией автозапуска в операционной системе.

## Impact

- `desktop/src-tauri/Cargo.toml` — зависимость `tauri-plugin-autostart`.
- `desktop/src-tauri/src/lib.rs` — инициализация плагина.
- `desktop/src-tauri/capabilities/desktop.json` — разрешение `autostart:default`.
- `desktop/cinny/package.json` — JS-пакет `@tauri-apps/plugin-autostart`.
- `desktop/cinny/src/app/state/settings.ts` — настройка `autostart`.
- `desktop/cinny/src/app/utils/desktop.ts` — обёртки включения/отключения автозапуска.
- `desktop/cinny/src/app/features/settings/general/General.tsx` — переключатель в секции Desktop.
- `desktop/cinny/src/app/pages/client/ClientNonUIFeatures.tsx` — синхронизация настройки с системой (файл входит в патч ребрендинга — проверить его применение).
