# Tasks: контракт ядра @harrier/core (фаза 0 эпика #59)

## 1. Baseline и workspace-пакеты

- [x] 1.1 Зафиксировать baseline `npm test`, typecheck, ESLint, production build и `cargo test`; сохранить инвентарь из 232 SDK-зависимых файлов для контроля миграции
- [x] 1.2 Настроить private npm workspace `packages/*` в `desktop/client` и проверить чистый `npm ci` без изменения версии приложения
- [x] 1.3 Создать `@harrier/core` с package exports и отдельным tsconfig; добавить smoke-test отсутствия зависимости от SDK, React и Tauri
- [x] 1.4 Создать `@harrier/core-matrix-js` и `@harrier/core-contract-tests`; подключить их исходники к Vitest, typecheck, Prettier и Vite без отдельного bundler

## 2. Базовый контракт

- [x] 2.1 Через failing type/runtime tests ввести `CoreError`, protocol JSON types, ID/string unions и нормализацию SDK-независимых ошибок
- [ ] 2.2 Через tests реализовать generic `SnapshotStore<T>` с revision, cached `getSnapshot`, subscribe/unsubscribe и structural sharing (сделано: revision, cache, subscribe/unsubscribe, пропуск одинаковых значений; структурное sharing вложенных элементов остаётся за адаптером, см. задачи 5–6)
- [x] 2.3 Определить корневой `HarrierCore`, lifecycle и `CoreCapabilities`; проверить отсутствие raw-client/raw-room/raw-event в публичных типах
- [x] 2.4 Определить базовые `SessionView`, `SyncView`, `RoomSummaryView`, `RoomView`, `MemberView`, `TimelineView`, `TimelineItem` и `SendStatusView` по потребностям текущих selectors
- [x] 2.5 Добавить compile-only consumer test: потребитель использует `@harrier/core` без импорта реализации и SDK-типов

## 3. Контрактный harness и sync-фикстуры

- [x] 3.1 Определить `CoreContractHarness`, регистратор общего Vitest-сьюта и нормализованные assertions, не импортирующие SDK
- [x] 3.2 Реализовать JS test driver на публичном API SDK (`Room`, `store.storeRoom`, `currentState`, `recalculate`) без private `processSyncResponse` и `SyncApi`; fake transport добавить вместе с командами (задачи 7.x)
- [x] 3.3 Добавить failing fixtures/tests для initial+incremental sync, join/invite/leave, room state, direct/space flags и hierarchy (initial — первое применение фикстуры, incremental — сохранение не упомянутых комнат и обновление имени; покрыто тестами)
- [x] 3.4 Добавить failing fixtures/tests для live timeline, pagination, reply/edit/reaction/redaction/thread и собственных/чужих receipts
- [x] 3.5 Добавить failing fixtures/tests для sending status, presence/typing, unread/push rules, encrypted/decryption failure и notification preview

## 4. Auth и корневой lifecycle

- [x] 4.1 Реализовать в `createMatrixJsCore()` инициализацию MatrixClient, IndexedDB stores, Rust crypto и обработку ошибок до прохождения auth contract tests (сделано через createMatrixJsClient; auth.start вызывается в ClientRoot)
- [ ] 4.2 Перенести server discovery, capabilities/media config, login/token/SSO и auth metadata в `auth`-модуль; перевести соответствующие pages/loaders
- [ ] 4.3 Перенести registration, UIA stages, password reset и email flows в типизированные auth-операции; перевести auth pages без `MatrixError`/SDK enums
- [ ] 4.4 Перенести session restore, start/stop/retry sync, session-logged-out, logout, clear cache/login data в ядро и покрыть lifecycle tests (сделано: start/stop/logout, session и sync snapshots, prepared/error, fake logout; не начато: restore, retry, session-logged-out, clear cache/login data)
- [ ] 4.5 Заменить `MatrixClientProvider` на `CoreProvider`, перевести `ClientRoot` и `useMatrixClient`-потребителей текущей партии; проверить login/start/logout smoke (сделано: CoreProvider добавлен, sync запускается через core.auth.start, production build зелёный; MatrixClientProvider пока сохранён для остальных потребителей)

