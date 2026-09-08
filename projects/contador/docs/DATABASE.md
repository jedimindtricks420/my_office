# Database — Contador

Полностью переписано 2026-09-08 (`TASK-0002`). Оба файла схем прочитаны
целиком построчно: `prisma/schema.prisma` (280 строк) и
`v2/prisma/schema.prisma` (585 строк).

Проект использует **две полностью независимые базы** на одном инстансе
PostgreSQL 16 — `contador` (legacy v1 + admin) и `contador_v2` (основное
приложение). Модели с одинаковыми именами (`AccountType`, `PlanType`,
`Voucher`, `Payment`, `Subscription`, `AuditLog`, `Account`, `Counterparty`)
**не являются общими** — раздельные определения, разные поля и в ряде случаев
разные значения enum'ов.

---

## Схема v2 (`v2/prisma/schema.prisma`, БД `contador_v2`) — 22 модели, 20 enum'ов

`datasource`: `postgresql`, `url = env("DATABASE_URL")`.
`generator client`: output `../node_modules/.prisma/client`.

### ER-обзор (основные связи)

```
User ──< OrgMember >── Organization ──< BankAccount
  │                        │  │             │
  │ PasswordResetToken     │  │             ├─< StagedTransaction >── Period
  │                        │  │                        │
  │                        │  ├─< Period ──< Document ──< JournalEntry >── Account (иерархия parentId)
  │                        │  │      │           │              │
  │                        │  │      │           ├─< OpenItem ──┴── Counterparty
  │                        │  │      │           └── DocumentType (postingTemplate JSON)
  │                        │  │      └─< ClosingJob (state machine мастера закрытия)
  │                        │  ├─< Rule ── DocumentType
  │                        │  ├─< TaxCalendarEvent, TaxDeadlineTemplate
  │                        │  ├─< AuditLog
  │                        │  ├── Subscription (1:1)
  │                        │  ├─< Payment, Voucher
  │                        │  ├─< TaxReportAccountMapping (orgId=null → дефолт для всех)
  │                        │  └─< OrgCostingMethodHistory
```

### Модели (алфавитный порядок полей — как в файле)

- **User** (`schema.prisma:11-20`) — `id`, `email` (unique), `passwordHash`,
  `name?`, `createdAt`, `activeOrgId?` (String, **не foreign key** — просто
  ссылка на `Organization.id`, не проверяется на уровне схемы). →
  `memberships OrgMember[]`, `passwordResetTokens[]`.
- **Organization** (22-74) — центр tenant-изоляции. Ключевые поля:
  `taxRegime` (enum, default `TURNOVER_TAX`), `isVatPayer` (default `false`),
  `autoAccrueProfitTax` (default `true`), `profitTaxRate` (Float, default
  `0.15`), `taxBenefit` (enum, default `NONE`), `itParkResidentSince?`,
  `itParkCertificateNumber?`, `avgHeadcount`/`avgHeadcountDisabled` (для шапки
  «Расчёта налога на прибыль»), `turnoverTaxRate` (default `0.04`),
  `aiConfidenceThreshold` (default `70`), `maxClarificationQuestions`
  (default `10`), `settings Json?` (используется как свободная карта:
  `hiddenAccounts[]`, `openItemDeadlines{}` — не отдельные таблицы),
  `activityGroup/Description/Custom` (категория деятельности, кормит AI-
  промпт), `charterCapitalAmount Decimal(18,2)?`, `charterCapitalDeclaredAt?`,
  `charterCapitalFundingType?` (enum). 15 обратных связей (см. ER выше).
- **OrgMember** (76-85) — `role` (enum `OrgRole`, default `OWNER`), `userId`,
  `orgId`; `@@unique([userId, orgId])`.
- **BankAccount** (87-100) — `orgId`, `name`, `currency` (default `"UZS"`),
  `bankName?`, `accountNumber?`, `lastBalance Decimal(20,2)` (default `0`),
  `lastSyncedAt?`.
