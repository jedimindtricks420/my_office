# API — 1PDF

Полностью переписано 2026-09-08 (`TASK-0002`), построчно прочитаны
`admin/server.ts` (1435 строк), `admin/routes/editorApi.ts` (358 строк),
`admin/controllers/paymeController.ts` (310 строк),
`admin/utils/paymentUtils.ts` (165 строк) целиком.

## Аутентификация — общий механизм

`authMiddleware` (`admin/server.ts:216-263`) — единая точка входа для всех
маршрутов, кроме явного списка `publicPaths` (14 путей, `admin/server.ts:217-231`):
проверяет cookie `1pdf_token` (пользователь) и/или `1pdf_admin_token`
(админ), либо заголовок `Authorization: Bearer`; при отсутствии обоих —
`401` до входа в конкретный обработчик. Далее по месту — `requireAdmin`,
`requireSuperAdmin`, `requireUser`, `requirePro` (админ-роли/PRO-подписка).

## admin (`admin/server.ts`, порт 8031)

### Публичные (в `publicPaths`, без JWT)

| Метод | Путь | Rate limit | Тело | Ответ |
|---|---|---|---|---|
| GET | `/api/auth/verify` | — | — | используется только nginx `auth_request`; 401 без сессии, 403 при `/editor/*` без PRO или неподтверждённом email |
| POST | `/api/auth/login` | authLimiter (10/15мин) | `{login,password}` | админ-логин, cookie `1pdf_admin_token` (7д) |
| POST | `/api/client/signup` | authLimiter | `{email,password,name}` | шлёт OTP на email, возвращает `signupToken` (JWT, 15 мин, содержит хешированный пароль и хеш OTP) |
| POST | `/api/client/verify-signup` | — | `{code,signupToken}` | создаёт `User`+`Subscription(FREE)`, cookie `1pdf_token` (30д) |
| POST | `/api/client/login` | authLimiter | `{email,password}` | cookie `1pdf_token` |
| POST | `/api/client/logout` | — | — | очищает cookie |
| GET | `/api/landing/content` | — | `?lang=RU\|UZ` | `{config, sections}` для лендинга |
| POST | `/api/documents/callback` | — | ONLYOFFICE-формат | **legacy**, см. ниже |
| POST | `/api/payments/payme` | — | JSON-RPC | вебхук Payme, делегирует в `paymeController.handlePayme` |
| POST | `/api/payments/click/prepare`, `/complete` | — | форма Click | вебхуки Click |
| POST | `/api/payments/alif/webhook` | — | HMAC-подписанное тело | вебхук Alifpay |

### Админские (`requireAdmin`/`requireSuperAdmin`)

| Метод | Путь | Роль | Тело/параметры | Ответ |
|---|---|---|---|---|
| GET | `/api/dashboard` | ADMIN | — | KPI: total_users, pro_active/expired, new_users_7d, revenue, pending_payments |
| GET/POST | `/api/admins` | SUPERADMIN | POST: `{login,password≥6,name?,role?}` | управление админами |
| PATCH/DELETE | `/api/admins/:id` | SUPERADMIN | нельзя менять роль/удалять самого себя | — |
| POST | `/api/admins/:id/reset-password` | SUPERADMIN | `{newPassword≥6}` | — |
| GET | `/api/auth/me` | ADMIN | — | профиль текущего админа |
| PATCH | `/api/admin/landing/content` | ADMIN | `{lang,config?,sections?}` | upsert CMS |
| GET/POST | `/api/users` | ADMIN | POST: `{email,name?,password,notes?}` | список/создание, создаваемые сразу `is_verified: true` |
| DELETE | `/api/users/:userId` | ADMIN | — | **без каскадной проверки** — `Document.owner_id`/`DocumentVersion.created_by_id` имеют `onDelete: Restrict`, удаление пользователя с документами упадёт ошибкой Prisma, не обработанной по месту (см. GAPS.md) |
| POST | `/api/users/:userId/upgrade` | ADMIN | `{days?}` (default 30) | продление PRO |
| PATCH | `/api/users/:userId/plan` | ADMIN | `{plan: FREE\|PRO}` | смена тарифа, `FREE` сбрасывает `valid_until` |
| GET/POST | `/api/payments` | ADMIN | POST: `{user_id,amount,currency?,provider,status?,notes?}` | журнал / ручное добавление платежа |
| GET/POST | `/api/admin/payment-settings` | SUPERADMIN | POST: `{payme_*,click_*,alif_*}`, `"********"` не перезаписывает секрет | конфигурация провайдеров в БД |
| GET | `/api/vouchers` | ADMIN | `?series=` | — |
| GET | `/api/vouchers/export` | ADMIN | `?series=` | `.txt`, один код на строку |
| DELETE | `/api/vouchers` | ADMIN | — | **всегда `405`** — массовое удаление намеренно отключено (комментарий в коде) |
| POST | `/api/vouchers/generate` | ADMIN | `{count≤5000,days,series}` | `series` валидируется regex `^[A-Z0-9]{2,15}$` |

### Пользовательские (`requireUser`, иногда `+ requirePro`)

