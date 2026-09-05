# Database — Contador

Снимок на 2026-09-05. Проект использует **две полностью независимые базы данных**
на одном PostgreSQL 16 инстансе (`postgres:16-alpine`, контейнер `contador-db`) —
`contador` (legacy v1 + admin) и `contador_v2` (основное приложение v2) — с двумя
независимыми Prisma-схемами. Модели с одинаковыми именами (`AccountType`,
`PlanType`, `Voucher`, `Payment`, `Subscription`, `AuditLog`) **не являются общими** —
это раздельные определения с разными наборами полей/значений.

⚠️ Проект **не использует** `prisma migrate deploy`/стандартную историю миграций
Prisma — обе директории миграций содержат **сырые `.sql`-файлы**, применяемые вручную
через `psql`. Скрипт `db:migrate` (`prisma migrate dev`) присутствует в
`v2/package.json`, но по имеющимся данным не отражает реальный процесс применения
изменений схемы (нет папок с Prisma-сгенерированной структурой миграций).

---

## Схема v2 (`v2/prisma/schema.prisma`, БД `contador_v2`)

`datasource`: `postgresql`, `url = env("DATABASE_URL")`.
`generator client`: `prisma-client-js`, `output = "../node_modules/.prisma/client"`.

### Модели (20)

- **User** — `id` (uuid), `email` (unique), `passwordHash`, `name?`, `createdAt`,
  `activeOrgId?`. → `memberships OrgMember[]`, `passwordResetTokens[]`.
- **Organization** — `id`, `name`, `inn?`, `taxRegime` (enum `TaxRegime`,
  default `TURNOVER_TAX`), `isVatPayer` (default `false`), `autoAccrueProfitTax`
  (default `true`), `profitTaxRate` (default `0.15`), `taxBenefit` (enum
  `TaxBenefitType`, default `NONE`), `itParkResidentSince?`,
  `itParkCertificateNumber?`, `avgHeadcount` (default `1`),
  `avgHeadcountDisabled` (default `0`), `createdAt`, `activityDescription?`,
  `turnoverTaxRate` (default `0.04`), `aiConfidenceThreshold` (default `70`),
  `maxClarificationQuestions` (default `10`), `settings Json?`,
  `activityCustom?`, `activityGroup?`, `charterCapitalAmount Decimal(18,2)?`,
  `charterCapitalDeclaredAt?`, `charterCapitalFundingType?` (enum). Центральная
  модель tenant-изоляции — почти все прочие модели ссылаются на `orgId`.
- **OrgMember** — `id`, `role` (enum `OrgRole`, default `OWNER`), `userId`, `orgId`;
  `@@unique([userId, orgId])`.
- **BankAccount** — `id`, `orgId`, `name`, `currency` (default `"UZS"`),
  `bankName?`, `accountNumber?`, `lastBalance Decimal(20,2)` (default `0`),
  `lastSyncedAt?`; `@@index([orgId])`.
- **Period** — `id`, `orgId`, `year Int`, `month Int`, `mode` (enum `PeriodMode`,
  default `ACTIVE`), `status` (enum `PeriodStatus`, default `OPEN`), `lockDate?`,
  `closingData Json?`; `@@unique([orgId, year, month])`.
- **StagedTransaction** — `id`, `orgId`, `bankAccountId`, `periodId`,
  `date`, `amount Decimal(20,2)`, `direction` (enum `TransactionDirection`),
  `description`, `counterpartyHint?`, `counterpartyInn?`, `hash` (дедупликация
  импорта), `status` (enum `StagedTransactionStatus`, default `IMPORTED`),
  `aiSuggestion Json?`, `documentId?`, `importBatchId?`, `createdAt`;
  `@@unique([orgId, hash])`.
- **Rule** — `id`, `orgId`, `matchType` (enum `RuleMatchType`), `matchValue`,
  `categoryId` (→ `DocumentType`), `direction?`, `createdFrom` (enum
  `RuleSource`, default `USER_ANSWER`), `order` (default `0`);
  `@@unique([orgId, matchType, matchValue])`.
