# Tasks: реорганизация репозиториев (фаза −1 эпика #59)

## 1. Веб-legacy: автономия schadin/harrier

- [ ] 1.1 Применить `0002-rebrand-harrier.patch` к ветке `harrier` репозитория `schadin/harrier` постоянным коммитом
- [ ] 1.2 Перенести ассеты брендинга из `desktop/assets/` в `schadin/harrier` (`harrier*.svg` → `public/res/svg/`, favicon-набор, apple-touch-icon и android-chrome наборы → `public/res/`, иконки → `public/icons/`)
- [ ] 1.3 Удалить из `pages-deploy.yml` шаги клонирования `harrier-desktop` и запуска `rebrand.mjs` (apply/revert)
- [ ] 1.4 Проверить сборку веб-версии локально (`VITE_BASE=/harrier/ npm run build`) и деплой Pages после мержа
- [ ] 1.5 Обновить README `schadin/harrier`: статус «веб-legacy», только критические фиксы

## 2. Desktop: переезд клиента в монорепо

- [ ] 2.1 Выполнить subtree merge ветки `harrier` репозитория `schadin/harrier` в `desktop/client/` с сохранением истории; зафиксировать источник в сообщении коммита
- [ ] 2.2 Проверить сохранность истории (`git log --follow` на выборке файлов)
- [ ] 2.3 Применить брендинг к `client/` постоянным коммитом (патч + ассеты, как в 1.1–1.2)
- [ ] 2.4 Удалить подмодуль `cinny/` (`.gitmodules`, `git submodule deinit`, запись в `.git/modules`)
- [ ] 2.5 Обновить пути `cinny/` → `client/` в `tauri.conf.json` (`frontendDist`, `beforeBuildCommand`, `beforeDevCommand`, `devUrl`), `package.json`, workflows (`tauri.yml`, `test.yml`, `archive.yml` — убрать `submodules: true` и `cd cinny`)
- [ ] 2.6 Удалить `scripts/rebrand.mjs`, `0002-rebrand-harrier.patch`, npm-скрипты `rebrand:*` и шаг копирования `config.json`; оставить единый `client/config.json`
- [ ] 2.7 Упростить `scripts/update-version.mjs`: удалить фетч тегов и переключение подмодуля cinny
- [ ] 2.8 Полный поиск по строке `cinny` в desktop и зонтике; поправить оставшиеся ссылки
- [ ] 2.9 Обновить README desktop: структура монорепо, команды (`cd client && npm ci`, `npm run tauri dev`), инструкция по переинициализации локальных клонов (`git submodule deinit`)

## 3. Зонтик: документация и конфигурация

- [ ] 3.1 Обновить `docs/RELEASING.md`: релиз в одном репозитории, удалить шаг веб-релиза из обязательного чек-листа, описать автономные релизы веб-legacy
- [ ] 3.2 Обновить `AGENTS.md` зонтика: состав репозиториев, команды локальной разработки, удалить описание ребрендинга при сборке
- [ ] 3.3 Обновить `openspec/config.yaml`: репозиторная карта (монорепо desktop, веб-legacy), удалить особенность «ребрендинг при сборке»

## 4. Контрольные проверки

- [ ] 4.1 `npm run lint` и `npm run typecheck` в `client/` — 0 errors
- [ ] 4.2 `npm run tauri dev` стартует без шагов ребрендинга
- [ ] 4.3 `npm run tauri build` собирает релизные артефакты локально
- [ ] 4.4 PR-сборка (`test.yml`) зелёная на всех платформах
- [ ] 4.5 Контрольный предрелиз по новому процессу: артефакты desktop + Pages-деплой веб-legacy по прежнему URL
