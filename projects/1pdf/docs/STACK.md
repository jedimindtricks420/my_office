# Stack — 1PDF

Снимок на 2026-09-05, источник — `package.json` (корень и `editor/`),
`Dockerfile.admin.v2`, прямая проверка хоста.

## Backend (`admin/`, корневой `package.json`, name: `1pdf-admin`)

| Область | Реализация |
|---|---|
| Рантайм | Node.js 22 (`node:22-alpine` в `Dockerfile.admin.v2`); хостовой `node -v` вне контейнера показывает 20.20.0 — используется только для инструментов, не для рантайма контейнера |
| Веб-фреймворк | Express `^4.19.2`, TypeScript `^5.4.5`, исполняется через `tsx` (без отдельной сборки — `CMD npx tsx admin/server.ts`) |
| ORM | Prisma `^6.19.3` / `@prisma/client` `^6.19.3` |
| Аутентификация | `jsonwebtoken` `^9.0.2`, `bcrypt` `^6.0.0`, `cookie-parser` |
| Загрузка файлов | `multer` `^2.2.0` |
| Почта | `nodemailer` `^9.0.3` (OTP-письма) |
| HTTP-клиент | `axios` `^1.6.8` (Alifpay API, проксирование ONLYOFFICE-колбэка) |
| Безопасность/инфра | `helmet` `^7.1.0`, `cors`, `express-rate-limit` `^7.2.0` |
| Логирование | `pino` `^10.3.1` + `pino-pretty` |
| Планировщик | `node-cron` `^4.6.0` (почасовая очистка `temp_uploads/`) |
| PDF (сервер) | `pdf-lib` `^1.17.1` (валидация/создание пустого PDF) |

Пакетный менеджер — npm (`package-lock.json` в корне и в `editor/`).

## Editor (`editor/package.json`, name: `1pdf-editor`, version 2.0.0)

| Область | Реализация |
|---|---|
| UI | React `^19.2.7` + React DOM |
| Сборка | Vite `^8.0.16`, TypeScript `^6.0.3` (`tsc -b && vite build`) |
| PDF-рендеринг | `pdfjs-dist` `^6.0.227` |
| PDF-манипуляция | `pdf-lib` `^1.17.1`, `@pdf-lib/fontkit` |
| Canvas/рисование | `konva` `^10.3.0`, `react-konva` |
| Состояние | `zustand` `^5.0.14` |
| Интернационализация | `i18next`, `react-i18next`, `i18next-browser-languagedetector` (локали `ru`/`uz`/`en`, `editor/src/i18n/locales/`) |
| Стили | `tailwindcss` `^4.3.0` (через `@tailwindcss/vite`) |
| Тесты | `vitest` `^4.1.8`, `@testing-library/react`, `jsdom` |

Editor основан на открытом проекте freePDF (Chinmay Shringi, MIT) —
`package.json` сохраняет исходные `license: MIT` и `author: Chinmay Shringi`;
адаптация и весь бэкенд/интеграции — собственность 1PDF (подробности
происхождения — [OVERVIEW.md](OVERVIEW.md)).

## Инфраструктура

| Область | Реализация |
|---|---|
| База данных | PostgreSQL 16 (`postgres:16-alpine`) |
| Прокси (в Docker) | Nginx (`nginx:alpine`) |
| Прокси (хост, TLS) | Host-nginx + Certbot, вне репозитория — см. [DEPLOYMENT.md](DEPLOYMENT.md) |
| Оркестрация | Docker Compose (один `docker-compose.yml`, 3 сервиса: `nginx-proxy`, `postgres`, `admin`) |
| Сборка образа | Многостадийный `Dockerfile.admin.v2`: стадия `editor-builder` (собирает `editor/dist`) → стадия рантайма (копирует `admin/`, `prisma/`, собранный `editor/dist`) |

## Тесты и проверки (подтверждено `package.json`/README)

- `npm run typecheck` — `tsc --noEmit` для серверного кода.
- `npm --prefix editor run typecheck` / `test` / `build` — для редактора.
- `npm run verify:migrations` → `scripts/assert-migrations-additive.sh`
  (проверяет только одну конкретную миграцию по имени файла, не универсальна
  — см. [ARCHITECTURE.md](ARCHITECTURE.md)).
- `npm audit --omit=dev --audit-level=high` — упомянут в README как
  обязательная проверка перед релизом; не запускался в рамках этой задачи
  (задача документационная, изменение состояния `node_modules`/потенциальный
  сетевой вызов не выполнялись).
- Тестовые файлы редактора (vitest), подтверждено найдены: `App.test.tsx`,
  `i18n/catalog.test.ts`, `lib/pdf/coordinateTransform.test.ts`,
  `lib/pdf/exportPdf.test.ts`, `lib/pdf/pageOperations.test.ts`,
  `lib/pdf/textRuns.test.ts`, `store/editorStore.test.ts` — 7 файлов.
  Серверная сторона (`admin/`) собственных тестовых файлов не имеет
  (не найдено `*.test.ts`/`*.spec.ts` вне `editor/`).
