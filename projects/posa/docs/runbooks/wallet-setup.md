# Настройка Apple Wallet и Google Wallet для «Моих карт»

Для кого: владелец аккаунтов Apple Developer и Google — шаги в кабинетах, которые может сделать только он.
Что делается на сервере после каждого шага — в конце раздела.
Проверить состояние: a.namo.uz → «Карты вендоров» → «Настройка кошельков» (`/admin/own-cards/wallet`).

## Как это устроено

- iPhone получает карту в **Apple Wallet**, Android — в **Google Wallet**. Страница карты сама определяет устройство и показывает одну кнопку.
- Файл Apple на Android не выдаётся: Google Wallet импортирует его копией, которая не обновляется и не получает рассылки.
- Apple: карта подписывается сертификатом Pass Type ID; обновления — через push Apple (APNs).
- Google: карты создаются через Google Wallet API от имени эмитента (Issuer) Namo; о сохранении и удалении карты Google сообщает на `https://wallet.namo.uz/v/google/callback`.

## Apple

Аккаунт Apple Developer Program Namo уже есть (Team ID `5UWY8WBV5Z`).

### 1. Сертификат Namo

Сертификат `pass.com.gorgeouspartners.namowallet` продлён 2026-11-06 и действует до **2027-12-06**. Продлевать — заранее, за месяц: после истечения новые карты без сертификата вендора не выдаются, а выданные перестают обновляться. Срок видно на «Настройке кошельков».

Как продлить, Mac не нужен:

1. Агент создаёт на сервере ключ и запрос (CSR) в `certs/renew-<год>/`. Закрытый ключ сервер не покидает.
2. developer.apple.com → Account → **Certificates, Identifiers & Profiles** → **Certificates** → «+».
3. Тип — **Pass Type ID Certificate** → Continue.
4. Pass Type ID — `pass.com.gorgeouspartners.namowallet` → Continue.
5. Загрузить CSR → Continue → **Download**: получится `pass.cer`. Передать его агенту.

**На сервере (агент):**

1. Проверить, что `pass.cer` сходится с ключом, а Pass Type ID, Team ID и издатель (WWDR) прежние.
2. Перевести сертификат в PEM. Ключ зашифровать паролем `PASS_KEY_PASSWORD`: `routes/wallet.js` требует ключ с паролем.
3. До переключения — пробная подпись карты и проверка APNs с новыми путями.
4. Сохранить копию `.env`, сменить `APPLE_WALLET_SIGNER_CERT_PATH` и `APPLE_WALLET_SIGNER_KEY_PATH`, выполнить `docker compose up -d --no-deps --force-recreate backend`.
5. Убедиться, что живая карта подписана новым сертификатом. Удалить незашифрованную копию ключа.

Pass Type ID не меняется — выданные карты продолжают обновляться.

### 2. Официальный значок «Добавить в Apple Wallet»

1. developer.apple.com/wallet/add-to-apple-wallet-guidelines → **Download badge files** (нужен вход).
2. Из архива взять русский значок в SVG (`…RU…svg`, тёмный вариант для светлого фона) и передать агенту.

**На сервере:** файл кладётся в `public/images/own-voucher/ru_add_to_apple_wallet.svg` — страница карты начинает показывать его вместо текущей кнопки.

### 3. Сертификат вендора — по желанию

Нужен, только если карты должны выпускаться от имени вендора, а не Namo. Вендору нужен свой аккаунт Apple Developer Program (организация, 99 $ в год, D-U-N-S номер).

1. Identifiers → «+» → **Pass Type IDs** → описание и идентификатор, например `pass.uz.vendor.cards`.
2. vendor.namo.uz → «Мои карты» → «Настройки» → «Сертификат Apple» → «Создать запрос (CSR)» → скачать CSR. (Или загрузить готовый `.p12` с паролем.)
3. Certificates → «+» → **Pass Type ID Certificate** → выбрать Pass Type ID → загрузить CSR → скачать `pass.cer`.
4. Загрузить `pass.cer` там же в «Сертификат Apple» → «Проверить у Apple».

То же самое админ Namo может сделать за вендора в карточке вендора.

## Google

Google Wallet в Узбекистане запущен в октябре 2026. Карты Google работают только у тестовых аккаунтов, пока Google не одобрит публикацию.

### 1. Аккаунт эмитента (Issuer)

1. pay.google.com/business/console — войти Google-аккаунтом, который будет администратором.
2. Заполнить публичное название компании (то, что увидят клиенты), принять условия.
3. На главной — карточка **Google Wallet API** → **Create a pass** → **Build your first pass** → принять условия Google Wallet API.
4. Записать **Issuer ID** — число вида `3388000000012345678` (на странице Google Wallet API).
5. Заполнить **Business profile** (адрес, сайт, контакты) — без него Google не одобрит публикацию.

### 2. Сервисный аккаунт и ключ

1. console.cloud.google.com — создать проект, например `namo-wallet`.
2. Открыть страницу **Google Wallet API** в Library → **Enable**.
3. IAM & Admin → **Service accounts** → **Create service account** (например `namo-wallet`) → Done; роли не нужны.
4. У созданного аккаунта → **Keys** → **Add key** → **Create new key** → **JSON** → скачается файл ключа.
   Если создание ключей запрещено политикой организации (`iam.disableServiceAccountKeyCreation`) — снять ограничение для проекта.
5. Вернуться в Pay & Wallet Console → **Users** → **Invite a user** → email сервисного аккаунта (`…@namo-wallet.iam.gserviceaccount.com`) → Access level **Developer** → Invite.

### 3. Тестировщики

В Pay & Wallet Console добавить тестовые Google-аккаунты (свой Android-телефон и тех, кто проверяет). Пока публикация не одобрена, сохранять карты могут только они, у карт в названии будет «[TEST ONLY]».

### 4. Передать агенту

- Issuer ID;
- JSON-ключ сервисного аккаунта — файлом, не в чат и не в репозиторий. Положить в `/opt/apps/posa/activation-system/certs/google-wallet-sa.json`.

**На сервере (агент):** в `.env` — `GOOGLE_WALLET_ISSUER_ID`, `GOOGLE_WALLET_SA_KEY_FILE=/app/certs/google-wallet-sa.json`, `GOOGLE_WALLET_MODE=live`; перезапуск `backend`; «Проверить подключение к Google» на «Настройке кошельков»; сохранить тестовую карту на Android и проверить, что пришло уведомление Google о сохранении, штамп обновил карту, рассылка дошла.

### 5. Публикация для всех клиентов

После проверки на тестовых аккаунтах: Pay & Wallet Console → Google Wallet API → **Request publishing access**. Нужны хотя бы один класс карт (создаётся при первой выдаче) и заполненный business profile. Google проверяет заявку и сообщает результат; до одобрения у остальных пользователей Android сохранение не сработает.

## SMS

Кабинет Eskiz: проверить, нужно ли согласовать шаблон SMS со ссылкой на карту:

> `{вендор}: ваша карта лояльности «{название}». Добавьте в Apple Wallet или Google Wallet: {ссылка}`

Вместо «ваша карта лояльности» бывает «ваш ваучер», «ваша клубная карта», «ваша карта».
