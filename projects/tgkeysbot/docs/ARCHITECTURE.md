# ARCHITECTURE — tgkeysbot

Последнее обновление: 2026-09-05 (TASK-0001).

## Общая схема (реально работающий путь)

```
┌──────────────┐  long polling   ┌────────────────────────────────────────┐
│ Партнёр      │ ◀─────────────▶ │ bot/main.py (aiogram Dispatcher)         │
│ (Telegram)   │                 │ процесс "bot" в Docker                   │
└──────────────┘                 │  ├─ Pyrogram-клиент (юзер-аккаунт,       │
                                  │  │  всегда подключён)                   │
                                  │  └─ Django ORM (общая БД с "web")       │
                                  └───────────┬──────────────┬──────────────┘
                                              │              │
                     delivery_source=telegram_bot     delivery_source=drweb
                                              │              │
                                              ▼              ▼
                              ┌───────────────────────┐  ┌─────────────────────────┐
                              │ @MicrosoftLeo_Bot      │  │ drweb-key-service        │
                              │ (внешний Telegram-бот, │  │ (отдельный HTTP-сервис,  │
                              │ команда → 60с ожидание  │  │ POST DRWEB_API_URL +     │
                              │ → парсинг ключа regex'ом│  │ Bearer DRWEB_API_TOKEN)  │
                              │ из истории чата)        │  │ см. ../drweb-key-service/│
                              └───────────────────────┘  └─────────────────────────┘

┌──────────────┐   HTTPS (TLS)   ┌───────────┐  proxy_pass /partner_bot/  ┌───────────────────────┐
│ Оператор     │ ──────────────▶ │  Nginx    │ ─────────────────────────▶│ Django (gunicorn:8000) │
│ (браузер)    │                 │ webhook.  │   127.0.0.1:8080           │ процесс "web" в Docker  │
└──────────────┘                 │ keyspro.uz│                            └───────────┬────────────┘
                                  └───────────┘                                        │
                                                                                        ▼
                                                                          ┌──────────────────────┐
                                                                          │ PostgreSQL 16          │
                                                                          │ контейнер "db",         │
                                                                          │ volume postgres_data    │
                                                                          └──────────────────────┘
```

## Процессы/деплой

- Три сервиса `docker-compose.yml`: `db` (PostgreSQL 16, healthcheck
  `pg_isready`), `web` (Django + gunicorn, `python manage.py migrate &&
  gunicorn config.wsgi:application`, порт контейнера 8000 опубликован на
  хосте как `8080`), `bot` (`python bot/main.py`, без порта). Оба `web` и
  `bot` собираются из одного и того же `Dockerfile` (общий образ,
  разная команда запуска), оба зависят от `db: service_healthy`.
- `web` и `bot` — независимые процессы, соединяются с одной и той же
  PostgreSQL через Django ORM; между собой не общаются напрямую (нет
  общего API/очереди) — единственная связь через общие строки в БД
  (например, `SourceAccount`, `Settings`, которые Django-админка может
  редактировать, но `bot/main.py` их не читает — см.
  [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).
- Единственный Pyrogram-клиент (`pyrogram_client` — глобальная переменная
  модуля) живёт всё время работы процесса `bot` и используется
  последовательно для всех партнёров и сценариев (покупка и активация).
  Конкурентная обработка двух заказов не сериализована явно (нет lock) —
  оба используют `asyncio.sleep(60)` и чтение одной и той же истории чата
  с ботом-источником.
- Django-админка проксируется на хосте через Nginx-виртуалхост
  `webhook.keyspro.uz`, который **разделяется с другим, не связанным
  проектом** `keyspro_auto_sender` (см.
  [`../keyspro_auto_sender/`](../keyspro_auto_sender/AGENTS.md)): корневой
  `location /` того же server-блока обслуживает `keyspro_auto_sender`, а
  `location /partner_bot/` проксирует на `tgkeysbot` (порт 8080). Подробнее
  — [DEPLOYMENT.md](DEPLOYMENT.md).

## Два независимых пути получения ключа

Выбор определяется полем `Product.delivery_source`:

1. **`telegram_bot` (по умолчанию)** — `get_key_from_source()`/
   `get_cid_from_source()` в `bot/main.py`: Pyrogram-клиент от лица
   пользовательского Telegram-аккаунта (`SESSION_STRING`) отправляет
   текстовую команду (`/{source_product_code}#{qty}` для покупки, сам
   Installation ID для активации) боту `TARGET_BOT_USERNAME`
   (`@MicrosoftLeo_Bot`), ждёт 60 секунд (`asyncio.sleep(60)`, без
   промежуточных проверок), затем читает последние 3 сообщения истории
   чата и ищет по ключевым словам ("Successful"/"Confirmation ID"/"Key
   blocked"), извлекая данные регулярными выражениями. Без повторных
   попыток при неудаче на этом уровне.
2. **`drweb`** — `bot/drweb_client.get_key_from_drweb()`: один HTTP POST
   (`aiohttp`) на `DRWEB_API_URL` с `Authorization: Bearer
   DRWEB_API_TOKEN`, телом `{"sku": <drweb_sku или source_product_code>,
   "source": DRWEB_API_SOURCE_NAME}`, ожидает JSON с полем `serial`. Без
   таймаута, заданного явно в коде (используется дефолт `aiohttp`), без
   повторных попыток.

## Второй, неподключённый путь той же логики

`transport/tasks.py` — это отдельная, самостоятельная реализация тех же
двух сценариев (`process_purchase`, `process_activation`) как задачи
Celery, с распределённой блокировкой Redis
(`lock:source_account:default`, `redis.lock`) вокруг работы с
Pyrogram-аккаунтом — то есть решающая ровно ту проблему конкурентности,
которая не решена в `bot/main.py`. Подтверждено, что этот код нерабочий в
текущем состоянии проекта:

- `celery` и `redis` (Python-пакеты) отсутствуют в `requirements.txt`.
- Нет ни одного создания `Celery(...)` приложения (`grep -rn "Celery("`
  ничего не находит) — то есть `@shared_task` не привязаны ни к какому
  приложению и не могут быть поставлены в очередь стандартным способом.
- `docker-compose.yml` не содержит ни сервиса `redis`, ни сервиса-воркера
  Celery.

`REDIS_URL` тем не менее объявлен в `config/settings.py` (дефолт
`redis://localhost:6379/0`) и присутствует как имя переменной в `.env`, но
ни один работающий сервис Redis в `docker-compose.yml` не поднимается. См.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md) о происхождении этого кода.

## Границы системы

- Внешние зависимости: Telegram (Bot API через `CLIENT_BOT_TOKEN` для
  aiogram, MTProto через Pyrogram для юзер-аккаунта), бот-источник
  `@MicrosoftLeo_Bot` (сторонний, не принадлежит этому проекту), внешний
  HTTP-сервис `drweb-key-service` (см. [AGENTS.md](../AGENTS.md)),
  PostgreSQL.
- Внутри репозитория нет очередей/кэшей, реально участвующих в текущем
  потоке (Redis/Celery объявлены, но не задействованы — см. выше). Нет
  входящих HTTP-webhook'ов от внешних систем — единственный внешний вход
  через HTTP это браузерный доступ к Django-админке.
