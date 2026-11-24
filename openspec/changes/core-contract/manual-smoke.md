# Ручной desktop smoke — core-contract

Статус: **основной функционал проверен пользователем на реальном аккаунте — работает**.
Полный чек-лист ниже остаётся для расширенного smoke (crypto/media/calls и восстановление сессии).

## Подготовка

1. Собрать release: `cd desktop && npm run tauri build`.
2. Запустить `desktop/src-tauri/target/release/harrier`.
3. Использовать тестовый аккаунт и комнаты:
   - обычная комната;
   - зашифрованная комната;
   - пространство с дочерней комнатой;
   - DM;
   - комната с вложениями и уведомлениями.

## Сценарии

### Auth и session

- [ ] Вход по паролю.
- [ ] Вход по SSO (если homeserver поддерживает).
- [ ] Вход по login token (если доступен).
- [ ] Перезапуск приложения восстанавливает сессию.
- [ ] Logout очищает локальные данные и возвращает на экран входа.
- [ ] Ошибка сети показывает reconnect, retry восстанавливает sync.

### Rooms и spaces

- [ ] Список joined/invite комнат отображается без SDK-объектов в UI.
- [ ] Создание комнаты и пространства.
- [ ] Join/leave/invite.
- [ ] Иерархия пространств и переходы.
- [ ] Настройки комнаты: topic, avatar, join rules, aliases, permissions.
- [ ] Unread и mute/unmute обновляются после sync/receipt.

### Timeline и messaging

- [ ] Отправка, редактирование, reply, reaction, redaction.
- [ ] Локальный echo и статусы sending/sent/failed.
- [ ] Пагинация назад/вперёд и jump-to-event.
- [ ] Threads и pinned events.
- [ ] Поиск сообщений.
- [ ] Read receipts и read markers.

### Crypto

- [ ] Просмотр списка устройств.
- [ ] Rename/delete устройства, включая UIA.
- [ ] SAS-верификация устройства.
- [ ] Recovery key/passphrase и secret storage.
- [ ] Backup restore и progress.
- [ ] Key request и повторная расшифровка после получения ключа.
- [ ] Расшифрованное сообщение не теряет preview в уведомлении.

### Media

- [ ] Загрузка изображения, видео, аудио и файла.
- [ ] Progress и cancel upload.
- [ ] Thumbnail и encrypted attachment.
- [ ] Скачивание и просмотр image/pdf/text.
- [ ] URL preview.
- [ ] Authenticated media на homeserver с v1.11.

### Presence и notifications

- [ ] Online/offline/unavailable и last active.
- [ ] Typing.
- [ ] Custom status, presets, restore после перезапуска.
- [ ] Inbox notifications и invites.
- [ ] System notification, звук, favicon и tray badge.
- [ ] Private room и E2E preview.

### Calls

- [ ] Входящий/исходящий звонок.
- [ ] Join/leave и список участников.
- [ ] Speaking indicator.
- [ ] LiveKit service URL и WebRTC support.
- [ ] Hangup/close widget.

## Критерий завершения

Все доступные сценарии отмечены, ошибки записаны в issue #61. Пункты, требующие неподдерживаемой homeserver-функции, помечаются `N/A` с причиной.
