# Core Contract Specification

## ADDED Requirements

### Requirement: Независимый от движка интерфейс ядра

Система SHALL предоставлять пакет `@harrier/core` с единым интерфейсом `HarrierCore` и доменными модулями auth, rooms, timeline, send, crypto, media, presence и notifications. Публичные типы контракта MUST NOT зависеть от `matrix-js-sdk`, Tauri или React.

#### Scenario: Потребление контракта без движка

- **WHEN** TypeScript-потребитель импортирует только `@harrier/core`
- **THEN** ему доступны все команды, view-модели и подписки без установки или импорта реализации Matrix SDK

#### Scenario: Запрет утечки реализации

- **WHEN** проверяются публичные типы и экспортируемые значения контракта
- **THEN** в них отсутствуют `MatrixClient`, `Room`, `MatrixEvent`, SDK event maps, SDK enums и внутренние subpath-импорты SDK

### Requirement: Заменяемые реализации ядра

Система SHALL допускать создание независимых реализаций `HarrierCore` за одним интерфейсом. React MUST обращаться к движку через `CoreProvider` и MUST NOT получать реализацию Matrix SDK напрямую.

#### Scenario: Замена движка без изменения потребителя

- **WHEN** одна реализация `HarrierCore` заменяется другой в `CoreProvider`
- **THEN** существующие hooks, state bindings и UI продолжают использовать те же типизированные операции и snapshots без изменения импортов SDK

#### Scenario: Отсутствие обходного доступа к SDK

- **WHEN** UI требуется выполнить операцию над комнатой или событием
- **THEN** он использует команду ядра с идентификаторами и данными, а public raw-client/raw-room/raw-event доступ отсутствует

### Requirement: Наблюдаемые immutable snapshots

Ядро SHALL публиковать наблюдаемое состояние как snapshots с монотонным revision и операциями `getSnapshot`/`subscribe`. Опубликованный snapshot MUST NOT изменяться реализацией после публикации, а отписка MUST прекращать уведомления.

#### Scenario: Первичное чтение и обновление

- **WHEN** потребитель читает текущий snapshot и затем движок обрабатывает новое состояние
- **THEN** потребитель получает уведомление и следующий snapshot имеет больший revision и обновлённые view-модели

#### Scenario: Отписка и структурное сохранение

- **WHEN** обновление затрагивает один элемент, а другой потребитель отписывается
- **THEN** неизменённые элементы сохраняют ссылочную идентичность, а отписанный потребитель больше не получает уведомлений

### Requirement: Аутентификация и жизненный цикл сессии

Модуль auth SHALL поддерживать используемые клиентом server discovery, login, registration/UIA, SSO, восстановление сессии, start/stop sync, logout и очистку локальной сессии через типы контракта.

#### Scenario: Восстановление и запуск сессии

- **WHEN** сохранённые session data переданы реализации ядра и выполнен запуск
- **THEN** auth публикует состояние готовности, а rooms и sync snapshots начинают отражать данные движка

#### Scenario: Завершение сессии

- **WHEN** пользователь выполняет logout или сервер отзывает сессию
- **THEN** sync останавливается, подписки завершаются, локальные данные сессии очищаются согласно команде, а SDK-операции не требуются от UI

### Requirement: View-модели комнат, пространств и участников

Модуль rooms SHALL предоставлять snapshots и команды для списков комнат, invites, membership, room state, permissions, direct/space flags, иерархии, участников, account data и настроек комнат, необходимых текущему клиенту.

#### Scenario: Изменение membership

- **WHEN** sync переводит комнату из invite в join
- **THEN** соответствующие snapshots обновляют membership и списки, а `RoomSummaryView` сохраняет стабильный `roomId`

#### Scenario: Чтение и изменение room state

- **WHEN** UI читает или изменяет поддерживаемое state/account-data событие
- **THEN** операция использует protocol event type, state key и JSON content, а результат отражается в `RoomView` без передачи `MatrixEvent`

### Requirement: View-модель таймлайна и пагинация

Модуль timeline SHALL предоставлять упорядоченные `TimelineItem` view-модели, live updates, pagination, relations, replies, edits, reactions, redactions, threads, read markers и message search. `TimelineItem` MUST содержать данные для рендеринга и MUST NOT содержать SDK event/timeline objects.

#### Scenario: Преобразование sync в элементы таймлайна

- **WHEN** harness применяет фикстуру с сообщениями, state и encrypted-событиями
- **THEN** timeline публикует упорядоченные элементы со стабильными `eventId`, sender, timestamp, типизированным content и соответствующим decryption/state представлением

#### Scenario: Связанные события и пагинация

- **WHEN** применяются edit, reaction, reply, thread и последующая страница истории
- **THEN** relations и элементы обновляются без дубликатов, а границы доступной истории отражены в `TimelineView`

### Requirement: Отправка и статусы команд

Модуль send SHALL поддерживать сообщения, generic events, edit, reaction, redaction, receipts, typing и upload-related команды текущего клиента. Реализация SHALL сообщать результат через `CoreError` и/или наблюдаемый `SendStatusView`.

#### Scenario: Локальная отправка и подтверждение

- **WHEN** пользователь отправляет сообщение и движок сначала создаёт локальный элемент, затем подтверждает его
- **THEN** UI наблюдает отправку и итоговый статус по стабильному идентификатору без обращения к `EventStatus` SDK

