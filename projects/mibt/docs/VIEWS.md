# VIEWS — MIBT

Последнее обновление: 2026-09-08. Источник: построчное чтение всех 35
файлов `.ejs` в `/home/admin1/mibt-2/views/` (TASK-0002). Для каждого шаблона
указаны: кто его рендерит (роут), какие переменные он ожидает от контроллера,
какие формы/AJAX-запросы он отправляет и куда.

Движок — EJS + `express-ejs-layouts`. Layout по умолчанию —
`layouts/main.ejs` (устанавливается `app.set('layout', 'layouts/main')` в
`index.js:49`); страница ошибки (`views/error.ejs`) явно отключает layout
(`res.render('error', { layout: false, ... })`,
`middleware/errorHandler.js:12`).

## Layout и partials

| Файл | Назначение | Переменные |
|---|---|---|
| `views/layouts/main.ejs` | Общий каркас: `<head>`, шрифты, `/css/themes.css` + `/css/app.css`, header, flash-блок, `<%- body %>`, footer, `/js/app.js` | `title` (опц.), `uiTheme` (опц., атрибут `data-theme`), `flash` (опц., `{type, message}`), `user` (через partial `header`) |
| `views/partials/header.ejs` | Хедер: логотип, навигация по роли, переключатель темы, корзина (только `CUSTOMER`), email + выход | `user` (`{id, role, email}` или `null`, из `res.locals.user`, `index.js:112-116`) |
| `views/partials/footer.ejs` | Копирайт, email поддержки | — |
| `views/error.ejs` | Страница ошибки, без layout | `status`, `message`, `error` (объект `{}` или `err` целиком, если `NODE_ENV=development` — тогда рендерится `error.stack`) |

## Auth (`views/auth/`)

| Шаблон | Роут (рендер) | Переменные | Формы / AJAX | Адресат |
|---|---|---|---|---|
| `login.ejs` | `GET /login` (`routes/auth.routes.js:5-10`) | `title` | `#loginForm` → `fetch(JSON)` | `POST /api/auth/login` |
| `forgot-password.ejs` | `GET /forgot-password` (`routes/auth.routes.js:18-23`) | `title` | `#forgotForm` → `fetch(JSON)` | `POST /api/auth/forgot-password` |
| `reset-password.ejs` | `GET /reset-password` (`routes/auth.routes.js:25-30`) | `title` | `#resetForm` → `fetch(JSON)`, клиентская сверка `newPassword === confirmPassword` до отправки | `POST /api/auth/reset-password` |

Ни одна из трёх форм не использует нативную отправку `<form action=...>` —
все три отправляют JSON через `fetch` и обрабатывают ответ вручную.

## Client (`views/client/`) — требуют `requireCustomer`

| Шаблон | Роут (рендер) | Переменные | Формы / AJAX | Адресат |
|---|---|---|---|---|
| `dashboard.ejs` | `GET /` смонтирован на `/catalog`, `/product`, `/dashboard` (`routes/catalog.routes.js:77-103`) | `title`, `products` (до 12, `isActive: true`), `balance` (только если `role === CUSTOMER`, иначе `null`) | Кнопки «В корзину» пишут прямо в `localStorage.mibt_cart` (JS-функция `addToCart`, без сети) | — (клиентское состояние) |
| `catalog.ejs` | `GET /software`, `GET /hardware` того же роутера (`routes/catalog.routes.js:106-133`, через `buildCatalogData`) | `title`, `categoryName`, `categorySlug`, `breadcrumb`, `products`, `totalCount`, `totalPages`, `currentPage`, `manufacturers`, `filters` (`{search, manufacturer, minPrice, maxPrice, sort}`) | Форма фильтра (`GET`, нативная, `action="/catalog/<slug>"`); кнопки «В корзину» → `localStorage` | Сама страница (query-параметры) / `localStorage` |
| `product.ejs` | `GET /:slug` того же роутера (`routes/catalog.routes.js:136-161`) | `title`, `product` (с `variants`), `categoryName`, `categorySlug` | Селектор варианта, счётчик количества, «Добавить в корзину» — весь JS, пишет в `localStorage.mibt_cart` | — |
| `cart.ejs` | `GET /cart` (`routes/cart.routes.js:6-10`) | `title` | Рендер корзины **целиком на клиенте** из `localStorage` (JS `renderCart()`); кнопка «Запросить КП» → `fetch` | `POST /api/quotes` |
| `checkout.ejs` | `GET /checkout` (`routes/cart.routes.js:14-34`) | `title`, `balance` (`Customer.balance` или `null`), `clientType` | `#checkoutForm` (contactName/Phone/Email/Note) читается через `FormData`, товары — из `localStorage`; кнопка «Подтвердить заказ» → `fetch` | `POST /api/orders` |
| `account.ejs` | `GET /account` (`routes/account.routes.js:7-39`) | `title`, `customer` (с `user`), `orders` (10 последних), `transactions` (10 последних), `balance` | — (только просмотр + ссылки) | — |
| `orders.ejs` | `GET /account/orders` (`routes/account.routes.js:41-61`) | `title`, `orders` (все, без пагинации) | — | — |
| `order.ejs` | `GET /account/orders/:id` (`routes/account.routes.js:63-102`) | `title`, `order` (с `lines.variant.product`, `lines.licenses` — `decryptedKey` заполнен только если `status === DELIVERED`, `documents`) | Загрузка платёжного документа `#paymentUploadBtn` → `fetch(FormData)`; кнопка «Скачать» документа → `fetch(JSON)` затем редирект на URL с токеном | `POST /api/documents/upload-payment`; `POST /api/documents/generate-link` → `GET /api/documents/download/:token` |
| `balance.ejs` | `GET /account/balance` (`routes/account.routes.js:104-126`) | `title`, `balance`, `transactions` (до 50), `balanceRequests` (до 10) | — | — |
| `top-up.ejs` | `GET /account/top-up` (`routes/account.routes.js:128-141`) | `title`, `gpTech` (объект из `SystemSetting`, ключи `company_name/inn/bank_name/mfo/bank_account`) | `#topUpForm` (amount, note) → `fetch(JSON)` | `POST /api/balance-requests` |
| `quotes.ejs` | `GET /account/quotes` (`routes/quotes.routes.js:7-27`, смонтирован под `/account/quotes` в `index.js:143`) | `title`, `quotes` (с `document`) | — | — |
| `quote.ejs` | `GET /account/quotes/:id` (`routes/quotes.routes.js:30-55`) | `title`, `quote` (с `document`) | Кнопка «Скачать PDF» → `fetch(JSON)` за токеном, затем переход | `POST /api/quotes/:id/token` → `GET /api/quotes/download/:token` |

