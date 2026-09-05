# Deployment — Contador

Снимок на 2026-09-05. Источники: `docker-compose.yml`, `Dockerfile.admin`,
`Dockerfile.v2`, `docker ps` (реальное состояние хоста), `pm2 list`, `backups/`.

## Оркестрация

**Docker Compose** (`/home/admin1/contador/docker-compose.yml`) — 3 сервиса,
сеть `contador-net` (bridge):

| Сервис | Образ | Порт (host:container) | Особенности |
|---|---|---|---|
| `db` | `postgres:16-alpine` | не публикуется (`expose: 5432`) | volume `./postgres_data:/var/lib/postgresql/data`, healthcheck `pg_isready`, `POSTGRES_DB=contador` |
| `admin` | собран из `Dockerfile.admin` | `3031:3031` | `depends_on: db` (по healthy), переменные заданы инлайн (`DATABASE_URL`, `V2_DATABASE_URL`, `ADMIN_PORT`, `ADMIN_PASSWORD`) |
| `v2` | собран из `Dockerfile.v2` | `3032:3032` | **без** `depends_on` на `db`; `env_file: ./v2/.env`; `mem_limit: 1g`, `mem_reservation: 256m`; `extra_hosts: host.docker.internal:host-gateway` |

`v1` (legacy) **не описан** в `docker-compose.yml` — подтверждено отсутствие
сервиса. Однако см. «Расхождение: v1 фактически запущен» ниже — на хосте
работают контейнеры, не управляемые этим compose-файлом.

**PM2**: собственная документация проекта (`docs/01_PROJECT_OVERVIEW.md`,
`docs/02_ARCHITECTURE.md`, дата 2026-06-26/06-16) описывает v2 как работающий
через PM2. Подтверждено командой `pm2 list` (2026-09-05): **список процессов
PM2 пуст** — PM2 не использует ни один процесс. README проекта (более позднее,
2026-07-05) уже корректно указывает Docker Compose, а не PM2. → доки `01`/`02`
устарели на этот счёт.

## Dockerfile.admin (полностью)

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
RUN npm install express @types/express
COPY prisma ./prisma
RUN npx prisma generate
COPY v2/prisma/schema.prisma ./v2/prisma/schema.prisma
RUN npx prisma generate --schema ./v2/prisma/schema.prisma
COPY admin ./admin
COPY tsconfig.json ./
EXPOSE 3031
CMD ["npx", "tsx", "admin/server.ts"]
```

Однослойная сборка, запуск через `tsx` без предварительной компиляции.
Генерирует **два** Prisma-клиента (root-схема + `v2/prisma/schema.prisma`) —
admin-приложение имеет доступ к обеим БД (`DATABASE_URL` и
`V2_DATABASE_URL`).

## Dockerfile.v2 (полностью)

Многостадийная сборка (`deps` → `builder` → `runner`), все стадии
`node:20-alpine`:

- `deps`: `npm ci` по `v2/package*.json`.
- `builder`: копирует весь `v2/`, `prisma generate`, сборка через
  `npm run build` с `NODE_OPTIONS="--max-old-space-size=3072"`.
- `runner`: копирует standalone-вывод Next.js (`.next/standalone`,
  `.next/static`, `public`, сгенерированный Prisma-клиент), запуск
  `node server.js`, `PORT=3032`, `HOSTNAME=0.0.0.0`.

Комментарии в файле и в `v2/next.config.ts` (`experimental.cpus: 1`) связывают
ограничение памяти сборки и числа воркеров с инцидентом **2026-08-19**: сборка
ранее потребляла всю доступную память хоста (без свопа) и приводила к падению
VM — это согласуется с зафиксированной в памяти агента заметкой «не поднимать
dev-серверы/docker для проверки на этом хосте» (см. AGENTS.md проекта в
`my_office`, раздел памяти).

## Реальное состояние хоста (проверено `docker ps`, 2026-09-05)

```
NAMES                       IMAGE                CREATED         STATUS
contador-v2                 contador-v2          2026-08-20      Up 2 недели
contador-admin               contador-admin       2026-07-31      Up 2 недели
contador-app                 contador-app         2026-06-12      Up 2 недели
f7ee8a3d81b8_contador-app    762253acf8ca         2026-05-28      Up 2 недели (порт 3030:3030)
contador-db                  postgres:16-alpine   2026-04-18      Up 2 недели (healthy)
```

### ⚠️ Расхождение: v1 фактически запущен, вопреки README

README проекта утверждает, что v1 «⛔ Не запущена» и что «код есть в `src/`,
контейнер остановлен». Проверка показала:

- Каталог `/home/admin1/contador/src/` **физически отсутствует** в текущем
  рабочем дереве репозитория.
- Тем не менее **два** контейнера с образами, похожими на v1, реально
  запущены: `contador-app` (без публикации порта) и
  `f7ee8a3d81b8_contador-app` (образ `762253acf8ca`, публикует **порт 3030** —
  ровно тот, что README называет портом v1). Оба выполняют команду
  `npm start`. Переменные окружения контейнера `contador-app` (только имена)
  включают `NEXT_PUBLIC_PORT`, `BASIC_AUTH_USER`, `BASIC_AUTH_PASSWORD`,
  `OPENAI_API_KEY`, `JWT_SECRET`, `DATABASE_URL` — набор, соответствующий
  корневому `.env` (легаси/v1/admin), не `v2/.env`.
- Оба образа собраны из более раннего снимка кода (2026-05-28 и 2026-06-12) —
  предположительно до того, как каталог `src/` был удалён из репозитория, либо
  собраны из Dockerfile, которого сейчас в репозитории нет.

Это не подтверждает и не опровергает, работает ли эта версия «вживую»
(обслуживает ли реальный трафик) — только то, что процессы запущены на хосте
на момент проверки. Подробнее см. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

## Резервные копии (`backups/`)

13 файлов — паттерн: дамп БД перед рискованным изменением
(`contador_v2_pre_<изменение>_<таймштамп>.dump`) + соответствующий SQL-патч.
Самый свежий бэкап — `contador_v2_pre_voucher_series_20260731_105644.dump`
(31 июля). Бэкапы не покрывают события после этой даты (например,
docker-compose изменения от 20 августа) — то есть бэкапы делаются вручную
перед конкретными миграциями, а не по расписанию.

## Миграции в проде

См. [DATABASE.md](DATABASE.md) — миграции применяются вручную через `psql`
поверх сырых `.sql`-файлов, README явно рекомендует после `git pull` сверять
`npx prisma migrate diff` перед тем как полагаться на новые поля/таблицы,
ссылаясь на реальный инцидент простоя из-за неприменённых миграций
(2026-07-03).
