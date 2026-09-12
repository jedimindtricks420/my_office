# Status — 1PDF

Последнее обновление: **2026-09-12** (`TASK-0004` — новая вкладка «SEO» в
админке: коды верификации Google/Yandex/Bing, управление
`sitemap.xml`/SEO-страницами, по прямому запросу пользователя; код
изменён, задеплоен в продакшн, коммит `283b9bd`). Предыдущие: `TASK-0003`
(2026-09-08, SEO-стратегия и публикация 22 landing pages), `TASK-0002`
(2026-09-08, полный независимый повторный аудит), `TASK-0001`
(2026-09-05, первичная регистрация).

## TASK-0004 — что сделано (админка: SEO-верификация + sitemap, код изменён и задеплоен)

- Новая таблица `SeoSettings` (Prisma, миграция
  `20260912000000_add_seo_settings`, additive-only) хранит коды
  верификации Google/Yandex/Bing, опциональные verification-файлы и
  произвольный код в `<head>` (аналитика).
- `admin/server.ts`: `GET/POST /api/admin/seo-settings`,
  `GET /api/admin/seo/status`, `POST /api/admin/seo/regenerate-sitemap`,
  `POST /api/admin/seo/regenerate-pages`; главная страница (`/` и
  `/index.html` — оба нужны из-за `nginx.freepdf.conf`) теперь рендерится
  динамически с подмешиванием метатегов из БД; отдельный маршрут отдаёт
  verification-файлы по точному совпадению имени.
- Новая вкладка «SEO» в `admin/public/admin.html` — UI для всего
  вышеперечисленного.
- `docker-compose.yml`/`Dockerfile.admin.v2`: каталог `scripts/` теперь
  смонтирован и копируется в контейнер `1pdf-admin`, чтобы админка могла
  вызывать канонические `scripts/generate-sitemap.mjs` и
  `scripts/generate-seo-pages.mjs`, не дублируя их логику.
- Задеплоено на прод: бэкап БД снят, `docker compose build admin` +
  `docker compose up -d --no-deps admin` (пересборка образа обязательна —
  новая Prisma-модель требует свежего generated client, который печётся в
  образ, а не монтируется с хоста), миграция применена автоматически при
  старте контейнера. Все существующие маршруты проверены `curl` после
  деплоя — без регрессий.
- **Побочная находка и исправление**: `.gitignore` содержал общее правило
  `*.sql`, которое с 2026 (коммит `be82ad8`) молча исключало из git все
  новые файлы Prisma-миграций — 2 реальные production-миграции
  (`add_voucher_series`, `add_editor_documents_and_language`) никогда не
  были закоммичены, хотя были применены к боевой БД. Исправлено
  исключением `!prisma/migrations/**/*.sql`, обе миграции восстановлены в
  git. Подробности — [TASK-0004](../tasks/completed/TASK-0004-seo-admin-verification.md),
  [GAPS.md](GAPS.md).
- **Не выполнено** (требует владельца продукта, не восстанавливается
  агентом): сама верификация в Google Search Console/Yandex.Webmaster —
  нужно создать свойство сайта и вставить выданный код через новую вкладку
  SEO в админке. Техническая инфраструктура для этого теперь готова. См.
  [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md) п.16.

## TASK-0003 — что сделано (SEO landing pages, код изменён и задеплоен)

- Read-only capability audit `editor/src` подтвердил реальные возможности
  редактора (добавление/замена текста, изображения, подпись, выделение,
  рисование, фигуры/штампы, ротация/перестановка/удаление/слияние/
  разбиение страниц) — полная таблица SUPPORTED/NOT_IMPLEMENTED в
  `tasks/completed/TASK-0003-seo-strategy-ru-uz.md`.
- Сгенерированы и опубликованы **22 статические SEO-страницы**
  (`admin/public/seo/`, 10 RU/UZ пар + хаб) через новый
  `scripts/generate-seo-pages.mjs`; `robots.txt` + `sitemap.xml` через
  `scripts/generate-sitemap.mjs`.
- `nginx.freepdf.conf` — новый публичный маршрут `/ru/<slug>/`,
  `/uz/<slug>/`, без PRO-гейта (в отличие от `/editor/*`).
- `admin/public/index.html` — навигация и футер дополнены ссылками на
  новые страницы.
- Всё закоммичено и запушено в `1pdf_v1@main` (`654c09c`), проверено
  `curl` через продовый `1pdf-nginx` — все 22 страницы отдают `200`.
- **Операционная находка**: правка `nginx.freepdf.conf` (bind-mount
  одного файла) не подхватилась через `nginx -s reload` из-за смены
  inode — потребовался `docker restart 1pdf-nginx`. См. [GAPS.md](GAPS.md).
- **Не выполнено** (требует владельца продукта): верификация Google
  Search Console и Yandex.Webmaster — нужны токены из соответствующих
  аккаунтов, которых нет у агента. См. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

## Итог повторного исследования

Весь код перепрочитан заново, независимо от `TASK-0001`:
`admin/server.ts` (1435 строк), `admin/routes/editorApi.ts` (358 строк),
`admin/controllers/paymeController.ts`, `admin/utils/paymentUtils.ts`,
`prisma/schema.prisma` (196 строк) + все 5 файлов миграций целиком,
`docker-compose.yml`, `Dockerfile.admin.v2`, `nginx.freepdf.conf` целиком;
ключевые файлы `editor/src` (config.ts, cloudStore.ts,
useDocumentBootstrap.ts, SaveButton.tsx, App.tsx, drafts.ts) целиком, карта
API-вызовов остального фронтенда — через `grep`. Документация проекта
(`.md`-файлы внутри `/home/admin1/1pdf/`) не читалась ни разу — все факты
выведены только из кода и конфигурации. Код проекта не изменялся. Выполнено
несколько read-only команд для перепроверки состояния системы: `docker ps
-a`, `docker exec 1pdf-postgres psql -c '\d ...'`/`SELECT ... FROM
_prisma_migrations`, чтение host-level `/etc/nginx/sites-available/1pdf.uz`
— ничего не изменялось.

