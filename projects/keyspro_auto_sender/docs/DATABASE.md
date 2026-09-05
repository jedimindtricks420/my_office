# DATABASE — keyspro_auto_sender

Последнее обновление: 2026-09-05 (TASK-0001). Источник —
`app/database/models.py`, `app/database/database.py` в
`/home/admin1/keyspro_auto_sender/`. Содержимое самой БД не читалось (может
содержать реальные email клиентов и выданные лицензионные ключи).

## СУБД

SQLite, доступ через SQLAlchemy 2.x (async ORM, `DeclarativeBase`) +
`aiosqlite`. Строка подключения: `sqlite+aiosqlite:///keyspro.db`
(относительный путь — файл создаётся в рабочей директории процесса, внутри
контейнера это `/app/keyspro.db`). Схема создаётся автоматически при
старте приложения (`Base.metadata.create_all`), формальных миграций
(Alembic и т.п.) нет.

## Модели

### `Order` (`orders`)

| Поле | Тип | Примечание |
|---|---|---|
| `id` | int, PK, autoincrement | |
| `woo_order_id` | int, unique, indexed | ID заказа в WooCommerce |
| `customer_email` | string(255) | |
| `customer_name` | string(255), nullable | |
| `status` | string(50) | `processing` / `completed` / `partial` / `failed` (значения из кода `processor.py`, не enum на уровне БД) |
| `email_sent` | bool, default False | |
| `admin_notified` | bool, default False | |
| `created_at` | datetime | |
| `updated_at` | datetime, onupdate | |

Связь: `issued_keys` — one-to-many к `IssuedKey`, `cascade="all,
delete-orphan"`.

### `IssuedKey` (`issued_keys`)

| Поле | Тип | Примечание |
|---|---|---|
| `id` | int, PK | |
| `order_id` | FK → `orders.id`, `ondelete="CASCADE"` | |
| `sku` | string(100), indexed | |
| `product_description` | string(255) | |
| `component_name` | string(255) | |
| `key` | string(50) | сам лицензионный ключ, хранится в открытом виде |
| `created_at` | datetime | |

### `Product` (`products`) — определена, но не используется

| Поле | Тип | Примечание |
|---|---|---|
| `id` | int, PK | |
| `sku` | string(100), unique, indexed | |
| `description` | string(255) | |
| `template_name` | string(100), default `simple_product_template.html` | |
| `components` | JSON | |
| `is_active` | bool, default True | |
| `created_at` / `updated_at` | datetime | |

Комментарий в коде: "replaces product_config.json". Фактически таблица
создаётся в БД, но **ни разу не читается и не пишется** ни в
`processor.py`, ни в `dashboard/router.py` — весь каталог товаров живёт в
файле `product_config.json`. `Product` импортирован в
`dashboard/router.py`, но используется там только как строка сообщения
об ошибке ("Product not found"), не как ORM-запрос. См.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

## Персистентность (риск)

`docker-compose.yml` монтирует как bind mounts только `.env`, `app/` и
`product_config.json`. Файл `keyspro.db` в списке volume'ов отсутствует —
подтверждено `docker inspect` (нет соответствующего mount) и `docker exec
... ls -la /app/keyspro.db` (файл существует только в writable-слое
контейнера, 36 КБ на момент проверки). Простой `docker compose restart`
данные не теряет (тот же контейнер), но пересоздание контейнера
(`up --force-recreate`, удаление и повторный `up`, миграция на новый
сервер по инструкции из README) приведёт к полной потере истории заказов и
выданных ключей.

## Каталог товаров (не в БД)

`product_config.json` в корне проекта — де-факто основной источник данных
о товарах, живёт вне SQL-схемы. Структура: словарь `{sku: {description,
template_name, components: [{name, command, source?}]}}`. Подробности —
[STACK.md](STACK.md) и [OVERVIEW.md](OVERVIEW.md).