## 5. Rooms, spaces, members и state

- [x] 5.1 Реализовать room/invite snapshots и membership updates до прохождения соответствующих sync contract tests
- [ ] 5.2 Перевести Jotai room-list/invite/unread bindings, `useGetRoom`, `useRoom`, `RoomProvider` и sidebar/room-nav на `RoomView` и core subscriptions (сделано: rooms.all и rooms.unreads, room-list, invite, roomToParents и roomToUnread bindings, useCoreRoom, useCoreRoomMeta, useGetCoreRoom, CallRoomName, useDirectUsers, палитра, Search, AddExisting, LobbyHero, LobbyHeader, RoomIntro (метаданные), Space header, RoomSettings и SpaceSettings headers, RoomNavItem (имя), RoomViewHeader, CallEmbedProvider, RoomProfile; useRoomMeta удалён; остались useGetRoom, useRoom, RoomProvider и прочий sidebar/room-nav)
- [x] 5.3 Реализовать room state/account data, permissions, members/profiles, direct/parent/children projections; перевести связанные hooks и utils (сделано: RoomView с topic/encryption, members snapshots, getState и children, permissions-проекции rooms.powerLevels/rooms.creators, чистая логика power levels в @harrier/core, хуки usePowerLevels/useRoomsPowerLevels/useRoomPermissions/useRoomCreators на core-снапшотах, rooms.accountData и миграция useRoomAccountData, rooms.parents с cycle-guard в ядре и перевод binding roomToParentsAtom, расширенный MemberView (avatarMxc/powerLevel/rawDisplayName/joinedAt/leftBySelf), логика сортировки/фильтров/поиска members в @harrier/core, миграция useRoomMembers и members-UI (MembersDrawer, Members, MemberTile, UserMentionAutocomplete, Room, Lobby); state-event хуки страниц настроек остаются для 5.5, глобальный account data — для 9.4)
- [x] 5.4 Перевести create/join/leave/invite, create room/chat/space, room directory и hierarchy на команды `rooms` (сделано: команды rooms.join/leave/invite, rooms.createRoom (тип, доступ с knock, шифрование, алиас, creators, parent+via, m.space.child родителю), rooms.searchDirectory (включая room_types-фильтр вместо raw HTTP), rooms.hierarchy (пагинация) и перевод LeaveRoomPrompt, RoomCard, InviteUserPrompt, CreateRoom, CreateSpace, Explore/Server, useFetchSpaceHierarchyLevel, HierarchyRoomSummaryLoader, lobby-компоненты)
- [x] 5.5 Перевести common/room/space settings, developer tools, image packs и account-data editors без generic raw-event escape hatch (сделано: rooms.state и rooms.sendState, хук useCoreRoomState и хелпер sendRoomState; мигрированы RoomEncryption, RoomHistoryVisibility, RoomPublish, usePublishedAliases, useRoomPinnedEvents, usePowerLevelTags, tombstone в RoomView и Space, join rules в Space, encrypted в RoomViewHeader; write-path через rooms.sendState в 15 файлах — mx.sendStateEvent вне адаптера остался только в call-плагине (10.x); rooms.upgrade (RoomUpgradeDialog без raw HTTP) и read-path RoomUpgrade через useCoreRoomState; rooms.stateList (RoomStateEntry: stateKey/content/eventId) с контракт-тестом и фикстурой stateEvents — image packs read (useRoomImagePack/useRoomImagePacks/useRoomsImagePacks) на core-хуках; rooms.setAccountData и editor room account data в DevelopTools; user/global image packs и прочие global account-data writers остаются для 9.4)
- [x] 5.6 Перенести чистую бизнес-логику room/unread/sort за ядро, оставив React formatting/selectors в UI; прогнать существующие tests и room contract tests (сделано: модуль @harrier/core/roomList — компараторы имён/активности/непрочитанных с инжектируемым getMeta-аксессором (factoryRoomIdSort/atoz/ztoa/activity), byOrderKey, byTsOldToNew, compareRoomIdsEqual и чистая агрегация непрочитанных (putUnreadInfo/deleteUnreadInfo/unreadEqual/unreadInfoToUnread, иммутабельные Map-in/Map-out вместо immer) с 16 юнит-тестами; UI: utils/sort.ts — тонкие обёртки над core с MatrixClient-аксессором, roomToUnreadAtom без produce, compareRoomsEqual из core)

