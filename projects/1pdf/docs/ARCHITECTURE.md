# Architecture — 1PDF

Полностью переписано 2026-09-08 (`TASK-0002`, независимый аудит по коду),
дополнено 2026-09-08 (`TASK-0003` — SEO landing pages, раздел «Поток
запроса» и структура каталогов ниже).

## Топология контейнеров (`docker-compose.yml`, полностью прочитан)

```
Host nginx (Certbot, /etc/nginx/sites-available/1pdf.uz, вне репозитория)
  TLS-терминация → proxy_pass http://127.0.0.1:8003
        │
        ▼
1pdf-nginx (nginx:alpine, контейнер, слушает 0.0.0.0:8003→80)
  конфиг: nginx.freepdf.conf (volume-mount, read-only)
        │
        ├─ auth_request /api/auth/verify  (гейт для /editor/*)
        ▼
1pdf-admin (Express, :8031, только 127.0.0.1)
        │  один PrismaClient в server.ts + ОТДЕЛЬНЫЙ PrismaClient в paymeController.ts
        ▼
1pdf-postgres (postgres:16-alpine, только 127.0.0.1:5435, БД "onepdf")
```

`onlyoffice-docs` — контейнер существует на хосте (образ
`onlyoffice/documentserver:latest`), но в статусе `Exited (127)` и не
описан в `docker-compose.yml` — не часть текущей развёрнутой топологии
(подтверждено read-only `docker ps -a`, см. [GAPS.md](GAPS.md)).

## Поток запроса

### Публичная страница / лендинг
```
GET /  →  1pdf-nginx (location = /)  →  proxy_pass admin:8031/index.html
                                          (статика express.static, admin/public/)
```

### Личный кабинет / вход
```
GET /portal → admin:8031/portal.html (vanilla JS, без сборки)
  → fetch('/api/client/...') → admin/server.ts (JWT в cookie 1pdf_token)
```

### Редактор (PRO-only путь)
```
GET /editor/... 
  → nginx: auth_request /api/auth/verify
       │  проверяет JWT (1pdf_token ИЛИ 1pdf_admin_token), email verified,
       │  активная PRO-подписка именно для /editor* (admin/server.ts:353-392)
       ├─ 401 → редирект /portal (не залогинен)
       ├─ 403 → редирект /portal?msg=pro_required (нет PRO)
       └─ 200 → proxy_pass admin:8031 → express.static(editor/dist) (собранный Vite-бандл)
  → React SPA (editor/src/app/App.tsx): status==='ready' ? <Editor/> : <Home/>
  → useDocumentBootstrap(): ?documentId=... → GET /api/v1/documents/:id/editor-session
                              ?fileId=...     → GET /uploads/:fileId (legacy-путь)
  → редактирование полностью в браузере (pdf.js рендер + konva слои + pdf-lib экспорт)
  → SaveButton: POST /api/v1/documents (новый) или /api/v1/documents/:id/versions (новая версия)
```

### Административная панель
```
GET /admin → admin:8031/admin.html
  → fetch('/api/auth/login') → cookie 1pdf_admin_token
  → fetch('/api/users', '/api/payments', '/api/vouchers', '/api/admin/payment-settings', ...)
```

### SEO landing pages (`TASK-0003`, публичный, без PRO-гейта)
```
GET /ru/<slug>/ или /uz/<slug>/
  → nginx: location ~ ^/(ru|uz)/([a-z0-9-]+)/?$   (БЕЗ auth_request — принципиально
        │   отличается от /editor/*, чтобы Google/новый пользователь видели страницу
        │   без входа в приватное приложение, см. GAPS.md/OVERVIEW.md TASK-0003)
        ▼
  proxy_pass admin:8031/seo/$1-$2.html  →  express.static(admin/public) отдаёт
        статический prerendered файл (сгенерирован scripts/generate-seo-pages.mjs)
        │
        └─ CTA-кнопка "Открыть в редакторе" → /editor/  (дальше — обычный PRO-гейт выше)
```
Несуществующий `<slug>` тоже проходит regex (`[a-z0-9-]+` совпадает с чем
угодно), но файла на диске нет → `express.static` пропускает запрос дальше
→ `authMiddleware` (нет в `publicPaths`) → **401**, а не 404 — тот же
паттерн, что и для любого другого отсутствующего статического файла на
сайте (см. [GAPS.md](GAPS.md)).

## Структура каталогов и назначение

