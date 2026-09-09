# CONFIGURATION

Срез локального исходного кода: 2026-09-08. Корень ссылок — `/home/admin1/posa`; локальные незакоммиченные изменения включены. Источники — исполняемый код, схемы и конфигурация; существующая документация и комментарии не используются как доказательства. Статус фактов ниже: **подтверждено кодом**, если явно не указано «предположение на основе имени и использования». Статический анализ не подтверждает состояние рабочей БД или успешный запуск.

## Порядок загрузки

`dotenv.config()` вызывается в index.js, mailer.js, smsService.js и seed-kassa.js. Часть конфигурации считывается на уровне модуля при ESM-импорте. Инициализация зависит от графа импортов; Docker явно передаёт `.env` через env_file. Для воспроизводимого запуска следует передать окружение до импорта приложения. `activation-system/index.js:27` `activation-system/utils/mailer.js:3` `activation-system/utils/smsService.js:5` `activation-system/docker-compose.yml:9`.

`.env.example` не найден в исследуемом дереве. Поэтому все чтения ниже отсутствуют в example; обратный список «example → код» пуст. Для практической сверки отдельно указано наличие имени в локальном `.env`; значения не публикуются. Базовые контракты конфигурации: `activation-system/index.js:29` `activation-system/prisma/schema.prisma:7`.

Обязательность обозначает требование конкретного пути исполнения. «Нужна для интеграции» не равнозначно обязательной проверке при старте.

