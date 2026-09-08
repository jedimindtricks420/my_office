# Deployment / Runbook — 1PDF

Полностью переписано 2026-09-08 (`TASK-0002`). Источники:
`docker-compose.yml`, `Dockerfile.admin.v2`, `package.json` (корень и
`editor/`), плюс read-only `docker ps`/`docker exec ... psql` и чтение
host-level `/etc/nginx/sites-available/1pdf.uz` (никакие контейнеры не
запускались/останавливались/пересобирались).

## Установка и локальный запуск (выведено из конфигов — не выполнялось)

Требования: Node.js **22** (`Dockerfile.admin.v2:1,8`), PostgreSQL **16**, npm.

```bash
cd /home/admin1/1pdf
npm install                      # backend (корневой package.json)
npm --prefix editor install      # frontend
npm run editor:build             # npm --prefix editor run build → editor/dist
npm run generate                 # prisma generate
npm run migrate                  # prisma migrate deploy (см. DATABASE.md — не воссоздаст текущую схему 1:1)
npm run seed                     # tsx admin/seed.ts — SUPERADMIN + CMS-контент (не вызывается автоматически)
npm run admin                    # tsx admin/server.ts → слушает ADMIN_PORT (дефолт 8031)
```

Обязательная переменная в production: `JWT_SECRET` (роняет процесс явно —
`admin/server.ts:41-43`). `DB_PASSWORD` нужен для подключения к БД.

### Тесты

```bash
npm run typecheck            # tsc --noEmit, корневой backend
npm --prefix editor run typecheck
npm run editor:test          # npm --prefix editor run test → vitest run, 7 test-файлов
                              # (App.test.tsx, editorStore.test.ts, coordinateTransform.test.ts,
                              #  exportPdf.test.ts, pageOperations.test.ts, textRuns.test.ts, catalog.test.ts)
                              # + tests/fixtures/*.pdf (5 PDF-файлов: digital, multipage, landscape, rotated, nonlatin)
```

## Docker Compose — оркестрация продакшна

`docker-compose.yml` (корень) — 3 сервиса, дефолтная bridge-сеть (имя сети
не задано явно):

| Сервис | Образ | Порт | Volumes |
|---|---|---|---|
| `nginx-proxy` (`1pdf-nginx`) | `nginx:alpine` | `8003:80` | `nginx.freepdf.conf` (ro) |
| `postgres` (`1pdf-postgres`) | `postgres:16-alpine` | `127.0.0.1:5435:5432` | `postgres_data/` |
| `admin` (`1pdf-admin`) | `Dockerfile.admin.v2` | `127.0.0.1:8031:8031` | `admin/`, `prisma/`, `temp_uploads/` (все — bind mount, не именованные volumes) |

```bash
docker compose up -d --build
```
(команда выведена из структуры файла, не выполнялась в рамках задачи)

`Dockerfile.admin.v2` — двухстадийная сборка: `editor-builder`
(`node:22-alpine`, `npm ci` + `npm run build` в `editor/`) → финальный слой
копирует `editor/dist` внутрь образа рядом с `admin/`. Контейнер стартует
непривилегированным пользователем `node` (`USER node`), команда запуска —
`npx prisma migrate deploy && npx tsx admin/server.ts` (миграции
применяются **автоматически при каждом рестарте контейнера**, в отличие от
проектов my_office с ручным `psql`-процессом).

`admin`-сервис монтирует `admin/`, `prisma/`, `temp_uploads/` как bind
mounts поверх образа — то есть в текущей конфигурации **код на хосте
подключается напрямую в контейнер**, а не только копируется на этапе
сборки; фактически исполняемый код — то, что лежит на диске хоста в момент
запуска контейнера, не обязательно то, что было в образе на момент `build`.

## Реальное состояние хоста (перепроверено read-only, 2026-09-08)

```
1pdf-nginx      Up 2 недели   0.0.0.0:8003->80
1pdf-admin      Up 2 недели   127.0.0.1:8031->8031
1pdf-postgres   Up 2 недели   127.0.0.1:5435->5432
onlyoffice-docs Exited (127) 2 недели назад   (не в docker-compose.yml)
```

TLS-терминация происходит **вне этого репозитория** — host-level nginx
(`/etc/nginx/sites-available/1pdf.uz`, управляется Certbot, подтверждено
прочтением файла) слушает 443, проксирует на `127.0.0.1:8003` — ровно
порт, который публикует контейнер `1pdf-nginx`. Сертификаты Let's Encrypt
через Certbot, автообновление не проверялось (вне репозитория).

## Резервные копии

Один файл — `backups/pre_freepdf_20260801_1040.dump` (перед миграцией
`20260801000000_add_editor_documents_and_language`). Ручной снимок перед
конкретным риском, не расписание; отдельного cron/systemd-таймера для
регулярных бэкапов в репозитории не найдено.

## Известное ограничение раннего восстановления окружения

`npm run migrate` (`prisma migrate deploy`) на **пустой** базе данных не
воспроизведёт текущую продакшн-схему один в один — см.
[DATABASE.md](DATABASE.md), раздел «Подтверждённый дрейф схемы»
(таблица `PaymentConfig` и часть колонок `PaymentAttempt` отсутствуют в
истории миграций). Для полного восстановления с нуля потребуется либо
восстановление из `backups/*.dump`, либо ручное исправление недостающих
объектов схемы — оба варианта не автоматизированы в репозитории.
