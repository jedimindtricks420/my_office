# ARCHITECTURE — MIBT

Последнее обновление: 2026-09-08 (TASK-0002, полная построчная сверка).
Основано на прямом чтении кода `/home/admin1/mibt-2/`.

## Структура каталогов и назначение

| Каталог | Назначение | Примечание |
|---|---|---|
| `index.js` | Точка входа, вся настройка Express | — |
| `routes/` | SSR-роуты верхнего уровня (`*.routes.js`) | Логика прямо здесь, не в контроллерах |
| `routes/api/` | JSON API верхнего уровня (`*.api.js`) | — |
| `routes/api/admin/index.js` | Весь admin JSON API (29 маршрутов, ~850 строк) | Один файл, не разбит по сущностям |
| `controllers/`, `controllers/api/`, `controllers/api/admin/` | Заявлены в дереве | **Полностью пусты**, см. [GAPS.md](GAPS.md) |
| `services/` | Бизнес-логика (баланс, лицензии, email, PDF, курс валют, аудит, тема) | — |
| `middleware/` | `auth.js`, `errorHandler.js`, `validate.js`, `upload.js` (мёртвый), `upload-image.js` | — |
| `lib/` | `prisma.js` (клиент), `themes.js` (реестр цветовых палитр + генератор CSS) | — |
| `views/` | EJS-шаблоны (`layouts/`, `partials/`, `auth/`, `client/`, `admin/`, `error.ejs`) | Подробности — [VIEWS.md](VIEWS.md) |
| `prisma/` | `schema.prisma`, `migrations/` (3 файла), `seed.js`, `prisma.config.ts` | Миграции не покрывают все модели схемы, см. [GAPS.md](GAPS.md) 2.7 |
| `public/` | Статика: `css/`, `js/app.js`, `img/`, `img/products/` (загруженные изображения товаров) | `express.static`, доступна без авторизации; содержит забытый `test-products.html` |
| `secure-assets/` | Загруженные/сгенерированные документы (PDF, платёжки) | Доступ только через одноразовые токены, не через `express.static` |
| `resources/css/` | Исходник Tailwind (`app.css`) | Компилируется в `public/css/app.css` |
| `i18n/`, `templates/` | Заявлены в дереве | Полностью пустые, см. [GAPS.md](GAPS.md) |
| `ssl/` | TLS-сертификаты для внутреннего `mibt2-nginx` | Содержит `.bak`-копии (см. GAPS.md) |
| `backups/` | Дампы БД (daily/weekly/monthly/last) | Пишутся контейнером `db-backup` |

## Поток одного HTTP-запроса (пример: `GET /catalog/software`)

```
Браузер → System nginx (TLS) → Docker mibt2-app:3000
  → helmet (CSP-заголовки)
  → express-rate-limit (только для /api/*, здесь не применяется)
  → cors
  → express.json / express.urlencoded / cookie-parser
  → express.static('public')  (не совпало — не статический путь)
  → express-session (читает cookie, подгружает сессию из Postgres)
  → глобальный middleware: res.locals.user / flash / uiTheme
  → app.use('/catalog', requireCustomer, catalogRoutes)
      → requireCustomer: проверка req.session.role === 'CUSTOMER'
      → catalogRoutes.get('/software', ...): buildCatalogData() → 3 запроса Prisma
        (findMany products, count, findMany distinct manufacturers) параллельно (Promise.all)
  → res.render('client/catalog', {...}) → EJS рендерит layouts/main.ejs
    с телом client/catalog.ejs → HTML → браузер
```

Пример для JSON API (`POST /api/orders`): те же общие middleware → `requireCustomer`
(на уровне `app.use('/api/orders', requireCustomer, ordersApiRoutes)`) →
`validate(schemas.createOrder)` → обработчик считает суммы → `prisma.$transaction`
(создание `Order` + `onOrderCreated` → заморозка баланса) → 2 письма
fire-and-forget → `res.json({success, orderId})`.

## Общая схема

Один Express-процесс (`index.js`) в одном Docker-контейнере обслуживает и
серверный рендеринг (EJS-шаблоны), и JSON API — нет разделения на
frontend/backend сервисы. Роль пользователя определяется сессией
(`express-session` + `connect-pg-simple`, хранение сессий в той же
PostgreSQL, таблица `session`), а не поддоменом или отдельным сервисом.

