# Deployment / Runbook — Contador

Полностью переписано 2026-09-08 (`TASK-0002`). Источники: `docker-compose.yml`,
оба `Dockerfile`, `package.json` (корень и `v2/`), и одна read-only команда
`docker ps` для сверки реального состояния хоста (никакие контейнеры не
запускались/останавливались/пересобирались в рамках этой задачи — см.
правило проекта в памяти агента: не поднимать dev-серверы/docker на этом
хосте).

## Установка и локальный запуск (выведено из `package.json`/Dockerfile — сам процесс не выполнялся)

Требования: Node.js **20**, PostgreSQL **16**, npm.

### admin (порт 3031)

```bash
cd /home/admin1/contador
npm install
npx prisma generate                                    # схема prisma/schema.prisma → contador
npx prisma generate --schema ../v2/prisma/schema.prisma # доп. клиент → contador_v2 (тот же процесс читает обе БД)
# ⚠️ прямого скрипта npm start НЕТ — package.json не содержит секции "scripts"
npx tsx admin/server.ts
```

Обязательные переменные окружения: `DATABASE_URL`, `V2_DATABASE_URL`,
`ADMIN_PASSWORD` (см. [CONFIGURATION.md](CONFIGURATION.md) — оба явно
проверяются кодом при старте и роняют процесс при отсутствии).

### v2 (порт 3032, basePath `/v2`)

```bash
cd /home/admin1/contador/v2
npm install
npm run db:generate     # prisma generate
npm run db:push         # применяет ТЕКУЩУЮ схему целиком (создание БД с нуля) —
                         #   НЕ то же самое, что применение 9 инкрементальных .sql-патчей,
                         #   см. ниже «Миграции»
npm run dev              # next dev -p 3032   (или: npm run build && npm run start)
```

При каждом старте (`dev`/`start`/production-рантайм) автоматически
выполняется `instrumentation.ts` → `ensureBaseData()` — сидинг 269 счетов
плана НСБУ и 180 типов документов происходит **сам**, отдельного шага
«накатить сид-данные» не требуется и не существует.

Обязательная переменная: `JWT_SECRET` (роняет процесс при отсутствии, см.
CONFIGURATION.md). `DATABASE_URL` — Prisma упадёт при первом обращении к БД
без неё.

### Тесты

```bash
cd v2 && npm run test          # vitest run, 35 файлов, 5483 строк — часть тестов
                                # интеграционные и пишут в реальную БД contador_v2
                                # (подтверждено импортом ensureBaseData() внутри
                                # нескольких *.test.ts — см. GAPS.md, методологическая оговорка)
npm run test:coverage
npm run docs:types             # генерирует docs/DOCUMENT_TYPES.md из baseDocumentTypes,
                                # проверяется отдельным тестом document-types-doc.test.ts
```

## Миграции — порядок применения (реальный, не `prisma migrate deploy`)

Ни admin, ни v2 **не вызывают** `prisma migrate deploy` ни в одном
Dockerfile, ни в одном `package.json`-скрипте. Оба каталога
`*/migrations/` — плоские наборы `.sql`-файлов без стандартной структуры
Prisma (подробности — [DATABASE.md](DATABASE.md)). Наблюдаемый на практике
процесс (по хронологии имён файлов и содержимому README-примечаний в самих
`.sql`, не читанных как источник фактов, а только как метаданные файла):

1. Изменить `schema.prisma`.
2. Написать соответствующий `.sql`-патч вручную, положить в
   `<app>/prisma/migrations/<YYYYMMDD>_<name>.sql`.
3. Сделать резервную копию БД перед применением (см. `backups/` — паттерн
   `contador_v2_pre_<изменение>_<таймштамп>.dump`).
4. Применить `.sql`-файл вручную через `psql` к работающей БД.
5. Убедиться, что `schema.prisma` и реальная структура БД синхронны (нет
   автоматической проверки в CI — не найдено файлов CI/CD в репозитории).

## Docker Compose — оркестрация продакшна

