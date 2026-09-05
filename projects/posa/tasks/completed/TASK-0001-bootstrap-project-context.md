---
id: TASK-0001
title: Bootstrap project context for posa
status: completed
created: 2026-09-05
owner: azureuser (agent-assisted)
---

# TASK-0001 — Bootstrap project context

## Цель

Зарегистрировать существующий проект `posa` (источник: `/home/admin1/posa/`) в
`my_office` и создать документацию его подтверждённого текущего состояния.
Без разработки нового функционала, без исправления кода, без рефакторинга,
без домыслов там, где факт не подтверждён кодом/конфигурацией.

## Примечание по формулировке задачи

В исходном запросе пользователя проект был назван "Contador", но все указанные
пути (`/admin1/posa/`, `/admin1/my_office/projects/posa/`, имя файла задачи)
указывали на проект `posa`. Уточнено у пользователя явно — подтверждено, что
цель — `posa`. `Contador` — отдельный, уже зарегистрированный проект
(`projects/contador/`), не затронут этой задачей.

## Журнал работы

### 2026-09-05

- Прочитан корневой `/home/admin1/my_office/AGENTS.md`.
- Обнаружено: `my_office` на диске принадлежит `admin1`, `azureuser` не имеет
  прав записи напрямую. Работа ведётся через клон
  `/home/azureuser/work/my_office`, изменения публикуются через `git push`.
- Создана структура `projects/posa/{docs,specs,tasks}`.
- Начато исследование `/home/admin1/posa/` (репозиторий доступен на чтение).

### Что было просмотрено

- Верхний уровень `/home/admin1/posa/`: README.md, V3_CHECK.txt, migration.md,
  .gitignore, verify_migration.sh, test-payme-transaction.sh, структура
  каталогов (activation-system, my-pwa, docs, backups, db-backups).
- `activation-system/`: package.json, package-lock.json, Dockerfile,
  docker-compose.yml, docker-compose.override.yml, index.js (полностью),
  middleware/ (все 5 файлов), routes/ (все 14 файлов — извлечены все
  объявления маршрутов), controllers/ (список + authController.js и
  client/authController.js прочитаны подробно, payment/* — размеры и
  структура), services/ (список), utils/ (список), prisma/schema.prisma
  (полностью, 672 строки), prisma/migrations/ (список 28 файлов),
  .github/copilot-instructions.md, README-PWA.md, test-*.js в корне (4 шт.),
  activation-system/docs/ (список файлов).
- Проектная документация верхнего уровня `docs/`: PRD.md (интро),
  BUSINESS_LOGIC.md (интро) — использованы как контекст, не как источник
  истины по коду.
- Имена переменных окружения из `.env` (`grep -oE '^[A-Za-z_]...'`) — только
  имена ключей, значения не читались.
- `my-pwa/twa-manifest.json` — подтверждение, что это Bubblewrap TWA-обёртка;
  проверено, что `android.keystore` в `.gitignore` (не в git).
- Текущее состояние хоста (только чтение, без изменений): `docker ps -a`
  (контейнеры activation-system/activation-studio/activation-db и их порты),
  `/etc/nginx/sites-available/*` для всех доменов `*.namo.uz` (server_name,
  proxy_pass, TLS), включая полное содержимое `ppdev.namo.uz` (обнаружена
  Basic Auth защита перед Prisma Studio).
- `git log`/`git status`/`git remote` в `/home/admin1/posa/activation-system/`
  (118 коммитов, remote `POSA-v3`, есть незакоммиченные изменения — не
  трогались) и в `/home/admin1/my_office/` (для понимания процесса push).
- `git ls-files` + `git check-ignore` — подтверждено, что
  `test-payme-transaction.sh` (с зашитым похожим на боевой ключом Payme)
  отслеживается git, а `android.keystore` — нет.
- Точные подсчёты через `grep -c`: 36 моделей и 12 enum'ов в schema.prisma,
  185 объявлений маршрутов суммарно, 104 в adminRoutes.js, 40 файлов
  контроллеров (с разбивкой по подпапкам), 109 view-файлов, отсутствие
  jest/mocha/vitest в зависимостях.

### Что установлено (ключевые факты)

- Один Express-процесс на порту 4000 обслуживает все поддомены `*.namo.uz`,
  роль определяется по `Host` в middleware, а не отдельными сервисами.
- Сессии сотрудников — пароль+bcrypt; клиенты — телефон+OTP (Eskiz), без
  пароля; обе схемы используют одну и ту же PgSession-таблицу.
- Три платёжных провайдера (Click/Payme/Alif) как вебхуки, реквизиты
  хранятся per-кассе в таблице `Kassa`, а не только в `.env`.
- `routes/apiV1.js` не подключён в index.js — вероятно мёртвый/тестовый код.
- Нет автоматических тестов, нет линтера/TS.
- `activation-db` (Postgres) проброшен на `0.0.0.0:7440` — подтверждено
  сейчас, совпадает с прошлым аудитом от 2026-08-01.
- `ppdev.namo.uz` публично проксирует Prisma Studio, но защищено Basic Auth.
- `docs/PRD.md` устарел относительно кода (не описывает Dr.Web/ManaStore/
  Alif/QR-платежи/Kassa); упоминание Python/Pyrogram в
  copilot-instructions.md не подтверждается наличием кода в репозитории.

### Какие документы созданы

`projects/posa/{README.md, AGENTS.md}` и
`projects/posa/docs/{OVERVIEW,ARCHITECTURE,STACK,STATUS,DATABASE,API,
DEPLOYMENT,OPEN_QUESTIONS}.md`, плюс `.gitkeep` в `docs/adr/`,
`docs/runbooks/`, `specs/active/`, `specs/archived/`.

### Какие вопросы остались

Полный список — [`docs/OPEN_QUESTIONS.md`](../../docs/OPEN_QUESTIONS.md).
Коротко: судьба Python/Telegram-модуля, актуальность PRD, назначение
`routes/apiV1.js` и `manual_migration_qr_payment.sql`, Bootstrap vs Tailwind,
скрипты `bmad:*`, путаница v2/v3 в нейминге — не устранены и не требовали
устранения в рамках этой задачи (только документирование).

## Статус: завершена

Все пункты чек-листа задачи выполнены: код `/home/admin1/posa/` не изменялся,
документация основана только на подтверждённых фактах, неподтверждённое
вынесено в OPEN_QUESTIONS.md, STATUS.md обновлён.
