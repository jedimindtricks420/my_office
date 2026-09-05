# AGENTS.md — keyspro_auto_sender

Инструкции для ИИ-агентов, работающих над документацией/задачами проекта
`keyspro_auto_sender` в `my_office`. Общие правила — в корневом
[`/AGENTS.md`](../../AGENTS.md); этот файл их не дублирует, только специфику
проекта.

## Проект

- **Исходный код (source of truth для кода):** `/home/admin1/keyspro_auto_sender/`
  — отдельный каталог вне `my_office`. Этот репозиторий
  (`my_office/projects/keyspro_auto_sender/`) содержит только документацию и
  задачи, НЕ код.
- **Не изменяй `/home/admin1/keyspro_auto_sender/` из задач `my_office`**,
  если задача явно не требует правок кода и это не согласовано отдельно.
- Приложение — единственный Python/FastAPI-проект, свой git-репозиторий
  (remote `origin` → `github.com/jedimindtricks420/keyspro_auto_sender`,
  ветка `master`). Есть также второй remote `new` →
  `github.com/jedimindtricks420/POSA-v3.git` — назначение не выяснено, см.
  [OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md).
- Внешний сервис, к API которого обращается `app/services/drweb_provider.py`
  (`DrWebProvider`), задокументирован отдельно — см.
  [`../drweb-key-service/`](../drweb-key-service/AGENTS.md) (исходный код:
  `/home/admin1/drweb-key-service/`, отдельный git-репозиторий). Он не
  исследовался в рамках задачи TASK-0001 этого проекта — связь между
  `keyspro_auto_sender` и `drweb-key-service` (конкретный `DRWEB_API_URL`,
  формат `sku`/`serial`) подтверждена только со стороны кода
  `keyspro_auto_sender`, не перепроверялась со стороны самого сервиса.

## Перед работой

1. Прочитай [`docs/STATUS.md`](docs/STATUS.md) — текущее подтверждённое
   состояние и известные проблемы.
2. Прочитай [`docs/OPEN_QUESTIONS.md`](docs/OPEN_QUESTIONS.md) — то, что не
   удалось подтвердить автоматически, включая находки безопасности.
3. Если задача касается кода — проверяй актуальный код в
   `/home/admin1/keyspro_auto_sender/`, а не полагайся только на эту
   документацию: это снимок на 2026-09-05.

## Известные особенности проекта (важно для агентов)

- **`WEBHOOK_SECRET` объявлен, но не используется.** `app/core/config.py`
  определяет поле `WEBHOOK_SECRET`, и `README.md` проекта утверждает, что
  оно "должно совпадать с настройками WooCommerce", но код эндпоинта
  `POST /api/v1/process-order` (`app/main.py`) нигде не читает и не проверяет
  этот секрет или какой-либо подписывающий заголовок. Любой JSON,
  прошедший валидацию Pydantic-схемой, будет обработан. См.
  [OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md).
- **Модель `Product` не используется.** В `app/database/models.py` есть
  ORM-модель `Product` с комментарием "replaces product_config.json", но
  весь дашборд (`app/dashboard/router.py`) и обработчик заказов
  (`app/services/processor.py`) читают/пишут товары исключительно через
  файл `product_config.json`. Таблица `products` создаётся в БД
  (`init_db`), но не заполняется и не читается кодом.
- **SQLite-база не персистентна.** `DATABASE_URL =
  "sqlite+aiosqlite:///keyspro.db"` — файл создаётся внутри контейнера
  (`/app/keyspro.db`), но `docker-compose.yml` НЕ монтирует его как volume
  (монтируются только `.env`, `app/`, `product_config.json`). Пересоздание
  контейнера (`docker compose up --force-recreate`, пересборка образа и
  т.п., не простой `restart`) приведёт к потере истории заказов и выданных
  ключей.
- **Два источника ключей по одному признаку `source`.** В
  `product_config.json` товары с `"source": "drweb"` идут через
  `DrWebProvider` (HTTP к внешнему API), остальные — через `KeyProvider`
  (Pyrogram, эмулирует пользователя Telegram, шлёт команду боту
  `@MicrosoftLeo_Bot` и парсит ответ regex'ом по истории чата). Это разные
  по надёжности механизмы: Pyrogram-путь зависит от парсинга текста
  сообщения и содержит `asyncio.sleep(60)` ожидания ответа бота.
  Не проверялось поведение при одновременных заказах на один и тот же
  Pyrogram-аккаунт (нет видимой блокировки/очереди — конкурентные вызовы
  `get_key` разных заказов могут гоняться за одной и той же историей чата).
- **Найден живой GitHub PAT в git-конфигурации хоста** (не в файлах
  репозитория): `origin` remote URL в `.git/config` содержит токен в
  открытом виде. НЕ записывать этот токен в документацию/задачи ни в каком
  виде. См. [OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md) — требует ротации
  вне рамок этой задачи (только документирование, не исправление).
- Секреты (`.env`, `keyspro.db` — может содержать реальные email клиентов
  и выданные лицензионные ключи) не читались и не должны копироваться в
  документацию `my_office`. Фиксируются только имена переменных, не
  значения.

## Структура этого каталога

Стандартная структура my_office-проекта — см. корневой `AGENTS.md`. Задача
`TASK-0001-bootstrap-project-context` (в `tasks/completed/` после
завершения) содержит журнал первоначального исследования — при сомнениях в
происхождении факта смотри туда.