- **Period** (102-119) — `orgId`, `year`, `month`, `mode` (enum `PeriodMode`,
  default `ACTIVE`), `status` (enum `PeriodStatus`, default `OPEN`),
  `lockDate?`, `closingData Json?` (устаревшее поле — состояние мастера
  закрытия мигрировало в отдельную таблицу `ClosingJob`, см. GAPS.md);
  `@@unique([orgId, year, month])`.
- **StagedTransaction** (121-146) — банковская операция до классификации.
  `bankAccountId`, `periodId`, `date`, `amount Decimal(20,2)`, `direction`
  (enum `CREDIT`/`DEBIT`), `description`, `counterpartyHint?`,
  `counterpartyInn?`, `hash` (SHA-256 для дедупликации импорта), `status`
  (enum, default `IMPORTED`: IMPORTED → AUTO_MATCHED/NEEDS_CLARIFICATION →
  CONFIRMED/SKIPPED; значение `POSTED` в enum существует, но код никогда его
  не присваивает — см. GAPS.md), `aiSuggestion Json?`, `documentId?`,
  `importBatchId?`; `@@unique([orgId, hash])`.
- **Rule** (148-163) — правило классификации: `matchType` (enum
  `RuleMatchType`), `matchValue`, `categoryId → DocumentType`, `direction?`,
  `createdFrom` (enum `RuleSource`), `order`; `@@unique([orgId, matchType,
  matchValue])`.
- **Account** (165-182) — план счетов (**глобальный, без `orgId`** — общий для
  всех организаций; видимость на организацию регулируется через
  `Organization.settings.hiddenAccounts[]`, не отдельной таблицей). `code`
  (unique), `name`, `type` (enum `AccountType`), `parentId?` (самоссылка —
  иерархия), `isSystem` (default `true`), `group?`, `isDeprecated` (default
  `false`), `layer` (enum `AccountLayer`: CORE/EXTENSION/INDUSTRY).
- **DocumentType** (184-192) — `code` (unique), `name`, `postingTemplate
  Json` (шаблон проводки, интерпретируется `postingEngine.ts`), `mode`
  (String, default `"BANK_AUTO"`; реально встречающиеся значения по коду:
  `BANK_AUTO`, `MANUAL_ONLY`, `HYBRID` — не enum, значение не валидируется на
  уровне схемы).
- **Document** (194-222) — `orgId`, `periodId`, `typeId`, `date`, `status`
  (enum `DocumentStatus`, default `POSTED`; `VOIDED` — вторая и последняя
  ценность), `payload Json` (свободная форма, специфична для типа
  документа), `sourceTransactionId? @unique`, `correctionForPeriodId?`
  (объявлено, не используется ни одним найденным местом кода — см.
  GAPS.md), `taxDeductibleOverride Boolean?` (переопределение вычитаемости
  расхода для налогового отчёта), `taxCalendarSyncStatus?`/`SyncError?`.
- **JournalEntry** (224-240) — проводка двойной записи: `documentId`,
  `accountId`, `debit Decimal(20,2)`, `credit Decimal(20,2)`, `date`,
  `counterpartyId?`, `contractId?` (объявлено, ни разу не заполняется кодом —
  см. GAPS.md).
- **Counterparty** (242-252) — `orgId`, `name`, `inn?`.
- **OpenItem** (254-276) — буферная «открытая позиция»: `accountId`,
  `counterpartyId?`, `openingDocumentId`, `amount`, `dateOpened`, `status`
  (enum `OpenItemStatus`: OPEN/CLOSED/RISK), `riskDeadline?`,
  `closingDocumentId?`, `dateClosed?`, `affectedPeriodId?`,
  `awaitingInvoiceSign Boolean` (default `false`, объявлено, нигде не
  устанавливается в `true` найденным кодом — см. GAPS.md).
- **TaxCalendarEvent** (278-291) — `periodId?`, `type` (enum
  `TaxEventType`: VAT/PERSONAL_INCOME_TAX/TURNOVER_TAX/PROFIT_TAX/
  STATISTICS/SOCIAL_TAX/INPS), `dueDate`, `status` (enum, PENDING/DONE),
  `estimatedAmount?`, `note?`. `STATISTICS` присутствует в enum, но ни одно
  место кода не создаёт событие с этим типом — см. GAPS.md.
