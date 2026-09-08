# API — MIBT (полный справочник эндпоинтов)

Последнее обновление: 2026-09-08 (TASK-0002, построчная сверка всех файлов
`routes/`). Логика обработчиков лежит непосредственно в файлах роутов —
каталог `controllers/` пуст и не используется (см. [GAPS.md](GAPS.md)).

Всего **77 объявлений маршрутов** в 13 файлах. Формат ответа определяется
кодом обработчика: `res.render(view)` — HTML, `res.json({...})` — JSON,
`res.redirect(url)` — 302. Многие auth/валидационные обработчики отвечают
по-разному в зависимости от `req.xhr || req.headers.accept?.includes(
'application/json')` — в таблице это отмечено как «JSON/redirect».

Общий middleware по префиксу монтирования (`index.js`) не повторяется в
каждой строке — см. колонку «Auth (mount)» первой в каждом разделе.

## Публичные SSR-маршруты — `routes/auth.routes.js` (mount: `/`, без auth)

| Метод | Путь | Query/Params | Ответ |
|---|---|---|---|
| GET | `/login` | — | Редирект на `/admin` или `/dashboard`, если уже в сессии; иначе `render('auth/login')` |
| GET | `/logout` | — | `req.session.destroy()` → редирект `/login` |
| GET | `/forgot-password` | — | Редирект, если в сессии; иначе `render('auth/forgot-password')` |
| GET | `/reset-password` | — | Редирект, если в сессии; иначе `render('auth/reset-password')` |

## Публичный JSON API — `routes/api/auth.api.js` (mount: `/api/auth`, без auth)

| Метод | Путь | Валидация (Joi) | Тело запроса | Ответ / коды |
|---|---|---|---|---|
| POST | `/api/auth/login` | `schemas.login` | `{email, password}` | 200 `{success, redirect}` или JSON/redirect с flash; 401 неверные креды; 403 если `customer.archivedAt` заполнен |
| POST | `/api/auth/forgot-password` | `schemas.forgotPassword` | `{email}` | Всегда 200 `{success, message}` (не раскрывает, существует ли email) — генерирует OTP (6 цифр, TTL 15 мин), отправляет письмо |
| POST | `/api/auth/reset-password` | `schemas.resetPassword` | `{email, otp, newPassword, confirmPassword}` | 200 `{success, redirect: '/login'}`; 400 если OTP неверный/истёк |

## `routes/api/orders.api.js` (mount: `/api/orders`, `requireCustomer`)

| Метод | Путь | Валидация | Тело / Params | Ответ / коды |
|---|---|---|---|---|
| POST | `/api/orders` | `schemas.createOrder` | `{lines:[{variantId,quantity}], contactName, contactPhone, contactEmail?, customerNote?}` | 200 `{success, orderId}`; 400 недостаточно средств/товар неактивен; создаёт заказ и (для `CORPORATE`) замораживает баланс в одной транзакции; отправляет 2 письма (админу и клиенту), не блокируя ответ |
| GET | `/api/orders/:id` | — | `:id` — Order.id | 200 `{success, order}` (лицензии в ответе только если `status===DELIVERED`, расшифрованы); 403 нет профиля клиента; 404 не найден/чужой |

## `routes/api/documents.api.js` (mount: `/api/documents`, `requireAuth`)

| Метод | Путь | Доп. проверка роли | Тело/Params | Ответ / коды |
|---|---|---|---|---|
| POST | `/api/documents/upload` | Только `ADMIN` (проверка внутри обработчика) | `multipart/form-data`: `file` (любой формат), `orderId`, `type` | 200 `{success, document}`; 403 не админ; 400 нет файла/полей |
| POST | `/api/documents/upload-payment` | Любой авторизованный, но заказ должен принадлежать клиенту | `multipart`: `file` (только PDF/JPEG/PNG, ≤10MB), `orderId` | 200 `{success, document}` (тип всегда `PAYMENT`); 400 неверный формат/размер/статус заказа (`DELIVERED`/`CANCELLED` запрещены); 403/404 |
| POST | `/api/documents/generate-link` | Владелец заказа или `ADMIN` | `{documentId}` | 200 `{success, token}` — создаёт `DocumentDownloadToken`, TTL 5 мин; 403/404 |
| GET | `/api/documents/download/:token` | Токен — единственный ключ доступа | `:token` | `res.download(...)`, токен удаляется сразу после использования (одноразовый); 404 неверный/использованный; 400 истёк |

## `routes/api/balance.api.js` (mount: `/api/balance-requests`, `requireCustomer`)

| Метод | Путь | Валидация | Тело | Ответ / коды |
|---|---|---|---|---|
| POST | `/api/balance-requests` | `schemas.balanceRequest` (сумма в тийинах, мин. 1 000 000 = 10 000 UZS) | `{amount, note?}` | 200 `{success, requestId}`; отправляет письмо администратору |

