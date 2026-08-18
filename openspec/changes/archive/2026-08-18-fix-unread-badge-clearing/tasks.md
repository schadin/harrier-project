## 1. Диагностический лог

- [x] 1.1 Добавить временный `console.log` в `markAsRead` (`src/app/utils/notifications.ts`): вызов (roomId, длина таймлайна, readEventId), ранний выход, результат `sendReadReceipt` (успех/ошибка)
- [x] 1.2 Добавить временный `console.log` в `useBindRoomToUnreadAtom` (`src/app/state/room/roomToUnread.ts`): `PUT`/`DELETE`/`RESET` с roomId и значением счётчика
- [x] 1.3 Собрать и воспроизвести на десктопе: подтвердить, что receipt уходит, `DELETE` срабатывает, затем приходит «stale» `PUT` (задокументировать в issue #33)

## 2. Фикс атома непрочитанного

- [x] 2.1 Добавить guard в `handleTimelineEvent`: не выполнять `PUT`, если `roomHaveUnread(mx, room) === false`
- [x] 2.2 Добавить helper пересчёта записи комнаты по предикату `getUnreadInfos` (space/не-join/mute исключаются; `roomHaveNotification || roomHaveUnread` → PUT, иначе DELETE)
- [x] 2.3 Подписаться на `RoomEvent.UnreadNotifications` для каждой комнаты `mx.getRooms()` в `useBindRoomToUnreadAtom`
- [x] 2.4 Регистрировать слушатель для новых комнат через `ClientEvent.Room`; снимать слушатель и удалять запись атома через `ClientEvent.DeleteRoom`; очистка слушателей при unmount/смене `mx`

## 3. Чистка и проверка

- [x] 3.1 Удалить диагностический лог (1.1, 1.2)
- [x] 3.2 Прогнать `npm run lint` и `npm test` в `desktop/cinny`
- [x] 3.3 Собрать десктоп и проверить вручную: бейдж снимается сразу после прочтения, без перезапуска; трей-бейдж обновляется; новое сообщение снова помечает комнату непрочитанной
- [x] 3.4 Обновить issue #33: отметить выполнение и результат проверки
