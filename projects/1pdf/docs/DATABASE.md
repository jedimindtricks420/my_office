# Database — 1PDF

Полностью переписано 2026-09-08 (`TASK-0002`). `prisma/schema.prisma` (196
строк) прочитан целиком; все 5 файлов `migration.sql` прочитаны целиком;
факты о дрейфе схемы перепроверены read-only SQL-запросами
(`docker exec 1pdf-postgres psql -U onepdf -d onepdf -c '\d "Table"'`,
`SELECT migration_name FROM _prisma_migrations`) — никаких `CREATE`/`ALTER`/
`INSERT`/`UPDATE`/`DELETE` не выполнялось, только `\d` и `SELECT`.

Одна база данных `onepdf` (PostgreSQL 16, `postgres:16-alpine`).

## Модели (15) — `prisma/schema.prisma`

- **User** (10-27) — `id`, `email` (unique), `password?` (nullable — учтено
  под голосовой/OTP-регистрацию без пароля? не проверялось; в реальности
  `verify-signup` всегда передаёт хешированный пароль, так что причина
  nullable не выяснена, см. OPEN_QUESTIONS.md), `telegram_id?` (unique, не
  встречено ни одного места кода, которое его заполняет — см. GAPS.md),
  `notes?`, `is_verified` (default `false`), `verification_code?`,
  `interface_language` (enum `InterfaceLanguage`, default `RU`),
  `language_updated_at?`. → `payments[]`, `attempts[]`, `subscription?`,
  `documents[]` (`OwnedDocuments`), `document_versions[]`
  (`CreatedDocumentVersions`).
- **Subscription** (29-37) — 1:1 с `User`, `plan` (enum `PlanType`:
  FREE/PRO), `valid_until?`.
- **Payment** (39-53) — запись факта успешной оплаты (история): `amount`,
  `currency` (default `"UZS"`), `provider String` (не enum — значения
  `PAYME`/`CLICK`/`ALIF`/`VOUCHER` контролируются только кодом), `status`
  (enum `PaymentStatus`), `external_id?` (unique), `notes?`.
- **PaymentAttempt** (55-70) — попытка оплаты (до подтверждения):
  `order_code?` (unique), `amount`, `provider String`, `status`,
  `amount_tiyin BigInt?`, `cancel_reason?`, `cancel_time?`,
  `external_payment_id?` (unique), `paid_at?`, `processing_status?`.
  **См. раздел «Дрейф схемы» ниже — 6 из этих полей не создаются ни одной
  миграцией.**
- **Voucher** (72-83) — `code` (unique), `series` (default `"1PDF"`),
  `days_granted`, `is_used`, `used_by_id?` (обычная строка, **не foreign
  key** — не проверяется ссылочная целостность на `User.id`), `activated_at?`.
- **Document** (85-98) — `owner_id → User` (`onDelete: Restrict` — нельзя
  удалить пользователя, у которого остались документы, без каскада),
  `name`, `mime_type` (default `"application/pdf"`), `current_version_id?`
  (unique, → `DocumentVersion`, `onDelete: SetNull`).
- **DocumentVersion** (100-117) — `document_id → Document`
  (`onDelete: Restrict`), `version_number Int`, `storage_key` (unique —
  путь к файлу на диске), `size_bytes BigInt`, `sha256`,
  `source_version_id?` (обычная строка, не FK — ссылка на версию, от
  которой сделан этот сейв, для истории редактирования, не проверяется
  ссылочно), `created_by_id → User` (`onDelete: Restrict`), `status` (enum
  `DocumentVersionStatus`: UPLOADING/READY/FAILED — код всегда создаёт со
  статусом `READY` сразу, `UPLOADING`/`FAILED` в схеме объявлены, но не
  встречены как присваиваемые в `editorApi.ts`, см. GAPS.md);
  `@@unique([document_id, version_number])`.
- **Admin** (131-139) — `login` (unique), `password_hash`, `role` (enum
  `AdminRole`: SUPERADMIN/ADMIN, default `ADMIN`).
- **LandingConfig** (141-150) — составной ключ `@@id([id, lang])`, `id`
  всегда `"config"` по факту использования — SEO-метаданные лендинга на
  язык.
- **LandingSection** (152-162) — составной ключ `@@id([section_key, lang])`,
  `content Json?` — блоки лендинга (hero, how_it_works, pricing, faq по
  данным `admin/seed.ts`).
- **PaymentConfig** (164-180) — **одна строка**, `id: "default"`:
  `payme_merchant_id/key/test_key/env`, `click_merchant_id/service_id/
  secret_key/env`, `alif_token_production/sandbox`,
  `alif_secret_key_production/sandbox`, `alif_env`. Секреты платёжных
  провайдеров хранятся здесь, в БД, редактируются из `admin.html`
  (`/api/admin/payment-settings`).

