# MIBT

Документация и задачи для проекта **MIBT** в системе `my_office`.

Исходный код проекта живёт вне этого репозитория, в
`/home/admin1/mibt-2/` — здесь только документация, спецификации и задачи
ИИ-агентов. См. [`AGENTS.md`](AGENTS.md) для правил работы над этим проектом.

## Быстрый старт для агента

1. [`AGENTS.md`](AGENTS.md) — правила и известные особенности проекта.
2. [`docs/STATUS.md`](docs/STATUS.md) — текущее подтверждённое состояние.
3. [`docs/OVERVIEW.md`](docs/OVERVIEW.md) — что это за продукт.
4. [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — техническая архитектура.
5. [`docs/API.md`](docs/API.md) — полный справочник всех 77 эндпоинтов.
6. [`docs/DATABASE.md`](docs/DATABASE.md) — модель данных, поля, связи.
7. [`docs/VIEWS.md`](docs/VIEWS.md) — карта всех 35 EJS-шаблонов и форм.
8. [`docs/CONFIGURATION.md`](docs/CONFIGURATION.md) — переменные окружения.
9. [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) — установка, запуск, миграции.
10. [`docs/GAPS.md`](docs/GAPS.md) — не реализовано / частично / мёртвый код.
11. [`docs/OPEN_QUESTIONS.md`](docs/OPEN_QUESTIONS.md) — что не подтверждено.

## Структура

```
docs/            — OVERVIEW, ARCHITECTURE, STACK, DATABASE, API, VIEWS,
                   CONFIGURATION, DEPLOYMENT, GAPS, STATUS, OPEN_QUESTIONS
docs/adr/        — архитектурные решения (пока пусто)
docs/runbooks/   — операционные инструкции (см. также раздел RUNBOOK в DEPLOYMENT.md)
specs/           — спецификации фич (active/archived, пока пусто)
tasks/           — задачи агентов (active/completed)
```
