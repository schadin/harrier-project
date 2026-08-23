# delivery Delta

## ADDED Requirements

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
