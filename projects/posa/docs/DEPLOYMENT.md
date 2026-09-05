# DEPLOYMENT — POSA

Последнее обновление: 2026-09-05 (TASK-0001). Факты проверены напрямую:
`docker ps`, содержимое `/etc/nginx/sites-available/`, файлы
`docker-compose*.yml` и `migration.md` в `/home/admin1/posa/`.

## Текущее состояние (подтверждено `docker ps` на 2026-09-05)

| Контейнер | Образ | Порты | Статус |
|---|---|---|---|
| `activation-system` | `activation-system-backend` (собран из `Dockerfile`) | `0.0.0.0:4000->4000`, `127.0.0.1:4001->4000` | Up 2 недели |
| `activation-studio` | `activation-system-studio` | `127.0.0.1:5557->5555` | Up 2 недели |
| `activation-db` | `postgres:16` | `0.0.0.0:7440->5432` | Up 2 недели |

Отдельно на том же хосте также работает `keyspro_auto_sender-key-distributor-1`
(сопутствующий сервис KeysPro, отдельный проект/репозиторий, не входит в
`activation-system`) — на порту `0.0.0.0:8000`.

**Замечание:** `activation-db` проброшен на `0.0.0.0:7440`, то есть доступен
не только с локального хоста. Это не изменялось и не проверялось глубже в
рамках этой задачи (см. [DATABASE.md](DATABASE.md), [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)).

## docker-compose.yml (activation-system/)

Три сервиса:
- `backend` — собирается из локального `Dockerfile`, `env_file: .env`,
  зависит от `db`, порт `4000`, команда пересобирает Tailwind CSS перед
  стартом (`npm run build:wallet && npm start`).
- `studio` — тот же образ, запускает `npx prisma studio` на порту 5555
  (проброшен только на `127.0.0.1:5557`, наружу не открыт).
- `db` — `postgres:16`, БД `activation_system`, пользователь `tguser`.
  Пароль задан прямо в `docker-compose.yml` (`tgpass123`) как значение
  `POSTGRES_PASSWORD` — не через `.env`/секрет. Порт `7440:5432` без
  указания bind-адреса ⇒ слушает на всех интерфейсах.

`docker-compose.override.yml` добавляет ещё один проброс порта backend:
`127.0.0.1:4001:4000` — используется для `api.namo.uz` (см. ниже).

## Dockerfile (activation-system/)

`FROM node:20` → `npm install` → `npx prisma generate` →
`npm run build:wallet` (сборка Tailwind CSS) → `EXPOSE 4000` →
`CMD ["npm", "start"]`. Многостадийной сборки нет, весь код копируется в
образ (`COPY . .`).

## Nginx (на хосте, подтверждено чтением конфигов)

Один и тот же backend (`127.0.0.1:4000`, кроме `api.namo.uz` → `4001`)
обслуживает несколько доменов, различие — только по `server_name`/`Host`,
которое затем читает `checkSubdomain.js` (см. ARCHITECTURE.md):

| Домен | proxy_pass | TLS |
|---|---|---|
| `namo.uz`, `www.namo.uz` | `127.0.0.1:4000` | Let's Encrypt (`namo.uz`) |
| `wallet.namo.uz` | `127.0.0.1:4000` | Let's Encrypt (`wallet.namo.uz`) |
| `merchant.namo.uz` | `127.0.0.1:4000` | Let's Encrypt (`merchant.namo.uz`) |
| `vendor.namo.uz` | `127.0.0.1:4000` | сертификат `merchant.namo.uz` (переиспользован) |
| `office.namo.uz` | `127.0.0.1:4000` | сертификат `merchant.namo.uz` (переиспользован) |
| `api.namo.uz` | `127.0.0.1:4001` | Let's Encrypt (`api.namo.uz`), HTTP/2 |
| `a.namo.uz` | `127.0.0.1:4000` | Let's Encrypt (`a.namo.uz`) |
| `ppdev.namo.uz` | `127.0.0.1:5557` (**Prisma Studio**, см. ниже) | Let's Encrypt (`ppdev.namo.uz`) |

В `sites-available` также есть несколько файлов `api.namo.uz.conf.bak.*` —
исторические резервные копии конфига, не активны (не в `sites-enabled`).

### ppdev.namo.uz → Prisma Studio

Домен `ppdev.namo.uz` через Nginx с валидным TLS-сертификатом проксирует на
`127.0.0.1:5557`, который в `docker-compose.yml` — это проброс порта
контейнера `studio` (`npx prisma studio`, порт 5555 внутри контейнера).
`docker-compose.yml` сам по себе биндит его только на `127.0.0.1`, но Nginx
на этом же хосте делает его доступным публично под доменным именем. Prisma
Studio даёт полный просмотр/редактирование всех таблиц БД через браузер.
Подтверждено: конфиг `ppdev.namo.uz` включает `auth_basic "Restricted"` с
`auth_basic_user_file /etc/nginx/.htpasswd-studio` — доступ защищён Basic
Auth на уровне Nginx. Содержимое `.htpasswd-studio` (список учётных записей,
стойкость паролей) не проверялось.

## Процесс миграции/развёртывания на новый сервер

Задокументирован проектом самим в `/home/admin1/posa/migration.md` (не
переписывается здесь дословно, см. оригинал). Кратко, по шагам файла:
1. Подготовка сервера (Docker, ufw, порты 22/80/443, опционально 5432).
2. Клонирование `git@github.com:jedimindtricks420/POSA-v3.git` в
   `/home/admin1/posa`.
3. Импорт дампа `activation-system/posa.sql` в контейнер `activation-db`.
4. `docker-compose up -d --build` для `activation-system` и отдельно для
   `keyspro_auto_sender`.
5. Настройка Nginx под домены `namo.uz`/`api.namo.uz`/`webhook.keyspro.uz`.
6. Верификация через `/home/admin1/posa/verify_migration.sh` (проверяет
   systemd-статус Nginx, наличие контейнеров по имени, HTTP-коды доменов).

Это описание из документации проекта, не проверялось путём фактического
повторного разворачивания в рамках этой задачи.

## Приложение my-pwa (Android)

Отдельный процесс сборки, не через Docker/Nginx: Gradle-проект,
сгенерированный `bubblewrap-cli` (см. ARCHITECTURE.md). Артефакты сборки
присутствуют в каталоге (`app-release-signed.apk`, `.aab`) — процесс их
публикации (Google Play и т.п.) не задокументирован в репозитории.
