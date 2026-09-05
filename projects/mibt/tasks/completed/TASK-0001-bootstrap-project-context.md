---
id: TASK-0001
title: Bootstrap project context for mibt
status: completed
created: 2026-09-05
owner: azureuser (agent-assisted)
---

# TASK-0001 — Bootstrap project context

## Цель

Зарегистрировать существующий проект `mibt` (источник: `/home/admin1/mibt-2/`)
в `my_office` и создать документацию его подтверждённого текущего состояния.
Без разработки нового функционала, без исправления кода, без рефакторинга,
без домыслов там, где факт не подтверждён кодом/конфигурацией.

## Журнал работы

### 2026-09-05

- Прочитан корневой `/home/azureuser/work/my_office/AGENTS.md`.
- Подтверждено: `my_office` на диске принадлежит `admin1`, `azureuser`
  работает через клон `/home/azureuser/work/my_office`, публикация — через
  `git push`. `git status`/`git fetch` в клоне — чисто, синхронизирован с
  `origin/main`.
- Создана структура `projects/mibt/{docs/{adr,runbooks},specs/{active,archived},tasks/{active,completed}}`.
- Проведено исследование `/home/admin1/mibt-2/` (каталог принадлежит
  `azureuser`, доступен на чтение без sudo).

### Что было просмотрено

- Верхний уровень: отсутствие README, `package.json` (полностью),
  `package-lock.json` (только `lockfileVersion`), `.gitignore`, `.dockerignore`,
  `.env`/`.env.example` (только имена переменных через `grep -oE`, значения
  не читались; подтверждено идентичное множество ключей в обоих файлах),
  `Dockerfile` (полностью), `docker-compose.yml` (полностью),
  `docker-compose.dev.yml` (полностью), `nginx.conf` (полностью),
  `index.js` (полностью), `prisma.config.ts` (полностью).
