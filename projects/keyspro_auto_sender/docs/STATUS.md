# STATUS — keyspro_auto_sender

Последнее обновление: **2026-09-05** (первичная регистрация проекта в
`my_office`, задача TASK-0001).

## Итог первичного исследования

Проект `/home/admin1/keyspro_auto_sender/` (FastAPI-микросервис на Python
3.11, единственный git-репозиторий, remote `origin` →
`github.com/jedimindtricks420/keyspro_auto_sender`, ветка `master`)
зарегистрирован в `my_office`. Документация (`docs/OVERVIEW.md`,
`ARCHITECTURE.md`, `STACK.md`, `DATABASE.md`, `API.md`, `DEPLOYMENT.md`)
создана на основании прямого чтения всего кода приложения (`app/`),
конфигурации (`docker-compose.yml`, `Dockerfile`, `product_config.json`,
имена переменных `.env` без чтения значений) и текущего состояния хоста
(`docker ps`/`docker inspect`/`docker exec`, конфиг Nginx). Код проекта не
изменялся.

## Что подтверждено и работает

- Контейнер `keyspro_auto_sender-key-distributor-1` (Docker Compose)
  запущен и здоров (`healthy`), поднят 2026-08-20, порт 8000 → Nginx
  (`webhook.keyspro.uz`, TLS через Certbot).
- Поток: WooCommerce webhook → FastAPI (`POST /api/v1/process-order`) →
  фоновая обработка по "рецептам" из `product_config.json` (7 SKU) →
  получение ключа (Telegram/Pyrogram-бот `@MicrosoftLeo_Bot` ИЛИ внешний
  Dr.Web HTTP API, в зависимости от поля `source`) → запись в SQLite
  (`Order`, `IssuedKey`) → email клиенту (Jinja2-шаблон) → уведомление
  админа в Telegram. Подробности — [ARCHITECTURE.md](ARCHITECTURE.md).
- Веб-дашборд администратора (`/dashboard`, HTTP Basic Auth): статус
  системы, список/детали заказов, CRUD товаров (через
  `product_config.json`), просмотр/предпросмотр email-шаблонов.
  Подробности — [API.md](API.md).
- Стек: FastAPI + SQLAlchemy(async)/SQLite + Pyrogram +
  python-telegram-bot + aiosmtplib + Jinja2, без тестов и CI. Подробности
  — [STACK.md](STACK.md).

## Известные проблемы / расхождения (подтверждены напрямую)

1. **`WEBHOOK_SECRET` объявлен в конфигурации, но нигде не проверяется в
   коде** — webhook-эндпоинт принимает любой валидный по схеме JSON без
   аутентификации источника.
2. **Живой GitHub PAT в открытом виде** в `origin` remote URL
   (`.git/config` на хосте, не в файлах репозитория) — требует ротации.
   Токен не скопирован в эту документацию.
3. **SQLite-база (`keyspro.db`) не персистентна**: не примонтирована как
   volume в `docker-compose.yml`, живёт только в writable-слое
   контейнера — пересоздание контейнера потеряет историю заказов и
   выданных ключей.
4. **ORM-модель `Product` определена, но не используется** — весь
   каталог товаров фактически хранится в `product_config.json`, а не в
   БД, вопреки комментарию в коде "replaces product_config.json".
5. **Пустые дублирующие файлы шаблонов** в корневом `templates/` (0
   байт) — реально используются одноимённые файлы в `app/templates/`.
6. **README проекта частично устарел**: помечен датой 2026-01-23, но в
   истории git есть более поздние изменения (до 2026-05-26).
7. **Нет автоматических тестов и CI** — только ручная эксплуатация.

Полный список с деталями — [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

## Что не проверялось

- Значения секретов/`.env` (только имена переменных).
- Содержимое `keyspro.db` (может содержать email клиентов и лицензионные
  ключи).
- Внешний сервис `drweb-key-service`, к API которого обращается
  `DrWebProvider` (есть предсуществующая пустая заготовка
  `my_office/projects/drweb-key-service/`, не заполнялась в этой задаче).
- Приложение `partner_bot`, делящее домен `webhook.keyspro.uz` через
  Nginx, но не входящее в этот репозиторий.
- Функциональное поведение системы через реальный запуск сценария заказа
  — только статический анализ кода и наблюдение уже работающего состояния
  хоста.
- Ротация/устранение находок безопасности (пп. 1–2 выше) — задача была
  ограничена документированием, не исправлением.
