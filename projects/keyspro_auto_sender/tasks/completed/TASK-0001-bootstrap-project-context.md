---
id: TASK-0001
title: Bootstrap project context for keyspro_auto_sender
status: completed
created: 2026-09-05
completed: 2026-09-05
owner: azureuser (agent-assisted)
---

# TASK-0001 — Bootstrap project context

## Цель

Зарегистрировать существующий проект `keyspro_auto_sender`
(источник: `/home/admin1/keyspro_auto_sender/`) в `my_office` и создать
документацию его подтверждённого текущего состояния. Без разработки нового
функционала, без исправления кода, без рефакторинга, без домыслов там, где
факт не подтверждён кодом/конфигурацией.

## Журнал работы

### 2026-09-05

- Прочитан корневой [`/home/admin1/my_office/AGENTS.md`](../../../AGENTS.md).
- Подтверждено (как и в прошлых задачах): `my_office` на диске принадлежит
  `admin1`, работа ведётся через клон `/home/azureuser/work/my_office`,
  публикация — через `git push`.
- Создана структура `projects/keyspro_auto_sender/{docs,specs,tasks}`.
- Обнаружен предсуществующий **пустой** каталог
  `projects/drweb-key-service/{docs,specs,tasks}` (только пустые
  поддиректории, ни одного файла, не отслеживается git — `git status`/`git
  log` по нему пустые). Не тронут этой задачей. Совпадение названия
  неслучайно: код `keyspro_auto_sender` обращается к внешнему API с именем
  `drweb-key-service` (см. `app/services/drweb_provider.py`) — вероятно,
  заготовка под документацию отдельного сервиса, который в этой задаче не
  исследовался (вне области).

### Что было просмотрено

- Верхний уровень `/home/admin1/keyspro_auto_sender/`: `README.md`,
  `.env` (только имена переменных через `grep -oE`, значения не читались),
  `.gitignore`, `.dockerignore`, `Dockerfile`, `docker-compose.yml`,
  `requirements.txt`, `product_config.json` (полностью, 7 SKU).
- `app/main.py` (полностью) — FastAPI приложение, lifespan, единственный
  бизнес-эндпоинт `POST /api/v1/process-order`.
- `app/core/`: `config.py` (Pydantic Settings, все поля), `models.py`
  (Pydantic-схемы `OrderPayload`/`BillingPayload`/`LineItemPayload`/
  `WebhookPayload`), `security.py` (HTTP Basic для дашборда).
- `app/database/`: `database.py` (SQLAlchemy async engine, SQLite),
  `models.py` (ORM-модели `Order`, `IssuedKey`, `Product`), `__init__.py`.
- `app/services/`: `key_provider.py` (Pyrogram-клиент, поиск ключа по
  regex в истории чата бота), `drweb_provider.py` (HTTP-клиент к внешнему
  Dr.Web API), `notifier.py` (email через `aiosmtplib` + Jinja2, Telegram
  через `python-telegram-bot`), `processor.py` (основная бизнес-логика
  обработки заказа).
- `app/dashboard/`: `router.py` (полностью, все маршруты), `services.py`
  (полностью, статусы системы и статистика).
- `app/crud/__init__.py`, `app/schemas/__init__.py`,
  `app/dashboard/__init__.py` — все пустые (заготовки, не используются).
- Списки файлов и размеры: `app/templates/*.html` (email-шаблоны, 7 файлов),
  `app/dashboard/templates/*.html` (UI дашборда, 8 файлов), корневой
  `templates/*.html` (2 файла).
- Содержимое корневых `templates/complex_product_template.html` и
  `templates/simple_product_template.html` — оба **0 байт** (пустые), в
  отличие от одноимённых файлов в `app/templates/` (684–2647 байт,
  реально используются кодом через `Path(__file__).parent.parent /
  'templates'`, т.е. `app/templates/`, не корневой `templates/`).
- `git log --oneline` (полная история, 20+ коммитов), `git status`, `git
  remote -v` — обнаружено, что `origin` содержит GitHub Personal Access
  Token непосредственно в URL (`.git/config`, drwo не в файлах репозитория).
  Токен НЕ скопирован в документацию — только зафиксирован факт его наличия
  как критическая находка безопасности (см. OPEN_QUESTIONS.md).
- Текущее состояние хоста (только чтение): `docker ps -a --filter
  name=keyspro` (контейнер `keyspro_auto_sender-key-distributor-1`,
  здоров, поднят 2026-08-20), `docker inspect` (образ собран 2026-03-26,
  bind-mounts: `.env`, `app/`, `product_config.json` — БД `keyspro.db` НЕ
  примонтирована), `docker exec ... ls -la /app/` (подтверждено:
  `keyspro.db` существует только внутри контейнера, 36 КБ), содержимое
  `/etc/nginx/sites-available/keyspro_auto_sender` (домен
  `webhook.keyspro.uz`, TLS через Certbot, плюс соседний unrelated
  `location /partner_bot/` → отдельный backend на 127.0.0.1:8080, не часть
  этого репозитория).