- **Account** — `id`, `code` (unique), `name`, `type` (enum `AccountType`),
  `parentId?` (самоссылка, иерархия план счетов), `isSystem` (default `true`),
  `group?`, `isDeprecated` (default `false`), `layer` (enum `AccountLayer`,
  default `CORE`).
- **DocumentType** — `id`, `code` (unique), `name`, `postingTemplate Json`
  (шаблон проводки), `mode` (default `"BANK_AUTO"`).
- **Document** — `id`, `orgId`, `periodId`, `typeId`, `date`, `status` (enum
  `DocumentStatus`, default `POSTED`), `payload Json`,
  `sourceTransactionId? @unique`, `correctionForPeriodId?`,
  `taxDeductibleOverride?`, `taxCalendarSyncStatus?`, `taxCalendarSyncError?`.
- **JournalEntry** — `id`, `documentId`, `accountId`, `debit Decimal(20,2)`,
  `credit Decimal(20,2)`, `date`, `counterpartyId?`, `contractId?`; проводки
  двойной записи, источник данных для Формы №1/№2.
- **Counterparty** — `id`, `orgId`, `name`, `inn?`.
- **OpenItem** — `id`, `orgId`, `accountId`, `counterpartyId?`,
  `openingDocumentId`, `amount Decimal(20,2)`, `dateOpened`, `status` (enum
  `OpenItemStatus`, default `OPEN`), `riskDeadline?`, `closingDocumentId?`,
  `dateClosed?`, `affectedPeriodId?`, `awaitingInvoiceSign` (default `false`).
- **TaxCalendarEvent** — `id`, `orgId`, `periodId?`, `type` (enum
  `TaxEventType`), `dueDate`, `status` (enum `TaxEventStatus`, default
  `PENDING`), `estimatedAmount Decimal(20,2)?`, `note?`.
- **TaxDeadlineTemplate** — `id`, `orgId`, `type` (enum `TaxEventType`),
  `dayOfMonth Int`, `frequency` (enum `Frequency`), `taxRegime?`,
  `isActive` (default `true`).
- **AuditLog** — `id`, `orgId`, `userId`, `action`, `entityType?`, `entityId?`,
  `oldValue Json?`, `newValue Json?`, `createdAt`.
- **Subscription** — `id`, `orgId @unique`, `plan` (enum `PlanType`, default
  `FREE`), `validUntil?`, `customApiKey?`, `updatedAt`.
- **Payment** — `id`, `orderCode? @unique`, `orgId`, `provider String`,
  `externalId @unique`, `amount Decimal(20,2)`, `currency` (default `"UZS"`),
  `status` (default `"PENDING"`), `daysGranted` (default `365`), `createdAt`,
  `completedAt?`, `cancelTime?`, `cancelReason?`.
- **Voucher** — `id`, `code @unique`, `series` (default `"CV2"`),
  `daysGranted Int`, `usedByOrgId?`, `usedAt?`, `createdAt`.
- **PasswordResetToken** — `id`, `userId`, `token @unique`, `expiresAt`,
  `usedAt?`, `createdAt`.
- **ClassificationJob** — `id` (без `@default` — обычно = `periodId`), `orgId`,
  `status` (default `"running"`), `total`, `processed`, `matched`,
  `needsClarification`, `error?`, `createdAt`, `updatedAt`.
- **TaxReportAccountMapping** — `id`, `orgId?`, `accountCode`,
  `documentTypeCode?`, `appendixCode` (enum `TaxAppendixCode`), `lineCode`,
  `column` (enum `TaxReportColumn`, default `TOTAL`), `isDefault` (default
  `true`). `orgId: null` = глобальный дефолт, `orgId` заполнен = переопределение
  для организации.
- **OrgCostingMethodHistory** — `id`, `orgId`, `fiscalYear Int`,
  `costingMethod` (enum `CostingMethod`), `setAt`, `setById?`;
  `@@unique([orgId, fiscalYear])`.
