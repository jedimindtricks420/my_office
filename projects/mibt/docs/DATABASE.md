# DATABASE — MIBT

Последнее обновление: 2026-09-08 (TASK-0002, повторная построчная сверка).
Источник: `prisma/schema.prisma` (401 строка, прочитан полностью),
`prisma/migrations/*/migration.sql` (прочитаны все 3, включая SQL).

## ER-диаграмма (текстовая, по `@relation` в схеме)

```
User (1) ──── (1) Customer (1) ──── (1) CustomerBalance
                    │  │  │
                    │  │  └── (1:N) BalanceRequest
                    │  └───── (1:N) BalanceTransaction
                    └──────── (1:N) Order ──(1:N)── OrderLine ──(N:1)── ProductVariant ──(N:1)── Product ──(1:N)── ProductLicense
                    │                │                  │
                    │                │                  └──(1:N)── OrderLineLicense
                    │                └──(1:N)── OrderDocument ──(1:N)── DocumentDownloadToken
                    │
                    └──────── (1:N) QuoteRequest ──(1:1)── QuoteDocument ──(1:N)── QuoteDownloadToken

SystemSetting (независимая key-value таблица)
AuditLog (независимая таблица, FK на User только по значению userId, без @relation)
session (создаётся connect-pg-simple, вне управления Prisma)
```

## СУБД и подключение

PostgreSQL 16 (`postgres:16-alpine`), доступ через `DATABASE_URL` (Prisma
Client, `@prisma/client` 6.19.3). Локально в Docker Compose БД называется
`mibt_db`, пользователь `mibt_user`, контейнер не публикует порт наружу —
доступен только внутри Docker-сети (см. [DEPLOYMENT.md](DEPLOYMENT.md)).

## Миграции

3 файла миграций (`prisma/migrations/`, `migration_lock.toml` → provider
`postgresql`), **все прочитаны построчно** в этом аудите:

| Миграция | SQL (кратко) |
|---|---|
| `20260608201128_init` | `CREATE TABLE` для 15 таблиц (все, кроме `quote_*` и `session`), все `@@map`-имена, unique-индексы (`users_email_key`, `customers_userId_key`, `customers_inn_key`, `customer_balances_customerId_key`, `products_slug_key`, `product_variants_sku_key`, `order_line_licenses_orderLineId_position_key`, `document_download_tokens_token_key`) и FK-констрейнты с явным `ON DELETE CASCADE`/`RESTRICT` |
| `20260613000000_add_payment_document_type` | `ALTER TYPE "DocumentType" ADD VALUE IF NOT EXISTS 'PAYMENT';` |
| `20260613100000_add_fk_indexes` | `CREATE INDEX ... ON "orders"("customerId")`, `CREATE INDEX ... ON "balance_requests"("customerId")` |

### ⚠️ Расхождение: таблицы КП не покрыты ни одной миграцией

`grep -ri "quote" prisma/migrations/*/migration.sql` не находит **ни одного**
совпадения — миграция, создающая `quote_requests`/`quote_documents`/
`quote_download_tokens`, отсутствует, хотя эти модели есть в
`schema.prisma`. Ни `Dockerfile`, ни `docker-compose.yml`, ни
`package.json` не запускают `prisma migrate deploy` ни на одном этапе
(`Dockerfile` вызывает только `npx prisma generate` — генерацию клиента).
Подробности и последствия — [GAPS.md](GAPS.md), раздел 2.7.

## Модели (18 моделей, 9 enum'ов — подтверждено `grep -c`)

### Пользователи и организации

- **User** — `email` (unique), `passwordHash` (bcrypt), `role` (`Role`),
  поля OTP для сброса пароля (`emailOtp`, `emailOtpExpiry`). 1:1 с `Customer`.
- **Customer** — `companyName`, `inn` (unique), `bankAccount`, `mfo`,
  `address`, `clientType` (`ClientType`), `archivedAt` (мягкое удаление/бан).
- Enum `Role`: `CUSTOMER`, `ADMIN`.
- Enum `ClientType`: `CORPORATE`, `BUDGET` (B2G).

### Баланс