## `routes/api/exchange-rate.api.js` (mount: `/api/exchange-rate`, `requireAuth`)

| Метод | Путь | Ответ |
|---|---|---|
| GET | `/api/exchange-rate` | 200 `{success, rate}` — курс USD с `cbu.uz`, кэш 4ч, фолбэк `13000` |

## `routes/api/quotes.api.js` (mount: `/api/quotes`, `requireAuth`)

| Метод | Путь | Доп. проверка роли | Валидация/Тело | Ответ / коды |
|---|---|---|---|---|
| POST | `/api/quotes` | Только `CUSTOMER` | `schemas.createQuoteRequest`: `{items:[{variantId,variantName,productName,quantity,unitPrice,vatRate?}]}` | 200 `{success, quoteId}`; 403 не клиент; отправляет письмо администратору |
| POST | `/api/quotes/:id/upload` | Только `ADMIN` | `multipart`: `file` (любой формат) | 200 `{success}` — заменяет старый документ, если был; переводит `status → PROCESSED`; отправляет письмо клиенту |
| POST | `/api/quotes/:id/token` | Владелец заявки (и только если `status===PROCESSED`) или `ADMIN` | — | 200 `{success, token}`, TTL 5 мин; 403/404 |
| GET | `/api/quotes/download/:token` | Токен — ключ доступа, **но эндпоинт всё равно смонтирован под `requireAuth`** — сессия обязательна вопреки комментарию в коде (см. [GAPS.md](GAPS.md) п.2.4) | `:token` | `res.download(...)`, токен удаляется после использования |

## `routes/api/admin/index.js` (mount: `/api/admin`, `requireAdmin`) — 29 маршрутов

### Заказы

| Метод | Путь | Валидация | Тело/Params | Ответ / коды |
|---|---|---|---|---|
| POST | `/api/admin/orders` | Ручная проверка (`customerId`, `lines` не пустой массив) | `{customerId, lines:[{variantId,quantity}], adminNote?}` | 200 `{success, orderId}`; 400/404; заморозка баланса как при клиентском заказе |
| POST | `/api/admin/orders/:id/deliver` | Проверка `!order` есть | `:id` | 200 `{success}`; 404 если нет; 400 если статус не `PENDING`/`IN_PROGRESS`; списывает баланс (`onOrderDelivered`) |
| POST | `/api/admin/orders/:id/cancel` | **Проверки `!order` нет** — см. [GAPS.md](GAPS.md) п.2.6 | `:id`, `{note?}` | 200 `{success}`; 400 если уже `DELIVERED`/`CANCELLED`; при отсутствии заказа — 500 (TypeError) вместо 404 |
| PATCH | `/api/admin/orders/:id/status` | **Проверки `!order` нет** для веток `DELIVERED`/`CANCELLED` — см. GAPS.md п.2.6 | `:id`, `{status: IN_PROGRESS\|DELIVERED\|CANCELLED}` | 200 `{success}`; 400 неверный переход/неизвестный статус |
| PATCH | `/api/admin/orders/:id/note` | — | `:id`, `{adminNote}` | 200 `{success}` |
| POST | `/api/admin/orders/:id/lines/:lineId/licenses` | — | `:id`, `:lineId`, `{licenseKey, position?}` | 200 `{success}` — шифрует ключ (AES-256-GCM) и создаёт `OrderLineLicense` |
| POST | `/api/admin/orders/:id/generate-specification` | — | `:id` | 200 `{success, document}` — генерирует PDF (PDFKit), сохраняет как `OrderDocument` типа `SPECIFICATION` |
| POST | `/api/admin/orders/bulk-action` | Ручная | `{action: cancel\|delete, ids:[]}` | 200 `{success, count}`; 400 неизвестное действие |
| DELETE | `/api/admin/orders/:id` | — | `:id` | 200 `{success}`; 404; если активен — сначала отменяет (возврат баланса), потом удаляет |

### Клиенты

