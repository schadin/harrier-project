# Tasks: fix-backup-restore-retry-loop

## 1. Реализация (`desktop/cinny/src/app/hooks/useRestoreBackupOnStartup.ts`)

- [ ] 1.1 Добавить `attemptedRef`: выставлять в `finally` первой попытки;
        guard — выход при уже совершённой попытке.
- [ ] 1.2 Сброс `attemptedRef` в колбэке `useKeyBackupDecryptionKeyCached`
        перед вызовом восстановления.
- [ ] 1.3 Перевести чтение статуса прогресса на `progressStatusRef`; убрать
        `restoreProgress.status` из deps колбэка.

## 2. Проверка

- [ ] 2.1 `npx tsc --noEmit` → 0 errors; `npm run check:eslint` → 0 errors.
- [ ] 2.2 Логика: ручной прогон сценариев (запуск без бэкапа — один проход).

## 3. Фиксация

- [ ] 3.1 Коммит в `desktop/cinny` со ссылкой schadin/harrier-project#43; бамп
        указателя в `desktop/`.
- [ ] 3.2 Архивирование change; коммит зонтика #43.
