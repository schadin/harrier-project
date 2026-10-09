# Delivery Specification

## Purpose

Сборка и доставка релизов (Windows/Linux/macOS), публикация веб-клиента на GitHub
Pages, платформо-специфичные фиксы сборки, интернационализация интерфейса и
синхронное версионирование.

## Terms

- **Релизная сборка** — нативные артефакты, собираемые workflow `tauri.yml` при
  публикации GitHub Release; именуются `Harrier_desktop-*`.
- **PR-сборка** — проверочная сборка workflow `test.yml` на каждый pull request,
  без публикации в релиз.
- **Shim pkg-config** — обёртка `scripts/bin/pkg-config`, подставляемая в `PATH`
  через `scripts/tauri.mjs`.
- **Bump** — синхронное поднятие версии через `npm run bump <version>`.
## Requirements
### Requirement: Релизные сборки
Система SHALL собирать нативные бинарники при публикации GitHub Release
(workflow `tauri.yml`): Windows (MSI, `Harrier_desktop-x86_64.msi`), Linux
(deb + AppImage, ubuntu-22.04 с зависимостями webkit2gtk/appindicator/rsvg),
macOS (DMG + tar.gz universal binary `--target universal-apple-darwin`).
Версия парсится из `tauri.conf.json`; исходники с подмодулями архивируются
в релиз (workflow `archive.yml`). На каждый PR собирается проверочная сборка
(workflow `test.yml`) без релиза. Все actions запинены по SHA.

#### Scenario: Публикация релиза
- **WHEN** публикуется GitHub Release
- **THEN** собираются артефакты `Harrier_desktop-*` для всех платформ

#### Scenario: Проверочная сборка PR
- **WHEN** открывается/обновляется pull request
- **THEN** артефакты всех платформ собираются как `release-artifacts-*`

### Requirement: Публикация релиза в двух репозиториях
Система SHALL требовать создания GitHub Release с одним и тем же тегом в двух
репозиториях: `schadin/harrier-desktop` (нативные артефакты и zip-архив) и
`schadin/harrier` (веб-клиент). Workflow `pages-deploy.yml` находится в форке
cinny и SHALL срабатывать только на публикации релиза в `schadin/harrier`,
поэтому публикация релиза только в `schadin/harrier-desktop` SHALL считаться
неполной: версия на GitHub Pages не обновляется. В релизе `harrier-desktop`
SHALL присутствовать шесть артефактов: MSI, deb, AppImage, DMG,
app.tar.gz и zip-архив исходников. Тег SHALL NOT переиспользоваться после
публикации: артефакты уже загружены, пересборка требует нового тега.

#### Scenario: Полный релиз обновляет и десктоп, и веб
- **WHEN** выходит предрелиз или релиз
- **THEN** релиз с тегом `v<version>` создан и в `schadin/harrier-desktop`,
  и в `schadin/harrier`

#### Scenario: Релиз только в десктопном репозитории
- **WHEN** создан релиз только в `schadin/harrier-desktop`
- **THEN** публикация считается неполной и версия на GitHub Pages остаётся
  на предыдущей

### Requirement: Публикация веб-клиента на GitHub Pages
Система SHALL автоматически деплоить веб-клиент на GitHub Pages при публикации
release (workflow `pages-deploy.yml` в форке cinny). Перед сборкой применяется
ребрендинг из ветки `harrier` harrier-desktop (`REBRAND_CINNY_DIR`), после
сборки revert выполняется всегда (`if: always()`). `base` задаётся через
`VITE_BASE=/harrier/`, версия — через `VITE_VERSION` (из тега релиза); версия
в UI через `__CINNY_VERSION__`. Деплой с `.nojekyll`,
`configure-pages`/`upload-pages-artifact`/`deploy-pages`, concurrency group `pages`.

#### Scenario: Релиз публикует веб-версию
- **WHEN** публикуется GitHub Release
- **THEN** веб-клиент собирается с брендингом, `VITE_BASE=/harrier/`
  и деплоится на Pages

### Requirement: Сборка на ALT Linux
Система SHALL собираться на ALT Linux через shim `scripts/bin/pkg-config`:
для `--libs-only-L` с пустым выводом синтезируется `-L<libdir>` (обход бага
`output.stdout[2..]` в tauri-cli); остальные вызовы прозрачны. npm-скрипт
`tauri` копирует `config.json` в cinny и запускается через `scripts/tauri.mjs`,
подставляющий `scripts/bin` в начало `PATH` и корректно пересылающий сигналы
(SIGINT/SIGTERM) и код выхода дочернему процессу.

