## Context

Форк веб-клиента (`desktop/cinny`) держит три раскладки сообщений
(`MessageLayout`): `Modern`, `Compact`, `Bubble`. Раскладка выбирается файлом
`src/app/features/room/message/Message.tsx`, который собирает три JSX-фрагмента:

- `headerJSX` — имя отправителя + `Time` + `MessageStatus`;
- `msgContentJSX` — `Box direction="Column"` с reply / контентом / реакциями;
- `avatarJSX`.

Эти фрагменты разносятся по `CompactLayout` / `BubbleLayout` / `ModernLayout`
(`src/app/components/message/layout/`). `BubbleLayout` принимает `before`,
`header`, `children`: `header` рендерится **над** пузырём, `children` — внутри
пузыря (`layout.css.ts` → `BubbleContent`, `SurfaceVariant`).

Метка «(изменено)» живёт ниже: `MessageEditedContent`
(`content/FallbackContent.tsx`) вызывается из `MText` / `MEmote` / `MNotice`
(`MsgTypeRenderers.tsx`) по флагу `edited`, который приходит из
`RenderMessageContent.tsx` ← `RoomTimeline.tsx` (`edited={!!editedEvent}`).

`MessageStatus` (`components/message/MessageStatus.tsx`) для собственных
сообщений показывает `Cross` / `Clock` всегда, но `Check` / `CheckTwice` — только
в Direct при `directReadReceipts === 'checkmark'`; иначе возвращает `null`.
Требование протокольного инварианта соблюдается: `Local echo` /
`getAssociatedStatus`, read receipts и `m.relates_to` → `m.replace` уже
используются клиентом.

Ограничение по мощности: правки лендинг-текстовых сообщений затрагивают общий
рендер-путь (`RenderMessageContent`), который переиспользуется
уведомлениями (`pages/client/inbox/Notifications.tsx`), меню закреплённых
(`room-pin-menu/RoomPinMenu.tsx`) и таймлайном.

## Goals / Non-Goals

**Goals:**
- Единая нижняя строка мета-информации внутри пузыря: время, статус доставки,
  «(изменено)».
- Ноль изменений для `Modern` и `Compact` — включая формат шапки.
- Показ статуса доставки для своих сообщений в пузыре всегда, без привязки к
  типу комнаты и режиму `directReadReceipts` (кроме `CheckTwice`, который
  остаётся за read receipts).
- Отсутствие дубля «(изменено)» (инлайн + нижняя строка) в Bubble.

**Non-Goals:**
- Выравнивание «время в одну строку с последней строкой текста» (как в
  Telegram) — требует измерения текста, вне задачи.
- Клик по нижней строке, тултипы, hover-состояния, i18n.
- Изменение поведения `MessageStatus` вне Bubble и правки `ReadReceiptGroup`.
- Любые изменения Rust/десктоп-обёртки и настроек.

## Decisions

**1. Собирать нижнюю строку в `Message.tsx`, а не в `BubbleLayout`.**

`BubbleLayout` — чистый презентационный контейнер (`before`/`header`/`children`),
который ничего не знает о событии, комнате, `mx` и настройках. Данные для нижней
строки (время, статус, признак редактирования) живут в `Message.tsx`, где уже
есть `mEvent`, `room`, `mx`. Поэтому футер формируется там же, где `headerJSX`, и
передаётся как часть `children` в `BubbleLayout`.

*Альтернатива:* передавать `footer` пропом в `BubbleLayout`. Отвергнута:
контейнер остаётся «немым», но требует нового публичного контракта ради одного
расклада; композиция через `children` даёт тот же результат без API-изменений.

**2. Футер — часть `msgContentJSX`, а не отдельный слот.**

`msgContentJSX` — `Box direction="Column"`, уже обёрнутый в пузырь. Футер
добавляется последним ребёнком этого `Box` с `marginTop` по токену
`config.space.S100`. Для не-Bubble раскладок футер не рендерится
(`messageLayout === MessageLayout.Bubble`).

