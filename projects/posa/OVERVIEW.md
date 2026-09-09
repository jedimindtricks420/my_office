# OVERVIEW

Срез локального исходного кода: 2026-09-08. Корень ссылок — `/home/admin1/posa`; локальные незакоммиченные изменения включены. Источники — исполняемый код, схемы и конфигурация; существующая документация и комментарии не используются как доказательства. Статус фактов ниже: **подтверждено кодом**, если явно не указано «предположение на основе имени и использования». Статический анализ не подтверждает состояние рабочей БД или успешный запуск.

## Назначение

Система ведёт каталог цифровых товаров, запас ваучеров, продажи мерчантов, привязку ваучеров к клиентам и выдачу активационных кодов. Предусмотрены административный кабинет, кабинет поставщика, кассы, клиентский кошелёк и страницы магазина. Это назначение выведено из операций, а не из названий документации. `activation-system/routes/adminRoutes.js:69` `activation-system/routes/merchantRoutes.js:20` `activation-system/routes/clientApiRoutes.js:7` `activation-system/routes/storeRoutes.js:7`.

Мерчант продаёт один ваучер за подтверждение корзины: берётся `cart[0]`, создаются Sale и VoucherTransaction, корректируются Vendor/Merchant/Kassa. ONLINE добавляет OnlineVoucher и отправку SMS; OFFLINE создаёт PDF. `activation-system/controllers/merchant/saleController.js:126` `activation-system/controllers/merchant/saleController.js:156` `activation-system/controllers/merchant/saleController.js:251`.

Публичная QR-ссылка связывает мерчанта и товар. Checkout создаёт попытку оплаты и redirect в Click, Payme или Alif. Callback вызывает общий процесс продажи. `activation-system/prisma/schema.prisma:549` `activation-system/controllers/public/qrPaymentController.js:111` `activation-system/routes/paymentRoutes.js:9`.

Поставщики имеют типы ROKKY, DRWEB, MANUAL, VOUCHER, MANASTORE. Режим выдачи по запросу дополнительно зависит от Product.walletOnDemandEnabled. ROKKY фактически всегда mock; Telegram helper возвращает случайный ключ. `activation-system/prisma/schema.prisma:40` `activation-system/services/walletActivationService.js:43` `activation-system/services/rokkyService.js:8` `activation-system/utils/telegramHelper.js:5`.

## Стек

| Компонент | Реализация | Источник |
| --- | --- | --- |
| Приложение | activation-system 1.0.0; ES modules; main index.js | `activation-system/package.json:2` |
| HTTP | express ^5.1.0 | `activation-system/package.json:32` |
| HTML | ejs ^3.1.10 + express-ejs-layouts ^2.5.1 | `activation-system/package.json:31` |
| БД | Prisma ^6.5.0, @prisma/client ^6.5.0; PostgreSQL 16 | `activation-system/package.json:20` `activation-system/docker-compose.yml:34` |
| Аутентификация | bcrypt, express-session, connect-pg-simple, cookie-parser; отдельный OTP-путь через Eskiz | `activation-system/package.json:25` `activation-system/index.js:109` `activation-system/utils/smsService.js:116` |
| HTTP/SMS/email | axios, nodemailer, Eskiz; интеграции реализованы собственными сервисами | `activation-system/package.json:24` `activation-system/utils/mailer.js:5` |
| PDF / QR / Wallet | pdfkit, qrcode, bwip-js, passkit-generator | `activation-system/package.json:26` `activation-system/package.json:39` |
| CSS | Tailwind 3.4.4, PostCSS, Autoprefixer | `activation-system/package.json:23` `activation-system/package.json:43` |
| Файлы / лимиты | Multer 2.0.2; express-rate-limit 8.2.1 | `activation-system/package.json:34` `activation-system/package.json:37` |
| Валидация | Joi объявлен, импортов в прикладном коде не найдено; ручные условия контроллеров | `activation-system/package.json:36` `activation-system/controllers/public/qrPaymentController.js:116` |
| Очередь / кэш | Серверная очередь и Redis не реализованы; браузерные Cache API и IndexedDB используются SW | `activation-system/package.json:19` `activation-system/public/sw.js:69` `activation-system/public/sw.js:257` |
| Android | TWA uz.namo.wallet.twa; Android Browser Helper 2.6.2, Gradle plugin 8.9.1 | `my-pwa/app/build.gradle:24` `my-pwa/app/build.gradle:211` `my-pwa/build.gradle:26` |


## Пользователи и границы доступа

Роли User: admin, merchant, vendor_user, financial_mgr, content_mgr, support_agent, kassa_admin, kassa_viewer. Клиент — отдельная сущность Client, а не значение enum Role. `activation-system/prisma/schema.prisma:29` `activation-system/prisma/schema.prisma:405`.

`ensureMerchant` проверяет только роль; статус Merchant.status при этом не проверяется. `allowFinance` включает kassa_admin и не ограничивает выборку его кассой. `activation-system/middleware/auth.js:51` `activation-system/middleware/auth.js:23`.

## Требования к запуску

Dockerfile выбирает Node 20; package.json не задаёт engines. Нужны PostgreSQL, заполненное окружение, Prisma Client, каталоги public/uploads и receipts, файлы модели PassKit и сертификаты. ManaStore без mock проверяет ключ и wallet ID ещё при импорте модуля. `activation-system/Dockerfile:1` `activation-system/package.json:1` `activation-system/middleware/uploadMiddleware.js:6` `activation-system/routes/wallet.js:26` `activation-system/services/manastoreService.js:38`.

Публичные клиентские URL зависят от Host `wallet.*`; простой запрос к localhost не эквивалентен запросу к wallet.namo.uz. `activation-system/middleware/checkSubdomain.js:4` `activation-system/index.js:194`.

Чистый запуск по цепочке миграций не подтверждён и статически блокируется её порядком. Полный путь развёртывания и ограничения приведены в RUNBOOK; ошибки безопасности и незавершённые части — в GAPS. `activation-system/prisma/migrations/20250211120000_add_refresh_token/migration.sql:19` `activation-system/prisma/migrations/20250404134644_/migration.sql:17`.

## Состав комплекта

- ARCHITECTURE — поток запроса, подсистемы, бизнес-операции, браузер и Android.
- API_REFERENCE — все регистрации маршрутов, параметры, ответы и проверки.
- DATA_MODEL — все поля, ограничения и SQL-миграции.
- VIEWS — 109 шаблонов, переменные, формы и зависимости.
- CONFIGURATION — 56 имён окружения и конфигурация в БД.
- GAPS — незавершённое, риски и недостижимый код.
- RUNBOOK — команды из проекта, ограничения установки и тестовые контракты.
