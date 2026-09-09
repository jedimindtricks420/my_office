# VIEWS

Срез локального исходного кода: 2026-09-08. Корень ссылок — `/home/admin1/posa`; локальные незакоммиченные изменения включены. Источники — исполняемый код, схемы и конфигурация; существующая документация и комментарии не используются как доказательства. Статус фактов ниже: **подтверждено кодом**, если явно не указано «предположение на основе имени и использования». Статический анализ не подтверждает состояние рабочей БД или успешный запуск.

## Общие правила

Default layout — `layouts/main`; store-контроллер явно выбирает `store/layout`; kassa-контроллер передаёт `layout: false`. `activation-system/index.js:39` `activation-system/controllers/storeController.js:61` `activation-system/controllers/kassa/kassaCabinetController.js:30`.

Переменные ниже — свободные имена в исполняемых EJS-блоках, после исключения локальных переменных, аргументов callback и JS built-ins. `include`, `locals`, `body` относятся к рендерингу. Упоминание через `typeof` может быть опциональным; полный набор не означает, что все значения обязательны на каждой ветке. Глобальные locals: user, appRole, isClient, appBaseUrl; клиентский роутер дополнительно задаёт isIPhone. `activation-system/index.js:142` `activation-system/routes/clientRoutes.js:9`.

В колонке источников рендера учитываются и недостижимые контроллеры; наличие вызова ещё не доказывает доступность HTTP-страницы. Полный статус разрывов — в GAPS.

## views/layouts/admin.ejs

Переменные: `appRole` `activation-system/views/layouts/admin.ejs:12`, `body` `activation-system/views/layouts/admin.ejs:94`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /logout | нет именованных полей | обычная форма | `activation-system/views/layouts/admin.ejs:60` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/layouts/admin.ejs:9` |


## views/layouts/main.ejs

Переменные: `isClient` `activation-system/views/layouts/main.ejs:10`, `appRole` `activation-system/views/layouts/main.ejs:19`, `body` `activation-system/views/layouts/main.ejs:26`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/layouts/main.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="capacitor://localhost/capacitor.js"> | `activation-system/views/layouts/main.ejs:16` |
| <script type="module" src="/js/admin-session.js"> | `activation-system/views/layouts/main.ejs:29` |


## views/pages/admin-add-merchant.ejs

Переменные: `include` `activation-system/views/pages/admin-add-merchant.ejs:12`, `user` `activation-system/views/pages/admin-add-merchant.ejs:12`, `error` `activation-system/views/pages/admin-add-merchant.ejs:28`, `formData` `activation-system/views/pages/admin-add-merchant.ejs:50`.

Прямые render-вызовы: `activation-system/controllers/admin/merchantController.js:124`; `activation-system/controllers/admin/merchantController.js:64`; `activation-system/controllers/admin/merchantController.js:88`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-add-merchant.ejs:12` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/add-merchant | "username", "password", "status", "legalInfo" | обычная форма | `activation-system/views/pages/admin-add-merchant.ejs:44` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-add-merchant.ejs:7` |


## views/pages/admin-add-product.ejs

Переменные: `include` `activation-system/views/pages/admin-add-product.ejs:11`, `user` `activation-system/views/pages/admin-add-product.ejs:11`, `error` `activation-system/views/pages/admin-add-product.ejs:25`, `vendors` `activation-system/views/pages/admin-add-product.ejs:83`.

Прямые render-вызовы: `activation-system/controllers/admin/productController.js:49`; `activation-system/controllers/admin/productController.js:73`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-add-product.ejs:11` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/add-product | "name", "price", "status", "vendorId", "merchantCommissionPercent", "vendorCommissionPercent" | обычная форма | `activation-system/views/pages/admin-add-product.ejs:43` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-add-product.ejs:7` |


## views/pages/admin-add-store.ejs

Переменные: `include` `activation-system/views/pages/admin-add-store.ejs:13`, `user` `activation-system/views/pages/admin-add-store.ejs:13`, `error` `activation-system/views/pages/admin-add-store.ejs:21`, `products` `activation-system/views/pages/admin-add-store.ejs:66`.

Прямые render-вызовы: `activation-system/controllers/admin/storeController.js:27`; `activation-system/controllers/admin/storeController.js:61`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-add-store.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/stores/add | "slug", "name", "productIds", "isActive" | обычная форма | `activation-system/views/pages/admin-add-store.ejs:40` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-add-store.ejs:8` |


## views/pages/admin-add-user.ejs

Переменные: `include` `activation-system/views/pages/admin-add-user.ejs:13`, `user` `activation-system/views/pages/admin-add-user.ejs:13`, `error` `activation-system/views/pages/admin-add-user.ejs:27`, `kassas` `activation-system/views/pages/admin-add-user.ejs:94`.

Прямые render-вызовы: `activation-system/controllers/admin/userController.js:15`; `activation-system/controllers/admin/userController.js:19`; `activation-system/controllers/admin/userController.js:35`; `activation-system/controllers/admin/userController.js:7`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-add-user.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/add-user | "username", "password", "role", "kassaId" | обычная форма | `activation-system/views/pages/admin-add-user.ejs:47` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-add-user.ejs:8` |


## views/pages/admin-add-vendor-user.ejs

Переменные: `include` `activation-system/views/pages/admin-add-vendor-user.ejs:11`, `user` `activation-system/views/pages/admin-add-vendor-user.ejs:11`, `vendors` `activation-system/views/pages/admin-add-vendor-user.ejs:32`.

Прямые render-вызовы: `activation-system/controllers/admin/vendorController.js:188`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-add-vendor-user.ejs:11` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/add-vendor-users | "vendorId", "username", "password", "note" | обычная форма | `activation-system/views/pages/admin-add-vendor-user.ejs:25` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-add-vendor-user.ejs:7` |


## views/pages/admin-add-vendor.ejs

Переменные: `include` `activation-system/views/pages/admin-add-vendor.ejs:13`, `user` `activation-system/views/pages/admin-add-vendor.ejs:13`, `error` `activation-system/views/pages/admin-add-vendor.ejs:27`.

Прямые render-вызовы: `activation-system/controllers/admin/vendorController.js:66`; `activation-system/controllers/admin/vendorController.js:86`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-add-vendor.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/add-vendor | "name", "category", "productType", "description", "defaultCommissionPercent" | обычная форма | `activation-system/views/pages/admin-add-vendor.ejs:47` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-add-vendor.ejs:8` |


## views/pages/admin-add-voucher.ejs

Переменные: `include` `activation-system/views/pages/admin-add-voucher.ejs:13`, `user` `activation-system/views/pages/admin-add-voucher.ejs:13`, `products` `activation-system/views/pages/admin-add-voucher.ejs:53`.

Прямые render-вызовы: `activation-system/controllers/admin/voucherController.js:12`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-add-voucher.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/add-voucher | "value", "productId", "type" | обычная форма | `activation-system/views/pages/admin-add-voucher.ejs:37` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-add-voucher.ejs:8` |


## views/pages/admin-add-vouchers.ejs

Переменные: `include` `activation-system/views/pages/admin-add-vouchers.ejs:13`, `user` `activation-system/views/pages/admin-add-vouchers.ejs:13`, `products` `activation-system/views/pages/admin-add-vouchers.ejs:44`.

Прямые render-вызовы: `activation-system/controllers/admin/voucherController.js:36`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-add-vouchers.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/vouchers/generate | "productId", "type", "series", "count" | обычная форма | `activation-system/views/pages/admin-add-vouchers.ejs:36` |
| POST | /admin/vouchers/add | "productId", "type", "voucherList", "voucherFile" | multipart/form-data | `activation-system/views/pages/admin-add-vouchers.ejs:103` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-add-vouchers.ejs:8` |


## views/pages/admin-change-password.ejs

Переменные: `userToEdit` `activation-system/views/pages/admin-change-password.ejs:5`, `include` `activation-system/views/pages/admin-change-password.ejs:11`, `user` `activation-system/views/pages/admin-change-password.ejs:11`, `error` `activation-system/views/pages/admin-change-password.ejs:34`.

Прямые render-вызовы: `activation-system/controllers/admin/passwordController.js:9`; `activation-system/controllers/admin/userController.js:59`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-change-password.ejs:11` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/users/<%= userToEdit.id %>/password | "password" | обычная форма | `activation-system/views/pages/admin-change-password.ejs:52` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-change-password.ejs:7` |


## views/pages/admin-client-details-simple.ejs

Переменные: `client` `activation-system/views/pages/admin-client-details-simple.ejs:5`, `include` `activation-system/views/pages/admin-client-details-simple.ejs:11`, `user` `activation-system/views/pages/admin-client-details-simple.ejs:12`.

Прямые render-вызовы: `activation-system/controllers/admin/clientController.js:92`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-client-details-simple.ejs:12` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/clients/<%= client.id %>/delete | нет именованных полей | обычная форма | `activation-system/views/pages/admin-client-details-simple.ejs:70` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-client-details-simple.ejs:7` |


## views/pages/admin-client-details.ejs

Переменные: `client` `activation-system/views/pages/admin-client-details.ejs:5`, `include` `activation-system/views/pages/admin-client-details.ejs:11`, `user` `activation-system/views/pages/admin-client-details.ejs:11`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-client-details.ejs:11` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/clients/<%= client.id %>/delete | нет именованных полей | обычная форма | `activation-system/views/pages/admin-client-details.ejs:186` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-client-details.ejs:7` |


## views/pages/admin-clients.ejs

