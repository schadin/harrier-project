# Персист черновиков сообщений между сессиями — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Черновики текста и reply-ответов per-room сохраняются в localStorage и восстанавливаются после перезапуска приложения.

**Architecture:** Существующие jotai-атомы `roomIdToMsgDraftAtomFamily` / `roomIdToReplyDraftAtomFamily` переводятся на готованную в кодовой базе утилиту `atomWithLocalStorage` (`src/app/state/utils/atomWithLocalStorage.ts`). Ключ localStorage включает `roomId`; черновик записывается в storage при каждом `set` атома (по факту — при размонтировании RoomInput или отправке/отмене reply). Восстановление происходит автоматически при создании атома (первое чтение из localStorage).

**Tech Stack:** jotai 2.6.0 (atomFamily), localStorage, vitest 4 (node-окружение + ручной стаб `globalThis.localStorage`), Slate (`Descendant[]`).

**Spec:** [openspec/specs/messaging/spec.md](../../../openspec/specs/messaging/spec.md) — в Task 4 добавляется Requirement «Персист черновиков между сессиями».

## Global Constraints

- Инвариант протокола не затрагивается: хранение строго локальное (localStorage), никаких сетевых событий/account data.
- Использовать существующую утилиту `atomWithLocalStorage`, а не jotai `atomWithStorage` (конвенция кодовой базы: `src/app/state/utils/atomWithLocalStorage.ts`, уже используется в `navToActivePath.ts`).
- Ключи localStorage должны включать `roomId` (двух аккаунтов в памяти не существует — атомы глобальны, поэтому user-scope не нужен; скоуп соответствует текущему in-memory поведению).
- Тесты: vitest, node-окружение (env не указан в конфиге), без jsdom/happy-dom — localStorage и window стабируются в тесте вручную.
- Коммиты в `desktop/` (подмодуль, ветка `harrier`) ссылаются на issue зонтика полным видом `schadin/harrier-project#39`.
- Версию приложения не поднимать. Push не выполнять без явной команды пользователя.

---

### Task 1: Unit-тесты для утилиты атомов с localStorage

**Files:**
- Create: `desktop/cinny/src/app/state/utils/atomWithLocalStorage.test.ts`

**Interfaces:**
- Consumes: `atomWithLocalStorage<T>(key, getItem, setItem)` из `./atomWithLocalStorage` (существующая сигнатура, менять не нужно)
- Produces: проверенное поведение утилиты: запись при `set`, чтение при создании атома, roundtrip `undefined`, синхронизация по `storage`-событию

- [ ] **Step 1: Написать падающий тест**

Создать файл `desktop/cinny/src/app/state/utils/atomWithLocalStorage.test.ts`:

