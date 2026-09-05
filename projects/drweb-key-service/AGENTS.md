# AGENTS.md — drweb-key-service

Инструкции для ИИ-агентов, работающих над документацией/задачами проекта
`drweb-key-service` в `my_office`. Общие правила — в корневом
[`/AGENTS.md`](../../AGENTS.md); этот файл их не дублирует, только специфику
проекта.

## Проект

- **Исходный код (source of truth для кода):** `/home/admin1/drweb-key-service/`
  — отдельный каталог вне `my_office`. Этот репозиторий
  (`my_office/projects/drweb-key-service/`) содержит только документацию и
  задачи, НЕ код.
- **Не изменяй `/home/admin1/drweb-key-service/` из задач `my_office`**, если
  задача явно не требует правок кода и это не согласовано отдельно.
- Свой git-репозиторий (remote `git@github.com:jedimindtricks420/
  dr.web-custom-api.git`), не подмодуль `my_office`.
- Микросервис из двух процессов в одном образе: FastAPI (`app/`, порт 8001,
  контейнер `drweb-api`) и Django-админка (`drweb_admin/` + `core/`, порт
  8002, контейнер `drweb-admin`), общая SQLite-база `data/drweb_keys.db`.
  Подробности — [ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Перед работой

1. Прочитай [`docs/STATUS.md`](docs/STATUS.md) — текущее подтверждённое
   состояние и известные проблемы.
2. Прочитай [`docs/OPEN_QUESTIONS.md`](docs/OPEN_QUESTIONS.md) — то, что не
   удалось подтвердить автоматически.
3. Если задача касается кода — проверяй актуальный код в
   `/home/admin1/drweb-key-service/`, а не полагайся только на эту
   документацию: это снимок на 2026-09-05.

## Известные особенности проекта (важно для агентов)

- Единственный бизнес-эндпоинт — `POST /drweb/api/v1/generate`, защищён
  статическим Bearer-токеном (`API_KEY` из `.env`, сравнение обычным `!=`).
  Также есть `/drweb/api/v1/products`, `/drweb/api/v1/logs` (тот же токен) и
  публичный `/health`. См. [API.md](docs/API.md).
- **Логин и пароль Dr.Web логируются в открытом виде.**
  `app/drweb_client.py` подставляет их прямо в URL и затем
  `logger.info(f"Requesting Dr.Web: url={final_url}")` пишет этот URL (уже с
  учётными данными) в лог контейнера. Подтверждено прямым чтением кода. Не
  исправлялось в рамках TASK-0001 — только задокументировано.
- **Пароль Django-суперпользователя захардкожен в `docker-compose.yml`**
  (`office_admin1` / `merchant123`), этот файл отслеживается git — то есть
  пароль лежит в репозитории в открытом виде. См. [STATUS.md](docs/STATUS.md).
- `drweb_admin/settings.py`: `DEBUG = True`, `ALLOWED_HOSTS = ['*']`,
  захардкоженный `SECRET_KEY`. Оба порта (8001, 8002) на момент проверки
  проброшены на `0.0.0.0` без reverse-proxy/TLS на уровне Nginx этого хоста
  (конфигурации для них не найдено). См. [DEPLOYMENT.md](docs/DEPLOYMENT.md).
- Таблицы `product_mapping` и `key_logs` физически ведёт SQLAlchemy
  (`app/database.py`, `Base.metadata.create_all`), Django-модели в
  `core/models.py` объявлены с `managed = False` и просто читают/пишут в те
  же таблицы через тот же файл SQLite. Файлов миграций Django для `core` нет
  (`core/migrations/` содержит только пустой `__init__.py`).
- `requirements.txt` содержит `django-environ`, `aiosqlite`, `databases` —
  использование этих пакетов в реальном коде (`os.environ` используется
  напрямую, `app/database.py` использует синхронный `sqlalchemy.
  create_engine`) не подтверждено. См. [OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md).
- Нет автоматических тестов, нет CI/CD, нет линтера. `test_cffi.py` в корне
  — ручной скрипт с захардкоженными похожими на боевые Dr.Web-учётными
  данными в открытом виде; файл в `.gitignore` и не отслеживается git, но
  физически присутствует на диске сервера.
- POSA (другой проект на этом хосте, `projects/posa/` в `my_office`) имеет
  собственные модели `DrWebOrder`/`DrWebApiLog`/`DrWebSku` в своей
  Prisma-схеме — связь между этой интеграцией POSA и `drweb-key-service` не
  выяснялась (не входило в область TASK-0001). См. OPEN_QUESTIONS.md.

## Структура этого каталога

Стандартная структура my_office-проекта — см. корневой `AGENTS.md`. Задача
`TASK-0001-bootstrap-project-context` (в `tasks/completed/` после
завершения) содержит журнал первоначального исследования — при сомнениях в
происхождении факта смотри туда.
