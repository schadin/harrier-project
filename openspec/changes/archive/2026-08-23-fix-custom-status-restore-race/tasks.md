# Tasks: fix-custom-status-restore-race

## 1. Реализация

- [x] 1.1 `custom-status.ts`: переписать `restoreCustomStatus` — матрица решений
        по известному серверному значению, приоритет сервера, восстановление с
        сохранением текущего presence.
- [x] 1.2 `ClientNonUIFeatures.tsx` (CustomStatusRestore): подписка на
        `UserEvent.Presence` своего пользователя + страховочный таймаут (15 c).

## 2. Проверка

- [x] 2.1 `npx tsc --noEmit` → 0 errors; `npm run check:eslint` → 0 errors.
- [x] 2.2 Логика соответствует сценариям спеки (совпадает / отличается /
        пустое / таймаут).

## 3. Фиксация

- [ ] 3.1 Коммит в `desktop/cinny` со ссылкой schadin/harrier-project#44; бамп
        указателя в `desktop/`.
- [ ] 3.2 Архивирование change; коммит зонтика #44.
