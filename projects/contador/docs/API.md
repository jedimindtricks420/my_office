# API — Contador v2

Снимок на 2026-09-05. Касается только приложения `v2` (Next.js App Router,
`v2/src/app/api/`). API admin-приложения (Payme/Click/Alifpay + панель
управления) описан в разделе «Admin API» ниже.

## Инвентарь маршрутов v2

Подтверждено: **78 файлов `route.ts`** под `v2/src/app/api/` (README проекта
заявляет «60+ endpoint-ов» — верно, но занижает: 78 файлов, при этом часть
файлов экспортирует несколько HTTP-методов, так что реальное число обрабатываемых
HTTP-эндпоинтов ещё выше; точное число методов не подсчитывалось).

Группировка по top-level папке:

| Папка | Файлов | Назначение (по README/коду) |
|---|---|---|
| `auth/` | 5 | forgot-password, login, logout, register, reset-password |
| `user/` | 2 | active-org, me |
| `org/` | 3 | members, members/[userId], members/invite |
| `orgs/` | 2 | список организаций, `[orgId]` |
| `accounts/` | 2 | план счетов, видимость счёта |
| `bank-accounts/` | 3 | CRUD + статистика по счёту |
| `transactions/` | 4 | реестр, категория, skip, by-counterparty |
| `import/` | 3 | банковская выписка + rollback, Soliq |
| `classification/` | 4 | ai-reconcile, run-ai, run-rules, статус по периоду |
| `clarification/` | 2 | ответ на уточнение, очередь |
| `rules/` | 5 | CRUD, apply, preview, reorder |
| `periods/` | 4 | CRUD, close, reopen |
| `closing/` | 7 | accruals, finalize, pending-invoices, state, шаг мастера, year-end + статус |
| `posting/` | 3 | post, repost, void |
| `open-items/` | 3 | CRUD, close, reopen |
| `pnl/` | 1 | Форма №2 |
| `cashflow/` | 1 | отчёт о движении денежных средств |
| `reports/` | 10 | account-analysis, account-card, accounts, balance (Форма №1), counterparties, journal, osv, profit-tax (+line-details), subconto |
| `dashboard/` | 1 | KPI дашборда |
| `settings/` | 6 | charter-capital, costing-method, open-item-deadlines, opening-balance, org, tax-deadlines |
| `payments/` | 4 | history, initiate, subscription, voucher |
| `documents/` + `document-types/` | 2 | реестр документов и типов |
| `cbu-rate/` | 1 | курс ЦБ |

## Аутентификация и авторизация

**Механизм: stateless JWT в HTTP-only cookie**, без серверных сессий.

- Cookie: `v2_session` (константа `COOKIE_NAME` в `v2/src/lib/auth-edge.ts`).
- Подпись/проверка через библиотеку **`jose`** (`SignJWT`/`jwtVerify`, HS256) —
  `jsonwebtoken` в проекте **не используется**.
- `v2/src/lib/auth-edge.ts` — Edge-safe верификация (`verifySession(token)` →
  `{ userId, email, activeOrgId } | null`).
- `v2/src/lib/auth.ts` — `createSession()` (подпись с истечением через
  `SESSION_MAX_AGE_SECONDS`), `getSessionFromCookie()` (читает cookie через
  `next/headers`).
- `v2/src/lib/context.ts` — точка входа для всех маршрутов:
  `getSession()` (кидает `"UNAUTHORIZED"`), `getUser()`, `getActiveOrgId()`
  (берёт `activeOrgId` **прямо из payload JWT**, кидает `"NO_ACTIVE_ORG"` — не
  перепроверяет по БД на каждый вызов), `getActiveMembership()` (более строгий
  вариант: реальный `prisma.orgMember.findFirst`, кидает `"FORBIDDEN"` при
  отсутствии членства). Плюс хелперы ответов `unauthorized()`/`forbidden()`/
  `badRequest()`/`notFound()`.
- **Мультитенантность**: обеспечивается тем, что каждый Prisma-запрос
  фильтруется по `orgId` из JWT (подтверждено в `pnl/route.ts`,
  `classification/status/[periodId]/route.ts` — последний явно сверяет
  `job.orgId !== orgId` → 403). Не каждый маршрут вызывает более строгий
  `getActiveMembership()` — многие полагаются только на `getActiveOrgId()`,
  то есть на доверие подписанному JWT. **Не проверено построчно по всем 78
  маршрутам**, что фильтрация по `orgId` применяется везде — см.
  [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).
- **Не найден** файл `middleware.ts` в исходниках (`v2/src`, `v2/`) — только
  скомпилированные артефакты в `.next/server/`. Проверка авторизации, по
  имеющимся данным, реализована на уровне отдельных route-хендлеров
  (`context.ts`), а не через единый Next.js middleware.

## Типичный паттерн route-хендлера

Пример из `pnl/route.ts` (Форма №2):

```ts
const orgId = await getActiveOrgId();
// ...raw SQL через $queryRaw, фильтр d."orgId" = ${orgId} AND d.status = 'POSTED'
//    AND dt.code != 'PERIOD_CLOSING', арифметика через decimal.js
```

Единый паттерн обработки ошибок (встречается в нескольких проверенных файлах):

