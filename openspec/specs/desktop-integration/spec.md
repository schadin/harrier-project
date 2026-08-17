# Desktop Integration Specification

## Purpose

Нативная интеграция с десктоп-окружением: системный трей, скрытие заголовка окна
под Niri и миграция настроек из Cinny в Harrier при первом запуске.

## Requirements

### Requirement: Системный трей
Система SHALL предоставлять иконку в системном трее со сменой статуса и бейджем
непрочитанных сообщений. Меню: Open → статусы-галочки → Quit. Иконка рендерится
фронтендом (база + emoji статуса + бейдж) и передаётся через `set_tray_icon`.
Показ определяется настройкой `showTrayIcon` (дефолт `false`). Фоновое поведение
(`startMinimized`, `minimizeOnClose`, перехват `CloseRequested`) не переносится —
закрытие окна завершает приложение.

#### Scenario: Включение трея
- **WHEN** пользователь включает «Show Tray Icon» в настройках Desktop
- **THEN** в трее появляется иконка Harrier с меню статусов

#### Scenario: Смена статуса из трея
- **WHEN** пользователь выбирает статус в меню трея
- **THEN** статус устанавливается и иконка обновляется (emoji статуса)

#### Scenario: Бейдж непрочитанных
- **WHEN** есть непрочитанные сообщения
- **THEN** на иконке трея отображается бейдж с количеством

### Requirement: Иконка трея — Harrier вместо Cinny
Система SHALL при сборке подставлять в трей Harrier-птицу вместо Cinny. Базовая
иконка `assets/tray-base.png` перезаписывает `public/icons/tray-base.png` через
`rebrand:apply`, revert восстанавливает upstream из git подмодуля.

#### Scenario: Сборка с брендингом
- **WHEN** выполняется ребрендинг при сборке
- **THEN** трей-иконка использует Harrier-птицу

### Requirement: Скрытие заголовка окна под Niri
Система SHALL при наличии переменной окружения `NIRI_SOCKET` отключать
decorations/shadow окна и скрывать GTK-titlebar (`set_titlebar(None)` +
`set_decorated(false)`).

#### Scenario: Запуск под Niri
- **WHEN** приложение запускается при установленном `NIRI_SOCKET`
- **THEN** у окна нет GTK-заголовка

### Requirement: Миграция настроек Cinny → Harrier
Система SHALL при первом старте копировать (не перемещать) настройки
`in.cinny.app` → `io.github.schadin.harrier` (config + data каталоги, XDG на Linux),
создавать маркер `migration.success`. Ничего не удаляется и не перезаписываются
существующие новые данные.

#### Scenario: Первый запуск после Cinny
- **WHEN** Harrier запускается впервые и находит настройки Cinny
- **THEN** настройки копируются в каталог Harrier без удаления исходных
- **AND** создаётся маркер `migration.success`
