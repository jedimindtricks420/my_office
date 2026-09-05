# TASK-0001 — Bootstrap project context (1PDF)

**Статус:** завершена
**Начата:** 2026-09-05
**Завершена:** 2026-09-05
**Тип:** документационная (bootstrap), без изменения кода

## Цель

Зарегистрировать существующий проект 1PDF (`/home/admin1/1pdf/`) в `my_office`
и создать достоверную документацию его текущего состояния — только на
основании подтверждённого чтения кода/конфигурации, без разработки нового
функционала, исправлений или рефакторинга.

## Примечание о происхождении задачи

Исходный текст задачи содержал конфликт: источник кода был указан как
`/admin1/1pdf/` (на самом деле `/home/admin1/1pdf/` — проект «1PDF», приватный
PDF-редактор и платформа подписок), а целевая папка документации была указана
как `projects/contador/` — но `projects/contador/` уже содержит полную,
ранее завершённую документацию другого, не связанного проекта (Contador,
бухгалтерия РУз, `/home/admin1/contador/`). Конфликт задан пользователю явно
через уточняющий вопрос; получен ответ: регистрировать 1PDF как отдельный
новый проект в `projects/1pdf/`, не трогая `projects/contador/`.

## Границы

- Источник кода (read-only): `/home/admin1/1pdf/`. Код не изменялся
  (проверено: `git status` в проекте не показал изменений на всём протяжении
  исследования).
- Значения `.env`/секретов не читались и не копировались — фиксировались
  только имена переменных (`grep -oE '^[A-Z_]+=' .env`, без значений).
- Значения из `test-payments.sh`, обнаруженные как похожие на реальные
  ключи платёжных провайдеров, увидены при чтении файла, но НЕ скопированы
  ни в один документ `my_office` — только упомянуты по названию переменных
  и факту обнаружения.
- Целевая документация: `my_office/projects/1pdf/`.

## Журнал исследования

1. Прочитан корневой `my_office/AGENTS.md`, `CONTRIBUTING.md`, `README.md` —
   усвоена структура и стиль (использован существующий `projects/contador/`
   как образец качества и формата документации, без копирования его
   содержимого).
2. Верхнеуровневая структура `/home/admin1/1pdf/` (`find -maxdepth 3`,
   исключая `node_modules`, `.git`, `postgres_data`, `data_oo`/`db_oo`/
   `lib_oo`/`logs_oo`, `backups`, `temp_uploads`).
3. Прочитаны полностью: `README.md`, `package.json` (корень и `editor/`),
   `docker-compose.yml`, `Dockerfile.admin.v2`, `prisma/schema.prisma`,
   `nginx.freepdf.conf`.
4. Прочитан целиком `admin/server.ts` (1435 строк) — все роуты, middleware,
   аутентификация, платёжная логика, cron, legacy document/ONLYOFFICE-код.
5. Прочитаны `admin/routes/editorApi.ts`, `admin/controllers/
   paymeController.ts`, `admin/utils/paymentUtils.ts`, `admin/seed.ts`.
6. Прочитана вся собственная документация проекта в `docs/` (`ARCHITECTURE`,
   `API`, `OPERATIONS`, `PAYMENTS`, `SECURITY`, `DEVELOPMENT`) — сверена с
   кодом, признана в целом актуальной, но неполной (не покрывает admin-API и
   legacy ONLYOFFICE-эндпоинты).
7. Проверено реальное состояние хоста: `docker ps -a` (все 3
   managed-контейнера `1pdf-nginx`/`1pdf-admin`/`1pdf-postgres` — `Up`;
   обнаружен `onlyoffice-docs` в статусе `Exited`); поиск всех
   `docker-compose*` файлов на хосте — ни один не определяет
   `onlyoffice-docs`.
8. Интроспекция живой БД через `docker exec 1pdf-postgres psql -U onepdf -d
   onepdf` (`\dt`, `\d "PaymentAttempt"`, `\d "PaymentConfig"`, `SELECT
   migration_name, finished_at FROM _prisma_migrations`) — обнаружен и
   подтверждён дрейф: таблица `PaymentConfig` и 6 колонок `PaymentAttempt`
   существуют в БД без соответствующей миграции в `prisma/migrations/`.
   Значения данных не читались, только структура.
9. Проверены все 5 файлов миграций построчно, сопоставлены со
   `schema.prisma` и с живой схемой.
10. Найден и прочитан host-уровневый nginx-конфиг
    `/etc/nginx/sites-available/1pdf.uz` (Certbot-managed) — подтверждена
    полная цепочка TLS-терминации вне репозитория проекта.
11. Прочитаны `editor/package.json`, структура `editor/src/`,
    `editor/src/config.ts`, `editor/src/store/cloudStore.ts`,
    `editor/src/hooks/useDocumentBootstrap.ts`, `LICENSE.freePDF`,
    `THIRD_PARTY_NOTICES.md` — подтверждено происхождение как форк freePDF и
    сосуществование legacy/текущего document-flow на уровне клиента.
12. `git log`, `git status`, `git remote -v`, `git ls-files` в
    `/home/admin1/1pdf/` — подтверждена ветка `main`, чистое рабочее дерево,
    remote `1pdf_v1`, история миграции на freePDF-редактор (коммит
    `d3d8efd`).
13. Просмотрен `test-payments.sh` — обнаружены захардкоженные значения,
    похожие на реальные ключи Payme/Click; подтверждено через `git log
    --follow`, что файл закоммичен 2026-06-30 и не изменялся с тех пор.
    Значения не скопированы в документацию.
14. Проверены `.gitignore`/`.dockerignore`, размеры `temp_uploads/`/
    `backups/`, содержимое `backups/` и `extraConfigs/backup/` (только
    имена файлов и даты, не содержимое дампов).
15. Проверены `scripts/assert-migrations-additive.sh` (обнаружено: проверяет
    только одну конкретную миграцию по имени файла, не универсальный guard).
16. Проверены переменные окружения `.env` только по именам (`grep -oE`) —
    сопоставлены с использованием в коде; `FREE_PORT` и
    `DOCKER_EXAMPLE_ENABLED` не найдены нигде в коде/конфигурации.
17. Проверено, что `editor/dist` не отслеживается git и исключён из
    Docker-контекста (`.dockerignore`) — пересобирается в отдельном стейдже
    при каждой сборке образа.

## Итоговые документы

Созданы в `my_office/projects/1pdf/`:

- `AGENTS.md`, `README.md`
- `docs/OVERVIEW.md`, `docs/ARCHITECTURE.md`, `docs/STACK.md`,
  `docs/STATUS.md`, `docs/DATABASE.md`, `docs/API.md`,
  `docs/DEPLOYMENT.md`, `docs/OPEN_QUESTIONS.md`
- Пустые структурные каталоги: `docs/adr/`, `docs/runbooks/`,
  `specs/active/`, `specs/archived/`, `tasks/completed/` (эта задача после
  завершения перемещена сюда).

## Открытые вопросы

Полный список — [docs/OPEN_QUESTIONS.md](../../docs/OPEN_QUESTIONS.md).
Наивысший приоритет: возможные боевые платёжные ключи в открытом виде в
`test-payments.sh` (см. пункт 1 там же) — требует решения владельца проекта,
не устранялось в рамках этой документационной задачи.
