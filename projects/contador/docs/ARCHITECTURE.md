# Architecture — Contador

Снимок на 2026-09-05.

## Три приложения, две базы данных

```
                         PostgreSQL 16 (postgres:16-alpine, контейнер contador-db)
                         ├── contador       (легаси v1 + admin)
                         └── contador_v2    (основное приложение v2)

contador-v2 (Next.js, порт 3032, basePath /v2)  ──┐
                                                    ├── читают/пишут в соответствующую БД
contador-admin (Express, порт 3031, /admin/api)  ──┘   через отдельные Prisma-клиенты
```

- **v2** — основное приложение, Next.js 16 (App Router), TypeScript, схема
  `v2/prisma/schema.prisma` → БД `contador_v2`. Реализует учёт, отчётность,
  импорт, AI-классификацию, закрытие периода.
- **admin** — Express-приложение (`admin/server.ts`, единый файл 1613 строк, без
  разбиения на routes/models/middleware), схема `prisma/schema.prisma` (корень)
  → БД `contador`, но при этом генерирует **и** клиент к `v2/prisma/schema.prisma`
  (см. `Dockerfile.admin`) — то есть имеет доступ к обеим базам. Обрабатывает
  платежи (Payme/Click/Alifpay), подписки, пользователей/организации,
  административные операции.
- **v1 (legacy)** — по README не развивается, порт 3030, использует ту же
  корневую схему/БД `contador`, что и admin. Исходный каталог `src/`
  физически отсутствует в текущем репозитории (см.
  [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)), при этом на хосте подтверждены
  запущенные Docker-контейнеры, похожие на v1 — см.
  [DEPLOYMENT.md](DEPLOYMENT.md).

## Структура `v2/src/` (основное приложение)

```
v2/
├── prisma/schema.prisma        # схема БД contador_v2 (20 моделей, 17 enum'ов)
├── prisma/migrations/          # сырые SQL-миграции (9 файлов)
├── scripts/generate-document-types-doc.ts   # генератор docs/DOCUMENT_TYPES.md
├── src/
│   ├── app/
│   │   ├── api/                # 78 файлов route.ts — см. docs/API.md
│   │   ├── closing/            # UI мастера закрытия месяца (7 шагов)
│   │   ├── pnl/, reports/balance/   # UI Форма №2 / Форма №1
│   │   ├── dashboard/, transactions/, settings/, ...
│   ├── lib/
│   │   ├── posting/            # движок проводок (postingEngine.ts)
│   │   ├── classification/     # правила + AI-классификатор (aiClassifier.ts)
│   │   ├── reports/             # OSV, журнал, карточка счёта, субконто
│   │   ├── closing.ts          # finalizePeriod()
│   │   ├── constants.ts        # коды счетов, налоговые ставки, AI.MODEL/BATCH_SIZE
│   │   ├── seed-coa.ts         # 269 счетов НСБУ №21 (план счетов)
│   │   ├── ensureBaseData.ts   # сидинг счетов + 180 типов документов при старте
│   │   ├── context.ts          # getSession/getActiveOrgId/getActiveMembership
│   │   ├── auth.ts, auth-edge.ts  # JWT (jose), cookie v2_session
│   │   └── taxReport/constants.ts  # DEFAULT_TAX_REPORT_MAPPING
│   └── __tests__/              # 34 файла тестов (vitest), плоская директория
└── docs/modules/                # техническая документация по модулям (00-K)
```

Полный список из 78 API-маршрутов и группировка по функциональным папкам — в
[API.md](API.md). Полная схема данных — в [DATABASE.md](DATABASE.md).

## Аутентификация — архитектурная точка зрения

Stateless: JWT в HTTP-only cookie (`v2_session`), подпись/проверка через
`jose`. Нет отдельного Next.js `middleware.ts` (не найден в исходниках) —
проверка сессии и `orgId`-скоупинга происходит на уровне каждого route-хендлера
через общий модуль `context.ts`. Подробности и оговорки о неполной проверке
всех 78 маршрутов — в [API.md](API.md).

## Конвейер обработки транзакций (по README + структуре кода)

```
Импорт выписки (import/) → StagedTransaction
        │
        ▼
Классификация: движок правил (classification/run-rules)
    → при отсутствии совпадения: AI-классификатор (classification/run-ai, Azure OpenAI)
        │
        ▼
Очередь уточнений (clarification/) — ответ пользователя может создавать новое Rule
        │
        ▼
Document (проведён через postingEngine.ts, шаблон из DocumentType.postingTemplate)
        │
        ▼
JournalEntry (двойная запись) → отчёты (pnl/, reports/balance/, reports/osv, ...)
```

## Три платёжных провайдера (admin)

Payme, Click, **Alifpay** (последний подтверждён кодом `admin/server.ts`, не
упомянут явно как реализованный в README — см. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)
и [API.md](API.md)). Реквизиты провайдеров хранятся в таблице БД
`PaymentConfig` (легаси-схема), не только в переменных окружения.

## Известные архитектурные ограничения (задокументированы в README проекта,
## не проверялись отдельно, приводятся как заявленные разработчиками факты)

- Движок проводок (`postingEngine.ts`) не поддерживает забалансовые
  (односторонние) записи — счета 001/002/007 сознательно не реализованы.
- Нет складского измерения в модели данных — перемещения между складами не
  поддерживаются.
- Форма №2 не использует периодический метод учёта ТМЗ (счета 9140/9150) —
  только perpetual-метод через счёт 2910; счёт 9220 (выбытие прочих активов,
  не ОС) также не задействован ни одним типом документа.
