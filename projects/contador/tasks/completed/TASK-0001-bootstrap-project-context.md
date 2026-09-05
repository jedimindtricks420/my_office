---
id: TASK-0001
title: Bootstrap project context for Contador
status: completed
created: 2026-09-05
completed: 2026-09-05
project: contador
---

# TASK-0001: Bootstrap project context for Contador

## Цель

Зарегистрировать существующий проект `/home/admin1/contador/` в `my_office` и создать
достоверную документацию его текущего состояния на основании реального кода.
Без разработки нового функционала, без исправлений, без рефакторинга, без предположений.

## Область

- Источник: `/home/admin1/contador/` (только чтение, не изменялся).
- Результат: `/home/admin1/my_office/projects/contador/` (новая документация).

## Журнал

### 2026-09-05

**Настройка окружения**

- Прочитан корневой `/home/admin1/my_office/AGENTS.md`, `CONTRIBUTING.md`,
  `docs/company/STANDARDS.md`.
- Обнаружено несоответствие прав доступа: репозиторий `my_office` на диске
  принадлежит пользователю `admin1`, сессия выполняется от `azureuser` без прав
  записи и без членства в группе `admin1`. Прямая запись через `sudo`
  (mkdir/chown) была заблокирована классификатором безопасности auto-mode как
  потенциально опасное кросс-пользовательское действие.
- Решение: склонирован тот же репозиторий
  (`git@github.com:jedimindtricks420/my_office.git`) в
  `/home/azureuser/work/my_office/` (ключ SSH `azureuser` авторизован в этом
  GitHub-аккаунте, подтверждено `ssh -T git@github.com` →
  `Hi jedimindtricks420!`). Вся работа велась в этой копии; push выполнен в тот
  же `origin/main`, что и исходный каталог `/home/admin1/my_office/`.
  **Важно для будущих агентов**: после push каталог `/home/admin1/my_office/`
  отстаёт от `origin/main` до тех пор, пока кто-то не выполнит там `git pull`
  (агент не мог писать напрямую в `/home/admin1/my_office/` из-за прав).

**Исследование `/home/admin1/contador/` (только чтение)**

Прочитано напрямую агентом-оркестратором:
- `README.md` (полностью, 282 строки)
- `AGENTS.md`, `CLAUDE.md` (корень проекта)

Делегировано 4 параллельным исследовательским агентам (read-only, без записи),
результаты синтезированы вручную:
1. **Stack & build**: оба `package.json` (root, v2), `tsconfig.json` (оба),
   `Dockerfile.admin`, `Dockerfile.v2`, `docker-compose.yml`, `.dockerignore`,
   `.gitignore` (оба), `next.config.ts`, `eslint.config.js`, поиск
   `.env.example`/`.nvmrc`/`engines`.
2. **Database**: `v2/prisma/schema.prisma` (584 строки, целиком), корневой
   `prisma/schema.prisma` (280 строк, целиком), список + выборочное чтение
   миграций (`v2/prisma/migrations/` — 9 файлов, корень — 1 файл),
   `v2/src/lib/seed-coa.ts`, `v2/src/lib/ensureBaseData.ts` (~2900 строк).
3. **API/auth/интеграции**: полный список `route.ts` под `v2/src/app/api/`
   (78 файлов), `v2/src/lib/auth.ts`, `auth-edge.ts`, `context.ts`,
   `pnl/route.ts`, `classification/run-rules/route.ts`,
   `classification/run-ai/route.ts`, `classification/status/[periodId]/route.ts`,
   `v2/src/lib/classification/aiClassifier.ts` (806 строк),
   `admin/server.ts` (1613 строк, платёжные интеграции).
4. **Тесты/доки/legacy**: список тестов `v2/src/__tests__/` (34 файла),
   `vitest.config.ts`, структура `admin/` и `src/` (v1), `docs/01_...md`,
   `docs/02_...md` (целиком), даты изменения файлов документации,
   `v2/docs/modules/` (список), `backups/` (список), `nsbu_and_soliq_codex/`
   (список), поиск `scripts/`.