```ts
import { createStore } from 'jotai';
import { afterEach, beforeEach, describe, expect, it, vi } from 'vitest';
import {
  atomWithLocalStorage,
  getLocalStorageItem,
  setLocalStorageItem,
} from './atomWithLocalStorage';

const storeKey = 'test.atomWithLocalStorage';

const getItemMock = (key: string) => getLocalStorageItem<string[]>(key, []);

let storage: Map<string, string>;

beforeEach(() => {
  storage = new Map();
  vi.stubGlobal('localStorage', {
    getItem: (key: string) => storage.get(key) ?? null,
    setItem: (key: string, value: string) => {
      storage.set(key, value);
    },
    removeItem: (key: string) => {
      storage.delete(key);
    },
  });
  vi.stubGlobal('window', { addEventListener: vi.fn(), removeEventListener: vi.fn() });
});

afterEach(() => {
  vi.unstubAllGlobals();
});

describe('atomWithLocalStorage', () => {
  it('использует значение из localStorage при создании атома', () => {
    storage.set(storeKey, JSON.stringify(['a', 'b']));
    const atom = atomWithLocalStorage<string[]>(storeKey, getItemMock, setLocalStorageItem);
    expect(createStore().get(atom)).toEqual(['a', 'b']);
  });

  it('использует default, когда ключа нет в localStorage', () => {
    const atom = atomWithLocalStorage<string[]>(storeKey, getItemMock, setLocalStorageItem);
    expect(createStore().get(atom)).toEqual([]);
  });

  it('пишет в localStorage при set', () => {
    const atom = atomWithLocalStorage<string[]>(storeKey, getItemMock, setLocalStorageItem);
    const store = createStore();
    store.set(atom, ['c']);
    expect(JSON.parse(storage.get(storeKey)!)).toEqual(['c']);
    expect(store.get(atom)).toEqual(['c']);
  });

  it('переживает roundtrip undefined (случай пустого reply-черновика)', () => {
    const atom = atomWithLocalStorage<string | undefined>(
      storeKey,
      (key) => getLocalStorageItem<string | undefined>(key, undefined),
      setLocalStorageItem
    );
    const store = createStore();
    store.set(atom, undefined);
    expect(store.get(atom)).toBeUndefined();
    const atom2 = atomWithLocalStorage<string | undefined>(
      storeKey,
      (key) => getLocalStorageItem<string | undefined>(key, undefined),
      setLocalStorageItem
    );
    expect(createStore().get(atom2)).toBeUndefined();
  });

  it('синхронизирует значение по storage-событию из другого окна', () => {
    const atom = atomWithLocalStorage<string[]>(storeKey, getItemMock, setLocalStorageItem);
    const store = createStore();
    store.get(atom);
    const [listener] = vi.mocked(window.addEventListener).mock.calls[0]!;
    storage.set(storeKey, JSON.stringify(['sync']));
    listener({ key: storeKey } as StorageEvent);
    expect(store.get(atom)).toEqual(['sync']);
  });
});
```

- [ ] **Step 2: Запустить тест и убедиться в падении**

Run: `cd desktop/cinny && npx vitest run src/app/state/utils/atomWithLocalStorage.test.ts`
Expected: FAIL (файл не существует — ошибка «No test files found»)

- [ ] **Step 3: Написать минимальную реализацию для прохождения теста**

Реализация уже существует (`atomWithLocalStorage.ts` + текущий `getLocalStorageItem`/`setLocalStorageItem`). Если после запуска теста есть падения — исправить утилиту минимально (например, обработка `undefined` в `setLocalStorageItem`, если roundtrip-тест не пройдёт).

- [ ] **Step 4: Запустить тест и убедиться в прохождении**

Run: `cd desktop/cinny && npm test -- src/app/state/utils/atomWithLocalStorage.test.ts`
Expected: PASS (5 тестов)

- [ ] **Step 5: Коммит**

```bash
cd desktop
git add cinny/src/app/state/utils/atomWithLocalStorage.test.ts
git commit -m "test: покрытие atomWithLocalStorage (schadin/harrier-project#39)"
```

---

### Task 2: Персист черновиков текста и reply в localStorage

**Files:**
- Modify: `desktop/cinny/src/app/state/room/roomInputDrafts.ts:29-57`

**Interfaces:**
- Consumes: `atomWithLocalStorage`, `getLocalStorageItem`, `setLocalStorageItem` из `../utils/atomWithLocalStorage`; `atomFamily` из `jotai/utils`
- Produces:
  - `roomIdToMsgDraftAtomFamily(roomId)` — тот же тип, что и сейчас: `WritableAtom<Descendant[], [Descendant[]], undefined>`
  - `roomIdToReplyDraftAtomFamily(roomId)` — `WritableAtom<IReplyDraft | undefined, [IReplyDraft | undefined], undefined>`
  - Ключи localStorage: `` `roomInputDrafts.msg.${roomId}` `` и `` `roomInputDrafts.reply.${roomId}` ``
  - Экспортированные для тестов/диагностики хелперы `getMsgDraftStorageKey(roomId)`, `getReplyDraftStorageKey(roomId)`

- [ ] **Step 1: Написать падающий тест**

Создать файл `desktop/cinny/src/app/state/room/roomInputDrafts.test.ts`:

