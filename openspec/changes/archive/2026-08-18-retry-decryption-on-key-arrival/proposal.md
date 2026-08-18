## Why

SDK автоматически ретраит дешифровку событий из pending-list при появлении ключа
(`onRoomKeyUpdated` / `onRoomKeysWithheld`, `EventDecryptor.eventsPendingKey`). Но
в pending-list попадают только события, прошедшие через `attemptDecryption` после
инициализации крипто. События, расшифровка которых была предпринята до готовности
crypto, и таймлайн, загруженный из кэша офлайн, в список не попадают и «зависают»
нерасшифрованными, даже когда ключи уже пришли.

## What Changes

- Страховочный повторный проход расшифровки для нерасшифрованных событий вне
  pending-list SDK при появлении ключей.
- Подписка на события прихода ключей/бэкапа: `CryptoEvent.KeyBackupSessionsRemaining`
  (счётчик достиг 0), `KeyBackupDecryptionKeyCached`, `DevicesUpdated`,
  `UserTrustStatusChanged`.
- По событию — `decryptAllTimelineEvent` для текущей комнаты и комнат с событиями
  `isDecryptionFailure()`, без ручного скролла/перезагрузки.
- Guard: debounce, не блокировать UI, не ретраить необратимые причины
  (`MEGOLM_KEY_WITHHELD_FOR_UNVERIFIED_DEVICE` и др.).

## Capabilities

### New Capabilities

_Нет новых доменов._

### Modified Capabilities

- `messaging`: добавляется требование о повторной расшифровке нерасшифрованных
  событий при появлении ключей и исключение необратимых причин из ретраев.

## Impact

- `desktop/cinny`:
  - `src/app/utils/room.ts` — `decryptAllTimelineEvent` (существует), guard на
    необратимые причины расшифровки;
  - `src/app/features/room/RoomTimeline.tsx` — вызов при пагинации (существует);
  - новый хук на `CryptoEvent`-подписки по образцу `useKeyBackup.ts` /
    `useUserTrustStatusChange.ts`;
  - `src/app/pages/client/ClientNonUIFeatures.tsx` — подключение хука.
- matrix-js-sdk v41.7.0: `CryptoEvent.KeyBackupSessionsRemaining`,
  `KeyBackupDecryptionKeyCached`, `DevicesUpdated`, `UserTrustStatusChanged`.
- Никаких новых зависимостей.
