# Proposal: fix-lint-typecheck-baseline

## Why

`npm run lint` и `npm run typecheck` в форке schadin/harrier падают на
предсуществующих ошибках (823 ошибки tsc, 42 ошибки eslint), поэтому ни одна
доработка не может проходить проверку качества — гейты «красные» на базовой
ревизии. Это блокирует весь цикл разработки.

Связанная задача: schadin/harrier-project#31.

## What Changes

- `tsconfig.json` (cinny): `module: "ESNext"` + `moduleResolution: "Bundler"` —
  устраняет каскад из ~500+ ошибок TS2614/TS2305 из-за `.ts`-суффиксов в
  относительных импортах `.d.ts` matrix-js-sdk v41.7.0.
- eslint: override для `src/util/cryptE2ERoomKeys.js` (легитимные крипто-идиомы:
  no-bitwise, no-plusplus, prefer-template, no-param-reassign).
- eslint: объявить глобал `__CINNY_VERSION__` (Vite define).
- Удалить неиспользуемые импорты/переменные (PresenceStatus.tsx, Message.tsx,
  lastActive.ts и др.).
- Исправить `no-use-before-define` в `utils/dom.ts`, dependency-массив в
  `useMemberSort.ts`, `react/no-array-index-key` в ProfileStatus.tsx /
  StatusButton.tsx.
- Цель: `lint` → 0 errors; `typecheck` → 0 errors. Рантайм не затрагивается
  (сборка Vite уже проходит).

## Capabilities

### New Capabilities

- нет

### Modified Capabilities

- нет — изменения касаются только инструментария проверки качества
  (delivery), видимого поведения продукта и требований спецификаций не меняют.

## Impact

- Код: `desktop/cinny/tsconfig.json`, `desktop/cinny/eslint.config.js`
  (или аналог), ~10 файлов фронтенда с точечными правками.
- Коммит в подмодуле `desktop/` (форк cinny внутри него), ссылка
  schadin/harrier-project#31.
- Спецификации OpenSpec не изменяются.
