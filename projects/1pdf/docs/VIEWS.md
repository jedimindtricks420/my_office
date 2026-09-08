# Views — 1PDF

Создан 2026-09-08 (`TASK-0002`), дополнен 2026-09-08 (`TASK-0003` — раздел
«SEO landing pages» ниже). Проект не использует EJS — три механизма
представления: 3 vanilla HTML/JS-файла без сборки (`admin/public/`),
22 статических prerendered SEO-страницы без фреймворка (`admin/public/seo/`)
и React SPA (`editor/`, Vite). Карта API-вызовов построена через
`grep -oE "['\"\`]/api/[a-zA-Z0-9/_:-]*"` по HTML-файлам и
`grep -rn "fetch("` по `editor/src` — не построчным чтением каждого файла
(методологическая оговорка, см. GAPS.md).

## `admin/public/seo/*.html` (22 файла) — публичные SEO landing pages (TASK-0003)

Статические HTML-файлы, сгенерированные `scripts/generate-seo-pages.mjs`
(source of truth для контента — редактировать через генератор, не файлы
напрямую) и `scripts/generate-sitemap.mjs`. Не за `authMiddleware` в
обычном смысле — доступны напрямую по URL `/ru/<slug>/` и `/uz/<slug>/`
через отдельный `location` в `nginx.freepdf.conf` (без `auth_request`, в
отличие от `/editor/*`); файлы физически лежат под `express.static(admin/public)`
и были бы доступны и по прямому пути `/seo/<lang>-<slug>.html`.

10 пар (RU/UZ) + хаб `pdf-tools`/`pdf-vositalari`:

| Ключ | RU URL | UZ URL |
|---|---|---|
| pdf-editor | `/ru/pdf-editor/` | `/uz/pdf-muharriri/` |
| edit-pdf | `/ru/edit-pdf/` | `/uz/pdf-tahrirlash/` |
| add-text | `/ru/add-text-to-pdf/` | `/uz/pdfga-matn-qoshish/` |
| add-image | `/ru/add-image-to-pdf/` | `/uz/pdfga-rasm-qoshish/` |
| draw | `/ru/draw-on-pdf/` | `/uz/pdfga-chizish/` |
| highlight | `/ru/highlight-pdf/` | `/uz/pdf-matnini-belgilash/` |
| annotate | `/ru/annotate-pdf/` | `/uz/pdfga-izoh-qoshish/` |
| sign | `/ru/sign-pdf/` | `/uz/pdfga-imzo-qoyish/` |
| fill | `/ru/fill-pdf/` | `/uz/pdf-toldirish/` |
| russian/localization | `/ru/pdf-editor-russian/` | `/uz/pdf-muharriri-ozbek-tilida/` |
| hub | `/ru/pdf-tools/` | `/uz/pdf-vositalari/` |

Каждая страница (кроме хаба): canonical (self), взаимный `hreflang`
ru/uz/x-default, Open Graph + Twitter meta, JSON-LD `BreadcrumbList` +
`FAQPage` (пары `pdf-editor`/`edit-pdf` дополнительно несут
`WebApplication`); единственный CTA — `<a href="/editor/">`, полагается на
существующий gate `auth_request`/`/api/auth/verify` для редиректа на
`/portal` (не авторизован) или `/portal?msg=pro_required` (нет PRO) —
логика авторизации не дублируется на самих SEO-страницах. Контент не
использует слово «бесплатно»/«free» — весь `/editor/*` требует PRO (см.
[GAPS.md](GAPS.md), пункт про `Home.tsx`).

`admin/public/robots.txt` (новый) и `admin/public/sitemap.xml`
(сгенерирован, 23 URL) исключают `/portal`, `/admin`, `/editor/`, `/api/`,
`/uploads/`.

## `admin/public/index.html` (лендинг)

Публичная страница, обслуживается по `GET /`. Загружает контент через
`GET /api/landing/content?lang=` (единственный обнаруженный вызов API —
остальное статичный HTML/CSS). Ссылки на `/portal`, `/admin`, `/editor`.
С `TASK-0003` (2026-09-08) навигация и футер дополнены ссылками на
PDF tools hub и 5 SEO tool pages (см. раздел ниже), переключаемыми по
языку (`#nav-tools`, `#footer-tools`, объект `TOOL_LINKS` в инлайн-скрипте).

