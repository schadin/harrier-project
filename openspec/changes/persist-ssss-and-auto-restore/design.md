## Context

Ключ SSSS кэшируется только в памяти: `src/client/secretStorageKeys.js` — `Map`,
который заполняется через `storePrivateKey` (ручная верификация в
`ManualVerification.tsx:139`) и `cacheSecretStorageKey` (SDK-колбэк). После
перезапуска клиента `Map` пуст, `getSecretStorageKey` возвращает `undefined`, и
исторические сообщения остаются нерасшифрованными до ручного ввода recovery
key/passphrase.

Восстановление бэкапа сейчас:
- ручное — «Restore Backup» (`BackupRestore.tsx:155`);
- автоматическое только после события верификации —
  `useRestoreBackupOnVerification.ts` (реагирует на `KeyBackupDecryptionKeyCached`,
  который приходит только после того, как ключ уже в SSSS).

## Goals / Non-Goals

**Goals:**
- Персист recovery key (производного ключа SSSS) в локальном защищённом хранилище.
- Авто-восстановление бэкапа ключей после перезапуска без ручного ввода.
- Прогресс восстановления в существующем `backupRestoreProgressAtom`.

**Non-Goals:**
- Хранение в OS keychain через Tauri (веб-клиент работает и на GitHub Pages;
  desktop-обёртка не добавляет зависимостей).
- Синхронизация ключа между аккаунтами.
- Изменение механики бэкапа ключей Matrix.

## Decisions

### 1. Персист ключа в IndexedDB

`src/client/secretStorageKeys.js` — расширяем до двухслойного хранилища:
- in-memory `Map` (как сейчас) — быстрый доступ для SDK-колбэков;
- IndexedDB (существующая БД `web-sync-store` не подходит — она очищается
  `clearCacheAndReload`; используем отдельную БД `secret-storage-keys` с одной
  записью `{ keyId, privateKeyBase64 }`).

Операции:
- `storePrivateKey` / `cacheSecretStorageKey` — пишут в `Map` **и** в IndexedDB;
- при старте клиента (импорт модуля / `initMatrix`) — `loadSecretStorageKeys()`
  читает из IndexedDB и заполняет `Map` до создания клиента (колбэки
  `cryptoCallbacks` должны видеть ключ с первого вызова);
- `clearSecretStorageKeys` — чистит `Map` и удаляет запись из IndexedDB
  (вызывается при logout — `logoutClient` вызывает `clearStores`).

`privateKey` (`Uint8Array`) сериализуется в base64.

### 2. Авто-восстановление бэкапа при старте

Новый хук `useRestoreBackupOnStartup` (по образцу `useRestoreBackupOnVerification`):
- после `ClientEvent.Sync === 'SYNCING'` (один раз) и по
  `KeyBackupDecryptionKeyCached`:
  1. `crypto.bootstrapSecretStorage({})` — загрузить ключи SSSS из account data;
  2. `await crypto.loadSessionBackupPrivateKeyFromSecretStorage()` — извлечь ключ
     бэкапа из SSSS;
  3. `await crypto.restoreKeyBackup({ progressCallback })` — восстановить ключи;
- прогресс пишется в `backupRestoreProgressAtom` (существующий атом,
  `src/app/state/backupRestore.ts`);
- guard: не запускать восстановление, если уже идёт (статус атома не `Idle`/`Done`);
- подключается в `ClientNonUIFeatures.tsx`.

### 3. Сохранение ключа после успешной проверки

`ManualVerificationTile.verifyAndRestoreBackup` (`ManualVerification.tsx:139`)
уже вызывает `storePrivateKey` только после успешной проверки ключа
(`checkKey` в `SecretStorageRecoveryKey`/`SecretStorageRecoveryPassphrase`) —
это точка персиста. Ничего вручную «сырое» не сохраняем.

## Risks / Trade-offs

- **Безопасность**: ключ в IndexedDB не защищён так же, как в OS keychain.
  Trade-off принят: пользователь явно включает «без ручного ввода». При logout
  ключ удаляется.
- **Двойное восстановление**: `useRestoreBackupOnStartup` + существующий
  `useRestoreBackupOnVerification` могут сработать одновременно — guard по
  статусу атома `backupRestoreProgressAtom` исключает повторный `restoreKeyBackup`.
- **ИндекседБД очищается** при «Clear Cache» (`clearCacheAndReload`) — это
  ожидаемо: после очистки кэша требуется ручное восстановление (иначе нет и
  синхронизированной истории).