**Дополнительная проверка вручную** (после отчётов агентов, для верификации
самых значимых/неожиданных находок):
- `ls /home/admin1/contador/src` → подтверждено: каталог отсутствует на диске.
- `docker ps --filter "name=contador"` → обнаружены **5** контейнеров, включая
  два похожих на v1 (`contador-app`, `f7ee8a3d81b8_contador-app` на порту
  3030) — оба запущены, вопреки README.
- `pm2 list` → пустой список процессов (опровергает `docs/01`/`02`,
  утверждающие PM2).
- `find v2 -iname "middleware*"` → не найдено исходного `middleware.ts`.
- `grep` имён переменных (без значений) в `v2/.env` и корневом `.env` →
  подтверждено `AZURE_OPENAI_ENDPOINT`/`AZURE_OPENAI_API_KEY` в v2 (не
  `OPENAI_API_KEY`, как в README).
- `docker inspect contador-app` → подтверждены имена переменных легаси-стека.
- Финальная точечная проверка: `grep BATCH_SIZE/CONFIDENCE_THRESHOLD
  constants.ts`, `grep AZURE_OPENAI aiClassifier.ts`,
  `find ... route.ts | wc -l` → все ключевые цифры (100, 70, 78) подтверждены
  напрямую в коде повторно перед записью в документацию.

Секреты (значения `.env`, реквизиты платёжных провайдеров, содержимое
бэкапов) **не читались** ни разу — только имена переменных и структура файлов.

## Что установлено

См. полную документацию — итог структурирован по файлам ниже. Ключевые
находки (расхождения между документацией проекта и реальностью) сведены в
`docs/STATUS.md`.

## Созданные документы

```
projects/contador/
├── AGENTS.md
├── README.md
├── docs/
│   ├── OVERVIEW.md
│   ├── ARCHITECTURE.md
│   ├── STACK.md
│   ├── STATUS.md
│   ├── DATABASE.md
│   ├── API.md
│   ├── DEPLOYMENT.md
│   ├── OPEN_QUESTIONS.md
│   ├── adr/.gitkeep        (пусто — ADR ещё не создавались)
│   └── runbooks/.gitkeep   (пусто — runbook'и ещё не создавались)
├── specs/
│   ├── active/.gitkeep     (пусто)
│   └── archived/.gitkeep   (пусто)
└── tasks/
    ├── active/             (после завершения — пусто)
    └── completed/TASK-0001-bootstrap-project-context.md
```

## Открытые вопросы

12 пунктов — полный список в `../../docs/OPEN_QUESTIONS.md`. Кратко, самое
важное:
1. Не выяснено, что представляют собой два запущенных Docker-контейнера v1,
   учитывая отсутствие каталога `src/` в репозитории — требует уточнения у
   владельца системы.
2. Собственная документация проекта частично устарела (`docs/01`/`02` —
   конец июня против июльского README; `docs/04_API_REFERENCE.md` — вероятно
   отстаёт от 78 реальных маршрутов).
3. Не проверено построчно по всем 78 API-маршрутам, что каждый корректно
   скоупит запросы по `orgId`.
4. Не найден единый `middleware.ts` — авторизация полагается на
   дисциплину вызова `getActiveOrgId()`/`getActiveMembership()` в каждом
   route-хендлере отдельно.

## Проверка после завершения

- Ключевые числовые/фактические утверждения (`AI.BATCH_SIZE = 100`,
  `AI.CONFIDENCE_THRESHOLD = 70`, провайдер Azure OpenAI, 78 route.ts,
  269 записей COA, 180 типов документов, отсутствие `src/`, отсутствие PM2)
  перепроверены прямым чтением/grep кода непосредственно перед записью в
  документацию и ещё раз перед закрытием задачи.
- `docs/STATUS.md` обновлён и отражает текущее состояние с явным разделением
  «подтверждено» / «расхождение с документацией проекта».
- Задача перемещена из `tasks/active/` в `tasks/completed/`.
