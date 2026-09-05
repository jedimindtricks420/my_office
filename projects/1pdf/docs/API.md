# API — 1PDF

Снимок на 2026-09-05, источник — `admin/server.ts`,
`admin/routes/editorApi.ts`, `admin/controllers/paymeController.ts`.
Проектная документация `docs/API.md` в `/home/admin1/1pdf/` описывает только
редакторскую часть и платёжные колбэки (см. ниже) — эта страница добавляет
остальную поверхность API, подтверждённую кодом, которую собственная
документация проекта не покрывает.

## Editor API — `/api/v1/*` (`admin/routes/editorApi.ts`)

Полностью задокументировано в собственном
`/home/admin1/1pdf/docs/API.md`: `GET/PATCH /api/v1/me/preferences*`,
`GET/POST /api/v1/documents`, `POST /api/v1/documents/blank`, `GET
/api/v1/documents/:id/editor-session`, `GET
/api/v1/documents/:id/content`, `GET /api/v1/documents/:id/versions`, `POST
/api/v1/documents/:id/versions`. Требует `requireUser` + `requirePro` (кроме
`/me/preferences`, только `requireUser`).

## Аутентификация (`admin/server.ts`)

| Метод | Путь | Доступ | Назначение |
|---|---|---|---|
| POST | `/api/auth/login` | публичный, `authLimiter` | Вход администратора |
| GET | `/api/auth/me` | admin | Текущий администратор |
| GET | `/api/auth/verify` | публичный (используется Nginx `auth_request`) | Проверка сессии + email-верификации + активной PRO-подписки для `/editor` |
| POST | `/api/client/signup` | публичный, `authLimiter` | Регистрация, отправка OTP на email |
| POST | `/api/client/verify-signup` | публичный | Подтверждение OTP, создание `User` |
| POST | `/api/client/login` | публичный, `authLimiter` | Вход пользователя |
| POST | `/api/client/logout` | публичный | Очистка cookie |
| POST | `/api/client/verify-email` | user | Подтверждение email по коду |
| GET | `/api/client/me` | user | Текущий пользователь + подписка |
| POST | `/api/client/change-password` | user | Смена пароля |

## Ваучеры и подписка пользователя

| Метод | Путь | Доступ | Назначение |
|---|---|---|---|
| POST | `/api/client/activate-voucher` | user | Активация ваучера (атомарно, `updateMany` защищает от гонки) |
| GET | `/api/client/history` | user | История платежей (последние 50) |

## Админка — пользователи, платежи, дашборд, ваучеры

| Метод | Путь | Доступ | Назначение |
|---|---|---|---|
| GET | `/api/dashboard` | admin | Агрегаты: пользователи, активные/истёкшие PRO, выручка, ожидающие платежи |
| GET/POST | `/api/admins` | superadmin | Список/создание администраторов |
| PATCH/DELETE | `/api/admins/:id` | superadmin | Изменение/удаление (нельзя себе роль/себя удалить) |
| POST | `/api/admins/:id/reset-password` | superadmin | Сброс пароля админа |
| GET/POST | `/api/users` | admin | Список (поиск по email/имени) / создание пользователя |
| DELETE | `/api/users/:userId` | admin | Удаление пользователя |
| POST | `/api/users/:userId/upgrade` | admin | Продление PRO на N дней |
| PATCH | `/api/users/:userId/plan` | admin | Прямая установка плана `FREE`/`PRO` |
| GET/POST | `/api/payments` | admin | Список / ручное добавление платежа |
| GET/POST | `/api/vouchers` | admin | Список (фильтр по серии) / — |
| GET | `/api/vouchers/export` | admin | Экспорт кодов серии в `.txt` |
| DELETE | `/api/vouchers` | admin | Всегда `405` — массовое удаление намеренно отключено |
| POST | `/api/vouchers/generate` | admin | Генерация партии (до 5000, валидация серии regex `^[A-Z0-9]{2,15}$`) |
| GET/PATCH | `/api/admin/payment-settings` | superadmin | Чтение (секреты скрыты как `********`) / запись реквизитов провайдеров |
| GET/PATCH | `/api/landing/content` / `/api/admin/landing/content` | публичный (GET) / admin (PATCH) | CMS лендинга |

## Платежи — инициация и колбэки провайдеров

Фиксированная цена: 199 000 сум/год (константа в коде, не в конфиге).

| Метод | Путь | Доступ | Назначение |
|---|---|---|---|
| POST | `/api/client/payments/initiate` | user | Создаёт `PaymentAttempt`, возвращает `redirectUrl` для `PAYME`/`CLICK`/`ALIF` |
| POST | `/api/payments/payme` | публичный, Basic Auth (Payme-протокол) | JSON-RPC вебхук: `CheckPerformTransaction`, `CreateTransaction`, `PerformTransaction`, `CancelTransaction`, `CheckTransaction`, `GetStatement` (`admin/controllers/paymeController.ts`) |
| POST | `/api/payments/click/prepare` | публичный, MD5-подпись | Click prepare callback |
| POST | `/api/payments/click/complete` | публичный, MD5-подпись | Click complete callback, вызывает `fulfillSubscription` |
| POST | `/api/payments/alif/webhook` | публичный, HMAC-SHA256 подпись (заголовок `Signature`) | Alifpay webhook, вызывает `fulfillSubscription` при `payment.status === "SUCCEEDED"` |

`fulfillSubscription(attemptId, externalId)` — общая идемпотентная функция:
внутри транзакции продлевает `Subscription` на 365 дней от максимума
(текущий `valid_until` или сейчас), создаёт запись `Payment`, помечает
`PaymentAttempt` как `SUCCESS`.

## Legacy document API (`admin/server.ts`, вне `/api/v1`)

Не описаны в собственном `docs/API.md` проекта. Статус — не подтверждён как
активно используемый (см. [ARCHITECTURE.md](ARCHITECTURE.md) и
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).

| Метод | Путь | Доступ | Назначение |
|---|---|---|---|
| GET | `/api/documents/config` | user + requirePro | Конфиг в формате ONLYOFFICE Document Server, подписан `JWT_SECRET_OO` |
| POST | `/api/documents/upload` | user + requirePro, `uploadLimiter` | Загрузка PDF в корень `temp_uploads/` (не в `documents/`) |
| POST | `/api/documents/create-new` | user + requirePro | Создание пустого PDF тем же способом |
| POST | `/api/documents/callback` | публичный, проверка JWT в заголовке/теле | Колбэк "сохранения" от ONLYOFFICE — скачивает файл с `http://onlyoffice-docs` (сервис недоступен, см. ARCHITECTURE.md) |
| GET | `/uploads/:filename` | публичный путь, доступ через `?token=` (JWT `JWT_SECRET_OO`) либо cookie `1pdf_token` с префиксом `userId_` в имени файла | Отдача файла из `temp_uploads/` |

## Общий формат ошибок

`/api/v1/*` — `{ error: { code: "..." } }`. Остальные эндпоинты —
`{ error: "..." }` (строка) либо провайдер-специфичный формат для платёжных
колбэков (см. выше).
