# Views — Contador

Новый файл, создан 2026-09-08 (`TASK-0002`). В проекте **нет EJS/шаблонов** —
v2 использует Next.js App Router (React 19, `"use client"`-компоненты,
Tailwind CSS 4), admin — единственный статический HTML-файл без шаблонизатора.
Раздел «Views» из типового технического задания адаптирован под этот стек:
вместо «переменные шаблона» — карта «страница → какие `/v2/api/...`-запросы
она делает», построенная построчным `grep -R "fetch("` по всем 63
`.tsx`-файлам `v2/src/app/` и `v2/src/components/` (не построчным чтением
каждого файла целиком — см. методологическую оговорку в GAPS.md).

## Структура страниц v2

Каждый маршрут `app/<route>/` обычно состоит из `page.tsx` (тонкая
серверная обёртка, часто ~10 строк, просто рендерит клиентский компонент) и
`<Route>Client.tsx` (`"use client"`, вся логика и `fetch()`-запросы).
Исключение — `documents/new/page.tsx`, где вся логика внутри самого
`page.tsx`.

## Публичные страницы (доступны без сессии — см. `proxy.ts` `PUBLIC_PATHS`)

| Страница | Компонент | Форма → API |
|---|---|---|
| `/login` | `login/page.tsx` (107 строк, прочитан целиком) | `<form>` email+password → `POST /v2/api/auth/login`; при успехе `router.push("/dashboard")` |
| `/register` | `register/page.tsx` | → `POST /v2/api/auth/register` |
| `/forgot-password` | `forgot-password/page.tsx` | → `POST /v2/api/auth/forgot-password` |
| `/reset-password` | `reset-password/page.tsx` | `GET .../reset-password?token=` (валидация при загрузке) → `POST .../reset-password` (сохранение) |

## Приватные страницы (за `proxy.ts`, требуют cookie `v2_session`)

| Страница | Клиентский компонент | Запросы к API |
|---|---|---|
| `/dashboard` | `DashboardClient.tsx` (373 строки) | `GET /v2/api/dashboard`*, `/v2/api/bank-accounts`, `/v2/api/settings/charter-capital` |
| `/transactions` | `TransactionsClient.tsx` | `bank-accounts`, `periods`, `document-types`, `transactions`, `transactions/[id]/category`, `/skip`, `by-counterparty`, `classification/run-ai`, `/status/[periodId]`, `clarification/answer`, `posting/void`, `rules/[id]` (отмена авто-правила) |
| `/documents` | `DocumentsClient.tsx` (286 строк) | `document-types`, `periods`, `documents`, `posting/void` |
| `/documents/new` | `page.tsx` | `document-types`, `periods`, `POST documents` |
| `/accounts` (банковские счета) | `AccountsClient.tsx` | `bank-accounts`, `bank-accounts/[id]/stats`, `DELETE bank-accounts/[id]` |
| `/balance` | `BalanceClient.tsx` | `reports/balance?to=` |
| `/pnl` | `PnLClient.tsx` | `pnl?from&to` |
| `/cashflow` | `CashFlowClient.tsx` | `bank-accounts`, `cashflow?...` |
| `/open-positions` | `OpenPositionsClient.tsx` | `open-items?...`, `open-items/[id]/close`, `/reopen`, `periods` |
| `/tax-dashboard` | `TaxDashboardClient.tsx` | `reports/profit-tax?year&quarter`, `reports/profit-tax/line-details?...` |
| `/profile` | `ProfileClient.tsx` | `payments/history`, `/initiate`, `/subscription`, `/voucher` |
| `/reports/balance`, `/journal`, `/osv`, `/account-card`, `/account-analysis`, `/subconto` | `reports/layout.tsx` + `<Раздел>Client.tsx` | по одному `GET /v2/api/reports/<раздел>?...` на страницу; `journal` дополнительно вызывает `posting/void` |
| `/settings` (главная) | `settings/page.tsx` | `settings/charter-capital`, `settings/org` |
| `/settings/accounts` | `settings/accounts/page.tsx` | `accounts?search=`, `accounts/[id]/visibility`, `document-types` |
| `/settings/members` | `settings/members/page.client.tsx` | `org/members`, `/invite`, `DELETE org/members/[userId]` |
| `/settings/open-items-deadlines` | `page.tsx` | `settings/open-item-deadlines` |
| `/settings/opening-balance` | `page.tsx` | `accounts`, `settings/opening-balance` |
| `/settings/profit-tax` | `page.tsx` | `settings/costing-method`, `settings/costing-method?year=`, `settings/org` |
| `/settings/rules` | `page.tsx` | `document-types`, `rules`, `rules/[id]/apply`, `/reorder`, `DELETE rules/[id]`, `PUT rules/[id]` |
| `/settings/tax-calendar` | `page.tsx` | `settings/tax-deadlines`, `DELETE ...?id=` |
| `/closing` (мастер закрытия месяца, 7 шагов) | `page.tsx`, `ClosingWizard.tsx`, `ClarificationQueue.tsx`, `steps/Step1Import.tsx`…`Step7Summary.tsx` | `dashboard`, `periods`, `periods/[id]/reopen`, DELETE `periods/[id]`, `closing/[periodId]/state`, `/accruals`, `/pending-invoices`, `/step/[n]/complete`, `closing/year-end`, `/year-end/status`, `import/bank`, `/rollback`, `document-types`, `transactions`, `transactions/[id]/category`, `cbu-rate`, `classification/ai-reconcile`, `import/soliq`, `closing/[periodId]/finalize` |

`*` На `/dashboard` собственно данные для KPI/чек-листа приходят через
`GET /v2/api/dashboard`, вызываемый из `dashboard/page.tsx` (не из
`DashboardClient.tsx` напрямую — серверный компонент передаёт данные как
props; подтверждено отсутствием `fetch("dashboard"` внутри самого
`DashboardClient.tsx` при наличии вызова из родителя).

## Layout-компоненты

- `app/layout.tsx` — корневой layout (18 строк).
- `components/Layout/ClientLayout.tsx` — обёртка с `Sidebar`.
- `components/Layout/Sidebar.tsx` (340 строк) — основная навигация; не делает
  собственных `fetch()`-запросов (структурная навигация).
- `components/Layout/SettingsSidebar.tsx` — навигация внутри `/settings/*`.
- `app/reports/layout.tsx`, `app/settings/layout.tsx` — под-layout'ы разделов.
- `components/reports/ReportFilters.tsx`, `ReportsTabNav.tsx` — общие для
  всех страниц `/reports/*`.
- `components/SearchableSelect.tsx`, `components/CharterCapitalModal.tsx` —
  переиспользуемые виджеты (модалка уставного капитала используется минимум
  на `/settings` и `/dashboard`, судя по общему компоненту).

## Admin-панель (`admin/public/index.html`)

Единственный HTML-файл, 2391 строка. Загружает Tailwind через
`cdn.tailwindcss.com` и библиотеку с `unpkg.com` (соответствует CSP в
`admin/server.ts:12-23`: `scriptSrc` явно разрешает оба домена). Не имеет
отдельного build-шага, source и «скомпилированный» файл — один и тот же
артефакт в `admin/public/`. Каждый запрос панели идёт на
`/admin/api/...` с заголовком `x-admin-password` (значение вводится и
хранится на стороне клиента — конкретный механизм хранения пароля в браузере
не изучался построчно по всем 2391 строкам файла, см. GAPS.md за
оговоркой о глубине проверки этого файла).