| Переменная | Тип | Обязательность / default | Назначение | Где читается | Локальный .env |
| --- | --- | --- | --- | --- | --- |
| APPLE_WALLET_SIGNER_CERT_PATH | путь | обязательна при импорте; файл должен существовать | PEM сертификат/ключ для PKPass; проверяется mustEnv/mustFile | `activation-system/routes/wallet.js:31` | `activation-system/.env:24` |
| APPLE_WALLET_SIGNER_KEY_PATH | путь | обязательна при импорте; файл должен существовать | PEM сертификат/ключ для PKPass; проверяется mustEnv/mustFile | `activation-system/routes/wallet.js:32` | `activation-system/.env:25` |
| APPLE_WWDR_CERT_PATH | путь | обязательна при импорте; файл должен существовать | PEM сертификат/ключ для PKPass; проверяется mustEnv/mustFile | `activation-system/routes/wallet.js:30` | `activation-system/.env:23` |
| APP_BASE_URL | URL | нет | Fallback после PUBLIC_BASE_URL для клиентских ссылок | `activation-system/controllers/client/api/voucherController.js:484`; `activation-system/index.js:170` | отсутствует |
| BASE_URL | URL | нет; fallback https://wallet.namo.uz | QR payment link и база чека; не единый с PUBLIC_BASE_URL | `activation-system/controllers/admin/qrLinkController.js:64`; `activation-system/controllers/admin/qrLinkController.js:102`; `activation-system/controllers/public/qrPaymentController.js:403` | отсутствует |
| CLICK_ENV | строка | нет; production | Читается, но baseURL не зависит от значения | `activation-system/utils/payment/clickSignature.js:48` | `activation-system/.env:47` |
| CLICK_MERCHANT_ID | строка | fallback если поле Kassa не заполнено | Click URL/MD5 | `activation-system/prisma/seed-kassa.js:33`; `activation-system/utils/payment/clickSignature.js:44` | `activation-system/.env:44` |
| CLICK_MERCHANT_USER_ID | строка | fallback если поле Kassa не заполнено | Click URL/MD5 | `activation-system/utils/payment/clickSignature.js:46` | `activation-system/.env:46` |
| CLICK_SECRET_KEY | секретная строка | fallback если поле Kassa не заполнено | Click URL/MD5 | `activation-system/prisma/seed-kassa.js:35`; `activation-system/utils/payment/clickSignature.js:20` | `activation-system/.env:45` |
| CLICK_SERVICE_ID | строка | fallback если поле Kassa не заполнено | Click URL/MD5 | `activation-system/prisma/seed-kassa.js:34`; `activation-system/utils/payment/clickSignature.js:45` | `activation-system/.env:43` |
| CLIENT_ENTER_URL | URL/путь | нет; /client/enter-voucher | Redirect не-iOS; serial передаётся query | `activation-system/routes/wallet.js:166`; `activation-system/routes/wallet.js:184` | `activation-system/.env:35` |
| DATABASE_URL | PostgreSQL URL | да для Prisma и session store | Общая БД приложения и сессий | `activation-system/index.js:111`; `activation-system/prisma/schema.prisma:7` | `activation-system/.env:3` |
| DRWEB_API_KEY | секретная строка | нет технически; литеральный fallback | Bearer для /generate | `activation-system/services/drwebService.js:5` | `activation-system/.env:63` |
| DRWEB_API_URL | URL | нет; http://localhost:8001/drweb/api/v1 | База генерации Dr.Web | `activation-system/services/drwebService.js:4` | `activation-system/.env:62` |
| DRWEB_MOCK | boolean: строка true | нет; false | Генерация mock serial вместо API | `activation-system/services/drwebService.js:6` | отсутствует |
| EMAIL_HOST | hostname | нужна для SMTP | SMTP host; secure всегда true | `activation-system/utils/mailer.js:6` | `activation-system/.env:8` |
| EMAIL_PASS | секретная строка | нужна для SMTP | SMTP auth password | `activation-system/utils/mailer.js:11` | `activation-system/.env:2`, `activation-system/.env:7` |
| EMAIL_PORT | целое через parseInt | нужна для SMTP | SMTP port; default не задан | `activation-system/utils/mailer.js:7` | `activation-system/.env:9` |
| EMAIL_USER | строка | нужна для SMTP | SMTP auth user и From | `activation-system/utils/mailer.js:10`; `activation-system/utils/mailer.js:17` | `activation-system/.env:6` |
| ESKIZ_API_URL | URL | нет; https://notify.eskiz.uz/api | База /auth/login и /message/sms/send | `activation-system/utils/smsService.js:10` | `activation-system/.env:13` |
| ESKIZ_EMAIL | строка | проверяется при получении токена | Логин Eskiz | `activation-system/utils/smsService.js:8` | `activation-system/.env:11` |
| ESKIZ_PASSWORD | секретная строка | проверяется при получении токена | Пароль Eskiz | `activation-system/utils/smsService.js:9` | `activation-system/.env:12` |
| ESKIZ_SENDER_NAME | строка | нет; 4546 | SMS from | `activation-system/utils/smsService.js:11` | `activation-system/.env:14` |
| MANASTORE_API_KEY_PRODUCTION | секретная строка | обязательна при старте выбранного контура без mock | Bearer ManaStore | `activation-system/services/manastoreService.js:15` | `activation-system/.env:77` |
| MANASTORE_API_KEY_STAGING | секретная строка | обязательна при старте выбранного контура без mock | Bearer ManaStore | `activation-system/services/manastoreService.js:10` | `activation-system/.env:73` |
| MANASTORE_API_URL_PRODUCTION | URL | нет; URL соответствующего контура | BaseURL ManaStore | `activation-system/services/manastoreService.js:14` | `activation-system/.env:76` |
| MANASTORE_API_URL_STAGING | URL | нет; URL соответствующего контура | BaseURL ManaStore | `activation-system/services/manastoreService.js:9` | `activation-system/.env:72` |
| MANASTORE_ENV | staging/production | нет; staging | Выбирает контур; любое значение кроме production означает staging | `activation-system/controllers/admin/manastoreController.js:4`; `activation-system/services/manastoreService.js:30` | `activation-system/.env:69` |
| MANASTORE_MOCK | boolean: строка true | нет; false | Mock-интеграция; снимает startup-проверку credentials | `activation-system/services/manastoreService.js:4` | `activation-system/.env:70` |
| MANASTORE_TIMEOUT_MS | целое мс через parseInt | нет; 30000 | Axios timeout; повтор на сетевой ошибке/5xx один раз | `activation-system/services/manastoreService.js:5` | `activation-system/.env:71` |
| MANASTORE_WALLET_ID_PRODUCTION | строка | обязательна при старте выбранного контура без mock | wallet_id создания заказа | `activation-system/services/manastoreService.js:16` | `activation-system/.env:78` |
| MANASTORE_WALLET_ID_STAGING | строка | обязательна при старте выбранного контура без mock | wallet_id создания заказа | `activation-system/services/manastoreService.js:11` | `activation-system/.env:75` |
| MANASTORE_WEBHOOK_SECRET | тип не определён кодом | не используется | Имя есть в .env; чтение и webhook ManaStore не реализованы | чтение не найдено | `activation-system/.env:79` |
| NODE_ENV | строка | нет | production включает secure cookie; сравнивается также в ManaStore | `activation-system/index.js:121`; `activation-system/services/manastoreService.js:45`; `activation-system/utils/authTokens.js:10` | отсутствует |
| ORG_NAME | строка | обязательна при импорте, непустая | Метаданные PassKit | `activation-system/routes/wallet.js:44` | `activation-system/.env:19` |
| PASS_BASE_URL | URL | нет | Отдельная база ссылок на pkpass | `activation-system/controllers/client/api/voucherController.js:501` | отсутствует |
| PASS_KEY_PASSWORD | секретная строка | обязательна при импорте, непустая | PassKit passphrase | `activation-system/routes/wallet.js:33` | `activation-system/.env:26` |
| PASS_TYPE_ID | строка | обязательна при импорте, непустая | Метаданные PassKit | `activation-system/routes/wallet.js:42` | `activation-system/.env:17` |
| PAYMENT_BASE_URL | URL | нет; https://wallet.namo.uz | Return/callback URLs оплаты | `activation-system/controllers/kassa/kassaCabinetController.js:270`; `activation-system/controllers/public/qrPaymentController.js:162` | `activation-system/.env:54` |
| PAYME_ENV | строка | нет; test | test выбирает test.paycom.uz и test key; иначе production URL/key | `activation-system/utils/payment/paymeHelpers.js:11`; `activation-system/utils/payment/paymeHelpers.js:40` | `activation-system/.env:52` |
| PAYME_KEY | секретная строка | fallback если поле Kassa не заполнено | Payme URL/Basic auth; seed копирует часть полей | `activation-system/prisma/seed-kassa.js:39`; `activation-system/utils/payment/paymeHelpers.js:43` | `activation-system/.env:50` |
| PAYME_MERCHANT_ID | строка | fallback если поле Kassa не заполнено | Payme URL/Basic auth; seed копирует часть полей | `activation-system/prisma/seed-kassa.js:38`; `activation-system/utils/payment/paymeHelpers.js:9` | `activation-system/.env:49` |
| PAYME_TEST_KEY | секретная строка | fallback если поле Kassa не заполнено | Payme URL/Basic auth; seed копирует часть полей | `activation-system/prisma/seed-kassa.js:40`; `activation-system/utils/payment/paymeHelpers.js:42` | `activation-system/.env:51` |
| PORT | порт/строка | нет; default 4000 | Порт app.listen | `activation-system/index.js:29` | `activation-system/.env:1` |
| PUBLIC_BASE_URL | URL | условно, при отсутствии origin | База QR, клиентских ссылок и PDF; приоритет зависит от вызываемой функции | `activation-system/controllers/client/api/voucherController.js:484`; `activation-system/controllers/public/qrPaymentController.js:403`; `activation-system/index.js:169`; `activation-system/routes/wallet.js:79`; `activation-system/utils/qr.js:25` | `activation-system/.env:32` |
| QR_SIGN_SECRET | секретная строка | fallback QR_TOKEN_SECRET | Второе имя HMAC secret | `activation-system/utils/qr.js:5` | `activation-system/.env:34` |
| QR_TOKEN_SECRET | секретная строка | для подписанного QR | Приоритетный HMAC-SHA256 secret; без обоих secrets smart URL содержит voucher query | `activation-system/utils/qr.js:5` | `activation-system/.env:33` |
| ROKKY_API_URL | URL | нет; https://api.rokky.com/v1 | База недостижимой real API ветки | `activation-system/services/rokkyService.js:4` | отсутствует |
| ROKKY_MOCK | строка | нет | Выражение === true &#124;&#124; true всегда включает mock независимо от значения | `activation-system/services/rokkyService.js:8` | отсутствует |
| ROKKY_PARTNER_CODE | строка | нет | Прочитана в константу; далее не используется | `activation-system/services/rokkyService.js:5` | отсутствует |
| ROKKY_PASSWORD | секретная строка | real API ветка недостижима | Basic password | `activation-system/services/rokkyService.js:7` | отсутствует |
| ROKKY_USERNAME | строка | real API ветка недостижима | Basic login | `activation-system/services/rokkyService.js:6` | отсутствует |
| SESSION_SECRET | секретная строка | нет технически; есть литеральный fallback | Подпись сессии; небезопасный default описан в GAPS | `activation-system/index.js:115` | `activation-system/.env:64` |
| TEAM_ID | строка | обязательна при импорте, непустая | Метаданные PassKit | `activation-system/routes/wallet.js:43` | `activation-system/.env:18` |
| WALLET_VAPID_PUBLIC_KEY | строка | нет; null | Публичный ключ браузерной push-подписки; отправитель push не реализован | `activation-system/controllers/client/dashboardController.js:80`; `activation-system/controllers/client/dashboardController.js:89`; `activation-system/routes/clientRoutes.js:42`; `activation-system/routes/clientRoutes.js:48` | отсутствует |
| WALLET_WEB_SERVICE_URL | URL | нет | Включает webServiceURL только вместе с dbVoucher.authToken; поля нет в схеме | `activation-system/routes/wallet.js:45` | `activation-system/.env:29` |


