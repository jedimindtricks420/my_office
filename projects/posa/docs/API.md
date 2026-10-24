# API — POSA

Последнее обновление: 2026-09-05 (TASK-0001). Извлечено напрямую из
`routes/*.js` в `/home/admin1/posa/activation-system/`. Это HTTP API,
частично отдающее HTML (EJS, для браузерных ролей) и частично JSON
(`/api/*`, вебхуки платежей). Отдельного OpenAPI/Swagger-описания в
проекте не найдено.

## Аутентификация

- **Сотрудники/мерчанты/вендоры/касса** — `authRoutes.js`, логин по
  `username`/`password` (bcrypt), сессия (`req.session.user`), опционально
  refresh-токен (remember-me). Middleware `middleware/auth.js`.
- **Клиенты** — `authController` в `controllers/client/`, вход по номеру
  телефона + OTP через SMS (Eskiz), без пароля. Middleware
  `middleware/authClient.js` (`isClientAuthenticated`).
- Рейт-лимиты (`express-rate-limit`) применены точечно: `loginLimiter`
  (`/auth/login`), `registerLimiter` и `otpVerifyLimiter`
  (`/client-register`, `/client-verify`), `generationLimiter`
  (генерация ваучеров), `activationLimiter` (`/activate` в wallet-роутере).

## `/auth` — общая авторизация (authRoutes.js)
- `GET /auth/login`, `POST /auth/login` (rate-limited)
- `POST /auth/refresh`
- `GET|POST /auth/logout`
- `register` — закомментирован в коде, не активен.

## `/admin` — бэк-офис (adminRoutes.js, 104 маршрута)
Требует `ensureAuthenticated` + ролевые guard'ы (`allowOfficeDashboard`,
`allowFinance`, `allowContent`, `allowSupport`, `allowFinanceOrContent`,
`ensureAdmin`). Основные разделы (полный список — в файле):
- `/admin/dashboard`
- `/admin/kassas*` — CRUD касс (только admin на создание/редактирование)
- `/admin/sales`, `/admin/merchants*`, `/admin/vendor/:id/*` — финансы
- `/admin/clients*` — поддержка
- `/admin/products*`, `/admin/product-images*`, `/admin/vouchers*`,
  `/admin/stores*` — контент
- `/admin/vendors*`, `/admin/add-vendor-users` — вендоры
- `/admin/add-user`, `/admin/users*` — управление пользователями (только admin)
- `/admin/rokky*`, `/admin/drweb*`, `/admin/manastore*` — по одному
  зеркальному набору роутов на каждого внешнего поставщика (dashboard, skus
  CRUD, products bind, activations, finance)
- `/admin/telegram-bots*` — CRUD ботов уведомлений
- `/admin/manual-activations*` — обработка заявок ручной активации
- `/admin/qr-links*` — генерация QR-ссылок на оплату (merchant+product)

## `/merchant` (merchantRoutes.js)
- `GET /merchant/login`
- `GET /merchant/dashboard`
- `GET /merchant/sales`, `POST /merchant/checkout/confirm`
- `GET /merchant/sell`, `POST /merchant/add-to-cart`,
  `GET /merchant/checkout`, `POST /merchant/update-cart`,
  `GET /merchant/cart/remove/:id`
- `GET|POST /merchant/activate`

## `/vendor` (vendorRoutes.js, всё под `ensureVendor`)
- `GET /vendor/`, `/vendor/dashboard`, `/vendor/activate` (GET+POST),
  `/vendor/vouchers`, `/vendor/transactions`, `/vendor/settings`

## `/kassa` (kassaCabinetRoutes.js, всё под `ensureAuthenticated` + `ensureKassa`)
- `/kassa/dashboard`, `/kassa/transactions`, `/kassa/obligations`
- `/kassa/payouts` (GET; POST требует дополнительно `ensureKassaAdmin`)
- `/kassa/settings`

## `/wallet`, `/client` — клиентский кошелёк (clientRoutes.js)
- `GET|POST /wallet` — вход клиента
- `GET|POST /client-register`, `GET|POST /client-verify` — регистрация + OTP
- `GET /client/logout`, `GET /logout`, `POST /refresh`
- `POST /client/delete-account` (авторизован)
- `GET /client/dashboard`, `/client/profile`, `/client/qr-scanner`
  (авторизованы через `isClientAuthenticated`)