- **CustomerBalance** — `creditAmount` (`BigInt`), `frozenAmount` (`BigInt`),
  1:1 с `Customer`.
- **BalanceRequest** — заявка клиента на пополнение, `amount` (`BigInt`),
  `status` (`BalanceRequestStatus`).
- **BalanceTransaction** — журнал движений: `amount`, `balanceBefore`,
  `balanceAfter` (все `BigInt`), `type` (`BalanceTransactionType`),
  `adminId` (строка, не FK на `User` — см. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).
- Enum `BalanceRequestStatus`: `PENDING`, `APPROVED`, `REJECTED`.
- Enum `BalanceTransactionType`: `CREDIT`, `DEBIT`, `FREEZE`, `UNFREEZE`, `REFUND`.

### Товары и лицензии

- **Product** — `price` (`BigInt`), `vatRate` (по умолчанию 12%), `ikpu`
  (код ИКПУ для узбекского налогового учёта), `specs` (`Json?`),
  `productType` (`ProductType`).
- **ProductVariant** — SKU-варианты товара (свой `price`, `sku` unique).
- **ProductLicense** — пул лицензионных ключей на уровне товара
  (`encryptedLicenseKey`, `status`). **Наполняется через
  `POST /api/admin/products/:id/licenses`, но не потребляется** — код
  назначения (`assignLicensesToOrder`) не вызывается нигде (см. AGENTS.md /
  OPEN_QUESTIONS.md). Фактически данные, попавшие сюда, не влияют на заказы.
- Enum `ProductType`: `SOFTWARE`, `HARDWARE`.
- Enum `LicenseStatus`: `AVAILABLE`, `ASSIGNED`.

### Заказы

- **Order** — `customerId` (`onDelete: Restrict` — заказ блокирует удаление
  клиента), `status` (`OrderStatus`), `totalPrice`/`vatAmount` (`BigInt`),
  `contactName/Phone/Email`, `customerNote`, `adminNote`.
- **OrderLine** — позиция заказа, `productVariantId`, `quantity`,
  `unitPrice` (`BigInt`), `vatRate`.
- **OrderLineLicense** — фактически выданный ключ на конкретную позицию
  заказа (`encryptedLicenseKey`, `position`, уникальность по
  `[orderLineId, position]`) — это реально используемый путь выдачи ключей
  (вводится вручную администратором через
  `POST /api/admin/orders/:id/lines/:lineId/licenses`).
- Enum `OrderStatus`: `PENDING`, `IN_PROGRESS`, `DELIVERED`, `CANCELLED`.

### Документы

- **OrderDocument** — файл, привязанный к заказу (`type`, `originalName`,
  `filePath` — имя файла в `secure-assets/`, не полный путь).
- **DocumentDownloadToken** — одноразовый токен скачивания (`expiresAt`,
  `used`), связан с `OrderDocument`.
- Enum `DocumentType`: `CONTRACT`, `INVOICE`, `ACT`, `COMMERCIAL_OFFER`,
  `SPECIFICATION`, `KEY_FILE`, `PAYMENT` (последний добавлен миграцией
  `add_payment_document_type`).

### Коммерческие предложения (КП)

- **QuoteRequest** — `items` (`Json`, снимок корзины), `totalAmount`/
  `vatAmount` — **намеренно `String`, а не `BigInt`** (комментарий в схеме:
  во избежание проблем с сериализацией `BigInt`), `status`
  (`QuoteRequestStatus`).
- **QuoteDocument** / **QuoteDownloadToken** — аналог `OrderDocument`/
  `DocumentDownloadToken` для документов КП (у `QuoteDownloadToken` нет поля
  `used` в отличие от `DocumentDownloadToken` — удаляется сразу после
  скачивания, см. код роута).
- Enum `QuoteRequestStatus`: `NEW`, `PROCESSED`.

### Прочее

- **SystemSetting** — key-value хранилище (реквизиты компании для
  реквизитов пополнения баланса, `ui_theme` — активная цветовая тема).
- **AuditLog** — `userId`, `action`, `entityType`, `entityId`, `metadata`
  (`Json?`), `ipAddress` — пишется через `services/audit.service.js`
  (`logAudit`), ошибки записи только логируются, не прерывают запрос.

