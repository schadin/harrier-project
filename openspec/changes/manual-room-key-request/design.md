## Context

В matrix-js-sdk v41.7.0 (Rust crypto) автоматические запросы ключей отключены:
`olmMachine.roomKeyRequestsEnabled = false` (per element-web#26524), публичного
API `requestRoomKey` в `CryptoApi` нет. Поэтому нерасшифрованное live-сообщение
«висит» неопределённо долго.

Инфраструктура UI для причины расшифровки и действия уже готова:
- `getDecryptionFailureReason` (`src/app/utils/decryptionFailure.ts`) возвращает
  `action: 'requestKey'` для `MEGOLM_UNKNOWN_INBOUND_SESSION_ID`;
- `MBadEncrypted` (`MsgTypeRenderers.tsx:38`) рендерит кнопку, если передан
  `onRequestKey`;
- но `onRequestKey` **нигде не проброшен** — ни в `RoomTimeline.tsx:1229`,
  ни в `SearchResultGroup`, ни в `Notifications`.

Ответ отправителя (forwarded key) SDK импортирует сам и автоматически ретраит
дешифровку ожидающих событий (`onRoomKeyUpdated`, pending-list в `EventDecryptor`) —
достаточно отправить запрос, остальное SDK делает сам.

## Goals / Non-Goals

**Goals:**
- Возможность вручную запросить недостающий ключ сессии для нерасшифрованного
  сообщения через to-device `m.room_key_request`.
- Дедупликация повторных запросов, чтобы не создать «шторм запросов».
- Индикация состояния запроса на сообщении.

**Non-Goals:**
- Автоматические запросы ключей (остаются отключёнными).
- Обработка входящих `m.room_key_request` на стороне отправителя (делает SDK).
- Запрос ключей для необратимых причин (`MEGOLM_KEY_WITHHELD_FOR_UNVERIFIED_DEVICE`
  и др.) — действие «Запросить ключ» для них не показывается.

## Decisions

### 1. Отправка `m.room_key_request` через `mx.sendToDevice`

Утилита `requestRoomKey(mx, mEvent)`:
- wire content: `const { algorithm, room_id, sender_key, session_id } = mEvent.getWireContent()` —
  поля `room_id`, `sender_key`, `session_id` могут отсутствовать → не отправляем, если нет.
- получатели: `mEvent.getKeyRequestRecipients(senderId)` — все устройства отправителя
  события (передаём `mEvent.getSender()`, SDK формирует `[{ userId, deviceId: "*" }]`).
- content для каждого получателя:
  ```
  {
    action: 'request',
    request_id: <UUID>,
    body: { room_id, sender_key, session_id, algorithm },
  }
  ```
- `mx.sendToDevice(EventType.RoomKeyRequest, contentMap)`.

### 2. Дедупликация запросов

Module-level `Map<eventId, timestamp>` (`keyRequestSentAt`). Повторный запрос для
одного события в течение `KEY_REQUEST_COOLDOWN_MS` (60 c) игнорируется. Этого
достаточно против «шторма» при повторных кликах/рендерах.

### 3. UI: кнопка и индикация состояния

- `MBadEncrypted` — полноценный компонент: локальный `useState` `requested`.
  При клике: `setRequested(true)`, вызов `onRequestKey()`. Кнопка меняется на
  «Запрошено» и становится неактивной.
- `onRequestKey` пробрасывается из `RoomTimeline.tsx` (место, где уже передаются
  `onVerifyDevice` / `onRestoreBackup`):
  ```
  onRequestKey={() => requestRoomKey(mx, mEvent)}
  ```
- Успешную расшифровку компонент увидит автоматически: `EncryptedContent`
  подписан на `MatrixEventEvent.Decrypted` и форсирует ре-рендер.

## Risks / Trade-offs

- **Ответ отправителя не гарантирован**: отправитель может быть офлайн или не
  иметь ключа. Это ограничение протокола — сообщение останется нерасшифрованным
  до получения ключа иным способом (бэкап).
- **`deviceId: "*"`** — запрос уходит всем устройствам отправителя; избыточность
  приемлема и соответствует поведению SDK (`getKeyRequestRecipients`).
- **Локальный стейт кнопки** сбрасывается при ре-рендере списка, если
  `MBadEncrypted` будет пересоздан — состояние «запрошено» держим в cooldown-map,
  чтобы повторный запрос не улетел.
