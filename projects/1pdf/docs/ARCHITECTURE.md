# Architecture — 1PDF

Снимок на 2026-09-05. Источник — код `/home/admin1/1pdf/`,
`docker-compose.yml`, `nginx.freepdf.conf`, host-конфигурация
`/etc/nginx/sites-available/1pdf.uz`, `docker ps -a`.

## Runtime-топология (полная, включая то, что вне репозитория)

```
Интернет
  → host nginx (TLS, Certbot, /etc/nginx/sites-available/1pdf.uz)
      слушает 443, проксирует на 127.0.0.1:8003
  → контейнер 1pdf-nginx (nginx:alpine, docker-compose.yml этого проекта)
      порт 8003→80, nginx.freepdf.conf, security-заголовки,
      auth_request на /editor/*
  → контейнер 1pdf-admin (Node.js 22 + Express, порт 127.0.0.1:8031)
      обслуживает портал/админку/API/собранный редактор
  → контейнер 1pdf-postgres (postgres:16-alpine, 127.0.0.1:5435, БД onepdf)
```

Host-уровневый nginx (`/etc/nginx/sites-available/1pdf.uz`) — **вне
репозитория 1PDF**, управляется Certbot, не отслеживается в git-репозитории
проекта. Найден и подтверждён напрямую на хосте, а не в коде проекта.

Все три managed-контейнера (`1pdf-nginx`, `1pdf-admin`, `1pdf-postgres`)
подтверждены запущенными (`docker ps`, статус `Up`) на 2026-09-05.

## Каталоги (верхний уровень репозитория)

- `admin/` — Express-бэкенд: `server.ts` (монолитный, ~1435 строк, все
  REST-роуты кроме `/api/v1/*`), `routes/editorApi.ts` (текущий
  document-API), `controllers/paymeController.ts`, `utils/` (платёжные
  утилиты, коды ошибок Payme), `public/` (статические HTML портала/админки/
  лендинга — `index.html`, `portal.html`, `admin.html`), `seed.ts`.
- `editor/` — отдельное React-приложение (Vite), собирается отдельным
  Docker-стейджем и монтируется под `/editor/` в `1pdf-admin`.
- `prisma/` — `schema.prisma` + `migrations/` (см. [DATABASE.md](DATABASE.md)).
- `nginx.freepdf.conf` — конфигурация контейнера `1pdf-nginx`.
- `scripts/assert-migrations-additive.sh` — CI-подобная проверка, что
  конкретная миграция (`20260801000000_add_editor_documents_and_language`)
  аддитивна и не трогает бизнес-поля `Voucher`. Проверяет только эту одну
  миграцию по имени файла, не универсальный guard для всех будущих миграций.
- `extraConfigs/backup/` — единичные ручные дампы БД (`db/*.sql`,
  апрель 2026) и резервные JWT-ключи (`keys/*.key`/`.pub`, апрель 2026);
  исключены из git (`.gitignore`).
- `temp_uploads/` — примонтированный volume для файлов: legacy-загрузки
  (`multer`, корень) и `documents/`/`incoming/` (текущий document-API).
- `logs/` — `info.log` (нетривиальный), `invalid-auths.log` (пустой на
  момент снимка).
- `data_oo/`, `db_oo/`, `lib_oo/`, `logs_oo/`, `postgres_data/` — каталоги с
  ограниченными правами (не читались, принадлежат другим системным
  пользователям); по именованию (`_oo`) относятся к прежней интеграции
  ONLYOFFICE, исключены из git и Docker-контекста (`.gitignore`,
  `.dockerignore`).

## Два поколения document-API (оба присутствуют в коде)

### Текущее — `/api/v1/documents/*` (`admin/routes/editorApi.ts`)

Prisma-модели `Document`/`DocumentVersion`, монтируется в `server.ts` как
`app.use("/api/v1", createEditorApiRouter(prisma, UPLOADS_DIR))`. Активный
React-редактор (`editor/src/hooks/useDocumentBootstrap.ts`) обращается именно
сюда при наличии параметра `documentId` в URL.