## Полные таблицы полей (по `prisma/schema.prisma`, построчно)

### User (`users`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id @default(cuid())` |
| email | String | `@unique` |
| passwordHash | String | — |
| role | Role | `@default(CUSTOMER)` |
| emailOtp | String? | — |
| emailOtpExpiry | DateTime? | — |
| customer | Customer? | обратная связь 1:1 |
| createdAt / updatedAt | DateTime | `@default(now())` / `@updatedAt` |

### Customer (`customers`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id @default(cuid())` |
| userId | String | `@unique`, FK → `User.id`, `onDelete: Cascade` |
| companyName | String | — |
| inn | String | `@unique` |
| bankAccount, mfo, address | String? | — |
| clientType | ClientType | `@default(CORPORATE)` |
| archivedAt | DateTime? | мягкая блокировка аккаунта (проверяется при логине) |
| balance, orders, balanceRequests, transactions, quoteRequests | relations | 1:1 / 1:N |

### CustomerBalance (`customer_balances`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| customerId | String | `@unique`, FK, `onDelete: Cascade` |
| creditAmount | BigInt | `@default(0)`, тийины |
| frozenAmount | BigInt | `@default(0)`, тийины |

### BalanceRequest (`balance_requests`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| customerId | String | FK, `onDelete: Cascade`, `@@index` |
| amount | BigInt | тийины |
| status | BalanceRequestStatus | `@default(PENDING)` |
| note | String? | — |

### BalanceTransaction (`balance_transactions`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| customerId | String | FK, `onDelete: Cascade` |
| amount, balanceBefore, balanceAfter | BigInt | тийины |
| type | BalanceTransactionType | — |
| note | String? | — |
| orderId | String? | **не FK** — просто строка, без `@relation` на `Order` |
| adminId | String | **не FK** — просто строка (может быть литерал `'system'` для авто-операций, см. `services/balance.service.js`) |
| createdAt | DateTime | нет `updatedAt` — запись неизменяема |

### Product (`products`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| name | String | — |
| slug | String | `@unique` |
| description, overview, manufacturer, mfrPartNo, imagePath | String? | — |
| specs | Json? | произвольная структура (плоская или сгруппированная — определяется в `views/client/product.ejs` эвристикой `isGrouped`) |
| price | BigInt | тийины, базовая цена (реально используется только если у товара 0 вариантов) |
| vatRate | Int | `@default(12)`, проценты |
| ikpu | String | код ИКПУ (обязателен, без явного формата на уровне схемы) |
| productType | ProductType | `@default(SOFTWARE)` |
| isActive | Boolean | `@default(true)` — флаг «черновик/опубликован» |
| variants, licenses | relations | 1:N |

### ProductVariant (`product_variants`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| productId | String | FK, `onDelete: Cascade` |
| name | String | — |
| price | BigInt | тийины |
| sku | String | `@unique` глобально (не только в рамках товара) |
| orderLines | relation | 1:N |

### ProductLicense (`product_licenses`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| productId | String | FK, `onDelete: Cascade` |
| encryptedLicenseKey | String | AES-256-GCM, формат `iv:tag:ciphertext` (hex) |
| status | LicenseStatus | `@default(AVAILABLE)` — **на практике никогда не переходит в `ASSIGNED`, см. GAPS.md 2.1** |
| assignedLineId | String? | не FK — просто строка; заполняется только кодом `assignLicensesToOrder`, который не вызывается |
| assignedAt | DateTime? | то же |

### Order (`orders`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| customerId | String | FK, **`onDelete: Restrict`** — нельзя удалить клиента с заказами, `@@index` |
| status | OrderStatus | `@default(PENDING)` |
| totalPrice, vatAmount | BigInt | тийины |
| contactName, contactPhone, contactEmail, customerNote, adminNote | String? | снимок контактов на момент заказа |
| lines, documents | relations | 1:N |

### OrderLine (`order_lines`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| orderId | String | FK, `onDelete: Cascade` |
| productVariantId | String | FK, **без `onDelete`** (по умолчанию Prisma/Postgres — `Restrict`) — нельзя удалить вариант, если по нему есть строки заказов (используется в проверках `_count.orderLines` перед удалением) |
| quantity | Int | — |
| unitPrice | BigInt | тийины, снимок цены на момент заказа (не ссылка на текущую цену варианта) |
| vatRate | Int | снимок ставки НДС на момент заказа |
| licenses | relation | 1:N |