| Метод | Путь | Валидация | Тело/Params | Ответ / коды |
|---|---|---|---|---|
| POST | `/api/admin/customers` | **Нет** — `schemas.createCustomer` определена, но не подключена (см. [GAPS.md](GAPS.md) п.2.3) | `{email, password, companyName, inn, mfo?, bankAccount?, address?, clientType?}` | 200 `{success, customerId}`; 400 `P2002` (дубликат email/ИНН); отправляет приветственное письмо |
| POST | `/api/admin/customers/bulk-action` | Ручная | `{action: delete, ids:[]}` | 200 `{success, count}`; 409 если у кого-то есть заказы |
| POST | `/api/admin/customers/:id` | — | `:id`, `{companyName, inn, mfo, bankAccount, address, clientType}` | 200 `{success}`; 400 `P2002` |
| POST | `/api/admin/customers/:id/credit` | `schemas.adminCredit` | `:id`, `{amount, note, requestId?}` | 200 `{success}`; 400 если `requestId` невалиден (чужой/не `PENDING`); зачисляет баланс (`FOR UPDATE`), опционально одобряет `BalanceRequest` |
| PATCH | `/api/admin/customers/:id/password` | — | `:id`, `{password}` | 200 `{success}` — меняет пароль без уведомления по умолчанию (письмо отправляется, но без специального предупреждения в UI) |
| DELETE | `/api/admin/customers/:id/transactions/:txId` | — | `:id`, `:txId` | 200 `{success}`; 404; удаляет только запись журнала, **баланс не пересчитывается** |

### Товары

| Метод | Путь | Валидация | Тело/Params | Ответ / коды |
|---|---|---|---|---|
| POST | `/api/admin/products/:id/licenses` | Ручная (`keys` — непустой массив) | `:id`, `{keys:[string]}` | 200 `{success, count}` — создаёт `ProductLicense(status=AVAILABLE)`; **см. [GAPS.md](GAPS.md) п.2.1 — эти данные не используются далее** |
| POST | `/api/admin/products/bulk-action` | Ручная | `{action: delete\|draft\|activate, ids:[]}` | 200 `{success, count}`; 409 если у варианта есть строки заказов (для `delete`) |
| POST | `/api/admin/products` | multer (`uploadProductImage`, только JPEG/PNG/WebP/GIF/SVG, ≤5MB) | multipart: `name, slug, description?, overview?, manufacturer?, mfrPartNo?, price, vatRate?, ikpu, productType?, specs?(JSON), variantsJson?(JSON), image?(file)` | 200 `{success, product}`; 400 `P2002` (дубликат slug/SKU) |
| PATCH | `/api/admin/products/:id/variants/:variantId` | — | `{price?, name?}` | 200 `{success}` |
| POST | `/api/admin/products/:id` | multer (та же валидация файла) | multipart, аналогично созданию + `isActive` | 200 `{success, product}`; 400 `P2002` |
| DELETE | `/api/admin/products/:id/image` | — | `:id` | 200 `{success}`; 404; удаляет файл с диска (с проверкой пути внутри `PRODUCT_IMAGE_DIR`) |
| DELETE | `/api/admin/products/:id` | — | `:id` | 200 `{success}`; 409 если есть заказы у вариантов |
| POST | `/api/admin/products/:id/variants` | — | `{name, price, sku}` | 200 `{success, variant}`; 400 `P2002` |
| DELETE | `/api/admin/products/:id/variants/:variantId` | — | `:variantId` | 200 `{success}` |

### Баланс

| Метод | Путь | Тело/Params | Ответ / коды |
|---|---|---|---|
| PATCH | `/api/admin/balance-requests/:id/approve` | `:id` | 200 `{success}`; 400 если не `PENDING`; зачисляет баланс, шлёт письмо |
| PATCH | `/api/admin/balance-requests/:id/reject` | `:id` | 200 `{success}`; 400 если не `PENDING`; шлёт письмо |
| POST | `/api/admin/balance-requests/bulk-action` | `{action: approve\|reject\|delete, ids:[]}` | 200 `{success, count}` |

### Прочее

| Метод | Путь | Валидация | Тело | Ответ / коды |
|---|---|---|---|---|
| POST | `/api/admin/settings` | Проверка `isValidTheme()` для ключа `ui_theme` | `{settings: {key: value, ...}}` | 200 `{success}`; 400 неверная тема; upsert в `SystemSetting`, сбрасывает кэш темы при изменении `ui_theme` |
| POST | `/api/admin/quotes/bulk-action` | Ручная | `{action: mark_processed\|delete, ids:[]}` | 200 `{success, count}` |

## SSR-маршруты клиента — `routes/catalog.routes.js` (mount: `/catalog`, `/product`, `/dashboard`, все `requireCustomer`)

Один и тот же роутер смонтирован на трёх префиксах — каждый маршрут ниже
доступен под всеми тремя (см. [ARCHITECTURE.md](ARCHITECTURE.md)).

| Метод | Путь (относительно префикса) | Query | Ответ |
|---|---|---|---|
| GET | `/` | — | `render('client/dashboard')`, до 12 активных товаров + баланс |
| GET | `/software` | `search, manufacturer, minPrice, maxPrice, page, sort` | `render('client/catalog')`, `productType=SOFTWARE` |
| GET | `/hardware` | то же | `render('client/catalog')`, `productType=HARDWARE` |
| GET | `/:slug` | — | `render('client/product')`; 404 если товар не найден/неактивен |

