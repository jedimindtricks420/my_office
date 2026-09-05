# STACK — tgkeysbot

Последнее обновление: 2026-09-05 (TASK-0001). Источник —
`requirements.txt`, `Dockerfile`, `docker-compose.yml` и `pip show` внутри
работающего контейнера `tgkeysbot_web_1` в
`/home/admin1/tgkeysbot/`.

## Backend

- **Runtime:** Python 3.12 (`python:3.12-slim` в `Dockerfile`).
- **Framework:** Django (в `requirements.txt` указано `Django>=4.2` без
  верхней границы; фактически установленная в контейнере версия —
  **6.0.3**, подтверждено `pip show django`). Запуск в проде через
  `gunicorn config.wsgi:application` (версия gunicorn в контейнере —
  25.2.0).
- **БД/ORM:** Django ORM + `psycopg2-binary` (2.9.11) — PostgreSQL 16 в
  проде. `dj-database-url` (3.1.2) парсит `DATABASE_URL`. Без
  `DATABASE_URL` в окружении Django откатывается на локальный
  `db.sqlite3` (см. [DATABASE.md](DATABASE.md)).
- **Статика:** `whitenoise` (6.12.0) — раздача статики Django-админки
  напрямую из процесса gunicorn, без отдельного веб-сервера для static/.
- **Файлы:** `Pillow` (12.1.1) — обработка изображений для поля
  `Instruction.image`.
- **Telegram:**
  - `aiogram>=3.0` (фактически 3.26.0) — клиентский бот-интерфейс партнёра.
  - `pyrogram` (фактически 2.0.106) + `tgcrypto` (1.2.5) — MTProto-клиент
    от лица обычного пользовательского Telegram-аккаунта, используется
    для отправки команд стороннему боту-источнику и чтения истории чата.
- **Async:** `asgiref` (3.11.1) — используется в `bot/main.py`
  (`sync_to_async`) для вызовов Django ORM из асинхронного aiogram-кода.
- **HTTP-клиент:** `aiohttp` — используется в `bot/drweb_client.py` для
  запроса к внешнему Dr.Web API (версия не зафиксирована отдельно в
  `requirements.txt`, ставится как транзитивная зависимость Pyrogram/
  aiogram; отдельно `pip show aiohttp` не снимался).
- **Конфигурация:** `python-dotenv` (1.2.2, `load_dotenv()` в
  `config/settings.py` и в служебных скриптах), `django-environ` (0.13.0,
  установлен, но использование в коде не найдено — `grep -rn environ` не
  находит обращений в `config/settings.py`, там читается `os.getenv`
  напрямую).

## Неподключённые/недостающие зависимости

- **`transport/tasks.py` импортирует `celery` и `redis`**, но оба пакета
  отсутствуют в `requirements.txt` — этот модуль не может быть
  импортирован в текущем окружении контейнеров. См.
  [ARCHITECTURE.md](ARCHITECTURE.md).

## Отсутствует / не подтверждено

- Нет фреймворка автоматического тестирования: `core/tests.py` — пустая
  Django-заготовка без единого теста; `pytest`/`django.test` runner не
  используются целенаправленно. `transport/test_pyrogram.py` — ручной
  диагностический скрипт (печатает результат в консоль, не содержит
  `assert`/test-runner интеграции), не автотест.
- Нет CI (каталог `.github/` или аналогичный отсутствует).
- Нет линтера/форматтера в зависимостях (flake8/ruff/black и т.п. не
  найдены).
- Версии в `requirements.txt` в основном не зафиксированы (только
  `Django>=4.2` и `aiogram>=3.0` задают нижнюю границу, остальные пакеты —
  без ограничений версии вообще).

## Инфраструктура

- **Контейнеризация:** Docker + Docker Compose, три сервиса (`db`, `web`,
  `bot`) собираются из одного `Dockerfile` (multi-stage: builder собирает
  wheels, финальный образ ставит их из `/wheels`). Контейнер работает от
  **root** — в `Dockerfile` не создаётся отдельный non-root пользователь
  (подтверждено `docker exec ... whoami` → `root` в обоих контейнерах
  `web` и `bot`).
- **Веб-сервер/reverse proxy:** Nginx на хосте, общий вход
  `webhook.keyspro.uz` (виртуалхост принадлежит соседнему проекту
  `keyspro_auto_sender`, `location /partner_bot/` проксирует сюда). TLS —
  Let's Encrypt/Certbot (сертификат общий для домена, не отдельный для
  `tgkeysbot`).
- Подробности текущего состояния контейнеров/деплоя — см.
  [DEPLOYMENT.md](DEPLOYMENT.md).
