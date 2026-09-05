# STACK — MIBT

Последнее обновление: 2026-09-05. Источник: `package.json`, `package-lock.json`
(`lockfileVersion: 3`, npm), `Dockerfile`, `docker-compose.yml`.

## Рантайм

- **Node.js 20** (`node:20-alpine` в `Dockerfile`; хост, где читался код,
  имеет `node --version` → `v20.20.0`).
- Модульная система — ESM (`"type": "module"` в `package.json`).
- Пакетный менеджер — npm (`package-lock.json`, `npm ci` в `Dockerfile`).
- Имя пакета: `mibt`, версия: `6.0.0`.

## Основные зависимости (`dependencies`, из `package.json`)

| Пакет | Версия | Роль |
|---|---|---|
| `express` | ^5.2.1 | HTTP-фреймворк (Express 5) |
| `ejs` | ^6.0.1 | Шаблонизатор SSR |
| `express-ejs-layouts` | ^2.5.1 | Layout-обёртки для EJS |
| `@prisma/client` / `prisma` | ^6.19.3 | ORM + CLI (Prisma 6) |
| `pg` | ^8.21.0 | PostgreSQL-драйвер |
| `express-session` | ^1.19.0 | Сессии |
| `connect-pg-simple` | ^10.0.0 | Хранилище сессий в PostgreSQL |
| `bcrypt` | ^6.0.0 | Хэширование паролей (нативный биндинг) |
| `cookie-parser` | ^1.4.7 | Парсинг cookie |
| `cors` | ^2.8.6 | CORS |
| `helmet` | ^8.2.0 | Security-заголовки, CSP |
| `express-rate-limit` | ^8.5.2 | Rate limiting |
| `joi` | ^18.2.1 | Валидация входных данных |
| `multer` | ^2.1.1 | Загрузка файлов |
| `pdfkit` | ^0.19.0 | Генерация PDF |
| `nodemailer` | ^8.0.10 | Отправка email |
| `fast-xml-parser` | ^5.8.0 | Парсинг XML курса ЦБ РУз |
| `dotenv` | ^17.4.2 | Загрузка `.env` |

## Dev-зависимости (`devDependencies`)

| Пакет | Версия | Роль |
|---|---|---|
| `tailwindcss` | 3.4 | CSS-фреймворк (сборка в `public/css/app.css`) |
| `autoprefixer`, `postcss` | ^10.5.0 / ^8.5.15 | Обработка CSS |
| `nodemon` | ^3.1.14 | Автоперезапуск в dev (`npm run dev`) |

Нет: TypeScript, React/Vue/любого фронтенд-фреймворка, ORM кроме Prisma,
GraphQL, тестовых фреймворков (см. ниже).

## Тестирование

**Автотестов нет.** В зависимостях отсутствуют `jest`, `mocha`, `vitest`,
`supertest` и аналоги; в дереве проекта не найдено файлов `*.test.js` или
`*.spec.js`; в `package.json` нет скрипта `test`.

## npm-скрипты (`package.json`, полностью)

```
start      → node index.js
dev        → nodemon index.js
build:css  → tailwindcss -i resources/css/app.css -o public/css/app.css --minify
watch:css  → tailwindcss -i resources/css/app.css -o public/css/app.css --watch
```

Скрипт `seed` для Prisma задан отдельно: `prisma.seed` → `node prisma/seed.js`.

## Инфраструктура

- **PostgreSQL 16** (`postgres:16-alpine`) — основная БД.
- **Docker + Docker Compose** — оркестрация 4 сервисов (см.
  [DEPLOYMENT.md](DEPLOYMENT.md)).
- **nginx** — дважды: system nginx на хосте (терминирует TLS для
  `portal.mibt.uz`) и отдельный `nginx:alpine` внутри Compose
  (`mibt2-nginx`, порт `127.0.0.1:3011`, self-signed TLS).
- **prodrigestivill/postgres-backup-local** — автоматический бэкап БД
  (ежедневно/еженедельно/ежемесячно).
- **Let's Encrypt / certbot** — TLS-сертификат для `portal.mibt.uz`
  (подтверждено: `certbot certificates`, годен до 2026-11-10).

## CI/CD

Не найдено — нет каталога `.github/workflows/`, нет других конфигов CI.

## Версионирование кода

`/home/admin1/mibt-2/` **не является git-репозиторием** (`.git` отсутствует).
История изменений не отслеживается; единственные источники хронологии —
mtime файлов и отдельные changelog-документы в `docs/` внутри самого проекта.