#### Scenario: Бандлинг на ALT Linux
- **WHEN** выполняется `tauri build` на ALT Linux
- **THEN** shim pkg-config подставляет `-L` и бандлинг не паникует

### Requirement: Интернационализация интерфейса
Система SHALL поддерживать локализацию интерфейса (i18next + react-i18next +
LanguageDetector + http-backend; `fallbackLng: 'en'`, `load: 'languageOnly'`,
локали `en`/`ru`/`de` загружаются по `${BASE_URL}/public/locales/{{lng}}.json`).
Инфраструктура встроена; покрытие частичное — локализованы причины неудачной
расшифровки (DecryptionFailure) и базовые строки ленты, основная работа:
извлечение захардкоженных строк в переводные ключи и наполнение файлов локалей.
*(Реализация #22 — в плане.)*

#### Scenario: Переключение языка
- **WHEN** пользователь выбирает язык в настройках
- **THEN** переведённые строки интерфейса отображаются на выбранном языке,
  отсутствующие ключи — фолбэк на английский

### Requirement: Release notes предрелиза и релиза
Система SHALL оформлять release notes с ссылками на задачи зонтика в виде полных
Markdown-ссылок `[#NN](https://github.com/schadin/harrier-project/issues/NN)`.
Короткая запись `#NN` SHALL использоваться в release notes, публикуемых в
`schadin/harrier-desktop` и `schadin/harrier`, поскольку GitHub резолвит её в
issues репозитория публикации, а issues в этих репозиториях не ведутся.
Внутренние идентификаторы задач (`TK-NNNNN`) SHALL NOT использоваться.
Состав изменений SHALL сверяться по коммитам диапазона предыдущий тег → текущий
тег. Полный процесс выпуска (чек-листы предрелиза и финального релиза, создание
релиза в обоих репозиториях, откат при сбое) SHALL быть описан в
`docs/RELEASING.md` зонтичного репозитория.

#### Scenario: Ссылки на задачи в release notes
- **WHEN** публикуется предрелиз или релиз
- **THEN** каждая задача в release notes указана полной Markdown-ссылкой на issue
  зонтика `schadin/harrier-project`

#### Scenario: Задачи в notes соответствуют issues зонтика
- **WHEN** готовится публикация релиза
- **THEN** номера задач из release notes проверены существованием issues в
  зонтичном репозитории

### Requirement: Синхронное версионирование
Система SHALL синхронизировать версию приложения между package.json,
Cargo.toml/Cargo.lock и tauri.conf.json командой `npm run bump <version>`
(без аргумента — ошибка). Bump также обновляет подмодуль cinny до последнего
тега. Версия изменяется только по явной команде пользователя.

Дополнительно bump SHALL выводить числовую версию
`bundle.windows.wix.version` для MSI-инсталлятора из semver-версии: WiX не
принимает человекочитаемый pre-release, поэтому номер сборки переносится в
четвёртое поле (`4.12.6-harrier.6-beta` → `4.12.6.6`). macOS и Linux собираются
из полной semver-версии.

#### Scenario: Подъём версии
- **WHEN** выполняется `npm run bump <version>`
- **THEN** версия обновляется во всех трёх манифестах, Cargo.lock переустановлен,
  подмодуль cinny указывает на последний тег, а `bundle.windows.wix.version`
  содержит числовое значение, выведенное из pre-release

### Requirement: Чистые гейты качества кода
Система SHALL поддерживать `npm run lint` и `npm run typecheck` в форке cinny
в состоянии «0 errors» на базовой ревизии, чтобы они могли использоваться как
гейты проверки любых доработок. Типовая резолюция TypeScript SHALL
использовать режим `moduleResolution: "Bundler"` (совместимый с поставкой
`.d.ts` matrix-js-sdk с `.ts`-суффиксами). Для легитимных идиом крипто-кода
(`src/util/cryptE2ERoomKeys.js`) правила eslint (no-bitwise, no-plusplus,
prefer-template, no-param-reassign) отключаются override'ом конфига.
Глобалы Vite define (например, `__CINNY_VERSION__`) SHALL быть объявлены в
eslint-конфиге.

#### Scenario: Гейт проверки доработки
- **WHEN** выполняются `npm run lint` и `npm run typecheck` в форке cinny
- **THEN** обе команды завершаются без ошибок (0 errors)

#### Scenario: Крипто-код не блокирует lint
- **WHEN** lint обрабатывает `src/util/cryptE2ERoomKeys.js`
- **THEN** битовые операции и инкременты не дают ошибок (override)

