## Context

SDK автоматически ретраит дешифровку событий из pending-list при появлении ключа
(`onRoomKeyUpdated` / `onRoomKeysWithheld`, `EventDecryptor.eventsPendingKey`).
Но в pending-list попадают только события, прошедшие через `attemptDecryption`
после инициализации крипто. Кейсы вне покрытия:

- события, расшифровка которых была предпринята до готовности crypto;
- таймлайн, загруженный из кэша офлайн;
- события, чья попытка расшифровки завершилась до прихода ключа вне
  pending-list-механизма.

Сейчас `decryptAllTimelineEvent` (`src/app/utils/room.ts:394`) вызывается только
при пагинации (`RoomTimeline.tsx:355`). Нерасшифрованные события «оживают» только
при ручном скролле/перезагрузке.

## Goals / Non-Goals

**Goals:**
- Страховочный повторный проход расшифровки для нерасшифрованных событий вне
  pending-list SDK при появлении ключей.
- Триггеры: `CryptoEvent.KeyBackupSessionsRemaining` (счётчик достиг 0),
  `KeyBackupDecryptionKeyCached`, `DevicesUpdated`, `UserTrustStatusChanged`.
- Guard от необратимых причин (`MEGOLM_KEY_WITHHELD_FOR_UNVERIFIED_DEVICE` и др.)
  и от гонки ретраев (debounce).

**Non-Goals:**
- Изменение логики pending-list SDK.
- Принудительный ретрай событий с необратимыми причинами.
- Замена штатного механизма ретраев SDK (наш проход — страховочный).

## Decisions

### 1. Утилита повторной расшифровки

`src/app/utils/room.ts` — новая функция `retryFailedTimelineEvents(mx, roomId?)`:
- для каждой комнаты (или заданной): пройтись по `room.getLiveTimeline().getEvents()`
  (или live-событиям всех `timelineSets`), отобрать события
  `mEvent.isEncryptionFailure()` с **обратимой** причиной;
- вызвать `mEvent.attemptDecryption(crypto, { isRetry: true })` для отобранных,
  `Promise.allSettled`;
- **обратимые причины** (`isRetryableDecryptionFailureReason`): только
  `MEGOLM_UNKNOWN_INBOUND_SESSION_ID` и `HISTORICAL_MESSAGE_WORKING_BACKUP` —
  остальные исключаются из ретраев (необратимы: `MEGOLM_KEY_WITHHELD_*`,
  `SENDER_IDENTITY_PREVIOUSLY_VERIFIED`, `UNSIGNED_SENDER_DEVICE`, ...).

### 2. Хук `useRetryDecryptionOnKeyArrival`

Новый хук (в `src/app/hooks/`), подписывается на:
- `KeyBackupSessionsRemaining` — при `count === 0` (восстановление бэкапа
  закончилось);
- `KeyBackupDecryptionKeyCached` — ключ бэкапа появился;
- `DevicesUpdated` — устройства (и их ключи) изменились;
- `UserTrustStatusChanged` — доверие к пользователю изменилось (могли появиться
  недостающие ключи).

На каждое событие — debounced вызов `retryFailedTimelineEvents(mx)` (один вызов
за ~500 мс, сбрасываемый таймер), плюс флаг «не запускать повторно, пока идёт»
(ReentrancyGuard).

Подключается в `ClientNonUIFeatures.tsx`.

### 3. Первый проход после синхронизации

При `ClientEvent.Sync === 'SYNCING'` (как `CustomStatusRestore`) один раз
выполнить `retryFailedTimelineEvents` — покрывает кейс «таймлайн из кэша офлайн»,
когда крипто ещё не готово на момент расшифровки.

## Risks / Trade-offs

- **Производительность**: массовый `attemptDecryption` на больших лентах.
  Смягчение: отбор только событий `isEncryptionFailure()` с обратимой причиной +
  debounce + ReentrancyGuard.
- **Конкуренция с pending-list SDK**: повторный `attemptDecryption` для событий,
  уже стоящих в pending-list, безвреден — `MatrixEvent.attemptDecryption` сам
  координирует повторные вызовы (`decryptionPromise` + `retryDecryption`).
- **Задержка «оживления»** сообщений: до ~1 с после события ключа (debounce) —
  приемлемо.