Переменные: `include` `activation-system/views/pages/admin-clients.ejs:11`, `user` `activation-system/views/pages/admin-clients.ejs:11`, `totalClients` `activation-system/views/pages/admin-clients.ejs:21`, `search` `activation-system/views/pages/admin-clients.ejs:33`, `query` `activation-system/views/pages/admin-clients.ejs:56`, `clients` `activation-system/views/pages/admin-clients.ejs:105`, `totalPages` `activation-system/views/pages/admin-clients.ejs:146`, `hasPrevPage` `activation-system/views/pages/admin-clients.ejs:149`, `prevPage` `activation-system/views/pages/admin-clients.ejs:150`, `hasNextPage` `activation-system/views/pages/admin-clients.ejs:155`, `nextPage` `activation-system/views/pages/admin-clients.ejs:156`, `currentPage` `activation-system/views/pages/admin-clients.ejs:166`, `limit` `activation-system/views/pages/admin-clients.ejs:166`.

Прямые render-вызовы: `activation-system/controllers/admin/clientController.js:53`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-clients.ejs:11` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | /admin/clients | "search" | обычная форма | `activation-system/views/pages/admin-clients.ejs:28` |
| POST | /admin/clients/<%= client.id %>/delete | нет именованных полей | обычная форма | `activation-system/views/pages/admin-clients.ejs:130` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-clients.ejs:7` |


## views/pages/admin-dashboard-content.ejs

Переменные: `user` `activation-system/views/pages/admin-dashboard-content.ejs:1`, `include` `activation-system/views/pages/admin-dashboard-content.ejs:15`, `stats` `activation-system/views/pages/admin-dashboard-content.ejs:25`.

Прямые render-вызовы: `activation-system/controllers/admin/dashboardController.js:106`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-dashboard-content.ejs:15` |


HTML-форм нет. `activation-system/views/pages/admin-dashboard-content.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-dashboard-content.ejs:11` |


## views/pages/admin-dashboard-finance.ejs

Переменные: `user` `activation-system/views/pages/admin-dashboard-finance.ejs:1`, `include` `activation-system/views/pages/admin-dashboard-finance.ejs:14`, `stats` `activation-system/views/pages/admin-dashboard-finance.ejs:22`.

Прямые render-вызовы: `activation-system/controllers/admin/dashboardController.js:75`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-dashboard-finance.ejs:14` |


HTML-форм нет. `activation-system/views/pages/admin-dashboard-finance.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-dashboard-finance.ejs:10` |


## views/pages/admin-dashboard-support.ejs

Переменные: `user` `activation-system/views/pages/admin-dashboard-support.ejs:1`, `include` `activation-system/views/pages/admin-dashboard-support.ejs:15`, `stats` `activation-system/views/pages/admin-dashboard-support.ejs:29`.

Прямые render-вызовы: `activation-system/controllers/admin/dashboardController.js:132`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-dashboard-support.ejs:15` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | /admin/clients | "search" | обычная форма | `activation-system/views/pages/admin-dashboard-support.ejs:81` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-dashboard-support.ejs:11` |


## views/pages/admin-drweb-activations.ejs

Переменные: `include` `activation-system/views/pages/admin-drweb-activations.ejs:13`, `user` `activation-system/views/pages/admin-drweb-activations.ejs:13`, `filters` `activation-system/views/pages/admin-drweb-activations.ejs:43`, `products` `activation-system/views/pages/admin-drweb-activations.ejs:56`, `activations` `activation-system/views/pages/admin-drweb-activations.ejs:100`.

Прямые render-вызовы: `activation-system/controllers/admin/drwebController.js:258`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-drweb-activations.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | /admin/drweb/activations | "startDate", "endDate", "productId", "phone" | обычная форма | `activation-system/views/pages/admin-drweb-activations.ejs:38` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-drweb-activations.ejs:8` |


## views/pages/admin-drweb-add-sku.ejs

Переменные: `include` `activation-system/views/pages/admin-drweb-add-sku.ejs:13`, `user` `activation-system/views/pages/admin-drweb-add-sku.ejs:13`, `error` `activation-system/views/pages/admin-drweb-add-sku.ejs:21`.

Прямые render-вызовы: `activation-system/controllers/admin/drwebController.js:68`; `activation-system/controllers/admin/drwebController.js:94`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-drweb-add-sku.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/drweb/skus/add | "sku", "name", "description", "category", "costPrice", "retailPrice" | обычная форма | `activation-system/views/pages/admin-drweb-add-sku.ejs:40` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-drweb-add-sku.ejs:8` |


## views/pages/admin-drweb-dashboard.ejs

Переменные: `include` `activation-system/views/pages/admin-drweb-dashboard.ejs:13`, `user` `activation-system/views/pages/admin-drweb-dashboard.ejs:13`, `stats` `activation-system/views/pages/admin-drweb-dashboard.ejs:37`, `apiLogs` `activation-system/views/pages/admin-drweb-dashboard.ejs:197`.

Прямые render-вызовы: `activation-system/controllers/admin/drwebController.js:41`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-drweb-dashboard.ejs:13` |


HTML-форм нет. `activation-system/views/pages/admin-drweb-dashboard.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-drweb-dashboard.ejs:8` |


## views/pages/admin-drweb-edit-sku.ejs

Переменные: `include` `activation-system/views/pages/admin-drweb-edit-sku.ejs:13`, `user` `activation-system/views/pages/admin-drweb-edit-sku.ejs:13`, `sku` `activation-system/views/pages/admin-drweb-edit-sku.ejs:18`.

Прямые render-вызовы: `activation-system/controllers/admin/drwebController.js:113`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-drweb-edit-sku.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/drweb/skus/edit/<%= sku.id %> | "sku", "name", "description", "category", "costPrice", "retailPrice", "isActive" | обычная форма | `activation-system/views/pages/admin-drweb-edit-sku.ejs:22` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-drweb-edit-sku.ejs:8` |


## views/pages/admin-drweb-finance.ejs

Переменные: `include` `activation-system/views/pages/admin-drweb-finance.ejs:13`, `user` `activation-system/views/pages/admin-drweb-finance.ejs:13`, `filters` `activation-system/views/pages/admin-drweb-finance.ejs:43`, `stats` `activation-system/views/pages/admin-drweb-finance.ejs:76`, `sales` `activation-system/views/pages/admin-drweb-finance.ejs:170`.

Прямые render-вызовы: `activation-system/controllers/admin/drwebController.js:302`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-drweb-finance.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | /admin/drweb/finance | "startDate", "endDate" | обычная форма | `activation-system/views/pages/admin-drweb-finance.ejs:38` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-drweb-finance.ejs:8` |


## views/pages/admin-drweb-products.ejs

Переменные: `include` `activation-system/views/pages/admin-drweb-products.ejs:13`, `user` `activation-system/views/pages/admin-drweb-products.ejs:13`, `products` `activation-system/views/pages/admin-drweb-products.ejs:57`, `skus` `activation-system/views/pages/admin-drweb-products.ejs:96`.

Прямые render-вызовы: `activation-system/controllers/admin/drwebController.js:176`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-drweb-products.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/drweb/products/<%= product.id %>/bind | "drwebSku" | обычная форма | `activation-system/views/pages/admin-drweb-products.ejs:91` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-drweb-products.ejs:8` |


## views/pages/admin-drweb-skus.ejs

Переменные: `include` `activation-system/views/pages/admin-drweb-skus.ejs:13`, `user` `activation-system/views/pages/admin-drweb-skus.ejs:13`, `skus` `activation-system/views/pages/admin-drweb-skus.ejs:67`.

Прямые render-вызовы: `activation-system/controllers/admin/drwebController.js:60`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-drweb-skus.ejs:13` |


HTML-форм нет. `activation-system/views/pages/admin-drweb-skus.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-drweb-skus.ejs:8` |


## views/pages/admin-edit-merchant.ejs

Переменные: `include` `activation-system/views/pages/admin-edit-merchant.ejs:12`, `user` `activation-system/views/pages/admin-edit-merchant.ejs:12`, `merchant` `activation-system/views/pages/admin-edit-merchant.ejs:18`, `error` `activation-system/views/pages/admin-edit-merchant.ejs:28`.

Прямые render-вызовы: `activation-system/controllers/admin/merchantController.js:145`; `activation-system/controllers/admin/merchantController.js:174`; `activation-system/controllers/admin/merchantController.js:216`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-edit-merchant.ejs:12` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/merchants/edit/<%= merchant.id %> | "username", "status", "legalInfo", "password" | обычная форма | `activation-system/views/pages/admin-edit-merchant.ejs:69` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-edit-merchant.ejs:7` |


## views/pages/admin-edit-store.ejs

Переменные: `include` `activation-system/views/pages/admin-edit-store.ejs:13`, `user` `activation-system/views/pages/admin-edit-store.ejs:13`, `store` `activation-system/views/pages/admin-edit-store.ejs:18`, `allProducts` `activation-system/views/pages/admin-edit-store.ejs:271`.

Прямые render-вызовы: `activation-system/controllers/admin/storeController.js:87`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-edit-store.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/stores/edit/<%= store.id %> | "slug", "name", "logo", "removeLogo", "themeColor", "backgroundColor", "productIds", "activationSmsTemplate", "isActive" | multipart/form-data | `activation-system/views/pages/admin-edit-store.ejs:22` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-edit-store.ejs:8` |


## views/pages/admin-edit-vendor.ejs

Переменные: `vendor` `activation-system/views/pages/admin-edit-vendor.ejs:6`, `include` `activation-system/views/pages/admin-edit-vendor.ejs:14`, `templateSchema` `activation-system/views/pages/admin-edit-vendor.ejs:199`.

Прямые render-вызовы: `activation-system/controllers/admin/vendorController.js:111`; `activation-system/controllers/admin/vendorController.js:174`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar') | `activation-system/views/pages/admin-edit-vendor.ejs:14` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/vendors/edit/<%= vendor.id %> | "name", "category", "productType", "description", "redeemInstructions", "defaultCommissionPercent", "templateSchema" | обычная форма | `activation-system/views/pages/admin-edit-vendor.ejs:40` |


AJAX в шаблоне (выражение адреса и начала вызова):

