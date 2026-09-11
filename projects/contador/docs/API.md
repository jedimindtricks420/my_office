# API — Contador

Полностью переписано 2026-09-08 (`TASK-0002`), построчно прочитаны все 78
файлов `route.ts` под `v2/src/app/api/`, `admin/server.ts` целиком (1613
строк) и оба тестовых shell-скрипта корня репозитория. Базовый путь v2 —
`/v2/api/...` (`next.config.ts: basePath: "/v2"`, реальный HTTP-путь = `/v2`
+ путь файла относительно `app/`).

Поправки 2026-09-11 ниже описывают локальный TASK-0005, не production.
Числа файлов в заголовках и непересмотренные строки относятся к снимку
8 сентября. Это не новая полная проверка всех маршрутов.
Доказательства и незавершённые контракты: [PROGRESS](PROGRESS-2026-09-11.md).
Добавлены также маршруты архива Soliq `GET /v2/api/import/soliq`,
`GET /v2/api/import/soliq/[batchId]/export` и
`POST /v2/api/import/soliq/[batchId]/discard`; контракт и проверки доступа
описаны в [ADR-0005](adr/ADR-0005-soliq-server-batch.md).

## Аутентификация — общий механизм v2

Stateless JWT (`jose`, HS256) в httpOnly cookie `v2_session`
(`SESSION_MAX_AGE_SECONDS` = 7 дней). Каждый обработчик вызывает один из
хелперов `lib/context.ts`:
- `getSession()` — только проверка JWT, бросает `"UNAUTHORIZED"`.
- `getActiveOrgId()` — вызывает `getActiveMembership()` и возвращает orgId
  подтверждённого текущего членства; старого доверия только JWT больше нет.
- `getActiveMembership()` — реальный `prisma.orgMember.findFirst` по userId
  и activeOrgId; ошибки `"UNAUTHORIZED"`, `"NO_ACTIVE_ORG"`, `"FORBIDDEN"`.
  Проверка членства сама по себе не ограничивает бухгалтерскую роль.

В таблицах ниже колонка «Auth» указывает, какой из хелперов вызывается первым
(это не гарантирует, что каждый последующий Prisma-запрос в обработчике
фильтрует по `orgId` — построчно проверено для показанных ниже примеров, не
для всех 78 файлов целиком, см. [GAPS.md](GAPS.md)).

---

## v2 — auth/ (5 файлов, все публичные)

| Метод | Путь | Auth | Тело/параметры | Ответ |
|---|---|---|---|---|
| POST | `/v2/api/auth/login` | публичный | `{email,password}` | `200 {ok:true}` + cookie; `401` неверные креды |
| POST | `/v2/api/auth/register` | публичный | `{email,password,name?,orgName,inn?}`; email-regex, пароль ≥8 символов | `201` создаёт User+Organization+OrgMember(OWNER)+Period(текущий)+Subscription(FREE) одной транзакцией + cookie |
| POST/GET | `/v2/api/auth/logout` | публичный | — | редирект на `/v2/login`, удаляет cookie |
| POST | `/v2/api/auth/forgot-password` | публичный | `{email}` | всегда `200 {success:true}` (защита от enumeration); создаёт `PasswordResetToken` (TTL 1ч), письмо через `mailer.ts` |
| POST | `/v2/api/auth/reset-password` | публичный | `{token,password}` (≥8 симв.) | сброс пароля + инвалидация токена |
| GET | `/v2/api/auth/reset-password?token=` | публичный | — | `{valid:boolean}` — проверка токена без расходования |

⚠️ Ни один из этих маршрутов не обёрнут в rate-limiter (в отличие от admin,
где `express-rate-limit` стоит на всех публичных и защищённых путях) —
`/v2/api/auth/login` технически не ограничен по количеству попыток на уровне
приложения — см. [GAPS.md](GAPS.md).

## v2 — user/, org/, orgs/ (7 файлов)

