# Design: контракт ядра @harrier/core (фаза 0 эпика #59)

## Context

После фазы −1 десктопный клиент находится в `harrier-desktop/client`, но архитектура остаётся связанной с `matrix-js-sdk`. Прямые импорты есть в 232 файлах: `MatrixClient` используется как глобальный контекст, `Room` — как room context и модель настроек, а `MatrixEvent`/`EventTimeline` — как модель рендеринга. Есть обращения к внутренним subpath-модулям SDK, включая crypto и MatrixRTC. Поэтому простая замена импортов на re-export создала бы видимость шва, но не позволила бы подключить Rust.

Эпик #59 фиксирует API-first миграцию: сначала JS-адаптер и полный перевод React на view-модели, затем Rust за тем же интерфейсом. В фазе 0 сохраняются все текущие возможности JS-клиента. Tauri, системный показ уведомлений и отрисовка PNG трея остаются в JS; протокольная и бизнес-логика уходит за интерфейс ядра.

## Goals / Non-Goals

**Goals:**

- Определить независимый от движка модуль `@harrier/core` с типизированными командами, запросами и подписками.
- Скрыть SDK-объекты и внутренние handle в адаптере; React работает только с plain data view-моделями и идентификаторами.
- Перевести все функции клиента без пользовательских регрессий и запретить новые прямые импорты SDK.
- Проверять одинаковое поведение реализаций общим сьютом на sync-фикстурах.
- Сохранить работоспособность ветки после каждого этапа миграции.

**Non-Goals:**

- Rust crate, Tauri commands/Channels, `specta` и build-time переключатель движков (фазы 1+).
- Изменение пользовательского поведения, протокола, схемы локальных данных или дизайна UI.
- Миграция веб-legacy, удаление `matrix-js-sdk`, отказ от calls/threads.
- Re-export всего `matrix-js-sdk` или универсальный `getMatrixClient()` в качестве «контракта».

## Decisions

### D1. Контракт, JS-адаптер и контрактные тесты — три workspace-пакета

В `client/packages/` создаются:

- `core` (`@harrier/core`) — только типы, интерфейсы и чистые value-объекты; зависимости от Matrix SDK нет.
- `core-matrix-js` (`@harrier/core-matrix-js`) — фабрика движка, SDK-адаптер, mappers и SDK-specific test driver; единственная production-зона импортов `matrix-js-sdk`.
- `core-contract-tests` (`@harrier/core-contract-tests`) — независимый от реализации сьют, JSON-фикстуры и тип `CoreContractHarness`.

`client/package.json` становится private npm workspace с `packages/*`; Vite и TypeScript потребляют исходники через package exports, отдельный bundler не добавляется. Альтернатива — один пакет с подпутями — отвергнута: она оставляет SDK в зависимостях контракта и упрощает случайную утечку типов.

### D2. Единый HarrierCore с доменными модулями вместо универсального query/command

Внешний интерфейс — один `HarrierCore`, предоставляющий доменные модули `auth`, `rooms`, `timeline`, `send`, `crypto`, `media`, `presence`, `notifications` и опциональный `calls`. Каждый модуль скрывает нетривиальную реализацию за небольшим набором типизированных операций. Например, read receipt принимает `roomId` и `eventId`, а не `MatrixEvent`; отправка возвращает локальный `eventId` и дальнейший статус доступен в timeline.

Доменная форма выбрана вместо универсальных `query(name, params)`/`command(name, payload)`: последние уменьшают число методов лишь ценой потери типизации и переноса знания движка обратно в UI. Отдельные независимые repository без корня отвергнуты, потому что жизненный цикл сессии и согласованные снимки должны быть общими.

Протокольные string-ID, event type и JSON content остаются допустимыми: они принадлежат Matrix, а не SDK. Generic-операции для state/account data сохраняются для developer tools и редких событий, но возвращают view-модели, не `MatrixEvent`.

### D3. View-модели покрывают UI, а не повторяют иерархию классов SDK

