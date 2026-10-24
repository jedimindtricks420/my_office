# ARCHITECTURE — POSA

Последнее обновление: 2026-09-05 (TASK-0001). Источник фактов — прямое чтение
кода в `/home/admin1/posa/activation-system/`.

> **Обновление 2026-10-24.** Процессов стало два: к `activation-system`
> добавлен `activation-uws` — шлюз партнёрских сетей на той же кодовой базе
> и той же БД, но с открытыми только роутами UWS (`uws-server.js`,
> `127.0.0.1:4010`). Каналов продаж тоже три: QR-ссылка мерчанта, витрина
> магазина и терминалы Paynet. Все они сходятся в одну функцию выдачи
> `processPaymentInternal`. См. [PAYNET.md](PAYNET.md).

## Общая схема

Один монолитный Node.js/Express-процесс (`activation-system`, порт 4000)
обслуживает все роли (клиент/мерчант/вендор/админ/публичные страницы) и все
поддомены `*.namo.uz`. Разделение по ролям происходит не на уровне отдельных
сервисов, а на уровне:

1. **Nginx** — терминирует TLS, определяет `Host`, проксирует все поддомены на
   один и тот же `127.0.0.1:4000` (кроме `api.namo.uz`, который идёт на
   `127.0.0.1:4001` — тот же контейнер, второй проброшенный порт, см.
   `docker-compose.override.yml`).
2. **`middleware/checkSubdomain.js`** — по `req.hostname` выставляет
   `req.appRole` (`client`/`merchant`/`vendor`/`admin`/`public`).
3. **`middleware/auth.js`** и **`middleware/authClient.js`** — проверяют
   `req.session.user` (роли `Role` enum: `admin`, `merchant`, `vendor_user`,
   `financial_mgr`, `content_mgr`, `support_agent`, `kassa_admin`,
   `kassa_viewer`) или `req.session.client` (для клиентов, авторизация по
   номеру телефона + OTP, без пароля).

Точка входа — `index.js`: настраивает Express, EJS + `express-ejs-layouts`,
сессии в Postgres (`connect-pg-simple`, таблица `session`), затем монтирует
роутеры (порядок важен — см. ниже) и слушает `PORT` (по умолчанию 4000).

## Сессии

Комментарий в самом `index.js` фиксирует историческую причину архитектурного
решения: раньше использовался `MemoryStore` по умолчанию, и любой рестарт
контейнера сбрасывал сессии/корзины всех пользователей одновременно. Сейчас
сессии хранятся в той же Postgres-БД через `connect-pg-simple`
(таблица `session`, создаётся автоматически).

Cookie: `sameSite: 'lax'`, `secure` только когда `NODE_ENV=production`;
`maxAge` — `SESSION_MAX_AGE` либо `REMEMBER_ME_MAX_AGE` (из
`utils/authTokens.js`) в зависимости от чекбокса "запомнить меня"; при
"запомнить меня" дополнительно выпускается `RefreshToken` (модель в Prisma) и
ставятся отдельные cookie (`utils/authTokens.js: setRememberCookies`).

## Порядок монтирования роутов (`index.js`)

Важен для понимания, какой обработчик выигрывает при конфликте путей:

1. `/auth` → `authRoutes.js`
2. `/admin` → `adminRoutes.js`
3. `/merchant` → `merchantRoutes.js`
4. `/vendor` → `vendorRoutes.js`
5. `/kassa` → `kassaCabinetRoutes.js`
6. `/api/client` → `clientApiRoutes.js`
7. `/api/v1` → `apiV1Routes.js`
8. `/receipts` → статика (сгенерированные PDF-чеки)
9. `/` → `walletRoutes.js` (wallet.js) — **до** клиентских и публичных роутов,
   комментарий в коде поясняет: критично, чтобы `.pkpass` отдавался раньше
   общих обработчиков
10. `/wallet`, `/client` → `clientRoutes.js`
11. Промежуточный redirect-middleware по `req.appRole`
12. `/` → `paymentRoutes.js` (Click/Payme/Alif webhook-коллбэки)
13. `/` → `qrPaymentRoutes.js` (публичный QR-платёж, без авторизации)
14. `/` → `storeRoutes.js` (динамические `/:storeSlug`)
15. `/` → `publicRoutes.js` (последний, общий catch-all)

## Слои приложения

- **routes/** — 14 файлов, только объявление путей + middleware, без бизнес-
  логики (см. [API.md](API.md) за полным списком).
- **controllers/** — 40 файлов, сгруппированы по ролям в подпапки:
  `admin/` (18), `client/` (включая `client/api/`), `merchant/`, `vendor/`,
  `kassa/`, `payment/` (`clickController.js`, `paymeController.js`,
  `alifController.js`), `public/`.
- **services/** — интеграции с внешними системами: `drwebService.js`,
  `manastoreService.js`, `rokkyService.js`, `telegramService.js`, плюс
  доменные сервисы `voucherActivationService.js`,
  `walletActivationService.js`.
- **utils/** — низкоуровневые помощники: `authTokens.js` (refresh-токены,
  remember-me), `smsService.js` (Eskiz), `phone.js` (нормализация номера),
  `qr.js`, `receiptRenderer.js` (PDF через `pdfkit`), `mailer.js`
  (`nodemailer`), `telegramHelper.js`, `utils/payment/*` (подпись/проверка
  запросов Click/Payme/Alif).
- **prisma/** — схема, миграции (28 шт.), `client.js` (единый экспортируемый
  `PrismaClient`), `seed-kassa.js`.
- **views/** — EJS-шаблоны (109 файлов), `express-ejs-layouts`, разложены по
  `pages/` (в т.ч. `pages/kassa`, `pages/vendor`), `partials/` (в т.ч.
  `partials/client`, `partials/merchant`), `store/`.
- **public/** — статика, включая PWA-артефакты клиентского кошелька
  (`manifest.json`, `sw.js`, `offline.html`) и `.well-known/assetlinks.json`
  (Android App Links для TWA-обёртки).

## Webhook-подпись для Alifpay

`index.js` подключает `express.json({ verify })`, сохраняя сырые байты тела
запроса в `req.rawBody` — это сделано специально для проверки HMAC-подписи
вебхука Alifpay в `controllers/payment/alifController.js` (комментарий в коде:
повторная сериализация `JSON.stringify(req.body)` может изменить порядок
ключей и подпись не совпадёт).

## Мобильное приложение (my-pwa/)

`/home/admin1/posa/my-pwa/` — отдельный Android-проект, сгенерированный
`bubblewrap-cli` (TWA, Trusted Web Activity), оборачивающий
`https://wallet.namo.uz` в нативное Android-приложение (`twa-manifest.json`:
`packageId: uz.namo.wallet.twa`, `host: wallet.namo.uz`). Не Node.js-код,
собирается отдельно через Gradle. Подписывающий keystore (`android.keystore`)
физически присутствует в каталоге, но в `.gitignore` и не в git.