## Admin (`views/admin/`) — требуют `requireAdmin`

| Шаблон | Роут (рендер) | Переменные | Формы / AJAX | Адресат |
|---|---|---|---|---|
| `dashboard.ejs` | `GET /admin` (`routes/admin.routes.js:7-45`) | `title`, `orders` (10 последних), `customers` (10 последних), `balanceRequests` (только `PENDING`), `stats` (6 счётчиков) | «Исполнить» заявку → `fetch`; «Выдать» заказ → `fetch` | `POST /api/admin/customers/:id/credit`; `POST /api/admin/orders/:id/deliver` |
| `products.ejs` | `GET /admin/products` (`routes/admin.routes.js:47-60`) | `title`, `products` (с `variants`) | Таб-фильтры и поиск — чисто клиентские (JS по `data-*` атрибутам, без запроса на сервер); массовые действия → `fetch` | `POST /api/admin/products/bulk-action` |
| `product-new.ejs` | `GET /admin/products/new` (`routes/admin.routes.js:62-64`) | `title` | `#productForm` (multipart, включая динамические варианты/спецификации, собираемые в JSON на клиенте) → `fetch(FormData)` | `POST /api/admin/products` |
| `product-edit.ejs` | `GET /admin/products/:id/edit` (`routes/admin.routes.js:66-77`) | `title`, `product` (с `variants`) | Основная форма → `fetch(FormData)`; удаление изображения, сохранение/добавление/удаление варианта — отдельные `fetch`-вызовы | `POST /api/admin/products/:id`; `DELETE .../image`; `PATCH .../variants/:variantId`; `POST .../variants`; `DELETE .../variants/:variantId` |
| `orders.ejs` | `GET /admin/orders` (`routes/admin.routes.js:79-92`, фильтр `?status=`) | `title`, `orders`, `activeStatus` | Таб-фильтры/поиск клиентские; массовые действия → `fetch` | `POST /api/admin/orders/bulk-action` |
| `order-new.ejs` | `GET /admin/orders/new` (`routes/admin.routes.js:94-102`) | `title`, `customers` (все), `products` (с `variants`) | `#orderNewForm` (динамические строки товаров) → `fetch(JSON)` | `POST /api/admin/orders` |
| `order.ejs` | `GET /admin/orders/:id` (`routes/admin.routes.js:104-119`) | `title`, `order` (customer+balance, lines+variant+product+licenses, documents) | Смена статуса (`.status-btn`) → `fetch`; сохранение `adminNote` → `fetch`; генерация спецификации → `fetch`; загрузка документа → `fetch(FormData)`; удаление заказа → `fetch`; скачивание документа → `fetch` за токеном | `PATCH .../status`; `PATCH .../note`; `POST .../generate-specification`; `POST /api/documents/upload`; `DELETE /api/admin/orders/:id`; `POST /api/documents/generate-link` → `GET /api/documents/download/:token` |
| `customers.ejs` | `GET /admin/customers` (`routes/admin.routes.js:121-131`) | `title`, `customers` (с `user`, `balance`) | Таб-фильтры/поиск клиентские; массовое удаление → `fetch` | `POST /api/admin/customers/bulk-action` |
| `customer-new.ejs` | `GET /admin/customers/new` (`routes/admin.routes.js:133-135`) | `title` | `#createCustomerForm` (ручная JS-валидация email/пароль≥8/ИНН, **слабее серверной Joi-схемы, которая всё равно не подключена — см. GAPS.md п.2.3**) → `fetch(JSON)` | `POST /api/admin/customers` |
| `customer.ejs` | `GET /admin/customers/:id` (`routes/admin.routes.js:137-153`) | `title`, `customer` (user, balance, transactions до 50, orders до 10) | Ручное пополнение баланса → `fetch`; смена пароля клиента → `fetch`; удаление транзакции → `fetch` | `POST .../credit`; `PATCH .../password`; `DELETE .../transactions/:txId` |
| `customer-edit.ejs` | `GET /admin/customers/:id/edit` (`routes/admin.routes.js:155-165`) | `title`, `customer` | `#customerEditForm` → `fetch(JSON)` | `POST /api/admin/customers/:id` |
| `balance-requests.ejs` | `GET /admin/balance-requests` (`routes/admin.routes.js:167-177`) | `title`, `requests` (с `customer`) | Одобрить/отклонить (построчно) и массово → `fetch` | `PATCH .../approve`; `PATCH .../reject`; `POST /api/admin/balance-requests/bulk-action` |
| `settings.ejs` | `GET /admin/settings` (`routes/admin.routes.js:179-195`) | `title`, `settings` (map), `themes` (`THEMES` из `lib/themes.js`), `activeTheme` | Выбор темы (превью на клиенте через `data-theme`) → `fetch`; форма реквизитов → `fetch` | оба `POST /api/admin/settings` (разные наборы ключей в теле) |
| `audit.ejs` | `GET /admin/audit` (`routes/admin.routes.js:197-207`) | `title`, `logs` (до 100) | — (только просмотр) | — (см. GAPS.md п.2.2 — таблица всегда пустая, `logAudit()` нигде не вызывается) |
| `quotes.ejs` | `GET /admin/quotes` (`routes/admin.routes.js:209-226`, фильтр `?status=`) | `title`, `quotes` (customer+user, document), `activeStatus` | Таб-фильтры/поиск клиентские; массовые действия → `fetch` | `POST /api/admin/quotes/bulk-action` |
| `quote.ejs` | `GET /admin/quotes/:id` (`routes/admin.routes.js:228-246`) | `title`, `quote` (customer+user, document) | Загрузка/замена PDF → `fetch(FormData)`; скачивание существующего документа → `fetch` за токеном | `POST /api/quotes/:id/upload`; `POST /api/quotes/:id/token` → `GET /api/quotes/download/:token` |

