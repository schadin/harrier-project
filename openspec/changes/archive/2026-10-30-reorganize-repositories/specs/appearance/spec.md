# Delta: appearance

## REMOVED Requirements

### Requirement: Ребрендинг Cinny → Harrier при сборке
**Reason**: Синк с upstream Cinny прекращён, подмодуль упразднён — подмодуль
больше не нужно удерживать в upstream-состоянии. Инфраструктура «применить
патч → собрать → откатить» (`scripts/rebrand.mjs`,
`0002-rebrand-harrier.patch`, шаги в `tauri.conf.json`) теряет смысл.
**Migration**: Брендинг применяется к коду клиента постоянным коммитом; ассеты
(логотипы, favicon-наборы, apple/android-иконки) добавляются в дерево.
Скрипт `rebrand.mjs` и патч удаляются.

## ADDED Requirements

### Requirement: Брендинг Harrier закоммичен в код клиента
Система SHALL хранить брендинг Harrier непосредственно в коде клиента:
About (логотип harrier.svg, «Powered by Cinny», удаление кнопки Support,
third-party notices с атрибуцией AGPL), Welcome, SplashScreen, AuthFooter/
AuthLayout/формы входа, LocalBackup (экспорт `harrier-keys.txt`), системные
уведомления (`brand: 'Harrier'`), имя устройства «Harrier»
(`initial_device_display_name`), `index.html`, `manifest.json`. Логотипы
`harrier*.svg`, favicon-набор и apple/android-иконки SHALL входить в дерево
клиента. `productName`/`mainBinaryName`/`identifier` SHALL быть
`Harrier`/`harrier`/`io.github.schadin.harrier`. Автообновление SHALL NOT
использоваться (updater, ключи подписи, `release.json` отсутствуют).
Сборка SHALL NOT требовать шагов применения/отката брендинга.

#### Scenario: Сборка с брендингом
- **WHEN** выполняется `tauri build` или сборка веб-версии
- **THEN** брендинг уже присутствует в исходниках, никакие патчи не применяются
- **AND** бинарник идентифицируется как `Harrier` (`io.github.schadin.harrier`)

#### Scenario: Dev-режим
- **WHEN** выполняется `tauri dev`
- **THEN** dev-сервер стартует без каких-либо шагов ребрендинга
