# Stack — 1PDF

Полностью переписано 2026-09-08 (`TASK-0002`). Все значения подтверждены
прямым чтением файлов конфигурации в `/home/admin1/1pdf/`.

## Пакетный менеджер

npm (`package-lock.json` в корне и в `editor/`; иных lock-файлов не найдено).

## Root — `/home/admin1/1pdf/package.json` (backend, `"1pdf-admin"`, v1.0.0)

- **scripts**: `admin` (`tsx admin/server.ts`), `migrate` (`prisma migrate
  deploy`), `seed` (`tsx admin/seed.ts`), `generate` (`prisma generate`),
  `typecheck` (`tsc --noEmit`), `editor:build`/`editor:test` (прокси на
  `npm --prefix editor`), `verify:migrations`
  (`./scripts/assert-migrations-additive.sh`).
- **dependencies**: `@prisma/client ^6.19.3`, `axios ^1.6.8`, `bcrypt
  ^6.0.0`, `cookie-parser ^1.4.7`, `cors ^2.8.5`, `express ^4.19.2`,
  `express-rate-limit ^7.2.0`, `helmet ^7.1.0`, `jsonwebtoken ^9.0.2`,
  `multer ^2.2.0`, `node-cron ^4.6.0`, `nodemailer ^9.0.3`, `pdf-lib
  ^1.17.1`, `pino ^10.3.1`/`pino-pretty ^13.1.3`, `uuid ^11.1.1`.
- **devDependencies**: `prisma ^6.19.3`, `tsx ^4.16.2`, `typescript
  ^5.4.5`, соответствующие `@types/*`.
- `engines`: не найдено (версия Node подтверждена только через
  `Dockerfile.admin.v2`).

## `editor/package.json` (frontend, `"1pdf-editor"`, v2.0.0)

- **scripts**: `dev` (`vite`), `build` (`tsc -b && vite build`), `preview`,
  `test`/`test:watch` (vitest), `typecheck` (`tsc -b --noEmit`).
- **dependencies**: `@pdf-lib/fontkit ^1.1.1`, `i18next ^25.6.0` +
  `i18next-browser-languagedetector ^8.2.0` + `react-i18next ^16.5.0`,
  `konva ^10.3.0` + `react-konva ^19.2.4`, `pdf-lib ^1.17.1`, `pdfjs-dist
  ^6.0.227`, `react ^19.2.7`/`react-dom ^19.2.7`, `zustand ^5.0.14`.
- **devDependencies**: `@tailwindcss/vite ^4.3.0`, `tailwindcss ^4.3.0`,
  `vite ^8.0.16`, `vitest ^4.1.8`, `@testing-library/react ^16.3.2` +
  `jest-dom ^6.9.1`, `jsdom ^29.1.1`, `typescript ^6.0.3` (заметно новее,
  чем `^5.4.5` в backend — два независимых TypeScript-таргета в одном
  репозитории).
- Метаданные пакета (`author: "Chinmay Shringi"`, `license: "MIT"`,
  `description`) подтверждают происхождение от open-source freePDF.

## Node.js

`.nvmrc`/`engines` не найдены. Подтверждено через `Dockerfile.admin.v2:1,8`
— обе стадии (`editor-builder`, финальная) используют `FROM
node:22-alpine` — **Node.js 22** (новее, чем у других проектов my_office,
использующих Node 20).

## TypeScript

Два независимых `tsconfig.json` с разными целевыми версиями TS (backend
`^5.4.5`, editor `^6.0.3`) — не проверялось построчно на предмет различий в
опциях компиляции.

## Vite / Tailwind (`editor/vite.config.ts`, `@tailwindcss/vite`)

Vite 8 (мажорная версия новее типичной на момент аудита), Tailwind CSS 4
через официальный Vite-плагин (`@tailwindcss/vite`), не через PostCSS-конфиг.

## Переменные окружения

Полный перечень с указанием использования — [CONFIGURATION.md](CONFIGURATION.md).

## `.gitignore` / `.dockerignore`

`.dockerignore` (корень): `node_modules`, `admin/node_modules`,
`temp_uploads`, `db_oo`, `data_oo`, `logs_oo`, `lib_oo`, `postgres_data`,
`.git`, `.env`, `editor/node_modules`, `editor/dist`.
