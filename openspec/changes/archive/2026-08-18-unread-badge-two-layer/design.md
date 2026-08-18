## Context

Бейджи непрочитанного (комната, Space, трей, favicon) читают атом
`roomToUnreadAtom`. Атом ведётся инкрементально: `putUnreadInfo`/`deleteUnreadInfo`
корректируют Space-агрегаты дельтами при PUT/DELETE дочерних комнат. Плюс фикс #33
добавил `updateRoomUnread` (подписка на `RoomEvent.UnreadNotifications`), который
для самих Space делает `DELETE` их агрегатов — прямое вмешательство в агрегацию,
создающее «застревание» бейджа Space до перезапуска.

Дополнительные дефекты текущей схемы:
- `getUnreadInfo` читает серверный счётчик, который отстаёт от реальности после
  прочтения (обновляется только в следующем sync) — отсюда исходный баг #33.
- Несколько писателей атома (Timeline, Receipt, UnreadNotifications, Membership,
  RESET) без единой реконсиляции.
- `handleReceipt` DELETE безусловный: любой свой receipt (даже старый) снимает
  комнату, даже если выше есть более новое непрочитанное.
- Бейдж трея суммирует атом целиком, включая Space-агрегаты (двойной счёт).

## Goals / Non-Goals

**Goals:**

- Бейджи комнат и Space снимаются сразу после прочтения, без перезапуска.
- Бейдж Space и трей не зависят от инкрементальной агрегации (живые суммы).
- Единая точка записи комнат в атоме (реконсиляция из состояния SDK).

**Non-Goals:**

- Не удаляем агрегацию в атоме полностью (она остаётся инвариантом для остальных
  потребителей, например Search) — только перестаём читать её в SpaceTab/трее.
- Не чиним обратную проблему «phantom badges» в бриджах (upstream #2589).
- Не возвращаем `m.fully_read` marker (upstream откатил).

## Decisions

### 1. Предикат `roomIsUnread` — локальный признак приоритетнее счётчика

```ts
export const roomIsUnread = (mx, room): boolean => {
  if (room.getLiveTimeline().getEvents().length === 0) {
    return roomHaveNotification(room); // таймлайн не загружен — верим серверу
  }
  return roomHaveUnread(mx, room);     // загружен — локальный признак по receipt
};
```

Устраняет корень #33: прочитанная комната (таймлайн загружен) больше не
пере-добавляется по устаревшему серверному счётчику.

Альтернатива (отклонена): `roomHaveNotification || roomHaveUnread` как раньше —
устаревший счётчик «побеждает» и возвращает прочитанную комнату.

### 2. Единый `recomputeRoom` — единственный писатель для комнат

```ts
const recomputeRoom = (room) => {
  if (room.isSpaceRoom()) return;                    // Space не трогаем напрямую
  if (room.getMyMembership() !== Membership.Join) { DELETE; return; }
  if (mute) { DELETE; return; }
  roomIsUnread(mx, room) ? PUT getUnreadInfo(room) : DELETE;
};
```

Все писатели (`handleTimelineEvent`, `handleReceipt`, `updateRoomUnread`,
`handleMembershipChange`) переводятся на `recomputeRoom`. Это чинит и безусловный
`DELETE` в `handleReceipt` (комната с более новым непрочитанным останется).

`getUnreadInfos` (RESET) переводится на `roomIsUnread` — согласованность между
RESET и in-session пересчётом.

### 3. Живые суммы для Space и трея (слой производных)

- `SpaceTab` (`SpaceTabs.tsx:434`): `RoomUnreadProvider roomId={space.roomId}` →
  `RoomsUnreadProvider rooms={allChild}` через `useSpaceChildren` (уже используется
  в `SpaceMenu` и лобби Space).
- Трей (`ClientNonUIFeatures.tsx`, DesktopFeatures): при суммировании пропускать
  `mx.getRoom(roomId)?.isSpaceRoom()`.
- Favicon: то же исключение Space.

Даже если агрегация в атоме сместится, бейджи Space/трея останутся корректными.

### 4. Агрегация в атоме остаётся

`putUnreadInfo`/`deleteUnreadInfo` сохраняются (инвариант «Space = сумма детей»).
Они вызываются только из PUT/DELETE комнат. Прямых операций над Space нет.

## Risks / Trade-offs

- [Изменение RESET-предиката на `roomIsUnread` влияет на E2EE/бриджи] → для E2EE
  счётчик локальный, для бриджей поведение сохраняется как в upstream; проверяется
  ручной проверкой.
- [`useSpaceChildren` в `SpaceTab` добавляет вычисления на каждую вкладку] →
  дёшево (рекурсивная выборка детей), аналогично `SpaceMenu`.
- [Бейдж Search для Space теперь может отличаться от бейджа вкладки] → Search
  остаётся на агрегате; при желании выровняется отдельно (вне скоупа).

## Migration Plan

- Только `desktop/cinny`. Миграции данных нет.
- Откат: revert коммитов.

## Open Questions

- Точная последовательность «застревания» Space подтверждается диагностическим
  логом; если механизм окажется иным — фикс корректируется по данным лога.