**3. Признак редактирования остаётся в `RoomTimeline.tsx`, а не считается заново в
`Message.tsx`.**

`RoomTimeline.tsx` уже вычисляет `editedEvent` и передаёт `edited` в
`RenderMessageContent`. Чтобы убрать инлайновую метку в Bubble, рендер-путь менять
не нужно: в `RoomTimeline` считается `showEditedInline = !!editedEvent &&
messageLayout !== Bubble` и уходит в `RenderMessageContent`, а сам признак
редактирования отдельно передаётся в `Message` пропом `edited`. Дублировать поиск
редактирования в `Message.tsx` (через `getEventEdits` + доступ к `timelineSet`,
которого у `Message` нет) значило бы задваивать обход relations и тащить в
`Message` зависимость от таймлайна.

*Альтернатива:* завести общий контекст/атом «раскладка + признак мета-футера» и
читать его в `MsgTypeRenderers`. Отвергнута: React-контекст ради одного флага в
глубоко вложенном рендере дороже, чем явный проп; и он создаёт неявную связь
рендера контента с UI-раскладкой. Потребители `RenderMessageContent` вне ленты
(уведомления, закреплённые) не трогаются вовсе — у них Bubble-раскладки нет.

**4. `MessageStatus` получает опциональный проп `always?: boolean`.**

`always` = показывать индикатор для всех своих сообщений независимо от
`mDirects`/`directReadReceipts`. `CheckTwice` при этом остаётся условным: он
зависит от наличия read receipt (читателей), а не от флага — флаг снимает
ограничение «только Direct», но не показывает «прочитано» там, где receipts нет.
Дефолт `false` — поведение вне Bubble не меняется.

*Альтернатива:* дублировать логику статуса в отдельном `BubbleMessageStatus`.
Отвергнута: дублирование подписок на `MatrixEventEvent.Status` и
`useRoomEventReaders` — источник расхождений.

**5. Формат нижней строки.**

`Box direction="Row"` c `justifyContent="End"` (right-aligned внутри пузыря,
пузырь сам узкий), `gap="100"`, размер `T200`, `priority="300"` — визуальный вес
совпадает с текущим `Time`. Порядок: `CheckTwice` → `Check`/`Clock`/`Cross` →
«(изменено)» → `Time`. Для правого края при RTL/узком пузыре допускается перенос
на вторую строку (`flexWrap`), ширина пузыря не фиксируется.

`MessageEditedContent` получает опциональный проп `noLeadingSpace`: инлайновая
метка остаётся с ведущим пробелом (как была), а в футере пробел не нужен — там
отступ задаёт `gap`. Отступ снизу задаётся инлайновым
`marginTop: config.space.S100` (как у `Reactions`), поэтому `layout.css.ts` не
меняется.

## Risks / Trade-offs

- **Пузырь растягивается по ширине футера** → футер короткий (`T200`, время +
  1–2 иконки), поэтому прирост ширины минимален; ширина пузыря и так задаётся
  контентом.
- **Регрессия в общем рендер-пути контента** (уведомления, закреплённые,
  таймлайн) → правки ограничиваются новым опциональным пропом с дефолтом
  «как раньше»; проверка `tsc`, `eslint`, `vitest`.
- **Расхождение статуса в Bubble и в Discrete/уведомлениях** → общая
  реализация `MessageStatus`, флаг только снимает ограничение Direct-проверки.
- **Дубль «(изменено)»** при частичном применении правок → в задачах явный
  пункт ручной проверки отредактированного сообщения в Bubble.
- **Поведение `Check` в групповых комнатах расходится с текущей спецификацией**
  → спецификация `messaging` правится в этом же изменении (это запрошенное
  поведение #55).

## Migration Plan

Откат — коммит-реверт одного PR в форке: изменения локализованы в четырёх файлах
`src/app/components/message/**` и одном файле `src/app/features/room/message/`,
настройки и данные не меняются, миграций нет.
