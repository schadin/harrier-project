## Why

В matrix-js-sdk v41.7.0 (Rust crypto) автоматические запросы ключей отключены
(`olmMachine.roomKeyRequestsEnabled = false`, per element-web#26524), а публичного
API `requestRoomKey` в `CryptoApi` нет. Поэтому нерасшифрованное live-сообщение
может «висеть» неопределённо долго: ключ не запрашивается автоматически, и
сообщение не расшифровывается до ручного действия (скролл/перезагрузка — и то не
всегда). Пользователь не может инициировать получение ключа из интерфейса.

## What Changes

- Кнопка «Запросить ключ» на нерасшифрованном сообщении для ретраябельных причин
  (`MEGOLM_UNKNOWN_INBOUND_SESSION_ID`, `HISTORICAL_MESSAGE_WORKING_BACKUP`).
- Отправка to-device `m.room_key_request` через `mEvent.requestKey()`:
  - content `{ action: "request", request_id, body: { room_id, sender_key, session_id } }`;
  - `session_id` / `sender_key` — из `event.getWireContent()`;
  - получатели — устройства отправителя.
- Rate-limit/дедуп по `request_id`, чтобы исключить «шторм запросов» (регрессия из
  element-web#26524).
- Индикация состояния запроса (отправлен / ключ получен). Ответ отправителя SDK
  импортирует сам и автоматически ретраит дешифровку ожидающих событий.

## Capabilities

### New Capabilities

_Нет новых доменов._

### Modified Capabilities

- `messaging`: добавляется требование о ручном запросе недостающего ключа сессии
  для нерасшифрованного сообщения и ограничении частоты таких запросов.

## Impact

- `desktop/cinny`:
  - `src/app/features/room/RoomTimeline.tsx` — проброс `onRequestKey` в
    `RenderMessageContent`/`MBadEncrypted`;
  - `src/app/components/message/MsgTypeRenderers.tsx`, `decryptionFailure.ts` —
    задействование существующего действия `requestKey`;
  - новый хук/утилита для отправки `m.room_key_request` с дедупом по
    `request_id`;
- matrix-js-sdk v41.7.0: `MatrixEvent.requestKey()`, `event.getWireContent()`,
  `mEvent.getKeyRequestRecipients(userId)` (устройства отправителя).
- Никаких новых зависимостей. Фича строится только на механизмах протокола
  (to-device `m.room_key_request`).
