# Tasks: fix-lint-typecheck-baseline

## 1. Typecheck

- [x] 1.1 В `desktop/cinny/tsconfig.json` установить `module: "ESNext"`,
      `moduleResolution: "Bundler"`; подъём typescript до ^5.4 (TS <5 не
      поддерживает Bundler); прогон typecheck — остаток 39 реальных ошибок.
- [x] 1.2 Исправлены все остатки: типизация event-хендлеров, касты кастомных
      ключей account-data/state-event к keyof SDK-типам, type-guard фильтры,
      SetPresence.Online, SendToDeviceContentMap, RoomMessageEventContent,
      setAttribute('sandbox'), lib ES2022.Intl, токены R200→R300/B200→B300.

## 2. ESLint

- [x] 2.1 Override для `src/util/cryptE2ERoomKeys.js`
      (no-bitwise, no-plusplus, prefer-template, no-param-reassign).
- [x] 2.2 `__CINNY_VERSION__` объявлен в eslint globals.
- [x] 2.3 Удалены неиспользуемые импорты/переменные (PresenceStatus.tsx,
      Message.tsx, lastActive.ts).
- [x] 2.4 dom.ts: хендлеры переведены на function declarations;
      правило no-use-before-define ослаблено для функций (hoisting).
- [x] 2.5 Стабильные ключи вместо индексов в ProfileStatus.tsx (×2) и
      StatusButton.tsx.
- [x] 2.6 Dependency-массив useMemberSort дополнен `getPowerLevel`.
- [x] 2.7 (сверх плана) Подъём @typescript-eslint до ^7 и eslint до ^8.57
      (v5-парсер несовместим с TS 5); no-explicit-any понижен до warning
      (сохранение прежней базы предупреждений); `*.css` в .eslintignore
      (убирает предсуществующую parsing-ошибку скрипта).

## 3. Проверка

- [x] 3.1 `npm run check:eslint` → 0 errors; `npx tsc --noEmit` → 0 errors.
      Примечание: `check:prettier` падает на 34 предсуществующих файлах,
      не затронутых изменениями (отдельная задача).
- [x] 3.2 `npm run build` проходит без регрессий.

## 4. Фиксация

- [ ] 4.1 Коммит в подмодуле `desktop/cinny` со ссылкой
      schadin/harrier-project#31; бамп указателя подмодуля.
- [ ] 4.2 Архивирование change; коммит зонтика #31.

