## 1. Зависимости и разрешения

- [x] 1.1 Добавить `tauri-plugin-autostart = "~2.5.1"` в `src-tauri/Cargo.toml` (секция `cfg(not(any(android, ios)))`) с комментарием про ограничение `tauri ^2.12` в 2.6+
  - Использован `~2.5.1`, а не `2.5.1`: при caret-диапазоне cargo подтянул 2.7.0, а тот требует `tauri ^2.12` — обновление фреймворка вынесено в [#57](https://github.com/schadin/harrier-project/issues/57). Тем же ограничением помечен `tauri-plugin-single-instance` (`~2.4.5`).
- [x] 1.2 Добавить `@tauri-apps/plugin-autostart@2.5.1` в `desktop/cinny/package.json` (`npm install --save-exact`)
- [x] 1.3 Добавить `autostart:default` в `src-tauri/capabilities/desktop.json`

## 2. Реализация

- [x] 2.1 Инициализировать плагин в `src-tauri/src/lib.rs`: `tauri_plugin_autostart::init(MacosLauncher::LaunchAgent, None)` — без аргументов (автозапуск = обычный старт, см. #36)
- [x] 2.2 Добавить настройку `autostart: boolean` (дефолт `false`) в `cinny/src/app/state/settings.ts`
- [x] 2.3 Добавить в `cinny/src/app/utils/desktop.ts` обёртку `applyAutostart(enabled): Promise<boolean>` (вызовы `enable`/`disable` плагина, гард `isTauri()`, `try/catch` + лог, возвращает признак успеха)
- [x] 2.4 Добавить `SettingTile` с переключателем в секцию Desktop настроек (`General.tsx`), стиль существующей секции (без i18n, как у «Show Tray Icon»)
- [x] 2.5 Синхронизировать настройку с системой в `DesktopFeatures` (`ClientNonUIFeatures.tsx`): эффект с debounce 100 мс; при неудаче вернуть `false` (защита от повторной попытки после размонтирования)

## 3. Проверка

- [x] 3.1 `cargo check --locked` и `cargo clippy --locked` в `src-tauri` — без ошибок; единственное предупреждение clippy (`unnecessary_cast` в коде libappindicator) существовало до правки. `Cargo.lock`: добавлены только `tauri-plugin-autostart` и `auto-launch` (плюс вторые версии `dirs`/`winreg`/`redox_users`, которых требует `auto-launch`) — версии существующих пакетов, включая `tauri 2.11.3`, не изменились
- [x] 3.2 `npm run check:eslint` (0 errors, 97 warnings — на одну больше из-за нового импорта), `npx tsc --noEmit`, `npx vitest run` (136 тестов), `prettier --write` — чисто
- [x] 3.3 `npm run rebrand:apply && npm run rebrand:revert` в `desktop/` — патч применяется и откатывается (наши правки в `ClientNonUIFeatures.tsx` не затрагивают hunk патча)
- [ ] 3.4 Ручная проверка: переключатель включает/снимает автозапуск (проверка записи автозапуска в системе), в веб-сборке секция скрыта