#### Scenario: Ошибка отправки

- **WHEN** транспорт отклоняет команду отправки
- **THEN** существующий механизм уведомления об ошибке получает нормализованное сообщение, а failed status доступен в таймлайне

### Requirement: Криптографические сценарии

Модуль crypto SHALL покрывать устройства и их доверие, verification, secret storage, key backup/restore, room key requests, decryption retry и представление decryption failures, необходимые текущему крипто-UX.

#### Scenario: Верификация и восстановление ключей

- **WHEN** пользователь проходит verification или восстанавливает backup
- **THEN** crypto snapshots отражают текущую фазу и доверие, а зашифрованные элементы обновляются после получения ключей без SDK crypto handles в UI

#### Scenario: Невозможность расшифровки

- **WHEN** событие не может быть расшифровано
- **THEN** timeline содержит стабильную причину failure и доступные действия recovery в терминах контракта

### Requirement: Медиа-операции

Модуль media SHALL предоставлять download/authenticated media, thumbnails, upload progress, URL previews и преобразование MXC-ссылок в данные, необходимые UI, без возврата SDK responses.

#### Scenario: Загрузка вложения

- **WHEN** пользователь загружает файл и движок сообщает прогресс
- **THEN** UI получает нормализованный прогресс и итоговый media reference для последующей отправки

### Requirement: Присутствие и пользовательский статус

Модуль presence SHALL предоставлять presence/typing snapshots и команды для online presence и custom status, включая используемое клиентом восстановление статуса.

#### Scenario: Обновление presence и custom status

- **WHEN** sync изменяет presence пользователя или пользователь устанавливает custom status
- **THEN** подписчики получают `PresenceView` с нормализованными состоянием, текстом и emoji без SDK presence enums

### Requirement: Уведомления и непрочитанные события

Модуль notifications SHALL вычислять unread/highlight, notification mode/push rules, invite/message notification view-модели и previews. Системный показ уведомления MAY выполняться UI-интеграцией, но решение и данные уведомления MUST приходить через контракт.

#### Scenario: Новое сообщение с учётом push rules

- **WHEN** incremental sync добавляет входящее сообщение в комнату с заданным notification mode
- **THEN** unread snapshot и `NotificationView` обновляются согласно push rules, а preview соответствует типу и redaction/decryption состоянию сообщения

#### Scenario: Прочтение комнаты

- **WHEN** пользователь отмечает комнату прочитанной
- **THEN** read receipt отправляется через ядро и unread/highlight счётчики обновляются без вычисления raw timeline в UI

### Requirement: Опциональные возможности движка

Ядро SHALL объявлять поддержку необязательных возможностей через capabilities. JS-реализация фазы 0 MUST сохранять текущие calls и threads; UI MUST обрабатывать отсутствие объявленной необязательной возможности без raw-доступа к движку.

#### Scenario: Calls доступны

- **WHEN** JS-адаптер объявляет calls
- **THEN** существующие call/MatrixRTC/widget сценарии используют calls-интерфейс и продолжают работать без импорта SDK в UI

#### Scenario: Calls недоступны

- **WHEN** другая реализация не объявляет calls
- **THEN** UI отключает соответствующие действия, а остальные домены ядра остаются доступными

### Requirement: Общий контрактный тест-сьют

Проект SHALL содержать независимый от реализации сьют, который принимает `CoreContractHarness`, применяет JSON sync-фикстуры и проверяет snapshots и команды через публичный интерфейс. JS-реализация MUST проходить сьют; будущая Rust-реализация MUST подключаться к тому же набору сценариев.

#### Scenario: Прогон одной фикстуры на реализациях

- **WHEN** одна и та же фикстура initial/incremental sync применяется через harness разных реализаций
- **THEN** нормализованные snapshots и результаты команд совпадают с общими ожиданиями

#### Scenario: Регрессия relation или unread

- **WHEN** реализация неверно обрабатывает edit/reaction/thread/receipt/push-rule фикстуру
- **THEN** общий сьют завершается ошибкой на уровне контракта

### Requirement: Запрет прямых зависимостей от Matrix SDK

Проект SHALL запрещать импорты `matrix-js-sdk` и его subpath-модулей во всех файлах клиента и workspace-пакетов, кроме `core-matrix-js`. Запрет MUST проверяться командами lint в CI и локально.

#### Scenario: Новый импорт вне адаптера

- **WHEN** разработчик добавляет импорт `matrix-js-sdk` в component, hook, state или `@harrier/core`
- **THEN** ESLint завершается ошибкой и указывает запрещённый импорт

#### Scenario: Импорт внутри адаптера

- **WHEN** реализация `core-matrix-js` импортирует поддерживаемый SDK API
- **THEN** ESLint разрешает импорт, а публичный интерфейс адаптера продолжает экспортировать только типы `@harrier/core`

### Requirement: Сохранение поведения клиента в фазе 0

Миграция SHALL сохранять существующее пользовательское поведение JS-движка. Перед завершением MUST пройти contract tests, существующие тесты, typecheck, ESLint без errors, production build и desktop release build.

#### Scenario: Полная проверка JS-реализации

- **WHEN** все точки касания переведены и запрет включён
- **THEN** автоматические проверки зелёные, прямые SDK-импорты вне адаптера отсутствуют, а ручной smoke подтверждает login, rooms, timeline, send, crypto, media, presence, notifications, calls и threads без регрессий
