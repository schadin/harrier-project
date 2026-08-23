# Design: fix-lint-typecheck-baseline

## Context

Форк cinny (подмодуль `desktop/cinny`) собирается Vite, но `tsc --noEmit`
падает с 823 ошибками. Корневая причина: matrix-js-sdk v41 поставляет `.d.ts`
с относительными импортами через `.ts`-суффикс (`export * from "./matrix.ts"`),
которые не резолвятся при `"moduleResolution": "Node"`. SDK резолвится как
пустой модуль → каскад TS2614/TS2305/TS7006/TS2786. Дополнительно 42 ошибки
eslint: 19 в крипто-файле `cryptE2ERoomKeys.js` (легитимные битовые операции),
глобал `__CINNY_VERSION__` из Vite define не объявлен, мелкие неиспользуемые
импорты и хуки.

## Goals / Non-Goals

**Goals**

- `npm run typecheck` → 0 errors; `npm run lint` → 0 errors.
- Нулевое влияние на рантайм и сборку (`npm run build` без регрессий).

**Non-Goals**

- Переписывание крипто-кода `cryptE2ERoomKeys.js`.
- Повышение строгости правил eslint / устранение warnings сверх errors.
- Обновление версий matrix-js-sdk или TypeScript.

## Decisions

1. **`module: "ESNext"` + `moduleResolution: "Bundler"`** в tsconfig.json —
   единственный режим, который резолвит `.ts`-спецификаторы в `.d.ts`
   зависимостей; проект уже собирается bundler'ом (Vite), `tsc` используется
   только как гейт. Альтернатива — патчить `.d.ts` SDK через patch-package —
   отвергнута как более хрупкая и тяжёлая в сопровождении.
2. **ESLint override для `src/util/cryptE2ERoomKeys.js`**, а не правка кода:
   no-bitwise/no-plusplus — легитимные идиомы BigInteger-крипто; переписывание
   повышает риск регрессии в шифровании. Альтернатива inline
   `/* eslint-disable */` — менее декларативна.
3. **`__CINNY_VERSION__` объявить в eslint `globals`** конфига (источник —
   Vite define), а не удалять использование.
4. **Точечные правки** остальных файлов: удаление неиспользуемых импортов,
   перестановка объявлений в `utils/dom.ts`, стабильные ключи вместо индексов,
   дополнение deps `useMemberSort.ts`.

## Risks / Trade-offs

- [Bundler-resolution может вскрыть скрытые типовые несоответствия] → после
  смены moduleResolution прогнать typecheck и дочинить реальный остаток;
  остаток фиксируем в tasks.
- [Правка dependency-массива useMemberSort может изменить поведение сортировки]
  → добавляем только реально используемую зависимость, ручной прогон списка.
- [Изменения затрагивают подмодуль] → коммит внутри `desktop/`, затем бамп
  указателя подмодуля в зонтике.

## Migration Plan

Один коммит в подмодуле + один в зонтике (change-артефакты). Откат — revert
обоих коммитов; рантайм не затронут, миграций нет.

## Open Questions

- нет — решения зафиксированы в issue #31.