- **ClosingJob** — `id`, `periodId`, `orgId`, `status` (default `"DRAFT"`),
  `step Int` (default `1`), `data Json` (default `{}`), `error?`, `startedAt?`,
  `completedAt?`, `createdAt`, `updatedAt`; `@@unique([periodId, orgId])`.
  Хранит прогресс мастера закрытия месяца (7 шагов).

### Enum'ы (17)

`TaxRegime` (VAT, TURNOVER_TAX) · `OrgRole` (OWNER, ACCOUNTANT, ADMIN) ·
`CharterCapitalFundingType` (FULLY_PAID_CASH, PARTIALLY_PAID, PAID_IN_KIND,
NOT_PAID) · `PeriodMode` (HISTORICAL, ACTIVE) · `PeriodStatus` (OPEN, CLOSED) ·
`TransactionDirection` (CREDIT, DEBIT) · `StagedTransactionStatus` (IMPORTED,
AUTO_MATCHED, NEEDS_CLARIFICATION, CONFIRMED, POSTED, SKIPPED) · `RuleMatchType`
(INN, KEYWORD, AMOUNT_RANGE, TREASURY_ACCOUNT) · `RuleSource` (USER_ANSWER,
MANUAL, AI_SUGGESTED) · `AccountType` (ASSET, CONTRA_ASSET, LIABILITY,
CONTRA_LIABILITY, ACTIVE_PASSIVE, TRANSIT, OFF_BALANCE) · `AccountLayer` (CORE,
EXTENSION, INDUSTRY) · `DocumentStatus` (POSTED, VOIDED) · `OpenItemStatus`
(OPEN, CLOSED, RISK) · `TaxEventType` (VAT, PERSONAL_INCOME_TAX, TURNOVER_TAX,
PROFIT_TAX, STATISTICS, SOCIAL_TAX, INPS) · `TaxEventStatus` (PENDING, DONE) ·
`PlanType` (FREE, PRO) · `Frequency` (MONTHLY, QUARTERLY, ANNUALLY) ·
`TaxBenefitType` (NONE, IT_PARK_RESIDENT) · `CostingMethod` (PROPORTIONAL,
DIRECT) · `TaxAppendixCode` (APPENDIX_1, APPENDIX_2) · `TaxReportColumn`
(TOTAL, NON_DEDUCTIBLE).

---

## Схема legacy v1/admin (`prisma/schema.prisma`, БД `contador`)

### Модели (14)

`User` (`password_hash`, snake_case-поля), `Organization` (принадлежит одному
`User`, есть `onboarding_state`/`onboarding_step`, `ai_description`,
`template_id → IndustryTemplate`), `Account` (`is_postable`, `is_custom`,
`master_account_id → MasterAccount`; `@@unique([code, organization_id])`),
`Counterparty`, `Transaction` (проводка: `debit_id`/`credit_id → Account`,
`period String` — формат `MM.YYYY`), `SystemSettings` (одна запись на
организацию: `closed_period_date`, `opening_balance_date`,
`is_initial_balance_fixed`), `MasterAccount` (эталонный план счетов для
шаблонов отраслей), `IndustryTemplate` / `IndustryTemplateItem` (наборы счетов
по отраслям), `AuditLog`, `ChatMessage` (AI-чат ввода проводок — специфично для
v1), `AiUsage` (учёт токенов/стоимости AI-запросов), `Subscription`, `Payment`
(`amount_tiyin BigInt?`, `provider` enum `PaymentProvider`), `PaymentConfig`
(**одна строка** конфигурации, `id: "default"` — хранит
`payme_merchant_id/key/test_key/env`, `click_merchant_id/service_id/secret_key/env`,
`alif_token_production/sandbox`, `alif_secret_key_production/sandbox`,
`alif_env`, `pro_price_yearly` (default `299000`); секретные реквизиты
платёжных провайдеров хранятся здесь, в БД, а не только в переменных
окружения), `Voucher` (`days_granted`, `is_used`, snake_case).

