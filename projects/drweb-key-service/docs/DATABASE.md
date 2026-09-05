# DATABASE — drweb-key-service

Последнее обновление: 2026-09-05 (TASK-0001). Источник — прямое чтение
`app/models.py` (SQLAlchemy) и `core/models.py` (Django, `managed=False`,
описывает те же таблицы). СУБД — SQLite, единый файл
`data/drweb_keys.db` (в контейнере — `/app/data/drweb_keys.db`, задаётся
`DATABASE_URL`).

## Таблицы (2 шт., по факту моделей)

### `product_mapping` (модель `ProductMapping`)

Каталог товаров/SKU, из которого сервис резолвит запрос на генерацию ключа.

| Поле | Тип | Примечание |
|---|---|---|
| `sku` | String, primary key | Локальный артикул, например `LHW-BK-12M-1-A3` |
| `product_id` | String | Код продукта в Dr.Web, например `624` |
| `period` | String | Срок действия, например `12m` |
| `description` | String | Описание для админки |
| `extra_params` | Text (nullable) | JSON-строка доп. параметров (`computers`, `amount`, `discount`) |
| `full_url` | Text (nullable) | Шаблон URL с плейсхолдерами `LOGIN`/`PWD`/`STRING` (`comment`, `order_item_id`) |

Первичное наполнение — `app/products.json` (7 записей на момент проверки),
синхронизируется в таблицу при каждом старте FastAPI (upsert по `sku`).
Дальнейшее редактирование — через Django Admin.

### `key_logs` (модель `KeyLog`)

История каждой попытки генерации ключа (успешной и неуспешной).

| Поле | Тип | Примечание |
|---|---|---|
| `id` | String, primary key | `uuid.uuid4()` по умолчанию |
| `sku` | String, indexed | |
| `product_id` | String | Скопировано из `ProductMapping` на момент запроса |
| `period` | String | |
| `serial` | String (nullable) | Выданный ключ; `None` при ошибке |
| `source` | String | Имя сервиса-потребителя, передаётся клиентом в запросе |
| `status` | String | `success` / `error` |
| `error_msg` | Text (nullable) | Текст ошибки от Dr.Web или исключения |
| `created_at` | DateTime | `datetime.utcnow` по умолчанию |
| `testing` | Boolean | Признак тестового (не боевого) ключа, определяется `DRWEB_TESTING` в момент запроса |

В Django Admin (`core/admin.py`) эта модель доступна только для чтения
(`has_add_permission` возвращает `False`, все поля — `readonly_fields`).

## Кто управляет схемой

Обе таблицы физически создаёт SQLAlchemy (`app/database.py:init_db()` →
`Base.metadata.create_all`), вызывается при старте FastAPI-процесса
(`lifespan` в `app/main.py`). Django видит те же таблицы через модели с
`Meta.managed = False` — не создаёт и не мигрирует их сам. Отдельного
инструмента миграций (Alembic и т.п.) для этих двух таблиц не найдено — при
изменении схемы моделей потребуется ручное вмешательство (не
задокументировано в проекте).

Собственные таблицы Django (`auth_user`, `django_session`,
`django_migrations` и т.д.) создаются стандартным `python manage.py migrate`
(команда прописана в `docker-compose.yml` для сервиса `drweb-admin`) в том
же файле SQLite — Django-миграции для встроенных `django.contrib.*`
приложений присутствуют внутри самого пакета Django, отдельных файлов в
проекте для них нет.

## Расположение и права на диске (подтверждено `stat`/`docker-compose.yml`)

- Хостовой каталог `data/` смонтирован в контейнеры как `./data:/app/data`
  (общий volume для обоих сервисов).
- На момент проверки (2026-09-05) `data/` имеет права `777`
  (`drwxrwxrwx`), файл `drweb_keys.db` внутри принадлежит `root:root`
  (создан процессом внутри контейнера).
- `data/` и `*.db` перечислены в `.gitignore` — файл базы не отслеживается
  git.

## Не проверялось

- Содержимое `data/drweb_keys.db` (реальные записи, объём накопленных
  логов) — не открывалось.
- Процесс резервного копирования БД — упоминаний бэкапов (в отличие от,
  например, POSA) в проекте не найдено.