```ts
import { createStore } from 'jotai';
import { afterEach, beforeEach, describe, expect, it, vi } from 'vitest';
import {
  getMsgDraftStorageKey,
  getReplyDraftStorageKey,
  roomIdToMsgDraftAtomFamily,
  roomIdToReplyDraftAtomFamily,
} from './roomInputDrafts';

let storage: Map<string, string>;

beforeEach(() => {
  storage = new Map();
  vi.stubGlobal('localStorage', {
    getItem: (key: string) => storage.get(key) ?? null,
    setItem: (key: string, value: string) => {
      storage.set(key, value);
    },
    removeItem: (key: string) => {
      storage.delete(key);
    },
  });
  vi.stubGlobal('window', { addEventListener: vi.fn(), removeEventListener: vi.fn() });
});

afterEach(() => {
  vi.unstubAllGlobals();
  roomIdToMsgDraftAtomFamily.remove('!roomA:example.org');
  roomIdToReplyDraftAtomFamily.remove('!roomA:example.org');
});

describe('roomIdToMsgDraftAtomFamily', () => {
  it('сохраняет черновик в localStorage при set и восстанавливает при новом атоме', () => {
    const draft = [{ type: 'paragraph', children: [{ text: 'черновик' }] }] as any;
    const store = createStore();
    const atom = roomIdToMsgDraftAtomFamily('!roomA:example.org');
    store.set(atom, draft);
    expect(JSON.parse(storage.get(getMsgDraftStorageKey('!roomA:example.org'))!)).toEqual(draft);

    roomIdToMsgDraftAtomFamily.remove('!roomA:example.org');
    const newAtom = roomIdToMsgDraftAtomFamily('!roomA:example.org');
    expect(createStore().get(newAtom)).toEqual(draft);
  });

  it('по умолчанию пустой массив для новой комнаты', () => {
    const atom = roomIdToMsgDraftAtomFamily('!roomA:example.org');
    expect(createStore().get(atom)).toEqual([]);
  });
});

describe('roomIdToReplyDraftAtomFamily', () => {
  it('сохраняет reply-черновик в localStorage и восстанавливает', () => {
    const reply = { userId: '@u:example.org', eventId: '$evt', body: 'цитата' };
    const store = createStore();
    store.set(roomIdToReplyDraftAtomFamily('!roomA:example.org'), reply);
    expect(JSON.parse(storage.get(getReplyDraftStorageKey('!roomA:example.org'))!)).toEqual(reply);

    roomIdToReplyDraftAtomFamily.remove('!roomA:example.org');
    const newAtom = roomIdToReplyDraftAtomFamily('!roomA:example.org');
    expect(createStore().get(newAtom)).toEqual(reply);
  });

  it('очистка черновика (undefined) сохраняется', () => {
    const store = createStore();
    const atom = roomIdToReplyDraftAtomFamily('!roomA:example.org');
    store.set(atom, { userId: '@u:example.org', eventId: '$evt', body: 'цитата' });
    store.set(atom, undefined);
    expect(createStore().get(roomIdToReplyDraftAtomFamily('!roomA:example.org'))).toBeUndefined();
  });
});
```

- [ ] **Step 2: Запустить тест и убедиться в падении**

Run: `cd desktop/cinny && npx vitest run src/app/state/room/roomInputDrafts.test.ts`
Expected: FAIL («getMsgDraftStorageKey is not defined» / атомы не пишут в localStorage)

- [ ] **Step 3: Реализовать персистенс в roomInputDrafts.ts**

Заменить блок строк 40-57 в `desktop/cinny/src/app/state/room/roomInputDrafts.ts`:

