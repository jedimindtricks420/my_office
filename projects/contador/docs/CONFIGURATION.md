# Configuration — Contador

Новый файл, создан 2026-09-08 (`TASK-0002`). Значения переменных **не
читались** — только имена (`grep -oE`), в соответствии с ограничением задачи
и правилом my_office «не помещай секреты в репозиторий документации».
`.env.example`/`.env.sample`/`.env.template` в репозитории не найдены — оба
`.env` реальные, с фактическими значениями.

## Корневой `.env` (root/admin/v1)

| Переменная | Где используется | Обязательность |
|---|---|---|
| `DATABASE_URL` | `prisma/schema.prisma:7` (v1/admin схема, `PrismaClient()` в `admin/server.ts:25`) | обязательна (Prisma упадёт без неё) |
| `V2_DATABASE_URL` | `admin/server.ts:30-34` — явная проверка `if (!process.env.V2_DATABASE_URL) throw` | **обязательна**, приложение падает при старте без неё |
| `ADMIN_PORT` | `admin/server.ts:35` | нет — дефолт `3031` |
| `ADMIN_PASSWORD` | `admin/server.ts:36-39` — явная проверка `if (!...) throw` | **обязательна** — общий пароль для всего защищённого `/admin/api` |
| `JWT_SECRET` | не встречен в `admin/server.ts` напрямую; используется, вероятно, легаси v1-кодом (физически отсутствует в репозитории, см. GAPS.md) — присутствие в `.env` не подтверждено использованием в найденном коде |
| `BASIC_AUTH_USER`, `BASIC_AUTH_PASSWORD` | не найдено использования ни в `admin/server.ts`, ни в `v2/` — вероятно, легаси-переменные для отсутствующего `src/` (v1) |
| `OPENAI_API_KEY` | не найдено использования ни в `admin/server.ts`, ни в `v2/` (v2 использует `AZURE_OPENAI_API_KEY`, см. ниже) — вероятно, легаси-переменная для отсутствующего v1 |
| `PORT` | не найдено использования в `admin/server.ts` (там `ADMIN_PORT`) — вероятно, для отсутствующего v1 |
| `PAYME_MERCHANT_ID`, `PAYME_SECRET_KEY`, `CLICK_SERVICE_ID`, `CLICK_SECRET_KEY` | **не найдено использования через `process.env`** ни в одном месте `admin/server.ts` — реквизиты платёжных провайдеров читаются из таблицы БД `PaymentConfig` (`getPaymentConfig()`, `admin/server.ts:197-199`), не из окружения. Эти переменные в `.env`, вероятно, унаследованный/неиспользуемый набор — не проверялось, есть ли путь кода, который всё же читает их как фолбэк, построчно по каждому вызову `config.*` |

## `v2/.env`

| Переменная | Где используется | Обязательность |
|---|---|---|
| `DATABASE_URL` | `v2/prisma/schema.prisma:8` | обязательна |
| `JWT_SECRET` | `v2/src/lib/auth.ts:6-9`, `v2/src/lib/auth-edge.ts:5-8` — обе явно бросают исключение при отсутствии | **обязательна**, приложение падает при импорте модуля без неё |
| `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_KEY` | `classification/aiClassifier.ts:477-483`, `classification/ai-reconcile/route.ts:47-57` | обязательны только для функций AI-классификации/AI-сверки (PRO-тариф) — при отсутствии эти два эндпоинта возвращают `400`, остальное приложение работает |
| `OPENAI_MODEL` | `v2/src/lib/constants.ts:143` — геттер `AI.MODEL`, дефолт `"gpt-4o-mini"` | нет (есть дефолт) |
| `PORT` | не найдено явного чтения в `v2/` (запуск управляется флагом `-p 3032` в `package.json` scripts и `Dockerfile.v2`'s `ENV PORT=3032`) | косвенно используется Next.js standalone-сервером |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURE`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM` | `v2/src/lib/mailer.ts:13-29` | не обязательны — `createTransport()` возвращает `null` при отсутствии `host`/`user`/`pass`, письма (сброс пароля, приглашение) просто не отправляются (`isMailConfigured()` используется UI для деградации на «покажи пароль в интерфейсе») |
| `ADMIN_API_URL` | `payments/initiate/route.ts:16`, `/voucher/route.ts:12`, `payments/subscription/route.ts:19` | не обязательна технически, но без неё все платёжные операции и реальная цена подписки не работают (явный `console.error` + `500`) |
| `NEXT_PUBLIC_APP_URL` | `auth/logout/route.ts:6`, `auth/forgot-password/route.ts:35`, `mailer.ts:93` — ссылки в письмах и редиректах | нет (дефолт `"https://contador.uz"`) |

⚠️ `SMTP_SECURE` объявлена в `.env`, но не встречена как читаемая
`process.env.SMTP_SECURE` в `mailer.ts` — там `secure: port === 465`
вычисляется из `SMTP_PORT`, а не из отдельного флага. `SMTP_SECURE`, похоже,
не используется кодом (см. GAPS.md).

## Переменные окружения `docker-compose.yml` (заданы инлайн, не из `.env`)

`admin`-сервис получает `DATABASE_URL`, `V2_DATABASE_URL`, `ADMIN_PORT`
захардкоженными прямо в `docker-compose.yml` (значения, не имена — указывают
на контейнер `db` по внутреннему DNS `db:5432`), и только `ADMIN_PASSWORD`
подставляется из внешней переменной окружения хоста (`${ADMIN_PASSWORD}`,
не файла `.env`). `v2`-сервис получает **весь** набор через `env_file:
./v2/.env` целиком.

## Секреты, обнаруженные вне `.env` (в версионируемых файлах)

- `test-payme.sh`, `test-payments.sh` (корень репозитория) — захардкоженные
  значения, похожие на реальные merchant ID/секретные ключи Payme/Click,
  прямо в тексте shell-скриптов. Значения **не читались**, тестовые они или
  боевые — не устанавливалось; риск описан в [GAPS.md](GAPS.md).
- Реквизиты платёжных провайдеров (Payme/Click/Alifpay merchant ID и
  секретные ключи) хранятся в таблице `PaymentConfig` (обе схемы БД) — то
  есть частично не в `.env`, а в базе данных, редактируемой через панель
  `admin`.

## Что читается из `.env`, но не находит подтверждённого использования в коде

См. таблицу выше (root `.env`: `JWT_SECRET`, `BASIC_AUTH_USER/PASSWORD`,
`OPENAI_API_KEY`, `PORT`, `PAYME_*`, `CLICK_*`) и `v2/.env: SMTP_SECURE` —
кандидаты на «мёртвая конфигурация», подробности в [GAPS.md](GAPS.md).