- **TaxDeadlineTemplate** (293-304) — `type`, `dayOfMonth`, `frequency`
  (enum `Frequency`), `taxRegime?`.
- **AuditLog** (306-319) — `userId`, `action`, `entityType?`, `entityId?`,
  `oldValue Json?`, `newValue Json?`. Реально пишется из
  `postingEngine.ts` (POST_DOCUMENT/VOID_DOCUMENT) и `closing.ts`
  (YEAR_END_CLOSE) — не из большинства route-хендлеров.
- **Subscription** (321-329) — 1:1 с `Organization`, `plan` (enum `PlanType`:
  FREE/PRO), `validUntil?`, `customApiKey?`.
- **Payment** (331-350) — `orderCode? @unique`, `provider String`
  (не enum — значения `PAYME`/`CLICK`/`ALIF` контролируются только кодом
  `admin/server.ts`), `externalId @unique`, `amount`, `currency` (default
  `"UZS"`), `status String` (default `"PENDING"`; значения `PENDING`/
  `PROCESSING`/`SUCCESS`/`FAILED` — тоже не enum), `daysGranted` (default
  `365`), `completedAt?`, `cancelTime?`, `cancelReason?`.
- **Voucher** (352-363) — `code @unique`, `series` (default `"CV2"`),
  `daysGranted`, `usedByOrgId? → Organization` (`onDelete: SetNull`),
  `usedAt?`.
- **PasswordResetToken** (365-376) — `token @unique`, `expiresAt`, `usedAt?`.
- **ClassificationJob** (486-500) — `id` **без `@default`** (в коде — либо
  `periodId`, либо `${orgId}_ALL`, см. `queue.ts:13`), `status String`
  (default `"running"`: running/done/failed), `total`/`processed`/`matched`/
  `needsClarification` (счётчики прогресса AI-классификации), `error?`.
- **TaxReportAccountMapping** (511-526) — маппинг «счёт (+опционально тип
  документа) → строка Приложения №1/№2». `orgId?` (`null` = глобальный
  дефолт, сидится `ensureTaxReportMapping()`; заполненный `orgId` —
  переопределение организации), `documentTypeCode?` (правило по типу
  документа приоритетнее правила по одному счёту), `appendixCode` (enum
  `TaxAppendixCode`), `lineCode`, `column` (enum `TaxReportColumn`: TOTAL/
  NON_DEDUCTIBLE), `isDefault`.
- **OrgCostingMethodHistory** (533-544) — метод себестоимости
  (PROPORTIONAL/DIRECT) по налоговым годам, `@@unique([orgId, fiscalYear])`
  — «тянется» из последней более ранней записи, если на текущий год нет
  своей (см. `taxReport/engine.ts:122-128`).
- **ClosingJob** (566-584) — state machine мастера закрытия месяца:
  `status String` (default `"DRAFT"`: DRAFT/RUNNING/FAILED/COMPLETED/
  CANCELLED — `CANCELLED` объявлен, ни один найденный код-путь его не
  устанавливает, см. GAPS.md), `step Int`, `data Json` (default `{}`,
  хранит `accruals`/`fxDiff`/`soliqMatched`), `error?`, `startedAt?`,
  `completedAt?`; `@@unique([periodId, orgId])`.

### Enum'ы (20)

