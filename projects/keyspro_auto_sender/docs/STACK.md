# STACK — keyspro_auto_sender

Последнее обновление: 2026-09-05 (TASK-0001). Источник —
`requirements.txt`, `Dockerfile`, `docker-compose.yml` в
`/home/admin1/keyspro_auto_sender/`.

## Backend

- **Runtime:** Python 3.11 (`python:3.11-slim` в `Dockerfile`).
- **Framework:** FastAPI + `uvicorn[standard]` (ASGI-сервер, запуск без
  `--reload`, один процесс).
- **Валидация/настройки:** `pydantic[email]`, `pydantic-settings`
  (конфигурация из `.env` через `Settings(BaseSettings)`).
- **ORM/БД:** `sqlalchemy[asyncio]` + `aiosqlite` (SQLite, асинхронный
  доступ). Нет Alembic и вообще формальных миграций — схема создаётся
  через `Base.metadata.create_all` при старте приложения.
- **Telegram:**
  - `pyrogram` + `tgcrypto` — клиент от лица пользовательского аккаунта
    (не бота) для получения ключей у стороннего бота `@MicrosoftLeo_Bot`.
  - `python-telegram-bot` — обычный бот-клиент для клиентских/админских
    уведомлений (`CLIENT_BOT_TOKEN`).
- **Шаблонизация:** `jinja2` (email-шаблоны и HTML-страницы дашборда,
  последний — через `fastapi.templating.Jinja2Templates`).
- **Email:** `aiosmtplib` (асинхронная отправка через SMTP, SSL/TLS или
  STARTTLS — управляется `SMTP_SSL_TLS`/`SMTP_STARTTLS`).
- **HTTP-клиент:** `aiohttp==3.9.5` (единственная зафиксированная версия в
  `requirements.txt`; остальные зависимости без пиннинга версий) — для
  запросов к внешнему Dr.Web API.
- **Формы:** `python-multipart` (загрузка форм дашборда — редактирование
  товаров).
- **Прочее:** `python-dotenv`.

## Отсутствует / не подтверждено

- Нет фреймворка автоматического тестирования (jest/pytest и т.п. не
  найдены ни в `requirements.txt`, ни в дереве проекта — `find . -iname
  "*test*"` ничего не находит).
- Нет CI (каталог `.github/` отсутствует).
- Нет линтера/форматтера (flake8/ruff/black и т.п. не в зависимостях).
- Версии большинства Python-пакетов в `requirements.txt` не зафиксированы
  (нет `==`), кроме `aiohttp==3.9.5` — фактически установленные версии в
  контейнере не сверялись (не было полного `pip freeze` в рамках задачи).

## Инфраструктура

- **Контейнеризация:** Docker + Docker Compose, один сервис
  `key-distributor` (образ собирается локально из `Dockerfile`, без
  registry). Non-root пользователь `app` внутри контейнера (`useradd` в
  `Dockerfile`).
- **Веб-сервер/reverse proxy:** Nginx на хосте, конфиг
  `/etc/nginx/sites-available/keyspro_auto_sender` для домена
  `webhook.keyspro.uz`. TLS — Let's Encrypt/Certbot.
- **Healthcheck:** `docker-compose.yml` — `curl -f
  http://localhost:8000/health` каждые 30с.
- Подробности текущего состояния контейнера/деплоя — см.
  [DEPLOYMENT.md](DEPLOYMENT.md).