Основные модели: `SessionView`, `SyncView`, `RoomSummaryView`, `RoomView`, `MemberView`, `TimelineView`, `TimelineItem`, `MessageContentView`, `RelationView`, `SendStatusView`, `DeviceView`, `VerificationView`, `KeyBackupView`, `MediaView`, `PresenceView`, `NotificationView` и `CallView`.

`TimelineItem` — discriminated union для message/state/encrypted/redacted/unsupported элементов. Он содержит стабильные `eventId`, `roomId`, sender, timestamp, content, relations, reactions, receipts и send/decryption status, необходимые текущему рендереру. Room содержит membership, space/direct flags, display data, permissions, unread и notification mode. Модели не содержат методов, EventEmitter, SDK-классов или `any`-handle.

### D4. Подписки построены как versioned snapshot stores

Каждый наблюдаемый набор данных предоставляет `getSnapshot()` и `subscribe(notify): Unsubscribe`, совместимые с `useSyncExternalStore` и Jotai bindings. Подписка не создаёт второй источник истины: адаптер владеет состоянием SDK и публикует новые immutable snapshots после SDK-событий.

Snapshot имеет монотонный `revision`; массивы и неизменённые элементы переиспользуются структурно. Это важно для списка комнат и крупных таймлайнов: полный snapshot не означает полную реаллокацию или ререндер всех элементов. Подписчик получает текущий snapshot немедленно через `getSnapshot`; уведомления приходят только после публикации нового revision. Отписка и закрытие сессии отменяют все SDK listeners.

Альтернатива — прокинуть SDK-подобные typed events (`RoomEvent`, `MatrixEventEvent`) — отвергнута: она сохраняет в UI знание порядка событий конкретного движка и усложняет будущий Tauri Channel.

### D5. JS-адаптер владеет SDK и внутренними handle

`createMatrixJsCore()` заменяет `initClient()` и инкапсулирует `MatrixClient`, IndexedDB stores, Rust crypto initialization и sync lifecycle. Mappers переводят SDK-модели в view-модели; commands находят внутренние `Room`/`MatrixEvent` по ID. Существующие чистые функции (unread, сортировка, relation handling, decryption failures) переносятся за интерфейс только если являются бизнес-логикой движка; форматирование и React-специфичные selectors остаются в UI.

Никакого public `rawClient`, `rawRoom`, `rawEvent` или общего escape hatch нет. Внутренний `MatrixClient` допустим только внутри `core-matrix-js`. Это обеспечивает deletion test: удаление адаптера не оставляет в UI протокольных handle, требующих замены.

### D6. React получает ядро через CoreProvider, room — через RoomView

`MatrixClientProvider`/`useMatrixClient` заменяются на `CoreProvider`/`useCore`; `ClientRoot` создаёт и запускает выбранную реализацию. `RoomProvider` передаёт `RoomView` либо стабильный `roomId` с подпиской на `RoomView`, а SDK `Room` исчезает из props и contexts.

Jotai остаётся UI-state слоем. Атомы больше не принимают `MatrixClient`; bindings подписываются на core snapshots. TanStack Query может оставаться для async UI-запросов, но query functions обращаются к core и не возвращают SDK-типы.

### D7. Calls — явная опциональная возможность, не постоянный JS-only обход

`HarrierCore.capabilities` описывает поддержку calls и других необязательных возможностей. JS-адаптер реализует calls, MatrixRTC и widget integration внутри своей зоны, поэтому фаза 0 сохраняет звонки. UI обязан корректно скрывать/отключать недоступную возможность; Rust-адаптер сможет не объявлять её до отдельной реализации. Threads остаются частью timeline/relations и обязательны в обеих реализациях при заявленной поддержке.

### D8. Общий сьют принимает harness реализации и одинаковые sync-фикстуры

`CoreContractHarness` определяет создание ядра, применение именованной sync-фикстуры, выполнение команд и чтение snapshots. Сьют не импортирует SDK. JS test driver создаёт SDK-модели через публичный API: `new Room(...)`, `client.store.storeRoom`, `room.updateMyMembership`, `room.currentState.setStateEvents` и `room.recalculate()`. Приватные методы SDK (`processSyncResponse`) и `SyncApi` (его конструктор помечен `@internal`) не используются. Вариант через `injectRoomEvents` отвергнут, так как требует `SyncApi`, который SDK не экспортирует как публичный конструктор.

