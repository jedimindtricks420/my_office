# Stack — Contador

Снимок на 2026-09-05. Все значения подтверждены прямым чтением файлов конфигурации
в `/home/admin1/contador/`.

## Пакетный менеджер

npm (только `package-lock.json` в корне и в `v2/`; yarn/pnpm/bun lock-файлы не
найдены нигде в репозитории).

## Root — `/home/admin1/contador/package.json` (это же — проект admin-приложения)

- name: `contador-admin`, version: `1.0.0`
- **scripts**: не найдено (секции `scripts` в файле нет).
- **dependencies**: `@prisma/client ^6.2.1`, `bcryptjs ^3.0.3`, `express ^5.2.1`,
  `express-rate-limit ^8.3.2`, `helmet ^8.1.0`, `prisma ^6.2.1`
- **devDependencies**: `@types/bcryptjs ^2.4.6`, `@types/express ^5.0.6`,
  `@types/node ^20`, `tsx ^4.21.0`, `typescript ^5`
- `engines`: не найдено
- Каталог `admin/` не содержит собственного `package.json` — зависимости и
  корневой `prisma/schema.prisma` (легаси/admin схема) принадлежат этому файлу.

## v2 — `/home/admin1/contador/v2/package.json` (основное приложение)

- name: `contador-v2`, version: `0.1.0`
- **scripts**:
  - `dev`: `next dev -p 3032`
  - `build`: `next build`
  - `start`: `next start -p 3032`
  - `lint`: `eslint`
  - `db:generate`: `prisma generate`
  - `db:push`: `prisma db push`
  - `db:migrate`: `prisma migrate dev` (по имеющимся данным — не реальный рабочий
    процесс проекта, см. [DATABASE.md](DATABASE.md))
  - `db:studio`: `prisma studio`
  - `test`: `vitest run`
  - `test:watch`: `vitest`
  - `test:coverage`: `vitest run --coverage`
  - `docs:types`: `tsx scripts/generate-document-types-doc.ts`
- **dependencies** (ключевые): `next 16.2.1`, `react 19.2.4`, `react-dom 19.2.4`,
  `@prisma/client ^6.2.1`, `prisma ^6.2.1`, `@tanstack/react-query ^5.95.2`,
  `bcryptjs ^3.0.3`, `decimal.js ^10.6.0`, `jose ^6.2.2`, `lucide-react ^1.7.0`,
  `nodemailer ^8.0.11`, `openai ^6.33.0`, `recharts ^3.8.1`, `xlsx ^0.18.5`,
  `zod ^4.3.6`
- **devDependencies** (ключевые): `@tailwindcss/postcss ^4`, `tailwindcss ^4`,
  `eslint ^9`, `eslint-config-next 16.2.1`, `typescript ^5`, `vitest ^4.1.9`,
  `@vitest/coverage-v8 ^4.1.9`
- `engines`: не найдено

## TypeScript

Оба `tsconfig.json` (корень и `v2/`) практически идентичны: `target: ES2017`,
`strict: true`, `module: esnext`, `moduleResolution: bundler`,
`jsx: react-jsx`, path-alias `@/* → ./src/*`. Корневой дополнительно исключает
`node_modules`, `admin`, `ai`; `v2/tsconfig.json` исключает только `node_modules`.

## Next.js — конфигурация (`v2/next.config.ts`)

- `output: "standalone"`
- `basePath: "/v2"`, `assetPrefix: "/v2"`
- `experimental.cpus: 1` — намеренно ограничивает сборку одним воркером
  (комментарий в коде связывает это с инцидентом нехватки памяти на хосте
  2026-08-19, см. [DEPLOYMENT.md](DEPLOYMENT.md))
- Глобальные security-заголовки (`X-Frame-Options: DENY`,
  `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`,
  `X-XSS-Protection: 1; mode=block`) через `headers()`.

## ESLint

`v2/eslint.config.js` существует и нетривиален (flat config, `typescript-eslint`,
`eslint-plugin-react`, `eslint-plugin-react-hooks`, `@next/eslint-plugin-next`
core-web-vitals). README проекта утверждает, что до 2026-07-03 линт был полностью
нерабочим (ESLint 9 без конфига) — сам факт нерабочего состояния _до_ правки не
подтверждён напрямую (файл содержит только текущую рабочую конфигурацию, без даты).

## Node.js

`.nvmrc` и `engines` не найдены нигде в репозитории. Подтверждено через
Dockerfile: оба `Dockerfile.admin` и `Dockerfile.v2` (все стадии) используют
`FROM node:20-alpine` — **Node.js 20**.

## Переменные окружения (только имена, без значений)

Подтверждено чтением имён переменных (без чтения значений) в реальных `.env`:

**`v2/.env`**: `DATABASE_URL`, `JWT_SECRET`, `AZURE_OPENAI_ENDPOINT`,
`AZURE_OPENAI_API_KEY`, `PORT`, `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURE`,
`SMTP_USER`, `SMTP_PASS`, `SMTP_FROM`, `ADMIN_API_URL`, `OPENAI_MODEL`,
`NEXT_PUBLIC_APP_URL`.

⚠️ Это расходится с README проекта, который в разделе «Переменные окружения»
документирует для v2 переменную `OPENAI_API_KEY` — в реальном `v2/.env` и в коде
(`v2/src/lib/classification/aiClassifier.ts`) используются `AZURE_OPENAI_ENDPOINT`/
`AZURE_OPENAI_API_KEY` (клиент `openai` SDK указывает на Azure AI Foundry
через `baseURL`), переменной `OPENAI_API_KEY` в `v2/.env` нет.

**Корневой `.env`** (root/admin/v1): `DATABASE_URL`, `BASIC_AUTH_USER`,
`BASIC_AUTH_PASSWORD`, `PORT`, `OPENAI_API_KEY`, `PAYME_MERCHANT_ID`,
`PAYME_SECRET_KEY`, `CLICK_SERVICE_ID`, `CLICK_SECRET_KEY`, `ADMIN_PORT`,
`ADMIN_PASSWORD`, `V2_DATABASE_URL`, `JWT_SECRET`.

Примечание: по коду `admin/server.ts` реквизиты платёжных провайдеров (Payme/
Click/Alifpay merchant ID и секретные ключи) фактически хранятся не в
переменных окружения, а в таблице БД `PaymentConfig` (см.
[DATABASE.md](DATABASE.md)) — `PAYME_MERCHANT_ID`/`CLICK_SERVICE_ID`/
`CLICK_SECRET_KEY` в `.env` могут быть унаследованным/неиспользуемым набором,
это не проверялось построчно по каждому месту использования.

Файлов `.env.example`/`.env.sample`/`.env.template` в репозитории не найдено.

## `.gitignore` / `.dockerignore` (что исключено)

- Корневой `.gitignore`: `/node_modules`, `/coverage`, `/.next/`, `/out/`,
  `/build`, `.env*`, `.vercel`, `*.tsbuildinfo`, `next-env.d.ts`,
  `/src/generated/prisma`, `postgres_data/`.
- `v2/.gitignore`: `.env`, `.next/`, `node_modules/`.
- `.dockerignore`: `postgres_data`, `node_modules`, `.git`, `*.md`, `docs`, `ai`.
