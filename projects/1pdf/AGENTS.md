# AGENTS.md — 1PDF

Инструкции для ИИ-агентов, работающих над документацией/задачами проекта 1PDF
в `my_office`. Общие правила — в корневом [`/AGENTS.md`](../../AGENTS.md); этот
файл их не дублирует, только специфику проекта.

## Проект

- **Исходный код (source of truth для кода):** `/home/admin1/1pdf/` — отдельный
  каталог вне `my_office`. Этот репозиторий (`my_office/projects/1pdf/`)
  содержит только документацию и задачи, НЕ код.
- **Не изменяй `/home/admin1/1pdf/` из задач `my_office`**, если задача явно не
  требует правок кода и это не согласовано отдельно.
- Внутри `/home/admin1/1pdf/` есть собственная документация в `docs/`
  (ARCHITECTURE.md, API.md, OPERATIONS.md, PAYMENTS.md, SECURITY.md,
  DEVELOPMENT.md) — она в целом актуальна (в отличие, например, от части
  собственной документации Contador), но описывает только текущий
  React/Vite-редактор и платежи; она не упоминает legacy-эндпоинты ONLYOFFICE
  и не покрывает admin-API целиком. Сверяйся с кодом.

## Известные особенности проекта (важно для агентов)

- Проект — форк открытого проекта **freePDF** (Chinmay Shringi, MIT,
  `LICENSE.freePDF`, `THIRD_PARTY_NOTICES.md`). Коммит
  `d3d8efd feat: migrate to stable freePDF editor platform` — переход с более
  раннего редактора на базе ONLYOFFICE Document Server на текущий
  React/Konva/PDF.js редактор в `editor/`.
- **Два независимых поколения "редактора документов" сосуществуют в коде:**
  1. Текущий: `POST/GET /api/v1/documents*` (`admin/routes/editorApi.ts`,
     модели `Document`/`DocumentVersion`) — используется активным
     React-редактором.
  2. Legacy (вероятно неактивный): `/api/documents/config`,
     `/api/documents/upload`, `/api/documents/create-new`,
     `/api/documents/callback`, `/uploads/:filename?token=...` в
     `admin/server.ts`, завязанные на `JWT_SECRET_OO` и контейнер
     `onlyoffice-docs`. Контейнер `onlyoffice-docs` подтверждён в
     `docker ps -a` в статусе `Exited`, и ни один найденный на хосте
     `docker-compose.yml` его больше не определяет. Не удалять и не чинить
     эти эндпоинты без отдельной задачи — не подтверждено, мёртвый ли это код
     или ещё используется старыми сохранёнными ссылками. См.
     [OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md).
- **Дрейф схемы БД от истории миграций Prisma**: таблица `PaymentConfig` и
  несколько колонок `PaymentAttempt` (`amount_tiyin`, `cancel_reason`,
  `cancel_time`, `external_payment_id`, `paid_at`, `processing_status`)
  подтверждены в реальной БД (`\d`/`\dt` через `docker exec 1pdf-postgres
  psql`), но НЕ создаются ни одним файлом в `prisma/migrations/`. Таблица
  `_prisma_migrations` содержит ровно 5 записей, совпадающих с файлами на
  диске — то есть эти объекты появились не через отслеживаемый `prisma
  migrate deploy`. Подробности — [DATABASE.md](docs/DATABASE.md) и
  [OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md). Это значит: **воссоздание
  окружения с нуля через `prisma migrate deploy` на пустой БД, вероятно, не
  даст текущую рабочую схему** — не проверялось напрямую (не выполнялось,
  чтобы не рисковать продакшн-данными).
- **`test-payments.sh` в корне репозитория (закоммичен, чист в `git status`)
  содержит захардкоженные значения, похожие на реальные ключи/ID платёжных
  провайдеров** (переменные `PAYME_KEY`, `CLICK_SECRET`, `CLICK_SERVICE_ID`,
  а также правдоподобный `USER_ID`) в открытом виде. Сами значения не
  скопированы в документацию `my_office` и не должны туда попадать. Не
  проверялось, тестовые это ключи песочницы или боевые. См.
  [OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md) — при работе с этим файлом или
  секретами проекта относись к нему как к потенциально чувствительному.
- **TLS терминируется вне этого репозитория**: настоящий publicly-facing
  nginx — конфигурация хоста `/etc/nginx/sites-available/1pdf.uz`
  (Certbot-managed), которая проксирует на `127.0.0.1:8003` — это уже порт,
  который слушает контейнер `1pdf-nginx` (`nginx.freepdf.conf` внутри
  `docker-compose.yml` этого проекта). Docker-конфигурация проекта сама по
  себе не содержит TLS/сертификатов.

- **`admin.html` вызывает два удалённых эндпоинта** (`PATCH
  /api/auth/me/login`, `PATCH /api/auth/me/password`) — подтверждено через
  `git log -p -S "auth/me/login" -- admin/server.ts`: оба маршрута
  существовали и были удалены коммитом `f1203b7` при рефакторинге
  auth-системы, фронтенд не обновлён. Самообслуживание админа (смена
  своего логина/пароля) в интерфейсе есть, но отдаёт `404`. См.
  [`docs/GAPS.md`](docs/GAPS.md).
- **Дрейф схемы БД от истории миграций подтверждён точно, с номерами
  строк**: базовая `PaymentConfig` и 6 полей `PaymentAttempt`
  (включая переименование `external_id`→`external_payment_id`) существуют
  в реальной БД, но не создаются ни одним из 5 файлов
  `prisma/migrations/*/migration.sql`. `prisma migrate deploy` на пустой
  базе не воспроизведёт текущую схему. См. [`docs/DATABASE.md`](docs/DATABASE.md).

## Структура этого каталога

Стандартная структура my_office-проекта — см. корневой `AGENTS.md`. Задачи
`TASK-0001-bootstrap-project-context` и `TASK-0002-full-technical-reaudit`
(в `tasks/completed/` после завершения) содержат журналы исследования — при
сомнениях в происхождении факта смотри туда. `TASK-0002` — независимый
повторный аудит с нуля (2026-09-08), актуальнее и полнее `TASK-0001`.
