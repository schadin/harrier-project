## 1. Подготовка

- [x] 1.1 Проверить чистоту дерева `desktop/cinny` на ветке `harrier` (незакоммиченный work-in-progress отсутствует) и создать ветку под задачи #54/#55 в форке `schadin/harrier`
- [x] 1.2 Зафиксировать базовые проверки в `desktop/cinny`: `npm run check:eslint`, `npx tsc --noEmit`, `npx vitest run` — убедиться, что до правок всё чисто
  - База: eslint 0 ошибок (97 предсуществующих warnings), tsc чисто, vitest 136/136, prettier — предсуществующее замечание по `message/Reaction.tsx`

## 2. Флаг снятия ограничения Direct в MessageStatus (#55)

- [x] 2.1 Добавить в `MessageStatusProps` (`src/app/components/message/MessageStatus.tsx`) опциональный проп `always?: boolean` (дефолт `false`)
- [x] 2.2 В `MessageStatus`: при `always === true` показывать индикатор для всех своих сообщений, минуя условие `mDirects.has(room.roomId) || directReadReceipts !== 'checkmark'`; `CheckTwice` остаётся условным по наличию читателей
- [x] 2.3 Убедиться, что при дефолтном значении `always` поведение вне Bubble не изменилось (Modern/Compact, уведомления, закреплённые)
  - Тип индикатора вынесен в `MessageStatusIndicatorProps`, иначе `react/no-unused-prop-types` ругается на неиспользуемый `always` в общем типе
- [x] 2.4 Пробросить `always` и в шапку Modern/Compact — индикатор статуса одинаков во всех раскладках, расходится только положение

## 3. Нижняя строка мета-информации в Bubble (#54, #55)

- [x] 3.1 В `src/app/features/room/message/Message.tsx` добавить проп `edited?: boolean` в `MessageProps`; признак редактирования приходит из `RoomTimeline` (там уже считается `editedEvent`)
- [x] 3.2 Собрать футер `bubbleMetaFooter` — `Box direction="Row"` (`justifyContent="End"`, `gap="100"`, `marginTop: config.space.S100`): `MessageStatus` (с `always`), метка «(изменено)», `Time`
- [x] 3.3 Убрать `Time` и `MessageStatus` из `headerJSX` для `MessageLayout.Bubble` (в шапке остаётся имя отправителя); для Modern/Compact шапка не меняется
- [x] 3.4 В `RoomTimeline.tsx` (оба рендерера — `m.room.message` и `m.room.encrypted`) посчитать `showEditedInline = !!editedEvent && messageLayout !== MessageLayout.Bubble` и передавать его в `RenderMessageContent`, а `edited={!!editedEvent}` — в `Message`
  - В шифрованном рендерере `editedEvent` поднят в область видимости элемента `Message`
- [x] 3.5 Отступ под футер — инлайновым `marginTop: config.space.S100` (как у `Reactions`); `layout.css.ts` не меняется
- [x] 3.6 Футер подавляется при `edit` (открыт `MessageEditor`); `hideBubble` применим только к `EventContent`, к `Message` не передаётся — отдельная обработка не нужна
- [x] 3.7 Проверить проброс из других потребителей `RenderMessageContent` (уведомления, закреплённые, RoomTimeline) — там футер не передаётся, инлайновая метка сохраняется
  - `Notifications.tsx` и `RoomPinMenu.tsx` правок не требуют: они не используют Bubble-раскладку
- [x] 3.8 Отметка редактирования в футере — иконка `Icons.Pencil` (не текстовая метка); время снабжено тултипом `title` с полной датой; инлайновая `MessageEditedContent` из футера не используется

## 4. Проверки

- [x] 4.1 `npm run check:eslint`, `npx tsc --noEmit`, `npx vitest run`, `prettier --check` в `desktop/cinny` — чисто
  - eslint 0 ошибок, tsc чисто, vitest 136/136, prettier по изменённым файлам чисто
- [x] 4.1b `npm run build` в `desktop/cinny` — продакшен-сборка проходит
  - Сборка проходит; `git apply --check ../0002-rebrand-harrier.patch` в `cinny` — патч ребрендинга применяется чисто (затронутых файлов не касается)
- [ ] 4.2 `npm run tauri dev` (или `npm run dev` в `desktop/cinny`): вручную проверить в раскладке Bubble — порядок нижней строки «карандаш → время → статус» (#54, #55), время всегда присутствует, тултип на времени с полной датой, статус доставки своих сообщений в Direct и группе, отсутствие дубля отметки редактирования
- [ ] 4.6 В Modern и Compact проверить, что индикатор статуса для своих сообщений отображается так же, как в Bubble (в группе — `Check`, при read receipt — `CheckTwice`), а положение остаётся в шапке
- [ ] 4.3 Проверить незатронутость Modern и Compact: шапка со временем и статусом, инлайновая метка «(изменено)» — как до правок
- [ ] 4.4 Проверить сценарии: сообщение в состоянии отправки/ошибки, прочтённое в группе (`CheckTwice`), узкий пузырь (перенос строки), редактирование (футер скрыт), системные/закреплённые события
- [ ] 4.5 После сборки/`rebrand` убедиться, что в дереве `cinny` не осталось применённого патча ребрендинга (`npm run rebrand:revert`) — иначе он попадёт в коммит

## 5. Завершение

- [ ] 5.1 Коммиты в форке `schadin/harrier` со ссылками `schadin/harrier-project#54` / `#55` (вида `feat: ...` по стилю истории)
- [ ] 5.2 Обновить issues #54 и #55: отметить выполненные пункты плана
- [ ] 5.3 По завершении — архивация изменения (`/opsx:archive`) с синхронизацией спецификаций `messaging` и `appearance`
- [ ] 5.4 Спросить пользователя, нужно ли описание функции в `README.md` (зонтик или `desktop/`)
- [ ] 5.5 Push — только после подтверждения пользователем