- `prisma/schema.prisma` — прочитан полностью (401 строка, 18 моделей,
  9 enum'ов, подсчитано `grep -c`); `prisma/migrations/` — список 3 миграций
  по именам файлов (SQL не читался построчно); `prisma/seed.js` — прочитан
  частично (первые ~30 строк, назначение admin-пользователя и дефолтных
  системных настроек).
- Структура каталогов целиком: `routes/` (13 файлов, дерево + все объявления
  маршрутов через `grep`), `controllers/` (обнаружено — полностью пустой
  скелет), `services/` (список + полностью прочитаны `license.service.js`,
  `audit.service.js`, `balance.service.js`, `exchangeRate.service.js`,
  `order.service.js`, `pdf.service.js`; частично — `theme.service.js`,
  `email.service.js` только список экспортов), `middleware/` (все 5 файлов,
  `auth.js` и `validate.js` — полностью), `lib/` (список, не читались
  построчно), `views/` (список всех 35 `.ejs`, языковой атрибут проверен
  в `layouts/main.ejs` — `lang="ru"`), `i18n/` и `templates/` — обнаружены
  полностью пустыми.
- Отдельные файлы роутов прочитаны полностью: `auth.routes.js`,
  `account.routes.js`, `cart.routes.js`, `quotes.routes.js`,
  `documents.api.js`, `auth.api.js`; частично (через `grep`/выборочные
  фрагменты) — `admin.routes.js`, `catalog.routes.js`,
  `routes/api/admin/index.js` (840+ строк — прочитаны конкретные фрагменты:
  наполнение пула лицензий, генерация спецификации, approve/reject заявок на
  баланс; полный список маршрутов извлечён через `grep`).
- Точные подсчёты маршрутов через `grep -E "[A-Za-z]+Routes\.(get|post|put|patch|delete)\("`
  по каждому файлу (первая попытка через `router\.` и общий `\.(get|post...)\(`
  дала неверные числа из-за иного именования переменных роутеров и ложных
  срабатываний на `prisma.model.delete(...)` — исправлено уточнением
  паттерна). Итог: 77 маршрутов суммарно, 29 из них в `routes/api/admin/index.js`.
- `grep` по всему дереву на предмет вызовов `assignLicensesToOrder(` —
  подтверждено, что функция импортируется, но нигде не вызывается (мёртвый
  код относительно текущего бизнес-флоу).
- `docs/` внутри `/home/admin1/mibt-2/`: `CHANGELOG_KEY_DELIVERY_REFACTOR.md`
  и `TZ-balance-topup-fix.md` прочитаны (начало каждого, ключевые разделы) —
  сопоставлены с кодом (`order.service.js`, `middleware/validate.js`,
  `routes/api/admin/index.js` reject-эндпоинт) и подтверждено, что описанные
  в них изменения/исправления присутствуют в текущем коде.
- Верхнеуровневые `.md` (`PRD.md`, `ARCHITECTURE.md`, `BUSINESS_LOGIC.md`,
  `USER_FLOWS.md`, `BUILD_SPEC.md`) — не читались построчно, только
  проверены даты изменения файлов (`ls -la`, все от 2026-06-08) в сравнении
  с датами файлов в `docs/` (2026-06-10 — 2026-06-12) для установления факта
  расхождения по времени.
- Текущее состояние хоста (только чтение, без изменений и перезапусков):
  `docker ps -a --filter name=mibt` (4 контейнера, все Up 2 недели),
  `/etc/nginx/sites-available/portal.mibt.uz` (полностью),
  `ls /etc/nginx/sites-enabled/` (подтверждено, что сайт включён),
  `sudo certbot certificates` (срок действия сертификата portal.mibt.uz и
  ppdev.namo.uz), `find backups/` (список файлов, подтверждена свежесть
  ежедневных бэкапов вплоть до сегодняшнего дня).
- Проверено отсутствие `.git` в `/home/admin1/mibt-2/` (`git status` →
  `fatal: not a git repository`).
- Проверено отсутствие CI (`.github/workflows`) и тестов (`*.test.js`,
  `*.spec.js`, зависимости `jest`/`mocha`/`vitest`/`supertest`).

### Что установлено (ключевые факты)

- Один Express-процесс обслуживает и SSR (EJS), и JSON API; логика — в
  `routes/**`, каталог `controllers/` — пустой неиспользуемый скелет.
- Пул лицензий `ProductLicense`/`assignLicensesToOrder` — мёртвый код;
  реальная выдача ключей — вручную, по одной позиции заказа
  (`OrderLineLicense`), что подтверждается и внутренним changelog проекта.
- Два типа клиента: `CORPORATE` (баланс с заморозкой/списанием) и `BUDGET`/
  B2G (постоплата, баланс не используется).
- CSRF-защиты нет; тестов и CI нет; проект не под git.
- Прод стабильно работает (4 контейнера, 2 недели uptime, свежие бэкапы),
  при этом код не менялся ~3 месяца (по mtime) — разработка приостановлена,
  система в поддерживаемой, но не активно развиваемой эксплуатации.
- Верхнеуровневая документация проекта (`PRD.md` и др.) старше ряда
  задокументированных изменений кода — вероятно частично устарела
  (построчно не сверялось).

### Какие документы созданы

`projects/mibt/{README.md, AGENTS.md}` и
`projects/mibt/docs/{OVERVIEW,ARCHITECTURE,STACK,STATUS,DATABASE,API,
DEPLOYMENT,OPEN_QUESTIONS}.md`, плюс `.gitkeep` в `docs/adr/`,
`docs/runbooks/`, `specs/active/`, `specs/archived/`.

### Какие вопросы остались

Полный список — [`docs/OPEN_QUESTIONS.md`](../../docs/OPEN_QUESTIONS.md).
Коротко: судьба пула `ProductLicense`, актуальность верхнеуровневых
`.md`-документов проекта относительно кода, назначение переменной
`ALLOWED_ORIGINS` (используется в коде, но не задокументирована в
`.env.example`), назначение второго nginx (`mibt2-nginx`/3011), происхождение
правила `.git` в `.dockerignore` при отсутствии git-репозитория — не
устранялись и не требовали устранения в рамках этой задачи (только
документирование).

## Статус: завершена

Все пункты чек-листа задачи выполнены: код `/home/admin1/mibt-2/` не
изменялся и не перезапускался; документация основана только на
подтверждённых фактах; неподтверждённое и расхождения вынесены в
OPEN_QUESTIONS.md; STATUS.md обновлён по итогам повторной сверки ключевых
утверждений с кодом (email.service.js — 14 экспортов, `lang="ru"` в
layout, точный пересчёт маршрутов).
