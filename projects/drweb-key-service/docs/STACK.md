# STACK — drweb-key-service

Последнее обновление: 2026-09-05 (TASK-0001). Источник — `requirements.txt`,
`Dockerfile`, `docker-compose.yml`, импорты в коде.

## Языки/рантайм

- **Python 3.11** — `FROM python:3.11-slim` в `Dockerfile`.

## FastAPI-сервис (`app/`)

- **Web-framework:** `fastapi` + `uvicorn` (версии не зафиксированы в
  `requirements.txt` — без пинов, кроме `curl_cffi`).
- **ORM/БД-доступ:** `sqlalchemy` (синхронный `create_engine`), СУБД —
  SQLite (`aiosqlite`) через файл (`sqlite:////app/data/drweb_keys.db` в
  контейнере). Пакеты `aiosqlite` и `databases` присутствуют в
  `requirements.txt`, но не импортируются в `app/database.py` — реальное
  использование не подтверждено, см. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).
- **Валидация/схемы:** `pydantic` (`app/schemas.py`).
- **HTTP-клиент для внешнего API:** `curl_cffi==0.14.0` — единственная
  зависимость с зафиксированной версией; выбрана специально для эмуляции
  TLS-отпечатка Chrome и обхода защиты QRATOR у Dr.Web (см.
  [OVERVIEW.md](OVERVIEW.md)).
- **Парсинг XML:** `xmltodict` (формат ответа Dr.Web).
- **Конфигурация:** `python-dotenv` (`load_dotenv()` в `main.py`,
  `database.py`, `drweb_client.py`).

## Django-админка (`drweb_admin/`, `core/`)

- **Framework:** `django` (версия не зафиксирована в `requirements.txt`;
  `INSTALLED_APPS`/API соответствуют Django 4/5).
- `django-environ` присутствует в `requirements.txt`, но
  `drweb_admin/settings.py` использует напрямую `os.environ.get(...)`, не
  `django_environ` — использование пакета в коде не подтверждено.
- Собственных дополнительных Django-приложений, кроме `core`
  (`ProductMapping`/`KeyLog` admin-обвязка), нет.

## Отсутствует / не подтверждено

- Нет фреймворка автоматического тестирования (`pytest`/`unittest` в
  `requirements.txt` не значатся; нет npm-аналога — это Python-проект).
  Единственный тестовый файл — `test_cffi.py` в корне, ручной скрипт для
  проверки `curl_cffi` напрямую (не запускается автоматически, не в
  `requirements.txt` как зависимость тестирования), содержит захардкоженные
  похожие на боевые Dr.Web логин/пароль в открытом виде; файл в
  `.gitignore`, не отслеживается git.
- Нет линтера/форматтера (`flake8`/`black`/`ruff` — не найдены).
- Нет CI/CD (`.github/workflows/` отсутствует, `Makefile` отсутствует).
- Нет виртуального окружения, зафиксированного в репозитории (`venv`/
  `.venv` на диске не найдены — зависимости ставятся только внутри Docker
  через `pip install -r requirements.txt`, версии пакетов вне
  `curl_cffi` не закреплены — нет lock-файла вроде `requirements.lock`/
  `poetry.lock`/`Pipfile.lock`).

## Инфраструктура

- **Контейнеризация:** один `Dockerfile` (без multi-stage), два сервиса в
  `docker-compose.yml` (`drweb-api`, `drweb-admin`), делят общий volume
  `./data` (SQLite-файл) и общий `.env`.
- **Reverse proxy/TLS:** не найдено — см.
  [DEPLOYMENT.md](DEPLOYMENT.md).
