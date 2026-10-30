# Delta: desktop-integration

## MODIFIED Requirements

### Requirement: Иконка трея — Harrier вместо Cinny
Система SHALL использовать Harrier-птицу как базовую иконку трея. Ассет
`client/public/icons/tray-base.png` SHALL храниться непосредственно в дереве
монорепозитория; применение или откат патча брендинга при сборке SHALL NOT
требоваться. Ошибка рендера иконки логируется и не приводит к падению приложения.

#### Scenario: Сборка с брендингом
- **WHEN** выполняется `tauri build`
- **THEN** трей-иконка использует Harrier-птицу из `client/public/icons/tray-base.png`
  без шагов ребрендинга