`TaxRegime` (VAT, TURNOVER_TAX) · `OrgRole` (OWNER, ACCOUNTANT, ADMIN) ·
`CharterCapitalFundingType` (FULLY_PAID_CASH, PARTIALLY_PAID, PAID_IN_KIND,
NOT_PAID) · `PeriodMode` (HISTORICAL, ACTIVE) · `PeriodStatus` (OPEN, CLOSED)
· `TransactionDirection` (CREDIT, DEBIT) · `StagedTransactionStatus`
(IMPORTED, AUTO_MATCHED, NEEDS_CLARIFICATION, CONFIRMED, POSTED, SKIPPED) ·
`RuleMatchType` (INN, KEYWORD, AMOUNT_RANGE, TREASURY_ACCOUNT) ·
`RuleSource` (USER_ANSWER, MANUAL, AI_SUGGESTED) · `AccountType` (ASSET,
CONTRA_ASSET, LIABILITY, CONTRA_LIABILITY, ACTIVE_PASSIVE, TRANSIT,
OFF_BALANCE) · `AccountLayer` (CORE, EXTENSION, INDUSTRY) · `DocumentStatus`
(POSTED, VOIDED) · `OpenItemStatus` (OPEN, CLOSED, RISK) · `TaxEventType`
(VAT, PERSONAL_INCOME_TAX, TURNOVER_TAX, PROFIT_TAX, STATISTICS,
SOCIAL_TAX, INPS) · `TaxEventStatus` (PENDING, DONE) · `PlanType` (FREE,
PRO) · `Frequency` (MONTHLY, QUARTERLY, ANNUALLY) · `TaxBenefitType` (NONE,
IT_PARK_RESIDENT) · `CostingMethod` (PROPORTIONAL, DIRECT) ·
`TaxAppendixCode` (APPENDIX_1, APPENDIX_2) · `TaxReportColumn` (TOTAL,
NON_DEDUCTIBLE).

### План счетов (`v2/src/lib/seed-coa.ts`)

`MASTER_COA` — подтверждено построчным `grep`: **269 записей** (не 214 —
комментарий в шапке файла `seed-coa.ts:12` ссылается на источник «НСБУ №21,
214 счетов», сам массив шире официального плана за счёт слоёв `EXTENSION`/
`INDUSTRY`; расхождение числа в комментарии с фактической длиной массива —
незначительное, но реальное, см. GAPS.md). Каждая запись: `{ code, name,
type, group, layer, isDeprecated }`. Применяется идемпотентным upsert'ом по
`code` (`upsertAllAccounts()`, `seed-coa.ts`) — обновляет существующие,
создаёт отсутствующие, не удаляет.

---

## Схема legacy v1/admin (`prisma/schema.prisma`, БД `contador`) — 14 моделей, 5 enum'ов

- **User** (`password_hash`, snake_case-поля) — `active_org_id?` (String,
  тоже без FK-constraint).
