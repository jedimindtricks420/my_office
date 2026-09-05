# STACK — POSA

Последнее обновление: 2026-09-05 (TASK-0001). Источник — `package.json`,
`Dockerfile`, `docker-compose.yml` в `/home/admin1/posa/activation-system/`.

## Backend

- **Runtime:** Node.js — образ `node:20` в Dockerfile; фактическая версия на
  хосте на момент проверки: `v20.20.0`. `package.json` использует ESM
  (`"type": "module"`).
- **Framework:** Express `^5.1.0`.
- **ORM:** Prisma `^6.5.0` + `@prisma/client` `^6.5.0`, PostgreSQL 16
  (`postgres:16` образ в `docker-compose.yml`).
- **Сессии:** `express-session` + `connect-pg-simple` (хранение в Postgres).
- **Шаблонизатор:** `ejs` + `express-ejs-layouts`.
- **Стили:** `tailwindcss` (сборка через `npm run build:wallet:css`) +
  Bootstrap упоминается в проектной документации/копилот-инструкциях (не
  проверено наличие в `package.json` — там явно Tailwind; вывод о Bootstrap
  не подтверждён кодом, см. OPEN_QUESTIONS.md).
- **Аутентификация/безопасность:** `bcrypt` (хэш паролей), `express-rate-limit`
  (лимитеры на логин/регистрацию/OTP), `cookie-parser`, `cors`, `joi`
  (валидация).
- **Файлы/загрузки:** `multer`.
- **QR/штрихкоды:** `qrcode`, `bwip-js`, `@zxing/browser` + `@zxing/library`
  (сканирование на клиенте).
- **PDF:** `pdfkit` (генерация чеков).
- **Apple Wallet:** `passkit-generator` (генерация `.pkpass`), `pngjs`.
- **Email:** `nodemailer`.
- **HTTP-клиент:** `axios` (интеграции с внешними API).
- **Прочее:** `uuid`, `dotenv`, `postcss`/`autoprefixer` (сборка CSS).
- **Dev:** `nodemon` (только devDependency).

## Отсутствует / не подтверждено

- Нет фреймворка автоматического тестирования (jest/mocha/vitest/supertest —
  не найдены ни в `package.json`, ни в `package-lock.json`). Нет npm-скрипта
  `test`.
- Нет TypeScript — код на чистом JS (ESM).
- Нет линтера/форматтера, настроенного в `package.json` (ни eslint, ни
  prettier не значатся в зависимостях).
- В `package.json` есть скрипты `bmad:*` (`bmad-method install/list/validate`)
  — назначение и статус использования не выяснялись в рамках этой задачи
  (см. OPEN_QUESTIONS.md).

## Инфраструктура

- **Контейнеризация:** Docker + Docker Compose (`docker-compose.yml` +
  `docker-compose.override.yml`), сервисы: `backend` (само приложение),
  `studio` (Prisma Studio на 5555→127.0.0.1:5557), `db` (Postgres 16).
- **Веб-сервер/reverse proxy:** Nginx на хосте, конфиги в
  `/etc/nginx/sites-available/` для доменов `namo.uz`, `wallet.namo.uz`,
  `merchant.namo.uz`, `vendor.namo.uz`, `office.namo.uz`, `api.namo.uz`,
  `a.namo.uz`, `ppdev.namo.uz`. TLS — Let's Encrypt/Certbot.
- Подробности деплоя и текущего состояния контейнеров — см.
  [DEPLOYMENT.md](DEPLOYMENT.md).
