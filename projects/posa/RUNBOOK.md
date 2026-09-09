# RUNBOOK

Срез локального исходного кода: 2026-09-08. Корень ссылок — `/home/admin1/posa`; локальные незакоммиченные изменения включены. Источники — исполняемый код, схемы и конфигурация; существующая документация и комментарии не используются как доказательства. Статус фактов ниже: **подтверждено кодом**, если явно не указано «предположение на основе имени и использования». Статический анализ не подтверждает состояние рабочей БД или успешный запуск.

## Границы проверки

Документация получена статически. Сервер, БД, SMS, платежи и тесты с изменениями данных не запускались. Команды ниже предназначены для отдельной среды; успешный запуск текущего проекта с пустой БД не подтверждён из-за дефектов миграций. `activation-system/prisma/migrations/20250211120000_add_refresh_token/migration.sql:19`.

## Установка Node-приложения

Рабочий каталог должен быть `activation-system`: сертификатная модель, uploads и PDF используют process.cwd()/относительные пути. Dockerfile использует Node 20, npm install, prisma generate и build:wallet. `activation-system/Dockerfile:1` `activation-system/Dockerfile:3` `activation-system/routes/wallet.js:26` `activation-system/middleware/uploadMiddleware.js:6`.

```sh
cd /home/admin1/posa/activation-system
npm install
npx prisma generate
npm run build:wallet
```

Первые три команды соответствуют Dockerfile; build:wallet запускает build:wallet:css с Tailwind. Отдельного frontend bundle нет в scripts. `activation-system/Dockerfile:5` `activation-system/package.json:10`.

Перед запуском окружение должно содержать DATABASE_URL, session secret, обязательные PassKit-переменные и рабочие файлы сертификатов; ManaStore требует credentials выбранного контура либо явно включённый mock. Значения и обязательность перечислены в CONFIGURATION. `activation-system/index.js:111` `activation-system/routes/wallet.js:30` `activation-system/services/manastoreService.js:38`.

```sh
npm start
# либо разработка
npm run dev
```

start = node index.js; dev = nodemon index.js. Приложение слушает PORT или 4000. `.env` подхватывается dotenv; для исключения зависимости от порядка ESM-импортов переменные лучше передавать процессу до старта. `activation-system/package.json:8` `activation-system/index.js:27` `activation-system/index.js:219`.

## Docker Compose

```sh
cd /home/admin1/posa/activation-system
docker compose config --services
docker compose up -d --build
```

Это эксплуатационные команды для объявленных сервисов backend, studio, db. Конфиг запускает backend через `npm run build:wallet && npm start`, studio через Prisma Studio, db — postgres:16. Depends_on задаёт зависимость, но healthcheck в compose не объявлен. `activation-system/docker-compose.yml:2` `activation-system/docker-compose.yml:16` `activation-system/docker-compose.yml:31` `activation-system/docker-compose.yml:34`.

Публикации в основном файле: backend 4000:4000, PostgreSQL 7440:5432, Studio 127.0.0.1:5557:5555; override добавляет 127.0.0.1:4001:4000. Проверять итоговый compose перед запуском: override сам по себе не подтверждает, что 4000 закрыт. Данные PostgreSQL — named volume postgres_data; приложение bind-mounted в /app, node_modules — отдельный volume. `activation-system/docker-compose.yml:7` `activation-system/docker-compose.yml:13` `activation-system/docker-compose.yml:29` `activation-system/docker-compose.yml:41` `activation-system/docker-compose.override.yml:5`.

## База данных и миграции

`npx prisma migrate deploy` — логичная команда для Prisma-миграций, но в scripts она не определена и для текущей цепочки с пустой БД неработоспособна: RefreshToken FK выполняется раньше создания User/Client, более поздние миграции изменяют отсутствующую Kassa. Это **предположение на основе имени и использования** команды Prisma; дефекты SQL подтверждены исходниками. `activation-system/package.json:7` `activation-system/prisma/migrations/20250211120000_add_refresh_token/migration.sql:19` `activation-system/prisma/migrations/20260501071200_add_click_merchant_user_id_and_env/migration.sql:2`.