| Метод | Путь | Auth | Роль | Тело/параметры | Ответ |
|---|---|---|---|---|---|
| PATCH | `/v2/api/user/active-org` | getSession | член орг. | `{orgId}` | переиздаёт JWT с новым `activeOrgId` |
| GET | `/v2/api/user/me` | getSession | — | — | профиль + все членства |
| GET | `/v2/api/org/members` | getActiveMembership | любая | — | список участников активной орг. |
| POST | `/v2/api/org/members/invite` | getActiveMembership | OWNER/ADMIN | `{email,role}` (ACCOUNTANT/ADMIN) | создаёт User (temp-пароль) если новый, добавляет в OrgMember; письмо через SMTP или `mockInvitePassword` в dev без SMTP |
| DELETE | `/v2/api/org/members/[userId]` | getActiveMembership | OWNER/ADMIN | — | нельзя удалить себя; последнего OWNER удаляет только другой OWNER |
| POST | `/v2/api/orgs` | getSession | — | `{name,inn?}` | создать доп. организацию — **требует активную PRO-подписку хотя бы на одной из уже существующих организаций** пользователя |
| DELETE | `/v2/api/orgs/[orgId]` | getSession | OWNER этой орг. | — | каскадное удаление (вручную, в FK-безопасном порядке); нельзя удалить последнюю организацию пользователя |

## v2 — periods/, bank-accounts/, accounts/ (9 файлов)

| Метод | Путь | Auth | Параметры | Ответ |
|---|---|---|---|---|
| GET/POST | `/v2/api/periods` | getActiveOrgId | POST: `{year,month,mode?}` | список / создание (upsert по факту — при существующем возвращает его) |
| DELETE | `/v2/api/periods/[id]` | текущее членство, OWNER/ADMIN | — | только пустой незаблокированный OPEN без истории/зависимостей; транзакция и аудит, без каскадной очистки учёта |
| POST | `/v2/api/periods/[id]/close` | текущее членство | force-обход удалён | блокировка периода и проверки готовности; наличие параметра force не разрешает обход |
| POST | `/v2/api/periods/[id]/reopen` | текущее членство | — | атомарная очистка/аудит; последующий CLOSED или зависимое погашение блокирует операцию |
| GET/POST | `/v2/api/bank-accounts` | текущее членство; POST OWNER/ADMIN | POST: строгая схема, 20-значный номер, денежные строки | нормализация номера/проверка дублей под org-lock; ADR-0011/0012 |
| PUT/DELETE | `/v2/api/bank-accounts/[id]` | текущее членство, OWNER/ADMIN | PUT: строгая частичная схема | использованные финансовые реквизиты защищены; DELETE только пустого нулевого счёта без истории; транзакция/аудит |
| GET | `/v2/api/bank-accounts/[id]/stats` | getActiveOrgId | — | `{transactionCount, periodCount}` |
| GET | `/v2/api/accounts` | getActiveOrgId | `?search=&showHidden=` | глобальный план + иерархия/isHidden; journalEntries/openItems counts и usageCount только своей организации; доступ 401/403 |
| PATCH | `/v2/api/accounts/[id]/visibility` | getActiveOrgId | `{isHidden}` | пишет в `Organization.settings.hiddenAccounts` (JSON, не отдельная таблица) |

## v2 — documents/, document-types/, transactions/ (7 файлов)

| Метод | Путь | Auth | Параметры | Ответ |
|---|---|---|---|---|
| GET | `/v2/api/documents` | getActiveMembership | `?periodId&typeId&status&page&limit(≤100)` | реестр документов, пагинация |
| POST | `/v2/api/documents` | getActiveMembership + бухгалтерская роль | `{typeId,periodId,date,payload}` | строгий запрос, tenant/дата/период и запрет системных типов, включая YEAR_END_CLOSE; ещё не схема первички для каждого типа |
| GET | `/v2/api/document-types` | getActiveOrgId | `?bankOnly=true` (фильтр `mode in [BANK_AUTO,HYBRID]`) | справочник 180 типов |
| GET | `/v2/api/transactions` | getActiveOrgId | `?status&periodId&accountId&direction&categoryId&search&page&limit(≤2000)` | реестр StagedTransaction |
| PATCH | `/v2/api/transactions/[id]/category` | getActiveMembership | `{documentTypeId,createRule?}` | создаёт/репостит Document; блок закрытого периода, проверка направления, row-lock от гонок |
| PATCH | `/v2/api/transactions/[id]/skip` | getActiveMembership | — | тумблер SKIPPED↔IMPORTED, гасит проводки при переходе в SKIPPED |
| GET | `/v2/api/transactions/by-counterparty` | getActiveOrgId | `?inn|hint&direction?&excludeId?` | другие операции того же контрагента (для массового уточнения) |

## v2 — import/ (3 файла)

