# Tasks: add-message-tables

**Задача:** [schadin/harrier-project#70](https://github.com/schadin/harrier-project/issues/70).
Реализация — в веб-клиенте `desktop/client` (ветка `harrier` в `desktop/`).
Номера строк и путей указаны от корня `desktop/`.

## 1. Приём: рендер HTML-таблиц

- [ ] 1.1 Добавить `colspan`, `rowspan`, `align` в белый список атрибутов для
  `th`/`td` в `permittedTagToAttributes` (`client/src/app/utils/sanitize.ts`);
  остальные атрибуты табличных тегов оставить вне списка
- [ ] 1.2 Обработать `table`, `caption`, `thead`, `tbody`, `tr`, `th`, `td` в
  `getReactCustomHtmlParser` (`client/src/app/plugins/react-custom-html-parser.tsx`):
  обёртка с горизонтальной прокруткой для `table`, валидация `colspan`/`rowspan`
  (целое 1..1000) и `align` (`left`/`center`/`right`), невалидные значения
  игнорировать
- [ ] 1.3 Добавить стили в `client/src/app/styles/CustomHtml.css.ts`: контейнер
  таблицы (`overflow-x: auto`, `max-width: 100%`), таблица (`border-collapse`,
  рамки `SurfaceVariant.ContainerLine`), ячейки (отступы `config.space`,
  `white-space: normal`, `overflow-wrap: anywhere`), заголовок (`th`, `thead`
  на `SurfaceVariant.Container`), `caption`
- [ ] 1.4 Проверить, что внутри ячеек продолжают работать инлайн-код, блок кода
  с подсветкой Prism, ссылки, спойлеры и упоминания matrix.to
- [ ] 1.5 Юнит-тест на таблицу: сохранение `colspan`/`rowspan`/`align` после
  `sanitizeCustomHtml` и отбрасывание невалидных значений
  (`client/src/app/utils/sanitize.test.ts`); при возможности тестовой среды —
  smoke-рендер через `renderToStaticMarkup`
- [ ] 1.6 Проверить отображение во всех точках рендера: лента, закреплённые
  сообщения, результаты поиска, inbox

## 2. Отправка: markdown-таблицы

- [ ] 2.1 Реализовать блочное правило pipe-таблиц
  (`client/src/app/plugins/markdown/block/table.ts`): строка заголовка +
  обязательная строка разделителя, выравнивание `:--`/`:--:`/`--:`,
  необязательные внешние `|`, `\|` как литерал внутри ячейки, инлайн-разметка в
  ячейках через `parseInlineMD`
- [ ] 2.2 Зарегистрировать правило в `parseBlockMD`
  (`client/src/app/plugins/markdown/block/parser.ts`) перед `ListRule` и
  экспортировать из `client/src/app/plugins/markdown/block/index.ts`
- [ ] 2.3 Проверить, что `toPlainText` оставляет исходный markdown в `body`, а
  `customHtmlEqualsPlainText` формирует `format: org.matrix.custom.html` +
  `formatted_body`
- [ ] 2.4 Юнит-тесты (`client/src/app/plugins/markdown/block/table.test.ts`):
  таблица с выравниванием, без внешних пайпов, экранированный `\|`, отсутствие
  строки разделителя, markdown выключен (через `toMatrixCustomHTML`)
- [ ] 2.5 Ручная проверка: отправить pipe-таблицу при включённом и выключенном
  Markdown, убедиться, что в событии есть `formatted_body` с `<table>`, а `body`
  содержит исходный текст

## 3. Правка: сохранение таблицы без потерь

- [ ] 3.1 Добавить `BlockType.Table` в `client/src/app/components/editor/types.ts`
  и тип элемента в `client/src/app/components/editor/slate.d.ts`: блочный узел,
  хранящий исходный HTML таблицы
- [ ] 3.2 В `htmlToEditorInput` (`client/src/app/components/editor/input.ts`)
  создавать узел таблицы с исходным HTML вместо схлопывания в абзацы
- [ ] 3.3 Рендерить узел в `Elements.tsx` read-only через
  `getReactCustomHtmlParser` (без обработчиков кликов), не допуская правки
  содержимого ячеек
- [ ] 3.4 В `toMatrixCustomHTML` (`client/src/app/components/editor/output.ts`)
  возвращать сохранённый HTML таблицы без изменений; в `toPlainText` — читаемый
  текстовый fallback
- [ ] 3.5 Проверить `MessageEditor`: правка текста вокруг таблицы не меняет
  таблицу и отправляется через существующий `m.replace`; тест round-trip
  `formatted_body → editor → formatted_body`
- [ ] 3.6 Ручная проверка: отредактировать собственное сообщение с таблицей и
  сообщение с таблицей, полученное из другого клиента

## 4. Спецификация и доменная модель

- [x] 4.1 Создать лейбл `rich-text` и issue
  [schadin/harrier-project#70](https://github.com/schadin/harrier-project/issues/70)
- [x] 4.2 Change `add-message-tables`: proposal, design, delta-спека `rich-text`,
  tasks
- [ ] 4.3 Добавить capability `rich-text` в доменную модель зонтика
  (`README.md` и `AGENTS.md` в `schadin/harrier-project`)
- [ ] 4.4 `openspec validate --specs` и `openspec status --change
  add-message-tables` без ошибок

## 5. Проверка

- [ ] 5.1 `cd desktop/client && npm ci` (в рабочей копии нет `node_modules`)
- [ ] 5.2 `npm run test` — все тесты, включая новые
- [ ] 5.3 `npm run lint`
- [ ] 5.4 `npm run typecheck`
- [ ] 5.5 `npm run build`
- [ ] 5.6 Ручная проверка в запущенном клиенте: отправить событие с
  `formatted_body`-таблицей в тестовую комнату (matrix API/curl), проверить ленту,
  поиск, закреплённые, inbox и E2EE-комнату

## 6. Финал

- [ ] 6.1 Спросить пользователя, нужно ли описание функции в `README.md`
  (зонтик или `desktop/`)
- [ ] 6.2 Обновить issue #70: отметить критерии приёмки и дополнить «Учёт
  изменений» по факту реализации
- [ ] 6.3 Коммит только после подтверждения пользователем, со ссылкой
  `schadin/harrier-project#70`
- [ ] 6.4 После приёмки архивировать change (`/opsx:archive`) — синхронизация
  требований в `openspec/specs/rich-text/spec.md`
