# API — MIBT

Последнее обновление: 2026-09-05. Источник: прямое чтение всех файлов в
`routes/` (13 файлов). Всего подтверждено **77 объявлений маршрутов**.
Логика обработчиков лежит непосредственно в этих файлах (каталог
`controllers/` пуст, не используется).

## SSR-маршруты (EJS, возвращают HTML)

### `routes/auth.routes.js` — публично, без auth (4)
`GET /login`, `GET /logout`, `GET /forgot-password`, `GET /reset-password`

### `routes/catalog.routes.js` — `requireCustomer` (4)
`GET /catalog`, `GET /catalog/software`, `GET /catalog/hardware`,
`GET /catalog/:slug` (карточка товара; также смонтирован под `/product` и
`/dashboard` в `index.js`)

### `routes/cart.routes.js` — `requireCustomer` (2)
`GET /cart`, `GET /checkout` (получает актуальный баланс и `clientType`
клиента с сервера при рендере)

### `routes/account.routes.js` — `requireCustomer` (5)
`GET /account`, `GET /account/orders`, `GET /account/orders/:id`
(расшифровывает лицензионные ключи, только если заказ `DELIVERED`),
`GET /account/balance`, `GET /account/top-up`

### `routes/quotes.routes.js` — `requireCustomer`, смонтирован под `/account/quotes` (2)
`GET /account/quotes`, `GET /account/quotes/:id`

### `routes/admin.routes.js` — `requireAdmin` (16)
`GET /admin`, `GET /admin/products`, `GET /admin/products/new`,
`GET /admin/products/:id/edit`, `GET /admin/orders`, `GET /admin/orders/new`,
`GET /admin/orders/:id`, `GET /admin/customers`, `GET /admin/customers/new`,
`GET /admin/customers/:id`, `GET /admin/customers/:id/edit`,
`GET /admin/balance-requests`, `GET /admin/settings`, `GET /admin/audit`,
`GET /admin/quotes`, `GET /admin/quotes/:id`

## JSON API

### `routes/api/auth.api.js` — публично (3)
`POST /api/auth/login`, `POST /api/auth/forgot-password`,
`POST /api/auth/reset-password`

### `routes/api/orders.api.js` — `requireCustomer` (2)
`POST /api/orders` (создание заказа, заморозка баланса внутри транзакции,
валидация `schemas.createOrder`), `GET /api/orders/:id` (детали заказа,
включая расшифровку лицензионных ключей при статусе `DELIVERED`)

### `routes/api/balance.api.js` — `requireCustomer` (1)
`POST /api/balance-requests` (создание заявки на пополнение)

### `routes/api/documents.api.js` — `requireAuth` + доп. проверка роли внутри (4)
`POST /api/documents/upload` (только ADMIN, проверка роли внутри
обработчика), `POST /api/documents/upload-payment` (клиент — платёжный
документ к своему заказу, только PDF/JPEG/PNG до 10MB),
`POST /api/documents/generate-link` (одноразовый токен, TTL 5 мин),
`GET /api/documents/download/:token` (скачивание, токен одноразовый —
удаляется сразу)

### `routes/api/exchange-rate.api.js` — `requireAuth` (1)
`GET /api/exchange-rate` (курс USD с кэшем, см. ARCHITECTURE.md)

### `routes/api/quotes.api.js` — `requireAuth` (4)
`POST /api/quotes` (создание заявки КП), `POST /api/quotes/:id/upload`
(прикрепление документа), `POST /api/quotes/:id/token` (одноразовая
ссылка), `GET /api/quotes/download/:token`

### `routes/api/admin/index.js` — `requireAdmin` (29, один файл ~840 строк)

Заказы: `POST /api/admin/orders`, `POST /api/admin/orders/:id/deliver`,
`POST /api/admin/orders/:id/cancel`, `PATCH /api/admin/orders/:id/status`,
`PATCH /api/admin/orders/:id/note`,
`POST /api/admin/orders/:id/lines/:lineId/licenses` (ручная выдача ключа),
`POST /api/admin/orders/:id/generate-specification` (PDF),
`POST /api/admin/orders/bulk-action`, `DELETE /api/admin/orders/:id`

Клиенты: `POST /api/admin/customers`, `POST /api/admin/customers/bulk-action`,
`POST /api/admin/customers/:id`, `POST /api/admin/customers/:id/credit`
(ручное зачисление на баланс), `PATCH /api/admin/customers/:id/password`,
`DELETE /api/admin/customers/:id/transactions/:txId`

Товары: `POST /api/admin/products/:id/licenses` (наполнение пула — см.
примечание о мёртвом коде в ARCHITECTURE.md), `POST /api/admin/products/bulk-action`,
`POST /api/admin/products` (создание, с загрузкой изображения),
`POST /api/admin/products/:id` (редактирование), `PATCH
/api/admin/products/:id/variants/:variantId`, `POST
/api/admin/products/:id/variants`, `DELETE /api/admin/products/:id/image`,
`DELETE /api/admin/products/:id`, `DELETE
/api/admin/products/:id/variants/:variantId`

Баланс: `PATCH /api/admin/balance-requests/:id/approve`, `PATCH
/api/admin/balance-requests/:id/reject`, `POST
/api/admin/balance-requests/bulk-action`

Прочее: `POST /api/admin/settings`, `POST /api/admin/quotes/bulk-action`

## Общие паттерны (подтверждено кодом)

- Валидация входных данных — Joi-схемы в `middleware/validate.js`
  (`login`, `createOrder`, `balanceRequest`, `adminCredit`, `createCustomer`,
  `forgotPassword`, `resetPassword`, `createQuoteRequest`); при ошибке — 422
  для JSON/XHR-запросов, редирект с flash-сообщением для обычных форм.
- Обработчики, ожидающие и SSR-форму, и AJAX, различают ответ по
  `req.xhr || req.headers.accept?.includes('application/json')`.
- Скачивание файлов — только через одноразовые токены
  (`DocumentDownloadToken`/`QuoteDownloadToken`), путь к файлу дополнительно
  проверяется на выход за пределы `secure-assets/`.
- CSRF-токенов в формах не найдено (см. ARCHITECTURE.md).
