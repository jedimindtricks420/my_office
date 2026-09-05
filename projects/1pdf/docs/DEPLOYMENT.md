# Deployment — 1PDF

Снимок на 2026-09-05. Источник — `docker-compose.yml`, `Dockerfile.admin.v2`,
`nginx.freepdf.conf`, host-конфигурация `/etc/nginx/sites-available/1pdf.uz`,
`docker ps -a` на хосте.

## Топология (см. также [ARCHITECTURE.md](ARCHITECTURE.md))

TLS терминируется **вне репозитория проекта**: host-nginx
(`/etc/nginx/sites-available/1pdf.uz`, управляется Certbot,
`ssl_certificate .../1pdf.uz/fullchain.pem`) слушает 443 и проксирует на
`127.0.0.1:8003`. Это порт, который проект публикует в
`docker-compose.yml` для контейнера `1pdf-nginx`. Порт 80 на хосте для
`1pdf.uz`/`www.1pdf.uz` — редирект на 443 (Certbot).

## Docker Compose (`docker-compose.yml`, 3 сервиса)

| Сервис | Образ | Порты | Volumes |
|---|---|---|---|
| `nginx-proxy` (контейнер `1pdf-nginx`) | `nginx:alpine` | `8003:80` | `nginx.freepdf.conf` (ro) |
| `postgres` (контейнер `1pdf-postgres`) | `postgres:16-alpine` | `127.0.0.1:5435:5432` | `postgres_data/` |
| `admin` (контейнер `1pdf-admin`) | собственная сборка (`Dockerfile.admin.v2`) | `127.0.0.1:8031:8031` | `admin/`, `prisma/`, `temp_uploads/` (все смонтированы с хоста, не встроены в образ на рантайме) |

`admin` монтирует исходники (`admin/`, `prisma/`) с хоста поверх образа —
то есть код в контейнере фактически совпадает с текущим деревом на диске в
момент старта контейнера, а не только с тем, что было на момент `docker
build`. Собранный `editor/dist` — исключение: он копируется в образ во время
сборки (отдельный стейдж `editor-builder`), не монтируется с хоста.

Образ собирается из `Dockerfile.admin.v2`: стадия 1 (`editor-builder`,
`node:22-alpine`) — `npm ci` + `npm run build` для `editor/`; стадия 2 —
`npm ci` для корня, копирование `admin/`, `prisma/`, собранного
`editor/dist`, `npx prisma generate`, создание `/app/uploads/{incoming,
documents}`. Контейнер стартует командой `npx prisma migrate deploy && npx
tsx admin/server.ts` — то есть миграции применяются автоматически при каждом
запуске контейнера.

## Переменные окружения (только имена — значения не читались)

Из `.env` в корне проекта (файл не публикуется, исключён `.gitignore`):
`DB_PASSWORD`, `JWT_SECRET`, `SMTP_PASS`, `SUPERADMIN_LOGIN`,
`SUPERADMIN_PASSWORD`, `PAYME_MERCHANT_ID`, `PAYME_KEY`, `PAYME_TEST_KEY`,
`PAYME_ENV`, `CLICK_MERCHANT_ID`, `CLICK_SERVICE_ID`, `CLICK_SECRET_KEY`,
`CLICK_ENV`, `FREE_PORT`, `DOCKER_EXAMPLE_ENABLED`.

Подтверждено кодом, что реквизиты Payme/Click/Alif читаются в первую очередь
из БД (`PaymentConfig`, управляется через защищённую страницу
super-администратора), а переменные окружения (`PAYME_*`, `CLICK_*`) —
только fallback. `FREE_PORT` и `DOCKER_EXAMPLE_ENABLED` не встречаются ни в
одном файле кода/конфигурации проекта — см.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

`JWT_SECRET_OO` (секрет legacy ONLYOFFICE-эндпоинтов) в списке переменных
`.env` не найден — в коде используется с пустой строкой по умолчанию
(`process.env.JWT_SECRET_OO || ""`), то есть, вероятно, не задан в текущем
окружении.

## Health-check и эксплуатационные команды (из README/OPERATIONS.md проекта)

`docker compose ps`, `docker compose logs --tail=100 admin nginx-proxy`,
`curl -I https://1pdf.uz/portal`. Ожидаемо: все три managed-сервиса `Up`,
портал отвечает 200, анонимный запрос к `/editor/` редиректит на портал.
Подтверждено на 2026-09-05: все три контейнера `Up`.

## Резервное копирование

См. [DATABASE.md](DATABASE.md#резервные-копии) — только ручные дампы,
автоматического расписания не найдено.

## Ограничения по объёму (проверено на хосте, 2026-09-05)

`temp_uploads/` — 8.6 МБ. `backups/` — 52 КБ (один dump). `postgres_data/` —
права на чтение ограничены (принадлежит системному пользователю Docker,
размер не определён).
