# DATABASE — tgkeysbot

Последнее обновление: 2026-09-05 (TASK-0001). Источник — `core/models.py`,
`core/migrations/*.py` в `/home/admin1/tgkeysbot/`. Содержимое самой БД не
читалось (может содержать реальные Telegram ID партнёров, их баланс и
выданные лицензионные ключи) — прочитаны только имена таблиц.

## СУБД

**Продакшн:** PostgreSQL 16 (`postgres:16-alpine`, контейнер `db`,
volume `postgres_data`), доступ через Django ORM по `DATABASE_URL`
(подтверждено `docker exec tgkeysbot_bot_1 env` — `DATABASE_URL=postgres:
//keysbot@db:5432/keysbot`, пароль замаскирован при проверке).

**Без `DATABASE_URL`** (например, локальный запуск `manage.py` вне
Docker) Django откатывается на `db.sqlite3` в корне проекта —
это файл со схемой Django (таблицы `core_partner`, `core_product` и т.д.,
подтверждено чтением списка таблиц), но он не используется работающими
контейнерами.

Миграции — стандартные Django-миграции, 3 файла: `0001_initial` (2025-12-
20), `0002_partner_allowed_products_instruction` (2026-03-25),
`0003_product_delivery_source_product_drweb_sku` (2026-03-25). Состояние
БД относительно применённых миграций внутри контейнера отдельно не
сверялось (команда `migrate` выполняется автоматически при каждом старте
`web`, согласно `docker-compose.yml`).

## Модели (`core/models.py`)

### `Partner` (партнёр/дилер)

| Поле | Тип | Примечание |
|---|---|---|
| `telegram_user_id` | BigInteger, unique | идентификатор в Telegram |
| `username`, `name` | CharField, nullable | |
| `status` | CharField, choices `ACTIVE`/`BLOCKED` | доступ к боту только при `ACTIVE` |
| `balance` | Integer, default 0 | в сумах, может уходить в минус |
| `negative_limit` | Integer, default 500000 | сум, порог для уведомления о долге |
| `debt_limit_notified` / `debt_limit_notified_at` | Boolean / DateTime | однократный флаг уведомления |
| `allowed_products` | ManyToMany → `Product`, blank | пусто = доступ ко всем товарам |
| `created_at` | DateTime, auto | |

### `Product` (товар)

| Поле | Тип | Примечание |
|---|---|---|
| `public_title`, `name` | CharField | публичное/внутреннее название |
| `price` | Integer | сум |
| `source_product_code` | CharField(50) | код для команды боту-источнику |
| `command_pattern` | CharField, default `/{code}#{qty}` | объявлен, но фактическое формирование команды в `bot/main.py` захардкожено как `f"/{product.source_product_code}#{qty}"`, само поле `command_pattern` в этом месте кода не читается — не выяснено, используется ли оно где-то ещё (см. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)) |
| `is_active` | Boolean, default True | |
| `delivery_source` | CharField, choices `telegram_bot`/`drweb` (добавлено в 0003) | |
| `drweb_sku` | CharField, blank/null (добавлено в 0003) | используется вместо `source_product_code`, если `delivery_source=drweb` |
| `created_at` | DateTime, auto | |

### `Order` (заказ: покупка или активация)

| Поле | Тип | Примечание |
|---|---|---|
| `partner` | FK → `Partner`, CASCADE | |
| `product` | FK → `Product`, SET_NULL, nullable | пусто для активаций |
| `order_type` | CharField, choices `PURCHASE`/`ACTIVATION` | |
| `qty`, `total_price` | Integer, nullable/default 0 | |
| `status` | CharField, choices (10 значений — см. ниже) | |
| `source_command_text` | Text, nullable | команда, отправленная источнику |
| `installation_id` | Text, nullable | для активаций |
| `extracted_key`, `extracted_date`, `confirmation_id` | Text/CharField, nullable | результат парсинга |
| `source_raw_final_message` | Text, nullable | объявлено в модели; в актуальном коде `bot/main.py` **не заполняется** (заполняется только в неподключённом `transport/tasks.py`) |
| `error_message` | Text, nullable | |
| `created_at` / `updated_at` | DateTime | |

`OrderStatus`: `CREATED`, `QUEUED`, `SENT`, `WAITING_FINAL`, `DELIVERED`,
`FAILED_KEY_BLOCKED`, `FAILED_PARSE`, `FAILED_SOURCE`, `TIMEOUT`,
`CANCELLED`. В актуальном коде `bot/main.py` реально проставляются только
`SENT`, `DELIVERED`, `FAILED_PARSE`, `FAILED_SOURCE`, `FAILED_KEY_BLOCKED`
— остальные значения enum (`CREATED`, `QUEUED`, `WAITING_FINAL`, `TIMEOUT`,
`CANCELLED`) объявлены, но не используются нигде в `bot/main.py`
(`TIMEOUT` используется только в неподключённом `transport/tasks.py`).

### `BalanceLedger` (движение баланса)

FK на `Partner` (CASCADE) и опционально на `Order` (SET_NULL). `type`:
`DEBIT`/`TOPUP`/`MANUAL_ADJUST` (свободный список кортежей, не
`TextChoices`, в отличие от остальных enum-полей). `amount` — Integer со
знаком, `comment`, `created_at`.

### `SourceAccount` (Pyrogram-аккаунт источника)

Поля `name`, `api_id`, `api_hash`, `session_string` (хранится в БД в
открытом виде как обычный `TextField`), `status`, `is_active`,
`updated_at`. **Определена и редактируема в админке, но не читается
кодом `bot/main.py`** — реальный Pyrogram-клиент в `bot/main.py`
конфигурируется напрямую из `settings.SESSION_STRING`/`API_ID`/`API_HASH`
(из `.env`), а не из этой таблицы. См. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

### `Settings` (глобальные настройки, key-value)

`key` (unique), `value` (Text), `description`. Читается только в
неподключённом `transport/tasks.py` (`get_activation_price()`, ключ
`activation_price`, дефолт 2000 сум при отсутствии записи) — в актуальном
`bot/main.py` цена активации захардкожена как `0` и эта таблица не
запрашивается.

### `Instruction` (инструкция к товару)

One-to-one к `Product` (CASCADE). `text` (Text), `image` (ImageField,
`upload_to='instructions/'`, blank/null), `created_at`/`updated_at`.
Отображается партнёру в разделе "Инструкции" бота.

## Данные не в основной БД

- `core/templates/admin/core/partner/*.html` — Django-шаблоны страницы
  ручной корректировки баланса в админке (не данные, но специфичная для
  `Partner` кастомизация вне стандартной модели/формы).
- `media/instructions/` — каталог для изображений `Instruction.image`; на
  момент проверки (2026-09-05) пуст (файлов не найдено).

## Не проверялось

- Фактическое количество записей в продакшн-таблицах (партнёры, товары,
  заказы) — не запрашивалось намеренно, чтобы не читать потенциально
  чувствительные бизнес-данные.
- Соответствие фактически применённых миграций в контейнере `web` и
  файлов в `core/migrations/` (не запускался `showmigrations`).