## Что подтверждено и работает согласованно (независимо перепроверено)

- Три слоя: **admin** (Express 4, `:8031`), **editor** (React 19 + Vite 8,
  собирается в статику и раздаётся тем же Express), **admin/public**
  (3 vanilla HTML-файла без сборки).
- Одна БД `onepdf` на PostgreSQL 16, 15 моделей/4 enum'а, **реальный**
  формат Prisma-миграций (`migration_lock.toml` + 5 папок,
  `prisma migrate deploy` выполняется автоматически при каждом старте
  контейнера) — качественно отличается от других проектов my_office.
- ~50 маршрутов API (admin + `/api/v1/*`) задокументированы построчно в
  [API.md](API.md).
- Два JWT-контура (`1pdf_token` для пользователей, `1pdf_admin_token` для
  админов) на одном `JWT_SECRET`, плюс отдельный `JWT_SECRET_OO` для
  legacy-интеграции.
- Реальное состояние Docker-хоста (2026-09-08, read-only) **полностью
  совпадает** с зафиксированным в TASK-0001 (2026-09-05): `1pdf-nginx`,
  `1pdf-admin`, `1pdf-postgres` — Up 2 недели; `onlyoffice-docs` — Exited,
  не управляется текущим `docker-compose.yml`.

## Новые находки этой задачи (уточняют/дополняют TASK-0001 конкретикой файл:строка)

1. **Дрейф схемы БД описан прежним аудитом качественно — в этой задаче
   подтверждён и локализован точно**: `grep` по всем 5 файлам миграций
   показал, что базовая `CREATE TABLE "PaymentConfig"` отсутствует вообще
   (есть только более поздний `ALTER TABLE ... ADD COLUMN alif_*`,
   `20260726000000_add_alif_payment_config/migration.sql:1-5`), а
   `PaymentAttempt` в исходной миграции содержит колонку `external_id`
   (`init_ecosystem/migration.sql:57`), тогда как в реальной БД она называется
   `external_payment_id` — переименование, никогда не проходившее через
   миграцию.
2. **Найдена рабочая форма с обращением к несуществующему API — и
   подтверждено через `git log -p`, что это регрессия, а не недострой**:
   `admin.html:1088,1097` вызывает `PATCH /api/auth/me/login` и
   `PATCH /api/auth/me/password` — оба маршрута реально существовали в
   `admin/server.ts` и были удалены коммитом `f1203b7` при рефакторинге
   auth-системы; фронтенд не был обновлён вслед за этим.
3. **`requirePro` продублирован независимо** в `admin/server.ts:282-288` и
   `admin/routes/editorApi.ts:92-101`.
4. **`VITE_KOFI_HANDLE` не переопределяется нигде в сборке** — donation-
   ссылка в собранном редакторе, по всей видимости, всё ещё указывает на
   автора оригинального open-source freePDF, не на 1PDF.
5. **`DocumentVersionStatus.UPLOADING`/`FAILED` и эндпоинт `.../complete`
   — готовая, но не используемая инфраструктура** — версии всегда
   создаются сразу `READY`, асинхронная загрузка не реализована ни в одном
   найденном месте фронтенда.
6. **`scripts/assert-migrations-additive.sh` — не общий guard**, а
   проверка одной захардкоженной миграции по пути в самом скрипте.
7. Полная построчная таблица ~50 маршрутов (admin + `/api/v1/*`) с методом/
   ролью/телом/ответом, карта фронтенд-компонентов → API — оба файла
   ([API.md](API.md), [VIEWS.md](VIEWS.md)) не существовали в таком виде
   после TASK-0001 (в её `AGENTS.md`-заметках упоминались только
   отдельные факты, не полная таблица).

## Что было переподтверждено без изменений (независимо, не унаследовано слепо)

Форк freePDF, сосуществование двух поколений редактирования документов,
статус `onlyoffice-docs` = `Exited`, наличие `test-payments.sh` с
похожими на реальные секретами, вынос TLS-терминации за пределы
репозитория — все переподтверждены независимым чтением кода/read-only
командами 2026-09-08.

## Какие документы созданы/изменены в рамках TASK-0002

**Новые файлы**: `docs/VIEWS.md`, `docs/CONFIGURATION.md`, `docs/GAPS.md`.

**Полностью переписаны** (заново, не дополнены поверх старого текста):
`docs/OVERVIEW.md`, `docs/ARCHITECTURE.md`, `docs/API.md`,
`docs/DATABASE.md`, `docs/DEPLOYMENT.md`, `docs/STACK.md`, этот файл.

**Обновлены**: `docs/OPEN_QUESTIONS.md` (новые пункты добавлены, старые
сохранены), `AGENTS.md` проекта (ссылка на актуальную задачу), `README.md`
(навигация на новые файлы).

## Открытые вопросы

Полный список — [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

## Следующие шаги (не выполнялись в рамках этой задачи)

Задача была строго документационной. Любые действия по факту находок
GAPS.md (починка/удаление `admin.html:1088,1097`, решение судьбы legacy
ONLYOFFICE-эндпоинтов и каталогов `*_oo`, восстановление недостающих
объектов схемы в истории миграций, ротация секретов в `test-payments.sh`)
требуют отдельной, явно поставленной задачи с владельцем продукта.