```ts
import {
  atomWithLocalStorage,
  getLocalStorageItem,
  setLocalStorageItem,
} from '../utils/atomWithLocalStorage';

const MSG_DRAFT_STORAGE_KEY = 'roomInputDrafts.msg';
const REPLY_DRAFT_STORAGE_KEY = 'roomInputDrafts.reply';

export const getMsgDraftStorageKey = (roomId: string) => `${MSG_DRAFT_STORAGE_KEY}.${roomId}`;
export const getReplyDraftStorageKey = (roomId: string) => `${REPLY_DRAFT_STORAGE_KEY}.${roomId}`;

const createMsgDraftAtom = (roomId: string) =>
  atomWithLocalStorage<Descendant[]>(
    getMsgDraftStorageKey(roomId),
    (key) => getLocalStorageItem<Descendant[]>(key, []),
    setLocalStorageItem
  );
export type TMsgDraftAtom = ReturnType<typeof createMsgDraftAtom>;
export const roomIdToMsgDraftAtomFamily = atomFamily<string, TMsgDraftAtom>((roomId) =>
  createMsgDraftAtom(roomId)
);

export type IReplyDraft = {
  userId: string;
  eventId: string;
  body: string;
  formattedBody?: string | undefined;
  relation?: IEventRelation | undefined;
};
const createReplyDraftAtom = (roomId: string) =>
  atomWithLocalStorage<IReplyDraft | undefined>(
    getReplyDraftStorageKey(roomId),
    (key) => getLocalStorageItem<IReplyDraft | undefined>(key, undefined),
    setLocalStorageItem
  );
export type TReplyDraftAtom = ReturnType<typeof createReplyDraftAtom>;
export const roomIdToReplyDraftAtomFamily = atomFamily<string, TReplyDraftAtom>((roomId) =>
  createReplyDraftAtom(roomId)
);
```

Не забыть добавить импорт утилит в начало файла (первая строка остаётся `import { atom } from 'jotai';` — он больше не нужен, проверить и удалить, если `atom` нигде не используется).

- [ ] **Step 4: Запустить тесты и убедиться в прохождении**

Run: `cd desktop/cinny && npm test`
Expected: PASS (все тесты, включая существующий `src/app/utils/decryptionFailure.test.ts`)

- [ ] **Step 5: Проверить восстановление и очистку в RoomInput (ручная проверка)**

Код `RoomInput.tsx` менять **не нужно** — механизм уже работает через существующие `setMsgDraft`/`setReplyDraft` (запись в localStorage происходит при set):
- Восстановление: `RoomInput.tsx:229-231` (`Transforms.insertFragment(editor, msgDraft)`) сработает с значением из localStorage на монтировании.
- Очистка при отправке: `RoomInput.tsx:376-379` — `resetEditor(editor)` + `setReplyDraft(undefined)`; на размонтировании `RoomInput.tsx:233-245` запишет `[]` (пустой editor) — старый черновик затрётся.

Ручная проверка:
1. `cd desktop && npm run tauri dev` (или `cd desktop/cinny && npm start`).
2. Ввести текст в комнату, перезапустить приложение — текст должен восстановиться.
3. Ответить на сообщение (reply), перезапустить — reply-панель должна восстановиться.
4. Отправить сообщение, перезапустить — поле ввода должно быть пустым.
5. Начать reply и нажать Escape, перезапустить — reply-панель должна отсутствовать.
6. Открыть DevTools → Application → Local Storage: ключи `roomInputDrafts.msg.*` / `roomInputDrafts.reply.*` на месте, корректный JSON.

- [ ] **Step 6: Коммит**

```bash
cd desktop
git add cinny/src/app/state/room/roomInputDrafts.ts cinny/src/app/state/room/roomInputDrafts.test.ts
git commit -m "feat: персист черновиков сообщений и reply между сессиями (schadin/harrier-project#39)"
```

---

### Task 3: Typecheck, lint и полный прогон тестов

**Files:**
- Нет изменений файлов.

**Interfaces:**
- Consumes: результат Task 2.

- [ ] **Step 1: Прогнать typecheck**

Run: `cd desktop/cinny && npm run typecheck`
Expected: без ошибок, связанных с `roomInputDrafts.ts` и `atomWithLocalStorage.test.ts` (известные базовые ошибки форка — `#31` — не относятся к диффу этой задачи; убедиться, что новых ошибок не добавлено, сравнив со списком до изменений).

- [ ] **Step 2: Прогнать полный набор тестов**

Run: `cd desktop/cinny && npm test`
Expected: PASS.

- [ ] **Step 3: Прогнать prettier для изменённых файлов**

