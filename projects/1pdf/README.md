# 1PDF

Документация проекта 1PDF внутри `my_office`. Сам код проекта живёт вне этого
репозитория — см. [AGENTS.md](AGENTS.md).

## Что это

Приватная production-платформа для браузерного редактирования PDF с
аутентификацией, версионированием документов, подписками, ваучерами и
региональными платёжными провайдерами (Payme, Click, Alif), развёрнутая на
`1pdf.uz`. Форк открытого проекта freePDF. Подробнее —
[docs/OVERVIEW.md](docs/OVERVIEW.md).

## Навигация

| Файл | Содержимое |
|---|---|
| [docs/OVERVIEW.md](docs/OVERVIEW.md) | Функциональное описание, домен, происхождение проекта |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Архитектура, топология, поток данных редактора |
| [docs/STACK.md](docs/STACK.md) | Технологический стек, зависимости, сборка |
| [docs/STATUS.md](docs/STATUS.md) | Текущее подтверждённое состояние (снимок 2026-09-08) |
| [docs/DATABASE.md](docs/DATABASE.md) | Схема БД, миграции, подтверждённый дрейф схемы |
| [docs/API.md](docs/API.md) | API-поверхность (редактор, платежи, админка) |
| [docs/VIEWS.md](docs/VIEWS.md) | Карта HTML-страниц/React-компонентов → вызываемые API |
| [docs/CONFIGURATION.md](docs/CONFIGURATION.md) | Все переменные окружения: где используются, обязательность |
| [docs/GAPS.md](docs/GAPS.md) | Не реализовано / реализовано частично / мёртвый код |
| [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) | Docker, переменные окружения, деплой, TLS (RUNBOOK) |
| [docs/OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md) | Что не удалось подтвердить автоматически |

## Источник

Код: `/home/admin1/1pdf/` (вне `my_office`, не изменялся при создании этой
документации). Первичное исследование — 2026-09-05
(`tasks/completed/TASK-0001-bootstrap-project-context.md`); полный
независимый повторный аудит с нуля — 2026-09-08
(`tasks/completed/TASK-0002-full-technical-reaudit.md`) — актуальная версия
документации в `docs/` отражает именно его результаты.
