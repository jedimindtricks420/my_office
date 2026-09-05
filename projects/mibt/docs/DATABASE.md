# DATABASE — MIBT

Последнее обновление: 2026-09-05. Источник: `prisma/schema.prisma` (прочитан
полностью, 401 строка), `prisma/migrations/` (список файлов).

## СУБД и подключение

PostgreSQL 16 (`postgres:16-alpine`), доступ через `DATABASE_URL` (Prisma
Client, `@prisma/client` 6.19.3). Локально в Docker Compose БД называется
`mibt_db`, пользователь `mibt_user`, контейнер не публикует порт наружу —
доступен только внутри Docker-сети (см. [DEPLOYMENT.md](DEPLOYMENT.md)).

## Миграции

3 миграции применены (`prisma/migrations/`, `migration_lock.toml` →
provider `postgresql`):

| Миграция | Назначение (по имени) |
|---|---|
| `20260608201128_init` | Начальная схема |
| `20260613000000_add_payment_document_type` | Добавление типа документа `PAYMENT` (загрузка платёжки клиентом) |
| `20260613100000_add_fk_indexes` | Индексы на внешних ключах |

Содержимое SQL самих миграций не читалось построчно — использовались только
имена файлов и итоговая `schema.prisma` как источник истины по текущей форме
таблиц.

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
