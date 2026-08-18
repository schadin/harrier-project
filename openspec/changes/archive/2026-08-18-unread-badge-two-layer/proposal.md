## Why

После фикса #33 бейдж комнаты снимается сразу, но **бейдж Space остаётся** до
перезапуска приложения (schadin/harrier-project#34). Причина: атом
`roomToUnreadAtom` ведётся инкрементально (дельты), а добавленный в #33
`updateRoomUnread` напрямую вмешивается в агрегатные записи Space.

## What Changes

- **Слой комнат**: единая точка записи `recomputeRoom(room)` на все события
  (Timeline, Receipt, UnreadNotifications, Membership); предикат `roomIsUnread`
  (локальный признак по read receipt при загруженном таймлайне, серверный
  счётчик — фолбэк при незагруженном). Space напрямую не записываются.
- **Слой производных**: бейдж Space в сайдбаре — живая сумма детей
  (`useRoomsUnread(allChild)`, как уже делают папки и лобби Space); трей и
  favicon исключают Space-агрегаты (устраняет двойной счёт).
- Временный диагностический лог для подтверждения механизма, удаляется после
  проверки.

## Capabilities

### New Capabilities

- `unread-badge-two-layer`: бейджи непрочитанного комнат и Space как двухслойная
  архитектура — точные записи комнат в атоме + производные суммы для Space/трея.

### Modified Capabilities

- `messaging`: уточняется требование снятия бейджа — бейджи Space и трея снимаются
  без перезапуска и без двойного счёта.

## Impact

- Форк веб-клиента (`desktop/cinny`):
  - `src/app/utils/room.ts` — новый предикат `roomIsUnread`;
  - `src/app/state/room/roomToUnread.ts` — `recomputeRoom`, переключение писателей,
    `getUnreadInfos` на `roomIsUnread`;
  - `src/app/pages/client/sidebar/SpaceTabs.tsx` — бейдж вкладки Space на живую
    сумму детей;
  - `src/app/pages/client/ClientNonUIFeatures.tsx` — трей и favicon исключают
    Space-агрегаты;
- Зависимости: matrix-js-sdk (события `RoomEvent.UnreadNotifications`,
  `RoomEvent.Receipt`, `RoomEvent.Timeline`) — без изменения версии.
