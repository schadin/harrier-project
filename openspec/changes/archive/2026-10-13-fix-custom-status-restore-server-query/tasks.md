# Tasks: fix-custom-status-restore-server-query

## 1. Реализация

- [x] 1.1 `custom-status.ts`: добавить чистую `decideStatusRestore(serverMsg, saved)`
      (действия `noop` / `show-server` / `adopt-server` / `restore`); переписать
      `restoreCustomStatus(mx)` на асинхронный запрос `mx.getPresence(userId)` с
      применением решения и результатом `done` / `retry`.
- [x] 1.2 `custom-status.ts`: обновлять локальное значение и UI
      (`updateLocalPresence`) в действиях `show-server`, `adopt-server` и
      `restore`; не выполнять автовосстановление, если пользователь изменил
      статус за время запроса (сравнение локального кэша).
- [x] 1.3 `ClientNonUIFeatures.tsx` (`CustomStatusRestore`): заменить подписку на
      `UserEvent.Presence` + одноразовый таймаут на цикл ретраев с нарастающей
      задержкой (1/3/10/30/60 с), остановка при размонтировании.

## 2. Тесты

- [x] 2.1 `custom-status.test.ts`: матрица решений — сервер пуст/отсутствует →
      `restore`; сервер отличается → `adopt-server`; совпадает → `show-server`;
      оба пусты → `noop`.
- [x] 2.2 Тесты `restoreCustomStatus`: ошибка `getPresence` → `retry` и
      неизменный локальный кэш; успешный пустой ответ → `setPresence` с текущим
      presence; смена статуса пользователем во время запроса → `done` без
      перезаписи.

## 3. Проверка

- [x] 3.1 `npm test` (156), `npx tsc --noEmit` (0 ошибок), `npm run check:eslint`
      (0 ошибок), `prettier --check` — без замечаний.
- [x] 3.2 Поведение соответствует сценариям спеки (пусто / отличается /
      совпадает / ошибка запроса / выбор пользователя).

## 4. Фиксация

- [x] 4.1 Коммит в `desktop/cinny` со ссылкой schadin/harrier-project#67; бамп
      указателя в `desktop/`.
- [x] 4.2 Архивирование change; коммит зонтика #67.