Поток: загрузка/создание → `Document` + `DocumentVersion` (`status:
READY`) → `current_version_id` указывает на активную версию → редактор
получает `editor-session` с `downloadUrl` → сохранение создаёт новую версию
с проверкой `baseVersionId` (оптимистическая блокировка,
`409 VERSION_CONFLICT` при рассинхронизации).

### Legacy — эндпоинты в `admin/server.ts`, завязанные на `JWT_SECRET_OO`

`POST /api/documents/upload`, `POST /api/documents/create-new`, `GET
/api/documents/config`, `POST /api/documents/callback`, `GET
/uploads/:filename` (с опциональным `?token=`, проверяемым через
`JWT_SECRET_OO`). `/api/documents/config` формирует конфиг в формате
ONLYOFFICE Document Server (`documentType: "word"`, `editorConfig.callbackUrl`
и т.д.), а `/api/documents/callback` при получении статуса "готово к
сохранению" перезаписывает файл, скачивая его с внутреннего адреса
`http://onlyoffice-docs` (Docker service name).

**Подтверждено**: контейнер `onlyoffice-docs`
(`onlyoffice/documentserver:latest`) существует на хосте, но его статус —
`Exited (127) 2 недели назад` (`docker ps -a`), и ни один найденный на хосте
`docker-compose.yml` (включая файл этого проекта) больше не объявляет сервис
`onlyoffice-docs`. То есть внутренний адрес `http://onlyoffice-docs`,
используемый в `/api/documents/callback`, на 2026-09-05 не резолвится ни в
один работающий контейнер.

Клиентский редактор всё же содержит путь, читающий файл напрямую через
`GET /uploads/:legacyFileId` (см. `editor/src/hooks/useDocumentBootstrap.ts`,
параметр URL `fileId`) — то есть legacy-путь физически достижим со стороны
браузера независимо от состояния ONLYOFFICE. Не подтверждено, генерирует ли
что-либо в текущем UI такие ссылки на 2026-09-05. См.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

## Аутентификация и авторизация

- Cookie-based JWT: `1pdf_token` (пользователь, 30 дней), `1pdf_admin_token`
  (администратор, 7 дней). Поддержан также заголовок `Authorization: Bearer`.
- `authMiddleware` в `server.ts` пропускает без токена только явный список
  публичных путей (список путей — см. код); для всех остальных требуется
  хотя бы один валидный токен (пользователь или админ).
- `requireAdmin`/`requireSuperAdmin`/`requireUser`/`requirePro` — отдельные
  guard-функции по ролям и статусу подписки.
- Nginx (`nginx.freepdf.conf`) дополнительно защищает `/editor/*` через
  `auth_request /api/auth/verify` — при 401/403 редиректит на `/portal`, но
  это не единственный слой: `GET /api/auth/verify` в `server.ts` сам
  проверяет верификацию email и активность PRO-подписки для путей `/editor`.
- Отдельный секрет `JWT_SECRET_OO` (может быть пустой строкой по умолчанию)
  используется только legacy ONLYOFFICE-путём — не путать с основным
  `JWT_SECRET`.

## Персистентность и защита данных

Собственная документация проекта (`docs/ARCHITECTURE.md`,
`docs/OPERATIONS.md`) явно формулирует "data-safety promise": обновления
используют только аддитивные Prisma-миграции, не удаляют и не перезаписывают
существующих пользователей, ваучеры, статусы ваучеров, платежи или
загруженные документы. Однако подтверждена **фактическая рассинхронизация
между историей миграций и реальной схемой БД** — см.
[DATABASE.md](DATABASE.md) и [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md): это не
противоречит самому "promise" (данные не удалялись), но означает, что
воссоздание схемы с нуля через одни только файлы миграций не гарантировано.
