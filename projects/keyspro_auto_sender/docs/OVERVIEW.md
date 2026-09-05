# OVERVIEW — keyspro_auto_sender

Последнее обновление: 2026-09-05 (TASK-0001). Источник — прямое чтение кода
`/home/admin1/keyspro_auto_sender/`.

## Что это

Микросервис, автоматизирующий выдачу лицензионных ключей после покупки на
e-commerce платформе KeysPro.uz (продажа лицензий Microsoft Office/Windows и
Dr.Web). При завершении заказа в WooCommerce сервис сам получает нужный
ключ и рассылает его клиенту и администратору — без участия человека в
штатном сценарии.

## Поток обработки заказа

```
WooCommerce (заказ завершён)
  → Nginx (webhook.keyspro.uz)
  → FastAPI POST /api/v1/process-order
  → валидация payload (Pydantic: OrderPayload)
  → фоновая задача (BackgroundTasks): process_order_task
       для каждого line_item → recipe из product_config.json по SKU
         для каждого component в recipe:
           source == "drweb"  → DrWebProvider (HTTP к внешнему Dr.Web API)
           иначе              → KeyProvider (Pyrogram-бот @MicrosoftLeo_Bot)
       → запись заказа/ключей в SQLite (Order, IssuedKey)
       → email клиенту (Jinja2-шаблон по template_name, aiosmtplib)
       → Telegram-уведомление администратору (успех/ошибка)
```

Обрабатываются только заказы со статусом `completed` (`app/main.py`);
остальные статусы игнорируются с ответом 200. Form-urlencoded запросы
(WooCommerce webhook "ping") тоже принимаются с 200 без обработки.

## Ключевые компоненты (файлы)

- `app/main.py` — FastAPI приложение, единственный бизнес-эндпоинт.
- `app/core/config.py`, `app/core/models.py`, `app/core/security.py` —
  настройки, Pydantic-схемы входного payload, HTTP Basic auth дашборда.
- `app/services/processor.py` — оркестрация обработки заказа по "рецептам"
  из `product_config.json`.
- `app/services/key_provider.py` — получение ключа через Telegram
  (Pyrogram, от лица пользовательского аккаунта, не бота).
- `app/services/drweb_provider.py` — получение ключа через внешний
  HTTP API `drweb-key-service` (отдельный сервис, не входит в этот
  репозиторий).
- `app/services/notifier.py` — email (aiosmtplib + Jinja2) и Telegram
  (python-telegram-bot) уведомления.
- `app/database/` — SQLAlchemy async ORM поверх SQLite (`Order`,
  `IssuedKey`, `Product` — `Product` не используется, см.
  [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).
- `app/dashboard/` — веб-панель администратора (статус системы, список
  заказов, управление товарами через `product_config.json`, просмотр/
  предпросмотр email-шаблонов).

## Кто пользователи

- **WooCommerce** — единственный внешний вызывающий (webhook).
- **Администратор** — получает уведомления в Telegram и имеет доступ к
  веб-дашборду (`/dashboard`, HTTP Basic).
- **Конечный клиент** KeysPro.uz — получает ключ по email, напрямую с
  сервисом не взаимодействует.

## Каталог товаров

`product_config.json` (в корне проекта, live-редактируется через дашборд)
— маппинг SKU WooCommerce → описание товара, email-шаблон и список
"компонентов" (каждый — команда для получения ключа + источник). На
2026-09-05 содержит 7 SKU: Office 2019/2021 Pro Plus, Windows 10/11 Pro,
два бандла (Windows+Office) и Dr.Web Security Space. Подробности —
[STACK.md](STACK.md) и [DATABASE.md](DATABASE.md).
