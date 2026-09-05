# DEPLOYMENT — keyspro_auto_sender

Последнее обновление: 2026-09-05 (TASK-0001). Источник —
`docker-compose.yml`, `Dockerfile`, `/etc/nginx/sites-available/
keyspro_auto_sender`, `docker ps`/`docker inspect` на хосте.

## Текущее состояние (на момент проверки)

- Хост: сервер, на котором расположен `/home/admin1/keyspro_auto_sender/`
  (README проекта указывает "VM1 (4.240.102.156)" — не перепроверялось
  отдельно в рамках этой задачи).
- Контейнер `keyspro_auto_sender-key-distributor-1`: статус `healthy`,
  поднят (`StartedAt`) 2026-08-20, образ `keyspro_auto_sender-
  key-distributor` собран локально 2026-03-26 (не пересобирался с тех пор
  на момент проверки, несмотря на более поздние коммиты в git — см.
  [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).
- Порт 8000 контейнера опубликован на `0.0.0.0:8000` хоста.
- Управление — только через Docker Compose CLI (`docker compose up -d /
  down / restart`), нет systemd unit или иного supervisor поверх Docker
  (не проверялось специально, но не упоминается ни в README, ни в конфигах).

## Docker Compose

Один сервис `key-distributor`:

- `build: .` — образ собирается из локального `Dockerfile`.
- `ports: 8000:8000`.
- `env_file: .env`.
- `volumes` (bind mounts): `./.env:/app/.env`, `./product_config.json:/app/
  product_config.json`, `./app:/app/app`. **Файл БД `keyspro.db` не
  примонтирован** — см. [DATABASE.md](DATABASE.md).
- `restart: unless-stopped`.
- `healthcheck`: `curl -f http://localhost:8000/health`, interval 30s,
  timeout 10s, retries 3, start_period 40s.

## Dockerfile

- Базовый образ `python:3.11-slim`.
- Системные зависимости: `build-essential`, `curl`.
- Зависимости ставятся из `requirements.txt` через `pip install`.
- Копируется весь код (`COPY . .`).
- Создаётся non-root пользователь `app`, контейнер работает от него.
- `CMD uvicorn app.main:app --host 0.0.0.0 --port 8000` (без `--reload`,
  без указания числа workers — один процесс uvicorn).

## Nginx (хост)

Конфиг `/etc/nginx/sites-available/keyspro_auto_sender` (443, TLS через
Certbot, HTTP → HTTPS редирект на 80):

```
server_name webhook.keyspro.uz;
location /               → proxy_pass http://127.0.0.1:8000;   # keyspro_auto_sender
location /partner_bot/   → proxy_pass http://127.0.0.1:8080/;  # другой backend, не в этом репозитории
```

`/partner_bot/` — отдельное приложение ("Partner Bot admin panel" по
комментарию в конфиге), делящее домен `webhook.keyspro.uz` с этим
сервисом. Что это за приложение и как оно связано с проектом — не
исследовалось (вне области задачи).

## Известные операционные инструкции (из README проекта)

Ниже — то, что описано в `README.md` `/home/admin1/keyspro_auto_sender/`
(зафиксировано как задокументированная процедура, не перепроверялось
выполнением):

- Пересоздание Pyrogram `SESSION_STRING` при ошибке `AUTH_KEY_DUPLICATED`
  (сессия используется в нескольких местах одновременно).
- Экранирование `$` → `$$` в `SMTP_PASSWORD` в `.env` (иначе Docker
  Compose интерпретирует `$` как переменную окружения).
- Процедура миграции на новый сервер: скопировать каталог проекта +
  Nginx-конфиг, поднять Docker, обновить DNS, получить новый TLS
  сертификат, **обязательно остановить контейнер на старом сервере**
  (во избежание `AUTH_KEY_DUPLICATED` от параллельно работающей
  Pyrogram-сессии).

## Не проверялось

- Наличие/содержимое резервных копий (`keyspro.db`, `.env`).
- Мониторинг/алертинг за пределами Docker healthcheck.
- Firewall-правила хоста (открыт ли порт 8000 наружу помимо Nginx).
