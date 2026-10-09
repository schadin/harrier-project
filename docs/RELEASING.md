# Выпуск релизов и предрелизов

Процедура публикации сборок Harrier. Охватывает предрелизы (`-harrier.N-beta`)
и финальные релизы (без суффикса).

## Куда что публикуется

| Репозиторий | Что | Триггер |
|-------------|-----|---------|
| [`schadin/harrier-desktop`](https://github.com/schadin/harrier-desktop) | MSI, deb, AppImage, DMG, app.tar.gz | workflow `tauri.yml` на `release: published` |
| [`schadin/harrier-desktop`](https://github.com/schadin/harrier-desktop) | zip-архив исходников с подмодулями | workflow `archive.yml` на `release: published` |
| [`schadin/harrier`](https://github.com/schadin/harrier) | веб-клиент на GitHub Pages | workflow `pages-deploy.yml` на `release: published` |

**Релиз в `harrier-desktop` не обновляет веб-версию.** Workflow `pages-deploy.yml`
живёт в форке cinny и срабатывает только на публикации релиза в `schadin/harrier`.
Для каждого тега нужно создавать релиз в обоих репозиториях, иначе Pages останется
на предыдущей версии.

Имена артефактов в релизе `harrier-desktop` (6 файлов):

```
Harrier_desktop-x86_64.msi
Harrier_desktop-x86_64.deb
Harrier_desktop-x86_64.AppImage
Harrier_desktop-universal.dmg
Harrier_desktop-universal.app.tar.gz
harrier-desktop-<tag>.zip
```

## Release notes

### Ссылки на задачи

Задачи ведутся как issues в зонтике — `schadin/harrier-project`. В release notes
ссылка на задачу указывается **полной Markdown-ссылкой**:

```markdown
[#50](https://github.com/schadin/harrier-project/issues/50)
```

Короткая запись `#50` недопустима. GitHub резолвит `#NN` в issues того репозитория,
где опубликован релиз, то есть в `harrier-desktop#NN` — а issues в нём не ведутся
(0 issues), и ссылка ведёт в пустоту. Ранее использовавшиеся внутренние
идентификаторы (`TK-00037`) на GitHub не резолвятся и не используются.

Перед публикацией стоит проверить, что номера задач существуют:

```sh
gh issue view 50 --repo schadin/harrier-project
```

### Структура

```markdown
Предрелиз десктоп-клиента Harrier.

## DsTaskBot
- регулярные задачи (серии): карточка и уведомление с расписанием
  ([#50](https://github.com/schadin/harrier-project/issues/50));
- шорткаты-помощники `/dstask_<команда>`
  ([#50](https://github.com/schadin/harrier-project/issues/50)).

## Сборка
- числовой `bundle.windows.wix.version` для MSI-инсталлятора
  ([#52](https://github.com/schadin/harrier-project/issues/52)).

Артефакты (MSI, deb/AppImage, DMG) собираются workflow `tauri.yml`.
Полный список изменений — в
[коммитах](https://github.com/schadin/harrier-desktop/compare/<prev-tag>...<tag>).
```

Разделы группируются по фиче или домену. В конце — раздел «Сборка» для изменений
в сборке и инфраструктуре, затем ссылка на диапазон коммитов.

### Источник правды о составе сборки

Notes пишутся по коммитам между предыдущим и текущим тегом:

```sh
cd desktop
git log v<prev-tag>..v<new-tag> --format='%s (%h)'
```

Сверять нужно и с issues зонтика: задача могла быть закрыта несколькими коммитами
с обновлением подмодуля `cinny`.

## Чек-лист предрелиза

1. **Задачи.** Все задачи, вошедшие в сборку, закрыты. Для незакрытых явно
   указать это в notes. Номера в notes проверены через `gh issue view`.
2. **Качество кода.** В форке cinny ноль ошибок:
   ```sh
   cd desktop/cinny
   npm run lint
   npm run typecheck
   ```
3. **Версия.** Поднимается **только по явной команде пользователя**
   (`npm run bump <version>` в `desktop/`). Команда синхронизирует
   `package.json`, `src-tauri/Cargo.toml`, `Cargo.lock` и
   `src-tauri/tauri.conf.json`, переводит подмодуль `cinny` на последний тег
   и выводит числовой `wix.version` для MSI. Самостоятельно версию не поднимать.
4. **Коммит.** Сообщение со ссылкой на issue зонтика в полном виде:
   `chore: bump до <version> (schadin/harrier-project#NN)`.
5. **Тег и релиз.** Тег `v<version>`:
   ```sh
   cd desktop
   git tag v<version>
   git push origin harrier --tags
   gh release create v<version> --repo schadin/harrier-desktop \
     --title "Harrier v<version>" --notes-file /tmp/notes.md --prerelease
   ```
6. **Сборка артефактов.** Отследить три job'а `tauri.yml`
   (windows-x86_64, linux-x86_64, macos-universal) и `archive.yml`. Все должны
   завершиться успешно, в релизе должно быть 6 файлов по списку выше.
7. **Веб-релиз.** Создать релиз с тем же тегом в форке cinny — иначе Pages не
   обновится:
   ```sh
   gh release create v<version> --repo schadin/harrier \
     --title "Harrier v<version> (web)" --notes-file /tmp/notes-web.md --prerelease
   ```
8. **Проверка веба.** Workflow `pages-deploy.yml` завершился успешно, версия в UI
   соответствует тегу (`VITE_VERSION` подставляется из имени тега), ребрендинг
   применён.

## Чек-лист финального релиза

Отличия от предрелиза:

- версия без суффикса `-beta`, например `4.12.6` (только по команде пользователя);
- `wix.version` для MSI — числовой без номера сборки, так как pre-release
  отсутствует;
- notes собираются по всем предрелизам, вошедшим в релиз, — не по последнему
  диапазону коммитов;
- релиз создаётся без `--prerelease` в обоих репозиториях;
- задачи предрелизов закрыты в зонтике.

## Откат при сбое

Сборка привязана к тегу, и артефакты уже загружены, поэтому тег после публикации
не переиспользуется:

1. Удалить релиз и тег:
   ```sh
   gh release delete v<version> --repo schadin/harrier-desktop --yes
   git push origin --delete v<version>
   ```
2. Откатить коммит bump версии и запушить в `harrier`.
3. Внести исправление, поднять версию на следующий номер и пройти чек-лист заново.

Если сборка упала в одном из job'ов — релиз остаётся, дописывается только недостающее:
дождаться успешных job'ов и проверить наличие всех 6 файлов. Пересборка целиком
требует нового тега.