# Proposal: fix-backup-restore-retry-loop

## Why

При отсутствии бэкапа ключей или ошибке восстановления авто-восстановление
зацикливается: статус прогресса остаётся `Idle`, guard пропускает, и цепочка
`bootstrapSecretStorage → loadSessionBackupPrivateKeyFromSecretStorage →
restoreKeyBackup` перезапускается на каждом переходе sync в `SYNCING`
(~каждые 20–30 с), создавая постоянную нагрузку на клиент и homeserver.

Связанная задача: schadin/harrier-project#43.

## What Changes

- Попытка авто-восстановления помечается как совершённая после первого запуска
  независимо от результата — повтор на sync-циклах прекращается.
- Повтор попытки возможен только по значимому событию — приходу
  `KeyBackupDecryptionKeyCached` (ключ бэкапа появился в SSSS).
- Колбэк восстановления переводится на ref для статуса прогресса: подписка на
  `ClientEvent.Sync` больше не пересоздаётся на каждом тике прогресса.

## Capabilities

### New Capabilities

- нет

### Modified Capabilities

- **messaging** — требование «Авто-восстановление бэкапа после перезапуска»:
  добавляется ненавязчивость (одна попытка, без циклов при ошибке/отсутствии
  бэкапа).

## Impact

- Код: `desktop/cinny/src/app/hooks/useRestoreBackupOnStartup.ts`.
- Ручное восстановление через настройки и восстановление по верификации
  устройства (`useRestoreBackupOnVerification`) не затрагиваются.
