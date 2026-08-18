## Why

Ключ SSSS кэшируется только в памяти (`src/client/secretStorageKeys.js` — `Map`),
поэтому после перезапуска клиент не может использовать ключ бэкапа, пока
пользователь вручную не введёт recovery key / passphrase (`ManualVerification.tsx`).
Из-за этого после каждого перезапуска исторические и пропущенные сообщения
остаются нерасшифрованными до ручных действий, несмотря на наличие ключа бэкапа
и работающего `PerSessionBackupDownloader`.

## What Changes

- Персист recovery key (или производного ключа SSSS) в защищённом хранилище
  (IndexedDB / localStorage) после успешной верификации или ввода recovery key.
- При старте, если ключ доступен: `bootstrapSecretStorage({})` +
  `loadSessionBackupPrivateKeyFromSecretStorage()` без участия пользователя.
- Авто `restoreKeyBackup()` при логине (и по существующему событию
  `KeyBackupDecryptionKeyCached`), прогресс — в существующий
  `backupRestoreProgressAtom`.
- Сохранение ключа — только после успешной проверки (не «сырого» ввода).

## Capabilities

### New Capabilities

_Нет новых доменов._

### Modified Capabilities

- `messaging`: добавляется требование о персисте ключа восстановления и
  автоматическом восстановлении бэкапа ключей после перезапуска клиента без
  ручного ввода recovery key/passphrase.

## Impact

- `desktop/cinny`:
  - `src/client/secretStorageKeys.js` — персист ключа вместо in-memory `Map`;
  - `src/app/components/ManualVerification.tsx`, `SecretStorage.tsx` — запись
    ключа в хранилище после успешной проверки;
  - `src/app/hooks/useRestoreBackupOnVerification.ts` / `BackupRestore.tsx` —
    авто-восстановление при старте;
  - `src/app/state/backupRestore.ts` — существующий прогресс-атом.
- matrix-js-sdk v41.7.0: `bootstrapSecretStorage`, `loadSessionBackupPrivateKeyFromSecretStorage`,
  `restoreKeyBackup`, `KeyBackupDecryptionKeyCached`.
- Никаких новых зависимостей (IndexedDB уже используется для sync/crypto-сторов).