- **Organization** — принадлежит **одному** `User` (`user_id`, 1:N, не M:N
  через отдельную таблицу членства, в отличие от v2's `OrgMember`),
  `onboarding_state`/`onboarding_step`, `ai_description?`, `template_id? →
  IndustryTemplate`.
- **Account** — `organization_id` (**в отличие от v2, здесь план счетов
  организации-специфичен**, не глобальный), `is_postable`, `is_custom`,
  `master_account_id? → MasterAccount`; `@@unique([code, organization_id])`.
- **Counterparty**, **Transaction** (проводка: `debit_id`/`credit_id →
  Account`, `period String` в формате `MM.YYYY` — не отдельная таблица
  периодов, как в v2's `Period`).
- **SystemSettings** — одна запись на организацию (`@@unique`):
  `closed_period_date`, `opening_balance_date`, `is_initial_balance_fixed`.
- **MasterAccount** / **IndustryTemplate** / **IndustryTemplateItem** —
  эталонный план счетов и отраслевые наборы для онбординга (концепция,
  отсутствующая в схеме v2).
- **AuditLog**, **ChatMessage** (AI-чат ввода проводок — специфично для v1,
  аналога в v2 нет), **AiUsage** (учёт токенов/стоимости).
- **Subscription**, **Payment** (`amount_tiyin BigInt?`, `provider` enum
  `PaymentProvider`), **PaymentConfig** (**одна строка**, `id: "default"`:
  `payme_merchant_id/key/test_key/env`, `click_merchant_id/service_id/
  secret_key/env`, `alif_token_production/sandbox`,
  `alif_secret_key_production/sandbox`, `alif_env`, `pro_price_yearly`
  (default `299000`) — секреты платёжных провайдеров хранятся здесь, в БД,
  используются **и v1-платежами, и v2-платежами** через `admin/server.ts`),
  **Voucher** (`days_granted`, `is_used`, `used_by_org_id?`).

### Enum'ы (5)

`OnboardingState` (PENDING, IN_PROGRESS, COMPLETED) · `AccountType`
(ACTIVE, CONTRA_ACTIVE, PASSIVE, CONTRA_PASSIVE, ACTIVE_PASSIVE, INCOME,
CONTRA_INCOME, EXPENSE, OFF_BALANCE) — **не пересекается по значениям** с
`AccountType` схемы v2 (там ASSET/LIABILITY/TRANSIT и т.д.) · `PlanType`
(FREE, PRO, MYAPI — v2 не знает `MYAPI`) · `PaymentProvider` (PAYME, CLICK,
VOUCHER — **`ALIF` в этом enum отсутствует**, хотя код `admin/server.ts`
явно принимает и обрабатывает Alifpay-платежи и для v1-таблицы `Payment`
тоже (`provider: provider as any` — приведение типа в обход enum, см.
GAPS.md)) · `PaymentStatus` (PENDING, SUCCESS, FAILED).

---

## Миграции — реальность против конвенции Prisma

Оба каталога `migrations/` **не соответствуют** стандартному формату Prisma
(`<timestamp>_<name>/migration.sql` + `migration_lock.toml`) — подтверждено
структурно: это плоские директории с одиночными `.sql`-файлами без
подпапок и без `migration_lock.toml` где-либо в репозитории.

### `v2/prisma/migrations/` — 9 файлов
```
20260703_closing_job.sql
20260703_tax_sync_and_rule_unique.sql
20260703_turnover_tax_rate_check.sql
20260704_document_source_transaction_unique.sql
20260705_charter_capital.sql
20260709_auto_accrue_profit_tax.sql
20260710_quarterly_profit_tax.sql
20260711_profit_tax_report_generator.sql
20260731_voucher_series.sql
```
9 инкрементальных SQL-патчей не могут покрыть эволюцию всей 22-модельной,
585-строчной схемы — начальное состояние базы, очевидно, создано один раз
через `prisma db push` (скрипт `db:push` есть в `v2/package.json`), а
патчи — только последующие точечные изменения. Применяются вручную через
`psql` (в `v2/package.json` нет ни одного скрипта, вызывающего
`migrate deploy`; `db:migrate` = `prisma migrate dev`, не production-команда).

### `prisma/migrations/` (root/legacy) — 1 файл

`20260703_account_type_enum_cleanup.sql` — убирает значение `TRANSIT`, ранее
случайно попавшее в enum `AccountType` схемы v1, плюс удаляет устаревший
`DEFAULT` у `SystemSettings.opening_balance_date`.

## Сидинг данных — не `prisma db seed`

Ни один `package.json` не содержит секции `"prisma": { "seed": ... }` или
скрипта `db:seed`. Вместо этого — идемпотентная функция
`ensureBaseData()` (`v2/src/lib/ensureBaseData.ts:2784-2837`), вызываемая
**на каждом старте Node.js-рантайма** через официальный хук Next.js
`instrumentation.ts` (`register()`, `v2/src/instrumentation.ts:1-6`):

1. `upsertAllAccounts()` — 269 счетов плана НСБУ.
2. `validateTemplate()` прогоняется по всем 180 `baseDocumentTypes` (кроме
   `PERIOD_CLOSING`, у которого проводки строятся вручную в `closing.ts`) —
   бросает исключение при структурной ошибке шаблона **до** записи в БД
   (`ensureBaseData.ts:2797-2808`).
3. Upsert 180 типов документов по `code`.
4. Миграция legacy-правил `SUPPLIER_PAYMENT` → `SUPPLIER_PAYMENT_SERVICES`.
5. `ensureTaxReportMapping()` — синхронизация `TaxReportAccountMapping`
   (`orgId: null`) с `DEFAULT_TAX_REPORT_MAPPING`.

Отдельный `v2/run-seed.mjs` — одноразовый ad-hoc скрипт (создан вручную,
не вызывается ни из `package.json`, ни из Dockerfile), добавляющий несколько
типов документов (`INPS_PAYMENT`, `SALARY_OFFSET`, `ADVANCE_RETURN_SENT`,
`FOUNDER_LOAN_REPAYMENT`), которые к текущему моменту уже присутствуют и в
`baseDocumentTypes` — похоже на исторический скрипт миграции, оставленный в
репозитории (см. GAPS.md).
