## 1. Маппинг причин (задача #27)

- [x] 1.1 Создать функцию `getDecryptionFailureReason(code: DecryptionFailureCode | null)` → `{ titleKey: string; actionKey?: string }` со `switch` по всем кодам `DecryptionFailureCode` и дефолтом для `UNKNOWN_ERROR`/`null` (общий текст)
- [x] 1.2 Добавить юнит-тесты маппинга: `MEGOLM_UNKNOWN_INBOUND_SESSION_ID` → ключ ожидания + CTA «Запросить ключ»; `MEGOLM_KEY_WITHHELD_FOR_UNVERIFIED_DEVICE` → CTA «Верифицировать устройство»; `HISTORICAL_MESSAGE_*` → CTA «Бэкап»; коды без действия → без CTA

## 2. Рендер причины в контенте сообщения

- [x] 2.1 Прокинуть `decryptionFailureReason` и `getWireContent()` от `Message.tsx` до `RenderMessageContent` (ветка `m.bad.encrypted`, `RenderMessageContent.tsx:263`) как props, без чтения события в листе
- [x] 2.2 Заменить единую заглушку `MessageBadEncryptedContent` (`FallbackContent.tsx:34`) на компонент, отображающий локализованную причину по результату `getDecryptionFailureReason`
- [x] 2.3 Реализовать фолбэк: `null`/`UNKNOWN_ERROR` → текущий общий текст «Unable to decrypt message»

## 3. Контекстные действия (CTA)

- [x] 3.1 CTA «Запросить ключ» — колбэк-заглушка, скрытый/disabled до реализации #28
- [x] 3.2 CTA «Верифицировать устройство» — навигация к настройкам устройств (существующий `DeviceVerificationSetup`)
- [x] 3.3 CTA «Настроить/восстановить бэкап» — открытие панели `BackupRestore`
- [x] 3.4 Отсутствие CTA для причин без эффективного действия (например, `SENDER_IDENTITY_PREVIOUSLY_VERIFIED`)

## 4. Интернационализация

- [x] 4.1 Добавить i18n-ключи причины и CTA в файлы локалей `ru`, `en`, `de` (с английским фолбэком)

## 5. Проверка

- [x] 5.1 Прогнать `npm run lint` и сборку/typecheck форка `cinny`, убедиться в отсутствии регрессий рендера обычных и нерасшифрованных сообщений