| Метод | Путь | Auth | Тело | Ответ |
|---|---|---|---|---|
| POST | `/v2/api/import/bank` | getActiveMembership + бухгалтерская роль | `multipart/form-data {file,bankAccountId,parserType?}`, `?preview=true` | только проверенный 1C; Excel/неизвестный 422; тело 6 МиБ/15 с, файл 5 МиБ, 1000 операций; итоги/остаток/архив атомарны, расхождение откатывает запись |
| DELETE | `/v2/api/import/bank/rollback` | getActiveMembership, OWNER/ADMIN | `{batchId}` | только нетронутые строки и OPEN-периоды, совпадающий snapshot v2, без зависимых изменений; новый архив сохраняется как ROLLED_BACK; неполные/старые snapshots 409 |
| GET | `/v2/api/import/bank/batches` | getActiveMembership, включая VIEWER | `?bankAccountId&cursor?` | `{batches,nextCursor}`, 20 метаданных IMPORTED/ROLLED_BACK, порядок createdAt/id DESC; чужой банк/cursor 404 |
| GET | `/v2/api/import/bank/batches/[batchId]/export` | getActiveMembership, включая VIEWER | `?format=source\|protocol` (default protocol) | исходные байты или JSON bank-audit-v1; tenant 404, хеш/размер 409 BANK_SOURCE_INTEGRITY; attachment/no-store/nosniff |
| POST | `/v2/api/import/soliq` | текущее членство + проверка роли | `multipart/form-data {file,periodId}` | серверный batch с оригиналом/хешем/контролями; разные ИНН не считаются автоматическим совпадением; completion не доверяет суммам клиента |

## v2 — classification/, clarification/, rules/ (11 файлов)

| Метод | Путь | Auth/тариф | Тело/параметры | Ответ |
|---|---|---|---|---|
| POST | `/v2/api/classification/run-rules` | getActiveOrgId | `{periodId}` | синхронный прогон `applyRules` |
| POST | `/v2/api/classification/run-ai` | getSession + **PRO-тариф обязателен** | `{periodId?}` (`ALL` если не задан) | запускает фоновую `ClassificationJob`, возвращает `jobId` немедленно |
| GET | `/v2/api/classification/status/[periodId]` | getActiveOrgId | — | прогресс job; 403 при чужом `orgId` |
| POST | `/v2/api/classification/ai-reconcile` | getSession + **PRO** | `{bankOnly[],soliqOnly[]}` (≤200 каждый) | вызов Azure OpenAI для сопоставления авансов↔ЭСФ; проверяет принадлежность `bankOnly.id` организации |
| GET | `/v2/api/clarification/queue` | getActiveOrgId | `?periodId` | группировка неклассифицированных по ИНН/hint/описанию |
| POST | `/v2/api/clarification/answer` | getActiveMembership | `{transactionIds[],documentTypeId,createRule?,ruleMatchType?,ruleMatchValue?,extraPayload?}` | массовая классификация с построчным row-lock, опциональное создание Rule |
| GET/POST | `/v2/api/rules` | getActiveOrgId | POST: `{matchType,matchValue,categoryId,direction?}` | — |
| PUT/DELETE | `/v2/api/rules/[id]` | getActiveOrgId | — | — |
| POST | `/v2/api/rules/[id]/apply` | getActiveOrgId | `{transactionIds[]}` | применяет одно правило к явному списку транзакций |
| GET | `/v2/api/rules/preview` | getActiveOrgId | `?matchType&matchValue&direction?` | какие транзакции совпадут (до сохранения правила) |
| PATCH | `/v2/api/rules/reorder` | getActiveOrgId | `{ids[]}` | массовое обновление `order` через `prismaWithOrg` |

## v2 — posting/, open-items/ (6 файлов)

| Метод | Путь | Auth | Тело | Ответ |
|---|---|---|---|---|
| POST | `/v2/api/posting/post` | getActiveMembership | `{documentId}` | ручной вызов `postDocument`, документ должен принадлежать активной организации |
| POST | `/v2/api/posting/repost` | getActiveMembership | `{documentId,newDocumentTypeId}` | void + смена типа + повторный posting |
| POST | `/v2/api/posting/void` | getActiveMembership | `{documentId}` | гасит проводки, возвращает связанную StagedTransaction в NEEDS_CLARIFICATION |
| GET | `/v2/api/posting/revisions` | getActiveMembership | `?documentId&cursor?` | до 20 метаданных, revision DESC, nextCursor; только своя организация, без загрузки снимков |
| GET | `/v2/api/posting/revisions/[revisionId]/export` | getActiveMembership | — | posting-revision-v1: заголовок, snapshotHash, строка snapshotCanonical; проверка SHA-256, фиксированное имя JSON-файла |
| GET | `/v2/api/open-items` | getActiveOrgId | `?status&accountCode&periodId&search` | сначала `markRiskyItems()` (пересчёт RISK), затем список + агрегаты по счетам |
| PATCH | `/v2/api/open-items/[id]/close` | текущее членство | — | ручное закрытие отключено; погашение только документом |
| POST | `/v2/api/open-items/[id]/reopen` | текущее членство | — | ручное открытие отключено; требуется отмена документа расчёта |

