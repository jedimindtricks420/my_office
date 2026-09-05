# DATABASE — POSA

Последнее обновление: 2026-09-05 (TASK-0001). Источник — прямое чтение
`/home/admin1/posa/activation-system/prisma/schema.prisma` (672 строки) и
каталога `prisma/migrations/` (28 миграций, с `20250211120000` по
`20260730232900`). СУБД — PostgreSQL 16, доступ через `DATABASE_URL` (Prisma).

## Модели (36 моделей, 12 enum'ов — по факту схемы)

Сгруппировано по смысловым доменам (группировка — интерпретация для
читаемости, сами модели плоские в одном файле):

### Пользователи и доступ
- `User` (роль через enum `Role`: `admin`, `merchant`, `vendor_user`,
  `financial_mgr`, `content_mgr`, `support_agent`, `kassa_admin`,
  `kassa_viewer`; опционально привязан к `Vendor` или `Kassa`)
- `RefreshToken` (remember-me, привязан к `User` ИЛИ `Client`, уникален по
  паре `(userId, role)` / `(clientId, role)`)
- `AuthSmsLog` (лог OTP-кодов через Eskiz, с полями `verified`/`verifiedAt`)
- `AuditLog` (действие, актор, IP, `details` JSON)

### Клиенты и ваучеры
- `Client` (идентификация по `phoneNumber`, без пароля)
- `Voucher` (`value` — сам код, `status`: `active|pending|sold|activated|
  deleted`; `type`: `Vendor|Telegram`)
- `VoucherActivation`, `OnlineVoucher`, `VoucherWalletLog` (добавление в
  Apple/Google Wallet, поле `pkpassId`, `deviceInfo`: `ios|android`),
  `VoucherSmsLog`

### Товары и вендоры
- `Vendor` (`productType` — enum `VendorProductType`:
  `ROKKY|DRWEB|MANUAL|VOUCHER|MANASTORE`; `balance`,
  `defaultCommissionPercent`, опционально привязан к `Kassa`)
- `Product` (цена, комиссии мерчанта/вендора, привязки к внешним SKU:
  `rokkySku`, `drwebSku`, `manastoreVariantId`; `walletOnDemandEnabled`)
- `ProductImage` (множественные изображения на товар)
- `Store` (витрина по `slug`, свой `activationSmsTemplate`, свои цвета темы)
- `MerchantProductLink` + `QrPaymentAttempt` — фича QR-платежей (см. ниже)

### Внешние интеграции поставки ключей
Три параллельные тройки моделей, одна на провайдера — `*Order` (заказ ключа
на конкретный ваучер, 1:1), `*ApiLog` (лог запросов), `*Sku` (справочник
товаров провайдера):
- Rokky: `RokkyOrder`, `RokkyApiLog`, `RokkySku`
- Dr.Web: `DrWebOrder`, `DrWebApiLog`, `DrWebSku`
- ManaStore: `ManaStoreOrder` (хранит `environment: staging|production`),
  `ManaStoreApiLog`, `ManaStoreSku`
- Ручная выдача: `ManualActivationRequest` (статусы `PENDING|COMPLETED|
  REJECTED|CANCELLED`, привязка к оператору `User`)

### Продажи и деньги
- `Merchant` (`status: active|off`, `balance`)
- `Sale` (`status: COMPLETED|CANCELLED|PENDING`, `saleType: ONLINE|OFFLINE`,
  `receiptPath` — путь к PDF)
- `VoucherTransaction` (снапшот долгов на момент продажи: `merchantDebt`,
  `kassaDebt`, `vendorDebt`; статус `PENDING|COMPLETED|CANCELLED`)
- `MerchantPayment`, `VendorPayment` (выплаты, с `balanceBefore`/
  `balanceAfter`)

### Касса (`Kassa`) и мультипровайдерная оплата
- `Kassa` — юрлицо/касса. **Хранит платёжные реквизиты как поля таблицы**, не
  только в `.env`: `clickMerchantId/ServiceId/SecretKey/MerchantUserId/Env`,
  `paymeMerchantId/Key/TestKey/Env`, `alifTokenProduction/Sandbox`,
  `alifSecretKeyProduction/Sandbox`, `alifEnv`. Также агрегаты
  `totalReceivedClick/Payme/Alif`, `balance`.
- `KassaPayment` (выплата из кассы вендору/мерчанту/платформе, с
  `balanceBefore/After`)
- `KassaAdjustment` (ручная корректировка долга)

### QR-платежи (отдельный блок в схеме, помечен комментарием
"QR Payment Feature")
- `MerchantProductLink` (постоянная ссылка мерчант+товар, `token` уникален)
- `QrPaymentAttempt` (`status`: `PENDING|PROCESSING|PAID|EXPIRED|FAILED`,
  `expiresAt`, `cancelReason`, привязка к `Sale` и `Kassa`)

### Telegram-уведомления
- `TelegramBot` (хранит `token` как поле таблицы), `StoreTelegramBot`
  (many-to-many со `Store`)

## Миграции

28 файлов в `prisma/migrations/`, диапазон дат по именам каталогов — от
`20250211120000_add_refresh_token` до `20260730232900_
lower_vendor_default_commission`. Применяются стандартным механизмом Prisma
Migrate (папки с `migration.sql` + `migration_lock.toml`); отдельно в корне
`activation-system/` лежит `manual_migration_qr_payment.sql` — не проверено,
применялась ли она вручную в обход Prisma Migrate или это черновик/резервная
копия одной из миграций (см. OPEN_QUESTIONS.md).

## Дампы и бэкапы (обнаружены, не проверялось содержимое)

- `/home/admin1/posa/activation-system/posa.sql` — полный дамп схемы/данных
  (упоминается в `migration.md` как исходный дамп для переноса на новый
  сервер).
- `/home/admin1/posa/db-backups/activation_system_20260731_152434_
  pre_commission_fix.sql`
- `/home/admin1/posa/backups/backup_before_kassa_20260228_1122.dump`
- `/home/admin1/posa/activation-system/db-backups/
  pre_test_data_cleanup_20260730_182431.dump`

Имена файлов указывают на ручной процесс "бэкап перед рискованной миграцией"
(commission fix, добавление Kassa, чистка тестовых данных) — сам процесс
(кто делал, по какому регламенту) не задокументирован отдельно, только имена
файлов.

## Подключение к БД в рантайме (подтверждено `docker ps`)

Контейнер `activation-db` (`postgres:16`) на момент проверки (2026-09-05)
проброшен как `0.0.0.0:7440->5432/tcp` — то есть доступен снаружи хоста на
всех интерфейсах, не только `127.0.0.1`. Это совпадает с находкой из прошлого
security-аудита (2026-08-01, см. память агента) — на 2026-09-05 не устранено.
Не изменялось и не проверялось глубже в рамках этой задачи (см.
[STATUS.md](STATUS.md), [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).