Run: `cd desktop/cinny && npx prettier --check src/app/state/room/roomInputDrafts.ts src/app/state/room/roomInputDrafts.test.ts src/app/state/utils/atomWithLocalStorage.test.ts`
Expected: PASS (если нет — `npx prettier --write` на те же файлы и перезапустить).

- [ ] **Step 4: Коммит (если prettier что-то поправил)**

```bash
cd desktop
git add -u
git commit -m "style: форматирование после персиста черновиков (schadin/harrier-project#39)"
```

---

### Task 4: Обновление спецификации openspec/specs/messaging

**Files:**
- Modify: `openspec/specs/messaging/spec.md` (в зонтичном репозитории)

**Interfaces:**
- Consumes: терминология спецификации (список Terms существует, добавлять туда «Черновик» необязательно).
- Produces: Requirement «Персист черновиков между сессиями» с WHEN/THEN-сценариями.

- [ ] **Step 1: Добавить Requirement в spec.md**

Вставить новый раздел перед `### Requirement: Повторная отправка и локальное удаление` (строка 317 файла `openspec/specs/messaging/spec.md`):

```markdown
### Requirement: Персист черновиков между сессиями
Система SHALL сохранять локально (localStorage) черновик текстового сообщения и
reply-черновик для каждой комнаты и восстанавливать их после перезапуска
приложения. Ключи хранилища SHALL включать `roomId`. Очистка черновика происходит
при отправке сообщения, отмене reply (Escape/закрытие) и не переносится между
аккаунтами (скоуп соответствует in-memory атомам `roomIdToMsgDraftAtomFamily`).

#### Scenario: Черновик восстановлен после перезапуска
- **WHEN** пользователь вводит текст в комнату и перезапускает приложение, не отправляя сообщение
- **THEN** введённый текст восстанавливается в поле ввода этой комнаты

#### Scenario: Reply-черновик восстановлен после перезапуска
- **WHEN** пользователь начинает ответ (reply) и перезапускает приложение, не отправляя его
- **THEN** reply-панель и прикреплённая цитата восстанавливаются

#### Scenario: Черновик очищен после отправки
- **WHEN** пользователь отправляет сообщение с восстановленным черновиком
- **THEN** черновик очищается
- **AND** после перезапуска поле ввода пустое

#### Scenario: Черновики не синхронизируются с сервером
- **WHEN** пользователь вводит черновик
- **THEN** данные черновика не покидают устройство (не отправляются в account data)
```

- [ ] **Step 2: Просмотр и проверка спецификации**

Run: `cd /home/schadin/workspace/github/harrier-project && openspec validate --specs 2>/dev/null || echo "openspec CLI недоступен — спецификация проверена глазами"`
Expected: валидация проходит либо явный вывод о недоступности CLI; текст согласован с остальными Requirement (русский язык, WHEN/THEN).

- [ ] **Step 3: Коммит в зонтичном репозитории**

```bash
cd /home/schadin/workspace/github/harrier-project
git add openspec/specs/messaging/spec.md
git commit -m "spec: персист черновиков сообщений между сессиями (#39)"
```

---

## Self-Review

**1. Spec coverage:** Issue #39 требует (а) localStorage-персист обоих атомов — Task 2, (б) проверку очистки при отправке — Task 2 Step 5, (в) обновление спецификации — Task 4. Опциональный шаг про account data (синхронизация между устройствами) сознательно исключён: в Global Constraints зафиксирован скоуп «только локально», что соответствует замечанию issue.

**2. Placeholder scan:** Все шаги содержат конкретный код и команды; ручная проверка имеет пошаговые инструкции. Предупреждение в Task 3 Step 1 про базовые ошибки `#31` — не placeholder, а известное ограничение форка.

**3. Type consistency:** `TMsgDraftAtom = ReturnType<typeof createMsgDraftAtom>` сохраняет тип между Task 1-2 (тесты используют `store.set(atom, draft)` с `Descendant[]`), сигнатуры семейств атомов не меняются, поэтому `RoomInput.tsx`/`RoomTimeline.tsx` не требуют правок. Ключи `getMsgDraftStorageKey`/`getReplyDraftStorageKey` определены в Task 2 и используются в тестах того же файла — одни и те же имена в плане.