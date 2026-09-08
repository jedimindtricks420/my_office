# Views — 1PDF

Новый файл, создан 2026-09-08 (`TASK-0002`). Проект не использует EJS —
два разных механизма представления: 3 vanilla HTML/JS-файла без сборки
(`admin/public/`) и React SPA (`editor/`, Vite). Карта API-вызовов
построена через `grep -oE "['\"\`]/api/[a-zA-Z0-9/_:-]*"` по HTML-файлам и
`grep -rn "fetch("` по `editor/src` — не построчным чтением каждого файла
(методологическая оговорка, см. GAPS.md).

## `admin/public/index.html` (340 строк) — лендинг

Публичная страница, обслуживается по `GET /`. Загружает контент через
`GET /api/landing/content?lang=` (единственный обнаруженный вызов API —
остальное статичный HTML/CSS). Ссылки на `/portal`, `/admin`, `/editor`.

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