### OrderLineLicense (`order_line_licenses`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| orderLineId | String | FK, `onDelete: Cascade` |
| encryptedLicenseKey | String | AES-256-GCM |
| position | Int | вместе с `orderLineId` — `@@unique([orderLineId, position])` |

### OrderDocument (`order_documents`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| orderId | String | FK, `onDelete: Cascade` |
| type | DocumentType | — |
| originalName | String | имя файла, как загрузил пользователь |
| filePath | String | **только имя файла** в `secure-assets/`, не полный путь |
| downloadTokens | relation | 1:N |

### DocumentDownloadToken (`document_download_tokens`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| token | String | `@unique` — но код обращается к записи по `id`, а не по `token` (см. `routes/api/documents.api.js:142` — `where: { id: req.params.token }`); поле `token` заполняется `crypto.randomUUID()` при создании, отдельно от `id` (тоже `cuid()`) — то есть в URL передаётся `id`, а не значение `token` |
| documentId | String | FK, `onDelete: Cascade` |
| expiresAt | DateTime | TTL 5 минут от создания |
| used | Boolean | `@default(false)` — **никогда не читается и не записывается в коде, см. GAPS.md «Мёртвый код» п.7**; вместо этого запись целиком удаляется после использования |

### QuoteRequest (`quote_requests`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| customerId | String | FK, `onDelete: Restrict` |
| status | QuoteRequestStatus | `@default(NEW)` |
| items | Json | снимок позиций корзины на момент запроса (произвольная структура, задаётся клиентом при `POST /api/quotes`, ограничена Joi-схемой `createQuoteRequest`) |
| totalAmount, vatAmount | **String** | намеренно не `BigInt` (комментарий в схеме — во избежание проблем сериализации) |
| adminNote | String? | — |
| document | relation | 1:1 |

### QuoteDocument (`quote_documents`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| quoteRequestId | String | `@unique`, FK, `onDelete: Cascade` |
| originalName, filePath | String | аналогично `OrderDocument` |
| downloadTokens | relation | 1:N |

### QuoteDownloadToken (`quote_download_tokens`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| documentId | String | FK, `onDelete: Cascade` |
| expiresAt | DateTime | TTL 5 минут |

Отличие от `DocumentDownloadToken`: нет полей `token` и `used` вообще — токеном для URL служит сам `id`, запись удаляется сразу после скачивания.

### SystemSetting (`system_settings`)
| Поле | Тип | Ограничения |
|---|---|---|
| key | String | `@id` |
| value | String | — |
| updatedAt | DateTime | `@updatedAt` |
| updatedBy | String? | не FK — строка (userId) |

### AuditLog (`audit_logs`)
| Поле | Тип | Ограничения |
|---|---|---|
| id | String | `@id` |
| userId | String | не FK — просто строка |
| action, entityType, entityId | String | — |
| metadata | Json? | — |
| ipAddress | String? | — |
| createdAt | DateTime | нет `updatedAt` |

**Таблица никогда не заполняется** — `logAudit()` определена, но не
вызывается ни из одного роута/сервиса (см. [GAPS.md](GAPS.md) 2.2).

## Деньги и единицы измерения

Все денежные суммы — `BigInt` в **тийинах** (1 UZS = 100 тийин), кроме сумм в
`QuoteRequest` (хранятся как `String` намеренно). Валидация минимальной суммы
пополнения баланса — `Joi.number().integer().min(1_000_000)` (=10 000 UZS) в
`middleware/validate.js`.

## Индексы

Явные `@@index`: `BalanceRequest.customerId`, `Order.customerId`. Остальные
внешние ключи проиндексированы миграцией `add_fk_indexes` (содержимое SQL не
проверялось построчно).

## Сессии

Таблица `session` создаётся автоматически `connect-pg-simple`
(`createTableIfMissing: true`) — **не описана в `schema.prisma`**, находится
вне управления Prisma.