Воспроизводимое создание текущей схемы с нуля не реализовано. Нельзя описать успешный bootstrap без изменения исходников или подтверждённого baseline БД. Не смешивать ручной manual_migration_qr_payment.sql с Prisma-миграцией QR: они создают одни и те же таблицы, а ручной файл объявляет enum позже его использования. `activation-system/manual_migration_qr_payment.sql:14` `activation-system/manual_migration_qr_payment.sql:33` `activation-system/prisma/migrations/20251224192700_add_qr_payment_feature/migration.sql:5`.

При уже подготовленной совместимой БД есть отдельный seed:

```sh
node prisma/seed-kassa.js
```

Он создаёт кассу при отсутствии id=1, присоединяет все записи с null kassaId в Vendor/VoucherTransaction/QrPaymentAttempt и пересчитывает кассовые суммы. Это изменение существующих данных, не нейтральная проверка и не создание администратора. `activation-system/prisma/seed-kassa.js:18` `activation-system/prisma/seed-kassa.js:47` `activation-system/prisma/seed-kassa.js:70`.

## Проверки после установки

- Проверить чтение сертификатов и наличие passModels/storeCard.pass: отсутствие приводит к исключению при импорте wallet router. `activation-system/routes/wallet.js:12` `activation-system/routes/wallet.js:26`.
- Проверить доступность /auth/login и клиентского /wallet с Host wallet.*. Различие Host влияет на маршрутизацию. `activation-system/routes/authRoutes.js:8` `activation-system/middleware/checkSubdomain.js:4` `activation-system/index.js:194`.
- Проверить создание session-таблицы и сохранение сессии в PostgreSQL: store использует DATABASE_URL, tableName session. `activation-system/index.js:110`.
- Проверить возможность записи public/uploads и receipts. Middleware создаёт uploads, QR-процесс создаёт receipts; обычная merchant-продажа рассчитывает путь и сразу пишет файл. `activation-system/middleware/uploadMiddleware.js:6` `activation-system/controllers/public/qrPaymentController.js:268` `activation-system/controllers/merchant/saleController.js:603`.
- Проверить /manifest.json, /sw.js и /.well-known/assetlinks.json; URL assetlinks имеет отдельный handler, возвращающий application/json либо 500. `activation-system/index.js:83` `activation-system/public/js/register-sw.js:51`.

Отдельного подключённого /health нет: health объявлен лишь в неимпортируемом routes/apiV1.js. Не использовать его как readiness probe этой версии. `activation-system/routes/apiV1.js:4` `activation-system/index.js:18`.

## Известные симптомы

| Симптом | Причина по коду | Источник |
| --- | --- | --- |
| Старт завершается до app.listen | Отсутствуют PassKit файлы/ENV или ManaStore credentials | `activation-system/routes/wallet.js:14` `activation-system/services/manastoreService.js:40` |
| Локальный /wallet не открывает вход | Нужен Host wallet.* для корневого клиентского пути; mount также содержит /wallet внутри | `activation-system/routes/clientRoutes.js:26` `activation-system/index.js:194` |
| Отчёт активаций возвращает ошибку Prisma | createdAt в фильтре/сортировке VoucherActivation, но в модели activatedAt | `activation-system/controllers/admin/drwebController.js:245` `activation-system/prisma/schema.prisma:387` |
| Ручные детали / старые страницы дают ошибку render | Соответствующие EJS отсутствуют | `activation-system/controllers/admin/manualActivationController.js:120` `activation-system/controllers/publicController.js:32` |
| Rokky возвращает MOCK-KEY в любом окружении | &#124;&#124; true делает MOCK_MODE истинным | `activation-system/services/rokkyService.js:8` |
| Код ManaStore остаётся pending | GET code-status читает только БД; провайдера опрашивает POST requestVoucherCode | `activation-system/services/walletActivationService.js:340` |
| Продажа PAID, callback сообщает ошибку обработки | PDF-сбой после commit возвращает success:false; PAID сохраняется | `activation-system/controllers/public/qrPaymentController.js:442` `activation-system/controllers/public/qrPaymentController.js:462` |
| Клиент видит offline 202 | SW сохранил POST в очередь; это не подтверждение успешной операции | `activation-system/public/sw.js:408` |


