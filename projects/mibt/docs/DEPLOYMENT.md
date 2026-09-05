# DEPLOYMENT — MIBT

Последнее обновление: 2026-09-05. Источник: `docker-compose.yml`,
`docker-compose.dev.yml`, `Dockerfile`, `nginx.conf`, `/etc/nginx/sites-available/portal.mibt.uz`,
вывод `docker ps -a` и `certbot certificates` на текущем хосте (только
чтение, без изменений).

## Текущее состояние прода (проверено напрямую, 2026-09-05)

```
docker ps -a --filter name=mibt
```

| Контейнер | Образ | Статус |
|---|---|---|
| `mibt2-app` | `mibt-2-app` (build из `Dockerfile`) | Up 2 недели |
| `mibt2-postgres` | `postgres:16-alpine` | Up 2 недели (healthy) |
| `mibt2-nginx` | `nginx:alpine` | Up 2 недели |
| `mibt-2-db-backup-1` | `prodrigestivill/postgres-backup-local` | Up 2 недели (healthy) |

Ежедневные бэкапы БД (`backups/daily/*.sql.gz`) идут вплоть до
2026-09-05 (сегодня) — бэкап-контейнер активен и исправно работает.
Исходный код на диске (по mtime файлов) не менялся с 13 июня 2026 — то есть
с момента последнего деплоя разработка не велась, но прод стабильно работает.

## Docker Compose (`docker-compose.yml`, прод)

4 сервиса:

1. **`db`** — `postgres:16-alpine`, БД `mibt_db`, пользователь `mibt_user`,
   пароль из `POSTGRES_PASSWORD` (фолбэк `devpassword123` в самом файле —
   используется, только если переменная не задана в окружении/`.env`).
   Данные — именованный volume `mibt2_pgdata`. **Порт наружу не пробрасывается.**
2. **`app`** — сборка из локального `Dockerfile`, `env_file: .env`,
   `NODE_ENV=production`. Порт `127.0.0.1:13000:3000` — доступен только с
   localhost хоста. Bind-mounts для разработки: `views/`, `routes/`,
   `services/`, `controllers/`, `lib/`, `middleware/`, `templates/`,
   `prisma/schema.prisma`, `index.js` — изменения этих файлов на хосте
   применяются без пересборки образа, только после `docker compose restart app`
   (комментарий в самом файле). Также смонтированы `secure-assets/` и
   `public/img/products/` как персистентные данные.
3. **`nginx`** — `nginx:alpine`, слушает `127.0.0.1:3011:443`, конфиг —
   `nginx.conf` из корня проекта, TLS — self-signed сертификаты из `ssl/`.
4. **`db-backup`** — `prodrigestivill/postgres-backup-local`, расписание
   `@daily`, хранит 30 дней / 8 недель / 12 месяцев, пишет в `./backups`.

## Dockerfile (многостадийная сборка)

- Стадия `builder` (`node:20-alpine`): `npm ci` (все зависимости) →
  `npm run build:css` (Tailwind) → `npx prisma generate`.
- Стадия `runner` (`node:20-alpine`): `npm ci --omit=dev` → копирует собранный
  CSS и сгенерированный Prisma-клиент из `builder` → копирует весь остальной
  код. `EXPOSE 3000`, `CMD ["node", "index.js"]`.
- Обе стадии ставят `openssl` (нужен Prisma-движку на Alpine).

## Сетевая архитектура и TLS

Два независимых пути входа существуют одновременно:

1. **Основной, публичный** — system nginx хоста (вне Docker Compose проекта):
   - `/etc/nginx/sites-available/portal.mibt.uz` (симлинк в `sites-enabled/`,
     сайт включён — подтверждено).
   - `server_name portal.mibt.uz`, слушает 80 (редирект на 443) и 443.
   - TLS — Let's Encrypt через certbot, сертификат `portal.mibt.uz`, тип
     ECDSA, **годен до 2026-11-10** (подтверждено `certbot certificates`),
     автопродление предполагается через стандартный certbot-таймер (не
     проверялось отдельно).
   - `proxy_pass http://127.0.0.1:13000` — то есть напрямую на порт,
     проброшенный Docker-контейнером `mibt2-app`, **в обход** внутреннего
     `mibt2-nginx`.
   - Отдельные `limit_req`-зоны: `/api/auth/login` (5 req/min, burst 5),
     `/api/auth/forgot-password` (3 req/min, burst 3), зона `mibt_auth`.
2. **Резервный/аварийный** — `mibt2-nginx` внутри Docker Compose, порт
   `127.0.0.1:3011`, self-signed TLS. Не публикуется наружу напрямую (только
   `127.0.0.1`) — назначение похоже на аварийный доступ с самого хоста, не
   для конечных пользователей. Не проверялось, кто и как им пользуется на
   практике.

Порт 3000 на хосте (не 13000) занят другим, несвязанным контейнером
(`backend_app`, другой проект) — по этой причине `app` в docker-compose.yml
проброшен именно на `13000`, а не на стандартный `3000` (см. память агента /
комментарии в конфигурации; не переисследовалось заново в этой задаче).

## Переменные окружения (только имена ключей, без значений — см. STACK.md/AGENTS.md о недопустимости чтения секретов)

`ADMIN_EMAIL`, `ADMIN_PASSWORD`, `APP_URL`, `DATABASE_URL`,
`ENCRYPTION_KEY`, `NODE_ENV`, `PORT`, `POSTGRES_PASSWORD`, `SESSION_SECRET`,
`SMTP_FROM`, `SMTP_HOST`, `SMTP_PASS`, `SMTP_PORT`, `SMTP_USER`.

`.env` и `.env.example` содержат **одинаковый набор ключей** (сверено
`diff` по именам без значений). `ALLOWED_ORIGINS` (используется в
`index.js` для CORS) в `.env.example` не упомянут — потенциально
недокументированная переменная (см. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).

## Dev-окружение

`docker-compose.dev.yml` — отдельный минимальный compose только с
`postgres:16-alpine` на порту `5437` (для локальной разработки без сборки
`app`-контейнера); `npm run dev` (nodemon) предполагается запускать вне
Docker, подключаясь к этой БД напрямую.

## Резервное копирование

`backups/{daily,weekly,monthly,last}/*.sql.gz` — подтверждено наличие файлов
вплоть до 2026-09-05 (сегодня) в `daily/`, и наличие последних снимков в
`weekly/`/`monthly/`/`last/`. Восстановление из бэкапа не тестировалось и не
документировано отдельным runbook'ом (см.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).
