# Appearance Specification

## Purpose

Внешний вид приложения: брендинг Harrier (применяется патчем при сборке без
изменения подмодуля cinny), темы оформления и иконка приложения.

## Terms

- **Ребрендинг** — применение патча `0002-rebrand-harrier.patch` к подмодулю
  `cinny` через `scripts/rebrand.mjs`; обратная операция возвращает подмодуль
  в upstream-состояние (`git apply -R` + `git checkout --`).
- **Тема оформления** — палитра vanilla-extract, регистрируемая в
  `useThemes()`/`useThemeNames()`; каждая тема связана с классом подсветки кода
  Prism.
- **Мастер-иконка** — исходник `assets/icon-round.png` (1024×1024), из которого
  генерируется весь набор иконок через `npx tauri icon`.

## Requirements

### Requirement: Ребрендинг Cinny → Harrier при сборке
Система SHALL применять брендинг Harrier патчем `0002-rebrand-harrier.patch` через
`scripts/rebrand.mjs` (`rebrand:apply` / `rebrand:revert`) и откатывать его после
сборки — подмодуль `cinny` остаётся в upstream-состоянии. Применение идемпотентно
(проверка обратной применимостью патча). Затрагиваются:
About (логотип harrier.svg, «Powered by Cinny», удаление кнопки Support,
third-party notices с атрибуцией AGPL), Welcome, SplashScreen, AuthFooter/
AuthLayout/формы входа, SplashScreen, LocalBackup (экспорт `harrier-keys.txt`),
системные уведомления (`brand: 'Harrier'`), имя устройства «Harrier»
(`initial_device_display_name`), `index.html`, `manifest.json`. Логотипы
`assets/harrier*.svg` копируются в `cinny/public/res/svg/`, favicon-набор и
apple/android-иконки перезаписываются из ассетов. `productName`/`mainBinaryName`/
`identifier` → `Harrier`/`harrier`/`io.github.schadin.harrier`. Автообновление
удалено (updater, ключи подписи, `release.json` не используются).

#### Scenario: Сборка с брендингом
- **WHEN** выполняется `tauri build`
- **THEN** патч брендинга применяется, фронтенд собирается, патч откатывается
- **AND** бинарник идентифицируется как `Harrier` (`io.github.schadin.harrier`)

#### Scenario: Dev-режим
- **WHEN** выполняется `tauri dev`
- **THEN** патч применяется (`beforeDevCommand`), revert выполняется вручную после сессии

#### Scenario: Повторное применение
- **WHEN** `rebrand:apply` вызывается поверх уже применённого патча
- **THEN** операция не выполняется повторно (идемпотентность)

### Requirement: Темы Dracula и Alucard
Система SHALL поддерживать тёмную тему Dracula и светлую Alucard (официальные
цвета из dracula/dracula-theme). Палитры `draculaTheme`/`alucardTheme` в
`colors.css.ts`; темы регистрируются в `useThemes()`/`useThemeNames()`;
подсветка кода `.prism-dracula`/`.prism-alucard`. Активная тема учитывает
системную тему и настройки `lightThemeId`/`darkThemeId` с фолбэком на
Light/Dark.

#### Scenario: Выбор тёмной темы
- **WHEN** пользователь выбирает Dracula
- **THEN** применяется тёмная палитра Dracula и подсветка кода `.prism-dracula`

#### Scenario: Выбор светлой темы
- **WHEN** пользователь выбирает Alucard
- **THEN** применяется светлая палитра Alucard и подсветка кода `.prism-alucard`

#### Scenario: Смена системной темы
- **WHEN** включено следование системной теме и ОС переключает светлый/тёмный режим
- **THEN** активная тема переключается между `lightThemeId` и `darkThemeId`

### Requirement: Круглая иконка приложения
Система SHALL использовать круглую иконку (тёмный круг `#1D2229` + светлая птица)
вместо светлого квадрата. Мастер-исходник `assets/icon-round.png` (1024×1024,
прозрачный фон); весь набор (`src-tauri/icons/`, веб-ассеты favicon/apple/android)
перегенерируется через `npx tauri icon`.

#### Scenario: Иконка в системе
- **WHEN** приложение установлено
- **THEN** в системе отображается круглая иконка Harrier