## Контракты существующих тестов

| Скрипт | Что фактически проверяет/делает | Ограничение | Источник |
| --- | --- | --- | --- |
| test-login-unregistered.js | POST /wallet; ищет строку ошибки и ссылку регистрации | Нет assert/ненулевого exit при отрицательном результате; node-fetch отсутствует в dependencies; localhost Host не wallet.* | `activation-system/test-login-unregistered.js:2` `activation-system/test-login-unregistered.js:23` |
| test-account-deletion.js | HTTP login → фиксированный OTP → delete-account; проверка deleteResult.ok | Пишет в БД через API; OTP фиксирован, но wallet OTP случайный; сообщение об успешной верификации без проверки ответа | `activation-system/test-account-deletion.js:2` `activation-system/test-account-deletion.js:45` |
| test-deletion-direct.js | Создаёт/находит Client; заменяет phoneNumber и name; пишет AuditLog; проверяет отсутствие старого телефона | Не вызывает HTTP-контроллер; нет assert, данные остаются | `activation-system/test-deletion-direct.js:17` `activation-system/test-deletion-direct.js:34` `activation-system/test-deletion-direct.js:83` |
| test-deletion-with-vouchers.js | Upsert клиента, читает связи, анонимизирует, снова читает связи | Не создаёт сами ваучеры; сообщение о сохранности без сравнения/assert | `activation-system/test-deletion-with-vouchers.js:10` `activation-system/test-deletion-with-vouchers.js:58` `activation-system/test-deletion-with-vouchers.js:73` |
| test-payme-transaction.sh | CheckPerform → Create → Perform; jq проверяет allow и state 1/2, иначе exit 1 | Публичный URL без обязательного kassaId, литеральный секрет, вызовы меняют состояние платежа | `test-payme-transaction.sh:8` `test-payme-transaction.sh:51` `test-payme-transaction.sh:106` `test-payme-transaction.sh:151` |
| verify_migration.sh | Проверки systemctl/docker/curl | check_url получает url/name, но использует name как hostname; вызовы передают человекочитаемые названия. Не проверяет миграции Prisma | `verify_migration.sh:34` `verify_migration.sh:47` |
| public/js/test-delete-elements.js | Диагностика DOM элементов удаления аккаунта в браузере | Не автоматический тест backend | `activation-system/public/js/test-delete-elements.js:1` |


В package.json нет test-script, тестового фреймворка и прямой зависимости node-fetch. Тестовый успешный контракт не следует приравнивать к результату запуска: проверки не выполнялись. `activation-system/package.json:7` `activation-system/package.json:19`.

## Android / TWA

Конфигурация Android: applicationId uz.namo.wallet.twa; versionCode 3, versionName 1.0.1; minSdk 21, targetSdk 35, compileSdk 36; Android Gradle plugin 8.9.1 и browser helper 2.6.2. Java source/target 1.8 не задают версию JDK, требуемую самой Gradle. `my-pwa/app/build.gradle:54` `my-pwa/app/build.gradle:158` `my-pwa/app/build.gradle:211` `my-pwa/build.gradle:26`.

В settings.gradle включён :app. Типовые задачи `./gradlew :app:assembleRelease` и `./gradlew :app:bundleRelease` — **предположение на основе имени и использования** Android plugin; запуск и подпись не проверены. Путь signingKey/alias задан в twa-manifest.json, но signingConfig в app/build.gradle не определён. `my-pwa/settings.gradle:1` `my-pwa/app/build.gradle:19` `my-pwa/twa-manifest.json:19`.

## Резервирование и восстановление

Автоматический backup/restore script приложения, политика хранения и проверенный restore-процесс не реализованы в package scripts/compose. Из конфигурации подтверждены только PostgreSQL volume и дисковые uploads/receipts; наличие файлов с именами backup не доказывает работоспособное восстановление. `activation-system/package.json:7` `activation-system/docker-compose.yml:43` `activation-system/middleware/uploadMiddleware.js:6` `activation-system/index.js:184`.
