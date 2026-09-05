# DEPLOYMENT — drweb-key-service

Последнее обновление: 2026-09-05 (TASK-0001). Факты проверены напрямую:
`docker ps`/`docker port`, `docker-compose.yml`, `Dockerfile`, поиск по
`/etc/nginx/sites-available/` и `/etc/nginx/sites-enabled/`.

## Текущее состояние (подтверждено `docker ps` на 2026-09-05)

| Контейнер | Образ | Порты | Статус |
|---|---|---|---|
| `drweb-api` | `drweb-key-service-drweb-api` (собран из общего `Dockerfile`) | `0.0.0.0:8001->8001`, `[::]:8001->8001` | Up 2 недели (создан 4 недели назад) |
| `drweb-admin` | `drweb-key-service-drweb-admin` | `0.0.0.0:8002->8002`, `[::]:8002->8002` | Up 2 недели (создан 4 недели назад) |

Оба порта проброшены на все интерфейсы (`0.0.0.0`/`[::]`), не только на
`127.0.0.1`.

## Nginx (на хосте, подтверждено поиском по конфигам)

Конфигурации для портов `8001`/`8002` или упоминаний `drweb` в
`/etc/nginx/sites-available/` и `/etc/nginx/sites-enabled/` **не найдено**.
Это означает, что оба сервиса (FastAPI-API и Django-админка) доступны
напрямую по IP хоста без reverse-proxy и без TLS на уровне Nginx.
`project_docs/API_INTEGRATION.md` подтверждает это косвенно, указывая
базовый URL как `http://4.240.102.156:8001/drweb` (обычный HTTP, IP вместо
домена).

## docker-compose.yml

Два сервиса, оба собираются из одного `build: .` (общий `Dockerfile`):

- **`drweb-api`**: `container_name: drweb-api`, порт `8001:8001`, volume
  `./data:/app/data`, `env_file: .env`, `restart: always`, команда
  `uvicorn app.main:app --host 0.0.0.0 --port 8001`.
- **`drweb-admin`**: `container_name: drweb-admin`, порт `8002:8002`,
  тот же volume и `.env`, `restart: always`, составная команда:
  `python manage.py migrate`, затем через `manage.py shell -c` — создание
  Django-суперпользователя `office_admin1` с паролем `merchant123`, если
  такого пользователя ещё нет, затем `python manage.py runserver
  0.0.0.0:8002`.

**Важно:** логин/пароль администратора Django (`office_admin1` /
`merchant123`) записаны прямым текстом в `docker-compose.yml`, который
отслеживается git (подтверждено `git ls-files`) — то есть эта пара
логин/пароль присутствует в истории репозитория в открытом виде, а не
только в `.env`.

## Dockerfile

`FROM python:3.11-slim` → `pip install -r requirements.txt` →
`COPY . .` → `mkdir -p /app/data` → `ENV DATABASE_URL=sqlite:////app/data/
drweb_keys.db` → `EXPOSE 8001` → `CMD ["uvicorn", "app.main:app",
"--host", "0.0.0.0", "--port", "8001"]`. Многостадийной сборки нет, весь
код копируется в образ. `CMD` по умолчанию запускает только FastAPI —
команда для `drweb-admin` в `docker-compose.yml` переопределяет его.

## Эксплуатация (по `project_docs/PROJECT_OVERVIEW.md`, совпадает с фактом расположения)

```bash
cd /home/admin1/drweb-key-service
docker-compose up -d --build
```

Админ-панель — `http://<IP хоста>:8002/admin`, стартовый логин
`office_admin1` (пароль см. выше, задаётся автоматически при первом
старте контейнера, если пользователь ещё не существует).

## Требуемые переменные окружения (только имена, из `.env`, значения не читались)

`DRWEB_LOGIN`, `DRWEB_PWD`, `DRWEB_API_URL`, `DRWEB_TESTING`, `API_KEY`,
`PORT`, `DATABASE_URL`. Файл `.env` присутствует на диске сервера, в
`.gitignore` и не отслеживается git.

## Не проверялось

- Фактическая доступность сервисов извне (порты проброшены на `0.0.0.0`, но
  реальный сетевой запрос с внешнего хоста/через firewall не выполнялся в
  рамках этой задачи).
- Правила firewall (ufw/iptables) на уровне хоста — не проверялись.
- Ресурсные ограничения контейнеров (CPU/RAM лимиты в `docker-compose.yml`
  не заданы).
