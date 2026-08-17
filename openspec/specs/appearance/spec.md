# Appearance Specification

## Purpose

Внешний вид приложения: брендинг Harrier (применяется патчем при сборке без
изменения подмодуля cinny), темы оформления и иконка приложения.

## Requirements

### Requirement: Ребрендинг Cinny → Harrier при сборке
Система SHALL применять брендинг Harrier патчем `0002-rebrand-harrier.patch` через
`scripts/rebrand.mjs` (`rebrand:apply` / `rebrand:revert`) и откатывать его после
сборки — подмодуль `cinny` остаётся в upstream-состоянии. Затрагиваются:
About/Welcome, AuthFooter/AuthLayout, SplashScreen, favicon/уведомления, имя
устройства «Harrier», `index.html`, `manifest.json`. Логотип `assets/harrier.svg`
копируется в `cinny/public/res/svg/`. `productName`/`mainBinaryName`/`identifier` →
`Harrier`/`harrier`/`io.github.schadin.harrier`. Автообновление удалено (updater,
ключи подписи, `release.json` не используются).

#### Scenario: Сборка с брендингом
- **WHEN** выполняется `tauri build`
- **THEN** патч брендинга применяется, фронтенд собирается, патч откатывается
- **AND** бинарник идентифицируется как `Harrier` (`io.github.schadin.harrier`)

#### Scenario: Dev-режим
- **WHEN** выполняется `tauri dev`
- **THEN** патч применяется (`beforeDevCommand`), revert выполняется вручную после сессии

### Requirement: Темы Dracula и Alucard
Система SHALL поддерживать тёмную тему Dracula и светлую Alucard (официальные
цвета из dracula/dracula-theme). Палитры `draculaTheme`/`alucardTheme` в
`colors.css.ts`; темы регистрируются в `useThemes()`/`useThemeNames()`;
подсветка кода `.prism-dracula`/`.prism-alucard`.

#### Scenario: Выбор тёмной темы
- **WHEN** пользователь выбирает Dracula
- **THEN** применяется тёмная палитра Dracula и подсветка кода `.prism-dracula`

#### Scenario: Выбор светлой темы
- **WHEN** пользователь выбирает Alucard
- **THEN** применяется светлая палитра Alucard и подсветка кода `.prism-alucard`

### Requirement: Круглая иконка приложения
Система SHALL использовать круглую иконку (тёмный круг `#1D2229` + светлая птица)
вместо светлого квадрата. Мастер-исходник `assets/icon-round.png` (1024×1024,
прозрачный фон); весь набор (`src-tauri/icons/`, веб-ассеты favicon/apple/android)
перегенерируется через `npx tauri icon`.

#### Scenario: Иконка в системе
- **WHEN** приложение установлено
- **THEN** в системе отображается круглая иконка Harrier