Новые архивные GET доступны всем фактическим ролям OWNER/ADMIN/ACCOUNTANT;
VIEWER в текущей схеме нет. Живой документ не требуется, но членство проверяется
заново, включая после удаления организации. Нет истории своей организации:
200 с пустым списком, одинаково для чужого и отсутствующего documentId.
Чужой/отсутствующий курсор или revisionId: 404; malformed id: 400;
нет сессии: 401; нет членства/активной организации: 403; неверный хеш: 409;
внутренняя ошибка: 500 без деталей. Все ответы private/no-store, nosniff.
Курсор принадлежит тому же документу и организации, сортировка не по createdAt.
Хеш экспорта считается по UTF-8 байтам snapshotCanonical, не по JSON-объекту
после повторной сериализации. Контракт и ограничения:
[ADR-0016](adr/ADR-0016-posting-revision-archive.md).

## v2 — closing/ (7 файлов) — мастер закрытия месяца

| Метод | Путь | Auth | Тело/параметры | Ответ |
|---|---|---|---|---|
| GET | `/v2/api/closing/[periodId]/state` | getActiveOrgId | — | текущее состояние ClosingJob; явные нули не заменяются начислениями прошлого периода |
| GET/DELETE | `/v2/api/closing/[periodId]/accruals` | getActiveOrgId | DELETE сбрасывает accruals-часть состояния | — |
| GET/POST | `/v2/api/closing/[periodId]/pending-invoices` | getActiveOrgId(+getUser) | POST: `{openItemId,receiptKind?}` | подтверждение постоплатного счёта по открытой позиции (6310/4310) |
| POST | `/v2/api/closing/[periodId]/step/[stepNumber]/complete` | текущее членство + проверка роли | шаги 4/5: строгие суммы/курс; шаг 6: `{batchId,decisions}` | данные/прогресс атомарны; шаг 6 загружает серверный источник, сверяет проводки и сохраняет протокол; старый parsedPayload не является источником сумм |
| POST | `/v2/api/closing/[periodId]/finalize` | getActiveOrgId(+getUser) | `{confirmMissingCogs?}` | вызывает `finalizePeriod()`; `409 MISSING_COGS`, если есть выручка 9020 без списания GOODS_SOLD и не подтверждено явно |
| POST | `/v2/api/closing/year-end` | getActiveOrgId | `{periodId}` | перенос 9910→8710, только для закрытого декабря, advisory-lock от дублей |
| GET | `/v2/api/closing/year-end/status` | getActiveOrgId | `?periodId` | `{done: boolean}` |

## v2 — reports/, pnl/, cashflow/, dashboard/, cbu-rate/ (16 файлов)

| Метод | Путь | Auth | Параметры | Ответ |
|---|---|---|---|---|
| GET | `/v2/api/pnl` | getActiveOrgId | `?from&to` | Форма №2 построчно (line010…line270) + помесячный разрез |
| GET | `/v2/api/reports/balance` | getActiveOrgId | `?to` | Форма №1 построчно (line010…line780) + `balanceCheck`/`difference` |
| GET | `/v2/api/reports/profit-tax` | getActiveOrgId | `?year&quarter` | квартальный «Расчёт налога на прибыль» — основная форма + Прил.1/2 + статусы 9 приложений |
| GET | `/v2/api/reports/profit-tax/line-details` | getActiveOrgId | `?year&quarter&appendix&line` | drill-down: документы, формирующие строку |
| GET | `/v2/api/reports/osv` | getActiveMembership | `?from&to&expandSubconto?` | оборотно-сальдовая ведомость |
| GET | `/v2/api/reports/journal` | getActiveMembership | `?from&to&accountId?&documentTypeId?&counterpartyId?` | журнал проводок |
| GET | `/v2/api/reports/account-card` | getActiveMembership | `?accountId&from&to` | карточка счёта |
| GET | `/v2/api/reports/account-analysis` | getActiveMembership | `?accountId&from&to` | анализ счёта (обороты с корреспондирующими счетами) |
| GET | `/v2/api/reports/subconto` | getActiveMembership | `?from&to&accountId?` | по субконто (контрагент/договор) |
| GET | `/v2/api/reports/accounts` | getActiveOrgId | — | справочник счетов для селектов отчётов |
| GET | `/v2/api/reports/counterparties` | getActiveMembership | — | справочник контрагентов организации |
| GET | `/v2/api/cashflow` | getActiveOrgId | `?from&to&accountId?` | даты Asia/Tashkent; конкретный банк по источнику в RepeatableRead, чужой 404/неоднозначный 409; полный валютный ДДС ещё не реализован |
| GET | `/v2/api/cbu-rate` | getActiveOrgId | — | курс USD с `cbu.uz` (внешний вызов, кэш `revalidate: 3600`) |
| GET | `/v2/api/dashboard` | getActiveOrgId | `?periodId?` | KPI, чек-лист закрытия, риск-позиции, ближайшие налоги |

