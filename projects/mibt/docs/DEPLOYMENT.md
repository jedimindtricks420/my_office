# DEPLOYMENT — MIBT

Последнее обновление: 2026-09-08 (TASK-0002). Источник: `docker-compose.yml`,
`docker-compose.dev.yml`, `Dockerfile`, `nginx.conf`,
`/etc/nginx/sites-available/portal.mibt.uz`, `package.json` (`scripts`),
`prisma.config.ts`, вывод `docker ps -a` и `certbot certificates` на
текущем хосте (только чтение, без изменений).

## RUNBOOK: установка и запуск с нуля

Выведено из `package.json`, `Dockerfile`, `docker-compose*.yml` —
**не тестировалось запуском** (задача прямо запрещает поднимать
dev-серверы/docker для проверки на этом хосте, см. `AGENTS.md`).

### Вариант А — через Docker Compose (соответствует проду)

1. Скопировать `.env.example` → `.env`, заполнить `DATABASE_URL`,
   `SESSION_SECRET` (случайная строка), `ENCRYPTION_KEY` (ровно 64
   hex-символа — иначе `services/license.service.js:4` бросит исключение
   при старте), при необходимости SMTP/`ADMIN_*`/`APP_URL`.
2. `docker compose up -d --build` — поднимет `db` (Postgres 16, здоровье
   через `pg_isready`), затем `app` (ждёт `db` через
   `depends_on.condition: service_healthy`), затем `nginx`, `db-backup`.
3. **Применить миграции БД** — ни один из файлов проекта не делает этого
   автоматически (см. [GAPS.md](GAPS.md) 2.7). Требуется вручную:
   `docker compose exec app npx prisma migrate deploy`. При текущем
   наборе файлов миграций (3 шт.) таблицы `quote_*` создан**ы не будут** —
   это расхождение нужно устранить до продакшен-разворачивания «с нуля»
   (сгенерировать недостающую миграцию через `prisma migrate dev` в
   dev-окружении и закоммитить её, либо явно вызвать `prisma db push`,
   осознавая, что это не создаёт файл миграции).
4. Заполнить начальные данные: `docker compose exec app npm run seed` (алиас
   `node prisma/seed.js`) — создаёт ADMIN-пользователя
   (`ADMIN_EMAIL`/`ADMIN_PASSWORD`, дефолт при отсутствии переменных —
   `admin@gptech.uz` / `AdminPass123!`, см. [CONFIGURATION.md](CONFIGURATION.md))
   и дефолтные `SystemSetting` (реквизиты компании — плейсхолдеры).
5. Приложение слушает `127.0.0.1:13000` (проброс из контейнера `app`) и
   `127.0.0.1:3011` (через встроенный `mibt2-nginx`, self-signed TLS).
   Для публичного доступа нужен внешний реверс-прокси (см. «Сетевая
   архитектура» ниже) — Compose-файл сам по себе не публикует ничего
   наружу, кроме `127.0.0.1`.

### Вариант Б — локально без Docker (для разработки)

1. `docker compose -f docker-compose.dev.yml up -d` — поднимает только
   Postgres на `localhost:5437`.
2. `.env` с `DATABASE_URL="postgresql://mibt_user:devpassword123@localhost:5437/mibt_db"`
   (пароль по умолчанию из `docker-compose.dev.yml:9`, если не переопределён).
3. `npm ci` (или `npm install`).
4. `npx prisma migrate deploy` (см. оговорку про недостающую миграцию КП
   выше) и `npm run seed`.
5. `npm run build:css` (единоразово, либо `npm run watch:css` в фоне при
   активной разработке вёрстки).
6. `npm run dev` (nodemon, автоперезапуск при изменении файлов) или
   `npm start` (`node index.js`, без автоперезапуска).
7. Приложение слушает `http://localhost:3000` (или `PORT` из `.env`).

### Известные требования к окружению

- Node.js 20 (зафиксировано только в `Dockerfile` через `node:20-alpine`;
  `package.json` не содержит поля `engines`).
- PostgreSQL 16 (или совместимая версия — schema не использует
  версионно-специфичных возможностей за пределами стандартного SQL/JSON).
- `ENCRYPTION_KEY` обязателен и должен быть ровно 32 байта в hex — без него
  процесс не запустится (см. `services/license.service.js:3-4`).
- Каталоги `secure-assets/` и `public/img/products/` должны быть доступны
  на запись процессу Node — создаются автоматически при первом обращении
  (`fs.mkdirSync(..., {recursive: true})` в `documents.api.js`,
  `quotes.api.js`, `upload-image.js`), но в Docker-сетапе смонтированы как
  volumes для персистентности между пересборками образа.

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
вплоть до 2026-09-05 в `daily/`, и наличие последних снимков в
`weekly/`/`monthly/`/`last/`. На 2026-09-08 (повторная проверка) в
`daily/` присутствуют файлы вплоть до `mibt_db-20260908.sql.gz` — бэкапы
продолжают исправно создаваться. Восстановление из бэкапа не тестировалось
и не документировано отдельным runbook'ом (см.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).

## См. также

- [CONFIGURATION.md](CONFIGURATION.md) — полная таблица переменных
  окружения с указанием обязательности и места использования.
- [GAPS.md](GAPS.md) — расхождение миграций Prisma со схемой (раздел 2.7),
  публично доступный статичный файл `public/test-products.html` (раздел
  «Мёртвый код»), и другие операционные риски.