```
/home/admin1/1pdf/
├── admin/
│   ├── server.ts               # 1435 строк: auth, платежи (Payme/Click/Alif),
│   │                            #   пользователи, ваучеры, CMS, legacy ONLYOFFICE-эндпоинты
│   ├── controllers/paymeController.ts   # JSON-RPC методы Payme (отдельный PrismaClient!)
│   ├── routes/editorApi.ts     # /api/v1/* — документы, версии, языковые настройки (текущий бэкенд редактора)
│   ├── utils/paymentUtils.ts   # Click/Payme/Alif подписи и URL-генерация
│   ├── utils/paymeErrors.ts    # коды ошибок Payme JSON-RPC
│   ├── seed.ts                 # сидинг SUPERADMIN + лендинг CMS (RU/UZ)
│   └── public/                 # index.html (лендинг), portal.html (кабинет), admin.html (админка) — vanilla JS, без сборки
│       ├── robots.txt          # TASK-0003: исключает /portal /admin /editor/ /api/ /uploads/
│       ├── sitemap.xml         # TASK-0003: сгенерирован, 23 URL (сгенерирован generate-sitemap.mjs)
│       └── seo/                # TASK-0003: 22 статических RU/UZ SEO landing pages (см. VIEWS.md)
├── editor/                     # независимый Vite/React проект
│   ├── src/app/                # App.tsx (простой стейт-свитч Home/Editor, без роутера), routes/
│   ├── src/components/editor/  # Toolbar, PdfViewer, PropertiesPanel, SaveButton, ExportButton, ...
│   ├── src/lib/pdf/            # loadPdf, renderPdf, exportPdf, pageOperations, textRuns, coordinateTransform
│   ├── src/store/              # zustand: documentStore, editorStore, cloudStore, fontStore, themeStore, toolStore
│   ├── src/lib/drafts.ts       # автосохранение правок в IndexedDB (браузер, не сервер)
│   ├── src/i18n/               # ru/uz/en локали
│   └── tests/, *.test.ts(x)    # vitest, colocated + tests/fixtures/*.pdf
├── prisma/
│   ├── schema.prisma           # 15 моделей, 4 enum'а
│   └── migrations/             # РЕАЛЬНЫЙ формат Prisma (migration_lock.toml + 5 папок) —
│                                #   в отличие от других проектов my_office (contador — сырой SQL)
├── scripts/
│   ├── assert-migrations-additive.sh   # проверяет ОДНУ конкретную (захардкоженную) миграцию на деструктивность
│   ├── generate-seo-pages.mjs   # TASK-0003: source of truth для контента 22 SEO-страниц → admin/public/seo/
│   └── generate-sitemap.mjs     # TASK-0003: собирает admin/public/sitemap.xml из admin/public/seo/*.html
├── nginx.freepdf.conf           # конфиг контейнера 1pdf-nginx; bind-mount ОДНОГО файла (:ro) —
│                                #   правка на хосте требует docker restart 1pdf-nginx, reload недостаточен
│                                #   при замене inode редактором (см. GAPS.md, TASK-0003)
├── docker-compose.yml           # 3 сервиса: nginx-proxy, postgres, admin
├── Dockerfile.admin.v2          # 2-стадийная сборка: editor (Vite build) → admin (tsx runtime)
├── data_oo/, db_oo/, lib_oo/, logs_oo/   # остатки самостоятельного ONLYOFFICE Document Server —
│                                #   исключены из Docker-контекста (.dockerignore), не используются рантаймом
├── test-payments.sh             # ручной shell-скрипт проверки вебхуков (см. GAPS.md — риск секретов)
└── backups/                     # один дамп БД (pre_freepdf_20260801)
```

## Аутентификация — архитектурная точка зрения

Два независимых JWT-контура на **одном** секрете `JWT_SECRET`
(`ACTUAL_JWT_SECRET`, `admin/server.ts:40-44`), различаются по полю payload:
`{ adminId, login, role }` для админов (cookie `1pdf_admin_token`, TTL 7 дней)
и `{ userId, email }` для пользователей (cookie `1pdf_token`, TTL 30 дней).
Единый `authMiddleware` (`admin/server.ts:216-263`) декодирует оба вида
cookie плюс заголовок `Authorization: Bearer` (по наличию `adminId` в
payload определяет, куда класть токен), кладёт результат в `req.admin`/
`req.user` и **блокирует запрос (401), если не распознан ни один** — за
вычетом явного `publicPaths`-списка (14 путей). Отдельный контур
`JWT_SECRET_OO` подписывает конфиги/токены для ONLYOFFICE (legacy-путь,
см. OVERVIEW.md).

`/api/v1/*` (`editorApi.ts`) — собственный `requireUser`/`requirePro`
внутри роутера, **дублирующий** (не переиспользующий) одноимённую логику
`admin/server.ts:277-288` — два места с одинаковым правилом «активная PRO =
`plan === 'PRO' && (!valid_until || valid_until > now)»`, независимо
поддерживаемые.

## Известные архитектурные особенности (заявлены в коде, не результат
## стороннего анализа)

- `App.tsx` не использует библиотеку роутинга — переключение Home/Editor
  через один флаг `documentStore.status` (комментарий:
  `editor/src/app/App.tsx:9-10`, «Routing stays this simple until the
  post-launch SEO tool pages land»).
- Реальные производственные Prisma-миграции (в отличие от других проектов
  my_office) — `npm run migrate` = `prisma migrate deploy`, вызывается
  автоматически при каждом старте контейнера (`Dockerfile.admin.v2:22`).
- Несмотря на «настоящие» миграции, часть текущей схемы БД в них не
  отражена (см. [DATABASE.md](DATABASE.md) — расхождение подтверждено
  read-only SQL-инспекцией, не только сравнением файлов).