## `admin/public/portal.html` (1148 строк) — личный кабинет пользователя

Обслуживается по `GET /portal`. Vanilla JS без фреймворка. Вызываемые
эндпоинты (по строковым литералам в файле):

`/api/client/login`, `/api/client/signup`, `/api/client/verify-signup`,
`/api/client/logout`, `/api/client/me`, `/api/client/verify-email`,
`/api/client/change-password`, `/api/client/history`,
`/api/client/activate-voucher`, `/api/client/payments/initiate`,
`/api/v1/documents`, `/api/v1/documents/blank`.

Функциональность (по составу вызовов): вход/регистрация с OTP, профиль,
смена пароля, история платежей, активация ваучера, инициация оплаты
подписки, список документов и создание нового документа (переход в
`/editor?documentId=...`).

## `admin/public/admin.html` (1255 строк) — панель администратора

Обслуживается по `GET /admin`. Вызываемые эндпоинты:

`/api/auth/login`, `/api/auth/me`, `/api/dashboard`, `/api/users`,
`/api/admins`, `/api/payments`, `/api/vouchers`, `/api/vouchers/export`,
`/api/vouchers/generate`, `/api/admin/landing/content`,
`/api/admin/payment-settings`, а также **`/api/auth/me/login`
(`admin.html:1088`) и `/api/auth/me/password`
(`admin.html:1097`)** — обе последние не соответствуют ни одному
зарегистрированному маршруту в `admin/server.ts` (подтверждено `grep`,
см. [GAPS.md](GAPS.md)) — форма самообслуживания админа (смена
собственного логина/пароля) в интерфейсе присутствует, но не работает.

## `editor/` — React SPA (Vite, обслуживается за `/editor/*`)

Не файлово-роутинговая структура — `src/app/App.tsx` переключает между
`Home`/`HomeV2` (лендинг внутри SPA для незалогиненных сценариев отображения
редактора без документа) и `Editor` (`src/app/routes/Editor.tsx`) по
состоянию `documentStore.status`. Карта запросов к API по компонентам:

| Компонент | Запросы |
|---|---|
| `hooks/useDocumentBootstrap.ts` | `GET /uploads/:fileId` (legacy-путь по query `?fileId=`) **или** `GET /api/v1/documents/:documentId/editor-session` + скачивание по `downloadUrl` (путь по query `?documentId=`) |
| `hooks/useLanguageProfile.ts` | `GET /api/v1/me/preferences` |
| `components/ui/LanguageSwitcher.tsx` | `PATCH /api/v1/me/preferences/language` |
| `components/editor/SaveButton.tsx` | `POST /api/v1/documents` (новый документ) или `POST /api/v1/documents/:id/versions` (новая версия), `baseVersionId` для оптимистичной блокировки |
| `store/fontStore.ts`, `lib/pdf/fontEmbedding.ts` | `GET <notoUrl>` — загрузка шрифта Noto Sans (статический ассет, не API) |

Остальные компоненты (`Toolbar`, `PdfViewer`, `PropertiesPanel`,
`ThumbnailStrip`, `overlayNodes`, `SignatureTool`, `ExportButton`,
`DownloadButton`, `EditorCanvas`, `EditorFooter`) работают целиком на
клиентском состоянии (`zustand`-сторы `documentStore`/`editorStore`/
`toolStore`/`themeStore`), не обращаются к серверу — вся логика правок и
финальный экспорт PDF (`ExportButton`/`DownloadButton` →
`lib/pdf/exportPdf.ts`) выполняется в браузере.

### Черновики (не серверный view, но часть UX сохранения)

`lib/drafts.ts` — IndexedDB (`1pdf-editor` база, store `drafts`), ключ
`${userId}:${documentId}:${versionId}`. Не показано отдельным экраном —
интегрировано в `SaveButton`/загрузку документа (сброс черновика после
успешного облачного сохранения).

## Локализация

`editor/src/i18n/locales/{ru,uz,en}.json` — react-i18next,
`i18next-browser-languagedetector`. Язык интерфейса синхронизируется с
`User.interface_language` через `/api/v1/me/preferences*` (см. таблицу
выше) — единственная связь локализации с сервером, остальной перевод
статичен на клиенте.