## 6. Timeline, relations и search

- [x] 6.1 Реализовать SDK→`TimelineItem` mappers и cached timeline snapshots до прохождения базовых timeline contract tests
- [x] 6.2 Реализовать live/paginated timelines, linked timeline projection, event-by-id и jump-to-event без `EventTimeline`/`Direction` в UI (сделано: buildTimeline теперь проекция связанной цепочки таймлайнов (firstLinked→forward), hasBackwardHistory от токена первого звена; TimelineModule += focusEvent(roomId, eventId): Promise<CoreResult<number>> — getEventTimeline + последовательная backward-пагинация цепочки до нахождения, абсолютный индекс; clearFocus; TimelineView += focusedEventId; пагинация при фокусе идёт по цепочке фокуса; драйвер: timelineSupport в createClient и Room, /context fake (knownEvents по messages+history); контракт-тесты: focus по загруженному событию, focus через context-транспорт, clearFocus, not-found; UI-хуки src/app/hooks/useCoreTimeline.ts: useCoreTimeline/paginateCoreTimelineBackward/focusCoreEvent/clearCoreTimelineFocus — перенос RoomTimeline внутрь этих хуков остаётся для 6.4)
- [x] 6.3 Реализовать reply/edit/reaction/redaction/thread/relations и receipts до прохождения relation contract tests
- [ ] 6.4 Перевести `RoomTimeline`, message/state/encrypted renderers, reactions, readers, pinned events и room view на view-модели (сделано: источник данных RoomTimeline переведён на core — useCoreTimeline, paginateBackward/Forward по hasBackwardHistory, focusEvent/clearFocus для jump-to-event и переходов к непрочитанным, unreadInfo из TimelineView.readUpTo/focusedEventId, up-arrow edit по TimelineItem, eventId-prop через focusCoreEvent; TimelineModule += paginateForward (forward-пагинация последнего звена цепочки фокуса/live, no-op при отсутствии токена), TimelineItem += kind sticker; диспетчер рендереров переведён на TimelineItem.kind (itemTimelineType/isState по kind), hasReactions и readReceiptUsers — из item.reactions/item.readBy, collapsed/dividers — по sender/timestamp/item; рендереры пока остаются на MatrixEvent через room.findEventById, перевод Message/Reactions/readers/pinned на TimelineItem — остаток задачи; следующая итерация: MessageContentView += formattedBody/format/attachment (image/video/audio/file/location из url+info), TimelineItem.message += edited, LocalEchoUpdated репаблишит таймлайн (статус отправки из item свежий), Message получает проп timelineItem и берёт sender/timestamp/status/threadRoot/edited из него, MessageStatus переведён на SendStatus+readBy без подписки на MatrixEvent; контракт-тесты: formatted body, edit formatted body, attachment metadata; Reactions/ReactionViewer/ReactionTooltipMsg переведены на ReactionCountView (ядро отдаёт senders и shortcode по каждому ключу), Reply рендерит replyItem из TimelineItem с fallback на старый путь для поиска/пинов/уведомлений; остаток: Event-рендерер, raw content в item, остальные вызовы Reply)
- [ ] 6.5 Перевести timeline visibility, member event parsing, custom HTML/message content и DsTaskBot helpers на типизированный content без `MatrixEvent`
- [ ] 6.6 Перевести message search, relations queries, local summaries и room latest event на core; проверить 10k+ timeline performance и structural sharing

## 7. Send, typing, receipts и uploads

- [ ] 7.1 Реализовать send message/generic event/edit/reaction/redaction с нормализованными локальными ID, statuses и `CoreError`; прогнать command contract tests
- [ ] 7.2 Перевести `RoomInput`, editor/output, commands, message editor, `sendMessageSafely`/`sendEventSafely` и send-error UX на `send`
- [ ] 7.3 Реализовать read receipts/read markers/typing через ID-based команды и перевести hooks/utils; проверить private receipt и unread updates
- [ ] 7.4 Реализовать upload/progress и attachment integration совместно с media; перевести upload state/cards без SDK response types