- `grep -rn WEBHOOK_SECRET app/` — подтверждено: переменная объявлена в
  `Settings`, но нигде не читается/не проверяется в `main.py` или где-либо
  ещё в коде. Заголовок с секретом от WooCommerce не валидируется.
- `grep -rn Product app/` — подтверждено: ORM-модель `Product` определена
  в `database/models.py` и импортируется в `dashboard/router.py`, но нигде
  не используется как запрос к БД (весь дашборд читает/пишет товары через
  `product_config.json`). Комментарий в коде "(replaces
  product_config.json)" не соответствует фактическому поведению.
- Поиск тестов/CI: `find . -iname "*test*"` и `.github/` — ничего не
  найдено (нет автоматических тестов, нет GitHub Actions).
- Содержимое `keyspro.db` **не читалось** (может содержать реальные email
  клиентов и выданные лицензионные ключи — PII/секреты).

### Что установлено (ключевые факты)

- Единственный сервис (FastAPI, порт 8000) в одном Docker-контейнере,
  Nginx-прокси на `webhook.keyspro.uz`.
- Поток: WooCommerce webhook → валидация Pydantic → фоновая задача →
  получение ключа (Pyrogram-бот `@MicrosoftLeo_Bot` ИЛИ внешний Dr.Web API,
  в зависимости от `source` в `product_config.json`) → запись в SQLite →
  email клиенту (Jinja2-шаблон по `template_name`) → уведомление админа в
  Telegram.
- Хранилище конфигурации товаров — файл `product_config.json` (не БД),
  несмотря на наличие ORM-модели `Product`.
- SQLite-база (`keyspro.db`) существует только в файловой системе
  контейнера, не примонтирована как volume — при пересоздании контейнера
  (не при простом restart) история заказов/ключей будет потеряна.
- `WEBHOOK_SECRET` объявлен в конфигурации, но не применяется нигде в коде
  — webhook-эндпоинт принимает любой корректный по схеме JSON без проверки
  подлинности источника.
- `origin` remote в `.git/config` содержит "живой" GitHub PAT в открытом
  виде в URL — не в файлах репозитория, но в локальной git-конфигурации
  хоста; НЕ зафиксирован в документации.
- Нет автотестов, нет CI, нет линтера/типизации (кроме Pydantic-моделей).

### Какие документы созданы

`projects/keyspro_auto_sender/{README.md, AGENTS.md}` и
`projects/keyspro_auto_sender/docs/{OVERVIEW,ARCHITECTURE,STACK,STATUS,
DATABASE,API,DEPLOYMENT,OPEN_QUESTIONS}.md`, плюс `.gitkeep` в
`docs/adr/`, `docs/runbooks/`, `specs/active/`, `specs/archived/`.

### Какие вопросы остались

Полный список — [`docs/OPEN_QUESTIONS.md`](../../docs/OPEN_QUESTIONS.md).
Коротко: судьба GitHub PAT в `origin` remote, отсутствие проверки
`WEBHOOK_SECRET`, неиспользуемая модель `Product`, непримонтированная
SQLite БД, назначение `location /partner_bot/` в Nginx, пустые дублирующие
шаблоны в корневом `templates/`, актуальность README (последнее обновление
указано 2026-01-23, но есть более поздние коммиты вплоть до 2026-05-26).

### Финальная сверка

После написания документации проведена точечная повторная проверка ключевых
утверждений напрямую по коду хоста:

- `grep -n "WEBHOOK_SECRET\|X-Wc-Webhook\|Signature" app/main.py` — пусто,
  подтверждает: секрет вебхука нигде не проверяется.
- Количество переменных `DASHBOARD_USERNAME`/`DASHBOARD_PASSWORD` в `.env`
  — обе присутствуют как заданные ключи (значения не читались).
  `SMTP_HOST`/`SMTP_PORT` — присутствуют как ключи.
- `product_config.json` — подтверждено 7 SKU (совпадает с ранее
  зафиксированным числом в OVERVIEW.md/STATUS.md).

Расхождений между документацией и кодом на момент сверки не найдено.
`docs/STATUS.md` обновлён с итоговым снимком состояния.

Также обнаружено: параллельно (в рамках этой же сессии `my_office`, другим
процессом) каталог `projects/drweb-key-service/`, который на момент начала
этой задачи был пустой заготовкой, оказался заполнен документацией
отдельного реального проекта `/home/admin1/drweb-key-service/` (коммит
`6deab5e`, уже в `origin/main`) — это именно тот внешний сервис, к API
которого обращается `app/services/drweb_provider.py` в
`keyspro_auto_sender`. Ссылки на него в `AGENTS.md`/`OPEN_QUESTIONS.md`
этого проекта обновлены, чтобы не оставлять устаревшее упоминание "пустой
заготовки".

## Статус: завершена

Все пункты чек-листа задачи выполнены: код `/home/admin1/keyspro_auto_sender/`
не изменялся, документация основана только на подтверждённых фактах,
неподтверждённое вынесено в OPEN_QUESTIONS.md, STATUS.md обновлён, находки
безопасности (незащищённый webhook, GitHub PAT в открытом виде)
зафиксированы, но не исправлялись — по прямому указанию задачи.