```
Браузер
  │  HTTPS (443)
  ▼
System nginx (хост)               ── терминирует TLS для portal.mibt.uz
  │  proxy_pass 127.0.0.1:13000
  ▼
Docker: mibt2-app (Express, порт 3000 в контейнере)
  │
  ├─ EJS SSR: routes/*.routes.js → views/**/*.ejs
  ├─ JSON API: routes/api/**/*.js (под /api/*)
  ├─ Сессии: connect-pg-simple → Postgres, таблица session
  └─ lib/prisma.js → Prisma Client
        │
        ▼
Docker: mibt2-postgres (PostgreSQL 16-alpine)
        │
        ▼
Docker: mibt-2-db-backup-1 (ежедневный/еженедельный/ежемесячный бэкап)
```

Есть также второй, отдельный nginx **внутри Docker Compose**
(`mibt2-nginx`, `nginx.conf` в корне проекта), слушающий `127.0.0.1:3011`
с собственным self-signed TLS (`./ssl/*.pem`) — по всем признакам это
резервный/аварийный путь доступа к приложению в обход системного nginx
хоста, не является основным путём для пользователей (см.
[DEPLOYMENT.md](DEPLOYMENT.md)).

## Точка входа и порядок middleware (`index.js`, прочитан полностью)

1. `helmet` — CSP разрешает только `'self'` + инлайн-скрипты/стили + Google
   Fonts + `data:`/`blob:` для изображений.
2. `express-rate-limit` — глобально на `/api/*`: 100 запросов/15 мин с одного
   IP.
3. `cors` — origin берётся из `ALLOWED_ORIGINS` (через запятую) или `false`
   (запрещено), если переменная не задана; `credentials: true`.
4. `express.json`, `express.urlencoded`, `cookie-parser`.
5. Статика генерируется на лету: `/css/themes.css` — CSS-переменные текущей
   цветовой темы (`lib/themes.js`), кэш `Cache-Control: public, max-age=3600`.
6. `express.static('public')`.
7. Сессии (`express-session`, cookie `httpOnly`, `secure` в production,
   `sameSite: lax`, TTL 8 часов).
8. Глобальный middleware прокидывает `res.locals.user`, `res.locals.flash`,
   `res.locals.uiTheme` (активная тема, кэш 30 сек в `theme.service.js`) во
   все EJS-шаблоны.
9. Роуты (см. ниже) → `errorHandler` (общий обработчик ошибок).

## Дерево маршрутизации (подтверждено `index.js` + файлами роутов)

Логика **не вынесена в контроллеры** — каталог `controllers/` присутствует в
дереве проекта, но пуст (только пустые вложенные подпапки `api/`,
`api/admin/`). Весь код обработчиков находится прямо в файлах
`routes/*.routes.js` и `routes/api/**/*.js`.

| Префикс | Файл | Auth middleware | Тип |
|---|---|---|---|
| `/` (login/logout/forgot/reset) | `routes/auth.routes.js` | — (публично) | SSR |
| `/api/auth` | `routes/api/auth.api.js` | — (публично) | JSON |
| `/api/orders` | `routes/api/orders.api.js` | `requireCustomer` | JSON |
| `/api/documents` | `routes/api/documents.api.js` | `requireAuth` (+ доп. проверка роли внутри) | JSON |
| `/api/balance-requests` | `routes/api/balance.api.js` | `requireCustomer` | JSON |
| `/api/exchange-rate` | `routes/api/exchange-rate.api.js` | `requireAuth` | JSON |
| `/api/admin` | `routes/api/admin/index.js` | `requireAdmin` | JSON |
| `/api/quotes` | `routes/api/quotes.api.js` | `requireAuth` | JSON |
| `/catalog`, `/product`, `/dashboard` | `routes/catalog.routes.js` | `requireCustomer` | SSR |
| `/cart`, `/checkout` | `routes/cart.routes.js` | `requireCustomer` | SSR |
| `/account`, `/account/quotes` | `routes/account.routes.js`, `routes/quotes.routes.js` | `requireCustomer` | SSR |
| `/admin` | `routes/admin.routes.js` | `requireAdmin` | SSR |

Всего подтверждено прямым подсчётом **77 объявлений HTTP-маршрутов**
(`router.get/post/put/patch/delete`) в 13 файлах роутеров; из них 29 — в
одном файле `routes/api/admin/index.js` (~840 строк). Подробный список — в
[API.md](API.md).

**Особенность монтирования `catalogRoutes`**: в `index.js:137-141` один и
тот же экземпляр роутера смонтирован на трёх разных префиксах —
`/catalog`, `/product`, `/dashboard`. Это значит, что, например,
`GET /dashboard/software` и `GET /product/software` технически отдают тот
же ответ, что и `GET /catalog/software` — не обнаружено, используется ли
это где-то намеренно (ссылки в `.ejs` всегда используют `/catalog/...`,
кроме навигации на `/dashboard` для главной страницы ЛК).