## 8. Crypto

- [ ] 8.1 Реализовать device list/trust, own/remote verification status и device commands; перевести device settings и status components
- [ ] 8.2 Инкапсулировать verification request/SAS/verifier lifecycle в `crypto` snapshots/commands; перевести verification UI без CryptoApi и SDK event maps
- [ ] 8.3 Перенести secret storage, recovery key, backup info/restore/import progress и auto-restore в ядро; перевести BackupRestore/SecretStorage и hooks
- [ ] 8.4 Перенести room key requests и retry-decryption-on-key-arrival; перевести encrypted content и decryption failure actions
- [ ] 8.5 Прогнать crypto contract fixtures и ручной smoke verification, backup/restore, key request и decryption recovery

## 9. Media, presence и notifications

- [ ] 9.1 Реализовать MXC/authenticated media, thumbnails, download/decrypt attachments, URL previews и media config в `media`; перевести media components/hooks
- [ ] 9.2 Реализовать presence/typing/custom status snapshots и команды; перевести presence UI, user profiles и typing hooks
- [ ] 9.3 Перенести custom status restore/retry и tray-status command handling на `presence`, сохранив desktop integration в JS
- [ ] 9.4 Реализовать unread/highlight, notification mode/push rules и notification previews в `notifications`; перевести state/hooks/settings
- [ ] 9.5 Перевести inbox notifications/invites и `ClientNonUIFeatures` на `NotificationView`; оставить только системный показ, звук, favicon и PNG трея в UI
- [ ] 9.6 Прогнать presence/unread/notification contract tests и ручной smoke desktop notifications, private rooms, E2E preview и custom status

## 10. Calls и MatrixRTC

- [ ] 10.1 Определить optional calls capability и `CallView`/commands для текущих MatrixRTC и call-status сценариев; добавить tests для available/unavailable capability
- [ ] 10.2 Инкапсулировать MatrixRTCSession, membership и widget driver/OpenID внутри `core-matrix-js`; перевести call plugins и `CallEmbedProvider`
- [ ] 10.3 Перевести call/call-status UI и LiveKit support на calls-интерфейс; выполнить smoke входа/выхода и отображения участников звонка

## 11. Полное удаление SDK-зависимостей вне адаптера

- [ ] 11.1 Сверить исходный инвентарь 232 файлов с текущим поиском и перевести оставшиеся tests, types, plugins и compatibility-обёртки
- [ ] 11.2 Удалить `MatrixClient`/`Room`/`MatrixEvent` из props, contexts, Jotai/TanStack results и публичных UI utils; проверить отсутствие raw handles и SDK enums
- [ ] 11.3 Включить `no-restricted-imports` для `matrix-js-sdk` и `matrix-js-sdk/*` с единственным override `packages/core-matrix-js/**`; расширить ESLint на `src` и `packages`
- [ ] 11.4 Добавить отдельный CI/локальный поиск запрещённых вхождений и проверить, что случайный импорт вне адаптера даёт ошибку, а внутри адаптера — нет

## 12. Финальная верификация и документация

- [ ] 12.1 Прогнать полный `core-contract-tests` против JS-адаптера и все существующие Vitest-тесты; устранить регрессии без ослабления assertions
- [ ] 12.2 Прогнать `npm run lint`, typecheck и production build в `desktop/client`; добиться 0 errors и не добавить новых предупреждений
- [ ] 12.3 Прогнать `cargo test` и `npm run tauri build`; проверить release binary и отсутствие изменений версии
- [ ] 12.4 Выполнить ручной desktop smoke login/logout, rooms/spaces, timeline/threads, send/edit/reactions, crypto, media, presence, notifications и calls
- [ ] 12.5 Обновить README пакетов и developer docs: назначение шва, создание адаптера, запуск contract suite и запрет raw SDK; согласовать с пользователем необходимость записи в основной README
- [ ] 12.6 Обновить issue #61 итогами, проверками и оставшимися ограничениями; провести review change и подготовить к архивации только после подтверждения пользователя