### Enum'ы (5)

`OnboardingState` (PENDING, IN_PROGRESS, COMPLETED) · `AccountType` (ACTIVE,
CONTRA_ACTIVE, PASSIVE, CONTRA_PASSIVE, ACTIVE_PASSIVE, INCOME, CONTRA_INCOME,
EXPENSE, OFF_BALANCE) — **не совпадает** со значениями `AccountType` в схеме v2
· `PlanType` (FREE, PRO, MYAPI) · `PaymentProvider` (PAYME, CLICK, VOUCHER) ·
`PaymentStatus` (PENDING, SUCCESS, FAILED).

---

## Миграции

### `v2/prisma/migrations/` — 9 файлов, все сырой SQL

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

Самая свежая (`20260731_voucher_series.sql`) добавляет `Voucher.series` (default
`'CV2'`) + индекс — соответствует текущей схеме. `20260711_...` — самая крупная
(создаёт 4 enum'а и таблицы `TaxReportAccountMapping`/`OrgCostingMethodHistory`,
полностью соответствует текущей схеме). Все проверенные файлы содержат
рукописные русскоязычные комментарии со ссылками на статьи Налогового кодекса —
не сгенерированы автоматически.

### `prisma/migrations/` (root/legacy) — 1 файл

`20260703_account_type_enum_cleanup.sql` — убирает значение `TRANSIT`, ранее
случайно попавшее в enum `AccountType` схемы v1 (через
`BEGIN`/`CREATE TYPE ... _new`/`ALTER TABLE ... USING`/`COMMIT`), плюс убирает
устаревший `DEFAULT` у `SystemSettings.opening_balance_date`. Комментарий в
файле явно описывает это как синхронизацию реальной БД со `schema.prisma`.

⚠️ README проекта фиксирует, что несколько миграций (в т.ч. `ClosingJob` и
колонки `Document.taxCalendarSyncStatus/Error`) существовали в коде неделями,
но не были применены к реальной БД до 2026-07-03, что приводило к падению
`postDocument()`. Это подтверждает, что раздельное отслеживание
"миграция в коде" vs "миграция применена к БД" — известный операционный риск
проекта, а не гипотетический.

---

## Сидинг данных (не через `prisma db seed`)

`package.json` (ни корневой, ни `v2/`) **не содержит** секции `"prisma": {
"seed": ... }` и не содержит скрипта `db:seed`. Вместо этого сидинг выполняется
идемпотентной функцией `ensureBaseData()` (`v2/src/lib/ensureBaseData.ts`,
~2900 строк) при старте приложения:

1. `upsertAllAccounts()` (`v2/src/lib/seed-coa.ts`) — апсертит план счетов НСБУ
   №21. Массив `MASTER_COA` подтверждён содержащим **269 записей** (совпадает с
   заявлением README), каждая запись: `{ code, name, type, group, layer,
   isDeprecated }`.
2. `validateTemplate()` прогоняется по каждому из **180** элементов
   `baseDocumentTypes` (кроме `PERIOD_CLOSING`, у которого проводки строятся
   вручную в `closing.ts`) — падает при структурной ошибке шаблона.
3. Апсерт всех 180 типов документов в `prisma.documentType` (по `code`).
   Массив помечен стадиями `STAGE 2`…`STAGE 6` внутри файла (явного маркера
   "STAGE 1" не найдено — вероятно, это неразмеченный исходный набор до
   введения такой разметки).
4. Миграция legacy-правил (`Rule`) с `SUPPLIER_PAYMENT` →
   `SUPPLIER_PAYMENT_SERVICES`, если оба типа существуют.
5. `ensureTaxReportMapping()` — сверяет `DEFAULT_TAX_REPORT_MAPPING`
   (`./taxReport/constants`) с существующими строками
   `TaxReportAccountMapping` с `orgId: null`, донастраивая их; строки с
   заполненным `orgId` (переопределения организаций) не трогает.
