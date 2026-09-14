# fix-message-send-error-visible

## Why

В зашифрованных комнатах сообщения иногда не попадают на сервер (локальный echo со статусом `NOT_SENT`), но в групповых комнатах это никак не отображается: индикатор статуса (`Cross`/`Clock`) показывается только в Direct при режиме `directReadReceipts === 'checkmark'`. Пользователь считает сообщение отправленным, хотя другие участники его не видят; текст сообщения теряется без возможности повторить отправку.

## What Changes

- В групповых комнатах для собственных сообщений показывать статус отправки только в аномальных состояниях: `Clock` (SENDING/QUEUED/ENCRYPTING) и `Cross` (NOT_SENT/CANCELLED). При успешной отправке индикатор не показывается.
- В Direct при `directReadReceipts === 'checkmark'` поведение остаётся прежним (`Check`/`CheckTwice`/`Clock`/`Cross`).
- В контекстном меню сообщения с ошибкой отправки добавить действия «Отправить снова» (`mx.resendEvent`) и «Удалить локально» (`mx.cancelPendingEvent` + удаление из таймлайна).
- Обработка ошибок всех отправок (сообщения, вложения, стикеры, реакции, редактирование): логирование причины и уведомление пользователя.

## Capabilities

### New Capabilities
- _(нет)_

### Modified Capabilities
- `messaging`: расширяется требование «Индикация статуса сообщения в Direct» — статус отправки отображается также в групповых комнатах (только аномальные состояния); добавляется возможность повторной отправки и локального удаления неудачных сообщений.

## Impact

- `desktop/cinny/src/app/components/message/MessageStatus.tsx` — расширение логики отображения.
- `desktop/cinny/src/app/features/room/message/Message.tsx` — пункты контекстного меню для событий с ошибкой отправки.
- `desktop/cinny/src/app/features/room/RoomInput.tsx`, `RoomTimeline.tsx`, `message/MessageEditor.tsx` — обработка ошибок отправки.
- Документация: `openspec/specs/messaging/spec.md`, README при необходимости.