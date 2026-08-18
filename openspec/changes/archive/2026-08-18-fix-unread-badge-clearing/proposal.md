## Why

В десктоп-клиенте Harrier (Tauri) бейдж непрочитанного в списке комнат и в трее не
снимается в текущей сессии: после открытия и прочтения комнаты она остаётся
помеченной непрочитанной. Отметки исчезают только после перезапуска приложения
(schadin/harrier-project#33).

## What Changes

- Атом `roomToUnreadAtom` перестанет «застревать» на устаревшем серверном счётчике:
  после прочтения комнаты бейдж снимается сразу, без перезапуска.
- `handleTimelineEvent` не будет повторно добавлять комнату как непрочитанную,
  если она фактически прочитана (`roomHaveUnread`).
- Атом будет самокорректироваться при изменении счётчика непрочитанного в
  matrix-js-sdk (`RoomEvent.UnreadNotifications`) — пересчёт записи комнаты.
- Временный диагностический лог для подтверждения механизма, удаляется после
  проверки.

## Capabilities

### New Capabilities

- `unread-badge-clearing`: снятие бейджа непрочитанного сразу после прочтения
  комнаты в текущей сессии, без перезапуска приложения.

### Modified Capabilities

- `messaging`: добавляется требование к снятию бейджа непрочитанного после
  прочтения комнаты (в дополнение к статусам сообщений и маркерам прочтения).

## Impact

- Форк веб-клиента (`desktop/cinny`):
  - `src/app/state/room/roomToUnread.ts` — guard в `handleTimelineEvent`,
    подписка на `RoomEvent.UnreadNotifications` (per-room, `ClientEvent.Room` /
    `ClientEvent.DeleteRoom`);
  - `src/app/utils/notifications.ts` — временный диагностический лог в
    `markAsRead`;
- Бейдж в трее (`src/app/pages/client/ClientNonUIFeatures.tsx`) чинится
  автоматически — читает тот же `roomToUnreadAtom`.
- Зависимости: matrix-js-sdk (события `RoomEvent.UnreadNotifications`,
  `ClientEvent.Room`, `ClientEvent.DeleteRoom`) — без изменений версии.
