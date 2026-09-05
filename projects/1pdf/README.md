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
| [docs/STATUS.md](docs/STATUS.md) | Текущее подтверждённое состояние (снимок 2026-09-05) |
| [docs/DATABASE.md](docs/DATABASE.md) | Схема БД, миграции, подтверждённый дрейф схемы |
| [docs/API.md](docs/API.md) | API-поверхность (редактор, платежи, админка) |
| [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) | Docker, переменные окружения, деплой, TLS |
| [docs/OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md) | Что не удалось подтвердить автоматически |

## Источник

Код: `/home/admin1/1pdf/` (вне `my_office`, не изменялся при создании этой
документации). Исследование проведено 2026-09-05, журнал — в
`tasks/completed/TASK-0001-bootstrap-project-context.md`.