## Разрывы конфигурации

Прочитаны в коде, отсутствуют в локальном `.env`: `APP_BASE_URL`, `BASE_URL`, `DRWEB_MOCK`, `NODE_ENV`, `PASS_BASE_URL`, `ROKKY_API_URL`, `ROKKY_MOCK`, `ROKKY_PARTNER_CODE`, `ROKKY_PASSWORD`, `ROKKY_USERNAME`, `WALLET_VAPID_PUBLIC_KEY`. Источники каждого чтения — таблица выше.

Есть в `.env`, чтение не найдено: `MANASTORE_WEBHOOK_SECRET` `activation-system/.env:79`.

Повторные имена в `.env`: `EMAIL_PASS` (2, 7).

## Конфигурация в БД и файлах

Платёжные credentials также хранятся в Kassa; Alif читает исключительно выбранные sandbox/production поля кассы. Telegram token и authorizedUsers хранятся в TelegramBot. `activation-system/prisma/schema.prisma:610` `activation-system/controllers/payment/alifController.js:29` `activation-system/prisma/schema.prisma:502`.

Docker: PostgreSQL 16; database credentials заданы литералами в compose; named volume postgres_data; backend bind mount .:/app; публикации 4000:4000, 7440:5432 и 127.0.0.1:5557:5555. Override добавляет 127.0.0.1:4001:4000, а не доказывает закрытие исходного порта. `activation-system/docker-compose.yml:7` `activation-system/docker-compose.yml:33` `activation-system/docker-compose.override.yml:5`.
