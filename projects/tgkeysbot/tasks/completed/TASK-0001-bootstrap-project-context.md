# TASK-0001 — Bootstrap project context

Статус: **завершено**
Начато: 2026-09-05
Завершено: 2026-09-05
Тип: регистрация существующего проекта в `my_office`, документирование
текущего состояния. Без изменений кода, без разработки нового функционала.

## Цель

Зарегистрировать существующий проект `/home/admin1/tgkeysbot/` в `my_office`
и создать достоверную документацию его текущего состояния на основании
прямого чтения кода — без изменения самого проекта.

## Область (scope)

- Источник: `/home/admin1/tgkeysbot/` (только чтение).
- Результат: `my_office/projects/tgkeysbot/` (AGENTS.md, README.md, docs/*,
  specs/*, tasks/*).
- Запрещено: разработка функционала, исправление кода, рефакторинг,
  изменение `/home/admin1/tgkeysbot/`, выдумывание отсутствующей информации.

`/home/admin1/tgkeysbot/` в рамках задачи не изменялся — подтверждено:
изменялись только файлы под `/home/azureuser/work/my_office/projects/tgkeysbot/`.

## Журнал исследования

### Просмотрено

- Структура каталогов (`find`), `.dockerignore`, `.env`/`.env.example`
  (только имена переменных, без значений), `Dockerfile`,
  `docker-compose.yml`, `requirements.txt`, `manage.py`.
- `config/settings.py`, `config/urls.py`, `config/asgi.py`, `config/wsgi.py`.
- `core/models.py`, `core/admin.py`, `core/views.py`, `core/tests.py`,
  `core/apps.py`, все 3 файла миграций (`core/migrations/000{1,2,3}_*.py`),
  Django-шаблоны админки (`core/templates/admin/core/partner/*.html`).
- `bot/main.py`, `bot/drweb_client.py`, `bot/generate_session.py`,
  `generate_session.py` (в корне, сравнён diff'ом с версией в `bot/`).
- `transport/tasks.py`, `transport/test_pyrogram.py`.
- Схемы (только имена таблиц, без строк) `db.sqlite3` и `keyspro.db` —
  через `python3 -c "sqlite3..."` (CLI `sqlite3` на хосте отсутствует).
- Состояние хоста: `git status` (не git-репозиторий), `docker ps -a`,
  `docker images`, `docker inspect` (образы/контейнеры, порты, `Created`/
  `StartedAt`), `docker exec ... env` (только имена переменных
  `DEBUG`/`ALLOWED_HOSTS`/`FORCE_SCRIPT_NAME`/`DATABASE_URL` — значение
  пароля в `DATABASE_URL` замаскировано при просмотре), `docker exec ...
  whoami`, `docker exec ... ls /app/*.session /app/*.db /app/*.tar.gz`,
  `docker exec ... pip show <пакеты>` для точных версий, `ss -ltnp` (кто
  слушает 8000/8080), `/etc/nginx/sites-available/*` и `sites-enabled/`,
  `ufw status`.
- Повторная сверка ключевых утверждений документации с кодом перед
  завершением задачи (`grep` по `environ`, `Celery(`, `activation_price`,
  `negative_limit`, статусам `OrderStatus`, `command_pattern`,
  `keyspro.db`).

### Установлено

- Проект — Django 6.0.3 back-office (`core` app) + отдельный
  aiogram/Pyrogram-бот (`bot/main.py`), общая PostgreSQL 16 через Django
  ORM. Три сервиса `docker-compose.yml`: `db`/`web`/`bot`, все три подняты
  и работают на хосте.
- Нет git-репозитория, нет README, нет автотестов и CI.
- Два независимых пути выдачи ключа (`Product.delivery_source`):
  Pyrogram-эмуляция пользователя против стороннего бота
  `@MicrosoftLeo_Bot`, либо HTTP к внешнему `drweb-key-service`.
- `transport/tasks.py` (Celery + Redis-lock) — нерабочий, неподключённый
  дублирующий код (нет зависимостей `celery`/`redis` в
  `requirements.txt`, нет `Celery()`-приложения, нет сервисов
  worker/Redis в compose).
- Найдена и подтверждена находка безопасности: `session_generator
  .session` (живая Pyrogram MTProto-сессия), `keyspro.db`, `media.tar.gz`
  не исключены `.dockerignore` и физически присутствуют внутри работающих
  контейнеров (`docker exec ... ls /app/`).
- `.env.example` содержит значения, похожие на реальные (не плейсхолдеры)
  для `CLIENT_BOT_TOKEN`/`ADMIN_TELEGRAM_ID`/`API_ID`/`API_HASH` — не
  скопированы никуда, зафиксировано как открытый вопрос/риск.
- Django-админка публично доступна на `https://webhook.keyspro.uz/partner_bot/`
  через Nginx-конфиг, физически принадлежащий другому проекту
  (`keyspro_auto_sender`) — это подтверждённый ответ на открытый вопрос
  №8/№15 из `my_office/projects/keyspro_auto_sender/docs/OPEN_QUESTIONS.md`
  (сам этот файл в рамках данной задачи не редактировался — вне
  согласованной области).
- `FORCE_SCRIPT_NAME`, ожидаемый для этого деплоя, не задан в фактическом
  окружении контейнера `web`.
- Контейнеры работают от `root`; порт 8080 (Django-админка) опубликован
  на `0.0.0.0` хоста; `ufw` на хосте неактивен.
- `keyspro.db` — файл с иной (не-Django) схемой, не используется кодом;
  `db.sqlite3` — Django-схема, но не прод-БД (прод использует Postgres).
- Отсутствие проверки `negative_limit` перед покупкой; баланс списывается
  после выдачи ключа; активация в текущем коде бесплатна (цена
  захардкожена как 0); модели `SourceAccount`/`Settings` определены в БД,
  но не используются актуальным кодом `bot/main.py`.
- Образ Docker собран 2026-03-25 и соответствует текущему коду на диске
  по датам изменения файлов — расхождений не обнаружено.

### Созданы документы

- `my_office/projects/tgkeysbot/AGENTS.md`
- `my_office/projects/tgkeysbot/README.md`
- `my_office/projects/tgkeysbot/docs/OVERVIEW.md`
- `my_office/projects/tgkeysbot/docs/ARCHITECTURE.md`
- `my_office/projects/tgkeysbot/docs/STACK.md`
- `my_office/projects/tgkeysbot/docs/DATABASE.md`
- `my_office/projects/tgkeysbot/docs/API.md`
- `my_office/projects/tgkeysbot/docs/DEPLOYMENT.md`
- `my_office/projects/tgkeysbot/docs/OPEN_QUESTIONS.md`
- `my_office/projects/tgkeysbot/docs/STATUS.md`
- `.gitkeep` в `docs/adr/`, `docs/runbooks/`, `specs/active/`,
  `specs/archived/` (пока без содержимого)
- Эта задача (`tasks/active/TASK-0001-...md`, будет перемещена в
  `tasks/completed/` по завершении).

### Открытые вопросы

Полный список (23 пункта, включая 5 находок безопасности) — в
[`../../docs/OPEN_QUESTIONS.md`](../../docs/OPEN_QUESTIONS.md). Кратко по
категориям: безопасность (запекание секретов/сессии в образ, похожие на
настоящие значения в `.env.example`, root-контейнеры, публичный порт
8080, незашифрованное хранение Pyrogram-кредов в БД), технические
неопределённости (неподключённый Celery-код, неиспользуемые модели/поля,
отсутствие лимита баланса, конкурентность Pyrogram-клиента, происхождение
`keyspro.db`/`db.sqlite3`, отсутствие README/git/тестов/CI), и то, что
намеренно не проверялось (значения секретов, содержимое БД, внешние
сервисы/боты, живое поведение публичной админки, функциональное
тестирование сценариев).

## Проверка перед завершением

Ключевые утверждения документации повторно сверены с кодом
(`grep`/`docker exec`) непосредственно перед закрытием задачи — см.
последний пункт раздела "Просмотрено". Расхождений между документацией и
кодом не найдено.