## v2 — settings/ (6 файлов)

| Метод | Путь | Auth | Роль | Тело | Ответ |
|---|---|---|---|---|---|
| GET/PATCH | `/v2/api/settings/org` | getActiveOrgId(+Membership на PATCH) | PATCH: OWNER/ADMIN | реквизиты, налоговый режим, льгота IT Park, среднесписочная численность | — |
| GET/POST | `/v2/api/settings/charter-capital` | getActiveOrgId(+Membership) | POST: OWNER/ADMIN | `{amount,fundingType,paidAmount?,fundedAccountCode?,confirmedRegistration?,registrationDate?}` | первая декларация или увеличение (уменьшение — запрещено API, требует ручной проводки) |
| GET/POST | `/v2/api/settings/costing-method` | getActiveOrgId(+Membership) | POST: OWNER/ADMIN | `{fiscalYear,costingMethod}` | фиксация на год, смена — только после полного закрытия предыдущего года |
| GET/PATCH | `/v2/api/settings/open-item-deadlines` | getActiveOrgId | — | `{код_счёта: дни}` | список счетов выводится динамически из `getOpenItemBufferAccountCodes()` |
| GET/POST/PUT/DELETE | `/v2/api/settings/tax-deadlines` | getActiveOrgId | — | шаблоны налоговых сроков (`TaxDeadlineTemplate`) | |
| GET | `/v2/api/settings/opening-balance` | getActiveMembership | любая | — | `{lines:[]}` либо дата/точные строки/итоги/documentId/readOnly; несколько POSTED балансов 409 OPENING_BALANCE_REQUIRES_REVIEW |
| POST | `/v2/api/settings/opening-balance` | getActiveMembership | OWNER/ADMIN/ACCOUNTANT | строгие `{date,lines:[{accountCode,debit,credit}]}`, 2–500 строк | 201 `{id,totals}` только первично, соответствующий OPEN-месяц; точные равные итоги/допустимые счета, атомарный аудит; нет периода/неверный ввод 400, история/закрытие/конфликт 409; ADR-0015 |

## v2 — payments/ (4 файла, проксируют admin)

| Метод | Путь | Auth | Тело | Ответ |
|---|---|---|---|---|
| GET | `/v2/api/payments/history` | getSession | — | платежи всех организаций пользователя |
| GET | `/v2/api/payments/subscription` | getSession | — | статус подписки (`getUserActivePro`) + цена (проксирует `ADMIN_API_URL`, деградирует на дефолтную цену при недоступности admin) |
| POST | `/v2/api/payments/initiate` | getActiveOrgId | `{provider: PAYME|CLICK|ALIF}` | проксирует `POST {ADMIN_API_URL}/admin/api/v2/payments/initiate` |
| POST | `/v2/api/payments/voucher` | getActiveOrgId | `{code}` | проксирует `POST {ADMIN_API_URL}/admin/api/v2/vouchers/redeem` |

---

## admin (`admin/server.ts`, порт 3031) — ~62 маршрута в одном файле

Два роутера: `publicRouter` (только `express-rate-limit`, 50 req/15мин, БЕЗ
пароля) под `/admin/api/*`, и `router` (`adminLimiter` 100 req/15мин + общий
пароль `x-admin-password === ADMIN_PASSWORD`) — оба монтируются на один
префикс `/admin/api`, порядок монтирования (`publicRouter` первым) определяет,
какие пути защищены.

