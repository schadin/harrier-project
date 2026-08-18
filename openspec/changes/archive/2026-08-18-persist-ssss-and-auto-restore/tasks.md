# Tasks

## 1. Персист ключа SSSS в IndexedDB

- [x] 1.1 Расширить `src/client/secretStorageKeys.js`: двухслойное хранилище —
      in-memory `Map` + IndexedDB (`secret-storage-keys`), base64-сериализация
      `Uint8Array`. `storePrivateKey`/`cacheSecretStorageKey` пишут в оба слоя,
      `clearSecretStorageKeys` очищает оба.
- [x] 1.2 Функция загрузки `loadSecretStorageKeys()` — читает из IndexedDB в `Map`
      до создания клиента; вызвать при инициализации (импорт в `initMatrix.ts`).
- [x] 1.3 Убедиться, что `logoutClient` (`initMatrix.ts`) очищает ключи
      (`clearSecretStorageKeys` вызывается при logout и в
      `ClientRoot.tsx:useLogoutListener`).

## 2. Авто-восстановление бэкапа

- [x] 2.1 Новый хук `useRestoreBackupOnStartup` (по образцу
      `useRestoreBackupOnVerification.ts`): при `ClientEvent.Sync === 'SYNCING'`
      (один раз) и по `KeyBackupDecryptionKeyCached`:
      `bootstrapSecretStorage({})` → `loadSessionBackupPrivateKeyFromSecretStorage()`
      → `restoreKeyBackup({ progressCallback })`.
- [x] 2.2 Guard: не запускать повторное восстановление, пока
      `backupRestoreProgressAtom` не в `Idle`/`Done`.
- [x] 2.3 Подключить хук в `ClientNonUIFeatures.tsx`.

## 3. Сохранение ключа после успешной проверки

- [x] 3.1 Проверить `ManualVerification.tsx`: `storePrivateKey` вызывается только
      после успешной проверки — дополнительных изменений не требуется.

## 4. Проверка

- [x] 4.1 `npm run lint` и сборка/typecheck форка `cinny`, убедиться в отсутствии
      регрессий верификации и восстановления бэкапа.
