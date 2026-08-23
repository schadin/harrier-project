# Proposal: fix-custom-status-restore-race

## Why

Восстановление кастомного статуса выполняется на первом SYNCING, когда
собственное `m.presence` ещё не пришло (`presenceStatusMsg === undefined`),
guard пропускает проверку, и устаревший статус из localStorage перезаписывает
актуальный серверный (например, сменённый с другого устройства). Дополнительно
при восстановлении всегда форсируется `presence: 'online'`, что ломает
авто-away.

Связанная задача: schadin/harrier-project#44.

## What Changes

- Восстановление статуса откладывается до прихода собственного presence
  (подписка на `UserEvent.Presence` своего пользователя) со страховочным
  таймаутом после SYNCING.
- Приоритет сервера: если сервер вернул непустой статус, отличающийся от
  локального, локальный кэш синхронизируется с серверным; отправка
  `setPresence` не выполняется.
- Локальный статус восстанавливается только когда серверное значение пустое
  (утеряно/истекло), при этом текущее presence пользователя сохраняется
  (без форсирования `online`).

## Capabilities

### New Capabilities

- нет

### Modified Capabilities

- **presence** — требование «Восстановление статуса после перезапуска»:
  восстановление после прихода собственного m.presence, приоритет серверного
  значения, отсутствие форсирования presence.

## Impact

- Код: `desktop/cinny/src/app/plugins/custom-status.ts`
  (`restoreCustomStatus`), `desktop/cinny/src/app/pages/client/
  ClientNonUIFeatures.tsx` (CustomStatusRestore).
- Установка/снятие статуса и публикация notice в комнату не затрагиваются.