```ts
} catch (err: any) {
  if (err.message === "UNAUTHORIZED" || err.message === "NO_ACTIVE_ORG") {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }
  console.error("...", err);
  return NextResponse.json({ error: "Internal error" }, { status: 500 });
}
```

Отчётные константы (например `PNL_COVERED_TRANSIT_CODES`,
`LINE010_CONTRA_CODES`) экспортируются прямо из route-файлов и переиспользуются
тестами-стражами (`pnl-transit-completeness.test.ts` и др.) — единый источник
истины между кодом и тестами.

`classification/run-ai/route.ts` дополнительно гейтится по тарифу:
`getUserActivePro(session.userId)`, иначе 403 «доступна только на тарифе PRO».

## AI-классификация (интеграция)

Файл: `v2/src/lib/classification/aiClassifier.ts`.

- Библиотека: официальный npm-пакет `openai` (не сырой `fetch`).
- **Провайдер: Azure OpenAI (Azure AI Foundry)**, не публичный OpenAI API —
  клиент создаётся как `new OpenAI({ apiKey: azureApiKey, baseURL:
  azureEndpoint })`, переменные окружения `AZURE_OPENAI_ENDPOINT`/
  `AZURE_OPENAI_API_KEY` (см. [STACK.md](STACK.md) — расходится с README,
  где упомянут просто `OPENAI_API_KEY`).
- Модель по умолчанию `gpt-4o-mini` (`AI.MODEL` в `constants.ts`, переопределима
  через `OPENAI_MODEL`).
- **Размер батча: 100** (константа `AI.BATCH_SIZE`) — расходится с описанием в
  README проекта («батчи до 20 транзакций»). Транзакции сначала делятся по
  `direction` (CREDIT/DEBIT, у каждого свой промпт/схема), затем чанкуются по
  100.
- Порог уверенности по умолчанию 70 (`AI.CONFIDENCE_THRESHOLD`), переопределим
  через `Organization.aiConfidenceThreshold`; используется и в промпте (просьба
  к модели), и как жёсткий гейт в коде (`if (res.confidence >= threshold)` →
  автопостинг, иначе — очередь уточнений). Отдельное бизнес-правило: для
  классификации взносов в уставный капитал уверенность принудительно занижается
  ниже порога, если `remainingCharterDebt` не покрывает заявленную сумму.
- Вызов: `openai.chat.completions.create({ model, messages, response_format:
  <строгая JSON-схема>, max_completion_tokens: 16384 })`. При ошибке на чанке —
  все транзакции чанка помечаются `NEEDS_CLARIFICATION`, не роняя всю задачу.

## Admin API (`admin/server.ts`, порт 3031)

Единый файл (1613 строк) — публичный роутер (вебхуки платежей) + защищённый
роутер `/admin/api` (пользователи, организации, ваучеры, платежи, подписки,
аудит-лог, банковские счета, периоды, правила, контрагенты). Подтверждено ~62
определения маршрутов.

- **Payme**: `checkPaymeAuth()` (Basic-Auth против `config.payme_key`/
  `payme_test_key`, источник — таблица БД `PaymentConfig`, не env),
  `generatePaymeUrl()`, вебхук `POST /payments/payme` (JSON-RPC методы
  `CheckPerformTransaction`/`CreateTransaction`/`PerformTransaction`/
  `CancelTransaction`/`CheckTransaction`), есть V1- и V2-варианты
  (`/v2/payments/payme`).
- **Click**: `verifyClickSignature()` (конкатенация полей + секрет → сверка),
  `generateClickUrl()`, `POST /v2/payments/click/prepare` и
  `/v2/payments/click/complete`, секрет — `config.click_secret_key` из БД.
- **Alifpay** (третий провайдер, не упомянут в README как реализованный):
  `createAlifInvoice()`, `getAlifCheckoutUrl()`, `verifyAlifSignature()`
  (HMAC-SHA256 по телу запроса, заголовок `Signature`), вебхук
  `POST /v2/payments/alif/webhook`, секреты
  `config.alif_secret_key_production/sandbox` — из БД.
- Успешный платёж → `fulfillSubscription()`/`fulfillSubscriptionV2()` —
  апсерт `Subscription` (`plan: "PRO"`, `validUntil`).
- Реквизиты платёжных провайдеров хранятся в БД (`PaymentConfig`, одна строка),
  не только в `.env` — секретные поля скрываются в GET-ответах панели
  (`"********"`).
- Тестовые скрипты `test-payme.sh`/`test-payments.sh` (корень репозитория)
  прогоняют сценарии вебхуков против `localhost:3031` или
  `https://contador.uz/admin/api/...`.

## Существующая документация API в проекте

`docs/04_API_REFERENCE.md` существует (753 строки, самоуказанная версия
«2.0», дата «2026-06-26»), описывает базовый URL `https://contador.uz/v2/api`
и `v2_session`-cookie — совпадает с тем, что подтверждено в коде выше. Файл
датирован 2026-07-03 по mtime; **42 из 78** файлов маршрутов v2 имеют более
позднюю дату изменения, чем этот документ — он, вероятнее всего, устарел
относительно текущего состава маршрутов (точный построчный дифф не проводился).
