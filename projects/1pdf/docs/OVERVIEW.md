# Overview — 1PDF

Полностью переписано 2026-09-08 в рамках `TASK-0002` (независимый повторный
аудит с нуля, только по коду `/home/admin1/1pdf/`, без чтения `.md`-файлов
самого проекта). Предыдущая версия (`TASK-0001`, 2026-09-05) не использовалась
как источник фактов — только как список для сверки полноты после написания
этого документа. Все факты ниже — «подтверждено кодом», если не указано иное.

## Что это

Приватная (не публичный форк-энд-гоу open source) production-платформа для
браузерного редактирования PDF, развёрнутая на `1pdf.uz`. Вывод о назначении
сделан из совокупности: клиентский PDF-редактор на канвасе (`editor/`,
React+Konva+PDF.js+pdf-lib), серверное версионирование документов (модели
`Document`/`DocumentVersion`), подписки FREE/PRO с оплатой через Payme/Click/
Alifpay, система ваучеров, многоязычный лендинг (RU/UZ) с CMS, админ-панель.

Клиентский редактор — форк открытого проекта **freePDF** (Chinmay Shringi,
MIT) — подтверждено `editor/package.json:4` (`"description": "1PDF browser
PDF editor based on freePDF."`), `editor/src/config.ts:14`
(`REPO_URL = 'https://github.com/ChinmayShringi/freePDF'`) и файлами
`LICENSE.freePDF`, `THIRD_PARTY_NOTICES.md` в корне репозитория (содержимое
последнего не читалось — расширение `.md`).

## Три слоя в одном репозитории

| Слой | Каталог | Стек | Назначение |
|---|---|---|---|
| **admin** (backend) | `admin/` | Express 4, TypeScript (`tsx`, без сборки), Prisma 6 | REST API: auth, платежи, подписки, документы, CMS, админка |
| **editor** (frontend) | `editor/` | React 19, Vite 8, TypeScript, Konva, PDF.js, pdf-lib, Zustand | Браузерный PDF-редактор, PRO-only |
| **admin/public** (статика) | `admin/public/` | Vanilla HTML/JS (без сборки), 3 файла | Лендинг (`index.html`), личный кабинет (`portal.html`), панель администратора (`admin.html`) |

Один `package.json` в корне (`"name": "1pdf-admin"`) управляет backend'ом;
`editor/` — независимый npm-проект (`"name": "1pdf-editor"`), собирается
отдельно (`npm run editor:build` в корневом `package.json` вызывает
`npm --prefix editor run build`) и раздаётся Express-сервером как статика
(`admin/server.ts:88-142`).

## Два поколения редактирования документов (сосуществуют в коде)

1. **Текущее** — `POST/GET /api/v1/documents*`
   (`admin/routes/editorApi.ts`), модели `Document`/`DocumentVersion` с
   версионированием и SHA-256-дедупликацией контента, используется активным
   React-редактором (`editor/src/hooks/useDocumentBootstrap.ts`,
   `editor/src/components/editor/SaveButton.tsx`).
2. **Legacy, подтверждено неактивным** — `/api/documents/config`,
   `/api/documents/upload`, `/api/documents/create-new`,
   `/api/documents/callback` (`admin/server.ts:1197-1395`), рассчитанные на
   интеграцию с **ONLYOFFICE Document Server** (подпись JWT через
   `JWT_SECRET_OO`, внутренний Docker-хост `onlyoffice-docs`). Подтверждено
   независимо тремя способами: (а) ни в `docker-compose.yml`, ни в каком-либо
   другом `.yml`/`.conf` файле репозитория сервис `onlyoffice-docs` не
   определён; (б) ни один файл `editor/src/**` не содержит упоминания
   ONLYOFFICE/DocsAPI; (в) read-only `docker ps -a --filter name=onlyoffice`
   (2026-09-08) показывает контейнер `onlyoffice-docs` в статусе **`Exited
   (127) 2 недели назад`** — то есть он физически существует на хосте, но не
   запущен и не управляется текущим `docker-compose.yml`. Подробности —
   [GAPS.md](GAPS.md).

## Ключевая функциональность (по коду)

- **Клиентское редактирование PDF** — рендер через `pdfjs-dist`, интерактивные
  слои через `react-konva`, финальная сборка через `pdf-lib`
  (`editor/src/lib/pdf/exportPdf.ts`). Обработка полностью в браузере — файл
  не уходит на сервер до нажатия «Сохранить».
- **Черновики в IndexedDB** — `editor/src/lib/drafts.ts`, автономное
  сохранение правок локально до явного сохранения в облако.
- **Версионирование документов** — `POST /api/v1/documents/:id/versions` с
  оптимистичной блокировкой по `baseVersionId` (409 `VERSION_CONFLICT` при
  гонке, `admin/routes/editorApi.ts:283-336`).
- **PRO-подписка обязательна для редактора целиком** — весь путь `/editor/*`
  закрыт на уровне nginx (`auth_request` → `/api/auth/verify`, редирект на
  `/portal?msg=pro_required` при отсутствии активной PRO,
  `nginx.freepdf.conf:50-59,84`).
- **Регистрация с подтверждением по email (OTP)** — код на 6 цифр, хешируется
  и кладётся в JWT `signupToken` (TTL 15 минут) вместо хранения в БД до
  подтверждения (`admin/server.ts:525-604`).
- **Три платёжных провайдера**: Payme, Click, Alifpay — фиксированная цена
  199 000 сум/год (`admin/server.ts:944`, захардкожено, не берётся из
  `PaymentConfig`).
- **Ваучеры** — генерация сериями, экспорт `.txt`, атомарная активация
  (`updateMany` с условием `is_used: false`, `admin/server.ts:656-662`).
- **CMS лендинга** — двуязычный контент (`RU`/`UZ`) через
  `LandingConfig`/`LandingSection`, редактируется из `admin.html`.
- **Многоязычный интерфейс редактора** — RU/UZ/EN
  (`editor/src/i18n/`, `User.interface_language`).

## Роли

Два независимых субъекта аутентификации с разными JWT-payload и cookie:
`Admin` (роли `SUPERADMIN`/`ADMIN`, cookie `1pdf_admin_token`) и `User`
(обычный пользователь, cookie `1pdf_token`) — см. [ARCHITECTURE.md](ARCHITECTURE.md).

## Требования к запуску (выведено из `package.json`/Dockerfile)

- Node.js **22** (`Dockerfile.admin.v2:1,8` — `FROM node:22-alpine`; отличие
  от предыдущего аудита других проектов my_office, использующих Node 20).
- PostgreSQL 16, одна база `onepdf`.
- npm (единственный найденный формат lock-файлов).
- Переменные окружения — [CONFIGURATION.md](CONFIGURATION.md).

Подробный порядок установки — [DEPLOYMENT.md](DEPLOYMENT.md) (RUNBOOK).
