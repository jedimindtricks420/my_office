# API — keyspro_auto_sender

Последнее обновление: 2026-09-05 (TASK-0001). Источник —
`app/main.py`, `app/dashboard/router.py` в
`/home/admin1/keyspro_auto_sender/`.

## Публичное API (бизнес)

### `GET /health`

Без аутентификации. Возвращает `{"status": "ok"}`. Используется
Docker healthcheck.

### `POST /api/v1/process-order`

Webhook от WooCommerce. Без аутентификации/проверки подписи на уровне
кода (см. ниже "Аутентификация").

- Если `Content-Type: application/x-www-form-urlencoded` — трактуется как
  WooCommerce webhook ping, возвращает `200 OK` с текстовым телом, никакой
  обработки не происходит.
- Если `Content-Type` не `application/json` — `415 Unsupported Media Type`.
- Тело должно соответствовать Pydantic-схеме `OrderPayload`:
  - `id: int` — ID заказа WooCommerce.
  - `status: str` — обрабатываются только заказы со `status ==
    "completed"`; остальные — `200 OK` с текстом об игнорировании.
  - `billing: {first_name?, last_name?, email (required, EmailStr),
    phone?}`.
  - `line_items: [{id, name, product_id, quantity, sku, price}]`.
  - `customer_note?: str`.
- При успешной валидации заказ ставится в фоновую задачу
  (`BackgroundTasks`) и эндпоинт сразу отвечает `202 Accepted
  {"status": "accepted", "order_id": ...}` — обработка (получение ключей,
  отправка email) происходит асинхронно, результат клиенту webhook'а не
  возвращается.
- Ошибки: `400` — невалидный JSON; `422` — не прошла Pydantic-валидация
  (тело ответа включает детали ошибок); `500` — необработанное исключение.

## Аутентификация

- **Webhook (`/api/v1/process-order`):** в конфигурации объявлено поле
  `WEBHOOK_SECRET`, и README проекта описывает его как обязательное для
  сверки с настройками WooCommerce, но **код эндпоинта не проверяет
  никакой секрет или подпись** — подтверждено чтением `app/main.py` и
  `grep -rn WEBHOOK_SECRET app/` (единственное вхождение — объявление поля
  в `Settings`). Любой синтаксически валидный JSON будет принят и
  обработан. См. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).
- **Дашборд (`/dashboard/*`):** HTTP Basic Auth
  (`app/core/security.py::authenticate_user`), сверка через
  `secrets.compare_digest` с `DASHBOARD_USERNAME`/`DASHBOARD_PASSWORD` из
  `.env` (в коде есть дефолты `admin`/`supersecret`, но в `.env` хоста эти
  переменные заданы явно — фактические значения не проверялись).

## Дашборд (веб-интерфейс + несколько API-подобных маршрутов)

Всё под префиксом `/dashboard`, защищено HTTP Basic (см. выше).

| Маршрут | Метод | Назначение |
|---|---|---|
| `/dashboard/` | GET | Обзорная страница: статус Pyrogram/бота/SMTP, сводная статистика |
| `/dashboard/orders` | GET | Список заказов с пагинацией (`page`, `per_page`) |
| `/dashboard/orders/{order_id}` | GET | Детали заказа по `woo_order_id` + список выданных ключей |
| `/dashboard/products` | GET | Список товаров из `product_config.json` |
| `/dashboard/products/new` | GET | Форма создания товара |
| `/dashboard/products/{sku}/edit` | GET | Форма редактирования товара |
| `/dashboard/products/save` | POST | Сохранение товара (перезаписывает `product_config.json` целиком) |
| `/dashboard/products/{sku}/delete` | POST | Удаление товара из `product_config.json` |
| `/dashboard/settings` | GET | Страница настроек: статус системы + конфигурация (включая `admin_telegram_id`, `target_bot` — не секреты, но идентификаторы) |
| `/dashboard/api/stats` | GET | JSON-статистика (заказы/ключи за сегодня/неделю, success rate) |
| `/dashboard/templates` | GET | Список email-шаблонов (`app/templates/*.html`) с размером и датой изменения |
| `/dashboard/templates/{template_name}` | GET | Просмотр исходного HTML шаблона |
| `/dashboard/templates/{template_name}/preview` | GET | Рендер шаблона с тестовыми данными |

## Внешние API, которые вызывает сервис

- **Telegram (Pyrogram)** — `send_message`/`get_chat_history` к боту
  `TARGET_BOT_USERNAME` (`@MicrosoftLeo_Bot`) от лица пользовательского
  аккаунта.
- **Telegram Bot API** (`python-telegram-bot`) — `send_message` админу
  (`ADMIN_TELEGRAM_ID`) от лица `CLIENT_BOT_TOKEN`.
- **SMTP** — `SMTP_HOST:SMTP_PORT` (README указывает `mail.keyspro.uz:465`,
  значения `.env` не проверялись).
- **Dr.Web API** — `POST {DRWEB_API_URL}` с `Authorization: Bearer
  {DRWEB_API_KEY}`, телом `{"sku": <command>, "source":
  "keyspro_auto_sender"}`, ожидаемый ответ — JSON с полем `serial`.
  Конкретный URL и природа этого API (внутренний сервис
  `drweb-key-service` или сторонний) не подтверждены в рамках этой задачи.
