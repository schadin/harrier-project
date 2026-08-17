# Delivery Specification

## Purpose

Сборка и доставка релизов (Windows/Linux/macOS), публикация веб-клиента на GitHub
Pages, платформо-специфичные фиксы сборки и интернационализация интерфейса.

## Requirements

### Requirement: Релизные сборки
Система SHALL собирать нативные бинарники при публикации GitHub Release:
Windows (MSI), Linux (deb + AppImage), macOS (DMG + universal binary). На каждый
PR собирается проверочная сборка без релиза.

#### Scenario: Публикация релиза
- **WHEN** публикуется GitHub Release
- **THEN** собираются артефакты `Harrier_*` для всех платформ

### Requirement: Публикация веб-клиента на GitHub Pages
Система SHALL автоматически деплоить веб-клиент на GitHub Pages при публикации
release. `base` задаётся через `VITE_BASE`, версия — через `VITE_VERSION` (из тега
релиза); версия в UI через `__CINNY_VERSION__`. Деплой с `.nojekyll`,
`configure-pages`/`upload-pages-artifact`/`deploy-pages`.

#### Scenario: Релиз публикует веб-версию
- **WHEN** публикуется GitHub Release
- **THEN** веб-клиент собирается с `VITE_BASE=/cinny/` и деплоится на Pages

### Requirement: Сборка на ALT Linux
Система SHALL собираться на ALT Linux через shim `scripts/bin/pkg-config`:
для `--libs-only-L` с пустым выводом синтезируется `-L<libdir>` (обход бага
`output.stdout[2..]` в tauri-cli). npm-скрипт `tauri` запускается через
`scripts/tauri.mjs`, подставляющий `scripts/bin` в `PATH`.

#### Scenario: Бандлинг на ALT Linux
- **WHEN** выполняется `tauri build` на ALT Linux
- **THEN** shim pkg-config подставляет `-L` и бандлинг не паникует

### Requirement: Интернационализация интерфейса
Система SHALL поддерживать локализацию интерфейса (i18next + react-i18next +
LanguageDetector + http-backend). Инфраструктура встроена, но фактически не
используется — основная работа: извлечение захардкоженных строк в переводные ключи
и наполнение файлов локалей (`ru` + `en`/`de`). *(Реализация TK-00033 — в плане.)*

#### Scenario: Переключение языка
- **WHEN** пользователь выбирает язык в настройках
- **THEN** интерфейс переводится на выбранный язык