## Проверка на «мёртвые» шаблоны и недостающие формы

- Все 35 файлов `.ejs` (кроме `layouts/`, `partials/`) подтверждённо
  рендерятся хотя бы одним `res.render(...)` в соответствующем файле роута —
  «view без рендера» не обнаружено.
- Обратное — «роут без view» — тоже не обнаружено среди SSR-роутов; все
  JSON API (`routes/api/**`) закономерно не рендерят view.
- **Отсутствующие формы для существующих API**: ни один `.ejs` не вызывает
  `POST /api/admin/products/:id/licenses` и
  `POST /api/admin/orders/:id/lines/:lineId/licenses` — см.
  [GAPS.md](GAPS.md), разделы «Не реализовано» п.8 и «Реализовано
  частично» п.2.1.
- `public/test-products.html` — статичная страница вне `views/`, не
  участвует в рендеринге через Express `res.render`, раздаётся напрямую как
  статика (см. [GAPS.md](GAPS.md), «Мёртвый код» п.8).

## Общие клиентские механизмы (см. `public/js/app.js`)

- Переключение тёмной темы — `localStorage.theme`, не связано с
  серверной цветовой палитрой (`uiTheme`/`ui_theme` в `SystemSetting`) —
  это два независимых понятия: одна тема — light/dark режим (клиент), другая
  — акцентная палитра (сервер, применяется глобально всем через
  `/admin/settings`).
- `formatUZS(tiyin)` и `updateCartCount()` — общие функции, используются
  всеми страницами с `.format-uzs[data-tiyin]` элементами и счётчиком
  корзины в хедере соответственно.
