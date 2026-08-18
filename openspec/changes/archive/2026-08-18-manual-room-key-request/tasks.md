# Tasks

## 1. Утилита запроса ключа

- [x] 1.1 Создать `src/app/utils/keyRequest.ts` с `requestRoomKey(mx, mEvent)`:
      `getWireContent()` → `{ room_id, sender_key, session_id }`, получатели —
      `mEvent.getKeyRequestRecipients(mEvent.getSender())`, content
      `{ action: 'request', request_id: <UUID>, body: { room_id, sender_key, session_id, algorithm } }`,
      отправка `mx.sendToDevice(EventType.RoomKeyRequest, ...)`.
- [x] 1.2 Дедуп: module-level `Map<eventId, timestamp>`, повторный запрос в течение
      60 с игнорируется.

## 2. Проброс действия в UI

- [x] 2.1 `RoomTimeline.tsx`: передать `onRequestKey={() => requestRoomKey(mx, mEvent)}`
      в `RenderMessageContent` рядом с `onVerifyDevice` / `onRestoreBackup`.
- [x] 2.2 `Notifications.tsx`, `RoomPinMenu.tsx` — проброшен `onRequestKey`
      (`SearchResultGroup` не рендерит `m.bad.encrypted` — не требуется).
- [x] 2.3 `MBadEncrypted` (`MsgTypeRenderers.tsx`): локальный стейт `requested`,
      кнопка «Запросить ключ» → «Запрошено» (disabled) после клика.

## 3. Проверка

- [x] 3.1 `npm run lint` и сборка/typecheck форка `cinny`, убедиться в отсутствии
      регрессий рендера обычных и нерасшифрованных сообщений.