## `routes/cart.routes.js` (mount: `/cart`, `/checkout`, `requireCustomer`)

| Метод | Путь | Ответ |
|---|---|---|
| GET | `/cart` (роутер `cartRoutes`) | `render('client/cart')` — вся логика на клиенте |
| GET | `/checkout` (роутер `checkoutRoutes`) | `render('client/checkout')` с актуальным `balance`/`clientType` |

## `routes/account.routes.js` (mount: `/account`, `requireCustomer`)

| Метод | Путь | Ответ |
|---|---|---|
| GET | `/account` | `render('client/account')` |
| GET | `/account/orders` | `render('client/orders')`, все заказы клиента |
| GET | `/account/orders/:id` | `render('client/order')`; 404 чужой/несуществующий; лицензии расшифрованы только при `DELIVERED` |
| GET | `/account/balance` | `render('client/balance')` |
| GET | `/account/top-up` | `render('client/top-up')` с реквизитами из `SystemSetting` |

## `routes/quotes.routes.js` (mount: `/account/quotes`, `requireCustomer`)

| Метод | Путь | Ответ |
|---|---|---|
| GET | `/account/quotes` | `render('client/quotes')` |
| GET | `/account/quotes/:id` | `render('client/quote')`; 404 чужой/несуществующий |

## `routes/admin.routes.js` (mount: `/admin`, `requireAdmin`) — 16 маршрутов

| Метод | Путь | Query | Ответ |
|---|---|---|---|
| GET | `/admin` | — | `render('admin/dashboard')` — статистика, последние заказы/клиенты, ожидающие заявки |
| GET | `/admin/products` | — | `render('admin/products')` |
| GET | `/admin/products/new` | — | `render('admin/product-new')` |
| GET | `/admin/products/:id/edit` | — | `render('admin/product-edit')` |
| GET | `/admin/orders` | `status?` | `render('admin/orders')` |
| GET | `/admin/orders/new` | — | `render('admin/order-new')` |
| GET | `/admin/orders/:id` | — | `render('admin/order')`; `next()` без ошибки, если не найден (404 по умолчанию Express) |
| GET | `/admin/customers` | — | `render('admin/customers')` |
| GET | `/admin/customers/new` | — | `render('admin/customer-new')` |
| GET | `/admin/customers/:id` | — | `render('admin/customer')` |
| GET | `/admin/customers/:id/edit` | — | `render('admin/customer-edit')` |
| GET | `/admin/balance-requests` | — | `render('admin/balance-requests')` |
| GET | `/admin/settings` | — | `render('admin/settings')` |
| GET | `/admin/audit` | — | `render('admin/audit')`, до 100 записей — **на практике всегда пусто, см. [GAPS.md](GAPS.md) п.2.2** |
| GET | `/admin/quotes` | `status?` | `render('admin/quotes')` |
| GET | `/admin/quotes/:id` | — | `render('admin/quote')`; 404 если не найдено |

## Прочее

| Метод | Путь | Auth | Ответ |
|---|---|---|---|
| GET | `/` | `requireAuth` (`index.js:149-152`) | Редирект на `/admin` (роль `ADMIN`) или `/dashboard` |
| GET | `/css/themes.css` | нет (`index.js:83-87`) | Сгенерированный CSS цветовых палитр, `Cache-Control: public, max-age=3600` |

## Общие паттерны валидации и безопасности

- Joi-схемы (`middleware/validate.js`): `login`, `createOrder`,
  `balanceRequest`, `adminCredit`, `forgotPassword`, `resetPassword`,
  `createQuoteRequest` — подключены к своим роутам. `createCustomer` —
  определена, но нигде не подключена (см. [GAPS.md](GAPS.md)).
- При ошибке валидации: 422 для JSON/XHR, редирект `back` с
  `req.session.flash` для обычных форм.
- Скачивание файлов — только через одноразовые токены
  (`DocumentDownloadToken`/`QuoteDownloadToken`, TTL 5 минут), путь к файлу
  дополнительно проверяется на выход за пределы `secure-assets/`.
- CSRF-токенов в формах не найдено.
- Rate limiting: `express-rate-limit` глобально на `/api/*` (100 запросов/15
  мин с IP, `index.js:66-71`) + отдельные зоны system nginx хоста для
  `/api/auth/login` и `/api/auth/forgot-password` (см.
  [DEPLOYMENT.md](DEPLOYMENT.md)).

См. также [VIEWS.md](VIEWS.md) — какие формы/JS реально вызывают эти
эндпоинты, и [GAPS.md](GAPS.md) — эндпоинты без UI, отсутствующая
валидация, несогласованности.
