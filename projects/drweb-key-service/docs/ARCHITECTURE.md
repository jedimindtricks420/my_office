# ARCHITECTURE — drweb-key-service

Последнее обновление: 2026-09-05 (TASK-0001). Источник фактов — прямое
чтение кода в `/home/admin1/drweb-key-service/`.

## Общая схема

Один Docker-образ (`Dockerfile`, `python:3.11-slim`), два независимых
процесса запускаются как два сервиса `docker-compose.yml` из одного и того
же образа с разными командами:

- **`drweb-api`** — `uvicorn app.main:app --host 0.0.0.0 --port 8001`.
- **`drweb-admin`** — `python manage.py migrate`, затем создание
  Django-суперпользователя (если не существует), затем
  `python manage.py runserver 0.0.0.0:8002`.

Оба монтируют один и тот же volume `./data:/app/data` и один и тот же
`.env` (`env_file`).

## Слой FastAPI (`app/`)

- `main.py` — точка входа. `lifespan`-хук при старте вызывает `init_db()`
  (`Base.metadata.create_all`) и синхронизирует таблицу `product_mapping` из
  `app/products.json` (upsert по `sku`: создаёт отсутствующие записи,
  обновляет поля существующих). Роуты: `/health` (алиас
  `/drweb/health`), `POST /drweb/api/v1/generate`, `GET
  /drweb/api/v1/products`, `GET /drweb/api/v1/logs?limit=100` — три
  последних под `Depends(get_api_key)`.
- `get_api_key()` — читает заголовок `Authorization` через
  `APIKeyHeader(auto_error=False)`, вручную убирает префикс `"Bearer "` при
  наличии, сравнивает с `API_KEY` из `.env` оператором `!=` (обычное
  строковое сравнение, не constant-time).
- `database.py` — SQLAlchemy, **синхронный** `create_engine(DATABASE_URL,
  connect_args={"check_same_thread": False})` для SQLite, `SessionLocal` +
  `get_db()` как FastAPI dependency. Несмотря на то, что `requirements.txt`
  содержит `aiosqlite` и `databases` (асинхронные драйверы), сам файл их не
  импортирует и не использует — подключение к БД синхронное.
- `models.py` — две SQLAlchemy-модели, `Base = declarative_base()`:
  `KeyLog` (таблица `key_logs`) и `ProductMapping` (таблица
  `product_mapping`).
- `drweb_client.py` — `DrWebClient`, единственный внешний HTTP-клиент,
  строит URL из `full_url_template` записи `ProductMapping` (заменяет
  плейсхолдеры `login=LOGIN`, `pwd=PWD`, `comment=STRING`,
  `order_item_id=STRING`), либо (если шаблона нет) собирает `POST` с телом
  из параметров. Выполняет запрос через `curl_cffi.requests.AsyncSession
  (impersonate="chrome")`, парсит XML-ответ через `xmltodict`.
  Перед запросом пишет `logger.info(f"Requesting Dr.Web: url={final_url}")`
  — на момент этого лога `final_url` уже содержит подставленные `login` и
  `pwd`, то есть учётные данные Dr.Web попадают в лог процесса в открытом
  виде.
- `schemas.py` — Pydantic-схемы запроса/ответа (`GenerateRequest`,
  `GenerateResponse`, `ProductSchema`, `LogSchema`).

## Слой Django (`drweb_admin/` + `core/`)

- `drweb_admin/settings.py` — единственное Django-приложение `core`.
  `DATABASE_URL` читается из того же `.env`, что и FastAPI, и указывает на
  тот же файл SQLite. `SECRET_KEY` захардкожен в исходном коде
  (`'django-insecure-drweb-admin-panel-secret'`), `DEBUG = True`,
  `ALLOWED_HOSTS = ['*']`, `AUTH_PASSWORD_VALIDATORS = []` (валидация
  сложности пароля отключена). `ROOT_URLCONF` регистрирует единственный
  маршрут — `admin/` (`django.contrib.admin.site.urls`); собственных
  Django-view/API в проекте нет.
- `core/models.py` — Django-модели `ProductMapping` и `KeyLog`, обе с
  `Meta.managed = False` — Django не создаёт и не мигрирует эти таблицы
  (это делает SQLAlchemy на стороне FastAPI, `Base.metadata.create_all`),
  только читает/пишет в уже существующие. `core/migrations/` содержит
  только пустой `__init__.py` — файлов конкретных миграций для приложения
  `core` нет и не требуется (модели `managed=False`). Служебные таблицы
  самого Django (`auth_user`, `django_session` и т.п.) при этом создаются
  штатным `python manage.py migrate` (встроенные миграции
  `django.contrib.*`) в том же файле SQLite.
- `core/admin.py` — регистрация обеих моделей в Django Admin.
  `ProductMappingAdmin` — обычный CRUD со списком/поиском.
  `KeyLogAdmin` — `has_add_permission` возвращает `False` (логи нельзя
  создавать вручную через админку), все поля, кроме списковых/фильтров,
  помечены `readonly_fields`.

## Общая БД между двумя процессами

Оба процесса (FastAPI через SQLAlchemy, Django через свой ORM) параллельно
читают и пишут в один и тот же файл SQLite `data/drweb_keys.db` без
дополнительного координирующего слоя (нет отдельной БД-сервера, нет
блокировок на уровне приложения сверх стандартного SQLite-файлового
локинга). Подтверждено конфигурацией `DATABASE_URL` в обоих `settings.py`/
`.env` и общим volume в `docker-compose.yml`.

## Загрузка каталога товаров

Единственный встроенный механизм первоначального наполнения — `app/
products.json` (7 записей на момент проверки), синхронизируемый в БД при
каждом старте FastAPI (`lifespan` в `main.py`). Дальнейшее управление
товарами — через Django Admin (`ProductMappingAdmin`), который редактирует
ту же таблицу напрямую.
