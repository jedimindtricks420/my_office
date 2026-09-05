# Overview — 1PDF

Снимок на 2026-09-05, источник — `/home/admin1/1pdf/README.md`, код,
конфигурация и `docker ps`/`git log` на хосте.

## Домен

Приватная (не публичный SaaS-шаблон) production web-платформа для
браузерного редактирования PDF-документов, развёрнутая на `1pdf.uz`. Не
предназначена для распространения как открытый продукт.

## Происхождение

Проект — форк открытого проекта **freePDF** (Chinmay Shringi, MIT-лицензия,
сохранена в `LICENSE.freePDF`; список адаптированных зависимостей — в
`THIRD_PARTY_NOTICES.md`). Каталог `editor/` — адаптированный клиентский
редактор freePDF; `admin/` (Express + Prisma) — собственный бэкенд 1PDF
(аккаунты, подписки, платежи, версии документов), не часть исходного
freePDF.

Git-история (`git log`, ветка `main`, remote
`github.com/jedimindtricks420/1pdf_v1`) показывает явный переход платформы:
коммит `d3d8efd feat: migrate to stable freePDF editor platform` — переход с
более раннего варианта редактора (интеграция с ONLYOFFICE Document Server) на
текущий редактор на базе freePDF/React/PDF.js/Konva. Подробности и
подтверждённые следы старой интеграции — [ARCHITECTURE.md](ARCHITECTURE.md) и
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

## Функциональность (подтверждено кодом)

- **Регистрация/вход**: email+пароль, подтверждение по одноразовому коду
  (OTP) через email (`nodemailer`, SMTP `de.ahost.cloud`), JWT в HTTP-only
  cookie (`1pdf_token`, 30 дней).
- **PDF-редактор** (`editor/`): React 19 + Vite + PDF.js + pdf-lib + Konva +
  Zustand, работает в браузере, локальное автосохранение черновиков
  (IndexedDB, см. `editor/src/lib/drafts.ts`), лимиты 50 МиБ / 500 страниц
  (`admin/routes/editorApi.ts`).
- **Версионирование документов**: неизменяемые версии PDF на пользователя
  (модели `Document`/`DocumentVersion`), конфликт при устаревшей базовой
  версии (`409 VERSION_CONFLICT`).
- **Подписки**: план `FREE`/`PRO`, фиксированная цена годовой подписки
  199 000 сум (константа в `admin/server.ts`, `POST
  /api/client/payments/initiate`).
- **Ваучеры**: генерация партиями с настраиваемой серией и сроком действия,
  массовое удаление отключено намеренно (`DELETE /api/vouchers` → 405).
- **Три платёжных провайдера**: Payme (JSON-RPC вебхук), Click (prepare/
  complete callback, MD5-подпись), Alif/Alifpay (HMAC-SHA256 подпись
  вебхука, инвойсы через `api(-dev).alifpay.uz`). Подробнее —
  [API.md](API.md).
- **Мультиязычный UI**: RU/UZ/EN, хранится на пользователе
  (`User.interface_language`) и в редакторе (i18next, `editor/src/i18n/`).
- **CMS лендинга**: `LandingConfig`/`LandingSection` по языку, редактируется
  через админ-панель.
- **Админ-панель** (`admin/public/admin.html` + `/api/*`): управление
  пользователями, подписками, платежами, ваучерами, платёжными настройками
  (роли `ADMIN`/`SUPERADMIN`).
- **Автоочистка**: почасовой cron (`node-cron`) удаляет файлы в
  `temp_uploads/`, не тронутые больше 24 часов (только legacy-путь загрузки
  через `multer`/`UPLOADS_DIR`, не затрагивает `Document`/`DocumentVersion`
  файлы в `temp_uploads/documents/`).

## Роли

Из кода (`admin/server.ts`, enum `AdminRole` в `prisma/schema.prisma`):
`SUPERADMIN`, `ADMIN` (административная сторона) и обычные пользователи
портала (`User`, не имеют enum-роли — статус определяется наличием
`Subscription.plan`).