| Метод | Путь | PRO? | Тело/параметры | Ответ |
|---|---|---|---|---|
| POST | `/api/client/verify-email` | нет | `{code}` | подтверждение email по коду |
| GET | `/api/client/me` | нет | — | профиль + подписка |
| POST | `/api/client/activate-voucher` | нет | `{code}` | атомарная активация (`updateMany` с `is_used: false`) |
| POST | `/api/client/change-password` | нет | `{currentPassword,newPassword}` | — |
| GET | `/api/client/history` | нет | — | последние 50 платежей |
| POST | `/api/client/payments/initiate` | нет | `{provider: PAYME\|CLICK\|ALIF}` | цена жёстко `199000` сум/год (`server.ts:944`), создаёт `PaymentAttempt`, возвращает `redirectUrl` |
| GET | `/api/documents/config` | **да** | `?fileId&title` | **legacy ONLYOFFICE** — см. ниже |
| POST | `/api/documents/upload` | **да** | `multipart {file}`, uploadLimiter (10/мин) | **legacy** — сохраняет в `UPLOADS_DIR` (плоский каталог, без версионирования), не в модель `Document` |
| POST | `/api/documents/create-new` | **да** | — | **legacy** — создаёт пустой PDF на диске |
| GET | `/uploads/:filename` | по токену/cookie | `?token=` (JWT `JWT_SECRET_OO`) или cookie `1pdf_token` (только свои файлы, префикс `userId_`) | отдаёт файл с диска |

⚠️ **Legacy ONLYOFFICE-эндпоинты** (`/api/documents/config`, `/upload`,
`/create-new`, `/callback`) технически исполняемы (код присутствует и
синтаксически корректен), но зависят от контейнера `onlyoffice-docs`,
который подтверждённо не запущен (`Exited`) и не описан в
`docker-compose.yml` — при реальном обращении `/api/documents/config`
вернёт корректный JSON (сам эндпоинт не обращается к ONLYOFFICE напрямую,
только подписывает конфиг), но сам процесс редактирования через ONLYOFFICE
работать не будет, так как редактор — iframe на несуществующий сервис.
`/api/documents/callback` целиком зависит от входящего вызова от
ONLYOFFICE — при его отсутствии никогда не вызывается. См.
[GAPS.md](GAPS.md).

## `/api/v1/*` (`admin/routes/editorApi.ts`) — текущий бэкенд редактора

Все маршруты за `router.use(requireUser)` (`editorApi.ts:103`), часть
дополнительно за `requirePro`.

| Метод | Путь | PRO? | Тело/параметры | Ответ |
|---|---|---|---|---|
| GET | `/api/v1/me/preferences` | нет | — | `{language, theme: 'system'}` (theme всегда захардкожен, не хранится) |
| PATCH | `/api/v1/me/preferences/language` | нет | `{language: ru\|uz\|en}` | — |
| GET | `/api/v1/documents` | да | — | список документов пользователя с текущей версией |
| POST | `/api/v1/documents/blank` | да | `{name?}` | создаёт пустой PDF (612×792pt) через `pdf-lib`, версия 1 |
| POST | `/api/v1/documents` | да | `multipart {file}` (≤50МБ, ≤500 стр., проверка `%PDF-` сигнатуры + `pdf-lib` парсинг) | создаёт документ + версию 1 |
| GET | `/api/v1/documents/:documentId/editor-session` | да | — | `{document, downloadUrl, permissions, limits, userId}` |
| GET | `/api/v1/documents/:documentId/content` | да | `?versionId=` | стримит PDF-файл (проверка `filePath.startsWith(documentsDir)` от path traversal) |
| GET | `/api/v1/documents/:documentId/versions` | да | — | список версий |
| POST | `/api/v1/documents/:documentId/versions` | да | `multipart {file,baseVersionId}` | новая версия; `409 VERSION_CONFLICT` при несовпадении `baseVersionId` с текущей (оптимистичная блокировка) |
| POST | `/api/v1/documents/:documentId/versions/:versionId/complete` | да | — | подтверждение готовности версии (в текущем коде версии всегда создаются сразу `READY` — эндпоинт для будущего асинхронного аплоада, см. GAPS.md) |

## Платёжные интеграции — детали проверки подписи

- **Payme** (`admin/controllers/paymeController.ts`) — JSON-RPC 2.0,
  Basic-Auth (`checkPaymeAuth`, логин обязан быть `Paycom`), методы
  `CheckPerformTransaction`/`CreateTransaction`/`PerformTransaction`/
  `CancelTransaction`/`CheckTransaction`/`GetStatement`. Использует
  **отдельный** `new PrismaClient()` (`paymeController.ts:7`), не тот же
  инстанс, что в `server.ts` — два независимых пула соединений к одной БД.
- **Click** — MD5-подпись конкатенации полей (`verifyClickSignature`,
  `paymentUtils.ts:7-29`), секрет — `config.click_secret_key` из БД либо
  `process.env.CLICK_SECRET_KEY` как фолбэк (`server.ts:1003,1036` — в
  отличие от Contador/MIBT, здесь фолбэк на `.env` **явно закодирован**).
- **Alifpay** — HMAC-SHA256 по сырому телу запроса (`req.rawBody`,
  захватывается верификатором `express.json()`), заголовок `Signature`,
  сравнение через `crypto.timingSafeEqual` (`paymentUtils.ts:157-164`).

## Существующая документация проекта об API

`/home/admin1/1pdf/docs/API.md` (не читалась по содержимому — `.md`,
только упомянута как существующий факт в `AGENTS.md` этого my_office-проекта
из TASK-0001) описывает, по имеющимся сведениям, только текущий
редактор+платежи, не покрывая legacy ONLYOFFICE-эндпоинты и не весь
admin-API — не проверялось построчно в рамках этой задачи.
