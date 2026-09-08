# Contador

Документация проекта Contador внутри `my_office`. Сам код проекта живёт вне этого
репозитория — см. [AGENTS.md](AGENTS.md).

## Что это

Мультитенантная бухгалтерская SaaS-система для Узбекистана (двойная запись,
НСБУ №1/№21), с AI-классификацией банковских транзакций. Подробнее —
[docs/OVERVIEW.md](docs/OVERVIEW.md).

## Навигация

| Файл | Содержимое |
|---|---|
| [docs/OVERVIEW.md](docs/OVERVIEW.md) | Функциональное описание, приложения, домен |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Архитектура, структура каталогов |
| [docs/STACK.md](docs/STACK.md) | Технологический стек, зависимости, сборка |
| [docs/STATUS.md](docs/STATUS.md) | Текущее подтверждённое состояние (снимок 2026-09-08) |
| [docs/DATABASE.md](docs/DATABASE.md) | Схемы БД, модели, миграции |
| [docs/API.md](docs/API.md) | REST API v2, аутентификация |
| [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) | Docker, переменные окружения, деплой (RUNBOOK) |
| [docs/VIEWS.md](docs/VIEWS.md) | Карта страниц v2 → вызываемые API, admin-панель |
| [docs/CONFIGURATION.md](docs/CONFIGURATION.md) | Все переменные окружения: где используются, обязательность |
| [docs/GAPS.md](docs/GAPS.md) | Не реализовано / реализовано частично / мёртвый код |
| [docs/OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md) | Что не удалось подтвердить автоматически |

## Источник

Код: `/home/admin1/contador/` (вне `my_office`, не изменялся при создании этой
документации). Первичное исследование — 2026-09-05
(`tasks/completed/TASK-0001-bootstrap-project-context.md`); полный независимый
повторный аудит с нуля — 2026-09-08
(`tasks/completed/TASK-0002-full-technical-reaudit.md`) — актуальная версия
документации в `docs/` отражает именно его результаты.
