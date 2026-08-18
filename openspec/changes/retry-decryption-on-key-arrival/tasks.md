# Tasks

## 1. Утилита повторной расшифровки

- [x] 1.1 `src/app/utils/decryptionFailure.ts`: добавить
      `isRetryableDecryptionFailureReason(code)` — только
      `MEGOLM_UNKNOWN_INBOUND_SESSION_ID` и `HISTORICAL_MESSAGE_WORKING_BACKUP`.
- [x] 1.2 `src/app/utils/room.ts`: новая функция `retryFailedTimelineEvents(mx, roomId?)` —
      отобрать события `isDecryptionFailure()` с обратимой причиной по
      live-таймлайнам комнат и вызвать `attemptDecryption(crypto, { isRetry: true })`,
      `Promise.allSettled`.

## 2. Хук на приход ключей

- [x] 2.1 Новый хук `useRetryDecryptionOnKeyArrival` (в `src/app/hooks/`):
      подписки `KeyBackupSessionsRemaining` (count === 0), `KeyBackupDecryptionKeyCached`,
      `DevicesUpdated`, `UserTrustStatusChanged` → debounced (500 мс)
      `retryFailedTimelineEvents(mx)` с ReentrancyGuard.
- [x] 2.2 Первый проход при `ClientEvent.Sync === 'SYNCING'` (один раз) — покрывает
      кейс «таймлайн из кэша офлайн».
- [x] 2.3 Подключить хук в `ClientNonUIFeatures.tsx`.

## 3. Проверка

- [x] 3.1 `npm run lint` и сборка/typecheck форка `cinny`, убедиться в отсутствии
      регрессий расшифровки live-сообщений и пагинации.