### Публичные (`publicRouter`, `admin/server.ts:236-757`)

| Метод | Путь | Назначение |
|---|---|---|
| POST | `/admin/api/payments/initiate` | v1: создание Payment, ссылка Payme/Click |
| GET | `/admin/api/payment-info` | публичная цена PRO |
| POST | `/admin/api/payments/payme` | v1 Payme JSON-RPC вебхук |
| POST | `/admin/api/payments/click/prepare`, `/complete` | v1 Click вебхук |
| POST | `/admin/api/v2/payments/initiate` | v2: Payme/Click/**Alif**, создаёт `Payment(orderCode)` |
| POST | `/admin/api/v2/payments/payme` | v2 Payme вебхук (доп. метод `GetStatement`) |
| POST | `/admin/api/v2/payments/click/prepare`, `/complete` | v2 Click вебхук |
| POST | `/admin/api/v2/payments/alif/webhook` | Alifpay вебхук, HMAC-SHA256 по `rawBody`, заголовок `Signature`, `timingSafeEqual` |
| POST | `/admin/api/v2/vouchers/redeem` | **⚠️ без какой-либо проверки владения** — принимает `{orgId, code}` из тела запроса напрямую; см. GAPS.md |

### Защищённые (`router`, требует `x-admin-password`, `admin/server.ts:792-1607`)

Полный список путей (метод, назначение), без повторения деталей реализации:

`GET /dashboard`, `GET /admin/analytics/funnel`, `GET/POST /users`,
`POST /users/:orgId/upgrade`, `POST /users/:orgId/reset-ai`,
`PATCH /users/:orgId/plan`, `PATCH /users/:orgId/api-key`,
`GET /ai-usage`, `GET /payments`, `GET/POST /vouchers`,
`DELETE /users/:userId`, `DELETE /organizations/:orgId`,
`PATCH /users/:userId/credentials`, `GET/POST /payment-settings` —
всё это v1/legacy-таблицы (`prisma`, схема `contador`).

`GET /v2/dashboard`, `GET/POST /v2/users`, `DELETE /v2/users/:userId`,
`PATCH /v2/users/:userId/credentials`, `PATCH /v2/users/:userId`,
`PATCH /v2/orgs/:orgId/plan`, `PATCH /v2/orgs/:orgId/api-key`,
`POST /v2/orgs/:orgId/upgrade`, `DELETE /v2/orgs/:orgId` (ручной
FK-безопасный каскад), `GET /v2/stats`, `GET /v2/orgs` (поиск/фильтр/пагинация),
`GET /v2/orgs/:orgId/detail`, `PATCH /v2/orgs/:orgId/settings`,
`DELETE /v2/orgs/:orgId/members/:userId`, `DELETE /v2/bank-accounts/:accountId`,
`GET /v2/subscriptions/expiring`, `GET /v2/orgs/:orgId/rules`,
`DELETE /v2/rules/:ruleId`, `GET /v2/audit-logs`,
`PATCH /v2/periods/:periodId/status`, `POST /v2/orgs`,
`POST /v2/orgs/:orgId/members`, `PATCH /v2/orgs/:orgId/members/:userId/role`,
`POST/PATCH /v2/bank-accounts`/`:accountId`, `POST /v2/orgs/:orgId/periods`,
`DELETE /v2/periods/:periodId`, `GET /v2/orgs/:orgId/counterparties`,
`GET /v2/payments`, `GET/POST /v2/vouchers`, `GET /v2/vouchers/export`
(экспорт `.txt`), `POST /v2/vouchers/:code/apply` — всё это `contador_v2`
через `prismaV2`.

Статика: `app.use("/admin", express.static("public"))` **и**
`app.use("/", express.static("public"))` — панель доступна и с префиксом, и
без; `public/` содержит единственный файл `index.html` (2391 строка,
Tailwind CDN + `unpkg.com`, без сборки).

## Тестовые скрипты вебхуков (корень репозитория)

`test-payme.sh`, `test-payments.sh` — shell-скрипты с `curl`-запросами к
локальному (`localhost:3031`) или продовому (`https://contador.uz/admin/api/...`)
эндпоинту; содержат захардкоженные значения, похожие на реальные ключи Payme/
Click прямо в тексте скрипта, версионируемого в git. Сами значения не
читались и не копировались сюда — см. [GAPS.md](GAPS.md) для оценки риска.