**Корзина — полностью клиентская.** В схеме Prisma нет модели корзины;
весь список товаров и количество хранятся в `localStorage.mibt_cart`
браузера (`views/client/catalog.ejs`, `product.ejs`, `cart.ejs`,
`checkout.ejs` — см. [VIEWS.md](VIEWS.md)). Сервер узнаёт о содержимом
корзины только в момент `POST /api/orders` или `POST /api/quotes`, когда
клиент присылает список позиций целиком в теле запроса.

## Слой сервисов (`services/`, прочитан списком + ключевые файлы целиком)

| Файл | Назначение | Подтверждено |
|---|---|---|
| `order.service.js` | Заморозка/списание/возврат баланса при смене статуса заказа, назначение лицензий из пула | Прочитан полностью. `assignLicensesToOrder` определена, но нигде не вызывается — мёртвый код (см. AGENTS.md) |
| `balance.service.js` | `freezeBalanceTx`/`debitBalance`/`unfreezeAndRefund`/`creditBalance`, все с `SELECT ... FOR UPDATE` | Прочитан полностью |
| `license.service.js` | AES-256-GCM шифрование/расшифровка ключей, ключ из `ENCRYPTION_KEY` (32 байта) | Прочитан полностью |
| `audit.service.js` | `logAudit()` — запись в `AuditLog`, ошибки записи только логируются в консоль, не пробрасываются | Прочитан полностью |
| `pdf.service.js` | Генерация PDF-спецификации заказа через PDFKit | Прочитан полностью |
| `exchangeRate.service.js` | Курс USD с `cbu.uz`, кэш 4 часа, фолбэк `13000` | Прочитан полностью |
| `theme.service.js` | Активная UI-тема из `SystemSetting`, кэш 30 сек | Прочитан частично (первые строки) |
| `email.service.js` + `email/{transport,layout,templates}.js` | Фасад отправки писем, 13 шаблонов | Список экспортов подтверждён (14 функций в фасаде, 13 в templates.js), содержимое шаблонов не читалось построчно |
| `format.js` | Форматирование сумм (`formatUZS`) | Использование подтверждено в `pdf.service.js` |

## Аутентификация и авторизация

- Вход — email + пароль (`bcrypt`, cost 12), сессия в Postgres.
- Восстановление пароля — email + одноразовый OTP (6 цифр, TTL 15 минут,
  хранится в полях `User.emailOtp`/`emailOtpExpiry`), НЕ используется для
  обычного входа.
- Авторизация — по `req.session.role` через 3 middleware-функции в
  `middleware/auth.js` (`requireAuth`, `requireAdmin`, `requireCustomer`);
  для XHR/JSON-запросов возвращают 401/403, для обычных — редирект на
  `/login`.
- **CSRF-защиты не найдено** — ни `csurf`, ни собственной реализации не
  обнаружено при поиске по коду и зависимостям.
- Rate limiting: глобальный в Express (`express-rate-limit`, 100/15мин на
  `/api/*`) + отдельные зоны в system nginx хоста для
  `/api/auth/login` (5 req/min) и `/api/auth/forgot-password` (3 req/min).

## Файловое хранилище

- `secure-assets/` — все загружаемые/генерируемые документы (PDF, изображения
  оплаты, спецификации), имена файлов — `crypto.randomUUID()` + расширение.
  Доступ — не напрямую по URL, а только через одноразовые токены
  (`DocumentDownloadToken`/`QuoteDownloadToken`, TTL 5 минут, удаляются после
  скачивания). Путь к файлу проверяется на выход за пределы `secure-assets/`
  (`path.resolve` + проверка префикса) при скачивании документов заказа.
- `public/img/products/` — публичные изображения товаров (multer,
  `middleware/upload-image.js`).

## Пустые/неиспользуемые директории и файлы (подтверждено `find`/`grep`)

- `controllers/` (и вложенные `api/`, `api/admin/`) — полностью пусты.
- `i18n/` — полностью пуста, несмотря на наличие в дереве проекта (признак
  планировавшейся, но не реализованной локализации).
- `templates/` — полностью пуста.
- `middleware/upload.js` — определяет рабочий multer-инстанс, но нигде не
  импортируется; фактические загрузчики документов определены заново,
  локально, в `routes/api/documents.api.js` и `routes/api/quotes.api.js`.

Полный список мёртвого кода с обоснованием — [GAPS.md](GAPS.md).

## Многоязычность

Не реализована: интерфейс — только на русском (весь текст в `.ejs`
хардкодится на русском языке), каталог `i18n/` пуст.

## См. также

- [VIEWS.md](VIEWS.md) — карта всех SSR-шаблонов, переменных и форм.
- [CONFIGURATION.md](CONFIGURATION.md) — полный список переменных окружения.
- [GAPS.md](GAPS.md) — незавершённый функционал, мёртвый код, риски.
