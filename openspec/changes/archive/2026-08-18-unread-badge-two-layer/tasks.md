## 1. Диагностика

- [x] 1.1 Добавить временный `console.log` в `roomToUnread.ts`: `ATOM PUT`/`ATOM DELETE` (roomId, total, дельта, родители и итог родителя), `RECOMPUTE` (roomId, isSpace, действие), слепок `RESET`
- [ ] 1.2 Воспроизвести на десктопе: подтвердить механизм «застревания» бейджа Space (задокументировать в issue #34)

## 2. Слой комнат — единый пересчёт

- [x] 2.1 Добавить предикат `roomIsUnread(mx, room)` в `src/app/utils/room.ts` (таймлайн загружен → `roomHaveUnread`, иначе `roomHaveNotification`)
- [x] 2.2 Добавить `recomputeRoom(room)` в `useBindRoomToUnreadAtom`: Space пропускается; не-join/mute → DELETE; `roomIsUnread` → PUT/DELETE
- [x] 2.3 Перевести `handleTimelineEvent` на `recomputeRoom`
- [x] 2.4 Перевести `handleReceipt` на `recomputeRoom` (вместо безусловного DELETE)
- [x] 2.5 Заменить `updateRoomUnread` (UnreadNotifications) на `recomputeRoom`
- [x] 2.6 Перевести `handleMembershipChange` на `recomputeRoom`
- [x] 2.7 Перевести `getUnreadInfos` (RESET) на `roomIsUnread`

## 3. Слой производных — живые суммы

- [x] 3.1 Бейдж вкладки Space (`SpaceTabs.tsx`): `RoomUnreadProvider` → `RoomsUnreadProvider rooms={allChild}` через `useSpaceChildren`
- [x] 3.2 Трей (`ClientNonUIFeatures.tsx`, DesktopFeatures): исключить Space-агрегаты при суммировании
- [x] 3.3 Favicon: исключить Space-агрегаты

## 4. Чистка и проверка

- [x] 4.1 Удалить диагностический лог (1.1)
- [x] 4.2 `npm run lint` и `npm test` в `desktop/cinny`
- [ ] 4.3 Собрать и проверить вручную: бейджи комнат и Space гаснут сразу; трей без двойного счёта; новое сообщение снова помечает; после перезапуска состояние то же
- [ ] 4.4 Обновить issue #34 и закрыть при успехе