`docker-compose.yml` (корень) — 4 сервиса, сеть `contador-net` (bridge):

| Сервис | Образ | Порт | depends_on |
|---|---|---|---|
| `db` | `postgres:16-alpine` | не публикуется (`expose: 5432`) | — |
| `admin` | `Dockerfile.admin` | `3031:3031` | `db` (healthy) |
| `v2` | `Dockerfile.v2` | `3032:3032` | **нет** `depends_on` на `db` |
| `marketing` | `Dockerfile.marketing` | `3033:3033` | `admin` |

`v1` не описан в compose-файле вовсе.

**`marketing`** добавлен 2026-09-09 (`TASK-0003`) — публичные SEO-лендинги
RU/UZ, отдельное Next.js-приложение (`marketing/`, без Prisma/сессий/БД).
Единственная сетевая зависимость — цена тарифа PRO на `/ru/tarify/` и
`/uz/tariflar/`, читается с `http://admin:3031/admin/api/payment-info` по
имени сервиса в `contador-net` (`ADMIN_API_URL` — та же переменная и тот же
паттерн, что и в `v2/.env`; **не** `localhost`, внутри контейнера это сам
контейнер). Nginx (`/etc/nginx/sites-available/contador.uz`) проксирует
`/ru/`, `/uz/`, `/_next/`, `/og/`, `= /sitemap.xml`, `= /robots.txt` на
`localhost:3033`; корневой `location /` (статическая `/home/admin1/landing/`)
и `/v2`, `/admin` не менялись.

```bash
docker compose up -d --build
```

(команда выведена из структуры файла, не выполнялась в рамках задачи)

`Dockerfile.v2` — многостадийная сборка (`deps` → `builder` → `runner`),
`NODE_OPTIONS="--max-old-space-size=3072"` и `next.config.ts`'s
`experimental.cpus: 1` — оба ограничения явно прокомментированы в коде как
последствие инцидента **19.08.2026**, когда неограниченная сборка исчерпала
всю память хоста без свопа и уронила VM (`Dockerfile.v2`, комментарий перед
`ENV NODE_OPTIONS`). `docker-compose.yml`'s `mem_limit: 1g` /
`mem_reservation: 256m` на сервисе `v2` — та же защита для рантайма.

## Реальное состояние хоста (перепроверено read-only `docker ps`, 2026-09-08 — независимо от TASK-0001)

```
contador-v2                 Up 2 недели   0.0.0.0:3032->3032
contador-admin               Up 2 недели   0.0.0.0:3031->3031
contador-app                 Up 2 недели   (без публикации порта)
f7ee8a3d81b8_contador-app    Up 2 недели   0.0.0.0:3030->3030
contador-db                  Up 2 недели (healthy)  (без публикации порта)
```

Полностью совпадает с состоянием, зафиксированным в TASK-0001
(2026-09-05) — независимая повторная проверка подтверждает, а не просто
наследует прежний вывод: **два v1-контейнера всё ещё запущены**, несмотря на
физическое отсутствие `src/` в текущем дереве git и на то, что
`docker-compose.yml` их не описывает. Оба образа собраны из более ранних
снимков кода (2026-05-28 и 2026-06-12 по `CREATED AT`) — то есть до
удаления `src/` из репозитория, либо из Dockerfile, которого сейчас в
репозитории нет. Не выясняется из кода репозитория, обслуживают ли эти два
контейнера реальный трафик — вопрос эксплуатации, а не источников
(см. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).

## Резервные копии (`backups/`)

13 файлов, паттерн: дамп БД перед конкретным изменением
(`contador_v2_pre_<изменение>_<таймштамп>.dump` + сопутствующий `.sql`).
Самый свежий — `contador_v2_pre_voucher_series_20260731_105644.dump`
(31 июля) — то есть бэкапы делаются вручную перед конкретными миграциями,
не по расписанию; отдельного cron/systemd-таймера для регулярных бэкапов в
репозитории не найдено (что не исключает его существования вне репозитория,
на уровне инфраструктуры хоста — не проверялось).