| Вызов | Источник |
| --- | --- |
| fetch(`/admin/vendors/${vendorId}/receipt/preview`, { method: 'POST', headers: { 'Content-Type': 'application/json', }, body: JSON.stringify({ schema: state }) | `activation-system/views/pages/admin-edit-vendor.ejs:506` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-edit-vendor.ejs:9` |


## views/pages/admin-kassa-add.ejs

Переменные: `include` `activation-system/views/pages/admin-kassa-add.ejs:13`, `user` `activation-system/views/pages/admin-kassa-add.ejs:13`, `error` `activation-system/views/pages/admin-kassa-add.ejs:18`, `vendors` `activation-system/views/pages/admin-kassa-add.ejs:162`.

Прямые render-вызовы: `activation-system/controllers/admin/kassaController.js:22`; `activation-system/controllers/admin/kassaController.js:39`; `activation-system/controllers/admin/kassaController.js:82`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-kassa-add.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/kassas/add | "name", "legalName", "inn", "clickMerchantId", "clickServiceId", "clickSecretKey", "clickMerchantUserId", "clickEnv", "paymeMerchantId", "paymeKey", "paymeTestKey", "paymeEnv", "alifTokenProduction", "alifTokenSandbox", "alifSecretKeyProduction", "alifSecretKeySandbox", "alifEnv", "contactPerson", "contactPhone", "contactEmail", "vendorIds" | обычная форма | `activation-system/views/pages/admin-kassa-add.ejs:26` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-kassa-add.ejs:8` |


## views/pages/admin-kassa-edit.ejs

Переменные: `include` `activation-system/views/pages/admin-kassa-edit.ejs:13`, `user` `activation-system/views/pages/admin-kassa-edit.ejs:13`, `kassa` `activation-system/views/pages/admin-kassa-edit.ejs:16`, `error` `activation-system/views/pages/admin-kassa-edit.ejs:19`, `availableVendors` `activation-system/views/pages/admin-kassa-edit.ejs:187`.

Прямые render-вызовы: `activation-system/controllers/admin/kassaController.js:100`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-kassa-edit.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/kassas/<%= kassa.id %>/edit | "name", "legalName", "inn", "isActive", "clickMerchantId", "clickServiceId", "clickSecretKey", "clickMerchantUserId", "clickEnv", "paymeMerchantId", "paymeKey", "paymeTestKey", "paymeEnv", "alifTokenProduction", "alifTokenSandbox", "alifSecretKeyProduction", "alifSecretKeySandbox", "alifEnv", "contactPerson", "contactPhone", "contactEmail", "vendorIds" | обычная форма | `activation-system/views/pages/admin-kassa-edit.ejs:27` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-kassa-edit.ejs:8` |


## views/pages/admin-kassa-list.ejs

Переменные: `include` `activation-system/views/pages/admin-kassa-list.ejs:13`, `user` `activation-system/views/pages/admin-kassa-list.ejs:13`, `kassas` `activation-system/views/pages/admin-kassa-list.ejs:41`.

Прямые render-вызовы: `activation-system/controllers/admin/kassaController.js:13`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-kassa-list.ejs:13` |


HTML-форм нет. `activation-system/views/pages/admin-kassa-list.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-kassa-list.ejs:8` |


## views/pages/admin-manastore-activations.ejs

Переменные: `include` `activation-system/views/pages/admin-manastore-activations.ejs:13`, `user` `activation-system/views/pages/admin-manastore-activations.ejs:13`, `filters` `activation-system/views/pages/admin-manastore-activations.ejs:43`, `products` `activation-system/views/pages/admin-manastore-activations.ejs:56`, `stores` `activation-system/views/pages/admin-manastore-activations.ejs:74`, `activations` `activation-system/views/pages/admin-manastore-activations.ejs:128`.

Прямые render-вызовы: `activation-system/controllers/admin/manastoreController.js:308`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-manastore-activations.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | /admin/manastore/activations | "startDate", "endDate", "productId", "phone", "storeSlug", "status" | обычная форма | `activation-system/views/pages/admin-manastore-activations.ejs:38` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-manastore-activations.ejs:8` |


## views/pages/admin-manastore-add-sku.ejs

Переменные: `include` `activation-system/views/pages/admin-manastore-add-sku.ejs:13`, `user` `activation-system/views/pages/admin-manastore-add-sku.ejs:13`, `error` `activation-system/views/pages/admin-manastore-add-sku.ejs:21`.

Прямые render-вызовы: `activation-system/controllers/admin/manastoreController.js:126`; `activation-system/controllers/admin/manastoreController.js:98`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-manastore-add-sku.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/manastore/skus/add | "variantId", "sku", "name", "description", "category", "costPrice", "retailPrice", "faceValue" | обычная форма | `activation-system/views/pages/admin-manastore-add-sku.ejs:40` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-manastore-add-sku.ejs:8` |


## views/pages/admin-manastore-dashboard.ejs

Переменные: `include` `activation-system/views/pages/admin-manastore-dashboard.ejs:13`, `user` `activation-system/views/pages/admin-manastore-dashboard.ejs:13`, `environment` `activation-system/views/pages/admin-manastore-dashboard.ejs:18`, `stats` `activation-system/views/pages/admin-manastore-dashboard.ejs:48`, `walletBalance` `activation-system/views/pages/admin-manastore-dashboard.ejs:114`, `apiLogs` `activation-system/views/pages/admin-manastore-dashboard.ejs:236`.

Прямые render-вызовы: `activation-system/controllers/admin/manastoreController.js:69`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-manastore-dashboard.ejs:13` |


HTML-форм нет. `activation-system/views/pages/admin-manastore-dashboard.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-manastore-dashboard.ejs:8` |


## views/pages/admin-manastore-edit-sku.ejs

Переменные: `include` `activation-system/views/pages/admin-manastore-edit-sku.ejs:13`, `user` `activation-system/views/pages/admin-manastore-edit-sku.ejs:13`, `sku` `activation-system/views/pages/admin-manastore-edit-sku.ejs:18`.

Прямые render-вызовы: `activation-system/controllers/admin/manastoreController.js:145`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-manastore-edit-sku.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/manastore/skus/edit/<%= sku.id %> | "variantId", "sku", "name", "description", "category", "costPrice", "retailPrice", "faceValue", "isActive" | обычная форма | `activation-system/views/pages/admin-manastore-edit-sku.ejs:22` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-manastore-edit-sku.ejs:8` |


## views/pages/admin-manastore-finance.ejs

Переменные: `include` `activation-system/views/pages/admin-manastore-finance.ejs:13`, `user` `activation-system/views/pages/admin-manastore-finance.ejs:13`, `environment` `activation-system/views/pages/admin-manastore-finance.ejs:22`, `walletBalance` `activation-system/views/pages/admin-manastore-finance.ejs:47`, `filters` `activation-system/views/pages/admin-manastore-finance.ejs:63`, `stats` `activation-system/views/pages/admin-manastore-finance.ejs:96`, `sales` `activation-system/views/pages/admin-manastore-finance.ejs:190`.

Прямые render-вызовы: `activation-system/controllers/admin/manastoreController.js:364`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-manastore-finance.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | /admin/manastore/finance | "startDate", "endDate" | обычная форма | `activation-system/views/pages/admin-manastore-finance.ejs:58` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-manastore-finance.ejs:8` |


## views/pages/admin-manastore-products.ejs

Переменные: `include` `activation-system/views/pages/admin-manastore-products.ejs:13`, `user` `activation-system/views/pages/admin-manastore-products.ejs:13`, `products` `activation-system/views/pages/admin-manastore-products.ejs:57`, `skus` `activation-system/views/pages/admin-manastore-products.ejs:96`.

Прямые render-вызовы: `activation-system/controllers/admin/manastoreController.js:215`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-manastore-products.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/manastore/products/<%= product.id %>/bind | "variantId" | обычная форма | `activation-system/views/pages/admin-manastore-products.ejs:91` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-manastore-products.ejs:8` |


## views/pages/admin-manastore-skus.ejs

Переменные: `include` `activation-system/views/pages/admin-manastore-skus.ejs:13`, `user` `activation-system/views/pages/admin-manastore-skus.ejs:13`, `skus` `activation-system/views/pages/admin-manastore-skus.ejs:80`.

Прямые render-вызовы: `activation-system/controllers/admin/manastoreController.js:90`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-manastore-skus.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/manastore/skus/sync | нет именованных полей | обычная форма | `activation-system/views/pages/admin-manastore-skus.ejs:34` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-manastore-skus.ejs:8` |


## views/pages/admin-manual-activations.ejs

Переменные: `include` `activation-system/views/pages/admin-manual-activations.ejs:12`, `filters` `activation-system/views/pages/admin-manual-activations.ejs:22`, `stores` `activation-system/views/pages/admin-manual-activations.ejs:29`, `requests` `activation-system/views/pages/admin-manual-activations.ejs:41`.

Прямые render-вызовы: `activation-system/controllers/admin/manualActivationController.js:77`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar') | `activation-system/views/pages/admin-manual-activations.ejs:12` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | текущий URL / JS-handler | "status", "storeId" | обычная форма | `activation-system/views/pages/admin-manual-activations.ejs:19` |


AJAX в шаблоне (выражение адреса и начала вызова):

| Вызов | Источник |
| --- | --- |
| fetch(`/admin/manual-activations/${currentRequestId}/complete`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ key }) | `activation-system/views/pages/admin-manual-activations.ejs:180` |
| fetch(`/admin/manual-activations/${requestId}/reject`, { method: 'POST', headers: { 'Content-Type': 'application/json' } }) | `activation-system/views/pages/admin-manual-activations.ejs:202` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-manual-activations.ejs:8` |


## views/pages/admin-merchant-payments.ejs

Переменные: `user` `activation-system/views/pages/admin-merchant-payments.ejs:1`, `merchant` `activation-system/views/pages/admin-merchant-payments.ejs:7`, `include` `activation-system/views/pages/admin-merchant-payments.ejs:15`, `payments` `activation-system/views/pages/admin-merchant-payments.ejs:38`.

Прямые render-вызовы: `activation-system/controllers/admin/merchantController.js:364`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-merchant-payments.ejs:15` |


HTML-форм нет. `activation-system/views/pages/admin-merchant-payments.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-merchant-payments.ejs:10` |


## views/pages/admin-merchant-transactions.ejs

Переменные: `user` `activation-system/views/pages/admin-merchant-transactions.ejs:1`, `merchant` `activation-system/views/pages/admin-merchant-transactions.ejs:7`, `include` `activation-system/views/pages/admin-merchant-transactions.ejs:15`, `transactions` `activation-system/views/pages/admin-merchant-transactions.ejs:58`.

Прямые render-вызовы: `activation-system/controllers/admin/merchantController.js:342`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-merchant-transactions.ejs:15` |


HTML-форм нет. `activation-system/views/pages/admin-merchant-transactions.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-merchant-transactions.ejs:10` |


## views/pages/admin-merchants.ejs

Переменные: `include` `activation-system/views/pages/admin-merchants.ejs:12`, `user` `activation-system/views/pages/admin-merchants.ejs:12`, `stats` `activation-system/views/pages/admin-merchants.ejs:31`, `merchants` `activation-system/views/pages/admin-merchants.ejs:72`.

Прямые render-вызовы: `activation-system/controllers/admin/merchantController.js:56`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-merchants.ejs:12` |


HTML-форм нет. `activation-system/views/pages/admin-merchants.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-merchants.ejs:7` |


## views/pages/admin-pay-merchant.ejs

Переменные: `include` `activation-system/views/pages/admin-pay-merchant.ejs:12`, `user` `activation-system/views/pages/admin-pay-merchant.ejs:12`, `merchant` `activation-system/views/pages/admin-pay-merchant.ejs:18`, `actualDebt` `activation-system/views/pages/admin-pay-merchant.ejs:40`, `error` `activation-system/views/pages/admin-pay-merchant.ejs:58`.

Прямые render-вызовы: `activation-system/controllers/admin/merchantController.js:236`; `activation-system/controllers/admin/merchantController.js:260`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-pay-merchant.ejs:12` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/merchant/<%= merchant.id %>/pay | "amount", "comment" | обычная форма | `activation-system/views/pages/admin-pay-merchant.ejs:74` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-pay-merchant.ejs:7` |


## views/pages/admin-pay-vendor.ejs

Переменные: `include` `activation-system/views/pages/admin-pay-vendor.ejs:12`, `user` `activation-system/views/pages/admin-pay-vendor.ejs:12`, `vendor` `activation-system/views/pages/admin-pay-vendor.ejs:18`, `error` `activation-system/views/pages/admin-pay-vendor.ejs:53`.

Прямые render-вызовы: `activation-system/controllers/admin/vendorController.js:220`; `activation-system/controllers/admin/vendorController.js:238`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-pay-vendor.ejs:12` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/vendor/<%= vendor.id %>/pay | "amount", "comment" | обычная форма | `activation-system/views/pages/admin-pay-vendor.ejs:69` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-pay-vendor.ejs:7` |


## views/pages/admin-product-images-manage.ejs

Переменные: `user` `activation-system/views/pages/admin-product-images-manage.ejs:1`, `product` `activation-system/views/pages/admin-product-images-manage.ejs:7`, `include` `activation-system/views/pages/admin-product-images-manage.ejs:14`, `error` `activation-system/views/pages/admin-product-images-manage.ejs:25`.

Прямые render-вызовы: `activation-system/controllers/admin/productImageController.js:38`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-product-images-manage.ejs:14` |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-product-images-manage.ejs:113` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/product-images/<%= product.id %>/upload | "image" | multipart/form-data | `activation-system/views/pages/admin-product-images-manage.ejs:34` |
| POST | /admin/product-images/image/<%= image.id %>/primary | нет именованных полей | обычная форма | `activation-system/views/pages/admin-product-images-manage.ejs:74` |
| POST | /admin/product-images/image/<%= image.id %>/delete | нет именованных полей | обычная форма | `activation-system/views/pages/admin-product-images-manage.ejs:82` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-product-images-manage.ejs:9` |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-product-images-manage.ejs:108` |


## views/pages/admin-product-images.ejs

Переменные: `user` `activation-system/views/pages/admin-product-images.ejs:1`, `include` `activation-system/views/pages/admin-product-images.ejs:14`, `products` `activation-system/views/pages/admin-product-images.ejs:28`.

Прямые render-вызовы: `activation-system/controllers/admin/productImageController.js:18`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-product-images.ejs:14` |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-product-images.ejs:106` |


HTML-форм нет. `activation-system/views/pages/admin-product-images.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-product-images.ejs:9` |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-product-images.ejs:101` |


## views/pages/admin-products.ejs

Переменные: `user` `activation-system/views/pages/admin-products.ejs:1`, `include` `activation-system/views/pages/admin-products.ejs:14`, `locals` `activation-system/views/pages/admin-products.ejs:37`, `error` `activation-system/views/pages/admin-products.ejs:37`, `products` `activation-system/views/pages/admin-products.ejs:57`.

Прямые render-вызовы: `activation-system/controllers/admin/productController.js:207`; `activation-system/controllers/admin/productController.js:39`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-products.ejs:14` |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-products.ejs:186` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/delete-product/<%= product.id %> | нет именованных полей | обычная форма | `activation-system/views/pages/admin-products.ejs:147` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-products.ejs:9` |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-products.ejs:181` |


## views/pages/admin-qr-links.ejs

Переменные: `user` `activation-system/views/pages/admin-qr-links.ejs:1`, `include` `activation-system/views/pages/admin-qr-links.ejs:14`, `merchants` `activation-system/views/pages/admin-qr-links.ejs:36`, `products` `activation-system/views/pages/admin-qr-links.ejs:50`.

Прямые render-вызовы: `activation-system/controllers/admin/qrLinkController.js:19`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-qr-links.ejs:14` |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-qr-links.ejs:197` |


HTML-форм нет. `activation-system/views/pages/admin-qr-links.ejs:1`

AJAX в шаблоне (выражение адреса и начала вызова):

| Вызов | Источник |
| --- | --- |
| fetch('/admin/qr-links/generate', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ merchantId: merchantSelect.value, productId: productSelect.value }) | `activation-system/views/pages/admin-qr-links.ejs:131` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-qr-links.ejs:9` |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-qr-links.ejs:192` |


## views/pages/admin-rokky-activations.ejs

Переменные: `include` `activation-system/views/pages/admin-rokky-activations.ejs:13`, `user` `activation-system/views/pages/admin-rokky-activations.ejs:13`, `filters` `activation-system/views/pages/admin-rokky-activations.ejs:43`, `products` `activation-system/views/pages/admin-rokky-activations.ejs:56`, `activations` `activation-system/views/pages/admin-rokky-activations.ejs:100`.

Прямые render-вызовы: `activation-system/controllers/admin/rokkyController.js:258`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-rokky-activations.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | /admin/rokky/activations | "startDate", "endDate", "productId", "phone" | обычная форма | `activation-system/views/pages/admin-rokky-activations.ejs:38` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-rokky-activations.ejs:8` |


## views/pages/admin-rokky-add-sku.ejs

Переменные: `include` `activation-system/views/pages/admin-rokky-add-sku.ejs:13`, `user` `activation-system/views/pages/admin-rokky-add-sku.ejs:13`, `error` `activation-system/views/pages/admin-rokky-add-sku.ejs:21`.

Прямые render-вызовы: `activation-system/controllers/admin/rokkyController.js:68`; `activation-system/controllers/admin/rokkyController.js:94`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-rokky-add-sku.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/rokky/skus/add | "sku", "name", "description", "category", "costPrice", "retailPrice" | обычная форма | `activation-system/views/pages/admin-rokky-add-sku.ejs:40` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-rokky-add-sku.ejs:8` |


## views/pages/admin-rokky-dashboard.ejs

Переменные: `include` `activation-system/views/pages/admin-rokky-dashboard.ejs:13`, `user` `activation-system/views/pages/admin-rokky-dashboard.ejs:13`, `stats` `activation-system/views/pages/admin-rokky-dashboard.ejs:37`, `apiLogs` `activation-system/views/pages/admin-rokky-dashboard.ejs:197`.

Прямые render-вызовы: `activation-system/controllers/admin/rokkyController.js:41`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-rokky-dashboard.ejs:13` |


HTML-форм нет. `activation-system/views/pages/admin-rokky-dashboard.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-rokky-dashboard.ejs:8` |


## views/pages/admin-rokky-edit-sku.ejs

Переменные: `include` `activation-system/views/pages/admin-rokky-edit-sku.ejs:13`, `user` `activation-system/views/pages/admin-rokky-edit-sku.ejs:13`, `sku` `activation-system/views/pages/admin-rokky-edit-sku.ejs:18`.

Прямые render-вызовы: `activation-system/controllers/admin/rokkyController.js:113`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-rokky-edit-sku.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/rokky/skus/edit/<%= sku.id %> | "sku", "name", "description", "category", "costPrice", "retailPrice", "isActive" | обычная форма | `activation-system/views/pages/admin-rokky-edit-sku.ejs:22` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-rokky-edit-sku.ejs:8` |


## views/pages/admin-rokky-finance.ejs

Переменные: `include` `activation-system/views/pages/admin-rokky-finance.ejs:13`, `user` `activation-system/views/pages/admin-rokky-finance.ejs:13`, `filters` `activation-system/views/pages/admin-rokky-finance.ejs:43`, `stats` `activation-system/views/pages/admin-rokky-finance.ejs:76`, `sales` `activation-system/views/pages/admin-rokky-finance.ejs:170`.

Прямые render-вызовы: `activation-system/controllers/admin/rokkyController.js:303`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-rokky-finance.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | /admin/rokky/finance | "startDate", "endDate" | обычная форма | `activation-system/views/pages/admin-rokky-finance.ejs:38` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-rokky-finance.ejs:8` |


## views/pages/admin-rokky-products.ejs

Переменные: `include` `activation-system/views/pages/admin-rokky-products.ejs:13`, `user` `activation-system/views/pages/admin-rokky-products.ejs:13`, `products` `activation-system/views/pages/admin-rokky-products.ejs:57`, `skus` `activation-system/views/pages/admin-rokky-products.ejs:96`.

Прямые render-вызовы: `activation-system/controllers/admin/rokkyController.js:176`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-rokky-products.ejs:13` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/rokky/products/<%= product.id %>/bind | "rokkySku" | обычная форма | `activation-system/views/pages/admin-rokky-products.ejs:91` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-rokky-products.ejs:8` |


## views/pages/admin-rokky-skus.ejs

Переменные: `include` `activation-system/views/pages/admin-rokky-skus.ejs:13`, `user` `activation-system/views/pages/admin-rokky-skus.ejs:13`, `skus` `activation-system/views/pages/admin-rokky-skus.ejs:67`.

Прямые render-вызовы: `activation-system/controllers/admin/rokkyController.js:60`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-rokky-skus.ejs:13` |


HTML-форм нет. `activation-system/views/pages/admin-rokky-skus.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-rokky-skus.ejs:8` |


## views/pages/admin-sales.ejs

Переменные: `include` `activation-system/views/pages/admin-sales.ejs:14`, `user` `activation-system/views/pages/admin-sales.ejs:14`, `stats` `activation-system/views/pages/admin-sales.ejs:16`, `sales` `activation-system/views/pages/admin-sales.ejs:85`, `receiptPath` `activation-system/views/pages/admin-sales.ejs:161`.

Прямые render-вызовы: `activation-system/controllers/admin/saleController.js:58`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-sales.ejs:14` |


HTML-форм нет. `activation-system/views/pages/admin-sales.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-sales.ejs:8` |


## views/pages/admin-stores.ejs

Переменные: `include` `activation-system/views/pages/admin-stores.ejs:13`, `user` `activation-system/views/pages/admin-stores.ejs:13`, `stores` `activation-system/views/pages/admin-stores.ejs:56`.

Прямые render-вызовы: `activation-system/controllers/admin/storeController.js:14`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-stores.ejs:13` |


HTML-форм нет. `activation-system/views/pages/admin-stores.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-stores.ejs:8` |


## views/pages/admin-telegram-bot-form.ejs

Переменные: `bot` `activation-system/views/pages/admin-telegram-bot-form.ejs:8`, `include` `activation-system/views/pages/admin-telegram-bot-form.ejs:14`, `error` `activation-system/views/pages/admin-telegram-bot-form.ejs:21`, `stores` `activation-system/views/pages/admin-telegram-bot-form.ejs:65`.

Прямые render-вызовы: `activation-system/controllers/admin/telegramBotController.js:141`; `activation-system/controllers/admin/telegramBotController.js:164`; `activation-system/controllers/admin/telegramBotController.js:35`; `activation-system/controllers/admin/telegramBotController.js:56`; `activation-system/controllers/admin/telegramBotController.js:99`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar') | `activation-system/views/pages/admin-telegram-bot-form.ejs:14` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | <%= bot ? `/admin/telegram-bots/${bot.id}/edit` : '/admin/telegram-bots/create' %> | "name", "token", "authorizedUsers", "storeIds", "isActive" | обычная форма | `activation-system/views/pages/admin-telegram-bot-form.ejs:27` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-telegram-bot-form.ejs:10` |


## views/pages/admin-telegram-bots.ejs

Переменные: `include` `activation-system/views/pages/admin-telegram-bots.ejs:12`, `bots` `activation-system/views/pages/admin-telegram-bots.ejs:23`.

Прямые render-вызовы: `activation-system/controllers/admin/telegramBotController.js:18`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar') | `activation-system/views/pages/admin-telegram-bots.ejs:12` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/telegram-bots/<%= bot.id %>/delete | нет именованных полей | обычная форма | `activation-system/views/pages/admin-telegram-bots.ejs:89` |


AJAX в шаблоне (выражение адреса и начала вызова):

| Вызов | Источник |
| --- | --- |
| fetch(`/admin/telegram-bots/${botId}/test`, { method: 'POST', headers: { 'Content-Type': 'application/json' } }) | `activation-system/views/pages/admin-telegram-bots.ejs:108` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-telegram-bots.ejs:8` |


## views/pages/admin-users.ejs

Переменные: `include` `activation-system/views/pages/admin-users.ejs:11`, `user` `activation-system/views/pages/admin-users.ejs:11`, `users` `activation-system/views/pages/admin-users.ejs:57`.

Прямые render-вызовы: `activation-system/controllers/admin/userController.js:42`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-users.ejs:11` |


HTML-форм нет. `activation-system/views/pages/admin-users.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-users.ejs:7` |


## views/pages/admin-vendor-transactions.ejs

Переменные: `include` `activation-system/views/pages/admin-vendor-transactions.ejs:12`, `user` `activation-system/views/pages/admin-vendor-transactions.ejs:12`, `vendor` `activation-system/views/pages/admin-vendor-transactions.ejs:18`, `transactions` `activation-system/views/pages/admin-vendor-transactions.ejs:42`.

Прямые render-вызовы: `activation-system/controllers/admin/vendorController.js:316`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-vendor-transactions.ejs:12` |


HTML-форм нет. `activation-system/views/pages/admin-vendor-transactions.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-vendor-transactions.ejs:7` |


## views/pages/admin-vendors.ejs

Переменные: `include` `activation-system/views/pages/admin-vendors.ejs:14`, `user` `activation-system/views/pages/admin-vendors.ejs:14`, `stats` `activation-system/views/pages/admin-vendors.ejs:16`, `vendors` `activation-system/views/pages/admin-vendors.ejs:112`.

Прямые render-вызовы: `activation-system/controllers/admin/vendorController.js:56`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/admin-vendors.ejs:14` |


HTML-форм нет. `activation-system/views/pages/admin-vendors.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-vendors.ejs:8` |


## views/pages/admin-vouchers.ejs

Переменные: `include` `activation-system/views/pages/admin-vouchers.ejs:12`, `vouchers` `activation-system/views/pages/admin-vouchers.ejs:57`.

Прямые render-вызовы: `activation-system/controllers/admin/voucherController.js:6`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar') | `activation-system/views/pages/admin-vouchers.ejs:12` |


HTML-форм нет. `activation-system/views/pages/admin-vouchers.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/admin-vouchers.ejs:8` |


## views/pages/checkout.ejs

Переменные: `include` `activation-system/views/pages/checkout.ejs:5`, `user` `activation-system/views/pages/checkout.ejs:10`, `totalQuantity` `activation-system/views/pages/checkout.ejs:10`, `items` `activation-system/views/pages/checkout.ejs:23`, `totalPrice` `activation-system/views/pages/checkout.ejs:27`, `checkoutToken` `activation-system/views/pages/checkout.ejs:56`.

Прямые render-вызовы: `activation-system/controllers/merchant/cartController.js:78`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/merchant/head', { title: 'Чек продажи' }) | `activation-system/views/pages/checkout.ejs:5` |
| include('../partials/merchant/sidebar', { active: 'checkout', user, cartCount: totalQuantity }) | `activation-system/views/pages/checkout.ejs:10` |
| include('../partials/merchant/topbar', { active: 'checkout', user }) | `activation-system/views/pages/checkout.ejs:13` |
| include('../partials/merchant/bottom-nav', { active: 'checkout', cartCount: totalQuantity }) | `activation-system/views/pages/checkout.ejs:186` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /merchant/checkout/confirm | "checkoutToken", "saleType", "customerPhone", "paymentConfirmed" | обычная форма | `activation-system/views/pages/checkout.ejs:54` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="/js/merchant-checkout.js"> | `activation-system/views/pages/checkout.ejs:188` |


## views/pages/client-dashboard.ejs

Переменные: `include` `activation-system/views/pages/client-dashboard.ejs:3`, `phone` `activation-system/views/pages/client-dashboard.ejs:27`, `vouchers` `activation-system/views/pages/client-dashboard.ejs:28`, `pushPublicKey` `activation-system/views/pages/client-dashboard.ejs:30`.

Прямые render-вызовы: `activation-system/controllers/client/dashboardController.js:77`; `activation-system/controllers/client/dashboardController.js:86`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/client/head', { title: 'Мои ваучеры' }) | `activation-system/views/pages/client-dashboard.ejs:3` |
| include('../partials/client/wallet-skeleton') | `activation-system/views/pages/client-dashboard.ejs:16` |
| include('../partials/client/bottom-nav', { active: 'wallet' }) | `activation-system/views/pages/client-dashboard.ejs:22` |
| include('../partials/client/voucher-modal') | `activation-system/views/pages/client-dashboard.ejs:23` |


HTML-форм нет. `activation-system/views/pages/client-dashboard.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script type="module" src="/js/client-session.js"> | `activation-system/views/pages/client-dashboard.ejs:32` |
| <script type="module" src="/js/client-app.js"> | `activation-system/views/pages/client-dashboard.ejs:33` |


## views/pages/client-login.ejs

Переменные: `include` `activation-system/views/pages/client-login.ejs:3`, `error` `activation-system/views/pages/client-login.ejs:14`.

Прямые render-вызовы: `activation-system/controllers/client/authController.js:127`; `activation-system/controllers/client/authController.js:148`; `activation-system/controllers/client/authController.js:160`; `activation-system/controllers/client/authController.js:166`; `activation-system/controllers/client/authController.js:92`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/client/head', { title: 'Вход в кошелек' }) | `activation-system/views/pages/client-login.ejs:3` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /wallet | "phoneNumber" | обычная форма | `activation-system/views/pages/client-login.ejs:17` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script type="module" src="/js/client-session.js"> | `activation-system/views/pages/client-login.ejs:30` |
| <script type="module" src="/js/client-auth.js"> | `activation-system/views/pages/client-login.ejs:31` |


## views/pages/client-profile.ejs

Переменные: `include` `activation-system/views/pages/client-profile.ejs:3`, `phone` `activation-system/views/pages/client-profile.ejs:13`, `pushPublicKey` `activation-system/views/pages/client-profile.ejs:94`.

Прямые render-вызовы: `activation-system/routes/clientRoutes.js:40`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/client/head', { title: 'Профиль' }) | `activation-system/views/pages/client-profile.ejs:3` |
| include('../partials/client/bottom-nav', { active: 'profile' }) | `activation-system/views/pages/client-profile.ejs:88` |


HTML-форм нет. `activation-system/views/pages/client-profile.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script type="module" src="/js/client-session.js"> | `activation-system/views/pages/client-profile.ejs:96` |
| <script type="module" src="/js/client-profile.js?v=2"> | `activation-system/views/pages/client-profile.ejs:97` |


## views/pages/client-qr-scanner.ejs

Переменные: `include` `activation-system/views/pages/client-qr-scanner.ejs:3`, `phone` `activation-system/views/pages/client-qr-scanner.ejs:49`, `pushPublicKey` `activation-system/views/pages/client-qr-scanner.ejs:51`.

Прямые render-вызовы: `activation-system/routes/clientRoutes.js:46`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/client/head', { title: 'Сканировать ваучер' }) | `activation-system/views/pages/client-qr-scanner.ejs:3` |
| include('../partials/client/bottom-nav', { active: 'scan' }) | `activation-system/views/pages/client-qr-scanner.ejs:44` |
| include('../partials/client/voucher-modal') | `activation-system/views/pages/client-qr-scanner.ejs:45` |


HTML-форм нет. `activation-system/views/pages/client-qr-scanner.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="/vendor/zxing-library/index.min.js" data-zxing-core="1" data-ready="1"> | `activation-system/views/pages/client-qr-scanner.ejs:53` |
| <script src="/vendor/zxing/browser/umd/zxing-browser.min.js" data-zxing-browser="1" data-ready="1"> | `activation-system/views/pages/client-qr-scanner.ejs:54` |
| <script src="/vendor/jsqr/jsQR.js" data-jsqr="1" data-ready="1"> | `activation-system/views/pages/client-qr-scanner.ejs:55` |
| <script type="module" src="/js/client-session.js"> | `activation-system/views/pages/client-qr-scanner.ejs:56` |
| <script type="module" src="/js/client-scan.js"> | `activation-system/views/pages/client-qr-scanner.ejs:57` |


## views/pages/client-register.ejs

Переменные: `include` `activation-system/views/pages/client-register.ejs:3`, `error` `activation-system/views/pages/client-register.ejs:14`.

Прямые render-вызовы: `activation-system/controllers/client/authController.js:253`; `activation-system/controllers/client/authController.js:266`; `activation-system/controllers/client/authController.js:278`; `activation-system/controllers/client/authController.js:289`; `activation-system/controllers/client/authController.js:295`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/client/head', { title: 'Регистрация' }) | `activation-system/views/pages/client-register.ejs:3` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /client-register | "name", "phoneNumber" | обычная форма | `activation-system/views/pages/client-register.ejs:17` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script type="module" src="/js/client-session.js"> | `activation-system/views/pages/client-register.ejs:34` |
| <script type="module" src="/js/client-auth.js"> | `activation-system/views/pages/client-register.ejs:35` |


## views/pages/client-verify.ejs

Переменные: `include` `activation-system/views/pages/client-verify.ejs:3`, `phone` `activation-system/views/pages/client-verify.ejs:11`, `error` `activation-system/views/pages/client-verify.ejs:14`.

Прямые render-вызовы: `activation-system/controllers/client/authController.js:173`; `activation-system/controllers/client/authController.js:188`; `activation-system/controllers/client/authController.js:195`; `activation-system/controllers/client/authController.js:239`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/client/head', { title: 'Подтверждение кода' }) | `activation-system/views/pages/client-verify.ejs:3` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /client-verify | "otp" | обычная форма | `activation-system/views/pages/client-verify.ejs:17` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script type="module" src="/js/client-session.js"> | `activation-system/views/pages/client-verify.ejs:35` |
| <script type="module" src="/js/client-auth.js"> | `activation-system/views/pages/client-verify.ejs:36` |


## views/pages/dashboard-admin.ejs

Переменные: `include` `activation-system/views/pages/dashboard-admin.ejs:26`, `user` `activation-system/views/pages/dashboard-admin.ejs:26`, `stats` `activation-system/views/pages/dashboard-admin.ejs:59`.

Прямые render-вызовы: `activation-system/controllers/admin/dashboardController.js:37`; `activation-system/controllers/adminController.js:22`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/dashboard-admin.ejs:26` |


HTML-форм нет. `activation-system/views/pages/dashboard-admin.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/dashboard-admin.ejs:9` |


## views/pages/edit-product.ejs

Переменные: `product` `activation-system/views/pages/edit-product.ejs:6`, `include` `activation-system/views/pages/edit-product.ejs:14`, `user` `activation-system/views/pages/edit-product.ejs:14`, `vendor` `activation-system/views/pages/edit-product.ejs:23`, `hasCustomInstructions` `activation-system/views/pages/edit-product.ejs:143`, `hasCustomTemplate` `activation-system/views/pages/edit-product.ejs:180`, `templateSchema` `activation-system/views/pages/edit-product.ejs:293`.

Прямые render-вызовы: `activation-system/controllers/admin/productController.js:98`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/admin-navbar', { user }) | `activation-system/views/pages/edit-product.ejs:14` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /admin/edit-product/<%= product.id %> | "name", "price", "status", "merchantCommissionPercent", "vendorCommissionPercent", "rokkySku", "drwebSku", "manastoreVariantId", "walletOnDemandEnabled", "useCustomInstructions", "redeemInstructions", "useCustomTemplate", "templateSchema" | обычная форма | `activation-system/views/pages/edit-product.ejs:40` |


AJAX в шаблоне (выражение адреса и начала вызова):

| Вызов | Источник |
| --- | --- |
| fetch(`/admin/products/${productId}/receipt/preview`, { method: 'POST', headers: { 'Content-Type': 'application/json', }, body: JSON.stringify({ schema: state }) | `activation-system/views/pages/edit-product.ejs:678` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/edit-product.ejs:9` |


## views/pages/enter-email.ejs

Переменные: `error` `activation-system/views/pages/enter-email.ejs:22`.

Прямые render-вызовы: `activation-system/controllers/publicController.js:16`; `activation-system/controllers/publicController.js:8`.

Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /client/enter-email | "email", "phone" | обычная форма | `activation-system/views/pages/enter-email.ejs:40` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/enter-email.ejs:7` |


## views/pages/kassa/dashboard.ejs

Переменные: `include` `activation-system/views/pages/kassa/dashboard.ejs:12`, `user` `activation-system/views/pages/kassa/dashboard.ejs:12`, `kassa` `activation-system/views/pages/kassa/dashboard.ejs:12`, `weekSales` `activation-system/views/pages/kassa/dashboard.ejs:41`, `recentTransactions` `activation-system/views/pages/kassa/dashboard.ejs:90`.

Прямые render-вызовы: `activation-system/controllers/kassa/kassaCabinetController.js:30`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../../partials/kassa-navbar', { user, activeNav: 'dashboard' }) | `activation-system/views/pages/kassa/dashboard.ejs:12` |


HTML-форм нет. `activation-system/views/pages/kassa/dashboard.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/kassa/dashboard.ejs:8` |


## views/pages/kassa/obligations.ejs

Переменные: `include` `activation-system/views/pages/kassa/obligations.ejs:12`, `user` `activation-system/views/pages/kassa/obligations.ejs:12`, `vendorStats` `activation-system/views/pages/kassa/obligations.ejs:38`, `adjustments` `activation-system/views/pages/kassa/obligations.ejs:85`.

Прямые render-вызовы: `activation-system/controllers/kassa/kassaCabinetController.js:115`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../../partials/kassa-navbar', { user, activeNav: 'obligations' }) | `activation-system/views/pages/kassa/obligations.ejs:12` |


HTML-форм нет. `activation-system/views/pages/kassa/obligations.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/kassa/obligations.ejs:8` |


## views/pages/kassa/payouts.ejs

Переменные: `include` `activation-system/views/pages/kassa/payouts.ejs:12`, `user` `activation-system/views/pages/kassa/payouts.ejs:12`, `kassaBalance` `activation-system/views/pages/kassa/payouts.ejs:21`, `totalPaid` `activation-system/views/pages/kassa/payouts.ejs:27`, `vendors` `activation-system/views/pages/kassa/payouts.ejs:52`, `payments` `activation-system/views/pages/kassa/payouts.ejs:111`.

Прямые render-вызовы: `activation-system/controllers/kassa/kassaCabinetController.js:152`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../../partials/kassa-navbar', { user, activeNav: 'payouts' }) | `activation-system/views/pages/kassa/payouts.ejs:12` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /kassa/payouts | "recipientType", "recipientId", "amount", "comment" | обычная форма | `activation-system/views/pages/kassa/payouts.ejs:37` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/kassa/payouts.ejs:8` |


## views/pages/kassa/settings.ejs

Переменные: `include` `activation-system/views/pages/kassa/settings.ejs:12`, `user` `activation-system/views/pages/kassa/settings.ejs:12`, `kassa` `activation-system/views/pages/kassa/settings.ejs:12`, `callbackUrls` `activation-system/views/pages/kassa/settings.ejs:110`.

Прямые render-вызовы: `activation-system/controllers/kassa/kassaCabinetController.js:277`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../../partials/kassa-navbar', { user, activeNav: 'settings' }) | `activation-system/views/pages/kassa/settings.ejs:12` |


HTML-форм нет. `activation-system/views/pages/kassa/settings.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/kassa/settings.ejs:8` |


## views/pages/kassa/transactions.ejs

Переменные: `include` `activation-system/views/pages/kassa/transactions.ejs:12`, `user` `activation-system/views/pages/kassa/transactions.ejs:12`, `vendors` `activation-system/views/pages/kassa/transactions.ejs:24`, `filters` `activation-system/views/pages/kassa/transactions.ejs:25`, `transactions` `activation-system/views/pages/kassa/transactions.ejs:12`, `totalPages` `activation-system/views/pages/kassa/transactions.ejs:104`, `page` `activation-system/views/pages/kassa/transactions.ejs:108`, `total` `activation-system/views/pages/kassa/transactions.ejs:115`.

Прямые render-вызовы: `activation-system/controllers/kassa/kassaCabinetController.js:71`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../../partials/kassa-navbar', { user, activeNav: 'transactions' }) | `activation-system/views/pages/kassa/transactions.ejs:12` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | текущий URL / JS-handler | "vendor", "status" | обычная форма | `activation-system/views/pages/kassa/transactions.ejs:18` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/kassa/transactions.ejs:8` |


## views/pages/login.ejs

Переменные: `error` `activation-system/views/pages/login.ejs:19`.

Прямые render-вызовы: `activation-system/controllers/authController.js:17`.

Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /auth/login | "username", "password", "rememberMe" | обычная форма | `activation-system/views/pages/login.ejs:37` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/login.ejs:7` |
| <script type="module" src="/js/admin-session.js"> | `activation-system/views/pages/login.ejs:77` |


## views/pages/merchant-login.ejs

Переменные: `include` `activation-system/views/pages/merchant-login.ejs:5`, `error` `activation-system/views/pages/merchant-login.ejs:61`.

Прямые render-вызовы: `activation-system/routes/merchantRoutes.js:13`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/merchant/head', { title: 'Вход для мерчанта' }) | `activation-system/views/pages/merchant-login.ejs:5` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /auth/login | "loginVariant", "username", "password", "rememberMe" | обычная форма | `activation-system/views/pages/merchant-login.ejs:78` |


## views/pages/merchant-sales.ejs

Переменные: `include` `activation-system/views/pages/merchant-sales.ejs:5`, `sales` `activation-system/views/pages/merchant-sales.ejs:9`, `merchantDebt` `activation-system/views/pages/merchant-sales.ejs:13`, `user` `activation-system/views/pages/merchant-sales.ejs:16`.

Прямые render-вызовы: `activation-system/controllers/merchant/dashboardController.js:37`; `activation-system/controllers/merchant/dashboardController.js:63`; `activation-system/controllers/merchant/saleController.js:730`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/merchant/head', { title: 'История продаж' }) | `activation-system/views/pages/merchant-sales.ejs:5` |
| include('../partials/merchant/sidebar', { active: 'sales', user }) | `activation-system/views/pages/merchant-sales.ejs:16` |
| include('../partials/merchant/topbar', { active: 'sales', user }) | `activation-system/views/pages/merchant-sales.ejs:19` |
| include('../partials/merchant/bottom-nav', { active: 'sales' }) | `activation-system/views/pages/merchant-sales.ejs:160` |


HTML-форм нет. `activation-system/views/pages/merchant-sales.ejs:1`

## views/pages/merchant-sell.ejs

Переменные: `include` `activation-system/views/pages/merchant-sell.ejs:5`, `cart` `activation-system/views/pages/merchant-sell.ejs:9`, `user` `activation-system/views/pages/merchant-sell.ejs:12`, `error` `activation-system/views/pages/merchant-sell.ejs:40`, `products` `activation-system/views/pages/merchant-sell.ejs:65`.

Прямые render-вызовы: `activation-system/controllers/merchant/cartController.js:35`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../partials/merchant/head', { title: 'Продажа товаров' }) | `activation-system/views/pages/merchant-sell.ejs:5` |
| include('../partials/merchant/sidebar', { active: 'sell', user, cartCount }) | `activation-system/views/pages/merchant-sell.ejs:12` |
| include('../partials/merchant/topbar', { active: 'sell', user }) | `activation-system/views/pages/merchant-sell.ejs:15` |
| include('../partials/merchant/bottom-nav', { active: 'sell', cartCount }) | `activation-system/views/pages/merchant-sell.ejs:138` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /merchant/add-to-cart | "productId" | обычная форма | `activation-system/views/pages/merchant-sell.ejs:114` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="/js/merchant-sell.js"> | `activation-system/views/pages/merchant-sell.ejs:140` |


## views/pages/pay-checkout.ejs

Переменные: `product` `activation-system/views/pages/pay-checkout.ejs:9`, `token` `activation-system/views/pages/pay-checkout.ejs:22`.

Прямые render-вызовы: `activation-system/controllers/public/qrPaymentController.js:98`.

Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET по умолчанию; возможен JS submit | текущий URL / JS-handler | "paymentMethod" | обычная форма | `activation-system/views/pages/pay-checkout.ejs:43` |


AJAX в шаблоне (выражение адреса и начала вызова):

| Вызов | Источник |
| --- | --- |
| fetch('/pay/<%= token %>/checkout', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ phoneNumber: '+998' + phone, paymentMethod: document.querySelector('input[name="paymentMethod"]:checked') | `activation-system/views/pages/pay-checkout.ejs:160` |


## views/pages/pay-error.ejs

Переменные: `error` `activation-system/views/pages/pay-error.ejs:22`, `message` `activation-system/views/pages/pay-error.ejs:67`, `token` `activation-system/views/pages/pay-error.ejs:74`.

Прямые render-вызовы: `activation-system/controllers/public/qrPaymentController.js:34`; `activation-system/controllers/public/qrPaymentController.js:41`; `activation-system/controllers/public/qrPaymentController.js:48`; `activation-system/controllers/public/qrPaymentController.js:496`; `activation-system/controllers/public/qrPaymentController.js:510`; `activation-system/controllers/public/qrPaymentController.js:527`; `activation-system/controllers/public/qrPaymentController.js:74`.

HTML-форм нет. `activation-system/views/pages/pay-error.ejs:1`

## views/pages/pay-product.ejs

Переменные: `product` `activation-system/views/pages/pay-product.ejs:9`, `oldPrice` `activation-system/views/pages/pay-product.ejs:48`, `discountPercent` `activation-system/views/pages/pay-product.ejs:51`, `hasVouchers` `activation-system/views/pages/pay-product.ejs:58`, `link` `activation-system/views/pages/pay-product.ejs:59`.

Прямые render-вызовы: `activation-system/controllers/public/qrPaymentController.js:64`.

HTML-форм нет. `activation-system/views/pages/pay-product.ejs:1`

## views/pages/pay-result.ejs

Переменные: `voucherCode` `activation-system/views/pages/pay-result.ejs:37`, `merchant` `activation-system/views/pages/pay-result.ejs:49`, `product` `activation-system/views/pages/pay-result.ejs:61`, `attempt` `activation-system/views/pages/pay-result.ejs:86`, `sale` `activation-system/views/pages/pay-result.ejs:94`, `token` `activation-system/views/pages/pay-result.ejs:105`.

Прямые render-вызовы: `activation-system/controllers/public/qrPaymentController.js:517`.

HTML-форм нет. `activation-system/views/pages/pay-result.ejs:1`

## views/pages/sale-receipt.ejs

Переменные: `user` `activation-system/views/pages/sale-receipt.ejs:4`, `product` `activation-system/views/pages/sale-receipt.ejs:5`, `price` `activation-system/views/pages/sale-receipt.ejs:6`, `voucher` `activation-system/views/pages/sale-receipt.ejs:7`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/pages/sale-receipt.ejs:1`

## views/pages/vendor/activate-qr-result.ejs

Переменные: `result` `activation-system/views/pages/vendor/activate-qr-result.ejs:13`.

Прямые render-вызовы: `activation-system/controllers/publicController.js:136`; `activation-system/controllers/publicController.js:148`; `activation-system/controllers/publicController.js:157`.

HTML-форм нет. `activation-system/views/pages/vendor/activate-qr-result.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/vendor/activate-qr-result.ejs:7` |


## views/pages/vendor/activate.ejs

Переменные: `include` `activation-system/views/pages/vendor/activate.ejs:14`, `user` `activation-system/views/pages/vendor/activate.ejs:14`, `error` `activation-system/views/pages/vendor/activate.ejs:30`, `success` `activation-system/views/pages/vendor/activate.ejs:44`, `form` `activation-system/views/pages/vendor/activate.ejs:61`.

Прямые render-вызовы: `activation-system/controllers/vendor/activationController.js:16`; `activation-system/controllers/vendor/activationController.js:30`; `activation-system/controllers/vendor/activationController.js:50`; `activation-system/controllers/vendor/activationController.js:56`; `activation-system/controllers/vendor/activationController.js:63`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../../partials/vendor-navbar', { user, active: 'activate' }) | `activation-system/views/pages/vendor/activate.ejs:14` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /vendor/activate | "code" | обычная форма | `activation-system/views/pages/vendor/activate.ejs:58` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/vendor/activate.ejs:9` |


## views/pages/vendor/dashboard.ejs

Переменные: `include` `activation-system/views/pages/vendor/dashboard.ejs:14`, `user` `activation-system/views/pages/vendor/dashboard.ejs:14`, `error` `activation-system/views/pages/vendor/dashboard.ejs:17`, `vendor` `activation-system/views/pages/vendor/dashboard.ejs:14`, `metrics` `activation-system/views/pages/vendor/dashboard.ejs:53`, `trendRange` `activation-system/views/pages/vendor/dashboard.ejs:77`, `activationTrend` `activation-system/views/pages/vendor/dashboard.ejs:79`, `recentActivations` `activation-system/views/pages/vendor/dashboard.ejs:115`, `pendingQueue` `activation-system/views/pages/vendor/dashboard.ejs:155`.

Прямые render-вызовы: `activation-system/controllers/vendor/dashboardController.js:157`; `activation-system/controllers/vendor/dashboardController.js:168`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../../partials/vendor-navbar', { user, active: 'dashboard' }) | `activation-system/views/pages/vendor/dashboard.ejs:14` |


HTML-форм нет. `activation-system/views/pages/vendor/dashboard.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/vendor/dashboard.ejs:9` |


## views/pages/vendor/settings.ejs

Переменные: `include` `activation-system/views/pages/vendor/settings.ejs:14`, `user` `activation-system/views/pages/vendor/settings.ejs:14`, `error` `activation-system/views/pages/vendor/settings.ejs:17`, `vendor` `activation-system/views/pages/vendor/settings.ejs:14`, `users` `activation-system/views/pages/vendor/settings.ejs:82`, `previewHtml` `activation-system/views/pages/vendor/settings.ejs:118`.

Прямые render-вызовы: `activation-system/controllers/vendor/settingsController.js:67`; `activation-system/controllers/vendor/settingsController.js:75`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../../partials/vendor-navbar', { user, active: 'settings' }) | `activation-system/views/pages/vendor/settings.ejs:14` |


HTML-форм нет. `activation-system/views/pages/vendor/settings.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/vendor/settings.ejs:9` |


## views/pages/vendor/transactions.ejs

Переменные: `include` `activation-system/views/pages/vendor/transactions.ejs:14`, `user` `activation-system/views/pages/vendor/transactions.ejs:14`, `error` `activation-system/views/pages/vendor/transactions.ejs:17`, `summary` `activation-system/views/pages/vendor/transactions.ejs:42`, `transactions` `activation-system/views/pages/vendor/transactions.ejs:14`, `payments` `activation-system/views/pages/vendor/transactions.ejs:120`.

Прямые render-вызовы: `activation-system/controllers/vendor/transactionController.js:100`; `activation-system/controllers/vendor/transactionController.js:92`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../../partials/vendor-navbar', { user, active: 'transactions' }) | `activation-system/views/pages/vendor/transactions.ejs:14` |


HTML-форм нет. `activation-system/views/pages/vendor/transactions.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/vendor/transactions.ejs:9` |


## views/pages/vendor/vouchers.ejs

Переменные: `include` `activation-system/views/pages/vendor/vouchers.ejs:14`, `user` `activation-system/views/pages/vendor/vouchers.ejs:14`, `filters` `activation-system/views/pages/vendor/vouchers.ejs:26`, `statusOptions` `activation-system/views/pages/vendor/vouchers.ejs:31`, `products` `activation-system/views/pages/vendor/vouchers.ejs:39`, `error` `activation-system/views/pages/vendor/vouchers.ejs:59`, `vouchers` `activation-system/views/pages/vendor/vouchers.ejs:14`, `pagination` `activation-system/views/pages/vendor/vouchers.ejs:149`.

Прямые render-вызовы: `activation-system/controllers/vendor/voucherController.js:128`; `activation-system/controllers/vendor/voucherController.js:138`.

Include:

| Вызов | Источник |
| --- | --- |
| include('../../partials/vendor-navbar', { user, active: 'vouchers' }) | `activation-system/views/pages/vendor/vouchers.ejs:14` |


Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| GET | /vendor/vouchers | "search", "status", "productId" | обычная форма | `activation-system/views/pages/vendor/vouchers.ejs:25` |


Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/pages/vendor/vouchers.ejs:9` |


## views/partials/admin-navbar.ejs

Переменные: `user` `activation-system/views/partials/admin-navbar.ejs:25`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /auth/logout | нет именованных полей | обычная форма | `activation-system/views/partials/admin-navbar.ejs:278` |
| POST | /auth/logout | нет именованных полей | обычная форма | `activation-system/views/partials/admin-navbar.ejs:466` |


## views/partials/client/bottom-nav.ejs

Переменные: `active` `activation-system/views/partials/client/bottom-nav.ejs:4`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/client/bottom-nav.ejs:1`

## views/partials/client/head.ejs

Переменные: `appBaseUrl` `activation-system/views/partials/client/head.ejs:16`, `title` `activation-system/views/partials/client/head.ejs:31`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/client/head.ejs:1`

## views/partials/client/topbar.ejs

Переменные: `phoneLabel` `activation-system/views/partials/client/topbar.ejs:10`, `phone` `activation-system/views/partials/client/topbar.ejs:10`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/client/topbar.ejs:1`

## views/partials/client/voucher-modal.ejs

Переменные: `isClient` `activation-system/views/partials/client/voucher-modal.ejs:76`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/client/voucher-modal.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="/vendor/capacitor/capacitor.js"> | `activation-system/views/partials/client/voucher-modal.ejs:77` |
| <script src="/js/voucher-modal.js" defer> | `activation-system/views/partials/client/voucher-modal.ejs:79` |
| <script type="module" src="/js/voucher-ondemand.js" defer> | `activation-system/views/partials/client/voucher-modal.ejs:80` |


## views/partials/client/wallet-card.ejs

Переменные: `voucher` `activation-system/views/partials/client/wallet-card.ejs:2`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/client/wallet-card.ejs:1`

## views/partials/client/wallet-skeleton.ejs

Переменные: нет свободных переменных `activation-system/views/partials/client/wallet-skeleton.ejs:1`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/client/wallet-skeleton.ejs:1`

## views/partials/kassa-navbar.ejs

Переменные: `activeNav` `activation-system/views/partials/kassa-navbar.ejs:15`, `user` `activation-system/views/partials/kassa-navbar.ejs:43`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/kassa-navbar.ejs:1`

## views/partials/merchant/bottom-nav.ejs

Переменные: `active` `activation-system/views/partials/merchant/bottom-nav.ejs:12`, `locals` `activation-system/views/partials/merchant/bottom-nav.ejs:15`, `cartCount` `activation-system/views/partials/merchant/bottom-nav.ejs:15`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/merchant/bottom-nav.ejs:1`

## views/partials/merchant/head.ejs

Переменные: `title` `activation-system/views/partials/merchant/head.ejs:4`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/merchant/head.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.tailwindcss.com"> | `activation-system/views/partials/merchant/head.ejs:11` |


## views/partials/merchant/sidebar.ejs

Переменные: `user` `activation-system/views/partials/merchant/sidebar.ejs:8`, `active` `activation-system/views/partials/merchant/sidebar.ejs:21`, `locals` `activation-system/views/partials/merchant/sidebar.ejs:24`, `cartCount` `activation-system/views/partials/merchant/sidebar.ejs:24`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/merchant/sidebar.ejs:1`

## views/partials/merchant/topbar.ejs

Переменные: `user` `activation-system/views/partials/merchant/topbar.ejs:3`, `active` `activation-system/views/partials/merchant/topbar.ejs:8`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/merchant/topbar.ejs:1`

## views/partials/merchant-navbar.ejs

Переменные: `user` `activation-system/views/partials/merchant-navbar.ejs:45`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/merchant-navbar.ejs:1`

## views/partials/vendor-navbar.ejs

Переменные: `active` `activation-system/views/partials/vendor-navbar.ejs:9`, `user` `activation-system/views/partials/vendor-navbar.ejs:53`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/partials/vendor-navbar.ejs:1`

## views/store/activate.ejs

Переменные: `history` `activation-system/views/store/activate.ejs:2`, `store` `activation-system/views/store/activate.ejs:24`.

Прямые render-вызовы: `activation-system/controllers/storeController.js:57`.

Формы:

| Метод | Action | Поля name | Кодировка | Источник |
| --- | --- | --- | --- | --- |
| POST | /<%= store.slug %>/logout | нет именованных полей | обычная форма | `activation-system/views/store/activate.ejs:26` |


AJAX в шаблоне (выражение адреса и начала вызова):

| Вызов | Источник |
| --- | --- |
| fetch(`/${storeSlug}/activate`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ voucherCode: code, storeSlug }) | `activation-system/views/store/activate.ejs:168` |
| fetch(`/${storeSlug}/history`) | `activation-system/views/store/activate.ejs:225` |


## views/store/layout.ejs

Переменные: `store` `activation-system/views/store/layout.ejs:7`, `body` `activation-system/views/store/layout.ejs:73`.

Прямые render-вызовы: не найдены; возможен include/layout — см. связи ниже.

HTML-форм нет. `activation-system/views/store/layout.ejs:1`

Внешние скрипты:

| Подключение | Источник |
| --- | --- |
| <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"> | `activation-system/views/store/layout.ejs:77` |


## views/store/login.ejs

Переменные: `store` `activation-system/views/store/login.ejs:2`.

Прямые render-вызовы: `activation-system/controllers/storeController.js:22`; `activation-system/controllers/storeController.js:66`; `activation-system/controllers/storeController.js:73`.

HTML-форм нет. `activation-system/views/store/login.ejs:1`

AJAX в шаблоне (выражение адреса и начала вызова):

| Вызов | Источник |
| --- | --- |
| fetch(`/${storeSlug}/otp`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ phone, storeSlug }) | `activation-system/views/store/login.ejs:117` |
| fetch(`/${storeSlug}/verify`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ phone: currentPhone, code, storeSlug }) | `activation-system/views/store/login.ejs:149` |

