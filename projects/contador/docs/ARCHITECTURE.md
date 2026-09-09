# Architecture — Contador

Полностью переписано 2026-09-08 (`TASK-0002`, независимый аудит по коду).

## Три приложения на одном PostgreSQL-инстансе + публичный marketing (без БД)

```
PostgreSQL 16 (контейнер contador-db, postgres:16-alpine)
├── contador       ← схема prisma/schema.prisma (корень)     — legacy v1 + admin
└── contador_v2    ← схема v2/prisma/schema.prisma           — основное приложение v2

contador-v2 (Next.js, :3032, basePath /v2)   ─┐  отдельные PrismaClient,
contador-admin (Express, :3031, /admin/api)  ─┘  admin генерирует ОБА клиента

contador-marketing (Next.js, :3033, без basePath) — публичные SEO-лендинги
RU/UZ (TASK-0003, добавлено 2026-09-09). Без Prisma, без сессий, без БД —
единственный сетевой вызов: GET admin:3031/admin/api/payment-info за ценой
тарифа PRO (тот же admin, что и v2, по имени сервиса в contador-net).
```

`admin/server.ts:1-34` явно создаёт два клиента: `prisma` (корневая схема,
`DATABASE_URL`) и `prismaV2` (клиент из `../v2/node_modules/.prisma/client`,
`V2_DATABASE_URL`) — то есть admin-приложение имеет прямой доступ к обеим
базам одновременно, v2 — только к своей.

## Структура каталогов и назначение

```
/home/admin1/contador/
├── admin/
│   ├── server.ts          # 1613 строк, ОДИН файл: и публичные вебхуки, и защищённый /admin/api
│   └── public/index.html  # 2391 строка — вся SPA-панель админа одним файлом (Tailwind CDN + unpkg, без сборки)
├── ai/                    # МЁРТВЫЙ КОД — не импортируется нигде (см. GAPS.md)
├── nsbu_and_soliq_codex/   # 29 .md — нормативные акты (не читались по требованию задачи)
├── backups/                # ручные .dump/.sql перед миграциями
├── prisma/                 # схема + 1 миграция legacy/admin БД (contador)
├── docker-compose.yml       # db + admin + v2 (v1 НЕ описан)
├── Dockerfile.admin, Dockerfile.v2
├── test-payme.sh, test-payments.sh   # ручные shell-скрипты для локальной проверки вебхуков
└── v2/
    ├── prisma/              # схема + 9 миграций (contador_v2)
    ├── src/
    │   ├── app/
    │   │   ├── api/          # 78 файлов route.ts — см. API.md
    │   │   ├── (страницы)    # dashboard/, transactions/, closing/, reports/, settings/, ...
    │   │   └── layout.tsx
    │   ├── components/       # Sidebar, SettingsSidebar, ReportFilters и т.д.
    │   ├── lib/
    │   │   ├── posting/       # postingEngine.ts, expressionEval.ts, templateValidator.ts
    │   │   ├── classification/ # rulesEngine.ts, aiClassifier.ts
    │   │   ├── reports/        # osv.ts, journal.ts, accountCard.ts, accountAnalysis.ts, subconto.ts
    │   │   ├── taxReport/      # constants.ts (строки приложений), engine.ts (генератор отчёта)
    │   │   ├── parsers/        # parser1c.ts, parserBankExcel.ts, parserSoliq.ts
    │   │   ├── closing.ts, ensureBaseData.ts (2901 строка), seed-coa.ts,
    │   │   │   constants.ts, context.ts, auth.ts/auth-edge.ts, prisma.ts,
    │   │   │   openItems.ts, charterCapital.ts, receiptKind.ts, billing.ts,
    │   │   │   queue.ts, mailer.ts, activityCategories.ts, format.ts
    │   │   └── proxy.ts        # роль middleware.ts в этой версии Next.js
    │   ├── instrumentation.ts  # register() → ensureBaseData() при каждом старте Node-рантайма
    │   └── __tests__/          # 35 файлов vitest, 5483 строки
    ├── audit_output.txt, audit_results.txt, audit_script.ts,
    │   doc_count.txt, test-azure-ai.mjs   # НЕИСПОЛЬЗУЕМЫЕ scratch-файлы в корне v2 (см. GAPS.md)
    └── scripts/generate-document-types-doc.ts
```

## Поток запроса (v2, типичный API-маршрут)

