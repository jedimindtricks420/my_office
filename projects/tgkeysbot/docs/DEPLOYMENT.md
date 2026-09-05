# DEPLOYMENT — tgkeysbot

Последнее обновление: 2026-09-05 (TASK-0001). Источник —
`docker-compose.yml`, `Dockerfile`, `/etc/nginx/sites-available/
keyspro_auto_sender` и `docker ps`/`docker inspect`/`docker exec` на хосте.

## Текущее состояние (на момент проверки, 2026-09-05)

Три контейнера подняты и работают на одном хосте с проектом `tgkeysbot`:

| Контейнер | Образ | Создан | Запущен (`StartedAt`) | Статус |
|---|---|---|---|---|
| `tgkeysbot_db_1` | `postgres:16-alpine` | ~5 мес. назад | — | `healthy` |
| `tgkeysbot_web_1` | `tgkeysbot_web:latest` (собран 2026-03-25T14:38 UTC) | 2026-03-25 | 2026-08-20 | `Up` |
| `tgkeysbot_bot_1` | `tgkeysbot_bot:latest` (тот же образ) | 2026-03-25 | 2026-08-20 | `Up` |

Образ собран из `Dockerfile` локально (не из внешнего registry). Дата
сборки образа (2026-03-25T14:38 UTC) позже mtime всех файлов исходного
кода на диске на момент проверки (последний изменённый `.py`-файл —
`config/settings.py`, 2026-03-25 14:36) — то есть **работающий образ
соответствует текущему состоянию кода на диске**, расхождений не
обнаружено. Контейнеры не пересоздавались с момента сборки, только
перезапускались (`StartedAt` 2026-08-20 при более раннем `Created`
2026-03-25).

## Docker Compose (`docker-compose.yml`)

- **`db`**: `postgres:16-alpine`, БД/пользователь/пароль `keysbot`
  (пароль — из `POSTGRES_PASSWORD`, дефолт в файле `keysbot_secret` —
  используется только если переменная не задана в `.env`), volume
  `postgres_data`, healthcheck `pg_isready -U keysbot`.
- **`web`**: `build: .`, команда `python manage.py migrate && gunicorn
  config.wsgi:application --bind 0.0.0.0:8000`, порт `8080:8000`, `depends_
  on: db (service_healthy)`, volume `media_data:/app/media`,
  `DATABASE_URL`/`SECRET_KEY` из переменных окружения compose +
  `DEBUG=False`/`ALLOWED_HOSTS=*` захардкожены в самом
  `docker-compose.yml` (не из `.env`), плюс `env_file: .env`.
- **`bot`**: `build: .`, команда `python bot/main.py`, без публикации
  портов, тот же `DATABASE_URL` и `env_file: .env`.
- Оба `web` и `bot` используют один и тот же `media_data` volume, но не
  разделяют код через volume — код "заморожен" в образе на момент сборки
  (нет bind mount исходников).
- `restart: unless-stopped` на всех трёх сервисах.

Подтверждено `docker exec`: в контейнерах `DEBUG=False`, `ALLOWED_HOSTS=*`
(соответствует значениям из `docker-compose.yml`), `DATABASE_URL` указывает
на `postgres://keysbot@db:5432/keysbot` (пароль замаскирован при
проверке), переменная `FORCE_SCRIPT_NAME` в окружении контейнера
**отсутствует** (не установлена, несмотря на то что `config/settings.py`
явно предусматривает её для деплоя под префиксом, например
`/partner_bot` — см. ниже и [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).

## Dockerfile

- Multi-stage: builder (`python:3.12-slim` + `build-essential`/`libpq-dev`)
  собирает wheels из `requirements.txt`; финальный образ
  (`python:3.12-slim` + `libpq5`) ставит их из `/wheels`.
- `COPY . .` копирует весь каталог проекта в `/app` **без ограничений**,
  кроме перечисленного в `.dockerignore`. `.dockerignore` исключает
  `.env`, `db.sqlite3`, `staticfiles/`, `*.log`, но **не исключает**
  `session_generator.session` (файл живой Pyrogram-сессии),
  `keyspro.db` и `media.tar.gz` — подтверждено `docker exec ... ls /app/`:
  все три файла присутствуют внутри образа/контейнеров. См.
  [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md) — находка безопасности.
- `RUN python manage.py collectstatic --noinput --clear` на этапе сборки
  (ошибки подавляются `|| true`).
- Не создаётся отдельный non-root пользователь — контейнер работает от
  `root` (подтверждено `docker exec ... whoami`).
- `CMD ["python", "bot/main.py"]` — дефолтная команда образа (используется
  сервисом `bot`), для `web` она переопределяется в `docker-compose.yml`.

## Nginx (хост) — общий домен с другим проектом

Домен `webhook.keyspro.uz` (виртуалхост принадлежит проекту
`keyspro_auto_sender`, конфиг `/etc/nginx/sites-available/
keyspro_auto_sender`, TLS через Certbot):

```
server_name webhook.keyspro.uz;
location /               → proxy_pass http://127.0.0.1:8000;   # keyspro_auto_sender, другой проект
location /partner_bot/         → proxy_pass http://127.0.0.1:8080/;         # tgkeysbot (Django admin)
location /partner_bot/static/  → proxy_pass http://127.0.0.1:8080/static/;
location /partner_bot/media/   → proxy_pass http://127.0.0.1:8080/media/;
```

Это подтверждённый ответ на открытый вопрос в документации соседнего
проекта (`../keyspro_auto_sender/docs/OPEN_QUESTIONS.md`, пункты №8/№15 на
момент их фиксации 2026-09-05): backend на `127.0.0.1:8080` — это
Django-админка `tgkeysbot` ("Partner Bot admin panel" по комментарию в
самом nginx-конфиге).

**Несоответствие:** `config/settings.py` содержит `FORCE_SCRIPT_NAME`,
явно предназначенный для работы за префиксом вида `/partner_bot`
(комментарий в коде: `# URL prefix for reverse proxy (e.g. /partner_bot)`),
но эта переменная не установлена в фактическом окружении контейнера
`web` — подтверждено `docker exec tgkeysbot_web_1 env`. Nginx физически
убирает префикс при проксировании (`proxy_pass .../` с завершающим
слэшем), поэтому базовая маршрутизация работает, но без
`FORCE_SCRIPT_NAME` Django, вероятно, генерирует статические/ссылочные URL
без префикса `/partner_bot` — фактическое поведение через браузер/curl к
публичному домену не проверялось в рамках этой задачи (см.
[OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).

`CSRF_TRUSTED_ORIGINS = ['https://webhook.keyspro.uz']` захардкожен в
`config/settings.py` (не читается из `.env`) — жёстко привязывает проект к
этому единственному домену.

## Не проверялось

- Поведение админки при реальном заходе через публичный URL
  `https://webhook.keyspro.uz/partner_bot/` (генерация ссылок на статику,
  корректность форм с учётом `FORCE_SCRIPT_NAME`) — не тестировалось живым
  запросом в рамках задачи.
- Резервное копирование БД/`media_data` volume.
- Мониторинг/алертинг сверх Docker `healthcheck` контейнера `db` (у `web`
  и `bot` healthcheck не определён в `docker-compose.yml`).
- Firewall хоста для портов 8080/5432 сверх того, что 5432 у `db` не
  опубликован наружу (нет `ports:` у сервиса `db` в `docker-compose.yml`),
  а 8080 у `web` опубликован на `0.0.0.0` (подтверждено `docker ps`).
