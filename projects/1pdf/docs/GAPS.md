# Gaps — 1PDF

Новый файл, создан 2026-09-08 (`TASK-0002`). Только то, что подтверждено
прямым чтением кода — файл:строка при каждом утверждении. Методологическая
оговорка: `admin/server.ts`, `admin/routes/editorApi.ts`,
`admin/controllers/paymeController.ts`, `admin/utils/paymentUtils.ts`,
`prisma/schema.prisma`, все 5 файлов миграций, `docker-compose.yml`,
`nginx.freepdf.conf`, `Dockerfile.admin.v2` прочитаны целиком построчно;
`editor/src` (63 файла, 5399 строк) — не построчно каждый файл, а через
`grep` по `fetch(`/`import.meta.env`/`onlyoffice` плюс полное чтение 4
ключевых файлов (`config.ts`, `cloudStore.ts`, `useDocumentBootstrap.ts`,
`SaveButton.tsx`, `App.tsx`, `drafts.ts`); 3 файла `admin/public/*.html`
(суммарно 2743 строки) — не построчно, только через `grep` по `/api/`.
Отдельные факты о состоянии контейнеров и реальной БД проверены read-only
командами (`docker ps -a`, `docker exec ... psql -c '\d ...'`,
`SELECT migration_name FROM _prisma_migrations`) — 2026-09-08, ничего не
запускалось/останавливалось/менялось.

---

## Не реализовано

1. **Самообслуживание админа (смена своего логина/пароля) не работает —
   регрессия, не изначально не реализованная функциональность.**
   `admin/public/admin.html:1088` вызывает `PATCH /api/auth/me/login`,
   `admin.html:1097` — `PATCH /api/auth/me/password`. Ни один из этих
   маршрутов не зарегистрирован в текущем `admin/server.ts` (подтверждено
   `grep -n "auth/me" admin/server.ts` — только `/api/auth/me` GET и
   `/api/auth/login` POST). **Подтверждено через `git log -p`**: оба
   маршрута (`PATCH /api/auth/me/password`, `PATCH /api/auth/me/login`)
   реально существовали в `admin/server.ts` и были удалены в коммите
   `f1203b7` («feat: implement user authentication, subscription
   management, and role-based access control...») в рамках рефакторинга
   auth-системы — фронтенд `admin.html` при этом не был обновлён и
   продолжает вызывать удалённые эндпоинты. При отправке формы пользователь
   получит `404`.
2. **Асинхронная загрузка версии документа не реализована** — enum
   `DocumentVersionStatus` объявляет `UPLOADING`/`FAILED`
   (`prisma/schema.prisma:125-129`), эндпоинт `POST
   /api/v1/documents/:documentId/versions/:versionId/complete`
   существует (`editorApi.ts:338-349`) для подтверждения готовности версии
   — но **оба места кода, создающие `DocumentVersion`
   (`editorApi.ts:160-163`, `192-203`), сразу ставят `status: 'READY'`**,
   минуя `UPLOADING`. `/complete`-эндпоинт не вызывается ни одним найденным
   местом в `editor/src` (проверено `grep -rn "complete"` по фронтенду) —
   готовая, но не задействованная инфраструктура для будущей асинхронной
   загрузки больших файлов.
3. **Ссылочная целостность части связей не реализована на уровне схемы** —
   `Voucher.used_by_id` (`schema.prisma:78`) и
   `DocumentVersion.source_version_id` (`schema.prisma:107`) — обычные
   `String?`, не `@relation`-поля; Prisma/Postgres не проверяют, что
   значение действительно существует в целевой таблице.
4. **`User.telegram_id`** (`schema.prisma:15`, unique) объявлено, но не
   найдено ни одного места кода (`admin/`, `editor/`), которое его читает
   или записывает — предположительно задел под Telegram-интеграцию,
   которой пока нет.
5. **`theme` в `GET /api/v1/me/preferences`** (`editorApi.ts:105-112`)
   всегда возвращает жёстко закодированную строку `'system'` — настройка
   темы объявлена в контракте ответа, но не хранится и не настраивается
   пользователем.

## Реализовано частично

1. **Дрейф схемы БД от истории миграций** (подробности —
   [DATABASE.md](DATABASE.md), перепроверено read-only SQL 2026-09-08):
   базовая таблица `PaymentConfig` и 6 полей `PaymentAttempt`
   (переименование `external_id`→`external_payment_id`,
   `amount_tiyin`, `cancel_reason`, `cancel_time`, `paid_at`,
   `processing_status`) существуют в реальной БД, но не создаются ни одним
   из 5 файлов `prisma/migrations/*/migration.sql`. `prisma migrate deploy`
   на пустой базе (`Dockerfile.admin.v2:22` выполняет его при каждом
   старте) **не воспроизведёт текущую рабочую схему** — риск
   непосредственно при восстановлении окружения с нуля.
2. **`requirePro` продублирован независимо в двух местах** —
   `admin/server.ts:282-288` и `admin/routes/editorApi.ts:92-101` —
   одинаковая логика («`plan === 'PRO'` и (`valid_until` пусто или в
   будущем)»), реализованная дважды без общего модуля. Изменение правила
   PRO-доступа в одном месте не гарантированно применится к другому.
3. **Секреты платёжных провайдеров — два источника истины без чёткого
   приоритета.** `admin/utils/paymentUtils.ts:35-36,49-50,76-79`
   (`generatePaymeUrl`/`generateClickUrl`/`checkPaymeAuth`) читают
   `process.env.PAYME_*`/`CLICK_*` только когда параметр `config` вообще не
   передан; при этом `admin/server.ts:1003,1036` (Click prepare/complete)
   явно фолбэчит на `process.env.CLICK_SECRET_KEY`, **даже когда `config`
   из БД существует, но поле в нём пусто** (`config?.click_secret_key ||
   process.env.CLICK_SECRET_KEY`) — то есть для Click два потенциальных
   источника секрета могут молча разойтись, для Payme через
   `paymeController.ts` — только БД (иное поведение в разных частях
   кодовой базы).
