## 1. Индикация статуса отправки во всех комнатах

- [x] 1.1 Расширить `MessageStatus.tsx`: для собственных сообщений показывать `Cross` (NOT_SENT/CANCELLED) и `Clock` (SENDING/QUEUED/ENCRYPTING) в любой комнате и при любых настройках; `Check`/`CheckTwice` — только Direct + `checkmark` (текущее условие)
- [x] 1.2 Проверить, что события меню/статуса обновляются реактивно (подписка на `MatrixEventEvent.Status` остаётся)

## 2. Действия «Отправить снова» и «Удалить локально»

- [x] 2.1 Добавить в `Message.tsx` пункты меню «Resend» (`mx.resendEvent`) и «Remove Locally» (`mx.cancelPendingEvent`) для собственных сообщений со статусом NOT_SENT/CANCELLED
- [x] 2.2 Обработать ошибки resend (`.catch` → лог + уведомление) и исключения `cancelPendingEvent` (try/catch → лог)

## 3. Обработка ошибок отправки

- [x] 3.1 Добавить утилиту `sendMessageSafely`/`sendEventSafely` в `utils/matrix.ts`: лог причины + публикация в `sendErrorMessageAtom`
- [x] 3.2 Создать компонент `SendErrorToast` (jotai + Portal folds, auto-dismiss) и смонтировать в корне приложения
- [x] 3.3 Заменить fire-and-forget вызовы в `RoomInput.tsx` (сообщения, вложения, стикеры), `MessageEditor.tsx`, `RoomTimeline.tsx` (реакции) на безопасные обёртки

## 4. Проверка

- [x] 4.1 `npm run lint` и `npm run typecheck` в `desktop/cinny` — без ошибок
- [x] 4.2 Ручная проверка: отправка при отключённой сети → `Clock`/`Cross` в групповой комнате; «Resend» после восстановления сети; «Remove Locally»; поведение Direct без изменений
- [ ] 4.3 Зафиксировать текст ошибки E2EE-отправки из логов/toast (при воспроизведении)