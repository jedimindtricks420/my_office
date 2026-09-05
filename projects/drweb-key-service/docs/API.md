# API — drweb-key-service

Последнее обновление: 2026-09-05 (TASK-0001). Извлечено напрямую из
`app/main.py`. Формальной OpenAPI-спецификации в репозитории не найдено
(хотя FastAPI генерирует её автоматически по умолчанию на `/docs`/
`/openapi.json` — не проверялось, доступны ли эти пути публично, см.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)). Дополнительно есть руководство для
внешних потребителей — `project_docs/API_INTEGRATION.md` (описывает тот же
`/generate`-эндпоинт с примером на `aiohttp`; базовый URL в документе указан
как `http://4.240.102.156:8001/drweb`).

## Аутентификация

Статический токен в заголовке `Authorization`, сравнивается с `API_KEY` из
`.env`. Поддерживается как `Authorization: <token>`, так и
`Authorization: Bearer <token>` (префикс `"Bearer "` обрезается вручную в
`get_api_key()`). Сравнение — обычное строковое `!=`, не constant-time.
Действует на все эндпоинты, кроме `/health`.

## `GET /health`, `GET /drweb/health`

Без авторизации. Возвращает `{"status": "ok"}`. Для проверки
работоспособности процесса.

## `POST /drweb/api/v1/generate`

Требует авторизации. Основной бизнес-эндпоинт.

**Request body:**
```json
{"sku": "LHW-BK-12M-1-A3", "source": "telegram_bot_v2"}
```

**Поведение (подтверждено кодом):**
1. Ищет `sku` в таблице `product_mapping` → `404`, если не найден.
2. Вызывает Dr.Web через `DrWebClient.generate_key(...)`, используя
   `full_url` записи и `extra_params` (JSON).
3. Пишет запись в `key_logs` независимо от результата (успех/ошибка).
4. При ошибке Dr.Web/сети — `500` с телом `detail` (взято из ответа
   `DrWebClient`, например `{"status": "error", "message": "..."}`).
5. При успехе — `200` с `GenerateResponse` (`serial`, `sku`, `product`,
   `period`, `testing`, `created_at`).

## `GET /drweb/api/v1/products`

Требует авторизации. Возвращает список всех `ProductMapping`
(`sku`, `product_id`, `period`, `description` — без `full_url`/
`extra_params` в ответе, схема `ProductSchema` их не включает).

## `GET /drweb/api/v1/logs?limit=100`

Требует авторизации. Возвращает последние записи `key_logs`, отсортированные
по `created_at DESC`, `limit` по умолчанию 100 (без верхнего ограничения на
значение параметра — не проверялось поведение при очень большом `limit`).

## Django Admin (порт 8002, отдельный процесс)

Единственный маршрут — `/admin/` (`django.contrib.admin.site.urls`), не
JSON-API, а серверный HTML-интерфейс с логином/паролем. Даёт CRUD по
`ProductMapping` и read-only просмотр `KeyLog` (см.
[ARCHITECTURE.md](ARCHITECTURE.md)).

## Не проверено в рамках этой задачи

- Доступность автосгенерированной документации FastAPI (`/docs`,
  `/redoc`, `/openapi.json`) — по умолчанию FastAPI включает их, явного
  отключения (`docs_url=None` и т.п.) в `app/main.py` не найдено, то есть,
  вероятно, доступны — но фактическая проверка HTTP-запросом не
  выполнялась в рамках этой задачи (задача не подразумевала обращения к
  живому сервису по сети).
- Поведение при некорректном/отсутствующем `.env` (значения по умолчанию —
  см. код — не тестировались запуском).