Фикстуры покрывают initial/incremental sync, join/invite/leave, state и space hierarchy, live timeline и pagination, edit/reaction/redaction/reply/thread, own/remote receipts, sending status, presence/typing, unread/push rules, encrypted и decryption failure. Ожидаемые значения записаны в терминах контракта. Команды, которые нельзя проверить только sync, используют fake transport на стороне harness; assertions остаются общими.

### D9. Ошибки и время нормализованы

Публичные операции возвращают `CoreError` со стабильным `code`, человекочитаемым сообщением, `retryable` и optional protocol details; `MatrixError` не пересекает шов. Временные метки — milliseconds Unix, направления и статусы — собственные string unions. Это исключает зависимость от `Direction`, `EventStatus`, `SyncState`, `ReceiptType` и других SDK enums.

### D10. ESLint-гейт включается после миграции и действует на весь workspace

В `.eslintrc.cjs` `no-restricted-imports` запрещает `matrix-js-sdk` и `matrix-js-sdk/*` для клиента и пакетов; override разрешает их только в `packages/core-matrix-js/**`. Команда ESLint расширяется с `src/*` на `src` и `packages`, чтобы пакеты не обходили проверку. Временных per-file disables не добавляется. Дополнительная проверка в CI считает оставшиеся вхождения вне разрешённой зоны.

## Risks / Trade-offs

- [Контракт выродится в 1:1 копию SDK] → view-модели проектируются от текущих UI selectors; review запрещает raw handles и SDK enums в публичных типах.
- [Большой единовременный diff 232 файлов] → миграция идёт вертикальными доменными партиями, после каждой проходят typecheck/tests/build; lint-гейт включается последним.
- [Полные snapshots вызовут лаги на комнатах 10k+] → revision + structural sharing, pagination и selectors; замер открытия крупной комнаты до/после на эталонной комнате.
- [Тонкие различия событий SDK потеряются при mapping] → фикстуры для relations, redaction, threads, receipts и decryption; ручные smoke-сценарии на JS-движке.
- [Sync-фикстуры будут зависеть от внутренностей SDK] → только публичные injection API в test driver; общие ожидания не содержат SDK-объектов.
- [Calls/widget потребуют raw MatrixClient] → весь widget driver инкапсулируется в calls-адаптере; отсутствие capability обрабатывается UI, обходной rawClient не создаётся.
- [Три пакета усложнят npm/Vite-настройку] → npm workspaces без дополнительного bundler; ранние проверки clean `npm ci`, typecheck и production build.
- [Регрессии проявятся только в релизном Tauri] → финальные `npm run tauri build` и ручной smoke login/rooms/timeline/send/crypto/media/presence/notifications/calls.

## Migration Plan

1. Зафиксировать baseline и инвентарь SDK-импортов; создать workspaces и три пакета без изменения поведения.
2. Ввести базовые типы, snapshots, errors, lifecycle и contract-test harness; написать failing contract tests до реализации.
3. Реализовать auth/session/start/stop и перевести auth pages и `ClientRoot`; заменить корневой provider.
4. Перевести rooms/spaces/members/state/account data и Jotai bindings на snapshots.
5. Перевести timeline/relations/search/read markers и рендеринг на `TimelineItem`.
6. Перевести send/typing/receipts/uploads и локальные статусы.
7. Перевести crypto, media, presence и notifications; затем optional calls/widget.
8. Удалить оставшиеся SDK-импорты и compatibility-обёртки, включить запрет и общий сьют в обязательные проверки.
9. Выполнить полные автоматические и ручные проверки, обновить документацию пакета и issue #61.

Каждый этап остаётся локальным коммитом только после подтверждения пользователя. Rollback до публикации — откат соответствующей партии; изменения пользовательских stores и схемы сессии не требуются, поэтому откат не затрагивает данные.

## Open Questions

Нет. Допустимый поздний регресс calls/threads относится только к будущим Rust-фазам; в фазе 0 обе возможности сохраняются.
