# Database — 1PDF

Снимок на 2026-09-05. Источник — `prisma/schema.prisma`,
`prisma/migrations/*/migration.sql`, и прямая интроспекция живой БД через
`docker exec 1pdf-postgres psql -U onepdf -d onepdf` (только структура —
`\dt`, `\d <table>`, `SELECT migration_name, finished_at FROM
_prisma_migrations` — значения данных не читались).

## Подключение

PostgreSQL 16, база `onepdf`, пользователь `onepdf`, доступна только на
`127.0.0.1:5435` (контейнер `1pdf-postgres`). `DATABASE_URL` передаётся в
контейнер `admin` через `docker-compose.yml` (пароль — из `.env`, значение не
читалось).

## Модели (`prisma/schema.prisma`, 11 моделей, 5 enum)

| Модель | Назначение |
|---|---|
| `User` | Пользователь портала: email/пароль, `is_verified`, `interface_language` (enum `RU`/`UZ`/`EN`), связи на `Subscription`, `Payment[]`, `PaymentAttempt[]`, `Document[]`, `DocumentVersion[]` |
| `Subscription` | 1:1 к `User`, `plan` (`FREE`/`PRO`), `valid_until` |
| `Payment` | История успешных/иных платежей (аудиторская запись, создаётся при фулфилменте) |
| `PaymentAttempt` | Одна попытка оплаты (создаётся до редиректа на провайдера); поле `order_code` — короткий человекочитаемый код заказа |
| `Voucher` | Код активации PRO, серия (`series`), срок в днях (`days_granted`), одноразовый (`is_used`) |
| `Document` | Документ пользователя, `current_version_id` указывает на активную `DocumentVersion` |
| `DocumentVersion` | Неизменяемая версия файла: `storage_key`, `size_bytes`, `sha256`, `status` (`UPLOADING`/`READY`/`FAILED`), `source_version_id` для цепочки версий |
| `Admin` | Администратор (`login`/`password_hash`/`role`: `ADMIN`/`SUPERADMIN`) |
| `LandingConfig` | SEO-конфиг лендинга по языку (составной PK `id`+`lang`) |
| `LandingSection` | Секция контента лендинга по языку (составной PK `section_key`+`lang`) |
| `PaymentConfig` | Единственная запись (`id: "default"`) с реквизитами Payme/Click/Alif — см. ниже про отсутствующую миграцию |

## Миграции (`prisma/migrations/`, 5 файлов)

| Миграция | Содержание |
|---|---|
| `20260412135220_init_ecosystem` | Начальная схема: `User`, `Subscription`, `Payment`, `PaymentAttempt` (с полем `external_id`, позже переименованным/заменённым), `Voucher`, `Admin`, `LandingConfig`, `LandingSection` |
| `20260630000000_add_order_code` | `PaymentAttempt.order_code` (уникальный) |
| `20260726000000_add_alif_payment_config` | `ALTER TABLE "PaymentConfig" ADD COLUMN ...` (5 колонок Alif) — **см. открытый вопрос ниже: сама таблица `PaymentConfig` этой или любой другой миграцией не создаётся** |
| `20260730120000_add_voucher_series` | `Voucher.series` (аддитивно, дефолт `'1PDF'`) |
| `20260801000000_add_editor_documents_and_language` | `User.interface_language`/`language_updated_at`, новые таблицы `Document`/`DocumentVersion` |

`_prisma_migrations` в живой БД содержит ровно эти 5 записей
(`finished_at` от 2026-04-16 до 2026-08-01) — миграции применялись
последовательно и совпадают с файлами на диске.

## Подтверждённый дрейф схемы (не предположение — проверено `\d`)

Следующие объекты **существуют в реальной БД, но не создаются ни одним
файлом `migration.sql`**:

- Таблица `PaymentConfig` целиком (только 5 полей Alif добавлены миграцией
  `20260726000000`; сама таблица — нет).
- Колонки `PaymentAttempt`: `amount_tiyin` (bigint), `cancel_reason`,
  `cancel_time`, `external_payment_id`, `paid_at`, `processing_status`.
  Изначальная миграция создавала для `PaymentAttempt` колонку `external_id`
  (не `external_payment_id`) — в текущей живой схеме `external_id`
  отсутствует, вместо неё `external_payment_id` с отдельным уникальным
  индексом `PaymentAttempt_external_payment_id_key`.

Живая схема **полностью соответствует** `prisma/schema.prisma` (проверено
построчно через `\d "PaymentAttempt"` и `\d "PaymentConfig"`) — то есть
приложение работает с ожидаемой схемой. Расхождение — только в истории
миграций, не в фактическом состоянии. Не установлено, как эти объекты были
созданы (`prisma db push`, ручной SQL, миграция, применённая вручную и не
попавшая в `git`, либо иной способ) — см.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

## Резервные копии

Единственный найденный дамп — `backups/pre_freepdf_20260801_1040.dump`
(46 КБ, перед миграцией `add_editor_documents_and_language`) и более старые
ручные дампы в `extraConfigs/backup/db/*.sql` (апрель 2026). Автоматического
расписания резервного копирования (cron/systemd) для БД или
`temp_uploads/` в рамках проекта не найдено — см.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).