4. **`JWT_SECRET_OO` не имеет обязательной проверки при старте** —
   `admin/server.ts:45`: `process.env.JWT_SECRET_OO || ""` — пустая строка
   как секрет технически принимается `jsonwebtoken` (`jwt.sign`/
   `jwt.verify` с `secret=""` работают), что делает подпись
   ONLYOFFICE-токенов (`/uploads/:filename?token=`, `/api/documents/config`,
   `/api/documents/callback`) предсказуемой/подделываемой при
   незаполненной переменной — в отличие от `JWT_SECRET`, для которого есть
   явная защита (`server.ts:41-43`).
5. **`VITE_KOFI_HANDLE` не переопределён нигде в найденной конфигурации
   сборки** (`.env`, `docker-compose.yml`, `Dockerfile.admin.v2` не
   передают этот build-time аргумент в `npm --prefix editor run build`) —
   собранный `editor/dist`, по всей видимости, ссылается на донат-аккаунт
   `chinmay_shringi` (автор оригинального open-source freePDF), а не на
   1PDF (`editor/src/config.ts:8`). Не проверено напрямую по содержимому
   `dist` (см. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).
6. **`DELETE /api/users/:userId` не обрабатывает конфликт внешних ключей**
   (`admin/server.ts:785-789`) — `Document.owner_id` и
   `DocumentVersion.created_by_id` объявлены с `onDelete: Restrict`
   (`schema.prisma:93,113`); попытка удалить пользователя, у которого есть
   документы, приведёт к необработанному исключению Prisma и общему `500`
   через глобальный error-handler (`server.ts:1428-1431`), а не к понятному
   `409`/сообщению для админа.
7. **`scripts/assert-migrations-additive.sh` — не общий guard, а разовая
   проверка одной захардкоженной миграции** (`20260801000000_add_editor_
   documents_and_language`, путь зашит буквально в
   `assert-migrations-additive.sh:4`) — имя скрипта звучит как постоянная
   защита для всех будущих миграций, но структурно проверяет только этот
   один файл; новая миграция не будет проверена этим скриптом автоматически.
8. **Отсутствие `SMTP_PASS` не блокирует регистрацию, но обессмысливает
   её** — `admin/server.ts:174-176` при отсутствии `SMTP_PASS` в
   production выводит только `console.warn`, не бросает исключение;
   `POST /api/client/signup` (`server.ts:525-567`) всё равно выдаёт рабочий
   `signupToken`, но пользователь никогда не получит OTP-код по почте и не
   сможет пройти `verify-signup` — тихий сбой без пользовательской ошибки
   на этапе регистрации.

## Мёртвый код

1. **Legacy ONLYOFFICE-эндпоинты в `admin/server.ts`** — `GET
   /api/documents/config` (1197-1274), `POST /api/documents/upload`
   (1276-1299), `POST /api/documents/create-new` (1301-1341) — ни один не
   вызывается из `editor/src` (проверено `grep -rn "fetch("` по всему
   каталогу — только `/api/v1/*` и `/uploads/*`) и не вызывается из
   `admin/public/*.html` (проверено `grep` по `/api/` литералам во всех
   трёх файлах). `POST /api/documents/callback` (1343-1395) —
   вебхук, вызываемый только самим ONLYOFFICE Document Server, который
   подтверждённо не запущен (`docker ps -a` → `onlyoffice-docs`, статус
   `Exited (127) 2 недели назад`) и не описан в `docker-compose.yml`.
   Зависимость `JWT_SECRET_OO` (см. выше) обслуживает исключительно эту
   мёртвую группу эндпоинтов.
2. **Каталоги `data_oo/`, `db_oo/`, `lib_oo/`, `logs_oo/`** (корень
   репозитория) — остатки самостоятельно развёрнутого ONLYOFFICE Document
   Server; явно исключены из контекста сборки Docker-образа
   (`.dockerignore:4-7`), не подключены ни к одному сервису в
   `docker-compose.yml`.
3. **`FREE_PORT`, `DOCKER_EXAMPLE_ENABLED`** (`.env`) — не найдено ни
   одного места чтения `process.env.FREE_PORT`/`DOCKER_EXAMPLE_ENABLED` в
   `admin/`, `editor/`, `docker-compose.yml`. `DOCKER_EXAMPLE_ENABLED`
   по имени, вероятно, относился к `lib_oo/documentserver-example`
   (готовый docker-example ONLYOFFICE от вендора) — той же мёртвой группе.
4. **`POST /api/v1/documents/:documentId/versions/:versionId/complete`**
   (`editorApi.ts:338-349`) — реализован, но не вызывается ни одним
   найденным местом фронтенда (см. пункт 2 «Не реализовано» — сам
   эндпоинт жив и работает, но его единственный правдоподобный вызывающий
   сценарий — асинхронная загрузка — не реализован клиентом).

---

## Дополнительно — не гап, но независимо перепроверенный существенный факт

Ни одна из read-only проверок 2026-09-08 (`docker ps -a`, `docker exec
1pdf-postgres psql`) не изменяла состояние системы. Все находки о
контейнере `onlyoffice-docs` и о дрейфе схемы БД воспроизводят и уточняют
(с точными именами столбцов и номерами строк миграций) находки предыдущей
задачи `TASK-0001` (2026-09-05, зафиксированы в `AGENTS.md` этого
my_office-проекта) — независимо подтверждены, а не унаследованы слепо.