```
Браузер
  │  fetch("/v2/api/...", { credentials: cookie v2_session })
  ▼
proxy.ts (роль middleware.ts в Next 16; matcher — всё, кроме /_next/static, /_next/image, favicon.ico)
  │  публичные пути (login/register/forgot-reset-password + их API) пропускаются без проверки;
  │  остальное — verifySession(cookie) → редирект на /v2/login при отсутствии/невалидности JWT
  ▼
app/api/<path>/route.ts  (обработчик конкретного HTTP-метода)
  │  getSession()/getActiveOrgId()/getActiveMembership()  (lib/context.ts) — 401/403 при отказе
  │  бизнес-валидация тела запроса вручную (без единой схемы/Zod-миддлвари, хотя zod есть в зависимостях)
  ▼
lib/*.ts (business logic: postingEngine, closing, rulesEngine, aiClassifier, reports/*)
  │  Prisma Client (v2/prisma/schema.prisma) — по умолчанию БЕЗ автоскоупинга по orgId,
  │  каждый Prisma-вызов вручную содержит { where: { orgId, ... } };
  │  для 10 моделей (см. DATABASE.md) есть prismaWithOrg(orgId) — extension с автоскоупингом,
  │  но используется лишь в ~6 из 78 маршрутов
  ▼
PostgreSQL (contador_v2)
  ▲
  └─ NextResponse.json(...) — единый паттерн catch: err.message==="UNAUTHORIZED"→401, иначе 500
```

Поток запроса для `admin` проще: `express.json()` (с сохранением `rawBody` для
HMAC-проверки Alifpay) → `express.Router()` (`publicRouter`, без пароля,
только `express-rate-limit`) для вебхуков, либо `router` за
`adminAuth` (сверка заголовка `x-admin-password` с `ADMIN_PASSWORD`, простой
общий секрет, без ролей/токенов) для панели.

## Конвейер обработки банковской операции (сквозной, по коду)

```
POST /v2/api/import/bank (файл выписки)
        │  parser1c.ts / parserBankExcel.ts → StagedTransaction(status=IMPORTED)
        ▼
POST /v2/api/classification/run-rules  (детерминированные Rule, приоритет INN→KEYWORD→AMOUNT_RANGE→TREASURY_ACCOUNT)
        │  не найдено совпадение
        ▼
POST /v2/api/classification/run-ai  (только PRO-тариф; Azure OpenAI, батчи по 100, раздельно CREDIT/DEBIT)
        │  confidence < порога / неизвестный код / направление не совпало → NEEDS_CLARIFICATION
        ▼
GET /v2/api/clarification/queue → POST /v2/api/clarification/answer (пользователь; может создать Rule)
        │
        ▼
Document (status=POSTED) ── postDocument() ──▶ JournalEntry[] (двойная запись)
        │                                            │
        │                                            ▼
        │                              OpenItem (если template.opensItem) — авансы/подотчёт с riskDeadline
        ▼
Отчёты: pnl/, reports/balance (Форма №1/№2), reports/osv, journal, account-card, subconto,
        taxReport/engine.ts (квартальный расчёт налога на прибыль)
```

## Клиентские состояния v2 (карта страница → API)

v2 — не EJS/SSR-шаблоны, а Next.js App Router: каждая директория `app/<route>/`
содержит `page.tsx` (в основном тонкая обёртка) и клиентский компонент
`*Client.tsx` (`"use client"`) с `fetch()`-вызовами к `/v2/api/...`. Полная
карта запросов каждого клиентского компонента — в [VIEWS.md](VIEWS.md)
(построена через `grep -R "fetch("` по всем 63 `.tsx`-файлам, не построчным
чтением каждого).

## Аутентификация — архитектурная точка зрения

Stateless JWT (`jose`, HS256) в httpOnly cookie `v2_session`
(`v2/src/lib/auth-edge.ts:3`). Next.js 16 переименовал файл-конвенцию
`middleware.ts` в `proxy.ts` (подтверждено наличием `v2/src/proxy.ts` с той же
сигнатурой `export async function proxy(req)` + `export const config = {
matcher }` и отсутствием файла `middleware.ts` где-либо в `v2/src`) — proxy.ts
проверяет только присутствие и валидность JWT для нестатических путей и
делает редирект на `/v2/login`; проверка `orgId`/роли — всегда внутри
конкретного route-хендлера через `context.ts`. Подробности и оговорки о
неполной проверке мультитенантности по всем 78 маршрутам — в
[API.md](API.md) и [GAPS.md](GAPS.md).

## Известные архитектурные ограничения (заявлены в коде проекта как
## осознанные решения, не результат стороннего анализа)

- Проводки не поддерживают забалансовые счета (001/002/007) — комментарии в
  коде явно называют это нереализованным намеренно.
- Только perpetual-учёт ТМЗ через счёт 2910 — счета 9140/9150 (периодический
  метод) не задействованы ни одним типом документа
  (`v2/src/app/api/pnl/route.ts:58-63`).
- Проводки одновалютные — `CURRENCY_EXCHANGE` как код операции существует в
  комментариях constants.ts, но явно помечен «намеренно не включён».