## `/api/client` — JSON API клиента (clientApiRoutes.js, всё под `isClientAuthenticated`)
- `GET /api/client/vouchers`, `GET /api/client/voucher/:id`
- `POST /api/client/voucher/:id/request-code`,
  `GET /api/client/voucher/:id/code-status`
- `POST /api/client/voucher/:id/log`
- `POST /api/client/push-subscription`
- `POST /api/client/voucher/claim`

## `/api/v1` (apiV1Routes.js — смонтирован в index.js)
- `POST /api/v1/voucher/:id/events` (под `isClientAuthenticated`)

**Отдельно:** `routes/apiV1.js` содержит ещё один набор `/health`, `/wallet`,
`/voucher/:id`, `/voucher/:id/events` с захардкоженными тестовыми данными
(`TEST-100-AAAA`) — этот файл **не импортирован** в `index.js` и, судя по
всему, не обслуживает реальный трафик. См. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

## Платежи — вебхуки провайдеров (paymentRoutes.js)
- `POST /api/payments/click/:kassaId/prepare`
- `POST /api/payments/click/:kassaId/complete`
- `POST /api/payments/payme/:kassaId` (единая точка входа, JSON-RPC-подобный
  протокол Payme — методы различаются внутри `paymeController.handlePayme`)
- `POST /api/payments/alif/:kassaId/webhook` (проверка HMAC-подписи по
  `req.rawBody`, см. ARCHITECTURE.md)

Каждый вебхук параметризован `:kassaId` — то есть реквизиты и логика
проверки подписи зависят от конкретной записи `Kassa` в БД, а не только от
глобальных `.env`-переменных (`CLICK_*`, `PAYME_*`).

## Партнёрские сети — UWS (uwsRoutes.js, отдельный процесс)

- `POST /api/payments/uws/:partner` — единая точка входа JSON-RPC 2.0;
  поддерживается `paynet`. Методы `GetInformation`, `PerformTransaction`,
  `CheckTransaction`, `CancelTransaction`, `GetStatement`.
- `GET /healthz` — проверка живости шлюза.

Направление вызовов обратное остальным платёжным интеграциям: партнёр
обращается к нам, HTTP Basic, логин и пароль выдаём мы и храним хешем в
`PartnerService`. Обслуживается контейнером `activation-uws`
(`127.0.0.1:4010`), а не основным приложением. См. [PAYNET.md](PAYNET.md).

Маршруты админки раздела: `/admin/partners`, `/admin/partners/:id/items`,
`/admin/partner-journal` (роли `admin`, `financial_mgr`, `content_mgr`).

## QR-платёж (публичный, без авторизации) — qrPaymentRoutes.js
- `GET /pay/:token`
- `GET|POST /pay/:token/checkout`
- `GET /pay/:token/result/:attemptId`
- `GET /pay/:token/receipt/:attemptId`

`:token` — значение `MerchantProductLink.token`, привязка мерчант+товар.

## Магазины по слагу (storeRoutes.js)
- `POST /:storeSlug/otp`, `/verify`, `/activate`, `/logout`
- `GET /:storeSlug/history`
- `GET /:storeSlug` (страница витрины, самый общий маршрут — смонтирован до
  `publicRoutes.js`, чтобы не конфликтовать с публичными путями)

## Публичные страницы (publicRoutes.js)
- `GET /`
- `GET|POST /email`
- `GET|POST /client/enter-voucher`
- `GET|POST /activate`

## Wallet / Apple Wallet (wallet.js, смонтирован на `/`)
- `GET /wallet/:serial.pkpass` — генерация `.pkpass`-файла
  (`passkit-generator`)
- `GET /activate` (rate-limited `activationLimiter`)
- `GET /qr/:token`

## Не проверено в рамках этой задачи

- Точные тела запросов/ответов (схемы полей) для каждого эндпоинта — не
  документировались построчно, см. сами контроллеры для деталей.
- Формальной спецификации (OpenAPI/Postman) в проекте не найдено.
