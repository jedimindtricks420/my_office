---
id: TASK-0001
title: Bootstrap project context for drweb-key-service
status: completed
created: 2026-09-05
completed: 2026-09-05
owner: azureuser (agent-assisted)
---

# TASK-0001 — Bootstrap project context

## Цель

Зарегистрировать существующий проект `drweb-key-service`
(источник: `/home/admin1/drweb-key-service/`) в `my_office` и создать
документацию его подтверждённого текущего состояния. Без разработки нового
функционала, без исправления кода, без рефакторинга, без домыслов там, где
факт не подтверждён кодом/конфигурацией.

## Журнал работы

### 2026-09-05

- Прочитан корневой `/home/admin1/my_office/AGENTS.md` (через клон
  `/home/azureuser/work/my_office/`, см. [[project_my_office]] в памяти
  агента — `my_office` на диске принадлежит `admin1`, `azureuser` пишет
  через собственный клон + `git push`).
- Изучена структура уже зарегистрированного проекта `projects/posa/` как
  образец оформления (AGENTS.md/README.md/docs/*/TASK-0001).
- Создана структура `projects/drweb-key-service/{docs/{adr,runbooks},
  specs/{active,archived},tasks/{active,completed}}`.

### Что было просмотрено

- Полный список файлов репозитория (`find`, минус `.git`): `.env`,
  `.gitignore`, `Dockerfile`, `docker-compose.yml`, `manage.py`,
  `requirements.txt`, `test_cffi.py`, `Edited_DrWeb esd uz.csv`,
  `app/{database.py,drweb_client.py,main.py,models.py,products.json,
  schemas.py}` (все прочитаны полностью), `core/{admin.py,apps.py,models.py,
  migrations/__init__.py}` (все прочитаны, `migrations/__init__.py` пустой —
  реальных файлов миграций нет), `drweb_admin/{settings.py,urls.py,wsgi.py}`
  (все прочитаны), `project_docs/{PROJECT_OVERVIEW.md,API_INTEGRATION.md}`
  (прочитаны полностью).
- `requirements.txt` — список зависимостей без версий (кроме
  `curl_cffi==0.14.0`).
- `.env`: прочитаны ТОЛЬКО имена переменных (`grep -oE`), значения не
  читались и не копировались: `DRWEB_LOGIN`, `DRWEB_PWD`, `DRWEB_API_URL`,
  `DRWEB_TESTING`, `API_KEY`, `PORT`, `DATABASE_URL`.
- `app/products.json` — посчитано количество записей программно (`json.load`
  + `len()` = 7), структура одной записи прочитана как пример.
- `Edited_DrWeb esd uz.csv` — прочитаны только первые строки (заголовок +
  2 строки) для понимания структуры, полное содержимое не копировалось в
  документацию (в нём такой же паттерн URL с плейсхолдерами LOGIN/PWD).
- `git log`, `git branch -a`, `git remote -v`, `git status`, `git ls-files` —
  4 коммита, remote `git@github.com:jedimindtricks420/dr.web-custom-api.git`,
  автор `Ahmad <crosssectionagency@gmail.com>`, working tree чистый.
  Подтверждено через `git ls-files`: `.env`, `test_cffi.py`, `data/`,
  `Edited_DrWeb esd uz.csv` НЕ отслеживаются git (список файлов в репозитории
  не включает их) — соответствует `.gitignore`.
- `docker ps -a --filter name=drweb`, `docker images`, `docker port` — оба
  контейнера (`drweb-api`, `drweb-admin`) запущены (создан образ 4 недели
  назад, `Up 2 недели` на момент проверки), порты `8001` и `8002` проброшены
  на `0.0.0.0` (и `[::]`) — то есть наружу, не только на localhost.
- `stat data/` — каталог `data/` имеет права `777`.
- Поиск конфигурации Nginx для портов 8001/8002/`drweb` в
  `/etc/nginx/sites-available/` и `/etc/nginx/sites-enabled/` — ничего не
  найдено. Значит, оба сервиса торчат в интернет напрямую, без
  reverse-proxy/TLS на уровне Nginx на этом хосте.
- Поиск `.github/workflows`, `Makefile`, `pytest.ini`, `.venv`/`venv` —
  ничего не найдено (нет CI, нет виртуального окружения на диске, нет
  конфигурации тестового фреймворка).
- Соседний каталог `/home/admin1/tgkeysbot` (упомянут в
  `project_docs/API_INTEGRATION.md` как пример клиента) — подтверждено
  существование на диске (`find`), содержимое НЕ исследовалось (вне области
  задачи, только `drweb-key-service`).

### Что установлено (ключевые факты)

- Два независимых процесса в одном образе (`Dockerfile` общий), оркестрованы
  `docker-compose.yml`: `drweb-api` (FastAPI/uvicorn, порт 8001) и
  `drweb-admin` (Django 5.x runserver, порт 8002). Оба используют один и тот
  же файл SQLite `data/drweb_keys.db` — FastAPI через SQLAlchemy (синхронный
  `create_engine`, несмотря на наличие `aiosqlite`/`databases` в
  `requirements.txt` — не подтверждено использование этих двух пакетов в
  коде), Django ORM — через модели с `managed = False` (таблицы создаёт и
  ведёт SQLAlchemy/Alembic-подобным способом `Base.metadata.create_all` в
  `app/database.py`, Django только читает/пишет в них).
- Единственный бизнес-эндпоинт `POST /drweb/api/v1/generate` защищён
  статическим Bearer-токеном (`API_KEY` из `.env`), сравнение — обычное `!=`
  (не constant-time). Есть также `/drweb/api/v1/products` и
  `/drweb/api/v1/logs` (тоже под тем же токеном) и публичный `/health`.
- `app/drweb_client.py` подставляет логин/пароль Dr.Web в URL-шаблон и
  выполняет запрос через `curl_cffi` с `impersonate="chrome"` (обход QRATOR).
  Строка `logger.info(f"Requesting Dr.Web: url={final_url}")` перед запросом
  логирует итоговый URL **вместе с уже подставленными логином и паролем** —
  то есть учётные данные Dr.Web попадают в логи контейнера в открытом виде.
  Подтверждено прямым чтением кода (`app/drweb_client.py`, `app/main.py`).
- `docker-compose.yml` (файл отслеживается git) содержит захардкоженную
  команду создания Django-суперпользователя с логином `office_admin1` и
  паролем `merchant123` прямо в тексте команды — этот пароль находится в
  открытом виде в самом репозитории (не в `.env`).
- `drweb_admin/settings.py`: `SECRET_KEY` захардкожен
  (`'django-insecure-drweb-admin-panel-secret'`), `DEBUG = True`,
  `ALLOWED_HOSTS = ['*']` — всё подтверждено прямым чтением файла.
- Корневого `README.md` в проекте нет — есть `project_docs/PROJECT_OVERVIEW.md`
  (на русском, описывает архитектуру и назначение) и
  `project_docs/API_INTEGRATION.md` (руководство для клиентов API, с примером
  на `aiohttp` и упоминанием `tgkeysbot` как потребителя).
- Автоматических тестов нет (`test_cffi.py` — ручной скрипт для проверки
  `curl_cffi` с захардкоженными в открытом виде похожими на боевые логином/
  паролем Dr.Web; файл существует на диске, но не отслеживается git).
- `requirements.txt` содержит `django-environ`, но `drweb_admin/settings.py`
  использует напрямую `os.environ`, не `django_environ` — использование
  пакета в коде не подтверждено.
- Нет конфигурации CI/CD, нет линтера/форматтера, нет виртуального окружения
  на диске.

### Какие документы созданы

`projects/drweb-key-service/{README.md, AGENTS.md}` и
`projects/drweb-key-service/docs/{OVERVIEW,ARCHITECTURE,STACK,STATUS,
DATABASE,API,DEPLOYMENT,OPEN_QUESTIONS}.md`, плюс `.gitkeep` в
`docs/adr/`, `docs/runbooks/`, `specs/active/`, `specs/archived/`.

### Какие вопросы остались

Полный список — [`docs/OPEN_QUESTIONS.md`](../../docs/OPEN_QUESTIONS.md).
Коротко: связь с собственной Dr.Web-интеграцией внутри проекта POSA
(`DrWebOrder`/`DrWebApiLog`/`DrWebSku` в его Prisma-схеме — дублирование или
независимые системы, не выяснялось), реальное использование
`django-environ`/`aiosqlite`/`databases`, назначение `Edited_DrWeb esd
uz.csv` как источника данных для `products.json` (совпадение по структуре
URL, но процесс синхронизации между ними не подтверждён), содержимое
дампа/данных в `data/drweb_keys.db` не проверялось (только структура таблиц
через код моделей).

### Финальная сверка (2026-09-05)

После создания документации выполнена повторная сверка ключевых
утверждений напрямую по коду/конфигурации:
- `app/main.py`: `FastAPI(title=...)` создан без `docs_url=None`/
  `redoc_url=None` — автодокументация не отключена явно (сама доступность
  по сети не проверялась, см. OPEN_QUESTIONS.md).
- `requirements.txt`: `pytest`/`unittest`-зависимостей нет — подтверждает
  вывод об отсутствии автотестов.
- `git ls-files` (22 файла) и `git check-ignore -v` — повторно подтверждено:
  `.env`, `test_cffi.py`, `Edited_DrWeb esd uz.csv`, `data/` игнорируются
  git и не отслеживаются; `docker-compose.yml` (с захардкоженным паролем
  админа) отслеживается.
- `project_docs/` содержит ровно два файла (`API_INTEGRATION.md`,
  `PROJECT_OVERVIEW.md`) — соответствует тому, что описано в OVERVIEW.md.

Расхождений между созданной документацией и кодом при сверке не найдено.

## Статус: завершена

Все пункты чек-листа задачи выполнены: код `/home/admin1/drweb-key-service/`
не изменялся, документация основана только на подтверждённых фактах,
неподтверждённое вынесено в OPEN_QUESTIONS.md, STATUS.md обновлён.