### Enum'ы (4)

`InterfaceLanguage` (RU, UZ, EN) · `DocumentVersionStatus` (UPLOADING,
READY, FAILED) · `PlanType` (FREE, PRO) · `PaymentStatus` (PENDING,
SUCCESS, FAILED) · `AdminRole` (SUPERADMIN, ADMIN).

## Миграции — формат правильный, содержимое неполное

В отличие от других проектов my_office (например, Contador — сырые `.sql`
без структуры), здесь **используется настоящий формат Prisma**:
`prisma/migrations/migration_lock.toml` + 5 папок
`<timestamp>_<name>/migration.sql`:

```
20260412135220_init_ecosystem                         (148 строк, базовая схема)
20260630000000_add_order_code                         (2 строки: PaymentAttempt.order_code)
20260726000000_add_alif_payment_config                (5 строк: 5 колонок alif_* в PaymentConfig)
20260730120000_add_voucher_series                      (Voucher.series)
20260801000000_add_editor_documents_and_language        (Document/DocumentVersion/interface_language)
```

`npm run migrate` = `prisma migrate deploy`, выполняется автоматически при
каждом старте контейнера (`Dockerfile.admin.v2:22`) — реальный,
production-корректный процесс, в отличие от ручного `psql`-подхода в других
проектах.

## ⚠️ Подтверждённый дрейф схемы (перепроверено read-only SQL, не только сравнением файлов)

1. **`PaymentConfig` — базовая таблица не создаётся ни одной миграцией.**
   `grep -l "PaymentConfig" prisma/migrations/*/migration.sql` находит
   только `20260726000000_add_alif_payment_config/migration.sql`, и там —
   **только** 5 `ALTER TABLE "PaymentConfig" ADD COLUMN alif_*` (файл
   прочитан целиком, `migration.sql:1-5`). Ни один файл не содержит `CREATE
   TABLE "PaymentConfig"`. Тем не менее таблица реально существует в БД
   (подтверждено `\d "PaymentConfig"`, 2026-09-08) со всеми базовыми полями
   (`id`, `payme_*`, `click_*`, `updated_at`) — создана вне отслеживаемой
   истории миграций (вероятно, через `prisma db push` в какой-то момент
   разработки).
2. **`PaymentAttempt` — 6 полей не создаются ни одной миграцией, при этом
   реально существуют в БД.** Исходная миграция `init_ecosystem` создаёт
   `PaymentAttempt` только с колонкой `external_id` (`migration.sql:51-61`,
   уникальный индекс `PaymentAttempt_external_id_key`,
   `migration.sql:133`). Ни один из 5 файлов миграций не содержит
   `amount_tiyin`, `cancel_reason`, `cancel_time`, `paid_at`,
   `processing_status`, и — что важнее — не содержит переименования
   `external_id` → `external_payment_id`. Тем не менее реальная таблица
   (подтверждено `\d "PaymentAttempt"`, 2026-09-08) содержит именно
   `external_payment_id` (с уникальным индексом
   `PaymentAttempt_external_payment_id_key`) и все 5 остальных полей — то
   есть колонка была переименована/добавлена вне миграций, а не только
   добавлена. `schema.prisma:55-70` описывает именно текущее (дрейфовавшее)
   состояние — то есть Prisma Client, сгенерированный из схемы, работает с
   реальной БД корректно, но **история миграций не воспроизводит текущую
   структуру**: `prisma migrate deploy` на пустой базе не создаст ни базовую
   `PaymentConfig`, ни рабочий `PaymentAttempt` в его нынешнем виде.
3. `_prisma_migrations` (таблица учёта Prisma) содержит ровно 5 записей,
   совпадающих 1:1 с файлами на диске — то есть сам факт применения
   отслеживаемых миграций не под вопросом, под вопросом **полнота** этой
   истории относительно фактической схемы.

## Сидинг данных

`admin/seed.ts` (`npm run seed` = `tsx admin/seed.ts`) — идемпотентный
upsert: один `SUPERADMIN` (логин/пароль из `SUPERADMIN_LOGIN`/
`SUPERADMIN_PASSWORD`, дефолты `admin`/`changeme123` если не заданы — см.
[GAPS.md](GAPS.md) про риск дефолтного пароля), 2 записи `LandingConfig`
(RU/UZ), 8 записей `LandingSection` (hero/how_it_works/pricing/faq × 2
языка). Не вызывается автоматически при старте контейнера (в отличие от
`migrate deploy`) — отдельный шаг, не найдено вызова `npm run seed` ни в
`Dockerfile.admin.v2`, ни в `docker-compose.yml`.
