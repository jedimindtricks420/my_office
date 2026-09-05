# API — tgkeysbot

Последнее обновление: 2026-09-05 (TASK-0001). Источник — `config/urls.py`,
`bot/main.py`, `bot/drweb_client.py` в `/home/admin1/tgkeysbot/`.

## Публичное HTTP API

**Отсутствует.** `config/urls.py` определяет единственный маршрут —
`admin/` (`django.contrib.admin.site.urls`). Django-приложение `core` не
регистрирует собственных views/URL (`core/views.py` — пустая заготовка
`render`, без единого маршрута). Нет REST API, нет webhook-эндпоинтов —
весь пользовательский интерфейс партнёра работает только через Telegram
(long polling aiogram, без HTTP-сервера бота).

## Django-админка (`/admin/`, за Nginx — `/partner_bot/`)

Аутентификация — стандартная сессионная аутентификация Django
(`django.contrib.auth`), логин/пароль суперпользователя/staff-пользователя.
Отдельных ролей/API-ключей не заведено.

Зарегистрированные модели (`core/admin.py`), все под стандартным
Django Admin UI:

| Модель | Возможности в админке |
|---|---|
| `Partner` | список/поиск/фильтр по статусу; inline-заказы и история баланса (read-only); custom action и отдельная страница `/{id}/adjust-balance/` для ручной корректировки баланса (создаёт `BalanceLedger` типа `TOPUP`/`MANUAL_ADJUST`, сбрасывает `debt_limit_notified` при возврате выше лимита) |
| `Product` | список/поиск/фильтр; `is_active` и `price` редактируются прямо из списка (`list_editable`) |
| `Order` | только просмотр (создание/удаление запрещено в админке — `has_add_permission`/`has_delete_permission` возвращают `False`), все поля readonly |
| `BalanceLedger` | только просмотр, создание/удаление запрещено |
| `SourceAccount` | обычный CRUD (см. [DATABASE.md](DATABASE.md) — не читается ботом) |
| `Settings` | обычный CRUD (key-value) |
| `Instruction` | CRUD, `raw_id_fields` для `product` |

## Telegram-бот (клиентский интерфейс, `bot/main.py`)

Не HTTP API, но фиксирую как основной "публичный" интерфейс продукта.
Long polling через aiogram (`dp.start_polling(bot)`), без вебхука.

| Команда/кнопка | Условие доступа | Действие |
|---|---|---|
| `/start` | `Partner` с данным `telegram_user_id` и `status=ACTIVE` должен существовать | приветствие + главное меню, иначе "Доступ отсутствует" |
| "Баланс" | активный партнёр | текущий баланс и лимит долга, предупреждение при превышении |
| "Инструкции" | активный партнёр | список товаров с инструкцией (с учётом `allowed_products`), по выбору — текст + опциональное фото |
| "Каталог" | активный партнёр | список активных товаров (первые 10, с учётом `allowed_products`) → выбор количества (1-10, с пагинацией) → покупка |
| "Активация" | активный партнёр | запрашивает Installation ID (≥20 цифр), пересылает боту-источнику, возвращает Confirmation ID |

Все проверки доступа партнёра выполняются заново на каждый хэндлер
(`get_partner()`), состояние диалога (FSM) хранится в памяти процесса
(`MemoryStorage` — не переживает перезапуск процесса `bot`).

## Внешние API, которые вызывает бот

- **Telegram Bot API** — через `aiogram.Bot(token=CLIENT_BOT_TOKEN)`,
  стандартные вызовы отправки сообщений/клавиатур/фото.
- **Telegram MTProto (Pyrogram)** — от лица пользовательского аккаунта
  (`SESSION_STRING`/`API_ID`/`API_HASH`): `send_message` и
  `get_chat_history` к боту `TARGET_BOT_USERNAME` (`@MicrosoftLeo_Bot`).
  Формат и поведение этого стороннего бота (набор команд, формат ответов)
  задокументированы только косвенно — через регулярные выражения и
  ключевые фразы, на которые опирается парсинг в `bot/main.py`
  (`KEY_REGEX`, `CID_REGEX`, поиск подстрок "Purchase Successful"/"Key
  blocked"/"Confirmation ID").
- **Dr.Web API (внешний сервис `drweb-key-service`)** —
  `bot/drweb_client.get_key_from_drweb()`: `POST {DRWEB_API_URL}`,
  заголовок `Authorization: Bearer {DRWEB_API_TOKEN}`, JSON-тело
  `{"sku": <код>, "source": DRWEB_API_SOURCE_NAME}`, ожидаемый успешный
  ответ — JSON с полем `serial`. Не подтверждено сверкой с самим сервисом
  `drweb-key-service` в рамках этой задачи (см.
  [AGENTS.md](../AGENTS.md)).
