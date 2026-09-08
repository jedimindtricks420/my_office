# Configuration — 1PDF

Новый файл, создан 2026-09-08 (`TASK-0002`). Значения переменных **не
читались** — только имена (`grep -oE`). `.env.example` в репозитории не
найден — единственный `.env` содержит реальные значения.

## `.env` (корень, используется и `admin`, и `docker-compose.yml`)

| Переменная | Где используется | Обязательность |
|---|---|---|
| `DB_PASSWORD` | `docker-compose.yml:20,37` (пароль Postgres, с дефолтом `onepdf_secure_pass` через `${DB_PASSWORD:-onepdf_secure_pass}`) | нет (есть дефолт в compose-файле, но использовать дефолт в проде — риск) |
| `JWT_SECRET` | `admin/server.ts:40-44` — при отсутствии в production (`NODE_ENV=production`) роняет процесс; `docker-compose.yml:40` — `${JWT_SECRET:?JWT_SECRET must be set in .env}` **останавливает `docker compose up` без него** (жёстче, чем в других проектах my_office) | обязательна в production |
| `JWT_SECRET_OO` | `admin/server.ts:45` — подпись конфигов/токенов legacy ONLYOFFICE-интеграции (`/uploads/:filename?token=`, `/api/documents/config`, `/api/documents/callback`) | нет проверки на пустоту при старте — код использует `""` как дефолт, что делает `jwt.sign(..., "")`/`jwt.verify(..., "")` технически рабочим, но криптографически бессмысленным (см. GAPS.md) |
| `SUPERADMIN_LOGIN`, `SUPERADMIN_PASSWORD` | только `admin/seed.ts:10-11`, с дефолтами `'admin'`/`'changeme123'`, если не заданы | нет (но дефолт — риск, см. GAPS.md) |
| `FREE_PORT` | **не найдено использования** ни в `admin/`, ни в `editor/`, ни в `docker-compose.yml` | вероятно, дохлая конфигурация (см. GAPS.md) |
| `DOCKER_EXAMPLE_ENABLED` | **не найдено использования** нигде в репозитории — по имени, вероятно, относится к `lib_oo/documentserver-example` (пример развёртывания ONLYOFFICE), которого сейчас нет в активном стеке | дохлая конфигурация |
| `PAYME_MERCHANT_ID`, `PAYME_KEY`, `PAYME_TEST_KEY`, `PAYME_ENV` | `admin/utils/paymentUtils.ts:35-36,76-79` и `admin/server.ts:894-895` — **используются как фолбэк**, только если строка `PaymentConfig` в БД отсутствует | нет (приоритет у значений в БД) |
| `CLICK_MERCHANT_ID`, `CLICK_SERVICE_ID`, `CLICK_SECRET_KEY`, `CLICK_ENV` | аналогично Payme — фолбэк в `paymentUtils.ts:49-50` и явно в `server.ts:1003,1036` (`config?.click_secret_key \|\| process.env.CLICK_SECRET_KEY`) | нет |
| `SMTP_PASS` | `admin/server.ts:173-191` — SMTP-транспорт для OTP/писем (host/user захардкожены: `de.ahost.cloud`, `no-reply@1pdf.uz`); при отсутствии в production — только `console.warn`, письма не отправляются, регистрация технически проходит (signup token всё равно выдаётся), но пользователь не получит код | нет строгой проверки (в отличие от `JWT_SECRET`) |

⚠️ Ни `ADMIN_PORT` (используется с дефолтом `8031`,
`admin/server.ts:37`), ни `EDITOR_DIST_DIR` (дефолт — путь относительно
`__dirname`, `admin/server.ts:88-89`), ни `NODE_ENV` не объявлены в
`.env` файле, но читаются кодом — `NODE_ENV`/`ADMIN_PORT` задаются явно в
`docker-compose.yml:38-39`, `EDITOR_DIST_DIR` нигде не задаётся (полагается
на дефолт).

## `editor/` — переменные времени сборки (Vite)

Отдельного `.env`-файла для `editor/` в репозитории не найдено. Через
`import.meta.env` читаются:

| Переменная | Где используется | Обязательность |
|---|---|---|
| `VITE_KOFI_HANDLE` | `editor/src/config.ts:8` — ссылка на донат (Ko-fi), дефолт `'chinmay_shringi'` (автор оригинального freePDF) | нет |
| `VITE_CF_ANALYTICS_TOKEN` | `editor/src/lib/analytics.ts:7` — токен Cloudflare Web Analytics | нет (аналитика просто не подключится) |

⚠️ **`VITE_KOFI_HANDLE` не переопределён нигде в найденной конфигурации
сборки** — собранный `editor/dist` в продакшне, по всей видимости, до сих
пор показывает донат-ссылку на аккаунт автора оригинального open-source
проекта freePDF, а не на 1PDF — не проверялось напрямую (не читалось
содержимое собранного `dist`, если он существует на диске), см.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

## Секреты вне `.env`

- `test-payments.sh` (корень репозитория, версионируется в git) —
  содержит похожие на реальные значения переменных `PAYME_KEY`,
  `CLICK_SECRET`, `CLICK_SERVICE_ID`, а также правдоподобный `USER_ID`, в
  открытом виде. Значения не читались и не копировались в эту
  документацию — риск описан в [GAPS.md](GAPS.md).
- Реквизиты Payme/Click/Alifpay хранятся в таблице `PaymentConfig` в БД,
  редактируемой из `admin.html` — частично не в `.env`.

## `.dockerignore` — что исключено из сборки образа

`node_modules`, `admin/node_modules`, `temp_uploads`, `db_oo`, `data_oo`,
`logs_oo`, `lib_oo`, `postgres_data`, `.git`, `.env`, `editor/node_modules`,
`editor/dist`. Явное исключение всех четырёх `*_oo` каталогов из контекста
сборки — дополнительное подтверждение того, что self-hosted ONLYOFFICE не
является частью текущего образа (см. [GAPS.md](GAPS.md)).
