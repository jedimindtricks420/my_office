# API_REFERENCE

Срез локального исходного кода: 2026-09-08. Корень ссылок — `/home/admin1/posa`; локальные незакоммиченные изменения включены. Источники — исполняемый код, схемы и конфигурация; существующая документация и комментарии не используются как доказательства. Статус фактов ниже: **подтверждено кодом**, если явно не указано «предположение на основе имени и использования». Статический анализ не подтверждает состояние рабочей БД или успешный запуск.

## Как читать справочник

Каждая строка — зарегистрированная пара HTTP-метод/путь с конкретным монтированием. Повторные регистрации сохранены. Поля `req.session` и cookies раскрыты в контракте обработчика ниже. Выражения параметров и ответов приведены из AST без комментариев: имя переменной в `res.json(result)` не выдается за готовую JSON-схему; возвращаемые структуры вспомогательных функций перечислены отдельно.

Глобальная цепочка, статика и зависимость от Host описаны в ARCHITECTURE. Неуказанный в обработчике status не означает отсутствие ошибок middleware/Express. В приложении нет собственного завершающего error middleware: `activation-system/index.js:216`.

Клиентский роутер имеет буквальные префиксы `/wallet`, `/client` и отдельное подключение на Host `wallet.*`; поэтому `/wallet/client/dashboard` и `/client/client/dashboard` действительно зарегистрированы. Корневые `/client/dashboard`, `/client-register`, `/client-verify`, `/wallet` получают ожидаемые обработчики через вызов по субдомену. `activation-system/index.js:190` `activation-system/index.js:194` `activation-system/routes/clientRoutes.js:18`.

`GET /activate` сначала обрабатывает wallet smart redirect, а при `next()` — публичный обработчик. `/:storeSlug` зарегистрирован до `/email`, поэтому GET `/email` попадает в магазин `email` и публичная форма недостижима обычным запросом. `activation-system/routes/wallet.js:156` `activation-system/routes/storeRoutes.js:15` `activation-system/index.js:213`.

## Все зарегистрированные эндпоинты

| Метод | Полный путь / Host | Регистрация | Middleware по порядку | Path params | Query | Body | Обработчик | Ответы | HTTP ошибки |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| GET | /wallet/icons/icon-192.png | `activation-system/index.js:60` | `expressLayouts`<br>`express.urlencoded({ extended: true })`<br>`express.json({   verify: (req, res, buf) => {     req.rawBody = buf;   }, })`<br>`cookieParser()` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/index.js:60` | `res.setHeader('Cache-Control', 'public, max-age=31536000, immutable')` `activation-system/index.js:61`<br>`res.sendFile(path.join(pwaIconsPath, '192x192.png'))` `activation-system/index.js:62` | Явный HTTP status не задан |
| GET | /wallet/icons/icon-512.png | `activation-system/index.js:65` | `expressLayouts`<br>`express.urlencoded({ extended: true })`<br>`express.json({   verify: (req, res, buf) => {     req.rawBody = buf;   }, })`<br>`cookieParser()` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/index.js:65` | `res.setHeader('Cache-Control', 'public, max-age=31536000, immutable')` `activation-system/index.js:66`<br>`res.sendFile(path.join(pwaIconsPath, '512x512.png'))` `activation-system/index.js:67` | Явный HTTP status не задан |
| GET | /wallet/icons/icon-512-maskable.png | `activation-system/index.js:70` | `expressLayouts`<br>`express.urlencoded({ extended: true })`<br>`express.json({   verify: (req, res, buf) => {     req.rawBody = buf;   }, })`<br>`cookieParser()` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/index.js:70` | `res.setHeader('Cache-Control', 'public, max-age=31536000, immutable')` `activation-system/index.js:71`<br>`res.sendFile(path.join(pwaIconsPath, 'maskable-icon.png'))` `activation-system/index.js:72` | Явный HTTP status не задан |
| GET | /wallet/icons/vec-icon.png | `activation-system/index.js:75` | `expressLayouts`<br>`express.urlencoded({ extended: true })`<br>`express.json({   verify: (req, res, buf) => {     req.rawBody = buf;   }, })`<br>`cookieParser()` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/index.js:75` | `res.setHeader('Cache-Control', 'public, max-age=31536000, immutable')` `activation-system/index.js:76`<br>`res.sendFile(path.join(pwaIconsPath, 'vec-icon.png'))` `activation-system/index.js:77` | Явный HTTP status не задан |
| GET | /.well-known/assetlinks.json | `activation-system/index.js:83` | `expressLayouts`<br>`express.urlencoded({ extended: true })`<br>`express.json({   verify: (req, res, buf) => {     req.rawBody = buf;   }, })`<br>`cookieParser()` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/index.js:83` | `res.setHeader('Content-Type', 'application/json')` `activation-system/index.js:89`<br>`res.setHeader('Cache-Control', 'public, max-age=3600')` `activation-system/index.js:90`<br>`res.end(content)` `activation-system/index.js:91`<br>`res.status(500).send('Internal server error')` `activation-system/index.js:94` | 500 |
| GET | /admin/dashboard | `activation-system/routes/adminRoutes.js:31` / mount `activation-system/index.js:178` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`allowOfficeDashboard` `activation-system/middleware/auth.js:16` | — | Прямое чтение не найдено | Прямое чтение не найдено | dashboardController.showAdminDashboard `activation-system/controllers/admin/dashboardController.js:6` | `res.redirect('/kassa/dashboard')` `activation-system/controllers/admin/dashboardController.js:11`<br>`res.render('pages/admin-dashboard-finance', { user: req.session.user, stats })` `activation-system/controllers/admin/dashboardController.js:75`<br>`res.status(500).send('Finance Dashboard Error')` `activation-system/controllers/admin/dashboardController.js:80`<br>`res.render('pages/admin-dashboard-content', { user: req.session.user, stats: { productsCount, storesCount, criticalProducts } })` `activation-system/controllers/admin/dashboardController.js:106`<br>`res.status(500).send('Content Dashboard Error')` `activation-system/controllers/admin/dashboardController.js:115`<br>`res.render('pages/admin-dashboard-support', { user: req.session.user, stats: { pendingRequests, recentActivations } })` `activation-system/controllers/admin/dashboardController.js:132`<br>`res.status(500).send('Support Dashboard Error')` `activation-system/controllers/admin/dashboardController.js:140`<br>`res.render('pages/dashboard-admin', { user: req.session.user, stats: { users: usersCount, vendors: merchantsCount, merchants: merchantsCount, sales: salesCount, vouchers: vouchersCount, clients: clientsCount } })` `activation-system/controllers/admin/dashboardController.js:37`<br>`res.status(500).send('Ошибка при загрузке данных дашборда')` `activation-system/controllers/admin/dashboardController.js:50` | 500; отказ доступа — см. middleware |
| GET | /admin/kassas | `activation-system/routes/adminRoutes.js:34` / mount `activation-system/index.js:178` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`allowFinance` `activation-system/middleware/auth.js:23` | — | Прямое чтение не найдено | Прямое чтение не найдено | kassaController.showKassaList `activation-system/controllers/admin/kassaController.js:4` | `res.render('pages/admin-kassa-list', { kassas, user: req.session.user })` `activation-system/controllers/admin/kassaController.js:13` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/kassas/add | `activation-system/routes/adminRoutes.js:35` / mount `activation-system/index.js:178` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | kassaController.showCreateKassaForm `activation-system/controllers/admin/kassaController.js:17` | `res.render('pages/admin-kassa-add', { vendors, error: null, user: req.session.user })` `activation-system/controllers/admin/kassaController.js:22` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/kassas/add | `activation-system/routes/adminRoutes.js:36` / mount `activation-system/index.js:178` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | `{ name, legalName, inn, clickMerchantId, clickServiceId, clickSecretKey, clickMerchantUserId, clickEnv, paymeMerchantId, paymeKey, paymeTestKey, paymeEnv, alifTokenProduction, alifTokenSandbox, alifSecretKeyProduction, alifSecretKeySandbox, alifEnv, contactPerson, contactPhone, contactEmail, vendorIds, } = req.body` `activation-system/controllers/admin/kassaController.js:28` | kassaController.handleCreateKassa `activation-system/controllers/admin/kassaController.js:26` | `res.render('pages/admin-kassa-add', { vendors, error: 'Укажите название кассы', user: req.session.user, })` `activation-system/controllers/admin/kassaController.js:39`<br>`res.redirect('/admin/kassas')` `activation-system/controllers/admin/kassaController.js:78`<br>`res.render('pages/admin-kassa-add', { vendors, error: 'Ошибка при создании кассы', user: req.session.user, })` `activation-system/controllers/admin/kassaController.js:82` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/kassas/:id/edit | `activation-system/routes/adminRoutes.js:37` / mount `activation-system/index.js:178` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | kassaController.showEditKassaForm `activation-system/controllers/admin/kassaController.js:89` | `res.status(404).send('Касса не найдена')` `activation-system/controllers/admin/kassaController.js:94`<br>`res.render('pages/admin-kassa-edit', { kassa, availableVendors, error: null, user: req.session.user, })` `activation-system/controllers/admin/kassaController.js:100` | 404; отказ доступа — см. middleware |
| POST | /admin/kassas/:id/edit | `activation-system/routes/adminRoutes.js:38` / mount `activation-system/index.js:178` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | `{ name, legalName, inn, isActive, clickMerchantId, clickServiceId, clickSecretKey, clickMerchantUserId, clickEnv, paymeMerchantId, paymeKey, paymeTestKey, paymeEnv, alifTokenProduction, alifTokenSandbox, alifSecretKeyProduction, alifSecretKeySandbox, alifEnv, contactPerson, contactPhone, contactEmail, vendorIds, } = req.body` `activation-system/controllers/admin/kassaController.js:109` | kassaController.handleUpdateKassa `activation-system/controllers/admin/kassaController.js:106` | `res.redirect('/admin/kassas')` `activation-system/controllers/admin/kassaController.js:157`<br>`res.redirect(ˋ/admin/kassas/${req.params.id}/editˋ)` `activation-system/controllers/admin/kassaController.js:160` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/sales | `activation-system/routes/adminRoutes.js:42` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | — | Прямое чтение не найдено | Прямое чтение не найдено | saleController.showSales `activation-system/controllers/admin/saleController.js:4` | `res.render('pages/admin-sales', { sales, stats, user: req.session.user, })` `activation-system/controllers/admin/saleController.js:58` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/merchants | `activation-system/routes/adminRoutes.js:43` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | — | Прямое чтение не найдено | Прямое чтение не найдено | merchantController.showMerchantsWithDebt `activation-system/controllers/admin/merchantController.js:5` | `res.render('pages/admin-merchants', { merchants, stats, user: req.session.user, })` `activation-system/controllers/admin/merchantController.js:56` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/merchant/:id/pay | `activation-system/routes/adminRoutes.js:44` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | id | Прямое чтение не найдено | Прямое чтение не найдено | merchantController.showPaymentForm `activation-system/controllers/admin/merchantController.js:225` | `res.render('pages/admin-pay-merchant', { merchant, actualDebt, error: null, user: req.session.user, })` `activation-system/controllers/admin/merchantController.js:236` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/merchant/:id/pay | `activation-system/routes/adminRoutes.js:45` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | id | Прямое чтение не найдено | `{ amount, comment } = req.body` `activation-system/controllers/admin/merchantController.js:247` | merchantController.handleMerchantPayment `activation-system/controllers/admin/merchantController.js:245` | `res.status(404).send('Merchant not found')` `activation-system/controllers/admin/merchantController.js:250`<br>`res.status(400).render('pages/admin-pay-merchant', { merchant, actualDebt, user: req.session.user, error: 'Сумма платежа должна быть больше нуля', })` `activation-system/controllers/admin/merchantController.js:260`<br>`res.redirect('/admin/merchants')` `activation-system/controllers/admin/merchantController.js:326` | 404, 400; отказ доступа — см. middleware |
| GET | /admin/merchant/:id/transactions | `activation-system/routes/adminRoutes.js:46` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | id | Прямое чтение не найдено | Прямое чтение не найдено | merchantController.showMerchantTransactions `activation-system/controllers/admin/merchantController.js:330` | `res.render('pages/admin-merchant-transactions', { merchant, transactions, user: req.session.user, })` `activation-system/controllers/admin/merchantController.js:342` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/merchant/:id/payments | `activation-system/routes/adminRoutes.js:47` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | id | Прямое чтение не найдено | Прямое чтение не найдено | merchantController.showMerchantPaymentHistory `activation-system/controllers/admin/merchantController.js:350` | `res.status(404).send('Мерчант не найден')` `activation-system/controllers/admin/merchantController.js:361`<br>`res.render('pages/admin-merchant-payments', { merchant, payments: merchant.payments, })` `activation-system/controllers/admin/merchantController.js:364` | 404; отказ доступа — см. middleware |
| GET | /admin/vendor/:id/pay | `activation-system/routes/adminRoutes.js:49` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | id | Прямое чтение не найдено | Прямое чтение не найдено | vendorController.showVendorPaymentForm `activation-system/controllers/admin/vendorController.js:216` | `res.render('pages/admin-pay-vendor', { vendor, error: null, user: req.session.user })` `activation-system/controllers/admin/vendorController.js:220` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/vendor/:id/pay | `activation-system/routes/adminRoutes.js:50` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | id | Прямое чтение не найдено | `{ amount, comment } = req.body` `activation-system/controllers/admin/vendorController.js:231` | vendorController.handleVendorPayment `activation-system/controllers/admin/vendorController.js:229` | `res.status(404).send('Vendor not found')` `activation-system/controllers/admin/vendorController.js:234`<br>`res.status(400).render('pages/admin-pay-vendor', { vendor, error: 'Сумма пополнения должна быть больше нуля', user: req.session.user })` `activation-system/controllers/admin/vendorController.js:238`<br>`res.redirect('/admin/vendors')` `activation-system/controllers/admin/vendorController.js:302` | 404, 400; отказ доступа — см. middleware |
| GET | /admin/vendor/:id/transactions | `activation-system/routes/adminRoutes.js:51` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | id | Прямое чтение не найдено | Прямое чтение не найдено | vendorController.showVendorTransactions `activation-system/controllers/admin/vendorController.js:306` | `res.render('pages/admin-vendor-transactions', { vendor, transactions, user: req.session.user })` `activation-system/controllers/admin/vendorController.js:316` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/add-merchant | `activation-system/routes/adminRoutes.js:56` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | merchantController.showAddMerchantForm `activation-system/controllers/admin/merchantController.js:63` | `res.render('pages/admin-add-merchant', { error: null, formData: { username: '', legalInfo: '', status: 'active', }, user: req.session.user, })` `activation-system/controllers/admin/merchantController.js:64` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/add-merchant | `activation-system/routes/adminRoutes.js:57` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | `req.body.username` `activation-system/controllers/admin/merchantController.js:76`<br>`req.body.username?.trim` `activation-system/controllers/admin/merchantController.js:76`<br>`req.body.legalInfo` `activation-system/controllers/admin/merchantController.js:77`<br>`req.body.legalInfo?.trim` `activation-system/controllers/admin/merchantController.js:77`<br>`req.body.status` `activation-system/controllers/admin/merchantController.js:78`<br>`req.body.password` `activation-system/controllers/admin/merchantController.js:79`<br>`req.body.password?.trim` `activation-system/controllers/admin/merchantController.js:79` | merchantController.handleAddMerchant `activation-system/controllers/admin/merchantController.js:75` | `res.status(400).render('pages/admin-add-merchant', { error: 'Заполните логин, пароль и юридическую информацию', formData, user: req.session.user, })` `activation-system/controllers/admin/merchantController.js:88`<br>`res.redirect('/admin/merchants')` `activation-system/controllers/admin/merchantController.js:116`<br>`res.status(400).render('pages/admin-add-merchant', { error: errorMessage, formData, user: req.session.user, })` `activation-system/controllers/admin/merchantController.js:124` | 400; отказ доступа — см. middleware |
| GET | /admin/merchants/edit/:id | `activation-system/routes/adminRoutes.js:58` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | merchantController.showEditMerchantForm `activation-system/controllers/admin/merchantController.js:132` | `res.status(404).send('Merchant not found')` `activation-system/controllers/admin/merchantController.js:137`<br>`res.render('pages/admin-edit-merchant', { merchant, error: null, user: req.session.user, })` `activation-system/controllers/admin/merchantController.js:145` | 404; отказ доступа — см. middleware |
| POST | /admin/merchants/edit/:id | `activation-system/routes/adminRoutes.js:59` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | `req.body.username` `activation-system/controllers/admin/merchantController.js:160`<br>`req.body.username?.trim` `activation-system/controllers/admin/merchantController.js:160`<br>`req.body.legalInfo` `activation-system/controllers/admin/merchantController.js:161`<br>`req.body.legalInfo?.trim` `activation-system/controllers/admin/merchantController.js:161`<br>`req.body.status` `activation-system/controllers/admin/merchantController.js:162`<br>`req.body.password` `activation-system/controllers/admin/merchantController.js:163`<br>`req.body.password?.trim` `activation-system/controllers/admin/merchantController.js:163` | merchantController.handleEditMerchant `activation-system/controllers/admin/merchantController.js:152` | `res.status(404).send('Merchant not found')` `activation-system/controllers/admin/merchantController.js:157`<br>`res.status(400).render('pages/admin-edit-merchant', { merchant: merchantView, error: 'Логин и юридическая информация обязательны', user: req.session.user, })` `activation-system/controllers/admin/merchantController.js:174`<br>`res.redirect('/admin/merchants')` `activation-system/controllers/admin/merchantController.js:208`<br>`res.status(400).render('pages/admin-edit-merchant', { merchant: merchantView, error: errorMessage, user: req.session.user, })` `activation-system/controllers/admin/merchantController.js:216` | 404, 400; отказ доступа — см. middleware |
| GET | /admin/clients | `activation-system/routes/adminRoutes.js:63` / mount `activation-system/index.js:178` | `allowSupport` `activation-system/middleware/auth.js:37` | — | `req.query.page` `activation-system/controllers/admin/clientController.js:6`<br>`req.query.limit` `activation-system/controllers/admin/clientController.js:7`<br>`req.query.search` `activation-system/controllers/admin/clientController.js:9` | Прямое чтение не найдено | clientController.showAllClients `activation-system/controllers/admin/clientController.js:4` | `res.render('pages/admin-clients', { user: req.session.user, clients, currentPage: page, totalPages, totalClients, search, limit, hasNextPage: page < totalPages, hasPrevPage: page > 1, nextPage: page + 1, prevPage: page - 1 })` `activation-system/controllers/admin/clientController.js:53`<br>`res.status(500).send('Ошибка при загрузке данных клиентов')` `activation-system/controllers/admin/clientController.js:68` | 500; отказ доступа — см. middleware |
| GET | /admin/clients/:id | `activation-system/routes/adminRoutes.js:64` / mount `activation-system/index.js:178` | `allowSupport` `activation-system/middleware/auth.js:37` | id | Прямое чтение не найдено | Прямое чтение не найдено | clientController.showClientDetails `activation-system/controllers/admin/clientController.js:73` | `res.status(404).send('Клиент не найден')` `activation-system/controllers/admin/clientController.js:89`<br>`res.render('pages/admin-client-details-simple', { user: req.session.user, client })` `activation-system/controllers/admin/clientController.js:92`<br>`res.status(500).send('Ошибка при загрузке данных клиента')` `activation-system/controllers/admin/clientController.js:98` | 404, 500; отказ доступа — см. middleware |
| POST | /admin/clients/:id/delete | `activation-system/routes/adminRoutes.js:65` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | clientController.deleteClient `activation-system/controllers/admin/clientController.js:103` | `res.redirect('/admin/clients?message=Клиент успешно удален')` `activation-system/controllers/admin/clientController.js:111`<br>`res.redirect('/admin/clients?error=Ошибка при удалении клиента')` `activation-system/controllers/admin/clientController.js:114` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/products | `activation-system/routes/adminRoutes.js:69` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | Прямое чтение не найдено | productController.showAllProducts `activation-system/controllers/admin/productController.js:12` | `res.render('pages/admin-products', { products: enrichedProducts, user: req.session.user })` `activation-system/controllers/admin/productController.js:39` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/add-product | `activation-system/routes/adminRoutes.js:70` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | Прямое чтение не найдено | productController.showAddProductForm `activation-system/controllers/admin/productController.js:47` | `res.render('pages/admin-add-product', { vendors, user: req.session.user, error: null })` `activation-system/controllers/admin/productController.js:49` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/add-product | `activation-system/routes/adminRoutes.js:71` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | `{ name, price, status, vendorId, merchantCommissionPercent, vendorCommissionPercent, rokkySku, drwebSku, manastoreVariantId } = req.body` `activation-system/controllers/admin/productController.js:54` | productController.handleAddProduct `activation-system/controllers/admin/productController.js:53` | `res.redirect('/admin/products')` `activation-system/controllers/admin/productController.js:71`<br>`res.render('pages/admin-add-product', { error: 'Ошибка при создании товара', vendors: await prisma.vendor.findMany({ orderBy: { name: 'asc' } }), user: req.session.user })` `activation-system/controllers/admin/productController.js:73` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/edit-product/:id | `activation-system/routes/adminRoutes.js:72` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | id | Прямое чтение не найдено | Прямое чтение не найдено | productController.showEditProductForm `activation-system/controllers/admin/productController.js:83` | `res.status(404).send('Товар не найден')` `activation-system/controllers/admin/productController.js:90`<br>`res.render('pages/edit-product', { product, vendor: product.vendor, user: req.session.user, templateSchema: JSON.stringify(schema), hasCustomTemplate: !!product.receiptTemplate, hasCustomInstructions: !!product.redeemInstructions, })` `activation-system/controllers/admin/productController.js:98` | 404; отказ доступа — см. middleware |
| POST | /admin/edit-product/:id | `activation-system/routes/adminRoutes.js:73` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | id | Прямое чтение не найдено | `{ name, price, status, merchantCommissionPercent, vendorCommissionPercent, rokkySku, drwebSku, manastoreVariantId, useCustomTemplate, templateSchema, walletOnDemandEnabled, useCustomInstructions, redeemInstructions, } = req.body` `activation-system/controllers/admin/productController.js:115` | productController.handleEditProduct `activation-system/controllers/admin/productController.js:113` | `res.redirect('/admin/products')` `activation-system/controllers/admin/productController.js:127` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/delete-product/:id | `activation-system/routes/adminRoutes.js:74` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | id | Прямое чтение не найдено | Прямое чтение не найдено | productController.handleDeleteProduct `activation-system/controllers/admin/productController.js:179` | `res.redirect('/admin/products')` `activation-system/controllers/admin/productController.js:183`<br>`res.render('pages/admin-products', { products: enrichedProducts, user: req.session.user, error: errorMessage })` `activation-system/controllers/admin/productController.js:207`<br>`res.status(500).send(errorMessage)` `activation-system/controllers/admin/productController.js:214` | 500; отказ доступа — см. middleware |
| POST | /admin/products/:id/receipt/preview | `activation-system/routes/adminRoutes.js:75` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | id | Прямое чтение не найдено | `req.body?.schema` `activation-system/controllers/admin/productController.js:231` | productController.previewProductReceiptTemplate `activation-system/controllers/admin/productController.js:220` | `res.status(404).send('Товар не найден')` `activation-system/controllers/admin/productController.js:228`<br>`res.send(html)` `activation-system/controllers/admin/productController.js:241`<br>`res.status(500).send('<h1>Ошибка рендера чека</h1><p>Проверьте структуру шаблона.</p>')` `activation-system/controllers/admin/productController.js:244` | 404, 500; отказ доступа — см. middleware |
| GET | /admin/product-images | `activation-system/routes/adminRoutes.js:77` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | Прямое чтение не найдено | productImageController.showProductImagesList `activation-system/controllers/admin/productImageController.js:6` | `res.render('pages/admin-product-images', { products, user: req.session.user, })` `activation-system/controllers/admin/productImageController.js:18` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/product-images/:productId | `activation-system/routes/adminRoutes.js:78` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | productId | Прямое чтение не найдено | Прямое чтение не найдено | productImageController.showManageProductImages `activation-system/controllers/admin/productImageController.js:25` | `res.status(404).send('Товар не найден')` `activation-system/controllers/admin/productImageController.js:35`<br>`res.render('pages/admin-product-images-manage', { product, user: req.session.user, error: null, })` `activation-system/controllers/admin/productImageController.js:38` | 404; отказ доступа — см. middleware |
| POST | /admin/product-images/:productId/upload | `activation-system/routes/adminRoutes.js:79` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30`<br>`uploadProductImage.single('image')` | productId | Прямое чтение не найдено | Прямое чтение не найдено | productImageController.uploadProductImage `activation-system/controllers/admin/productImageController.js:46` | `res.status(404).send('Товар не найден')` `activation-system/controllers/admin/productImageController.js:51`<br>`res.redirect(ˋ/admin/product-images/${productId}ˋ)` `activation-system/controllers/admin/productImageController.js:55` | 404; отказ доступа — см. middleware |
| POST | /admin/product-images/image/:imageId/primary | `activation-system/routes/adminRoutes.js:80` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | imageId | Прямое чтение не найдено | Прямое чтение не найдено | productImageController.setPrimaryImage `activation-system/controllers/admin/productImageController.js:77` | `res.status(404).send('Изображение не найдено')` `activation-system/controllers/admin/productImageController.js:82`<br>`res.redirect(ˋ/admin/product-images/${image.productId}ˋ)` `activation-system/controllers/admin/productImageController.js:96` | 404; отказ доступа — см. middleware |
| POST | /admin/product-images/image/:imageId/delete | `activation-system/routes/adminRoutes.js:81` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | imageId | Прямое чтение не найдено | Прямое чтение не найдено | productImageController.deleteProductImage `activation-system/controllers/admin/productImageController.js:100` | `res.status(404).send('Изображение не найдено')` `activation-system/controllers/admin/productImageController.js:105`<br>`res.redirect(ˋ/admin/product-images/${productId}ˋ)` `activation-system/controllers/admin/productImageController.js:132` | 404; отказ доступа — см. middleware |
| GET | /admin/vouchers | `activation-system/routes/adminRoutes.js:83` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | Прямое чтение не найдено | voucherController.showAllVouchers `activation-system/controllers/admin/voucherController.js:4` | `res.render('pages/admin-vouchers', { vouchers })` `activation-system/controllers/admin/voucherController.js:6` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/add-voucher | `activation-system/routes/adminRoutes.js:84` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | Прямое чтение не найдено | voucherController.showAddVoucherForm `activation-system/controllers/admin/voucherController.js:10` | `res.render('pages/admin-add-voucher', { products })` `activation-system/controllers/admin/voucherController.js:12` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/add-voucher | `activation-system/routes/adminRoutes.js:85` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | `{ value, productId, type } = req.body` `activation-system/controllers/admin/voucherController.js:17` | voucherController.handleAddVoucher `activation-system/controllers/admin/voucherController.js:16` | `res.redirect('/admin/vouchers')` `activation-system/controllers/admin/voucherController.js:30` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/vouchers/add | `activation-system/routes/adminRoutes.js:86` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | Прямое чтение не найдено | voucherController.showAddVouchersPage `activation-system/controllers/admin/voucherController.js:34` | `res.render('pages/admin-add-vouchers', { products, user: req.session.user })` `activation-system/controllers/admin/voucherController.js:36` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/vouchers/add | `activation-system/routes/adminRoutes.js:87` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30`<br>`uploadVoucherFile.single('voucherFile')` | — | Прямое чтение не найдено | `{ productId, voucherList, type } = req.body` `activation-system/controllers/admin/voucherController.js:44` | voucherController.addVouchers `activation-system/controllers/admin/voucherController.js:43` | `res.status(400).send('Товар не найден')` `activation-system/controllers/admin/voucherController.js:48`<br>`res.status(400).send('Не найдено ни одного кода — вставьте текст или загрузите файл')` `activation-system/controllers/admin/voucherController.js:62`<br>`res.status(400).send(ˋСлишком много строк (${uniqueValues.length}). Максимум ${MAX_VOUCHER_LINES} за раз.ˋ)` `activation-system/controllers/admin/voucherController.js:65`<br>`res.send(ˋ <html><head><meta charset="UTF-8"><script src="https://cdn.tailwindcss.com"></script></head> <body class="bg-slate-50 min-h-screen flex items-center justify-center"> <div class="bg-white rounded-lg shadow-lg p-8 max-w-md w-full text-center"> <h2 class="text-xl font-bold text-slate-900 mb-2">Готово</h2> <p class="text-slate-600">Добавлено новых кодов: <strong>${result.count}</strong></p> ${skipped > 0 ? ˋ<p class="text-amber-600 text-sm mt-1">Пропущено как дубликаты: ${skipped}</p>ˋ : ''} <a href="/admin/vouchers" class="mt-6 inline-block bg-indigo-600 text-white px-4 py-2 rounded-lg">К списку ваучеров</a> </div> </body></html> ˋ)` `activation-system/controllers/admin/voucherController.js:79` | 400; отказ доступа — см. middleware |
| POST | /admin/vouchers/generate | `activation-system/routes/adminRoutes.js:88` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8`<br>`generationLimiter` | — | Прямое чтение не найдено | `{ productId, type, series, count } = req.body` `activation-system/controllers/admin/voucherController.js:94` | voucherController.generateVouchers `activation-system/controllers/admin/voucherController.js:93` | `res.redirect('/admin/vouchers')` `activation-system/controllers/admin/voucherController.js:132` | Явный HTTP status не задан; 429 (limiter); отказ доступа — см. middleware |
| GET | /admin/stores | `activation-system/routes/adminRoutes.js:90` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | Прямое чтение не найдено | storeController.showStores `activation-system/controllers/admin/storeController.js:6` | `res.render('pages/admin-stores', { stores, user: req.session.user })` `activation-system/controllers/admin/storeController.js:14` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/stores/add | `activation-system/routes/adminRoutes.js:91` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | Прямое чтение не найдено | storeController.showAddStoreForm `activation-system/controllers/admin/storeController.js:21` | `res.render('pages/admin-add-store', { vendors, products, error: null, user: req.session.user })` `activation-system/controllers/admin/storeController.js:27` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/stores/add | `activation-system/routes/adminRoutes.js:92` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | `{ slug, name, isActive, productIds } = req.body` `activation-system/controllers/admin/storeController.js:37` | storeController.handleAddStore `activation-system/controllers/admin/storeController.js:36` | `res.redirect('/admin/stores')` `activation-system/controllers/admin/storeController.js:55`<br>`res.render('pages/admin-add-store', { vendors, products, error: 'Ошибка при создании магазина', user: req.session.user })` `activation-system/controllers/admin/storeController.js:61` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/stores/edit/:id | `activation-system/routes/adminRoutes.js:93` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | id | Прямое чтение не найдено | Прямое чтение не найдено | storeController.showEditStoreForm `activation-system/controllers/admin/storeController.js:71` | `res.status(404).send('Магазин не найден')` `activation-system/controllers/admin/storeController.js:80`<br>`res.render('pages/admin-edit-store', { store, allProducts, user: req.session.user })` `activation-system/controllers/admin/storeController.js:87` | 404; отказ доступа — см. middleware |
| POST | /admin/stores/edit/:id | `activation-system/routes/adminRoutes.js:94` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30`<br>`upload.single('logo')` | id | Прямое чтение не найдено | `{ slug, name, isActive, productIds } = req.body` `activation-system/controllers/admin/storeController.js:97`<br>`req.body.themeColor` `activation-system/controllers/admin/storeController.js:108`<br>`req.body.backgroundColor` `activation-system/controllers/admin/storeController.js:109`<br>`req.body.activationSmsTemplate` `activation-system/controllers/admin/storeController.js:110`<br>`req.body.removeLogo` `activation-system/controllers/admin/storeController.js:119` | storeController.handleEditStore `activation-system/controllers/admin/storeController.js:95` | `res.redirect('/admin/stores')` `activation-system/controllers/admin/storeController.js:128`<br>`res.status(500).send('Ошибка при обновлении магазина')` `activation-system/controllers/admin/storeController.js:131` | 500; отказ доступа — см. middleware |
| GET | /admin/vendors | `activation-system/routes/adminRoutes.js:98` / mount `activation-system/index.js:178` | `allowFinanceOrContent` `activation-system/middleware/auth.js:45` | — | Прямое чтение не найдено | Прямое чтение не найдено | vendorController.showVendors `activation-system/controllers/admin/vendorController.js:13` | `res.render('pages/admin-vendors', { vendors, stats, user: req.session.user })` `activation-system/controllers/admin/vendorController.js:56` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/add-vendor | `activation-system/routes/adminRoutes.js:99` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | Прямое чтение не найдено | vendorController.showAddVendorForm `activation-system/controllers/admin/vendorController.js:65` | `res.render('pages/admin-add-vendor', { error: null, user: req.session.user })` `activation-system/controllers/admin/vendorController.js:66` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/add-vendor | `activation-system/routes/adminRoutes.js:100` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | `{ name, category, productType, description, defaultCommissionPercent } = req.body` `activation-system/controllers/admin/vendorController.js:71` | vendorController.handleAddVendor `activation-system/controllers/admin/vendorController.js:70` | `res.redirect('/admin/vendors')` `activation-system/controllers/admin/vendorController.js:84`<br>`res.render('pages/admin-add-vendor', { error: 'Ошибка при добавлении вендора', user: req.session.user })` `activation-system/controllers/admin/vendorController.js:86` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/vendors/edit/:id | `activation-system/routes/adminRoutes.js:101` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | id | Прямое чтение не найдено | Прямое чтение не найдено | vendorController.showEditVendorForm `activation-system/controllers/admin/vendorController.js:94` | `res.status(404).send('Вендор не найден')` `activation-system/controllers/admin/vendorController.js:106`<br>`res.render('pages/admin-edit-vendor', { vendor, user: req.session.user, templateSchema: JSON.stringify(schema), sampleProduct: vendor.products?.[0] &#124;&#124; null, })` `activation-system/controllers/admin/vendorController.js:111` | 404; отказ доступа — см. middleware |
| POST | /admin/vendors/edit/:id | `activation-system/routes/adminRoutes.js:102` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | id | Прямое чтение не найдено | `{ name, category, productType, description, templateSchema, defaultCommissionPercent, redeemInstructions, } = req.body` `activation-system/controllers/admin/vendorController.js:121` | vendorController.handleEditVendor `activation-system/controllers/admin/vendorController.js:120` | `res.redirect('/admin/vendors')` `activation-system/controllers/admin/vendorController.js:154`<br>`res.status(404).send('Вендор не найден')` `activation-system/controllers/admin/vendorController.js:169`<br>`res.render('pages/admin-edit-vendor', { vendor: { ...vendor, name, category, productType, description, defaultCommissionPercent }, user: req.session.user, templateSchema: JSON.stringify(schema), sampleProduct: vendor.products?.[0] &#124;&#124; null, error: 'Ошибка при обновлении вендора. Проверьте введенные данные.' })` `activation-system/controllers/admin/vendorController.js:174` | 404; отказ доступа — см. middleware |
| POST | /admin/vendors/:id/receipt/preview | `activation-system/routes/adminRoutes.js:103` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | id | Прямое чтение не найдено | `req.body?.schema` `activation-system/controllers/admin/vendorController.js:339` | vendorController.previewReceiptTemplate `activation-system/controllers/admin/vendorController.js:323` | `res.status(404).send('Вендор не найден')` `activation-system/controllers/admin/vendorController.js:336`<br>`res.send(html)` `activation-system/controllers/admin/vendorController.js:354`<br>`res.status(500).send('<h1>Ошибка рендера чека</h1><p>Проверьте структуру шаблона.</p>')` `activation-system/controllers/admin/vendorController.js:357` | 404, 500; отказ доступа — см. middleware |
| GET | /admin/add-vendor-users | `activation-system/routes/adminRoutes.js:110` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | showAddVendorUserForm `activation-system/controllers/admin/vendorController.js:186` | `res.render('pages/admin-add-vendor-user', { vendors })` `activation-system/controllers/admin/vendorController.js:188` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/add-vendor-users | `activation-system/routes/adminRoutes.js:111` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | `{ vendorId, username, password, note } = req.body` `activation-system/controllers/admin/vendorController.js:193` | createVendorUser `activation-system/controllers/admin/vendorController.js:192` | `res.send('Пользователь с таким логином уже существует.')` `activation-system/controllers/admin/vendorController.js:197`<br>`res.redirect('/admin/vendors')` `activation-system/controllers/admin/vendorController.js:212` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/add-user | `activation-system/routes/adminRoutes.js:116` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | userController.showCreateUserForm `activation-system/controllers/admin/userController.js:5` | `res.render('pages/admin-add-user', { error: null, kassas })` `activation-system/controllers/admin/userController.js:7` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/add-user | `activation-system/routes/adminRoutes.js:117` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | `{ username, password, role, kassaId } = req.body` `activation-system/controllers/admin/userController.js:12` | userController.handleCreateUser `activation-system/controllers/admin/userController.js:11` | `res.render('pages/admin-add-user', { error: 'Заполните все поля', kassas })` `activation-system/controllers/admin/userController.js:15`<br>`res.render('pages/admin-add-user', { error: 'Выберите кассу для пользователя с кассовой ролью', kassas })` `activation-system/controllers/admin/userController.js:19`<br>`res.redirect('/admin/users')` `activation-system/controllers/admin/userController.js:33`<br>`res.render('pages/admin-add-user', { error: 'Ошибка при создании пользователя', kassas })` `activation-system/controllers/admin/userController.js:35` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/users | `activation-system/routes/adminRoutes.js:118` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | userController.showUsersList `activation-system/controllers/admin/userController.js:40` | `res.render('pages/admin-users', { users, user: req.session.user })` `activation-system/controllers/admin/userController.js:42` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/users/:id/delete | `activation-system/routes/adminRoutes.js:119` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | userController.deleteUser `activation-system/controllers/admin/userController.js:46` | `res.redirect('/admin/users')` `activation-system/controllers/admin/userController.js:49`<br>`res.send('Ошибка при удалении')` `activation-system/controllers/admin/userController.js:51` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/users/:id/password | `activation-system/routes/adminRoutes.js:120` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | passwordController.showChangePasswordForm `activation-system/controllers/admin/passwordController.js:5` | `res.send('Пользователь не найден')` `activation-system/controllers/admin/passwordController.js:8`<br>`res.render('pages/admin-change-password', { userToEdit, error: null })` `activation-system/controllers/admin/passwordController.js:9` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/users/:id/password | `activation-system/routes/adminRoutes.js:121` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | `{ password } = req.body` `activation-system/controllers/admin/passwordController.js:15` | passwordController.changeUserPassword `activation-system/controllers/admin/passwordController.js:13` | `res.send('Пароль обязателен')` `activation-system/controllers/admin/passwordController.js:16`<br>`res.redirect('/admin/users')` `activation-system/controllers/admin/passwordController.js:21` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/rokky | `activation-system/routes/adminRoutes.js:126` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | rokkyController.showRokkyDashboard `activation-system/controllers/admin/rokkyController.js:4` | `res.render('pages/admin-rokky-dashboard', { products, skus, stats: { products: products.length, skus: skus.length, activations }, apiLogs, user: req.session.user })` `activation-system/controllers/admin/rokkyController.js:41` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/rokky/skus | `activation-system/routes/adminRoutes.js:127` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | rokkyController.showRokkySkus `activation-system/controllers/admin/rokkyController.js:55` | `res.render('pages/admin-rokky-skus', { skus, user: req.session.user })` `activation-system/controllers/admin/rokkyController.js:60` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/rokky/skus/add | `activation-system/routes/adminRoutes.js:128` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | rokkyController.showAddSkuForm `activation-system/controllers/admin/rokkyController.js:67` | `res.render('pages/admin-rokky-add-sku', { error: null, user: req.session.user })` `activation-system/controllers/admin/rokkyController.js:68` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/rokky/skus/add | `activation-system/routes/adminRoutes.js:129` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | `{ sku, name, description, category, costPrice, retailPrice } = req.body` `activation-system/controllers/admin/rokkyController.js:76` | rokkyController.handleAddSku `activation-system/controllers/admin/rokkyController.js:75` | `res.redirect('/admin/rokky/skus')` `activation-system/controllers/admin/rokkyController.js:91`<br>`res.render('pages/admin-rokky-add-sku', { error: 'Ошибка при создании SKU. Возможно, такой SKU уже существует.', user: req.session.user })` `activation-system/controllers/admin/rokkyController.js:94` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/rokky/skus/edit/:id | `activation-system/routes/adminRoutes.js:130` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | rokkyController.showEditSkuForm `activation-system/controllers/admin/rokkyController.js:102` | `res.status(404).send('SKU не найден')` `activation-system/controllers/admin/rokkyController.js:110`<br>`res.render('pages/admin-rokky-edit-sku', { sku, user: req.session.user })` `activation-system/controllers/admin/rokkyController.js:113` | 404; отказ доступа — см. middleware |
| POST | /admin/rokky/skus/edit/:id | `activation-system/routes/adminRoutes.js:131` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | `{ sku, name, description, category, costPrice, retailPrice, isActive } = req.body` `activation-system/controllers/admin/rokkyController.js:122` | rokkyController.handleEditSku `activation-system/controllers/admin/rokkyController.js:120` | `res.redirect('/admin/rokky/skus')` `activation-system/controllers/admin/rokkyController.js:149`<br>`res.status(500).send('Ошибка при обновлении SKU')` `activation-system/controllers/admin/rokkyController.js:152` | 500; отказ доступа — см. middleware |
| GET | /admin/rokky/products | `activation-system/routes/adminRoutes.js:132` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | rokkyController.showRokkyProducts `activation-system/controllers/admin/rokkyController.js:157` | `res.render('pages/admin-rokky-products', { products, skus, user: req.session.user })` `activation-system/controllers/admin/rokkyController.js:176` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/rokky/products/:id/bind | `activation-system/routes/adminRoutes.js:133` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | `{ rokkySku } = req.body` `activation-system/controllers/admin/rokkyController.js:186` | rokkyController.handleBindSku `activation-system/controllers/admin/rokkyController.js:184` | `res.redirect('/admin/rokky/products')` `activation-system/controllers/admin/rokkyController.js:196`<br>`res.status(500).send('Ошибка при привязке SKU')` `activation-system/controllers/admin/rokkyController.js:199` | 500; отказ доступа — см. middleware |
| GET | /admin/rokky/activations | `activation-system/routes/adminRoutes.js:134` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | `{ startDate, endDate, productId, phone, storeSlug } = req.query` `activation-system/controllers/admin/rokkyController.js:205` | Прямое чтение не найдено | rokkyController.showRokkyActivations `activation-system/controllers/admin/rokkyController.js:204` | `res.render('pages/admin-rokky-activations', { activations, products, filters: { startDate, endDate, productId, phone, storeSlug }, user: req.session.user })` `activation-system/controllers/admin/rokkyController.js:258` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/rokky/finance | `activation-system/routes/adminRoutes.js:135` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | — | `{ startDate, endDate } = req.query` `activation-system/controllers/admin/rokkyController.js:268` | Прямое чтение не найдено | rokkyController.showRokkyFinance `activation-system/controllers/admin/rokkyController.js:267` | `res.render('pages/admin-rokky-finance', { sales, stats: { totalRevenue, totalCost, totalMargin, salesCount: sales.length }, filters: { startDate, endDate }, user: req.session.user })` `activation-system/controllers/admin/rokkyController.js:303` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/drweb | `activation-system/routes/adminRoutes.js:139` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | drwebController.showDrWebDashboard `activation-system/controllers/admin/drwebController.js:4` | `res.render('pages/admin-drweb-dashboard', { products, skus, stats: { products: products.length, skus: skus.length, activations }, apiLogs, user: req.session.user })` `activation-system/controllers/admin/drwebController.js:41` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/drweb/skus | `activation-system/routes/adminRoutes.js:140` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | drwebController.showDrWebSkus `activation-system/controllers/admin/drwebController.js:55` | `res.render('pages/admin-drweb-skus', { skus, user: req.session.user })` `activation-system/controllers/admin/drwebController.js:60` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/drweb/skus/add | `activation-system/routes/adminRoutes.js:141` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | drwebController.showAddSkuForm `activation-system/controllers/admin/drwebController.js:67` | `res.render('pages/admin-drweb-add-sku', { error: null, user: req.session.user })` `activation-system/controllers/admin/drwebController.js:68` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/drweb/skus/add | `activation-system/routes/adminRoutes.js:142` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | `{ sku, name, description, category, costPrice, retailPrice } = req.body` `activation-system/controllers/admin/drwebController.js:76` | drwebController.handleAddSku `activation-system/controllers/admin/drwebController.js:75` | `res.redirect('/admin/drweb/skus')` `activation-system/controllers/admin/drwebController.js:91`<br>`res.render('pages/admin-drweb-add-sku', { error: 'Ошибка при создании SKU. Возможно, такой SKU уже существует.', user: req.session.user })` `activation-system/controllers/admin/drwebController.js:94` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/drweb/skus/edit/:id | `activation-system/routes/adminRoutes.js:143` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | drwebController.showEditSkuForm `activation-system/controllers/admin/drwebController.js:102` | `res.status(404).send('SKU не найден')` `activation-system/controllers/admin/drwebController.js:110`<br>`res.render('pages/admin-drweb-edit-sku', { sku, user: req.session.user })` `activation-system/controllers/admin/drwebController.js:113` | 404; отказ доступа — см. middleware |
| POST | /admin/drweb/skus/edit/:id | `activation-system/routes/adminRoutes.js:144` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | `{ sku, name, description, category, costPrice, retailPrice, isActive } = req.body` `activation-system/controllers/admin/drwebController.js:122` | drwebController.handleEditSku `activation-system/controllers/admin/drwebController.js:120` | `res.redirect('/admin/drweb/skus')` `activation-system/controllers/admin/drwebController.js:149`<br>`res.status(500).send('Ошибка при обновлении SKU')` `activation-system/controllers/admin/drwebController.js:152` | 500; отказ доступа — см. middleware |
| GET | /admin/drweb/products | `activation-system/routes/adminRoutes.js:145` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | drwebController.showDrWebProducts `activation-system/controllers/admin/drwebController.js:157` | `res.render('pages/admin-drweb-products', { products, skus, user: req.session.user })` `activation-system/controllers/admin/drwebController.js:176` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/drweb/products/:id/bind | `activation-system/routes/adminRoutes.js:146` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | `{ drwebSku } = req.body` `activation-system/controllers/admin/drwebController.js:186` | drwebController.handleBindSku `activation-system/controllers/admin/drwebController.js:184` | `res.redirect('/admin/drweb/products')` `activation-system/controllers/admin/drwebController.js:196`<br>`res.status(500).send('Ошибка при привязке SKU')` `activation-system/controllers/admin/drwebController.js:199` | 500; отказ доступа — см. middleware |
| GET | /admin/drweb/activations | `activation-system/routes/adminRoutes.js:147` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | `{ startDate, endDate, productId, phone, storeSlug } = req.query` `activation-system/controllers/admin/drwebController.js:205` | Прямое чтение не найдено | drwebController.showDrWebActivations `activation-system/controllers/admin/drwebController.js:204` | `res.render('pages/admin-drweb-activations', { activations, products, filters: { startDate, endDate, productId, phone, storeSlug }, user: req.session.user })` `activation-system/controllers/admin/drwebController.js:258` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/drweb/finance | `activation-system/routes/adminRoutes.js:148` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | — | `{ startDate, endDate } = req.query` `activation-system/controllers/admin/drwebController.js:268` | Прямое чтение не найдено | drwebController.showDrWebFinance `activation-system/controllers/admin/drwebController.js:267` | `res.render('pages/admin-drweb-finance', { sales, stats: { totalRevenue, totalCost, totalMargin, salesCount: sales.length }, filters: { startDate, endDate }, user: req.session.user })` `activation-system/controllers/admin/drwebController.js:302` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/manastore | `activation-system/routes/adminRoutes.js:152` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | manastoreController.showManaStoreDashboard `activation-system/controllers/admin/manastoreController.js:30` | `res.render('pages/admin-manastore-dashboard', { products, skus, stats: { products: products.length, skus: skus.length, activations }, apiLogs, environment: CURRENT_ENVIRONMENT, walletBalance, user: req.session.user })` `activation-system/controllers/admin/manastoreController.js:69` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/manastore/skus | `activation-system/routes/adminRoutes.js:153` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | manastoreController.showManaStoreSkus `activation-system/controllers/admin/manastoreController.js:85` | `res.render('pages/admin-manastore-skus', { skus, user: req.session.user })` `activation-system/controllers/admin/manastoreController.js:90` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/manastore/skus/add | `activation-system/routes/adminRoutes.js:154` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | manastoreController.showAddSkuForm `activation-system/controllers/admin/manastoreController.js:97` | `res.render('pages/admin-manastore-add-sku', { error: null, user: req.session.user })` `activation-system/controllers/admin/manastoreController.js:98` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/manastore/skus/add | `activation-system/routes/adminRoutes.js:155` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | `{ variantId, sku, name, description, category, costPrice, retailPrice, faceValue } = req.body` `activation-system/controllers/admin/manastoreController.js:106` | manastoreController.handleAddSku `activation-system/controllers/admin/manastoreController.js:105` | `res.redirect('/admin/manastore/skus')` `activation-system/controllers/admin/manastoreController.js:123`<br>`res.render('pages/admin-manastore-add-sku', { error: 'Ошибка при создании SKU. Возможно, такой Variant ID уже существует.', user: req.session.user })` `activation-system/controllers/admin/manastoreController.js:126` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/manastore/skus/edit/:id | `activation-system/routes/adminRoutes.js:156` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | manastoreController.showEditSkuForm `activation-system/controllers/admin/manastoreController.js:134` | `res.status(404).send('SKU не найден')` `activation-system/controllers/admin/manastoreController.js:142`<br>`res.render('pages/admin-manastore-edit-sku', { sku, user: req.session.user })` `activation-system/controllers/admin/manastoreController.js:145` | 404; отказ доступа — см. middleware |
| POST | /admin/manastore/skus/edit/:id | `activation-system/routes/adminRoutes.js:157` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | `{ variantId, sku, name, description, category, costPrice, retailPrice, faceValue, isActive } = req.body` `activation-system/controllers/admin/manastoreController.js:154` | manastoreController.handleEditSku `activation-system/controllers/admin/manastoreController.js:152` | `res.status(404).send('SKU не найден')` `activation-system/controllers/admin/manastoreController.js:159`<br>`res.redirect('/admin/manastore/skus')` `activation-system/controllers/admin/manastoreController.js:188`<br>`res.status(500).send('Ошибка при обновлении SKU')` `activation-system/controllers/admin/manastoreController.js:191` | 404, 500; отказ доступа — см. middleware |
| POST | /admin/manastore/skus/sync | `activation-system/routes/adminRoutes.js:158` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | manastoreController.handleSyncSkuCatalog `activation-system/controllers/admin/manastoreController.js:380` | `res.redirect('/admin/manastore/skus')` `activation-system/controllers/admin/manastoreController.js:428`<br>`res.status(500).send('Ошибка синхронизации каталога ManaStore')` `activation-system/controllers/admin/manastoreController.js:431` | 500; отказ доступа — см. middleware |
| GET | /admin/manastore/products | `activation-system/routes/adminRoutes.js:159` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | manastoreController.showManaStoreProducts `activation-system/controllers/admin/manastoreController.js:196` | `res.render('pages/admin-manastore-products', { products, skus, user: req.session.user })` `activation-system/controllers/admin/manastoreController.js:215` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /admin/manastore/products/:id/bind | `activation-system/routes/adminRoutes.js:160` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | `{ variantId } = req.body` `activation-system/controllers/admin/manastoreController.js:225` | manastoreController.handleBindSku `activation-system/controllers/admin/manastoreController.js:223` | `res.redirect('/admin/manastore/products')` `activation-system/controllers/admin/manastoreController.js:235`<br>`res.status(500).send('Ошибка при привязке варианта')` `activation-system/controllers/admin/manastoreController.js:238` | 500; отказ доступа — см. middleware |
| GET | /admin/manastore/activations | `activation-system/routes/adminRoutes.js:161` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | `{ startDate, endDate, productId, phone, storeSlug, status } = req.query` `activation-system/controllers/admin/manastoreController.js:244` | Прямое чтение не найдено | manastoreController.showManaStoreActivations `activation-system/controllers/admin/manastoreController.js:243` | `res.render('pages/admin-manastore-activations', { activations: filteredActivations, products, stores, filters: { startDate, endDate, productId, phone, storeSlug, status }, user: req.session.user })` `activation-system/controllers/admin/manastoreController.js:308` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/manastore/finance | `activation-system/routes/adminRoutes.js:162` / mount `activation-system/index.js:178` | `allowFinance` `activation-system/middleware/auth.js:23` | — | `{ startDate, endDate } = req.query` `activation-system/controllers/admin/manastoreController.js:319` | Прямое чтение не найдено | manastoreController.showManaStoreFinance `activation-system/controllers/admin/manastoreController.js:318` | `res.render('pages/admin-manastore-finance', { sales, stats: { totalRevenue, totalCost, totalMargin, salesCount: sales.length }, walletBalance, environment: CURRENT_ENVIRONMENT, filters: { startDate, endDate }, user: req.session.user })` `activation-system/controllers/admin/manastoreController.js:364` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/telegram-bots | `activation-system/routes/adminRoutes.js:166` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | telegramBotController.listBots `activation-system/controllers/admin/telegramBotController.js:5` | `res.render('pages/admin-telegram-bots', { bots, user: req.session.user })` `activation-system/controllers/admin/telegramBotController.js:18`<br>`res.status(500).send('Ошибка при загрузке списка ботов')` `activation-system/controllers/admin/telegramBotController.js:24` | 500; отказ доступа — см. middleware |
| GET | /admin/telegram-bots/create | `activation-system/routes/adminRoutes.js:167` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | Прямое чтение не найдено | telegramBotController.showCreateForm `activation-system/controllers/admin/telegramBotController.js:29` | `res.render('pages/admin-telegram-bot-form', { bot: null, stores, error: null, user: req.session.user })` `activation-system/controllers/admin/telegramBotController.js:35`<br>`res.status(500).send('Ошибка при загрузке формы')` `activation-system/controllers/admin/telegramBotController.js:43` | 500; отказ доступа — см. middleware |
| POST | /admin/telegram-bots/create | `activation-system/routes/adminRoutes.js:168` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | — | Прямое чтение не найдено | `{ name, token, authorizedUsers, storeIds, isActive } = req.body` `activation-system/controllers/admin/telegramBotController.js:50` | telegramBotController.createBot `activation-system/controllers/admin/telegramBotController.js:48` | `res.render('pages/admin-telegram-bot-form', { bot: null, stores, error: ˋНевалидный токен бота: ${validation.error}ˋ, user: req.session.user })` `activation-system/controllers/admin/telegramBotController.js:56`<br>`res.redirect('/admin/telegram-bots')` `activation-system/controllers/admin/telegramBotController.js:95`<br>`res.render('pages/admin-telegram-bot-form', { bot: null, stores, error: 'Ошибка при создании бота', user: req.session.user })` `activation-system/controllers/admin/telegramBotController.js:99` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /admin/telegram-bots/:id/edit | `activation-system/routes/adminRoutes.js:169` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | telegramBotController.showEditForm `activation-system/controllers/admin/telegramBotController.js:109` | `res.status(404).send('Бот не найден')` `activation-system/controllers/admin/telegramBotController.js:125`<br>`res.render('pages/admin-telegram-bot-form', { bot: { ...bot, authorizedUsersString: usersString }, stores, error: null, user: req.session.user })` `activation-system/controllers/admin/telegramBotController.js:141`<br>`res.status(500).send('Ошибка при загрузке формы')` `activation-system/controllers/admin/telegramBotController.js:149` | 404, 500; отказ доступа — см. middleware |
| POST | /admin/telegram-bots/:id/edit | `activation-system/routes/adminRoutes.js:170` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | `{ name, token, authorizedUsers, storeIds, isActive } = req.body` `activation-system/controllers/admin/telegramBotController.js:157` | telegramBotController.updateBot `activation-system/controllers/admin/telegramBotController.js:154` | `res.render('pages/admin-telegram-bot-form', { bot, stores, error: ˋНевалидный токен бота: ${validation.error}ˋ, user: req.session.user })` `activation-system/controllers/admin/telegramBotController.js:164`<br>`res.redirect('/admin/telegram-bots')` `activation-system/controllers/admin/telegramBotController.js:209`<br>`res.status(500).send('Ошибка при обновлении бота')` `activation-system/controllers/admin/telegramBotController.js:212` | 500; отказ доступа — см. middleware |
| POST | /admin/telegram-bots/:id/delete | `activation-system/routes/adminRoutes.js:171` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | telegramBotController.deleteBot `activation-system/controllers/admin/telegramBotController.js:217` | `res.redirect('/admin/telegram-bots')` `activation-system/controllers/admin/telegramBotController.js:226`<br>`res.status(500).send('Ошибка при удалении бота')` `activation-system/controllers/admin/telegramBotController.js:229` | 500; отказ доступа — см. middleware |
| POST | /admin/telegram-bots/:id/test | `activation-system/routes/adminRoutes.js:172` / mount `activation-system/index.js:178` | `ensureAdmin` `activation-system/middleware/auth.js:8` | id | Прямое чтение не найдено | Прямое чтение не найдено | telegramBotController.testBot `activation-system/controllers/admin/telegramBotController.js:234` | `res.json(result)` `activation-system/controllers/admin/telegramBotController.js:243`<br>`res.status(500).json({ success: false, error: error.message })` `activation-system/controllers/admin/telegramBotController.js:246` | 500; отказ доступа — см. middleware |
| GET | /admin/manual-activations | `activation-system/routes/adminRoutes.js:176` / mount `activation-system/index.js:178` | `allowSupport` `activation-system/middleware/auth.js:37` | — | `{ status, storeId } = req.query` `activation-system/controllers/admin/manualActivationController.js:13` | Прямое чтение не найдено | manualActivationController.listRequests `activation-system/controllers/admin/manualActivationController.js:11` | `res.render('pages/admin-manual-activations', { requests, stores, filters: { status, storeId }, user: req.session.user })` `activation-system/controllers/admin/manualActivationController.js:77`<br>`res.status(500).send('Ошибка при загрузке запросов')` `activation-system/controllers/admin/manualActivationController.js:85` | 500; отказ доступа — см. middleware |
| GET | /admin/manual-activations/:id | `activation-system/routes/adminRoutes.js:177` / mount `activation-system/index.js:178` | `allowSupport` `activation-system/middleware/auth.js:37` | id | Прямое чтение не найдено | Прямое чтение не найдено | manualActivationController.showRequestDetails `activation-system/controllers/admin/manualActivationController.js:90` | `res.status(404).send('Запрос не найден')` `activation-system/controllers/admin/manualActivationController.js:117`<br>`res.render('pages/admin-manual-activation-details', { request, user: req.session.user })` `activation-system/controllers/admin/manualActivationController.js:120`<br>`res.status(500).send('Ошибка при загрузке деталей')` `activation-system/controllers/admin/manualActivationController.js:126` | 404, 500; отказ доступа — см. middleware |
| POST | /admin/manual-activations/:id/complete | `activation-system/routes/adminRoutes.js:178` / mount `activation-system/index.js:178` | `allowSupport` `activation-system/middleware/auth.js:37` | id | Прямое чтение не найдено | `{ key } = req.body` `activation-system/controllers/admin/manualActivationController.js:134` | manualActivationController.completeRequest `activation-system/controllers/admin/manualActivationController.js:131` | `res.status(400).json({ success: false, error: 'Ключ активации не может быть пустым' })` `activation-system/controllers/admin/manualActivationController.js:138`<br>`res.status(404).json({ success: false, error: 'Запрос не найден' })` `activation-system/controllers/admin/manualActivationController.js:169`<br>`res.status(400).json({ success: false, error: 'Запрос уже обработан' })` `activation-system/controllers/admin/manualActivationController.js:176`<br>`res.json({ success: true, message: 'Активация успешно завершена', key: key.trim() })` `activation-system/controllers/admin/manualActivationController.js:265`<br>`res.status(500).json({ success: false, error: 'Ошибка при завершении активации' })` `activation-system/controllers/admin/manualActivationController.js:273` | 400, 404, 500; отказ доступа — см. middleware |
| POST | /admin/manual-activations/:id/reject | `activation-system/routes/adminRoutes.js:179` / mount `activation-system/index.js:178` | `allowSupport` `activation-system/middleware/auth.js:37` | id | Прямое чтение не найдено | `{ reason } = req.body` `activation-system/controllers/admin/manualActivationController.js:284` | manualActivationController.rejectRequest `activation-system/controllers/admin/manualActivationController.js:281` | `res.status(404).json({ success: false, error: 'Запрос не найден' })` `activation-system/controllers/admin/manualActivationController.js:295`<br>`res.status(400).json({ success: false, error: 'Запрос уже обработан' })` `activation-system/controllers/admin/manualActivationController.js:302`<br>`res.json({ success: true, message: 'Запрос отклонен' })` `activation-system/controllers/admin/manualActivationController.js:335`<br>`res.status(500).json({ success: false, error: 'Ошибка при отклонении запроса' })` `activation-system/controllers/admin/manualActivationController.js:342` | 404, 400, 500; отказ доступа — см. middleware |
| GET | /admin/qr-links | `activation-system/routes/adminRoutes.js:183` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | Прямое чтение не найдено | qrLinkController.showQrLinksPage `activation-system/controllers/admin/qrLinkController.js:6` | `res.render('pages/admin-qr-links', { merchants, products, user: req.session.user })` `activation-system/controllers/admin/qrLinkController.js:19`<br>`res.status(500).send('Ошибка загрузки страницы')` `activation-system/controllers/admin/qrLinkController.js:26` | 500; отказ доступа — см. middleware |
| POST | /admin/qr-links/generate | `activation-system/routes/adminRoutes.js:184` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | — | Прямое чтение не найдено | `{ merchantId, productId } = req.body` `activation-system/controllers/admin/qrLinkController.js:33` | qrLinkController.generateLink `activation-system/controllers/admin/qrLinkController.js:31` | `res.json({ success: true, link: { id: link.id, token: link.token, url: payUrl, qrCode: qrDataUrl, merchant: link.merchant.username, product: link.product.name, price: link.product.price } })` `activation-system/controllers/admin/qrLinkController.js:73`<br>`res.status(500).json({ success: false, error: 'Ошибка генерации' })` `activation-system/controllers/admin/qrLinkController.js:87` | 500; отказ доступа — см. middleware |
| GET | /admin/qr-links/:id/download | `activation-system/routes/adminRoutes.js:185` / mount `activation-system/index.js:178` | `allowContent` `activation-system/middleware/auth.js:30` | id | Прямое чтение не найдено | Прямое чтение не найдено | qrLinkController.downloadQr `activation-system/controllers/admin/qrLinkController.js:92` | `res.status(404).send('Ссылка не найдена')` `activation-system/controllers/admin/qrLinkController.js:99`<br>`res.setHeader('Content-Type', 'image/png')` `activation-system/controllers/admin/qrLinkController.js:110`<br>`res.setHeader('Content-Disposition', ˋattachment; filename=qr-${link.token}.pngˋ)` `activation-system/controllers/admin/qrLinkController.js:111`<br>`res.send(qrBuffer)` `activation-system/controllers/admin/qrLinkController.js:112`<br>`res.status(500).send('Ошибка скачивания')` `activation-system/controllers/admin/qrLinkController.js:115` | 404, 500; отказ доступа — см. middleware |
| POST | /api/v1/voucher/:id/events | `activation-system/routes/apiV1Routes.js:7` / mount `activation-system/index.js:183` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | id | Прямое чтение не найдено | `{ event, meta } = req.body &#124;&#124; {}` `activation-system/controllers/client/api/voucherController.js:280` | logEvent `activation-system/controllers/client/api/voucherController.js:273` | `res.status(401).json({ error: 'Unauthorized' })` `activation-system/controllers/client/api/voucherController.js:276`<br>`res.status(400).json({ error: 'Unsupported event' })` `activation-system/controllers/client/api/voucherController.js:283`<br>`res.status(404).json({ error: 'Voucher not found' })` `activation-system/controllers/client/api/voucherController.js:294`<br>`res.json({ ok: true })` `activation-system/controllers/client/api/voucherController.js:307`<br>`res.status(500).json({ error: 'Failed to log event' })` `activation-system/controllers/client/api/voucherController.js:310` | 401, 400, 404, 500; отказ доступа — см. middleware |
| GET | /auth/login | `activation-system/routes/authRoutes.js:8` / mount `activation-system/index.js:177` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.showLoginForm `activation-system/controllers/authController.js:16` | `res.render('pages/login', { error: null })` `activation-system/controllers/authController.js:17` | Явный HTTP status не задан |
| POST | /auth/login | `activation-system/routes/authRoutes.js:9` / mount `activation-system/index.js:177` | `loginLimiter` | — | Прямое чтение не найдено | `{ username, password } = req.body` `activation-system/controllers/authController.js:21`<br>`req.body.rememberMe` `activation-system/controllers/authController.js:22`<br>`req.body.loginVariant` `activation-system/controllers/authController.js:28` | authController.handleLogin `activation-system/controllers/authController.js:20` | `res.render(view, { error: 'Invalid credentials' })` `activation-system/controllers/authController.js:29`<br>`res.redirect('/admin/dashboard')` `activation-system/controllers/authController.js:55`<br>`res.redirect('/merchant/dashboard')` `activation-system/controllers/authController.js:57`<br>`res.redirect('/vendor/dashboard')` `activation-system/controllers/authController.js:59`<br>`res.redirect('/kassa/dashboard')` `activation-system/controllers/authController.js:61`<br>`res.status(403).send('Unknown role')` `activation-system/controllers/authController.js:63` | 403; 429 (limiter) |
| POST | /auth/refresh | `activation-system/routes/authRoutes.js:10` / mount `activation-system/index.js:177` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.refreshSession `activation-system/controllers/authController.js:109` | `res.status(401).json({ ok: false, message: 'Token expired' })` `activation-system/controllers/authController.js:119`<br>`res.status(401).json({ ok: false, message: 'User not found' })` `activation-system/controllers/authController.js:126`<br>`res.status(500).json({ ok: false })` `activation-system/controllers/authController.js:131`<br>`res.json({ ok: true, role: user.role })` `activation-system/controllers/authController.js:149` | 401, 500 |
| GET | /auth/logout | `activation-system/routes/authRoutes.js:12` / mount `activation-system/index.js:177` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.logout `activation-system/controllers/authController.js:69` | `res.redirect('/auth/login')` `activation-system/controllers/authController.js:83` | Явный HTTP status не задан |
| POST | /auth/logout | `activation-system/routes/authRoutes.js:13` / mount `activation-system/index.js:177` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.logout `activation-system/controllers/authController.js:69` | `res.redirect('/auth/login')` `activation-system/controllers/authController.js:83` | Явный HTTP status не задан |
| GET | /api/client/vouchers | `activation-system/routes/clientApiRoutes.js:7` / mount `activation-system/index.js:182` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | voucherApi.list `activation-system/controllers/client/api/voucherController.js:150` | `res.status(401).json({ error: 'Unauthorized' })` `activation-system/controllers/client/api/voucherController.js:153`<br>`res.json({ vouchers, syncedAt: new Date().toISOString() })` `activation-system/controllers/client/api/voucherController.js:180`<br>`res.status(500).json({ error: 'Failed to load vouchers' })` `activation-system/controllers/client/api/voucherController.js:183` | 401, 500; отказ доступа — см. middleware |
| GET | /api/client/voucher/:id | `activation-system/routes/clientApiRoutes.js:8` / mount `activation-system/index.js:182` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | id | Прямое чтение не найдено | Прямое чтение не найдено | voucherApi.show `activation-system/controllers/client/api/voucherController.js:187` | `res.status(401).json({ error: 'Unauthorized' })` `activation-system/controllers/client/api/voucherController.js:190`<br>`res.status(400).json({ error: 'Invalid voucher id' })` `activation-system/controllers/client/api/voucherController.js:195`<br>`res.status(404).json({ error: 'Voucher not found' })` `activation-system/controllers/client/api/voucherController.js:211`<br>`res.json(detail)` `activation-system/controllers/client/api/voucherController.js:216`<br>`res.status(500).json({ error: 'Failed to load voucher' })` `activation-system/controllers/client/api/voucherController.js:219` | 401, 400, 404, 500; отказ доступа — см. middleware |
| POST | /api/client/voucher/:id/request-code | `activation-system/routes/clientApiRoutes.js:9` / mount `activation-system/index.js:182` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | id | Прямое чтение не найдено | Прямое чтение не найдено | voucherApi.requestCode `activation-system/controllers/client/api/voucherController.js:224` | `res.status(401).json({ error: 'Unauthorized' })` `activation-system/controllers/client/api/voucherController.js:227`<br>`res.status(400).json({ error: 'Invalid voucher id' })` `activation-system/controllers/client/api/voucherController.js:232`<br>`res.json({ success: true, ...result })` `activation-system/controllers/client/api/voucherController.js:238`<br>`res.status(error.status).json({ success: false, error: error.message, code: error.code })` `activation-system/controllers/client/api/voucherController.js:241`<br>`res.status(500).json({ success: false, error: 'Не удалось получить код. Попробуйте ещё раз.' })` `activation-system/controllers/client/api/voucherController.js:244` | 401, 400, error.status, 500; отказ доступа — см. middleware |
| GET | /api/client/voucher/:id/code-status | `activation-system/routes/clientApiRoutes.js:10` / mount `activation-system/index.js:182` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | id | Прямое чтение не найдено | Прямое чтение не найдено | voucherApi.codeStatus `activation-system/controllers/client/api/voucherController.js:249` | `res.status(401).json({ error: 'Unauthorized' })` `activation-system/controllers/client/api/voucherController.js:252`<br>`res.status(400).json({ error: 'Invalid voucher id' })` `activation-system/controllers/client/api/voucherController.js:257`<br>`res.json({ success: true, ...result })` `activation-system/controllers/client/api/voucherController.js:263`<br>`res.status(error.status).json({ success: false, error: error.message, code: error.code })` `activation-system/controllers/client/api/voucherController.js:266`<br>`res.status(500).json({ success: false, error: 'Не удалось получить статус' })` `activation-system/controllers/client/api/voucherController.js:269` | 401, 400, error.status, 500; отказ доступа — см. middleware |
| POST | /api/client/voucher/:id/log | `activation-system/routes/clientApiRoutes.js:11` / mount `activation-system/index.js:182` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | id | Прямое чтение не найдено | `{ event, meta } = req.body &#124;&#124; {}` `activation-system/controllers/client/api/voucherController.js:280` | voucherApi.logEvent `activation-system/controllers/client/api/voucherController.js:273` | `res.status(401).json({ error: 'Unauthorized' })` `activation-system/controllers/client/api/voucherController.js:276`<br>`res.status(400).json({ error: 'Unsupported event' })` `activation-system/controllers/client/api/voucherController.js:283`<br>`res.status(404).json({ error: 'Voucher not found' })` `activation-system/controllers/client/api/voucherController.js:294`<br>`res.json({ ok: true })` `activation-system/controllers/client/api/voucherController.js:307`<br>`res.status(500).json({ error: 'Failed to log event' })` `activation-system/controllers/client/api/voucherController.js:310` | 401, 400, 404, 500; отказ доступа — см. middleware |
| POST | /api/client/push-subscription | `activation-system/routes/clientApiRoutes.js:12` / mount `activation-system/index.js:182` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | `req.body?.subscription` `activation-system/controllers/client/api/voucherController.js:320` | voucherApi.subscribePush `activation-system/controllers/client/api/voucherController.js:314` | `res.status(401).json({ error: 'Unauthorized' })` `activation-system/controllers/client/api/voucherController.js:317`<br>`res.status(400).json({ error: 'Invalid subscription payload' })` `activation-system/controllers/client/api/voucherController.js:322`<br>`res.json({ ok: true })` `activation-system/controllers/client/api/voucherController.js:346`<br>`res.status(500).json({ error: 'Failed to store subscription' })` `activation-system/controllers/client/api/voucherController.js:349` | 401, 400, 500; отказ доступа — см. middleware |
| POST | /api/client/voucher/claim | `activation-system/routes/clientApiRoutes.js:13` / mount `activation-system/index.js:182` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | `req.body?.payload` `activation-system/controllers/client/api/voucherController.js:368` | voucherApi.claim `activation-system/controllers/client/api/voucherController.js:361` | `res.status(401).json({ error: 'Unauthorized' })` `activation-system/controllers/client/api/voucherController.js:364`<br>`res.status(400).json({ error: 'Не удалось распознать QR код' })` `activation-system/controllers/client/api/voucherController.js:372`<br>`res.status(404).json({ error: 'Ваучер не найден' })` `activation-system/controllers/client/api/voucherController.js:390`<br>`res.status(409).json({ error: ˋВаучер находится в статусе «${voucher.status}» и не может быть добавленˋ })` `activation-system/controllers/client/api/voucherController.js:394`<br>`res.json(detail)` `activation-system/controllers/client/api/voucherController.js:455`<br>`res.status(error.status).json({ error: error.message })` `activation-system/controllers/client/api/voucherController.js:458`<br>`res.status(500).json({ error: 'Не удалось добавить ваучер в кошелёк' })` `activation-system/controllers/client/api/voucherController.js:461` | 401, 400, 404, 409, error.status, 500; отказ доступа — см. middleware |
| GET | /wallet/ | `activation-system/routes/clientRoutes.js:18` / mount `activation-system/index.js:190` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/clientRoutes.js:18` | `res.redirect('/client/dashboard')` `activation-system/routes/clientRoutes.js:22`<br>`res.redirect('/wallet')` `activation-system/routes/clientRoutes.js:22` | Явный HTTP status не задан |
| GET | /client/ | `activation-system/routes/clientRoutes.js:18` / mount `activation-system/index.js:191` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/clientRoutes.js:18` | `res.redirect('/client/dashboard')` `activation-system/routes/clientRoutes.js:22`<br>`res.redirect('/wallet')` `activation-system/routes/clientRoutes.js:22` | Явный HTTP status не задан |
| GET | /; только Host wallet.* | `activation-system/routes/clientRoutes.js:18` / mount `activation-system/index.js:196` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/clientRoutes.js:18` | `res.redirect('/client/dashboard')` `activation-system/routes/clientRoutes.js:22`<br>`res.redirect('/wallet')` `activation-system/routes/clientRoutes.js:22` | Явный HTTP status не задан |
| GET | /wallet/wallet | `activation-system/routes/clientRoutes.js:26` / mount `activation-system/index.js:190` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.showLogin `activation-system/controllers/client/authController.js:75` | `res.redirect('/client/dashboard')` `activation-system/controllers/client/authController.js:80`<br>`res.render('pages/client-login', { error: null })` `activation-system/controllers/client/authController.js:92` | Явный HTTP status не задан |
| GET | /client/wallet | `activation-system/routes/clientRoutes.js:26` / mount `activation-system/index.js:191` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.showLogin `activation-system/controllers/client/authController.js:75` | `res.redirect('/client/dashboard')` `activation-system/controllers/client/authController.js:80`<br>`res.render('pages/client-login', { error: null })` `activation-system/controllers/client/authController.js:92` | Явный HTTP status не задан |
| GET | /wallet; только Host wallet.* | `activation-system/routes/clientRoutes.js:26` / mount `activation-system/index.js:196` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.showLogin `activation-system/controllers/client/authController.js:75` | `res.redirect('/client/dashboard')` `activation-system/controllers/client/authController.js:80`<br>`res.render('pages/client-login', { error: null })` `activation-system/controllers/client/authController.js:92` | Явный HTTP status не задан |
| POST | /wallet/wallet | `activation-system/routes/clientRoutes.js:27` / mount `activation-system/index.js:190` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | `{ phoneNumber, phone } = req.body` `activation-system/controllers/client/authController.js:121` | authController.handleLogin `activation-system/controllers/client/authController.js:117` | `res.render('pages/client-login', { error: 'Введите номер телефона' })` `activation-system/controllers/client/authController.js:127`<br>`res.render('pages/client-login', { error: 'Аккаунт не найден. Пожалуйста, пройдите регистрацию.' })` `activation-system/controllers/client/authController.js:148`<br>`res.render('pages/client-login', { error: 'Не удалось сохранить сессию. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:160`<br>`res.redirect('/client-verify')` `activation-system/controllers/client/authController.js:162`<br>`res.render('pages/client-login', { error: 'Произошла ошибка. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:166` | Явный HTTP status не задан |
| POST | /client/wallet | `activation-system/routes/clientRoutes.js:27` / mount `activation-system/index.js:191` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | `{ phoneNumber, phone } = req.body` `activation-system/controllers/client/authController.js:121` | authController.handleLogin `activation-system/controllers/client/authController.js:117` | `res.render('pages/client-login', { error: 'Введите номер телефона' })` `activation-system/controllers/client/authController.js:127`<br>`res.render('pages/client-login', { error: 'Аккаунт не найден. Пожалуйста, пройдите регистрацию.' })` `activation-system/controllers/client/authController.js:148`<br>`res.render('pages/client-login', { error: 'Не удалось сохранить сессию. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:160`<br>`res.redirect('/client-verify')` `activation-system/controllers/client/authController.js:162`<br>`res.render('pages/client-login', { error: 'Произошла ошибка. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:166` | Явный HTTP status не задан |
| POST | /wallet; только Host wallet.* | `activation-system/routes/clientRoutes.js:27` / mount `activation-system/index.js:196` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | `{ phoneNumber, phone } = req.body` `activation-system/controllers/client/authController.js:121` | authController.handleLogin `activation-system/controllers/client/authController.js:117` | `res.render('pages/client-login', { error: 'Введите номер телефона' })` `activation-system/controllers/client/authController.js:127`<br>`res.render('pages/client-login', { error: 'Аккаунт не найден. Пожалуйста, пройдите регистрацию.' })` `activation-system/controllers/client/authController.js:148`<br>`res.render('pages/client-login', { error: 'Не удалось сохранить сессию. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:160`<br>`res.redirect('/client-verify')` `activation-system/controllers/client/authController.js:162`<br>`res.render('pages/client-login', { error: 'Произошла ошибка. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:166` | Явный HTTP status не задан |
| GET | /wallet/client-register | `activation-system/routes/clientRoutes.js:28` / mount `activation-system/index.js:190` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.showRegister `activation-system/controllers/client/authController.js:249` | `res.redirect('/client/dashboard')` `activation-system/controllers/client/authController.js:251`<br>`res.render('pages/client-register', { error: null })` `activation-system/controllers/client/authController.js:253` | Явный HTTP status не задан |
| GET | /client/client-register | `activation-system/routes/clientRoutes.js:28` / mount `activation-system/index.js:191` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.showRegister `activation-system/controllers/client/authController.js:249` | `res.redirect('/client/dashboard')` `activation-system/controllers/client/authController.js:251`<br>`res.render('pages/client-register', { error: null })` `activation-system/controllers/client/authController.js:253` | Явный HTTP status не задан |
| GET | /client-register; только Host wallet.* | `activation-system/routes/clientRoutes.js:28` / mount `activation-system/index.js:196` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.showRegister `activation-system/controllers/client/authController.js:249` | `res.redirect('/client/dashboard')` `activation-system/controllers/client/authController.js:251`<br>`res.render('pages/client-register', { error: null })` `activation-system/controllers/client/authController.js:253` | Явный HTTP status не задан |
| POST | /wallet/client-register | `activation-system/routes/clientRoutes.js:29` / mount `activation-system/index.js:190` | `registerLimiter` | — | Прямое чтение не найдено | `{ phoneNumber, phone, name } = req.body` `activation-system/controllers/client/authController.js:261` | authController.handleRegister `activation-system/controllers/client/authController.js:257` | `res.render('pages/client-register', { error: 'Заполните все поля' })` `activation-system/controllers/client/authController.js:266`<br>`res.render('pages/client-register', { error: 'Пользователь уже существует' })` `activation-system/controllers/client/authController.js:278`<br>`res.render('pages/client-register', { error: 'Не удалось сохранить сессию. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:289`<br>`res.redirect('/client-verify')` `activation-system/controllers/client/authController.js:291`<br>`res.render('pages/client-register', { error: 'Не удалось зарегистрировать пользователя. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:295` | Явный HTTP status не задан; 429 (limiter) |
| POST | /client/client-register | `activation-system/routes/clientRoutes.js:29` / mount `activation-system/index.js:191` | `registerLimiter` | — | Прямое чтение не найдено | `{ phoneNumber, phone, name } = req.body` `activation-system/controllers/client/authController.js:261` | authController.handleRegister `activation-system/controllers/client/authController.js:257` | `res.render('pages/client-register', { error: 'Заполните все поля' })` `activation-system/controllers/client/authController.js:266`<br>`res.render('pages/client-register', { error: 'Пользователь уже существует' })` `activation-system/controllers/client/authController.js:278`<br>`res.render('pages/client-register', { error: 'Не удалось сохранить сессию. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:289`<br>`res.redirect('/client-verify')` `activation-system/controllers/client/authController.js:291`<br>`res.render('pages/client-register', { error: 'Не удалось зарегистрировать пользователя. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:295` | Явный HTTP status не задан; 429 (limiter) |
| POST | /client-register; только Host wallet.* | `activation-system/routes/clientRoutes.js:29` / mount `activation-system/index.js:196` | `registerLimiter` | — | Прямое чтение не найдено | `{ phoneNumber, phone, name } = req.body` `activation-system/controllers/client/authController.js:261` | authController.handleRegister `activation-system/controllers/client/authController.js:257` | `res.render('pages/client-register', { error: 'Заполните все поля' })` `activation-system/controllers/client/authController.js:266`<br>`res.render('pages/client-register', { error: 'Пользователь уже существует' })` `activation-system/controllers/client/authController.js:278`<br>`res.render('pages/client-register', { error: 'Не удалось сохранить сессию. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:289`<br>`res.redirect('/client-verify')` `activation-system/controllers/client/authController.js:291`<br>`res.render('pages/client-register', { error: 'Не удалось зарегистрировать пользователя. Попробуйте позже.' })` `activation-system/controllers/client/authController.js:295` | Явный HTTP status не задан; 429 (limiter) |
| GET | /wallet/client-verify | `activation-system/routes/clientRoutes.js:30` / mount `activation-system/index.js:190` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.showOtpPage `activation-system/controllers/client/authController.js:171` | `res.redirect('/wallet')` `activation-system/controllers/client/authController.js:172`<br>`res.render('pages/client-verify', { phone: req.session.phone, error: null, })` `activation-system/controllers/client/authController.js:173` | Явный HTTP status не задан |
| GET | /client/client-verify | `activation-system/routes/clientRoutes.js:30` / mount `activation-system/index.js:191` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.showOtpPage `activation-system/controllers/client/authController.js:171` | `res.redirect('/wallet')` `activation-system/controllers/client/authController.js:172`<br>`res.render('pages/client-verify', { phone: req.session.phone, error: null, })` `activation-system/controllers/client/authController.js:173` | Явный HTTP status не задан |
| GET | /client-verify; только Host wallet.* | `activation-system/routes/clientRoutes.js:30` / mount `activation-system/index.js:196` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.showOtpPage `activation-system/controllers/client/authController.js:171` | `res.redirect('/wallet')` `activation-system/controllers/client/authController.js:172`<br>`res.render('pages/client-verify', { phone: req.session.phone, error: null, })` `activation-system/controllers/client/authController.js:173` | Явный HTTP status не задан |
| POST | /wallet/client-verify | `activation-system/routes/clientRoutes.js:31` / mount `activation-system/index.js:190` | `otpVerifyLimiter` | — | Прямое чтение не найдено | `{ otp } = req.body` `activation-system/controllers/client/authController.js:181` | authController.verifyOtp `activation-system/controllers/client/authController.js:180` | `res.render('pages/client-verify', { phone: req.session.phone, error: 'Код истёк. Пожалуйста, вернитесь назад и запросите новый код.' })` `activation-system/controllers/client/authController.js:188`<br>`res.render('pages/client-verify', { phone: req.session.phone, error: 'Неверный код' })` `activation-system/controllers/client/authController.js:195`<br>`res.redirect('/wallet')` `activation-system/controllers/client/authController.js:200`<br>`res.render('pages/client-verify', { phone: phoneForRender, error: 'Не удалось сохранить сессию. Попробуйте снова.', })` `activation-system/controllers/client/authController.js:239`<br>`res.redirect('/client/dashboard')` `activation-system/controllers/client/authController.js:244` | Явный HTTP status не задан; 429 (limiter) |
| POST | /client/client-verify | `activation-system/routes/clientRoutes.js:31` / mount `activation-system/index.js:191` | `otpVerifyLimiter` | — | Прямое чтение не найдено | `{ otp } = req.body` `activation-system/controllers/client/authController.js:181` | authController.verifyOtp `activation-system/controllers/client/authController.js:180` | `res.render('pages/client-verify', { phone: req.session.phone, error: 'Код истёк. Пожалуйста, вернитесь назад и запросите новый код.' })` `activation-system/controllers/client/authController.js:188`<br>`res.render('pages/client-verify', { phone: req.session.phone, error: 'Неверный код' })` `activation-system/controllers/client/authController.js:195`<br>`res.redirect('/wallet')` `activation-system/controllers/client/authController.js:200`<br>`res.render('pages/client-verify', { phone: phoneForRender, error: 'Не удалось сохранить сессию. Попробуйте снова.', })` `activation-system/controllers/client/authController.js:239`<br>`res.redirect('/client/dashboard')` `activation-system/controllers/client/authController.js:244` | Явный HTTP status не задан; 429 (limiter) |
| POST | /client-verify; только Host wallet.* | `activation-system/routes/clientRoutes.js:31` / mount `activation-system/index.js:196` | `otpVerifyLimiter` | — | Прямое чтение не найдено | `{ otp } = req.body` `activation-system/controllers/client/authController.js:181` | authController.verifyOtp `activation-system/controllers/client/authController.js:180` | `res.render('pages/client-verify', { phone: req.session.phone, error: 'Код истёк. Пожалуйста, вернитесь назад и запросите новый код.' })` `activation-system/controllers/client/authController.js:188`<br>`res.render('pages/client-verify', { phone: req.session.phone, error: 'Неверный код' })` `activation-system/controllers/client/authController.js:195`<br>`res.redirect('/wallet')` `activation-system/controllers/client/authController.js:200`<br>`res.render('pages/client-verify', { phone: phoneForRender, error: 'Не удалось сохранить сессию. Попробуйте снова.', })` `activation-system/controllers/client/authController.js:239`<br>`res.redirect('/client/dashboard')` `activation-system/controllers/client/authController.js:244` | Явный HTTP status не задан; 429 (limiter) |
| GET | /wallet/client/logout | `activation-system/routes/clientRoutes.js:32` / mount `activation-system/index.js:190` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.logout `activation-system/controllers/client/authController.js:300` | `res.redirect('/wallet')` `activation-system/controllers/client/authController.js:318` | Явный HTTP status не задан |
| GET | /client/client/logout | `activation-system/routes/clientRoutes.js:32` / mount `activation-system/index.js:191` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.logout `activation-system/controllers/client/authController.js:300` | `res.redirect('/wallet')` `activation-system/controllers/client/authController.js:318` | Явный HTTP status не задан |
| GET | /client/logout; только Host wallet.* | `activation-system/routes/clientRoutes.js:32` / mount `activation-system/index.js:196` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.logout `activation-system/controllers/client/authController.js:300` | `res.redirect('/wallet')` `activation-system/controllers/client/authController.js:318` | Явный HTTP status не задан |
| GET | /wallet/logout | `activation-system/routes/clientRoutes.js:33` / mount `activation-system/index.js:190` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.logout `activation-system/controllers/client/authController.js:300` | `res.redirect('/wallet')` `activation-system/controllers/client/authController.js:318` | Явный HTTP status не задан |
| GET | /client/logout | `activation-system/routes/clientRoutes.js:33` / mount `activation-system/index.js:191` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.logout `activation-system/controllers/client/authController.js:300` | `res.redirect('/wallet')` `activation-system/controllers/client/authController.js:318` | Явный HTTP status не задан |
| GET | /logout; только Host wallet.* | `activation-system/routes/clientRoutes.js:33` / mount `activation-system/index.js:196` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.logout `activation-system/controllers/client/authController.js:300` | `res.redirect('/wallet')` `activation-system/controllers/client/authController.js:318` | Явный HTTP status не задан |
| POST | /wallet/refresh | `activation-system/routes/clientRoutes.js:34` / mount `activation-system/index.js:190` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.refreshSession `activation-system/controllers/client/authController.js:395` | `res.status(401).json({ ok: false, message: 'Token expired' })` `activation-system/controllers/client/authController.js:405`<br>`res.status(401).json({ ok: false, message: 'Client not found' })` `activation-system/controllers/client/authController.js:412`<br>`res.status(500).json({ ok: false })` `activation-system/controllers/client/authController.js:417`<br>`res.json({ ok: true })` `activation-system/controllers/client/authController.js:429` | 401, 500 |
| POST | /client/refresh | `activation-system/routes/clientRoutes.js:34` / mount `activation-system/index.js:191` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.refreshSession `activation-system/controllers/client/authController.js:395` | `res.status(401).json({ ok: false, message: 'Token expired' })` `activation-system/controllers/client/authController.js:405`<br>`res.status(401).json({ ok: false, message: 'Client not found' })` `activation-system/controllers/client/authController.js:412`<br>`res.status(500).json({ ok: false })` `activation-system/controllers/client/authController.js:417`<br>`res.json({ ok: true })` `activation-system/controllers/client/authController.js:429` | 401, 500 |
| POST | /refresh; только Host wallet.* | `activation-system/routes/clientRoutes.js:34` / mount `activation-system/index.js:196` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.refreshSession `activation-system/controllers/client/authController.js:395` | `res.status(401).json({ ok: false, message: 'Token expired' })` `activation-system/controllers/client/authController.js:405`<br>`res.status(401).json({ ok: false, message: 'Client not found' })` `activation-system/controllers/client/authController.js:412`<br>`res.status(500).json({ ok: false })` `activation-system/controllers/client/authController.js:417`<br>`res.json({ ok: true })` `activation-system/controllers/client/authController.js:429` | 401, 500 |
| POST | /wallet/client/delete-account | `activation-system/routes/clientRoutes.js:35` / mount `activation-system/index.js:190` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.deleteAccount `activation-system/controllers/client/authController.js:326` | `res.status(401).json({ ok: false, message: 'Not authenticated' })` `activation-system/controllers/client/authController.js:331`<br>`res.status(404).json({ ok: false, message: 'Client not found' })` `activation-system/controllers/client/authController.js:337`<br>`res.status(500).json({ ok: false, message: 'Session destruction failed' })` `activation-system/controllers/client/authController.js:385`<br>`res.json({ ok: true, message: 'Account deleted successfully' })` `activation-system/controllers/client/authController.js:387`<br>`res.status(500).json({ ok: false, message: 'Failed to delete account' })` `activation-system/controllers/client/authController.js:391` | 401, 404, 500; отказ доступа — см. middleware |
| POST | /client/client/delete-account | `activation-system/routes/clientRoutes.js:35` / mount `activation-system/index.js:191` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.deleteAccount `activation-system/controllers/client/authController.js:326` | `res.status(401).json({ ok: false, message: 'Not authenticated' })` `activation-system/controllers/client/authController.js:331`<br>`res.status(404).json({ ok: false, message: 'Client not found' })` `activation-system/controllers/client/authController.js:337`<br>`res.status(500).json({ ok: false, message: 'Session destruction failed' })` `activation-system/controllers/client/authController.js:385`<br>`res.json({ ok: true, message: 'Account deleted successfully' })` `activation-system/controllers/client/authController.js:387`<br>`res.status(500).json({ ok: false, message: 'Failed to delete account' })` `activation-system/controllers/client/authController.js:391` | 401, 404, 500; отказ доступа — см. middleware |
| POST | /client/delete-account; только Host wallet.* | `activation-system/routes/clientRoutes.js:35` / mount `activation-system/index.js:196` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | authController.deleteAccount `activation-system/controllers/client/authController.js:326` | `res.status(401).json({ ok: false, message: 'Not authenticated' })` `activation-system/controllers/client/authController.js:331`<br>`res.status(404).json({ ok: false, message: 'Client not found' })` `activation-system/controllers/client/authController.js:337`<br>`res.status(500).json({ ok: false, message: 'Session destruction failed' })` `activation-system/controllers/client/authController.js:385`<br>`res.json({ ok: true, message: 'Account deleted successfully' })` `activation-system/controllers/client/authController.js:387`<br>`res.status(500).json({ ok: false, message: 'Failed to delete account' })` `activation-system/controllers/client/authController.js:391` | 401, 404, 500; отказ доступа — см. middleware |
| GET | /wallet/client/dashboard | `activation-system/routes/clientRoutes.js:38` / mount `activation-system/index.js:190` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | showDashboard `activation-system/controllers/client/dashboardController.js:28` | `res.redirect('/wallet')` `activation-system/controllers/client/dashboardController.js:31`<br>`res.render('pages/client-dashboard', { phone, vouchers, pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, isClient: true, })` `activation-system/controllers/client/dashboardController.js:77`<br>`res.status(500).render('pages/client-dashboard', { phone, vouchers: [], pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, isClient: true, })` `activation-system/controllers/client/dashboardController.js:86` | 500; отказ доступа — см. middleware |
| GET | /client/client/dashboard | `activation-system/routes/clientRoutes.js:38` / mount `activation-system/index.js:191` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | showDashboard `activation-system/controllers/client/dashboardController.js:28` | `res.redirect('/wallet')` `activation-system/controllers/client/dashboardController.js:31`<br>`res.render('pages/client-dashboard', { phone, vouchers, pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, isClient: true, })` `activation-system/controllers/client/dashboardController.js:77`<br>`res.status(500).render('pages/client-dashboard', { phone, vouchers: [], pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, isClient: true, })` `activation-system/controllers/client/dashboardController.js:86` | 500; отказ доступа — см. middleware |
| GET | /client/dashboard; только Host wallet.* | `activation-system/routes/clientRoutes.js:38` / mount `activation-system/index.js:196` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | showDashboard `activation-system/controllers/client/dashboardController.js:28` | `res.redirect('/wallet')` `activation-system/controllers/client/dashboardController.js:31`<br>`res.render('pages/client-dashboard', { phone, vouchers, pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, isClient: true, })` `activation-system/controllers/client/dashboardController.js:77`<br>`res.status(500).render('pages/client-dashboard', { phone, vouchers: [], pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, isClient: true, })` `activation-system/controllers/client/dashboardController.js:86` | 500; отказ доступа — см. middleware |
| GET | /wallet/client/profile | `activation-system/routes/clientRoutes.js:39` / mount `activation-system/index.js:190` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/clientRoutes.js:39` | `res.render('pages/client-profile', { phone: req.session.client.phone, pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, })` `activation-system/routes/clientRoutes.js:40` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /client/client/profile | `activation-system/routes/clientRoutes.js:39` / mount `activation-system/index.js:191` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/clientRoutes.js:39` | `res.render('pages/client-profile', { phone: req.session.client.phone, pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, })` `activation-system/routes/clientRoutes.js:40` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /client/profile; только Host wallet.* | `activation-system/routes/clientRoutes.js:39` / mount `activation-system/index.js:196` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/clientRoutes.js:39` | `res.render('pages/client-profile', { phone: req.session.client.phone, pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, })` `activation-system/routes/clientRoutes.js:40` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /wallet/client/qr-scanner | `activation-system/routes/clientRoutes.js:45` / mount `activation-system/index.js:190` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/clientRoutes.js:45` | `res.render('pages/client-qr-scanner', { phone: req.session.client.phone, pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, })` `activation-system/routes/clientRoutes.js:46` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /client/client/qr-scanner | `activation-system/routes/clientRoutes.js:45` / mount `activation-system/index.js:191` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/clientRoutes.js:45` | `res.render('pages/client-qr-scanner', { phone: req.session.client.phone, pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, })` `activation-system/routes/clientRoutes.js:46` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /client/qr-scanner; только Host wallet.* | `activation-system/routes/clientRoutes.js:45` / mount `activation-system/index.js:196` | `isClientAuthenticated` `activation-system/middleware/authClient.js:1` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/clientRoutes.js:45` | `res.render('pages/client-qr-scanner', { phone: req.session.client.phone, pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, })` `activation-system/routes/clientRoutes.js:46` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /kassa/dashboard | `activation-system/routes/kassaCabinetRoutes.js:11` / mount `activation-system/index.js:181` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`ensureKassa` `activation-system/middleware/auth.js:62` | — | Прямое чтение не найдено | Прямое чтение не найдено | kassaCabinetController.showDashboard `activation-system/controllers/kassa/kassaCabinetController.js:4` | `res.status(404).send('Касса не найдена')` `activation-system/controllers/kassa/kassaCabinetController.js:12`<br>`res.render('pages/kassa/dashboard', { layout: false, user: req.session.user, kassa, recentTransactions, weekSales: { count: weekSales._count?.id &#124;&#124; 0, total: Number(weekSales._sum?.price &#124;&#124; 0), commission: Number(weekSales._sum?.kassaDebt &#124;&#124; 0), }, })` `activation-system/controllers/kassa/kassaCabinetController.js:30`<br>`res.status(500).send('Ошибка загрузки дашборда')` `activation-system/controllers/kassa/kassaCabinetController.js:43` | 404, 500; отказ доступа — см. middleware |
| GET | /kassa/transactions | `activation-system/routes/kassaCabinetRoutes.js:14` / mount `activation-system/index.js:181` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`ensureKassa` `activation-system/middleware/auth.js:62` | — | `{ page = 1, vendor, status } = req.query` `activation-system/controllers/kassa/kassaCabinetController.js:51` | Прямое чтение не найдено | kassaCabinetController.showTransactions `activation-system/controllers/kassa/kassaCabinetController.js:48` | `res.render('pages/kassa/transactions', { layout: false, user: req.session.user, transactions, vendors, total, page: parseInt(page), totalPages: Math.ceil(total / limit), filters: { vendor, status }, })` `activation-system/controllers/kassa/kassaCabinetController.js:71`<br>`res.status(500).send('Ошибка загрузки транзакций')` `activation-system/controllers/kassa/kassaCabinetController.js:83` | 500; отказ доступа — см. middleware |
| GET | /kassa/obligations | `activation-system/routes/kassaCabinetRoutes.js:17` / mount `activation-system/index.js:181` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`ensureKassa` `activation-system/middleware/auth.js:62` | — | Прямое чтение не найдено | Прямое чтение не найдено | kassaCabinetController.showObligations `activation-system/controllers/kassa/kassaCabinetController.js:88` | `res.render('pages/kassa/obligations', { layout: false, user: req.session.user, vendorStats: vendorStats.map((vs) => ({ ...vs, vendorName: vendorMap[vs.vendorId]?.name &#124;&#124; ˋVendor #${vs.vendorId}ˋ, vendorBalance: vendorMap[vs.vendorId]?.balance &#124;&#124; 0, })), adjustments, })` `activation-system/controllers/kassa/kassaCabinetController.js:115`<br>`res.status(500).send('Ошибка загрузки обязательств')` `activation-system/controllers/kassa/kassaCabinetController.js:127` | 500; отказ доступа — см. middleware |
| GET | /kassa/payouts | `activation-system/routes/kassaCabinetRoutes.js:20` / mount `activation-system/index.js:181` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`ensureKassa` `activation-system/middleware/auth.js:62` | — | Прямое чтение не найдено | Прямое чтение не найдено | kassaCabinetController.showPayouts `activation-system/controllers/kassa/kassaCabinetController.js:132` | `res.render('pages/kassa/payouts', { layout: false, user: req.session.user, payments, vendors, kassaBalance: kassa?.balance &#124;&#124; 0, totalPaid: kassa?.totalPaid &#124;&#124; 0, })` `activation-system/controllers/kassa/kassaCabinetController.js:152`<br>`res.status(500).send('Ошибка загрузки выплат')` `activation-system/controllers/kassa/kassaCabinetController.js:162` | 500; отказ доступа — см. middleware |
| POST | /kassa/payouts | `activation-system/routes/kassaCabinetRoutes.js:21` / mount `activation-system/index.js:181` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`ensureKassa` `activation-system/middleware/auth.js:62`<br>`ensureKassaAdmin` `activation-system/middleware/auth.js:69` | — | Прямое чтение не найдено | `{ amount, recipientType, recipientId, comment } = req.body` `activation-system/controllers/kassa/kassaCabinetController.js:170` | kassaCabinetController.createPayout `activation-system/controllers/kassa/kassaCabinetController.js:167` | `res.status(400).send('Некорректная сумма')` `activation-system/controllers/kassa/kassaCabinetController.js:174`<br>`res.status(400).send('Выберите вендора для выплаты')` `activation-system/controllers/kassa/kassaCabinetController.js:179`<br>`res.status(404).send('Касса не найдена')` `activation-system/controllers/kassa/kassaCabinetController.js:183`<br>`res.redirect('/kassa/payouts')` `activation-system/controllers/kassa/kassaCabinetController.js:252`<br>`res.status(500).send('Ошибка создания выплаты')` `activation-system/controllers/kassa/kassaCabinetController.js:255` | 400, 404, 500; отказ доступа — см. middleware |
| GET | /kassa/settings | `activation-system/routes/kassaCabinetRoutes.js:24` / mount `activation-system/index.js:181` | `ensureAuthenticated` `activation-system/middleware/auth.js:2`<br>`ensureKassa` `activation-system/middleware/auth.js:62` | — | Прямое чтение не найдено | Прямое чтение не найдено | kassaCabinetController.showSettings `activation-system/controllers/kassa/kassaCabinetController.js:260` | `res.status(404).send('Касса не найдена')` `activation-system/controllers/kassa/kassaCabinetController.js:268`<br>`res.render('pages/kassa/settings', { layout: false, user: req.session.user, kassa, callbackUrls, })` `activation-system/controllers/kassa/kassaCabinetController.js:277`<br>`res.status(500).send('Ошибка загрузки настроек')` `activation-system/controllers/kassa/kassaCabinetController.js:285` | 404, 500; отказ доступа — см. middleware |
| GET | /merchant/login | `activation-system/routes/merchantRoutes.js:13` / mount `activation-system/index.js:179` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/merchantRoutes.js:13` | `res.render('pages/merchant-login', { error: null })` `activation-system/routes/merchantRoutes.js:13` | Явный HTTP status не задан |
| GET | /merchant/dashboard | `activation-system/routes/merchantRoutes.js:16` / mount `activation-system/index.js:179` | `ensureMerchant` `activation-system/middleware/auth.js:51` | — | Прямое чтение не найдено | Прямое чтение не найдено | dashboardController.showMerchantDashboard `activation-system/controllers/merchant/dashboardController.js:4` | `res.render('pages/merchant-sales', { sales: maskedSales, user: req.session.user &#124;&#124; null, merchantDebt, })` `activation-system/controllers/merchant/dashboardController.js:37` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /merchant/sales | `activation-system/routes/merchantRoutes.js:19` / mount `activation-system/index.js:179` | `ensureMerchant` `activation-system/middleware/auth.js:51` | — | Прямое чтение не найдено | Прямое чтение не найдено | saleController.showMerchantSales `activation-system/controllers/merchant/saleController.js:700` | `res.render('pages/merchant-sales', { sales: maskedSales, user, merchantDebt, })` `activation-system/controllers/merchant/saleController.js:730` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /merchant/checkout/confirm | `activation-system/routes/merchantRoutes.js:20` / mount `activation-system/index.js:179` | `ensureMerchant` `activation-system/middleware/auth.js:51` | — | Прямое чтение не найдено | `req.body.saleType` `activation-system/controllers/merchant/saleController.js:45`<br>`req.body.customerPhone` `activation-system/controllers/merchant/saleController.js:47`<br>`req.body.paymentConfirmed` `activation-system/controllers/merchant/saleController.js:56`<br>`req.body.checkoutToken` `activation-system/controllers/merchant/saleController.js:61` | saleController.confirmCheckout `activation-system/controllers/merchant/saleController.js:30` | `res.status(403).send('Мерчант не найден')` `activation-system/controllers/merchant/saleController.js:42`<br>`res.status(400).send('Корзина пуста')` `activation-system/controllers/merchant/saleController.js:52`<br>`res.status(400).send('Подтвердите, что оплата от клиента получена, прежде чем завершить продажу')` `activation-system/controllers/merchant/saleController.js:57`<br>`res.status(409).send('Эта продажа уже обработана либо срок чека истёк. Обновите страницу чека и попробуйте снова.')` `activation-system/controllers/merchant/saleController.js:63`<br>`res.status(400).send('Для онлайн продажи требуется корректный номер телефона в формате +998XXXXXXXXX')` `activation-system/controllers/merchant/saleController.js:72`<br>`res.status(400).send(ˋТовар с id ${cartItem.productId} недоступен для продажиˋ)` `activation-system/controllers/merchant/saleController.js:135`<br>`res.status(409).send(ˋНедостаточно активных ваучеров для товара "${product.name}".ˋ)` `activation-system/controllers/merchant/saleController.js:144`<br>`res.send(HTML-документ; динамические поля определены в исходнике)` `activation-system/controllers/merchant/saleController.js:308`<br>`res.send(HTML-документ; динамические поля определены в исходнике)` `activation-system/controllers/merchant/saleController.js:366`<br>`res.status(409).send(ˋВаучер для товара "${product.name}" только что был продан другой кассой. Обновите страницу чека и попробуйте снова.ˋ)` `activation-system/controllers/merchant/saleController.js:412`<br>`res.status(500).send('Ошибка при оформлении покупки: ' + error.message)` `activation-system/controllers/merchant/saleController.js:415` | 403, 400, 409, 500; отказ доступа — см. middleware |
| GET | /merchant/sell | `activation-system/routes/merchantRoutes.js:23` / mount `activation-system/index.js:179` | `ensureMerchant` `activation-system/middleware/auth.js:51` | — | Прямое чтение не найдено | Прямое чтение не найдено | cartController.showProductsForSale `activation-system/controllers/merchant/cartController.js:5` | `res.render('pages/merchant-sell', { products: enrichedProducts, cart: req.session.cart, user: req.session.user, })` `activation-system/controllers/merchant/cartController.js:35` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /merchant/add-to-cart | `activation-system/routes/merchantRoutes.js:24` / mount `activation-system/index.js:179` | `ensureMerchant` `activation-system/middleware/auth.js:51` | — | Прямое чтение не найдено | `req.body.productId` `activation-system/controllers/merchant/cartController.js:44` | cartController.addToCart `activation-system/controllers/merchant/cartController.js:43` | `res.redirect('/merchant/sell')` `activation-system/controllers/merchant/cartController.js:45`<br>`res.status(404).send('Товар не найден')` `activation-system/controllers/merchant/cartController.js:48`<br>`res.redirect('/merchant/checkout')` `activation-system/controllers/merchant/cartController.js:57` | 404; отказ доступа — см. middleware |
| GET | /merchant/checkout | `activation-system/routes/merchantRoutes.js:25` / mount `activation-system/index.js:179` | `ensureMerchant` `activation-system/middleware/auth.js:51` | — | Прямое чтение не найдено | Прямое чтение не найдено | cartController.showCart `activation-system/controllers/merchant/cartController.js:61` | `res.render('pages/checkout', { items: enrichedCart, totalPrice, totalQuantity, checkoutToken: req.session.checkoutToken, user: req.session.user, })` `activation-system/controllers/merchant/cartController.js:78` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /merchant/update-cart | `activation-system/routes/merchantRoutes.js:26` / mount `activation-system/index.js:179` | `ensureMerchant` `activation-system/middleware/auth.js:51` | — | Прямое чтение не найдено | Прямое чтение не найдено | cartController.updateCart `activation-system/controllers/merchant/cartController.js:88` | `res.redirect('/merchant/checkout')` `activation-system/controllers/merchant/cartController.js:89` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /merchant/cart/remove/:id | `activation-system/routes/merchantRoutes.js:27` / mount `activation-system/index.js:179` | `ensureMerchant` `activation-system/middleware/auth.js:51` | id | Прямое чтение не найдено | Прямое чтение не найдено | cartController.removeFromCart `activation-system/controllers/merchant/cartController.js:93` | `res.redirect('/merchant/checkout')` `activation-system/controllers/merchant/cartController.js:95` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /merchant/activate | `activation-system/routes/merchantRoutes.js:30` / mount `activation-system/index.js:179` | Нет middleware в роутере; глобальная цепочка действует | — | `req.query.voucher` `activation-system/controllers/publicController.js:120` | Прямое чтение не найдено | publicController.handleVoucherActivation `activation-system/controllers/publicController.js:119` | `res.send('❌ Введите код ваучера в ссылке')` `activation-system/controllers/publicController.js:123`<br>`res.render('pages/vendor/activate-qr-result', { user, result: { success: true, code: result.voucher.value, productName: result.productName, activationKey: result.activationKey, wasLinkedToClient: result.wasLinkedToClient, }, })` `activation-system/controllers/publicController.js:136`<br>`res.render('pages/vendor/activate-qr-result', { user, result: { success: false, message: error.message, }, })` `activation-system/controllers/publicController.js:148`<br>`res.render('pages/vendor/activate-qr-result', { user, result: { success: false, message: 'Не удалось активировать ваучер. Попробуйте ещё раз.', }, })` `activation-system/controllers/publicController.js:157`<br>`res.send('❌ Ваучер не найден')` `activation-system/controllers/publicController.js:172`<br>`res.render('pages/activate-form', { voucherCode: code })` `activation-system/controllers/publicController.js:176` | Явный HTTP status не задан |
| POST | /merchant/activate | `activation-system/routes/merchantRoutes.js:31` / mount `activation-system/index.js:179` | Нет middleware в роутере; глобальная цепочка действует | — | `req.query.voucher` `activation-system/controllers/publicController.js:120` | Прямое чтение не найдено | publicController.handleVoucherActivation `activation-system/controllers/publicController.js:119` | `res.send('❌ Введите код ваучера в ссылке')` `activation-system/controllers/publicController.js:123`<br>`res.render('pages/vendor/activate-qr-result', { user, result: { success: true, code: result.voucher.value, productName: result.productName, activationKey: result.activationKey, wasLinkedToClient: result.wasLinkedToClient, }, })` `activation-system/controllers/publicController.js:136`<br>`res.render('pages/vendor/activate-qr-result', { user, result: { success: false, message: error.message, }, })` `activation-system/controllers/publicController.js:148`<br>`res.render('pages/vendor/activate-qr-result', { user, result: { success: false, message: 'Не удалось активировать ваучер. Попробуйте ещё раз.', }, })` `activation-system/controllers/publicController.js:157`<br>`res.send('❌ Ваучер не найден')` `activation-system/controllers/publicController.js:172`<br>`res.render('pages/activate-form', { voucherCode: code })` `activation-system/controllers/publicController.js:176` | Явный HTTP status не задан |
| POST | /api/payments/click/:kassaId/prepare | `activation-system/routes/paymentRoutes.js:9` / mount `activation-system/index.js:207` | Нет middleware в роутере; глобальная цепочка действует | kassaId | Прямое чтение не найдено | Объект req.body целиком: click_trans_id, service_id, merchant_trans_id, merchant_prepare_id (complete), amount, sign_time, sign_string; error (complete). `activation-system/utils/payment/clickSignature.js:10` `activation-system/controllers/payment/clickController.js:9` | clickController.handlePrepare `activation-system/controllers/payment/clickController.js:8` | `res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'Kassa not specified' })` `activation-system/controllers/payment/clickController.js:17`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'Kassa not found' })` `activation-system/controllers/payment/clickController.js:21`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SIGN_CHECK_FAILED, error_note: 'SIGN CHECK FAILED!' })` `activation-system/controllers/payment/clickController.js:27`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.ORDER_NOT_FOUND, error_note: 'Order not found' })` `activation-system/controllers/payment/clickController.js:51`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.WRONG_AMOUNT, error_note: 'Wrong amount' })` `activation-system/controllers/payment/clickController.js:65`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, merchant_prepare_id: attempt.id, error: CLICK_ERRORS.ALREADY_PAID, error_note: 'Already paid' })` `activation-system/controllers/payment/clickController.js:76`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, merchant_prepare_id: attempt.id, error: CLICK_ERRORS.SUCCESS, error_note: 'Success' })` `activation-system/controllers/payment/clickController.js:97`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'System error' })` `activation-system/controllers/payment/clickController.js:107` | Явный HTTP status не задан |
| POST | /api/payments/click/:kassaId/complete | `activation-system/routes/paymentRoutes.js:10` / mount `activation-system/index.js:207` | Нет middleware в роутере; глобальная цепочка действует | kassaId | Прямое чтение не найдено | Объект req.body целиком: click_trans_id, service_id, merchant_trans_id, merchant_prepare_id (complete), amount, sign_time, sign_string; error (complete). `activation-system/utils/payment/clickSignature.js:10` `activation-system/controllers/payment/clickController.js:9` | clickController.handleComplete `activation-system/controllers/payment/clickController.js:119` | `res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'Kassa not specified' })` `activation-system/controllers/payment/clickController.js:128`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'Kassa not found' })` `activation-system/controllers/payment/clickController.js:132`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SIGN_CHECK_FAILED, error_note: 'SIGN CHECK FAILED!' })` `activation-system/controllers/payment/clickController.js:138`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.TRANSACTION_NOT_FOUND, error_note: 'Transaction not found' })` `activation-system/controllers/payment/clickController.js:162`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.TRANSACTION_NOT_FOUND, error_note: 'Invalid prepare_id' })` `activation-system/controllers/payment/clickController.js:173`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, merchant_confirm_id: attempt.id, error: CLICK_ERRORS.CANCELLED, error_note: 'Transaction cancelled' })` `activation-system/controllers/payment/clickController.js:189`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, merchant_confirm_id: attempt.id, error: CLICK_ERRORS.SUCCESS, error_note: 'Success' })` `activation-system/controllers/payment/clickController.js:201`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: result.error &#124;&#124; 'Payment processing failed' })` `activation-system/controllers/payment/clickController.js:216`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, merchant_confirm_id: attempt.id, error: CLICK_ERRORS.SUCCESS, error_note: 'Success' })` `activation-system/controllers/payment/clickController.js:226`<br>`res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'System error' })` `activation-system/controllers/payment/clickController.js:236` | Явный HTTP status не задан |
| POST | /api/payments/payme/:kassaId | `activation-system/routes/paymentRoutes.js:13` / mount `activation-system/index.js:207` | Нет middleware в роутере; глобальная цепочка действует | kassaId | Прямое чтение не найдено | `{ method, params, id } = req.body` `activation-system/controllers/payment/paymeController.js:9` | paymeController.handlePayme `activation-system/controllers/payment/paymeController.js:8` | `res.json({ result, id })` `activation-system/controllers/payment/paymeController.js:54`<br>`res.json({ error: { code: error.code, message: error.messageObj, data: error.data }, id })` `activation-system/controllers/payment/paymeController.js:60`<br>`res.json({ error: { code: PAYME_ERRORS.SYSTEM_ERROR.code, message: PAYME_ERRORS.SYSTEM_ERROR.message }, id })` `activation-system/controllers/payment/paymeController.js:70` | Явный HTTP status не задан |
| POST | /api/payments/alif/:kassaId/webhook | `activation-system/routes/paymentRoutes.js:17` / mount `activation-system/index.js:207` | Нет middleware в роутере; глобальная цепочка действует | kassaId | Прямое чтение не найдено | JSON: id; payment.status, payment.id; meta.order_id. Заголовок signature; точные байты req.rawBody. `activation-system/controllers/payment/alifController.js:13` `activation-system/controllers/payment/alifController.js:42` | alifController.handleWebhook `activation-system/controllers/payment/alifController.js:12` | `res.status(200).json({ status: 'kassa_not_found' })` `activation-system/controllers/payment/alifController.js:20`<br>`res.status(403).json({ status: 'invalid_signature' })` `activation-system/controllers/payment/alifController.js:35`<br>`res.status(200).json({ status: 'attempt_not_found' })` `activation-system/controllers/payment/alifController.js:68`<br>`res.status(200).json({ status: 'already_paid' })` `activation-system/controllers/payment/alifController.js:73`<br>`res.status(200).json({ status: 'not_succeeded', paymentStatus })` `activation-system/controllers/payment/alifController.js:83`<br>`res.status(200).json({ status: 'processing_failed', error: result.error })` `activation-system/controllers/payment/alifController.js:92`<br>`res.status(200).json({ status: 'ok' })` `activation-system/controllers/payment/alifController.js:96`<br>`res.status(200).json({ status: 'internal_error' })` `activation-system/controllers/payment/alifController.js:102` | 200, 403 |
| GET | / | `activation-system/routes/publicRoutes.js:7` / mount `activation-system/index.js:216` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/publicRoutes.js:7` | `res.redirect('/auth/login')` `activation-system/routes/publicRoutes.js:8` | Явный HTTP status не задан |
| GET | /email | `activation-system/routes/publicRoutes.js:12` / mount `activation-system/index.js:216` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | publicController.showEmailForm `activation-system/controllers/publicController.js:7` | `res.render('pages/enter-email', { error: null })` `activation-system/controllers/publicController.js:8` | Явный HTTP status не задан |
| POST | /email | `activation-system/routes/publicRoutes.js:13` / mount `activation-system/index.js:216` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | `{ email, phone } = req.body` `activation-system/controllers/publicController.js:13` | publicController.handleEmailForm `activation-system/controllers/publicController.js:12` | `res.render('pages/enter-email', { error: 'Введите email и номер телефона' })` `activation-system/controllers/publicController.js:16`<br>`res.redirect('/client/enter-voucher')` `activation-system/controllers/publicController.js:21` | Явный HTTP status не задан |
| GET | /client/enter-voucher | `activation-system/routes/publicRoutes.js:16` / mount `activation-system/index.js:216` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | Прямое чтение не найдено | publicController.showVoucherForm `activation-system/controllers/publicController.js:25` | `res.redirect('/email')` `activation-system/controllers/publicController.js:29`<br>`res.render('pages/enter-voucher', { email, phone, name: req.session.customer.name &#124;&#124; '', error: null })` `activation-system/controllers/publicController.js:32` | Явный HTTP status не задан |
| POST | /client/enter-voucher | `activation-system/routes/publicRoutes.js:17` / mount `activation-system/index.js:216` | Нет middleware в роутере; глобальная цепочка действует | — | Прямое чтение не найдено | `{ voucher } = req.body` `activation-system/controllers/publicController.js:42` | publicController.handleVoucherSubmit `activation-system/controllers/publicController.js:41` | `res.render('pages/enter-voucher', { email, phone, name, error: 'Все поля обязательны' })` `activation-system/controllers/publicController.js:46`<br>`res.render('pages/activation-key', { error: 'Неверный или использованный ваучер', key: null })` `activation-system/controllers/publicController.js:57`<br>`res.render('pages/activation-key', { key: activationKey, error: null })` `activation-system/controllers/publicController.js:92` | Явный HTTP status не задан |
| GET | /activate | `activation-system/routes/publicRoutes.js:20` / mount `activation-system/index.js:216` | Нет middleware в роутере; глобальная цепочка действует | — | `req.query.voucher` `activation-system/controllers/publicController.js:120` | Прямое чтение не найдено | handleVoucherActivation `activation-system/controllers/publicController.js:119` | `res.send('❌ Введите код ваучера в ссылке')` `activation-system/controllers/publicController.js:123`<br>`res.render('pages/vendor/activate-qr-result', { user, result: { success: true, code: result.voucher.value, productName: result.productName, activationKey: result.activationKey, wasLinkedToClient: result.wasLinkedToClient, }, })` `activation-system/controllers/publicController.js:136`<br>`res.render('pages/vendor/activate-qr-result', { user, result: { success: false, message: error.message, }, })` `activation-system/controllers/publicController.js:148`<br>`res.render('pages/vendor/activate-qr-result', { user, result: { success: false, message: 'Не удалось активировать ваучер. Попробуйте ещё раз.', }, })` `activation-system/controllers/publicController.js:157`<br>`res.send('❌ Ваучер не найден')` `activation-system/controllers/publicController.js:172`<br>`res.render('pages/activate-form', { voucherCode: code })` `activation-system/controllers/publicController.js:176` | Явный HTTP status не задан |
| GET | /pay/:token | `activation-system/routes/qrPaymentRoutes.js:7` / mount `activation-system/index.js:210` | Нет middleware в роутере; глобальная цепочка действует | token | Прямое чтение не найдено | Прямое чтение не найдено | qrPaymentController.showProductPage `activation-system/controllers/public/qrPaymentController.js:16` | `res.render('pages/pay-error', { error: 'link_invalid', message: 'Ссылка недействительна' })` `activation-system/controllers/public/qrPaymentController.js:34`<br>`res.render('pages/pay-error', { error: 'merchant_inactive', message: 'Продавец недоступен' })` `activation-system/controllers/public/qrPaymentController.js:41`<br>`res.render('pages/pay-error', { error: 'product_inactive', message: 'Товар снят с продажи' })` `activation-system/controllers/public/qrPaymentController.js:48`<br>`res.render('pages/pay-product', { link, merchant: link.merchant, product: link.product, hasVouchers: voucherCount > 0, oldPrice, discountPercent })` `activation-system/controllers/public/qrPaymentController.js:64`<br>`res.render('pages/pay-error', { error: 'server_error', message: 'Ошибка сервера' })` `activation-system/controllers/public/qrPaymentController.js:74` | Явный HTTP status не задан |
| GET | /pay/:token/checkout | `activation-system/routes/qrPaymentRoutes.js:8` / mount `activation-system/index.js:210` | Нет middleware в роутере; глобальная цепочка действует | token | Прямое чтение не найдено | Прямое чтение не найдено | qrPaymentController.showCheckoutPage `activation-system/controllers/public/qrPaymentController.js:82` | `res.redirect(ˋ/pay/${token}ˋ)` `activation-system/controllers/public/qrPaymentController.js:95`<br>`res.render('pages/pay-checkout', { link, merchant: link.merchant, product: link.product, token })` `activation-system/controllers/public/qrPaymentController.js:98`<br>`res.redirect(ˋ/pay/${req.params.token}ˋ)` `activation-system/controllers/public/qrPaymentController.js:106` | Явный HTTP status не задан |
| POST | /pay/:token/checkout | `activation-system/routes/qrPaymentRoutes.js:9` / mount `activation-system/index.js:210` | Нет middleware в роутере; глобальная цепочка действует | token | Прямое чтение не найдено | `{ phoneNumber, paymentMethod } = req.body` `activation-system/controllers/public/qrPaymentController.js:114` | qrPaymentController.processCheckout `activation-system/controllers/public/qrPaymentController.js:111` | `res.status(400).json({ success: false, error: 'Некорректный номер телефона' })` `activation-system/controllers/public/qrPaymentController.js:119`<br>`res.status(400).json({ success: false, error: 'Выберите способ оплаты' })` `activation-system/controllers/public/qrPaymentController.js:127`<br>`res.status(404).json({ success: false, error: 'Ссылка не найдена' })` `activation-system/controllers/public/qrPaymentController.js:142`<br>`res.status(400).json({ success: false, error: 'Alifpay не настроен для этой кассы' })` `activation-system/controllers/public/qrPaymentController.js:185`<br>`res.status(400).json({ success: false, error: 'Сумма вне допустимого диапазона Alifpay' })` `activation-system/controllers/public/qrPaymentController.js:194`<br>`res.status(502).json({ success: false, error: invoiceResult.error })` `activation-system/controllers/public/qrPaymentController.js:218`<br>`res.json({ success: true, redirectUrl: redirectUrl })` `activation-system/controllers/public/qrPaymentController.js:233`<br>`res.status(500).json({ success: false, error: 'Ошибка обработки' })` `activation-system/controllers/public/qrPaymentController.js:240` | 400, 404, 502, 500 |
| GET | /pay/:token/result/:attemptId | `activation-system/routes/qrPaymentRoutes.js:10` / mount `activation-system/index.js:210` | Нет middleware в роутере; глобальная цепочка действует | token,attemptId | Прямое чтение не найдено | Прямое чтение не найдено | qrPaymentController.showResultPage `activation-system/controllers/public/qrPaymentController.js:478` | `res.render('pages/pay-error', { error: 'not_found', message: 'Запись не найдена' })` `activation-system/controllers/public/qrPaymentController.js:496`<br>`res.render('pages/pay-error', { error: attempt.status.toLowerCase(), message: errorMessages[attempt.status] &#124;&#124; 'Ошибка', token })` `activation-system/controllers/public/qrPaymentController.js:510`<br>`res.render('pages/pay-result', { attempt, merchant: attempt.link.merchant, product: attempt.link.product, sale: attempt.sale, voucherCode: attempt.voucherValue, token })` `activation-system/controllers/public/qrPaymentController.js:517`<br>`res.render('pages/pay-error', { error: 'server_error', message: 'Ошибка сервера' })` `activation-system/controllers/public/qrPaymentController.js:527` | Явный HTTP status не задан |
| GET | /pay/:token/receipt/:attemptId | `activation-system/routes/qrPaymentRoutes.js:11` / mount `activation-system/index.js:210` | Нет middleware в роутере; глобальная цепочка действует | token,attemptId | Прямое чтение не найдено | Прямое чтение не найдено | qrPaymentController.downloadReceipt `activation-system/controllers/public/qrPaymentController.js:535` | `res.status(404).send('Чек недоступен')` `activation-system/controllers/public/qrPaymentController.js:547`<br>`res.status(404).send('Чек не найден')` `activation-system/controllers/public/qrPaymentController.js:554`<br>`res.status(404).send('Файл чека не найден')` `activation-system/controllers/public/qrPaymentController.js:560`<br>`res.setHeader('Content-Type', 'application/pdf')` `activation-system/controllers/public/qrPaymentController.js:564`<br>`res.setHeader('Content-Disposition', ˋattachment; filename=receipt-${attempt.id}.pdfˋ)` `activation-system/controllers/public/qrPaymentController.js:565`<br>`res.status(500).send('Ошибка загрузки чека')` `activation-system/controllers/public/qrPaymentController.js:572` | 404, 500 |
| POST | /:storeSlug/otp | `activation-system/routes/storeRoutes.js:7` / mount `activation-system/index.js:213` | Нет middleware в роутере; глобальная цепочка действует | storeSlug | Прямое чтение не найдено | `{ phone, storeSlug } = req.body` `activation-system/controllers/storeController.js:82` | sendOtp `activation-system/controllers/storeController.js:81` | `res.status(400).json({ error: 'Phone is required' })` `activation-system/controllers/storeController.js:86`<br>`res.status(500).json({ error: 'Failed to send SMS' })` `activation-system/controllers/storeController.js:125`<br>`res.json({ success: true, message: 'OTP sent' })` `activation-system/controllers/storeController.js:140`<br>`res.status(500).json({ error: 'Internal Server Error' })` `activation-system/controllers/storeController.js:144` | 400, 500 |
| POST | /:storeSlug/verify | `activation-system/routes/storeRoutes.js:8` / mount `activation-system/index.js:213` | Нет middleware в роутере; глобальная цепочка действует | storeSlug | Прямое чтение не найдено | `{ phone, code, storeSlug } = req.body` `activation-system/controllers/storeController.js:149` | verifyOtp `activation-system/controllers/storeController.js:148` | `res.status(400).json({ error: 'Invalid or expired code' })` `activation-system/controllers/storeController.js:183`<br>`res.json({ success: true, redirectUrl: ˋ/${storeSlug}ˋ })` `activation-system/controllers/storeController.js:205`<br>`res.status(500).json({ error: 'Internal Server Error' })` `activation-system/controllers/storeController.js:209` | 400, 500 |
| POST | /:storeSlug/activate | `activation-system/routes/storeRoutes.js:9` / mount `activation-system/index.js:213` | Нет middleware в роутере; глобальная цепочка действует | storeSlug | Прямое чтение не найдено | `{ voucherCode, storeSlug } = req.body` `activation-system/controllers/storeController.js:214` | activateVoucher `activation-system/controllers/storeController.js:213` | `res.status(401).json({ error: 'Unauthorized' })` `activation-system/controllers/storeController.js:217`<br>`res.status(404).json({ error: 'Store not found' })` `activation-system/controllers/storeController.js:221`<br>`res.status(429).json({ error: 'Произошла ошибка. Пожалуйста обратитесь в службу поддержки.' })` `activation-system/controllers/storeController.js:241`<br>`res.status(400).json({ error: 'Произошла ошибка. Пожалуйста обратитесь в службу поддержки.' })` `activation-system/controllers/storeController.js:336`<br>`res.json({ success: true, message: 'Ваша заявка принята. Ожидайте SMS с ключом активации в течение нескольких минут.', pending: true })` `activation-system/controllers/storeController.js:400`<br>`res.status(500).json({ error: 'Произошла ошибка. Пожалуйста обратитесь в службу поддержки.' })` `activation-system/controllers/storeController.js:412`<br>`res.json({ success: true, key: rokkyOrder.key })` `activation-system/controllers/storeController.js:480`<br>`res.json({ success: true, message: 'Activation in progress. Please check back later.' })` `activation-system/controllers/storeController.js:484`<br>`res.json({ success: true, key: drwebOrder.key })` `activation-system/controllers/storeController.js:549`<br>`res.json({ success: true, key: manastoreOrder.key })` `activation-system/controllers/storeController.js:671`<br>`res.json({ success: true, message: 'Activation in progress. Please check back later.', pending: true })` `activation-system/controllers/storeController.js:675`<br>`res.json({ success: true, message: 'Voucher activated successfully', voucher: true })` `activation-system/controllers/storeController.js:699`<br>`res.status(500).json({ error: 'Internal Server Error' })` `activation-system/controllers/storeController.js:713` | 401, 404, 429, 400, 500 |
| POST | /:storeSlug/logout | `activation-system/routes/storeRoutes.js:10` / mount `activation-system/index.js:213` | Нет middleware в роутере; глобальная цепочка действует | storeSlug | Прямое чтение не найдено | Прямое чтение не найдено | logout `activation-system/controllers/storeController.js:717` | `res.redirect(ˋ/${storeSlug}ˋ)` `activation-system/controllers/storeController.js:720` | Явный HTTP status не задан |
| GET | /:storeSlug/history | `activation-system/routes/storeRoutes.js:11` / mount `activation-system/index.js:213` | Нет middleware в роутере; глобальная цепочка действует | storeSlug | Прямое чтение не найдено | Прямое чтение не найдено | getHistoryStatus `activation-system/controllers/storeController.js:725` | `res.status(401).json({ error: 'Unauthorized' })` `activation-system/controllers/storeController.js:729`<br>`res.status(404).json({ error: 'Store not found' })` `activation-system/controllers/storeController.js:733`<br>`res.json({ success: true, history: result })` `activation-system/controllers/storeController.js:771`<br>`res.status(500).json({ error: 'Internal Server Error' })` `activation-system/controllers/storeController.js:774` | 401, 404, 500 |
| GET | /:storeSlug | `activation-system/routes/storeRoutes.js:15` / mount `activation-system/index.js:213` | Нет middleware в роутере; глобальная цепочка действует | storeSlug | Прямое чтение не найдено | Прямое чтение не найдено | getStorePage `activation-system/controllers/storeController.js:12` | `res.status(404).render('store/login', { store: null, error: 'Магазин не найден', layout: 'store/layout' })` `activation-system/controllers/storeController.js:22`<br>`res.render('store/activate', { store, client, history, layout: 'store/layout' })` `activation-system/controllers/storeController.js:57`<br>`res.render('store/login', { store, layout: 'store/layout' })` `activation-system/controllers/storeController.js:66`<br>`res.status(500).render('store/login', { store: null, error: 'Произошла ошибка', layout: 'store/layout' })` `activation-system/controllers/storeController.js:73` | 404, 500 |
| GET | /vendor/ | `activation-system/routes/vendorRoutes.js:13` / mount `activation-system/index.js:180` | `ensureVendor` `activation-system/middleware/auth.js:56` | — | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/vendorRoutes.js:13` | `res.redirect('/vendor/dashboard')` `activation-system/routes/vendorRoutes.js:13` | Явный HTTP status не задан; отказ доступа — см. middleware |
| GET | /vendor/dashboard | `activation-system/routes/vendorRoutes.js:14` / mount `activation-system/index.js:180` | `ensureVendor` `activation-system/middleware/auth.js:56` | — | Прямое чтение не найдено | Прямое чтение не найдено | showDashboard `activation-system/controllers/vendor/dashboardController.js:25` | `res.status(403).send('Не удалось определить вендора для сессии')` `activation-system/controllers/vendor/dashboardController.js:30`<br>`res.render('pages/vendor/dashboard', { user, vendor, metrics, activationTrend, recentActivations, pendingQueue: sanitizedQueue, trendRange: TREND_DAYS, })` `activation-system/controllers/vendor/dashboardController.js:157`<br>`res.render('pages/vendor/dashboard', { user, vendor: null, metrics: { balance: 0, pendingCount: 0, soldCount: 0, activationsLast7Days: 0, totalActivated: 0, }, activationTrend: [], recentActivations: [], pendingQueue: [], trendRange: TREND_DAYS, error: 'Не удалось загрузить данные дашборда', })` `activation-system/controllers/vendor/dashboardController.js:168` | 403; отказ доступа — см. middleware |
| GET | /vendor/activate | `activation-system/routes/vendorRoutes.js:15` / mount `activation-system/index.js:180` | `ensureVendor` `activation-system/middleware/auth.js:56` | — | Прямое чтение не найдено | Прямое чтение не найдено | showActivationForm `activation-system/controllers/vendor/activationController.js:14` | `res.render('pages/vendor/activate', buildViewModel({ user }))` `activation-system/controllers/vendor/activationController.js:16` | Явный HTTP status не задан; отказ доступа — см. middleware |
| POST | /vendor/activate | `activation-system/routes/vendorRoutes.js:16` / mount `activation-system/index.js:180` | `ensureVendor` `activation-system/middleware/auth.js:56` | — | Прямое чтение не найдено | `req.body?.code` `activation-system/controllers/vendor/activationController.js:27` | handleActivation `activation-system/controllers/vendor/activationController.js:19` | `res.status(403).send('Не удалось определить вендора для сессии')` `activation-system/controllers/vendor/activationController.js:24`<br>`res.render('pages/vendor/activate', buildViewModel({ user, form: { code }, error: 'Введите код ваучера', }))` `activation-system/controllers/vendor/activationController.js:30`<br>`res.render('pages/vendor/activate', buildViewModel({ user, success: successPayload, }))` `activation-system/controllers/vendor/activationController.js:50`<br>`res.render('pages/vendor/activate', buildViewModel({ user, form: { code }, error: error.message, }))` `activation-system/controllers/vendor/activationController.js:56`<br>`res.render('pages/vendor/activate', buildViewModel({ user, form: { code }, error: 'Не удалось активировать ваучер. Попробуйте ещё раз или обратитесь к администратору.', }))` `activation-system/controllers/vendor/activationController.js:63` | 403; отказ доступа — см. middleware |
| GET | /vendor/vouchers | `activation-system/routes/vendorRoutes.js:17` / mount `activation-system/index.js:180` | `ensureVendor` `activation-system/middleware/auth.js:56` | — | `{ status = '', productId = '', search = '', page: rawPage = '1' } = req.query` `activation-system/controllers/vendor/voucherController.js:44` | Прямое чтение не найдено | listVouchers `activation-system/controllers/vendor/voucherController.js:36` | `res.status(403).send('Не удалось определить вендора для сессии')` `activation-system/controllers/vendor/voucherController.js:41`<br>`res.render('pages/vendor/vouchers', { user, filters: { status, productId, search }, statusOptions: STATUS_OPTIONS, products, vouchers: sanitizedVouchers, pagination, })` `activation-system/controllers/vendor/voucherController.js:128`<br>`res.render('pages/vendor/vouchers', { user, filters: { status, productId, search }, statusOptions: STATUS_OPTIONS, products: [], vouchers: [], pagination: buildPagination(req, 0, page), error: 'Не удалось загрузить список ваучеров', })` `activation-system/controllers/vendor/voucherController.js:138` | 403; отказ доступа — см. middleware |
| GET | /vendor/transactions | `activation-system/routes/vendorRoutes.js:18` / mount `activation-system/index.js:180` | `ensureVendor` `activation-system/middleware/auth.js:56` | — | Прямое чтение не найдено | Прямое чтение не найдено | showTransactions `activation-system/controllers/vendor/transactionController.js:13` | `res.status(403).send('Не удалось определить вендора для сессии')` `activation-system/controllers/vendor/transactionController.js:18`<br>`res.render('pages/vendor/transactions', { user, transactions: maskedTransactions, payments, summary, })` `activation-system/controllers/vendor/transactionController.js:92`<br>`res.render('pages/vendor/transactions', { user, transactions: [], payments: [], summary: { pendingAmount: 0, totalAmount: 0, lastPayment: null, }, error: 'Не удалось загрузить финансовые данные', })` `activation-system/controllers/vendor/transactionController.js:100` | 403; отказ доступа — см. middleware |
| GET | /vendor/settings | `activation-system/routes/vendorRoutes.js:19` / mount `activation-system/index.js:180` | `ensureVendor` `activation-system/middleware/auth.js:56` | — | Прямое чтение не найдено | Прямое чтение не найдено | showSettings `activation-system/controllers/vendor/settingsController.js:4` | `res.status(403).send('Не удалось определить вендора для сессии')` `activation-system/controllers/vendor/settingsController.js:9`<br>`res.render('pages/vendor/settings', { user, vendor, users, previewHtml, })` `activation-system/controllers/vendor/settingsController.js:67`<br>`res.render('pages/vendor/settings', { user, vendor: { name: '—', category: null, productType: null, description: null, balance: 0, defaultCommissionPercent: 0, }, users: [], previewHtml: '', error: 'Не удалось загрузить настройки вендора', })` `activation-system/controllers/vendor/settingsController.js:75` | 403; отказ доступа — см. middleware |
| GET | /wallet/:serial.pkpass | `activation-system/routes/wallet.js:48` / mount `activation-system/index.js:187` | Нет middleware в роутере; глобальная цепочка действует | serial | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/wallet.js:48` | `res.status(404).send('Voucher not found')` `activation-system/routes/wallet.js:57`<br>`res.status(410).send('Voucher not available')` `activation-system/routes/wallet.js:60`<br>`res.setHeader('Content-Type', 'application/vnd.apple.pkpass')` `activation-system/routes/wallet.js:145`<br>`res.setHeader('Content-Disposition', ˋinline; filename="${serial}.pkpass"ˋ)` `activation-system/routes/wallet.js:146`<br>`res.setHeader('Cache-Control', 'no-store')` `activation-system/routes/wallet.js:147`<br>`res.send(buf)` `activation-system/routes/wallet.js:148`<br>`res.status(500).send('Pass generation error')` `activation-system/routes/wallet.js:151` | 404, 410, 500 |
| GET | /activate | `activation-system/routes/wallet.js:156` / mount `activation-system/index.js:187` | `activationLimiter` | — | `req.query.voucher` `activation-system/routes/wallet.js:157`<br>`req.query.code` `activation-system/routes/wallet.js:157`<br>`req.query.payload` `activation-system/routes/wallet.js:157` | Прямое чтение не найдено | inline `activation-system/routes/wallet.js:156` | `res.redirect(302, ˋ/wallet/${encodeURIComponent(code)}.pkpassˋ)` `activation-system/routes/wallet.js:164`<br>`res.redirect(302, ˋ${enterUrl}?serial=${encodeURIComponent(code)}ˋ)` `activation-system/routes/wallet.js:167` | Явный HTTP status не задан; 429 (limiter) |
| GET | /qr/:token | `activation-system/routes/wallet.js:171` / mount `activation-system/index.js:187` | Нет middleware в роутере; глобальная цепочка действует | token | Прямое чтение не найдено | Прямое чтение не найдено | inline `activation-system/routes/wallet.js:171` | `res.status(400).send('Bad or expired token')` `activation-system/routes/wallet.js:174`<br>`res.status(400).send('Token payload has no serial')` `activation-system/routes/wallet.js:177`<br>`res.redirect(302, ˋ/wallet/${encodeURIComponent(serial)}.pkpassˋ)` `activation-system/routes/wallet.js:182`<br>`res.redirect(302, ˋ${clientUrl}?serial=${encodeURIComponent(serial)}ˋ)` `activation-system/routes/wallet.js:185`<br>`res.status(400).send('Invalid QR token')` `activation-system/routes/wallet.js:187` | 400 |

## Статика

| Префикс | Назначение / доступ | Источник |
| --- | --- | --- |
| / | public; dotfiles: allow; до сессии | `activation-system/index.js:98` |
| /vendor/zxing | Каталог @zxing, только если существует | `activation-system/index.js:101` |
| /receipts | PDF без auth middleware | `activation-system/index.js:184` |

## Контракты обработчиков и вызываемых функций

Ниже перечислены чтения входа, условия и структуры ответов/возвращаемых значений. Условие приведено буквально: смысл ветки следует читать вместе с ответами. Это позволяет различать HTTP-ошибку и ошибку внутри JSON при HTTP 200.

### controllers/admin/clientController.js :: deleteClient

`activation-system/controllers/admin/clientController.js:103`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/clientController.js:105` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/clients?message=Клиент успешно удален') | `activation-system/controllers/admin/clientController.js:111` |
| res.redirect('/admin/clients?error=Ошибка при удалении клиента') | `activation-system/controllers/admin/clientController.js:114` |


### controllers/admin/clientController.js :: showAllClients

`activation-system/controllers/admin/clientController.js:4`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.query.page | `activation-system/controllers/admin/clientController.js:6` |
| req.query.limit | `activation-system/controllers/admin/clientController.js:7` |
| req.query.search | `activation-system/controllers/admin/clientController.js:9` |
| req.session.user | `activation-system/controllers/admin/clientController.js:54` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-clients', { user: req.session.user, clients, currentPage: page, totalPages, totalClients, search, limit, hasNextPage: page < totalPages, hasPrevPage: page > 1, nextPage: page + 1, prevPage: page - 1 }) | `activation-system/controllers/admin/clientController.js:53` |
| res.status(500).send('Ошибка при загрузке данных клиентов') | `activation-system/controllers/admin/clientController.js:68` |


### controllers/admin/clientController.js :: showClientDetails

`activation-system/controllers/admin/clientController.js:73`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/clientController.js:75` |
| req.session.user | `activation-system/controllers/admin/clientController.js:93` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !client | `activation-system/controllers/admin/clientController.js:88` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Клиент не найден') | `activation-system/controllers/admin/clientController.js:89` |
| res.render('pages/admin-client-details-simple', { user: req.session.user, client }) | `activation-system/controllers/admin/clientController.js:92` |
| res.status(500).send('Ошибка при загрузке данных клиента') | `activation-system/controllers/admin/clientController.js:98` |


### controllers/admin/dashboardController.js :: showAdminDashboard

`activation-system/controllers/admin/dashboardController.js:6`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/dashboardController.js:7` |
| req.session.user?.role | `activation-system/controllers/admin/dashboardController.js:7` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| role === 'kassa_admin' &#124;&#124; role === 'kassa_viewer' | `activation-system/controllers/admin/dashboardController.js:10` |
| role === 'financial_mgr' | `activation-system/controllers/admin/dashboardController.js:14` |
| role === 'content_mgr' | `activation-system/controllers/admin/dashboardController.js:17` |
| role === 'support_agent' | `activation-system/controllers/admin/dashboardController.js:20` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/kassa/dashboard') | `activation-system/controllers/admin/dashboardController.js:11` |


### controllers/admin/dashboardController.js :: showContentDashboard

`activation-system/controllers/admin/dashboardController.js:85`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/dashboardController.js:107` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-dashboard-content', { user: req.session.user, stats: { productsCount, storesCount, criticalProducts } }) | `activation-system/controllers/admin/dashboardController.js:106` |
| res.status(500).send('Content Dashboard Error') | `activation-system/controllers/admin/dashboardController.js:115` |


### controllers/admin/dashboardController.js :: showFinanceDashboard

`activation-system/controllers/admin/dashboardController.js:55`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/dashboardController.js:76` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-dashboard-finance', { user: req.session.user, stats }) | `activation-system/controllers/admin/dashboardController.js:75` |
| res.status(500).send('Finance Dashboard Error') | `activation-system/controllers/admin/dashboardController.js:80` |


### controllers/admin/dashboardController.js :: showSuperAdminDashboard

`activation-system/controllers/admin/dashboardController.js:29`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/dashboardController.js:38` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/dashboard-admin', { user: req.session.user, stats: { users: usersCount, vendors: merchantsCount, merchants: merchantsCount, sales: salesCount, vouchers: vouchersCount, clients: clientsCount } }) | `activation-system/controllers/admin/dashboardController.js:37` |
| res.status(500).send('Ошибка при загрузке данных дашборда') | `activation-system/controllers/admin/dashboardController.js:50` |


### controllers/admin/dashboardController.js :: showSupportDashboard

`activation-system/controllers/admin/dashboardController.js:120`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/dashboardController.js:133` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-dashboard-support', { user: req.session.user, stats: { pendingRequests, recentActivations } }) | `activation-system/controllers/admin/dashboardController.js:132` |
| res.status(500).send('Support Dashboard Error') | `activation-system/controllers/admin/dashboardController.js:140` |


### controllers/admin/drwebController.js :: handleAddSku

`activation-system/controllers/admin/drwebController.js:75`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { sku, name, description, category, costPrice, retailPrice } = req.body | `activation-system/controllers/admin/drwebController.js:76` |
| req.session.user | `activation-system/controllers/admin/drwebController.js:96` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/drweb/skus') | `activation-system/controllers/admin/drwebController.js:91` |
| res.render('pages/admin-drweb-add-sku', { error: 'Ошибка при создании SKU. Возможно, такой SKU уже существует.', user: req.session.user }) | `activation-system/controllers/admin/drwebController.js:94` |


### controllers/admin/drwebController.js :: handleBindSku

`activation-system/controllers/admin/drwebController.js:184`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/drwebController.js:185` |
| { drwebSku } = req.body | `activation-system/controllers/admin/drwebController.js:186` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/drweb/products') | `activation-system/controllers/admin/drwebController.js:196` |
| res.status(500).send('Ошибка при привязке SKU') | `activation-system/controllers/admin/drwebController.js:199` |


### controllers/admin/drwebController.js :: handleEditSku

`activation-system/controllers/admin/drwebController.js:120`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/drwebController.js:121` |
| { sku, name, description, category, costPrice, retailPrice, isActive } = req.body | `activation-system/controllers/admin/drwebController.js:122` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| parsedRetailPrice !== null | `activation-system/controllers/admin/drwebController.js:142` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/drweb/skus') | `activation-system/controllers/admin/drwebController.js:149` |
| res.status(500).send('Ошибка при обновлении SKU') | `activation-system/controllers/admin/drwebController.js:152` |


### controllers/admin/drwebController.js :: showAddSkuForm

`activation-system/controllers/admin/drwebController.js:67`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/drwebController.js:70` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-drweb-add-sku', { error: null, user: req.session.user }) | `activation-system/controllers/admin/drwebController.js:68` |


### controllers/admin/drwebController.js :: showDrWebActivations

`activation-system/controllers/admin/drwebController.js:204`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { startDate, endDate, productId, phone, storeSlug } = req.query | `activation-system/controllers/admin/drwebController.js:205` |
| req.session.user | `activation-system/controllers/admin/drwebController.js:262` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| startDate | `activation-system/controllers/admin/drwebController.js:217` |
| endDate | `activation-system/controllers/admin/drwebController.js:220` |
| productId | `activation-system/controllers/admin/drwebController.js:223` |
| phone | `activation-system/controllers/admin/drwebController.js:226` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-drweb-activations', { activations, products, filters: { startDate, endDate, productId, phone, storeSlug }, user: req.session.user }) | `activation-system/controllers/admin/drwebController.js:258` |


### controllers/admin/drwebController.js :: showDrWebDashboard

`activation-system/controllers/admin/drwebController.js:4`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/drwebController.js:50` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-drweb-dashboard', { products, skus, stats: { products: products.length, skus: skus.length, activations }, apiLogs, user: req.session.user }) | `activation-system/controllers/admin/drwebController.js:41` |


### controllers/admin/drwebController.js :: showDrWebFinance

`activation-system/controllers/admin/drwebController.js:267`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { startDate, endDate } = req.query | `activation-system/controllers/admin/drwebController.js:268` |
| req.session.user | `activation-system/controllers/admin/drwebController.js:311` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| startDate | `activation-system/controllers/admin/drwebController.js:278` |
| endDate | `activation-system/controllers/admin/drwebController.js:281` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-drweb-finance', { sales, stats: { totalRevenue, totalCost, totalMargin, salesCount: sales.length }, filters: { startDate, endDate }, user: req.session.user }) | `activation-system/controllers/admin/drwebController.js:302` |


### controllers/admin/drwebController.js :: showDrWebProducts

`activation-system/controllers/admin/drwebController.js:157`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/drwebController.js:179` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-drweb-products', { products, skus, user: req.session.user }) | `activation-system/controllers/admin/drwebController.js:176` |


### controllers/admin/drwebController.js :: showDrWebSkus

`activation-system/controllers/admin/drwebController.js:55`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/drwebController.js:62` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-drweb-skus', { skus, user: req.session.user }) | `activation-system/controllers/admin/drwebController.js:60` |


### controllers/admin/drwebController.js :: showEditSkuForm

`activation-system/controllers/admin/drwebController.js:102`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/drwebController.js:103` |
| req.session.user | `activation-system/controllers/admin/drwebController.js:115` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !sku | `activation-system/controllers/admin/drwebController.js:109` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('SKU не найден') | `activation-system/controllers/admin/drwebController.js:110` |
| res.render('pages/admin-drweb-edit-sku', { sku, user: req.session.user }) | `activation-system/controllers/admin/drwebController.js:113` |


### controllers/admin/kassaController.js :: handleCreateKassa

`activation-system/controllers/admin/kassaController.js:26`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { name, legalName, inn, clickMerchantId, clickServiceId, clickSecretKey, clickMerchantUserId, clickEnv, paymeMerchantId, paymeKey, paymeTestKey, paymeEnv, alifTokenProduction, alifTokenSandbox, alifSecretKeyProduction, alifSecretKeySandbox, alifEnv, contactPerson, contactPhone, contactEmail, vendorIds, } = req.body | `activation-system/controllers/admin/kassaController.js:28` |
| req.session.user | `activation-system/controllers/admin/kassaController.js:40` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !name | `activation-system/controllers/admin/kassaController.js:37` |
| vendorIds | `activation-system/controllers/admin/kassaController.js:70` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-kassa-add', { vendors, error: 'Укажите название кассы', user: req.session.user, }) | `activation-system/controllers/admin/kassaController.js:39` |
| res.redirect('/admin/kassas') | `activation-system/controllers/admin/kassaController.js:78` |
| res.render('pages/admin-kassa-add', { vendors, error: 'Ошибка при создании кассы', user: req.session.user, }) | `activation-system/controllers/admin/kassaController.js:82` |


### controllers/admin/kassaController.js :: handleUpdateKassa

`activation-system/controllers/admin/kassaController.js:106`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/kassaController.js:108` |
| { name, legalName, inn, isActive, clickMerchantId, clickServiceId, clickSecretKey, clickMerchantUserId, clickEnv, paymeMerchantId, paymeKey, paymeTestKey, paymeEnv, alifTokenProduction, alifTokenSandbox, alifSecretKeyProduction, alifSecretKeySandbox, alifEnv, contactPerson, contactPhone, contactEmail, vendorIds, } = req.body | `activation-system/controllers/admin/kassaController.js:109` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| vendorIds | `activation-system/controllers/admin/kassaController.js:149` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/kassas') | `activation-system/controllers/admin/kassaController.js:157` |
| res.redirect(ˋ/admin/kassas/${req.params.id}/editˋ) | `activation-system/controllers/admin/kassaController.js:160` |


### controllers/admin/kassaController.js :: showCreateKassaForm

`activation-system/controllers/admin/kassaController.js:17`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/kassaController.js:22` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-kassa-add', { vendors, error: null, user: req.session.user }) | `activation-system/controllers/admin/kassaController.js:22` |


### controllers/admin/kassaController.js :: showEditKassaForm

`activation-system/controllers/admin/kassaController.js:89`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/kassaController.js:91` |
| req.session.user | `activation-system/controllers/admin/kassaController.js:101` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !kassa | `activation-system/controllers/admin/kassaController.js:94` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Касса не найдена') | `activation-system/controllers/admin/kassaController.js:94` |
| res.render('pages/admin-kassa-edit', { kassa, availableVendors, error: null, user: req.session.user, }) | `activation-system/controllers/admin/kassaController.js:100` |


### controllers/admin/kassaController.js :: showKassaList

`activation-system/controllers/admin/kassaController.js:4`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/kassaController.js:13` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-kassa-list', { kassas, user: req.session.user }) | `activation-system/controllers/admin/kassaController.js:13` |


### controllers/admin/manastoreController.js :: getWalletBalanceSafe

`activation-system/controllers/admin/manastoreController.js:8`

### controllers/admin/manastoreController.js :: handleAddSku

`activation-system/controllers/admin/manastoreController.js:105`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { variantId, sku, name, description, category, costPrice, retailPrice, faceValue } = req.body | `activation-system/controllers/admin/manastoreController.js:106` |
| req.session.user | `activation-system/controllers/admin/manastoreController.js:128` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/manastore/skus') | `activation-system/controllers/admin/manastoreController.js:123` |
| res.render('pages/admin-manastore-add-sku', { error: 'Ошибка при создании SKU. Возможно, такой Variant ID уже существует.', user: req.session.user }) | `activation-system/controllers/admin/manastoreController.js:126` |


### controllers/admin/manastoreController.js :: handleBindSku

`activation-system/controllers/admin/manastoreController.js:223`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/manastoreController.js:224` |
| { variantId } = req.body | `activation-system/controllers/admin/manastoreController.js:225` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/manastore/products') | `activation-system/controllers/admin/manastoreController.js:235` |
| res.status(500).send('Ошибка при привязке варианта') | `activation-system/controllers/admin/manastoreController.js:238` |


### controllers/admin/manastoreController.js :: handleEditSku

`activation-system/controllers/admin/manastoreController.js:152`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/manastoreController.js:153` |
| { variantId, sku, name, description, category, costPrice, retailPrice, faceValue, isActive } = req.body | `activation-system/controllers/admin/manastoreController.js:154` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !existing | `activation-system/controllers/admin/manastoreController.js:158` |
| parsedRetailPrice !== null | `activation-system/controllers/admin/manastoreController.js:181` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('SKU не найден') | `activation-system/controllers/admin/manastoreController.js:159` |
| res.redirect('/admin/manastore/skus') | `activation-system/controllers/admin/manastoreController.js:188` |
| res.status(500).send('Ошибка при обновлении SKU') | `activation-system/controllers/admin/manastoreController.js:191` |


### controllers/admin/manastoreController.js :: handleSyncSkuCatalog

`activation-system/controllers/admin/manastoreController.js:380`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !variant?.id | `activation-system/controllers/admin/manastoreController.js:392` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/manastore/skus') | `activation-system/controllers/admin/manastoreController.js:428` |
| res.status(500).send('Ошибка синхронизации каталога ManaStore') | `activation-system/controllers/admin/manastoreController.js:431` |


### controllers/admin/manastoreController.js :: showAddSkuForm

`activation-system/controllers/admin/manastoreController.js:97`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/manastoreController.js:100` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-manastore-add-sku', { error: null, user: req.session.user }) | `activation-system/controllers/admin/manastoreController.js:98` |


### controllers/admin/manastoreController.js :: showEditSkuForm

`activation-system/controllers/admin/manastoreController.js:134`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/manastoreController.js:135` |
| req.session.user | `activation-system/controllers/admin/manastoreController.js:147` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !sku | `activation-system/controllers/admin/manastoreController.js:141` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('SKU не найден') | `activation-system/controllers/admin/manastoreController.js:142` |
| res.render('pages/admin-manastore-edit-sku', { sku, user: req.session.user }) | `activation-system/controllers/admin/manastoreController.js:145` |


### controllers/admin/manastoreController.js :: showManaStoreActivations

`activation-system/controllers/admin/manastoreController.js:243`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { startDate, endDate, productId, phone, storeSlug, status } = req.query | `activation-system/controllers/admin/manastoreController.js:244` |
| req.session.user | `activation-system/controllers/admin/manastoreController.js:313` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| startDate | `activation-system/controllers/admin/manastoreController.js:256` |
| endDate | `activation-system/controllers/admin/manastoreController.js:259` |
| productId | `activation-system/controllers/admin/manastoreController.js:262` |
| phone | `activation-system/controllers/admin/manastoreController.js:265` |
| storeSlug | `activation-system/controllers/admin/manastoreController.js:268` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-manastore-activations', { activations: filteredActivations, products, stores, filters: { startDate, endDate, productId, phone, storeSlug, status }, user: req.session.user }) | `activation-system/controllers/admin/manastoreController.js:308` |


### controllers/admin/manastoreController.js :: showManaStoreDashboard

`activation-system/controllers/admin/manastoreController.js:30`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/manastoreController.js:80` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-manastore-dashboard', { products, skus, stats: { products: products.length, skus: skus.length, activations }, apiLogs, environment: CURRENT_ENVIRONMENT, walletBalance, user: req.session.user }) | `activation-system/controllers/admin/manastoreController.js:69` |


### controllers/admin/manastoreController.js :: showManaStoreFinance

`activation-system/controllers/admin/manastoreController.js:318`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { startDate, endDate } = req.query | `activation-system/controllers/admin/manastoreController.js:319` |
| req.session.user | `activation-system/controllers/admin/manastoreController.js:375` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| startDate | `activation-system/controllers/admin/manastoreController.js:329` |
| endDate | `activation-system/controllers/admin/manastoreController.js:332` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-manastore-finance', { sales, stats: { totalRevenue, totalCost, totalMargin, salesCount: sales.length }, walletBalance, environment: CURRENT_ENVIRONMENT, filters: { startDate, endDate }, user: req.session.user }) | `activation-system/controllers/admin/manastoreController.js:364` |


### controllers/admin/manastoreController.js :: showManaStoreProducts

`activation-system/controllers/admin/manastoreController.js:196`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/manastoreController.js:218` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-manastore-products', { products, skus, user: req.session.user }) | `activation-system/controllers/admin/manastoreController.js:215` |


### controllers/admin/manastoreController.js :: showManaStoreSkus

`activation-system/controllers/admin/manastoreController.js:85`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/manastoreController.js:92` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-manastore-skus', { skus, user: req.session.user }) | `activation-system/controllers/admin/manastoreController.js:90` |


### controllers/admin/manastoreController.js :: toFloatFromPrice

`activation-system/controllers/admin/manastoreController.js:19`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| priceObj == null | `activation-system/controllers/admin/manastoreController.js:20` |
| typeof priceObj === 'number' | `activation-system/controllers/admin/manastoreController.js:21` |
| typeof priceObj === 'string' | `activation-system/controllers/admin/manastoreController.js:22` |
| raw == null | `activation-system/controllers/admin/manastoreController.js:24` |


### controllers/admin/manualActivationController.js :: completeRequest

`activation-system/controllers/admin/manualActivationController.js:131`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/manualActivationController.js:133` |
| { key } = req.body | `activation-system/controllers/admin/manualActivationController.js:134` |
| req.session.user | `activation-system/controllers/admin/manualActivationController.js:135` |
| req.session.user.id | `activation-system/controllers/admin/manualActivationController.js:135` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !key &#124;&#124; !key.trim() | `activation-system/controllers/admin/manualActivationController.js:137` |
| !request | `activation-system/controllers/admin/manualActivationController.js:168` |
| request.status !== 'PENDING' | `activation-system/controllers/admin/manualActivationController.js:175` |
| !voucher.product.walletOnDemandEnabled | `activation-system/controllers/admin/manualActivationController.js:219` |
| client | `activation-system/controllers/admin/manualActivationController.js:218` |
| smsResult.success | `activation-system/controllers/admin/manualActivationController.js:250` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(400).json({ success: false, error: 'Ключ активации не может быть пустым' }) | `activation-system/controllers/admin/manualActivationController.js:138` |
| res.status(404).json({ success: false, error: 'Запрос не найден' }) | `activation-system/controllers/admin/manualActivationController.js:169` |
| res.status(400).json({ success: false, error: 'Запрос уже обработан' }) | `activation-system/controllers/admin/manualActivationController.js:176` |
| res.json({ success: true, message: 'Активация успешно завершена', key: key.trim() }) | `activation-system/controllers/admin/manualActivationController.js:265` |
| res.status(500).json({ success: false, error: 'Ошибка при завершении активации' }) | `activation-system/controllers/admin/manualActivationController.js:273` |


### controllers/admin/manualActivationController.js :: listRequests

`activation-system/controllers/admin/manualActivationController.js:11`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { status, storeId } = req.query | `activation-system/controllers/admin/manualActivationController.js:13` |
| req.session.user | `activation-system/controllers/admin/manualActivationController.js:14` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| status | `activation-system/controllers/admin/manualActivationController.js:18` |
| user.role === 'vendor_user' && user.vendorId | `activation-system/controllers/admin/manualActivationController.js:23` |
| storeId | `activation-system/controllers/admin/manualActivationController.js:32` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-manual-activations', { requests, stores, filters: { status, storeId }, user: req.session.user }) | `activation-system/controllers/admin/manualActivationController.js:77` |
| res.status(500).send('Ошибка при загрузке запросов') | `activation-system/controllers/admin/manualActivationController.js:85` |


### controllers/admin/manualActivationController.js :: rejectRequest

`activation-system/controllers/admin/manualActivationController.js:281`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/manualActivationController.js:283` |
| { reason } = req.body | `activation-system/controllers/admin/manualActivationController.js:284` |
| req.session.user | `activation-system/controllers/admin/manualActivationController.js:285` |
| req.session.user.id | `activation-system/controllers/admin/manualActivationController.js:285` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !request | `activation-system/controllers/admin/manualActivationController.js:294` |
| request.status !== 'PENDING' | `activation-system/controllers/admin/manualActivationController.js:301` |
| !request.voucher.product?.walletOnDemandEnabled | `activation-system/controllers/admin/manualActivationController.js:328` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).json({ success: false, error: 'Запрос не найден' }) | `activation-system/controllers/admin/manualActivationController.js:295` |
| res.status(400).json({ success: false, error: 'Запрос уже обработан' }) | `activation-system/controllers/admin/manualActivationController.js:302` |
| res.json({ success: true, message: 'Запрос отклонен' }) | `activation-system/controllers/admin/manualActivationController.js:335` |
| res.status(500).json({ success: false, error: 'Ошибка при отклонении запроса' }) | `activation-system/controllers/admin/manualActivationController.js:342` |


### controllers/admin/manualActivationController.js :: showRequestDetails

`activation-system/controllers/admin/manualActivationController.js:90`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/manualActivationController.js:92` |
| req.session.user | `activation-system/controllers/admin/manualActivationController.js:122` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !request | `activation-system/controllers/admin/manualActivationController.js:116` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Запрос не найден') | `activation-system/controllers/admin/manualActivationController.js:117` |
| res.render('pages/admin-manual-activation-details', { request, user: req.session.user }) | `activation-system/controllers/admin/manualActivationController.js:120` |
| res.status(500).send('Ошибка при загрузке деталей') | `activation-system/controllers/admin/manualActivationController.js:126` |


### controllers/admin/merchantController.js :: handleAddMerchant

`activation-system/controllers/admin/merchantController.js:75`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.body.username | `activation-system/controllers/admin/merchantController.js:76` |
| req.body.username?.trim | `activation-system/controllers/admin/merchantController.js:76` |
| req.body.legalInfo | `activation-system/controllers/admin/merchantController.js:77` |
| req.body.legalInfo?.trim | `activation-system/controllers/admin/merchantController.js:77` |
| req.body.status | `activation-system/controllers/admin/merchantController.js:78` |
| req.body.password | `activation-system/controllers/admin/merchantController.js:79` |
| req.body.password?.trim | `activation-system/controllers/admin/merchantController.js:79` |
| req.session.user | `activation-system/controllers/admin/merchantController.js:91` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !normalizedUsername &#124;&#124; !password &#124;&#124; !normalizedLegalInfo | `activation-system/controllers/admin/merchantController.js:87` |
| error.code === 'P2002' | `activation-system/controllers/admin/merchantController.js:120` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(400).render('pages/admin-add-merchant', { error: 'Заполните логин, пароль и юридическую информацию', formData, user: req.session.user, }) | `activation-system/controllers/admin/merchantController.js:88` |
| res.redirect('/admin/merchants') | `activation-system/controllers/admin/merchantController.js:116` |
| res.status(400).render('pages/admin-add-merchant', { error: errorMessage, formData, user: req.session.user, }) | `activation-system/controllers/admin/merchantController.js:124` |


### controllers/admin/merchantController.js :: handleEditMerchant

`activation-system/controllers/admin/merchantController.js:152`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/merchantController.js:153` |
| req.body.username | `activation-system/controllers/admin/merchantController.js:160` |
| req.body.username?.trim | `activation-system/controllers/admin/merchantController.js:160` |
| req.body.legalInfo | `activation-system/controllers/admin/merchantController.js:161` |
| req.body.legalInfo?.trim | `activation-system/controllers/admin/merchantController.js:161` |
| req.body.status | `activation-system/controllers/admin/merchantController.js:162` |
| req.body.password | `activation-system/controllers/admin/merchantController.js:163` |
| req.body.password?.trim | `activation-system/controllers/admin/merchantController.js:163` |
| req.session.user | `activation-system/controllers/admin/merchantController.js:177` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !existing | `activation-system/controllers/admin/merchantController.js:156` |
| !normalizedUsername &#124;&#124; !normalizedLegalInfo | `activation-system/controllers/admin/merchantController.js:173` |
| normalizedUsername !== existing.username | `activation-system/controllers/admin/merchantController.js:192` |
| newPassword | `activation-system/controllers/admin/merchantController.js:199` |
| error.code === 'P2002' | `activation-system/controllers/admin/merchantController.js:212` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Merchant not found') | `activation-system/controllers/admin/merchantController.js:157` |
| res.status(400).render('pages/admin-edit-merchant', { merchant: merchantView, error: 'Логин и юридическая информация обязательны', user: req.session.user, }) | `activation-system/controllers/admin/merchantController.js:174` |
| res.redirect('/admin/merchants') | `activation-system/controllers/admin/merchantController.js:208` |
| res.status(400).render('pages/admin-edit-merchant', { merchant: merchantView, error: errorMessage, user: req.session.user, }) | `activation-system/controllers/admin/merchantController.js:216` |


### controllers/admin/merchantController.js :: handleMerchantPayment

`activation-system/controllers/admin/merchantController.js:245`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/merchantController.js:246` |
| { amount, comment } = req.body | `activation-system/controllers/admin/merchantController.js:247` |
| req.session.user | `activation-system/controllers/admin/merchantController.js:263` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !merchant | `activation-system/controllers/admin/merchantController.js:250` |
| isNaN(paymentAmount) &#124;&#124; paymentAmount <= 0 | `activation-system/controllers/admin/merchantController.js:253` |
| remaining <= 0 | `activation-system/controllers/admin/merchantController.js:295` |
| debt <= 0 | `activation-system/controllers/admin/merchantController.js:298` |
| remaining >= debt | `activation-system/controllers/admin/merchantController.js:306` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Merchant not found') | `activation-system/controllers/admin/merchantController.js:250` |
| res.status(400).render('pages/admin-pay-merchant', { merchant, actualDebt, user: req.session.user, error: 'Сумма платежа должна быть больше нуля', }) | `activation-system/controllers/admin/merchantController.js:260` |
| res.redirect('/admin/merchants') | `activation-system/controllers/admin/merchantController.js:326` |


### controllers/admin/merchantController.js :: showAddMerchantForm

`activation-system/controllers/admin/merchantController.js:63`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/merchantController.js:71` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-add-merchant', { error: null, formData: { username: '', legalInfo: '', status: 'active', }, user: req.session.user, }) | `activation-system/controllers/admin/merchantController.js:64` |


### controllers/admin/merchantController.js :: showEditMerchantForm

`activation-system/controllers/admin/merchantController.js:132`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/merchantController.js:133` |
| req.session.user | `activation-system/controllers/admin/merchantController.js:148` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !merchantRecord | `activation-system/controllers/admin/merchantController.js:136` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Merchant not found') | `activation-system/controllers/admin/merchantController.js:137` |
| res.render('pages/admin-edit-merchant', { merchant, error: null, user: req.session.user, }) | `activation-system/controllers/admin/merchantController.js:145` |


### controllers/admin/merchantController.js :: showMerchantPaymentHistory

`activation-system/controllers/admin/merchantController.js:350`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/merchantController.js:351` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !merchant | `activation-system/controllers/admin/merchantController.js:360` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Мерчант не найден') | `activation-system/controllers/admin/merchantController.js:361` |
| res.render('pages/admin-merchant-payments', { merchant, payments: merchant.payments, }) | `activation-system/controllers/admin/merchantController.js:364` |


### controllers/admin/merchantController.js :: showMerchantTransactions

`activation-system/controllers/admin/merchantController.js:330`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/merchantController.js:331` |
| req.session.user | `activation-system/controllers/admin/merchantController.js:345` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-merchant-transactions', { merchant, transactions, user: req.session.user, }) | `activation-system/controllers/admin/merchantController.js:342` |


### controllers/admin/merchantController.js :: showMerchantsWithDebt

`activation-system/controllers/admin/merchantController.js:5`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/merchantController.js:59` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-merchants', { merchants, stats, user: req.session.user, }) | `activation-system/controllers/admin/merchantController.js:56` |


### controllers/admin/merchantController.js :: showPaymentForm

`activation-system/controllers/admin/merchantController.js:225`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/merchantController.js:226` |
| req.session.user | `activation-system/controllers/admin/merchantController.js:240` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-pay-merchant', { merchant, actualDebt, error: null, user: req.session.user, }) | `activation-system/controllers/admin/merchantController.js:236` |


### controllers/admin/passwordController.js :: changeUserPassword

`activation-system/controllers/admin/passwordController.js:13`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/passwordController.js:14` |
| { password } = req.body | `activation-system/controllers/admin/passwordController.js:15` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !password | `activation-system/controllers/admin/passwordController.js:16` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.send('Пароль обязателен') | `activation-system/controllers/admin/passwordController.js:16` |
| res.redirect('/admin/users') | `activation-system/controllers/admin/passwordController.js:21` |


### controllers/admin/passwordController.js :: showChangePasswordForm

`activation-system/controllers/admin/passwordController.js:5`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/passwordController.js:6` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !userToEdit | `activation-system/controllers/admin/passwordController.js:8` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.send('Пользователь не найден') | `activation-system/controllers/admin/passwordController.js:8` |
| res.render('pages/admin-change-password', { userToEdit, error: null }) | `activation-system/controllers/admin/passwordController.js:9` |


### controllers/admin/productController.js :: handleAddProduct

`activation-system/controllers/admin/productController.js:53`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { name, price, status, vendorId, merchantCommissionPercent, vendorCommissionPercent, rokkySku, drwebSku, manastoreVariantId } = req.body | `activation-system/controllers/admin/productController.js:54` |
| req.session.user | `activation-system/controllers/admin/productController.js:76` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/products') | `activation-system/controllers/admin/productController.js:71` |
| res.render('pages/admin-add-product', { error: 'Ошибка при создании товара', vendors: await prisma.vendor.findMany({ orderBy: { name: 'asc' } }), user: req.session.user }) | `activation-system/controllers/admin/productController.js:73` |


### controllers/admin/productController.js :: handleDeleteProduct

`activation-system/controllers/admin/productController.js:179`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/productController.js:180` |
| req.session.user | `activation-system/controllers/admin/productController.js:209` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| error.code === 'P2003' | `activation-system/controllers/admin/productController.js:190` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/products') | `activation-system/controllers/admin/productController.js:183` |
| res.render('pages/admin-products', { products: enrichedProducts, user: req.session.user, error: errorMessage }) | `activation-system/controllers/admin/productController.js:207` |
| res.status(500).send(errorMessage) | `activation-system/controllers/admin/productController.js:214` |


### controllers/admin/productController.js :: handleEditProduct

`activation-system/controllers/admin/productController.js:113`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/productController.js:114` |
| { name, price, status, merchantCommissionPercent, vendorCommissionPercent, rokkySku, drwebSku, manastoreVariantId, useCustomTemplate, templateSchema, walletOnDemandEnabled, useCustomInstructions, redeemInstructions, } = req.body | `activation-system/controllers/admin/productController.js:115` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !existing | `activation-system/controllers/admin/productController.js:126` |
| useCustomTemplate === 'true' && templateSchema | `activation-system/controllers/admin/productController.js:140` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/products') | `activation-system/controllers/admin/productController.js:127` |


### controllers/admin/productController.js :: previewProductReceiptTemplate

`activation-system/controllers/admin/productController.js:220`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/productController.js:221` |
| req.body?.schema | `activation-system/controllers/admin/productController.js:231` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !product | `activation-system/controllers/admin/productController.js:227` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Товар не найден') | `activation-system/controllers/admin/productController.js:228` |
| res.send(html) | `activation-system/controllers/admin/productController.js:241` |
| res.status(500).send('<h1>Ошибка рендера чека</h1><p>Проверьте структуру шаблона.</p>') | `activation-system/controllers/admin/productController.js:244` |


### controllers/admin/productController.js :: showAddProductForm

`activation-system/controllers/admin/productController.js:47`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/productController.js:49` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-add-product', { vendors, user: req.session.user, error: null }) | `activation-system/controllers/admin/productController.js:49` |


### controllers/admin/productController.js :: showAllProducts

`activation-system/controllers/admin/productController.js:12`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/productController.js:41` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-products', { products: enrichedProducts, user: req.session.user }) | `activation-system/controllers/admin/productController.js:39` |


### controllers/admin/productController.js :: showEditProductForm

`activation-system/controllers/admin/productController.js:83`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/productController.js:85` |
| req.session.user | `activation-system/controllers/admin/productController.js:101` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !product | `activation-system/controllers/admin/productController.js:89` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Товар не найден') | `activation-system/controllers/admin/productController.js:90` |
| res.render('pages/edit-product', { product, vendor: product.vendor, user: req.session.user, templateSchema: JSON.stringify(schema), hasCustomTemplate: !!product.receiptTemplate, hasCustomInstructions: !!product.redeemInstructions, }) | `activation-system/controllers/admin/productController.js:98` |


### controllers/admin/productImageController.js :: deleteProductImage

`activation-system/controllers/admin/productImageController.js:100`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.imageId | `activation-system/controllers/admin/productImageController.js:101` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !image | `activation-system/controllers/admin/productImageController.js:104` |
| err && err.code !== 'ENOENT' | `activation-system/controllers/admin/productImageController.js:114` |
| nextImage | `activation-system/controllers/admin/productImageController.js:124` |
| isPrimary | `activation-system/controllers/admin/productImageController.js:119` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Изображение не найдено') | `activation-system/controllers/admin/productImageController.js:105` |
| res.redirect(ˋ/admin/product-images/${productId}ˋ) | `activation-system/controllers/admin/productImageController.js:132` |


### controllers/admin/productImageController.js :: setPrimaryImage

`activation-system/controllers/admin/productImageController.js:77`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.imageId | `activation-system/controllers/admin/productImageController.js:78` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !image | `activation-system/controllers/admin/productImageController.js:81` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Изображение не найдено') | `activation-system/controllers/admin/productImageController.js:82` |
| res.redirect(ˋ/admin/product-images/${image.productId}ˋ) | `activation-system/controllers/admin/productImageController.js:96` |


### controllers/admin/productImageController.js :: showManageProductImages

`activation-system/controllers/admin/productImageController.js:25`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.productId | `activation-system/controllers/admin/productImageController.js:26` |
| req.session.user | `activation-system/controllers/admin/productImageController.js:40` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !product | `activation-system/controllers/admin/productImageController.js:34` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Товар не найден') | `activation-system/controllers/admin/productImageController.js:35` |
| res.render('pages/admin-product-images-manage', { product, user: req.session.user, error: null, }) | `activation-system/controllers/admin/productImageController.js:38` |


### controllers/admin/productImageController.js :: showProductImagesList

`activation-system/controllers/admin/productImageController.js:6`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/productImageController.js:20` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-product-images', { products, user: req.session.user, }) | `activation-system/controllers/admin/productImageController.js:18` |


### controllers/admin/productImageController.js :: uploadProductImage

`activation-system/controllers/admin/productImageController.js:46`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.productId | `activation-system/controllers/admin/productImageController.js:47` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !product | `activation-system/controllers/admin/productImageController.js:50` |
| !req.file | `activation-system/controllers/admin/productImageController.js:54` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Товар не найден') | `activation-system/controllers/admin/productImageController.js:51` |
| res.redirect(ˋ/admin/product-images/${productId}ˋ) | `activation-system/controllers/admin/productImageController.js:55` |


### controllers/admin/qrLinkController.js :: downloadQr

`activation-system/controllers/admin/qrLinkController.js:92`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/qrLinkController.js:95` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !link | `activation-system/controllers/admin/qrLinkController.js:98` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Ссылка не найдена') | `activation-system/controllers/admin/qrLinkController.js:99` |
| res.setHeader('Content-Type', 'image/png') | `activation-system/controllers/admin/qrLinkController.js:110` |
| res.setHeader('Content-Disposition', ˋattachment; filename=qr-${link.token}.pngˋ) | `activation-system/controllers/admin/qrLinkController.js:111` |
| res.send(qrBuffer) | `activation-system/controllers/admin/qrLinkController.js:112` |
| res.status(500).send('Ошибка скачивания') | `activation-system/controllers/admin/qrLinkController.js:115` |


### controllers/admin/qrLinkController.js :: generateLink

`activation-system/controllers/admin/qrLinkController.js:31`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { merchantId, productId } = req.body | `activation-system/controllers/admin/qrLinkController.js:33` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !link | `activation-system/controllers/admin/qrLinkController.js:50` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.json({ success: true, link: { id: link.id, token: link.token, url: payUrl, qrCode: qrDataUrl, merchant: link.merchant.username, product: link.product.name, price: link.product.price } }) | `activation-system/controllers/admin/qrLinkController.js:73` |
| res.status(500).json({ success: false, error: 'Ошибка генерации' }) | `activation-system/controllers/admin/qrLinkController.js:87` |


### controllers/admin/qrLinkController.js :: showQrLinksPage

`activation-system/controllers/admin/qrLinkController.js:6`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/qrLinkController.js:22` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-qr-links', { merchants, products, user: req.session.user }) | `activation-system/controllers/admin/qrLinkController.js:19` |
| res.status(500).send('Ошибка загрузки страницы') | `activation-system/controllers/admin/qrLinkController.js:26` |


### controllers/admin/rokkyController.js :: handleAddSku

`activation-system/controllers/admin/rokkyController.js:75`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { sku, name, description, category, costPrice, retailPrice } = req.body | `activation-system/controllers/admin/rokkyController.js:76` |
| req.session.user | `activation-system/controllers/admin/rokkyController.js:96` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/rokky/skus') | `activation-system/controllers/admin/rokkyController.js:91` |
| res.render('pages/admin-rokky-add-sku', { error: 'Ошибка при создании SKU. Возможно, такой SKU уже существует.', user: req.session.user }) | `activation-system/controllers/admin/rokkyController.js:94` |


### controllers/admin/rokkyController.js :: handleBindSku

`activation-system/controllers/admin/rokkyController.js:184`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/rokkyController.js:185` |
| { rokkySku } = req.body | `activation-system/controllers/admin/rokkyController.js:186` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/rokky/products') | `activation-system/controllers/admin/rokkyController.js:196` |
| res.status(500).send('Ошибка при привязке SKU') | `activation-system/controllers/admin/rokkyController.js:199` |


### controllers/admin/rokkyController.js :: handleEditSku

`activation-system/controllers/admin/rokkyController.js:120`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/rokkyController.js:121` |
| { sku, name, description, category, costPrice, retailPrice, isActive } = req.body | `activation-system/controllers/admin/rokkyController.js:122` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| parsedRetailPrice !== null | `activation-system/controllers/admin/rokkyController.js:142` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/rokky/skus') | `activation-system/controllers/admin/rokkyController.js:149` |
| res.status(500).send('Ошибка при обновлении SKU') | `activation-system/controllers/admin/rokkyController.js:152` |


### controllers/admin/rokkyController.js :: showAddSkuForm

`activation-system/controllers/admin/rokkyController.js:67`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/rokkyController.js:70` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-rokky-add-sku', { error: null, user: req.session.user }) | `activation-system/controllers/admin/rokkyController.js:68` |


### controllers/admin/rokkyController.js :: showEditSkuForm

`activation-system/controllers/admin/rokkyController.js:102`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/rokkyController.js:103` |
| req.session.user | `activation-system/controllers/admin/rokkyController.js:115` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !sku | `activation-system/controllers/admin/rokkyController.js:109` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('SKU не найден') | `activation-system/controllers/admin/rokkyController.js:110` |
| res.render('pages/admin-rokky-edit-sku', { sku, user: req.session.user }) | `activation-system/controllers/admin/rokkyController.js:113` |


### controllers/admin/rokkyController.js :: showRokkyActivations

`activation-system/controllers/admin/rokkyController.js:204`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { startDate, endDate, productId, phone, storeSlug } = req.query | `activation-system/controllers/admin/rokkyController.js:205` |
| req.session.user | `activation-system/controllers/admin/rokkyController.js:262` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| startDate | `activation-system/controllers/admin/rokkyController.js:217` |
| endDate | `activation-system/controllers/admin/rokkyController.js:220` |
| productId | `activation-system/controllers/admin/rokkyController.js:223` |
| phone | `activation-system/controllers/admin/rokkyController.js:226` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-rokky-activations', { activations, products, filters: { startDate, endDate, productId, phone, storeSlug }, user: req.session.user }) | `activation-system/controllers/admin/rokkyController.js:258` |


### controllers/admin/rokkyController.js :: showRokkyDashboard

`activation-system/controllers/admin/rokkyController.js:4`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/rokkyController.js:50` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-rokky-dashboard', { products, skus, stats: { products: products.length, skus: skus.length, activations }, apiLogs, user: req.session.user }) | `activation-system/controllers/admin/rokkyController.js:41` |


### controllers/admin/rokkyController.js :: showRokkyFinance

`activation-system/controllers/admin/rokkyController.js:267`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { startDate, endDate } = req.query | `activation-system/controllers/admin/rokkyController.js:268` |
| req.session.user | `activation-system/controllers/admin/rokkyController.js:312` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| startDate | `activation-system/controllers/admin/rokkyController.js:278` |
| endDate | `activation-system/controllers/admin/rokkyController.js:281` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-rokky-finance', { sales, stats: { totalRevenue, totalCost, totalMargin, salesCount: sales.length }, filters: { startDate, endDate }, user: req.session.user }) | `activation-system/controllers/admin/rokkyController.js:303` |


### controllers/admin/rokkyController.js :: showRokkyProducts

`activation-system/controllers/admin/rokkyController.js:157`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/rokkyController.js:179` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-rokky-products', { products, skus, user: req.session.user }) | `activation-system/controllers/admin/rokkyController.js:176` |


### controllers/admin/rokkyController.js :: showRokkySkus

`activation-system/controllers/admin/rokkyController.js:55`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/rokkyController.js:62` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-rokky-skus', { skus, user: req.session.user }) | `activation-system/controllers/admin/rokkyController.js:60` |


### controllers/admin/saleController.js :: showSales

`activation-system/controllers/admin/saleController.js:4`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/saleController.js:61` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-sales', { sales, stats, user: req.session.user, }) | `activation-system/controllers/admin/saleController.js:58` |


### controllers/admin/storeController.js :: handleAddStore

`activation-system/controllers/admin/storeController.js:36`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { slug, name, isActive, productIds } = req.body | `activation-system/controllers/admin/storeController.js:37` |
| req.session.user | `activation-system/controllers/admin/storeController.js:65` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/stores') | `activation-system/controllers/admin/storeController.js:55` |
| res.render('pages/admin-add-store', { vendors, products, error: 'Ошибка при создании магазина', user: req.session.user }) | `activation-system/controllers/admin/storeController.js:61` |


### controllers/admin/storeController.js :: handleEditStore

`activation-system/controllers/admin/storeController.js:95`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/storeController.js:96` |
| { slug, name, isActive, productIds } = req.body | `activation-system/controllers/admin/storeController.js:97` |
| req.body.themeColor | `activation-system/controllers/admin/storeController.js:108` |
| req.body.backgroundColor | `activation-system/controllers/admin/storeController.js:109` |
| req.body.activationSmsTemplate | `activation-system/controllers/admin/storeController.js:110` |
| req.body.removeLogo | `activation-system/controllers/admin/storeController.js:119` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| req.body.removeLogo === 'true' | `activation-system/controllers/admin/storeController.js:119` |
| req.file | `activation-system/controllers/admin/storeController.js:117` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/stores') | `activation-system/controllers/admin/storeController.js:128` |
| res.status(500).send('Ошибка при обновлении магазина') | `activation-system/controllers/admin/storeController.js:131` |


### controllers/admin/storeController.js :: showAddStoreForm

`activation-system/controllers/admin/storeController.js:21`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/storeController.js:31` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-add-store', { vendors, products, error: null, user: req.session.user }) | `activation-system/controllers/admin/storeController.js:27` |


### controllers/admin/storeController.js :: showEditStoreForm

`activation-system/controllers/admin/storeController.js:71`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/storeController.js:72` |
| req.session.user | `activation-system/controllers/admin/storeController.js:90` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !store | `activation-system/controllers/admin/storeController.js:79` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Магазин не найден') | `activation-system/controllers/admin/storeController.js:80` |
| res.render('pages/admin-edit-store', { store, allProducts, user: req.session.user }) | `activation-system/controllers/admin/storeController.js:87` |


### controllers/admin/storeController.js :: showStores

`activation-system/controllers/admin/storeController.js:6`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/storeController.js:16` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-stores', { stores, user: req.session.user }) | `activation-system/controllers/admin/storeController.js:14` |


### controllers/admin/telegramBotController.js :: createBot

`activation-system/controllers/admin/telegramBotController.js:48`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { name, token, authorizedUsers, storeIds, isActive } = req.body | `activation-system/controllers/admin/telegramBotController.js:50` |
| req.session.user | `activation-system/controllers/admin/telegramBotController.js:60` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !validation.success | `activation-system/controllers/admin/telegramBotController.js:54` |
| authorizedUsers | `activation-system/controllers/admin/telegramBotController.js:66` |
| storeIds && storeIds.length > 0 | `activation-system/controllers/admin/telegramBotController.js:81` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-telegram-bot-form', { bot: null, stores, error: ˋНевалидный токен бота: ${validation.error}ˋ, user: req.session.user }) | `activation-system/controllers/admin/telegramBotController.js:56` |
| res.redirect('/admin/telegram-bots') | `activation-system/controllers/admin/telegramBotController.js:95` |
| res.render('pages/admin-telegram-bot-form', { bot: null, stores, error: 'Ошибка при создании бота', user: req.session.user }) | `activation-system/controllers/admin/telegramBotController.js:99` |


### controllers/admin/telegramBotController.js :: deleteBot

`activation-system/controllers/admin/telegramBotController.js:217`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/telegramBotController.js:219` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/telegram-bots') | `activation-system/controllers/admin/telegramBotController.js:226` |
| res.status(500).send('Ошибка при удалении бота') | `activation-system/controllers/admin/telegramBotController.js:229` |


### controllers/admin/telegramBotController.js :: listBots

`activation-system/controllers/admin/telegramBotController.js:5`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/telegramBotController.js:20` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-telegram-bots', { bots, user: req.session.user }) | `activation-system/controllers/admin/telegramBotController.js:18` |
| res.status(500).send('Ошибка при загрузке списка ботов') | `activation-system/controllers/admin/telegramBotController.js:24` |


### controllers/admin/telegramBotController.js :: showCreateForm

`activation-system/controllers/admin/telegramBotController.js:29`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/telegramBotController.js:39` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-telegram-bot-form', { bot: null, stores, error: null, user: req.session.user }) | `activation-system/controllers/admin/telegramBotController.js:35` |
| res.status(500).send('Ошибка при загрузке формы') | `activation-system/controllers/admin/telegramBotController.js:43` |


### controllers/admin/telegramBotController.js :: showEditForm

`activation-system/controllers/admin/telegramBotController.js:109`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/telegramBotController.js:111` |
| req.session.user | `activation-system/controllers/admin/telegramBotController.js:145` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !bot | `activation-system/controllers/admin/telegramBotController.js:124` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Бот не найден') | `activation-system/controllers/admin/telegramBotController.js:125` |
| res.render('pages/admin-telegram-bot-form', { bot: { ...bot, authorizedUsersString: usersString }, stores, error: null, user: req.session.user }) | `activation-system/controllers/admin/telegramBotController.js:141` |
| res.status(500).send('Ошибка при загрузке формы') | `activation-system/controllers/admin/telegramBotController.js:149` |


### controllers/admin/telegramBotController.js :: testBot

`activation-system/controllers/admin/telegramBotController.js:234`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/telegramBotController.js:236` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.json(result) | `activation-system/controllers/admin/telegramBotController.js:243` |
| res.status(500).json({ success: false, error: error.message }) | `activation-system/controllers/admin/telegramBotController.js:246` |


### controllers/admin/telegramBotController.js :: updateBot

`activation-system/controllers/admin/telegramBotController.js:154`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/telegramBotController.js:156` |
| { name, token, authorizedUsers, storeIds, isActive } = req.body | `activation-system/controllers/admin/telegramBotController.js:157` |
| req.session.user | `activation-system/controllers/admin/telegramBotController.js:168` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !validation.success | `activation-system/controllers/admin/telegramBotController.js:161` |
| authorizedUsers | `activation-system/controllers/admin/telegramBotController.js:174` |
| storeIds && storeIds.length > 0 | `activation-system/controllers/admin/telegramBotController.js:195` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-telegram-bot-form', { bot, stores, error: ˋНевалидный токен бота: ${validation.error}ˋ, user: req.session.user }) | `activation-system/controllers/admin/telegramBotController.js:164` |
| res.redirect('/admin/telegram-bots') | `activation-system/controllers/admin/telegramBotController.js:209` |
| res.status(500).send('Ошибка при обновлении бота') | `activation-system/controllers/admin/telegramBotController.js:212` |


### controllers/admin/userController.js :: deleteUser

`activation-system/controllers/admin/userController.js:46`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/userController.js:48` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/users') | `activation-system/controllers/admin/userController.js:49` |
| res.send('Ошибка при удалении') | `activation-system/controllers/admin/userController.js:51` |


### controllers/admin/userController.js :: handleCreateUser

`activation-system/controllers/admin/userController.js:11`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { username, password, role, kassaId } = req.body | `activation-system/controllers/admin/userController.js:12` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !username &#124;&#124; !password &#124;&#124; !role | `activation-system/controllers/admin/userController.js:15` |
| (role === 'kassa_admin' &#124;&#124; role === 'kassa_viewer') && !kassaId | `activation-system/controllers/admin/userController.js:18` |
| kassaId | `activation-system/controllers/admin/userController.js:25` |
| role === 'merchant' | `activation-system/controllers/admin/userController.js:29` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-add-user', { error: 'Заполните все поля', kassas }) | `activation-system/controllers/admin/userController.js:15` |
| res.render('pages/admin-add-user', { error: 'Выберите кассу для пользователя с кассовой ролью', kassas }) | `activation-system/controllers/admin/userController.js:19` |
| res.redirect('/admin/users') | `activation-system/controllers/admin/userController.js:33` |
| res.render('pages/admin-add-user', { error: 'Ошибка при создании пользователя', kassas }) | `activation-system/controllers/admin/userController.js:35` |


### controllers/admin/userController.js :: showCreateUserForm

`activation-system/controllers/admin/userController.js:5`

Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-add-user', { error: null, kassas }) | `activation-system/controllers/admin/userController.js:7` |


### controllers/admin/userController.js :: showUsersList

`activation-system/controllers/admin/userController.js:40`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/userController.js:42` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-users', { users, user: req.session.user }) | `activation-system/controllers/admin/userController.js:42` |


### controllers/admin/vendorController.js :: createVendorUser

`activation-system/controllers/admin/vendorController.js:192`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { vendorId, username, password, note } = req.body | `activation-system/controllers/admin/vendorController.js:193` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| existing | `activation-system/controllers/admin/vendorController.js:196` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.send('Пользователь с таким логином уже существует.') | `activation-system/controllers/admin/vendorController.js:197` |
| res.redirect('/admin/vendors') | `activation-system/controllers/admin/vendorController.js:212` |


### controllers/admin/vendorController.js :: handleAddVendor

`activation-system/controllers/admin/vendorController.js:70`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { name, category, productType, description, defaultCommissionPercent } = req.body | `activation-system/controllers/admin/vendorController.js:71` |
| req.session.user | `activation-system/controllers/admin/vendorController.js:88` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/vendors') | `activation-system/controllers/admin/vendorController.js:84` |
| res.render('pages/admin-add-vendor', { error: 'Ошибка при добавлении вендора', user: req.session.user }) | `activation-system/controllers/admin/vendorController.js:86` |


### controllers/admin/vendorController.js :: handleEditVendor

`activation-system/controllers/admin/vendorController.js:120`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { name, category, productType, description, templateSchema, defaultCommissionPercent, redeemInstructions, } = req.body | `activation-system/controllers/admin/vendorController.js:121` |
| req.params.id | `activation-system/controllers/admin/vendorController.js:131` |
| req.session.user | `activation-system/controllers/admin/vendorController.js:176` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !vendor | `activation-system/controllers/admin/vendorController.js:168` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/vendors') | `activation-system/controllers/admin/vendorController.js:154` |
| res.status(404).send('Вендор не найден') | `activation-system/controllers/admin/vendorController.js:169` |
| res.render('pages/admin-edit-vendor', { vendor: { ...vendor, name, category, productType, description, defaultCommissionPercent }, user: req.session.user, templateSchema: JSON.stringify(schema), sampleProduct: vendor.products?.[0] &#124;&#124; null, error: 'Ошибка при обновлении вендора. Проверьте введенные данные.' }) | `activation-system/controllers/admin/vendorController.js:174` |


### controllers/admin/vendorController.js :: handleVendorPayment

`activation-system/controllers/admin/vendorController.js:229`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/vendorController.js:230` |
| { amount, comment } = req.body | `activation-system/controllers/admin/vendorController.js:231` |
| req.session.user | `activation-system/controllers/admin/vendorController.js:241` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !vendor | `activation-system/controllers/admin/vendorController.js:234` |
| isNaN(paymentAmount) &#124;&#124; paymentAmount <= 0 | `activation-system/controllers/admin/vendorController.js:237` |
| remaining <= 0 | `activation-system/controllers/admin/vendorController.js:272` |
| payout <= 0 | `activation-system/controllers/admin/vendorController.js:274` |
| remaining >= payout | `activation-system/controllers/admin/vendorController.js:282` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Vendor not found') | `activation-system/controllers/admin/vendorController.js:234` |
| res.status(400).render('pages/admin-pay-vendor', { vendor, error: 'Сумма пополнения должна быть больше нуля', user: req.session.user }) | `activation-system/controllers/admin/vendorController.js:238` |
| res.redirect('/admin/vendors') | `activation-system/controllers/admin/vendorController.js:302` |


### controllers/admin/vendorController.js :: previewReceiptTemplate

`activation-system/controllers/admin/vendorController.js:323`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/vendorController.js:324` |
| req.body?.schema | `activation-system/controllers/admin/vendorController.js:339` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !vendor | `activation-system/controllers/admin/vendorController.js:335` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Вендор не найден') | `activation-system/controllers/admin/vendorController.js:336` |
| res.send(html) | `activation-system/controllers/admin/vendorController.js:354` |
| res.status(500).send('<h1>Ошибка рендера чека</h1><p>Проверьте структуру шаблона.</p>') | `activation-system/controllers/admin/vendorController.js:357` |


### controllers/admin/vendorController.js :: showAddVendorForm

`activation-system/controllers/admin/vendorController.js:65`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/vendorController.js:66` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-add-vendor', { error: null, user: req.session.user }) | `activation-system/controllers/admin/vendorController.js:66` |


### controllers/admin/vendorController.js :: showAddVendorUserForm

`activation-system/controllers/admin/vendorController.js:186`

Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-add-vendor-user', { vendors }) | `activation-system/controllers/admin/vendorController.js:188` |


### controllers/admin/vendorController.js :: showEditVendorForm

`activation-system/controllers/admin/vendorController.js:94`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/vendorController.js:96` |
| req.session.user | `activation-system/controllers/admin/vendorController.js:113` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !vendor | `activation-system/controllers/admin/vendorController.js:105` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Вендор не найден') | `activation-system/controllers/admin/vendorController.js:106` |
| res.render('pages/admin-edit-vendor', { vendor, user: req.session.user, templateSchema: JSON.stringify(schema), sampleProduct: vendor.products?.[0] &#124;&#124; null, }) | `activation-system/controllers/admin/vendorController.js:111` |


### controllers/admin/vendorController.js :: showVendorPaymentForm

`activation-system/controllers/admin/vendorController.js:216`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/vendorController.js:217` |
| req.session.user | `activation-system/controllers/admin/vendorController.js:223` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-pay-vendor', { vendor, error: null, user: req.session.user }) | `activation-system/controllers/admin/vendorController.js:220` |


### controllers/admin/vendorController.js :: showVendorTransactions

`activation-system/controllers/admin/vendorController.js:306`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.id | `activation-system/controllers/admin/vendorController.js:307` |
| req.session.user | `activation-system/controllers/admin/vendorController.js:319` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-vendor-transactions', { vendor, transactions, user: req.session.user }) | `activation-system/controllers/admin/vendorController.js:316` |


### controllers/admin/vendorController.js :: showVendors

`activation-system/controllers/admin/vendorController.js:13`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/vendorController.js:59` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-vendors', { vendors, stats, user: req.session.user }) | `activation-system/controllers/admin/vendorController.js:56` |


### controllers/admin/voucherController.js :: addVouchers

`activation-system/controllers/admin/voucherController.js:43`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { productId, voucherList, type } = req.body | `activation-system/controllers/admin/voucherController.js:44` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !product | `activation-system/controllers/admin/voucherController.js:47` |
| uniqueValues.length === 0 | `activation-system/controllers/admin/voucherController.js:61` |
| uniqueValues.length > MAX_VOUCHER_LINES | `activation-system/controllers/admin/voucherController.js:64` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(400).send('Товар не найден') | `activation-system/controllers/admin/voucherController.js:48` |
| res.status(400).send('Не найдено ни одного кода — вставьте текст или загрузите файл') | `activation-system/controllers/admin/voucherController.js:62` |
| res.status(400).send(ˋСлишком много строк (${uniqueValues.length}). Максимум ${MAX_VOUCHER_LINES} за раз.ˋ) | `activation-system/controllers/admin/voucherController.js:65` |
| res.send(ˋ <html><head><meta charset="UTF-8"><script src="https://cdn.tailwindcss.com"></script></head> <body class="bg-slate-50 min-h-screen flex items-center justify-center"> <div class="bg-white rounded-lg shadow-lg p-8 max-w-md w-full text-center"> <h2 class="text-xl font-bold text-slate-900 mb-2">Готово</h2> <p class="text-slate-600">Добавлено новых кодов: <strong>${result.count}</strong></p> ${skipped > 0 ? ˋ<p class="text-amber-600 text-sm mt-1">Пропущено как дубликаты: ${skipped}</p>ˋ : ''} <a href="/admin/vouchers" class="mt-6 inline-block bg-indigo-600 text-white px-4 py-2 rounded-lg">К списку ваучеров</a> </div> </body></html> ˋ) | `activation-system/controllers/admin/voucherController.js:79` |


### controllers/admin/voucherController.js :: generateCode

`activation-system/controllers/admin/voucherController.js:102`

### controllers/admin/voucherController.js :: generateVouchers

`activation-system/controllers/admin/voucherController.js:93`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { productId, type, series, count } = req.body | `activation-system/controllers/admin/voucherController.js:94` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| createdCodes.has(fullCode) | `activation-system/controllers/admin/voucherController.js:113` |
| !exists | `activation-system/controllers/admin/voucherController.js:116` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/vouchers') | `activation-system/controllers/admin/voucherController.js:132` |


### controllers/admin/voucherController.js :: handleAddVoucher

`activation-system/controllers/admin/voucherController.js:16`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { value, productId, type } = req.body | `activation-system/controllers/admin/voucherController.js:17` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/admin/vouchers') | `activation-system/controllers/admin/voucherController.js:30` |


### controllers/admin/voucherController.js :: showAddVoucherForm

`activation-system/controllers/admin/voucherController.js:10`

Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-add-voucher', { products }) | `activation-system/controllers/admin/voucherController.js:12` |


### controllers/admin/voucherController.js :: showAddVouchersPage

`activation-system/controllers/admin/voucherController.js:34`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/admin/voucherController.js:36` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-add-vouchers', { products, user: req.session.user }) | `activation-system/controllers/admin/voucherController.js:36` |


### controllers/admin/voucherController.js :: showAllVouchers

`activation-system/controllers/admin/voucherController.js:4`

Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/admin-vouchers', { vouchers }) | `activation-system/controllers/admin/voucherController.js:6` |


### controllers/authController.js :: handleLogin

`activation-system/controllers/authController.js:20`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { username, password } = req.body | `activation-system/controllers/authController.js:21` |
| req.body.rememberMe | `activation-system/controllers/authController.js:22` |
| req.body.loginVariant | `activation-system/controllers/authController.js:28` |
| req.session.user | `activation-system/controllers/authController.js:32` |
| req.session.rememberMe | `activation-system/controllers/authController.js:39` |
| req.session.cookie | `activation-system/controllers/authController.js:40` |
| req.session.cookie.maxAge | `activation-system/controllers/authController.js:40` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !user &#124;&#124; !(await bcrypt.compare(password, user.password)) | `activation-system/controllers/authController.js:25` |
| rememberMe | `activation-system/controllers/authController.js:42` |
| user.role === 'kassa_admin' &#124;&#124; user.role === 'kassa_viewer' | `activation-system/controllers/authController.js:60` |
| user.role === 'vendor_user' | `activation-system/controllers/authController.js:58` |
| user.role === 'merchant' | `activation-system/controllers/authController.js:56` |
| user.role === 'admin' &#124;&#124; user.role === 'financial_mgr' &#124;&#124; user.role === 'content_mgr' &#124;&#124; user.role === 'support_agent' | `activation-system/controllers/authController.js:54` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render(view, { error: 'Invalid credentials' }) | `activation-system/controllers/authController.js:29` |
| res.redirect('/admin/dashboard') | `activation-system/controllers/authController.js:55` |
| res.redirect('/merchant/dashboard') | `activation-system/controllers/authController.js:57` |
| res.redirect('/vendor/dashboard') | `activation-system/controllers/authController.js:59` |
| res.redirect('/kassa/dashboard') | `activation-system/controllers/authController.js:61` |
| res.status(403).send('Unknown role') | `activation-system/controllers/authController.js:63` |


### controllers/authController.js :: logout

`activation-system/controllers/authController.js:69`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.cookies?.refresh_token | `activation-system/controllers/authController.js:70` |
| req.session?.user | `activation-system/controllers/authController.js:76` |
| req.session?.user?.id | `activation-system/controllers/authController.js:76` |
| req.session?.user?.role | `activation-system/controllers/authController.js:77` |
| req.session.destroy | `activation-system/controllers/authController.js:82` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| token | `activation-system/controllers/authController.js:71` |
| userId && role | `activation-system/controllers/authController.js:78` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/auth/login') | `activation-system/controllers/authController.js:83` |


### controllers/authController.js :: refreshSession

`activation-system/controllers/authController.js:109`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.cookies?.refresh_token | `activation-system/controllers/authController.js:111` |
| req.session.regenerate | `activation-system/controllers/authController.js:129` |
| req.session.user | `activation-system/controllers/authController.js:134` |
| req.session.rememberMe | `activation-system/controllers/authController.js:141` |
| req.session.cookie | `activation-system/controllers/authController.js:142` |
| req.session.cookie.maxAge | `activation-system/controllers/authController.js:142` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| record?.id | `activation-system/controllers/authController.js:115` |
| !record &#124;&#124; record.expiresAt < new Date() &#124;&#124; !record.userId | `activation-system/controllers/authController.js:114` |
| !user | `activation-system/controllers/authController.js:123` |
| err | `activation-system/controllers/authController.js:130` |
| rotated | `activation-system/controllers/authController.js:145` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ ok: false, message: 'Token expired' }) | `activation-system/controllers/authController.js:119` |
| res.status(401).json({ ok: false, message: 'User not found' }) | `activation-system/controllers/authController.js:126` |
| res.status(500).json({ ok: false }) | `activation-system/controllers/authController.js:131` |
| res.json({ ok: true, role: user.role }) | `activation-system/controllers/authController.js:149` |


### controllers/authController.js :: showLoginForm

`activation-system/controllers/authController.js:16`

Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/login', { error: null }) | `activation-system/controllers/authController.js:17` |


### controllers/client/api/voucherController.js :: buildDetail

`activation-system/controllers/client/api/voucherController.js:80`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| displayMode === 'ondemand' | `activation-system/controllers/client/api/voucherController.js:100` |


### controllers/client/api/voucherController.js :: buildPassDownloadUrl

`activation-system/controllers/client/api/voucherController.js:496`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !normalizedSerial | `activation-system/controllers/client/api/voucherController.js:498` |
| !normalizedBase | `activation-system/controllers/client/api/voucherController.js:503` |


### controllers/client/api/voucherController.js :: claim

`activation-system/controllers/client/api/voucherController.js:361`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/api/voucherController.js:362` |
| req.session?.client?.phone | `activation-system/controllers/client/api/voucherController.js:362` |
| req.body?.payload | `activation-system/controllers/client/api/voucherController.js:368` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phone | `activation-system/controllers/client/api/voucherController.js:363` |
| !voucherCode | `activation-system/controllers/client/api/voucherController.js:371` |
| !voucher | `activation-system/controllers/client/api/voucherController.js:389` |
| !['sold', 'pending'].includes(voucher.status) | `activation-system/controllers/client/api/voucherController.js:393` |
| throw new ClaimError('Ваучер уже привязан к другому клиенту', 409); | `activation-system/controllers/client/api/voucherController.js:403` |
| current && current.clientId !== client.id | `activation-system/controllers/client/api/voucherController.js:402` |
| current | `activation-system/controllers/client/api/voucherController.js:408` |
| isNewAssignment | `activation-system/controllers/client/api/voucherController.js:426` |
| throw new ClaimError('Не удалось обновить ваучер', 500); | `activation-system/controllers/client/api/voucherController.js:451` |
| !onlineVoucher | `activation-system/controllers/client/api/voucherController.js:450` |
| error instanceof ClaimError | `activation-system/controllers/client/api/voucherController.js:457` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ error: 'Unauthorized' }) | `activation-system/controllers/client/api/voucherController.js:364` |
| res.status(400).json({ error: 'Не удалось распознать QR код' }) | `activation-system/controllers/client/api/voucherController.js:372` |
| res.status(404).json({ error: 'Ваучер не найден' }) | `activation-system/controllers/client/api/voucherController.js:390` |
| res.status(409).json({ error: ˋВаучер находится в статусе «${voucher.status}» и не может быть добавленˋ }) | `activation-system/controllers/client/api/voucherController.js:394` |
| res.json(detail) | `activation-system/controllers/client/api/voucherController.js:455` |
| res.status(error.status).json({ error: error.message }) | `activation-system/controllers/client/api/voucherController.js:458` |
| res.status(500).json({ error: 'Не удалось добавить ваучер в кошелёк' }) | `activation-system/controllers/client/api/voucherController.js:461` |


### controllers/client/api/voucherController.js :: codeStatus

`activation-system/controllers/client/api/voucherController.js:249`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/api/voucherController.js:250` |
| req.session?.client?.phone | `activation-system/controllers/client/api/voucherController.js:250` |
| req.params.id | `activation-system/controllers/client/api/voucherController.js:255` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phone | `activation-system/controllers/client/api/voucherController.js:251` |
| !Number.isInteger(voucherId) | `activation-system/controllers/client/api/voucherController.js:256` |
| error instanceof WalletActivationError | `activation-system/controllers/client/api/voucherController.js:265` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ error: 'Unauthorized' }) | `activation-system/controllers/client/api/voucherController.js:252` |
| res.status(400).json({ error: 'Invalid voucher id' }) | `activation-system/controllers/client/api/voucherController.js:257` |
| res.json({ success: true, ...result }) | `activation-system/controllers/client/api/voucherController.js:263` |
| res.status(error.status).json({ success: false, error: error.message, code: error.code }) | `activation-system/controllers/client/api/voucherController.js:266` |
| res.status(500).json({ success: false, error: 'Не удалось получить статус' }) | `activation-system/controllers/client/api/voucherController.js:269` |


### controllers/client/api/voucherController.js :: ensureClient

`activation-system/controllers/client/api/voucherController.js:46`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phone | `activation-system/controllers/client/api/voucherController.js:47` |
| client | `activation-system/controllers/client/api/voucherController.js:50` |
| fallback | `activation-system/controllers/client/api/voucherController.js:53` |


### controllers/client/api/voucherController.js :: formatSummary

`activation-system/controllers/client/api/voucherController.js:64`

### controllers/client/api/voucherController.js :: list

`activation-system/controllers/client/api/voucherController.js:150`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/api/voucherController.js:151` |
| req.session?.client?.phone | `activation-system/controllers/client/api/voucherController.js:151` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phone | `activation-system/controllers/client/api/voucherController.js:152` |
| ['used', 'deleted'].includes(status) | `activation-system/controllers/client/api/voucherController.js:171` |
| status === 'activated' && !item.voucher.product?.walletOnDemandEnabled | `activation-system/controllers/client/api/voucherController.js:176` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ error: 'Unauthorized' }) | `activation-system/controllers/client/api/voucherController.js:153` |
| res.json({ vouchers, syncedAt: new Date().toISOString() }) | `activation-system/controllers/client/api/voucherController.js:180` |
| res.status(500).json({ error: 'Failed to load vouchers' }) | `activation-system/controllers/client/api/voucherController.js:183` |


### controllers/client/api/voucherController.js :: logEvent

`activation-system/controllers/client/api/voucherController.js:273`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/api/voucherController.js:274` |
| req.session?.client?.phone | `activation-system/controllers/client/api/voucherController.js:274` |
| req.params.id | `activation-system/controllers/client/api/voucherController.js:279` |
| { event, meta } = req.body &#124;&#124; {} | `activation-system/controllers/client/api/voucherController.js:280` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phone | `activation-system/controllers/client/api/voucherController.js:275` |
| !allowed.has(event) | `activation-system/controllers/client/api/voucherController.js:282` |
| !onlineVoucher | `activation-system/controllers/client/api/voucherController.js:293` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ error: 'Unauthorized' }) | `activation-system/controllers/client/api/voucherController.js:276` |
| res.status(400).json({ error: 'Unsupported event' }) | `activation-system/controllers/client/api/voucherController.js:283` |
| res.status(404).json({ error: 'Voucher not found' }) | `activation-system/controllers/client/api/voucherController.js:294` |
| res.json({ ok: true }) | `activation-system/controllers/client/api/voucherController.js:307` |
| res.status(500).json({ error: 'Failed to log event' }) | `activation-system/controllers/client/api/voucherController.js:310` |


### controllers/client/api/voucherController.js :: normalizeBaseUrl

`activation-system/controllers/client/api/voucherController.js:471`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !candidate &#124;&#124; typeof candidate !== 'string' | `activation-system/controllers/client/api/voucherController.js:472` |
| !trimmed | `activation-system/controllers/client/api/voucherController.js:476` |


### controllers/client/api/voucherController.js :: requestCode

`activation-system/controllers/client/api/voucherController.js:224`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/api/voucherController.js:225` |
| req.session?.client?.phone | `activation-system/controllers/client/api/voucherController.js:225` |
| req.params.id | `activation-system/controllers/client/api/voucherController.js:230` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phone | `activation-system/controllers/client/api/voucherController.js:226` |
| !Number.isInteger(voucherId) | `activation-system/controllers/client/api/voucherController.js:231` |
| error instanceof WalletActivationError | `activation-system/controllers/client/api/voucherController.js:240` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ error: 'Unauthorized' }) | `activation-system/controllers/client/api/voucherController.js:227` |
| res.status(400).json({ error: 'Invalid voucher id' }) | `activation-system/controllers/client/api/voucherController.js:232` |
| res.json({ success: true, ...result }) | `activation-system/controllers/client/api/voucherController.js:238` |
| res.status(error.status).json({ success: false, error: error.message, code: error.code }) | `activation-system/controllers/client/api/voucherController.js:241` |
| res.status(500).json({ success: false, error: 'Не удалось получить код. Попробуйте ещё раз.' }) | `activation-system/controllers/client/api/voucherController.js:244` |


### controllers/client/api/voucherController.js :: resolveAppBaseUrl

`activation-system/controllers/client/api/voucherController.js:483`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| normalizedEnv | `activation-system/controllers/client/api/voucherController.js:486` |
| normalizedOrigin | `activation-system/controllers/client/api/voucherController.js:490` |


### controllers/client/api/voucherController.js :: resolveDisplayMode

`activation-system/controllers/client/api/voucherController.js:17`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !vendor &#124;&#124; vendor.productType === LEGACY_ONLY_TYPE | `activation-system/controllers/client/api/voucherController.js:19` |


### controllers/client/api/voucherController.js :: resolveRequestOrigin

`activation-system/controllers/client/api/voucherController.js:465`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.headers['x-forwarded-proto'] | `activation-system/controllers/client/api/voucherController.js:466` |


### controllers/client/api/voucherController.js :: show

`activation-system/controllers/client/api/voucherController.js:187`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/api/voucherController.js:188` |
| req.session?.client?.phone | `activation-system/controllers/client/api/voucherController.js:188` |
| req.params.id | `activation-system/controllers/client/api/voucherController.js:193` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phone | `activation-system/controllers/client/api/voucherController.js:189` |
| !Number.isInteger(voucherId) | `activation-system/controllers/client/api/voucherController.js:194` |
| !onlineVoucher | `activation-system/controllers/client/api/voucherController.js:210` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ error: 'Unauthorized' }) | `activation-system/controllers/client/api/voucherController.js:190` |
| res.status(400).json({ error: 'Invalid voucher id' }) | `activation-system/controllers/client/api/voucherController.js:195` |
| res.status(404).json({ error: 'Voucher not found' }) | `activation-system/controllers/client/api/voucherController.js:211` |
| res.json(detail) | `activation-system/controllers/client/api/voucherController.js:216` |
| res.status(500).json({ error: 'Failed to load voucher' }) | `activation-system/controllers/client/api/voucherController.js:219` |


### controllers/client/api/voucherController.js :: subscribePush

`activation-system/controllers/client/api/voucherController.js:314`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/api/voucherController.js:315` |
| req.session?.client?.phone | `activation-system/controllers/client/api/voucherController.js:315` |
| req.body?.subscription | `activation-system/controllers/client/api/voucherController.js:320` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phone | `activation-system/controllers/client/api/voucherController.js:316` |
| !subscription?.endpoint | `activation-system/controllers/client/api/voucherController.js:321` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ error: 'Unauthorized' }) | `activation-system/controllers/client/api/voucherController.js:317` |
| res.status(400).json({ error: 'Invalid subscription payload' }) | `activation-system/controllers/client/api/voucherController.js:322` |
| res.json({ ok: true }) | `activation-system/controllers/client/api/voucherController.js:346` |
| res.status(500).json({ error: 'Failed to store subscription' }) | `activation-system/controllers/client/api/voucherController.js:349` |


### controllers/client/authController.js :: deleteAccount

`activation-system/controllers/client/authController.js:326`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/authController.js:329` |
| req.session?.client?.id | `activation-system/controllers/client/authController.js:329` |
| req.session.destroy | `activation-system/controllers/client/authController.js:382` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !clientId | `activation-system/controllers/client/authController.js:330` |
| !client | `activation-system/controllers/client/authController.js:336` |
| err | `activation-system/controllers/client/authController.js:383` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ ok: false, message: 'Not authenticated' }) | `activation-system/controllers/client/authController.js:331` |
| res.status(404).json({ ok: false, message: 'Client not found' }) | `activation-system/controllers/client/authController.js:337` |
| res.status(500).json({ ok: false, message: 'Session destruction failed' }) | `activation-system/controllers/client/authController.js:385` |
| res.json({ ok: true, message: 'Account deleted successfully' }) | `activation-system/controllers/client/authController.js:387` |
| res.status(500).json({ ok: false, message: 'Failed to delete account' }) | `activation-system/controllers/client/authController.js:391` |


### controllers/client/authController.js :: generateOtp

`activation-system/controllers/client/authController.js:17`

### controllers/client/authController.js :: handleLogin

`activation-system/controllers/client/authController.js:117`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { phoneNumber, phone } = req.body | `activation-system/controllers/client/authController.js:121` |
| req.session.pendingRememberMe | `activation-system/controllers/client/authController.js:153` |
| req.session.pendingClientId | `activation-system/controllers/client/authController.js:154` |
| req.session.save | `activation-system/controllers/client/authController.js:157` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !normalizedPhone &#124;&#124; normalizedPhone.trim() === '+' | `activation-system/controllers/client/authController.js:126` |
| client | `activation-system/controllers/client/authController.js:137` |
| fallback | `activation-system/controllers/client/authController.js:135` |
| !client | `activation-system/controllers/client/authController.js:133` |
| err | `activation-system/controllers/client/authController.js:158` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/client-login', { error: 'Введите номер телефона' }) | `activation-system/controllers/client/authController.js:127` |
| res.render('pages/client-login', { error: 'Аккаунт не найден. Пожалуйста, пройдите регистрацию.' }) | `activation-system/controllers/client/authController.js:148` |
| res.render('pages/client-login', { error: 'Не удалось сохранить сессию. Попробуйте позже.' }) | `activation-system/controllers/client/authController.js:160` |
| res.redirect('/client-verify') | `activation-system/controllers/client/authController.js:162` |
| res.render('pages/client-login', { error: 'Произошла ошибка. Попробуйте позже.' }) | `activation-system/controllers/client/authController.js:166` |


### controllers/client/authController.js :: handleRegister

`activation-system/controllers/client/authController.js:257`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { phoneNumber, phone, name } = req.body | `activation-system/controllers/client/authController.js:261` |
| req.session.pendingRememberMe | `activation-system/controllers/client/authController.js:283` |
| req.session.pendingClientId | `activation-system/controllers/client/authController.js:284` |
| req.session.save | `activation-system/controllers/client/authController.js:286` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !normalizedPhone &#124;&#124; !name &#124;&#124; normalizedPhone.trim() === '+' &#124;&#124; name.trim() === '' | `activation-system/controllers/client/authController.js:265` |
| fallback | `activation-system/controllers/client/authController.js:273` |
| !existing | `activation-system/controllers/client/authController.js:271` |
| existing | `activation-system/controllers/client/authController.js:277` |
| err | `activation-system/controllers/client/authController.js:287` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/client-register', { error: 'Заполните все поля' }) | `activation-system/controllers/client/authController.js:266` |
| res.render('pages/client-register', { error: 'Пользователь уже существует' }) | `activation-system/controllers/client/authController.js:278` |
| res.render('pages/client-register', { error: 'Не удалось сохранить сессию. Попробуйте позже.' }) | `activation-system/controllers/client/authController.js:289` |
| res.redirect('/client-verify') | `activation-system/controllers/client/authController.js:291` |
| res.render('pages/client-register', { error: 'Не удалось зарегистрировать пользователя. Попробуйте позже.' }) | `activation-system/controllers/client/authController.js:295` |


### controllers/client/authController.js :: logout

`activation-system/controllers/client/authController.js:300`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.cookies?.refresh_token | `activation-system/controllers/client/authController.js:304` |
| req.session?.client | `activation-system/controllers/client/authController.js:310` |
| req.session?.client?.id | `activation-system/controllers/client/authController.js:310` |
| req.session.destroy | `activation-system/controllers/client/authController.js:315` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| token | `activation-system/controllers/client/authController.js:305` |
| clientId | `activation-system/controllers/client/authController.js:311` |
| err | `activation-system/controllers/client/authController.js:316` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/wallet') | `activation-system/controllers/client/authController.js:318` |


### controllers/client/authController.js :: refreshSession

`activation-system/controllers/client/authController.js:395`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.cookies?.refresh_token | `activation-system/controllers/client/authController.js:397` |
| req.session.regenerate | `activation-system/controllers/client/authController.js:415` |
| req.session.client | `activation-system/controllers/client/authController.js:420` |
| req.session.rememberMe | `activation-system/controllers/client/authController.js:421` |
| req.session.cookie | `activation-system/controllers/client/authController.js:422` |
| req.session.cookie.maxAge | `activation-system/controllers/client/authController.js:422` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| record?.id | `activation-system/controllers/client/authController.js:401` |
| !record &#124;&#124; record.expiresAt < new Date() &#124;&#124; !record.clientId | `activation-system/controllers/client/authController.js:400` |
| !client | `activation-system/controllers/client/authController.js:409` |
| err | `activation-system/controllers/client/authController.js:416` |
| rotated | `activation-system/controllers/client/authController.js:425` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ ok: false, message: 'Token expired' }) | `activation-system/controllers/client/authController.js:405` |
| res.status(401).json({ ok: false, message: 'Client not found' }) | `activation-system/controllers/client/authController.js:412` |
| res.status(500).json({ ok: false }) | `activation-system/controllers/client/authController.js:417` |
| res.json({ ok: true }) | `activation-system/controllers/client/authController.js:429` |


### controllers/client/authController.js :: setOtpSessionAndSend

`activation-system/controllers/client/authController.js:22`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.otp | `activation-system/controllers/client/authController.js:24` |
| req.session.phone | `activation-system/controllers/client/authController.js:25` |
| req.session.otpCreatedAt | `activation-system/controllers/client/authController.js:26` |


### controllers/client/authController.js :: showLogin

`activation-system/controllers/client/authController.js:75`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/authController.js:79` |
| req.session.regenerate | `activation-system/controllers/client/authController.js:89` |
| req.session.client | `activation-system/controllers/client/authController.js:95` |
| req.session.rememberMe | `activation-system/controllers/client/authController.js:96` |
| req.session.cookie | `activation-system/controllers/client/authController.js:97` |
| req.session.cookie.maxAge | `activation-system/controllers/client/authController.js:97` |
| req.session.save | `activation-system/controllers/client/authController.js:102` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/authController.js:79` |
| err | `activation-system/controllers/client/authController.js:90` |
| rotated | `activation-system/controllers/client/authController.js:100` |
| saveErr | `activation-system/controllers/client/authController.js:103` |
| restoreResult | `activation-system/controllers/client/authController.js:86` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/client/dashboard') | `activation-system/controllers/client/authController.js:80` |
| res.render('pages/client-login', { error: null }) | `activation-system/controllers/client/authController.js:92` |


### controllers/client/authController.js :: showOtpPage

`activation-system/controllers/client/authController.js:171`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.phone | `activation-system/controllers/client/authController.js:172` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !req.session.phone | `activation-system/controllers/client/authController.js:172` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/wallet') | `activation-system/controllers/client/authController.js:172` |
| res.render('pages/client-verify', { phone: req.session.phone, error: null, }) | `activation-system/controllers/client/authController.js:173` |


### controllers/client/authController.js :: showRegister

`activation-system/controllers/client/authController.js:249`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/authController.js:250` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/authController.js:250` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/client/dashboard') | `activation-system/controllers/client/authController.js:251` |
| res.render('pages/client-register', { error: null }) | `activation-system/controllers/client/authController.js:253` |


### controllers/client/authController.js :: tryRestoreSessionFromRefreshToken

`activation-system/controllers/client/authController.js:38`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.cookies?.refresh_token | `activation-system/controllers/client/authController.js:39` |
| req.cookies?.remember_me | `activation-system/controllers/client/authController.js:40` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !refreshToken &#124;&#124; !rememberMe | `activation-system/controllers/client/authController.js:42` |
| record | `activation-system/controllers/client/authController.js:50` |
| !record &#124;&#124; record.expiresAt < new Date() &#124;&#124; !record.clientId | `activation-system/controllers/client/authController.js:49` |
| !client | `activation-system/controllers/client/authController.js:59` |


### controllers/client/authController.js :: verifyOtp

`activation-system/controllers/client/authController.js:180`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { otp } = req.body | `activation-system/controllers/client/authController.js:181` |
| req.session.otpCreatedAt | `activation-system/controllers/client/authController.js:185` |
| req.session.otp | `activation-system/controllers/client/authController.js:186` |
| req.session.phone | `activation-system/controllers/client/authController.js:189` |
| req.session.pendingClientId | `activation-system/controllers/client/authController.js:198` |
| req.session.pendingRememberMe | `activation-system/controllers/client/authController.js:210` |
| req.session.client | `activation-system/controllers/client/authController.js:215` |
| req.session.rememberMe | `activation-system/controllers/client/authController.js:216` |
| req.session.cookie | `activation-system/controllers/client/authController.js:217` |
| req.session.cookie.maxAge | `activation-system/controllers/client/authController.js:217` |
| req.session.save | `activation-system/controllers/client/authController.js:236` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !req.session.otpCreatedAt &#124;&#124; Date.now() - req.session.otpCreatedAt > OTP_TTL_MS | `activation-system/controllers/client/authController.js:185` |
| !otp &#124;&#124; otp !== req.session.otp | `activation-system/controllers/client/authController.js:194` |
| !clientId | `activation-system/controllers/client/authController.js:199` |
| !client | `activation-system/controllers/client/authController.js:204` |
| rememberMe | `activation-system/controllers/client/authController.js:224` |
| err | `activation-system/controllers/client/authController.js:237` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/client-verify', { phone: req.session.phone, error: 'Код истёк. Пожалуйста, вернитесь назад и запросите новый код.' }) | `activation-system/controllers/client/authController.js:188` |
| res.render('pages/client-verify', { phone: req.session.phone, error: 'Неверный код' }) | `activation-system/controllers/client/authController.js:195` |
| res.redirect('/wallet') | `activation-system/controllers/client/authController.js:200` |
| res.render('pages/client-verify', { phone: phoneForRender, error: 'Не удалось сохранить сессию. Попробуйте снова.', }) | `activation-system/controllers/client/authController.js:239` |
| res.redirect('/client/dashboard') | `activation-system/controllers/client/authController.js:244` |


### controllers/client/dashboardController.js :: showDashboard

`activation-system/controllers/client/dashboardController.js:28`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session?.client | `activation-system/controllers/client/dashboardController.js:29` |
| req.session?.client?.phone | `activation-system/controllers/client/dashboardController.js:29` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phone | `activation-system/controllers/client/dashboardController.js:30` |
| !client | `activation-system/controllers/client/dashboardController.js:52` |
| ['used', 'deleted'].includes(status) | `activation-system/controllers/client/dashboardController.js:71` |
| status === 'activated' && !item.voucher.product?.walletOnDemandEnabled | `activation-system/controllers/client/dashboardController.js:72` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/wallet') | `activation-system/controllers/client/dashboardController.js:31` |
| res.render('pages/client-dashboard', { phone, vouchers, pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, isClient: true, }) | `activation-system/controllers/client/dashboardController.js:77` |
| res.status(500).render('pages/client-dashboard', { phone, vouchers: [], pushPublicKey: process.env.WALLET_VAPID_PUBLIC_KEY &#124;&#124; null, isClient: true, }) | `activation-system/controllers/client/dashboardController.js:86` |


### controllers/kassa/kassaCabinetController.js :: createPayout

`activation-system/controllers/kassa/kassaCabinetController.js:167`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/kassa/kassaCabinetController.js:169` |
| req.session.user.kassaId | `activation-system/controllers/kassa/kassaCabinetController.js:169` |
| { amount, recipientType, recipientId, comment } = req.body | `activation-system/controllers/kassa/kassaCabinetController.js:170` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !parsedAmount &#124;&#124; parsedAmount <= 0 | `activation-system/controllers/kassa/kassaCabinetController.js:173` |
| recipientType === 'vendor' && !recipientId | `activation-system/controllers/kassa/kassaCabinetController.js:178` |
| !kassa | `activation-system/controllers/kassa/kassaCabinetController.js:183` |
| remaining <= 0 | `activation-system/controllers/kassa/kassaCabinetController.js:225` |
| payout <= 0 | `activation-system/controllers/kassa/kassaCabinetController.js:227` |
| remaining >= payout | `activation-system/controllers/kassa/kassaCabinetController.js:235` |
| recipientType === 'vendor' && recipientId | `activation-system/controllers/kassa/kassaCabinetController.js:209` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(400).send('Некорректная сумма') | `activation-system/controllers/kassa/kassaCabinetController.js:174` |
| res.status(400).send('Выберите вендора для выплаты') | `activation-system/controllers/kassa/kassaCabinetController.js:179` |
| res.status(404).send('Касса не найдена') | `activation-system/controllers/kassa/kassaCabinetController.js:183` |
| res.redirect('/kassa/payouts') | `activation-system/controllers/kassa/kassaCabinetController.js:252` |
| res.status(500).send('Ошибка создания выплаты') | `activation-system/controllers/kassa/kassaCabinetController.js:255` |


### controllers/kassa/kassaCabinetController.js :: showDashboard

`activation-system/controllers/kassa/kassaCabinetController.js:4`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/kassa/kassaCabinetController.js:6` |
| req.session.user.kassaId | `activation-system/controllers/kassa/kassaCabinetController.js:6` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !kassa | `activation-system/controllers/kassa/kassaCabinetController.js:12` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Касса не найдена') | `activation-system/controllers/kassa/kassaCabinetController.js:12` |
| res.render('pages/kassa/dashboard', { layout: false, user: req.session.user, kassa, recentTransactions, weekSales: { count: weekSales._count?.id &#124;&#124; 0, total: Number(weekSales._sum?.price &#124;&#124; 0), commission: Number(weekSales._sum?.kassaDebt &#124;&#124; 0), }, }) | `activation-system/controllers/kassa/kassaCabinetController.js:30` |
| res.status(500).send('Ошибка загрузки дашборда') | `activation-system/controllers/kassa/kassaCabinetController.js:43` |


### controllers/kassa/kassaCabinetController.js :: showObligations

`activation-system/controllers/kassa/kassaCabinetController.js:88`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/kassa/kassaCabinetController.js:90` |
| req.session.user.kassaId | `activation-system/controllers/kassa/kassaCabinetController.js:90` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/kassa/obligations', { layout: false, user: req.session.user, vendorStats: vendorStats.map((vs) => ({ ...vs, vendorName: vendorMap[vs.vendorId]?.name &#124;&#124; ˋVendor #${vs.vendorId}ˋ, vendorBalance: vendorMap[vs.vendorId]?.balance &#124;&#124; 0, })), adjustments, }) | `activation-system/controllers/kassa/kassaCabinetController.js:115` |
| res.status(500).send('Ошибка загрузки обязательств') | `activation-system/controllers/kassa/kassaCabinetController.js:127` |


### controllers/kassa/kassaCabinetController.js :: showPayouts

`activation-system/controllers/kassa/kassaCabinetController.js:132`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/kassa/kassaCabinetController.js:134` |
| req.session.user.kassaId | `activation-system/controllers/kassa/kassaCabinetController.js:134` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/kassa/payouts', { layout: false, user: req.session.user, payments, vendors, kassaBalance: kassa?.balance &#124;&#124; 0, totalPaid: kassa?.totalPaid &#124;&#124; 0, }) | `activation-system/controllers/kassa/kassaCabinetController.js:152` |
| res.status(500).send('Ошибка загрузки выплат') | `activation-system/controllers/kassa/kassaCabinetController.js:162` |


### controllers/kassa/kassaCabinetController.js :: showSettings

`activation-system/controllers/kassa/kassaCabinetController.js:260`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/kassa/kassaCabinetController.js:262` |
| req.session.user.kassaId | `activation-system/controllers/kassa/kassaCabinetController.js:262` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !kassa | `activation-system/controllers/kassa/kassaCabinetController.js:268` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Касса не найдена') | `activation-system/controllers/kassa/kassaCabinetController.js:268` |
| res.render('pages/kassa/settings', { layout: false, user: req.session.user, kassa, callbackUrls, }) | `activation-system/controllers/kassa/kassaCabinetController.js:277` |
| res.status(500).send('Ошибка загрузки настроек') | `activation-system/controllers/kassa/kassaCabinetController.js:285` |


### controllers/kassa/kassaCabinetController.js :: showTransactions

`activation-system/controllers/kassa/kassaCabinetController.js:48`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/kassa/kassaCabinetController.js:50` |
| req.session.user.kassaId | `activation-system/controllers/kassa/kassaCabinetController.js:50` |
| { page = 1, vendor, status } = req.query | `activation-system/controllers/kassa/kassaCabinetController.js:51` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| vendor | `activation-system/controllers/kassa/kassaCabinetController.js:56` |
| status | `activation-system/controllers/kassa/kassaCabinetController.js:57` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/kassa/transactions', { layout: false, user: req.session.user, transactions, vendors, total, page: parseInt(page), totalPages: Math.ceil(total / limit), filters: { vendor, status }, }) | `activation-system/controllers/kassa/kassaCabinetController.js:71` |
| res.status(500).send('Ошибка загрузки транзакций') | `activation-system/controllers/kassa/kassaCabinetController.js:83` |


### controllers/merchant/cartController.js :: addToCart

`activation-system/controllers/merchant/cartController.js:43`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.body.productId | `activation-system/controllers/merchant/cartController.js:44` |
| req.session.cart | `activation-system/controllers/merchant/cartController.js:50` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !productId | `activation-system/controllers/merchant/cartController.js:45` |
| !product | `activation-system/controllers/merchant/cartController.js:48` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/merchant/sell') | `activation-system/controllers/merchant/cartController.js:45` |
| res.status(404).send('Товар не найден') | `activation-system/controllers/merchant/cartController.js:48` |
| res.redirect('/merchant/checkout') | `activation-system/controllers/merchant/cartController.js:57` |


### controllers/merchant/cartController.js :: removeFromCart

`activation-system/controllers/merchant/cartController.js:93`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.cart | `activation-system/controllers/merchant/cartController.js:94` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/merchant/checkout') | `activation-system/controllers/merchant/cartController.js:95` |


### controllers/merchant/cartController.js :: showCart

`activation-system/controllers/merchant/cartController.js:61`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.cart | `activation-system/controllers/merchant/cartController.js:62` |
| req.session.cart.length | `activation-system/controllers/merchant/cartController.js:62` |
| req.session.cart[0] | `activation-system/controllers/merchant/cartController.js:62` |
| req.session.checkoutToken | `activation-system/controllers/merchant/cartController.js:76` |
| req.session.user | `activation-system/controllers/merchant/cartController.js:83` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/checkout', { items: enrichedCart, totalPrice, totalQuantity, checkoutToken: req.session.checkoutToken, user: req.session.user, }) | `activation-system/controllers/merchant/cartController.js:78` |


### controllers/merchant/cartController.js :: showProductsForSale

`activation-system/controllers/merchant/cartController.js:5`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.cart | `activation-system/controllers/merchant/cartController.js:31` |
| req.session.user | `activation-system/controllers/merchant/cartController.js:38` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !Array.isArray(req.session.cart) | `activation-system/controllers/merchant/cartController.js:31` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/merchant-sell', { products: enrichedProducts, cart: req.session.cart, user: req.session.user, }) | `activation-system/controllers/merchant/cartController.js:35` |


### controllers/merchant/cartController.js :: updateCart

`activation-system/controllers/merchant/cartController.js:88`

Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/merchant/checkout') | `activation-system/controllers/merchant/cartController.js:89` |


### controllers/merchant/dashboardController.js :: maskVoucher

`activation-system/controllers/merchant/dashboardController.js:52`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| typeof val !== 'string' | `activation-system/controllers/merchant/dashboardController.js:53` |
| val.length <= 3 | `activation-system/controllers/merchant/dashboardController.js:54` |


### controllers/merchant/dashboardController.js :: showMerchantDashboard

`activation-system/controllers/merchant/dashboardController.js:4`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/merchant/dashboardController.js:5` |
| req.session.user.username | `activation-system/controllers/merchant/dashboardController.js:5` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| typeof val !== 'string' | `activation-system/controllers/merchant/dashboardController.js:17` |
| val.length <= 3 | `activation-system/controllers/merchant/dashboardController.js:18` |
| merchant | `activation-system/controllers/merchant/dashboardController.js:25` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/merchant-sales', { sales: maskedSales, user: req.session.user &#124;&#124; null, merchantDebt, }) | `activation-system/controllers/merchant/dashboardController.js:37` |


### controllers/merchant/saleController.js :: confirmCheckout

`activation-system/controllers/merchant/saleController.js:30`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/merchant/saleController.js:31` |
| req.body.saleType | `activation-system/controllers/merchant/saleController.js:45` |
| req.body.customerPhone | `activation-system/controllers/merchant/saleController.js:47` |
| req.session.cart | `activation-system/controllers/merchant/saleController.js:49` |
| req.body.paymentConfirmed | `activation-system/controllers/merchant/saleController.js:56` |
| req.body.checkoutToken | `activation-system/controllers/merchant/saleController.js:61` |
| req.session.checkoutToken | `activation-system/controllers/merchant/saleController.js:62` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !merchant | `activation-system/controllers/merchant/saleController.js:41` |
| cart.length === 0 | `activation-system/controllers/merchant/saleController.js:51` |
| req.body.paymentConfirmed !== 'on' | `activation-system/controllers/merchant/saleController.js:56` |
| !submittedToken &#124;&#124; submittedToken !== req.session.checkoutToken | `activation-system/controllers/merchant/saleController.js:62` |
| !normalizedPhone &#124;&#124; !normalizedPhone.match(/^\+998[0-9]{9}$/) | `activation-system/controllers/merchant/saleController.js:71` |
| saleType === 'ONLINE' | `activation-system/controllers/merchant/saleController.js:70` |
| client | `activation-system/controllers/merchant/saleController.js:106` |
| fallback | `activation-system/controllers/merchant/saleController.js:104` |
| !client | `activation-system/controllers/merchant/saleController.js:114` |
| !cartItem | `activation-system/controllers/merchant/saleController.js:125` |
| !product | `activation-system/controllers/merchant/saleController.js:134` |
| !activeVoucher | `activation-system/controllers/merchant/saleController.js:143` |
| throw new Error('VOUCHER_ALREADY_TAKEN'); | `activation-system/controllers/merchant/saleController.js:162` |
| claim.count !== 1 | `activation-system/controllers/merchant/saleController.js:161` |
| saleType === 'ONLINE' && client | `activation-system/controllers/merchant/saleController.js:179` |
| kassaId | `activation-system/controllers/merchant/saleController.js:232` |
| saleType === 'OFFLINE' | `activation-system/controllers/merchant/saleController.js:251` |
| error.message === 'VOUCHER_ALREADY_TAKEN' | `activation-system/controllers/merchant/saleController.js:411` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(403).send('Мерчант не найден') | `activation-system/controllers/merchant/saleController.js:42` |
| res.status(400).send('Корзина пуста') | `activation-system/controllers/merchant/saleController.js:52` |
| res.status(400).send('Подтвердите, что оплата от клиента получена, прежде чем завершить продажу') | `activation-system/controllers/merchant/saleController.js:57` |
| res.status(409).send('Эта продажа уже обработана либо срок чека истёк. Обновите страницу чека и попробуйте снова.') | `activation-system/controllers/merchant/saleController.js:63` |
| res.status(400).send('Для онлайн продажи требуется корректный номер телефона в формате +998XXXXXXXXX') | `activation-system/controllers/merchant/saleController.js:72` |
| res.status(400).send(ˋТовар с id ${cartItem.productId} недоступен для продажиˋ) | `activation-system/controllers/merchant/saleController.js:135` |
| res.status(409).send(ˋНедостаточно активных ваучеров для товара "${product.name}".ˋ) | `activation-system/controllers/merchant/saleController.js:144` |
| res.status(409).send(ˋВаучер для товара "${product.name}" только что был продан другой кассой. Обновите страницу чека и попробуйте снова.ˋ) | `activation-system/controllers/merchant/saleController.js:412` |
| res.status(500).send('Ошибка при оформлении покупки: ' + error.message) | `activation-system/controllers/merchant/saleController.js:415` |


### controllers/merchant/saleController.js :: drawLegalInfo

`activation-system/controllers/merchant/saleController.js:608`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| merchant?.legalInfo | `activation-system/controllers/merchant/saleController.js:609` |


### controllers/merchant/saleController.js :: maskVoucherCode

`activation-system/controllers/merchant/saleController.js:461`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !raw | `activation-system/controllers/merchant/saleController.js:463` |
| cleaned.length <= 3 | `activation-system/controllers/merchant/saleController.js:465` |


### controllers/merchant/saleController.js :: registerFont

`activation-system/controllers/merchant/saleController.js:594`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| fs.existsSync(fontPath) | `activation-system/controllers/merchant/saleController.js:595` |


### controllers/merchant/saleController.js :: resolveReceiptPlaceholders

`activation-system/controllers/merchant/saleController.js:439`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| resolver | `activation-system/controllers/merchant/saleController.js:442` |


### controllers/merchant/saleController.js :: showMerchantSales

`activation-system/controllers/merchant/saleController.js:700`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/merchant/saleController.js:701` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| merchant | `activation-system/controllers/merchant/saleController.js:719` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/merchant-sales', { sales: maskedSales, user, merchantDebt, }) | `activation-system/controllers/merchant/saleController.js:730` |


### controllers/payment/alifController.js :: handleWebhook

`activation-system/controllers/payment/alifController.js:12`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.headers['signature'] | `activation-system/controllers/payment/alifController.js:14` |
| req.params.kassaId | `activation-system/controllers/payment/alifController.js:17` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !kassaId | `activation-system/controllers/payment/alifController.js:18` |
| !kassa | `activation-system/controllers/payment/alifController.js:24` |
| !verifyAlifSignature(rawBody, signature, secretKey) | `activation-system/controllers/payment/alifController.js:33` |
| orderIdFromMeta | `activation-system/controllers/payment/alifController.js:53` |
| !attempt | `activation-system/controllers/payment/alifController.js:59` |
| attempt.status === 'PAID' | `activation-system/controllers/payment/alifController.js:72` |
| paymentStatus && !ALIF_NON_FINAL_STATUSES.includes(paymentStatus) | `activation-system/controllers/payment/alifController.js:77` |
| paymentStatus !== 'SUCCEEDED' | `activation-system/controllers/payment/alifController.js:76` |
| !result.success | `activation-system/controllers/payment/alifController.js:90` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(200).json({ status: 'kassa_not_found' }) | `activation-system/controllers/payment/alifController.js:20` |
| res.status(403).json({ status: 'invalid_signature' }) | `activation-system/controllers/payment/alifController.js:35` |
| res.status(200).json({ status: 'attempt_not_found' }) | `activation-system/controllers/payment/alifController.js:68` |
| res.status(200).json({ status: 'already_paid' }) | `activation-system/controllers/payment/alifController.js:73` |
| res.status(200).json({ status: 'not_succeeded', paymentStatus }) | `activation-system/controllers/payment/alifController.js:83` |
| res.status(200).json({ status: 'processing_failed', error: result.error }) | `activation-system/controllers/payment/alifController.js:92` |
| res.status(200).json({ status: 'ok' }) | `activation-system/controllers/payment/alifController.js:96` |
| res.status(200).json({ status: 'internal_error' }) | `activation-system/controllers/payment/alifController.js:102` |


### controllers/payment/clickController.js :: handleComplete

`activation-system/controllers/payment/clickController.js:119`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.kassaId | `activation-system/controllers/payment/clickController.js:126` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !kassaId | `activation-system/controllers/payment/clickController.js:127` |
| !kassa | `activation-system/controllers/payment/clickController.js:131` |
| !verifyClickSign(params, 1, kassa) | `activation-system/controllers/payment/clickController.js:136` |
| !attempt | `activation-system/controllers/payment/clickController.js:160` |
| parseInt(params.merchant_prepare_id) !== attempt.id | `activation-system/controllers/payment/clickController.js:171` |
| parseInt(params.error) < 0 | `activation-system/controllers/payment/clickController.js:182` |
| attempt.status === 'PAID' | `activation-system/controllers/payment/clickController.js:199` |
| !result.success | `activation-system/controllers/payment/clickController.js:214` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'Kassa not specified' }) | `activation-system/controllers/payment/clickController.js:128` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'Kassa not found' }) | `activation-system/controllers/payment/clickController.js:132` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SIGN_CHECK_FAILED, error_note: 'SIGN CHECK FAILED!' }) | `activation-system/controllers/payment/clickController.js:138` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.TRANSACTION_NOT_FOUND, error_note: 'Transaction not found' }) | `activation-system/controllers/payment/clickController.js:162` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.TRANSACTION_NOT_FOUND, error_note: 'Invalid prepare_id' }) | `activation-system/controllers/payment/clickController.js:173` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, merchant_confirm_id: attempt.id, error: CLICK_ERRORS.CANCELLED, error_note: 'Transaction cancelled' }) | `activation-system/controllers/payment/clickController.js:189` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, merchant_confirm_id: attempt.id, error: CLICK_ERRORS.SUCCESS, error_note: 'Success' }) | `activation-system/controllers/payment/clickController.js:201` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: result.error &#124;&#124; 'Payment processing failed' }) | `activation-system/controllers/payment/clickController.js:216` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, merchant_confirm_id: attempt.id, error: CLICK_ERRORS.SUCCESS, error_note: 'Success' }) | `activation-system/controllers/payment/clickController.js:226` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'System error' }) | `activation-system/controllers/payment/clickController.js:236` |


### controllers/payment/clickController.js :: handlePrepare

`activation-system/controllers/payment/clickController.js:8`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.params.kassaId | `activation-system/controllers/payment/clickController.js:15` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !kassaId | `activation-system/controllers/payment/clickController.js:16` |
| !kassa | `activation-system/controllers/payment/clickController.js:20` |
| !verifyClickSign(params, 0, kassa) | `activation-system/controllers/payment/clickController.js:25` |
| !attempt | `activation-system/controllers/payment/clickController.js:49` |
| Math.abs(expectedAmount - receivedAmount) > 0.01 | `activation-system/controllers/payment/clickController.js:63` |
| attempt.status === 'PAID' | `activation-system/controllers/payment/clickController.js:74` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'Kassa not specified' }) | `activation-system/controllers/payment/clickController.js:17` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'Kassa not found' }) | `activation-system/controllers/payment/clickController.js:21` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SIGN_CHECK_FAILED, error_note: 'SIGN CHECK FAILED!' }) | `activation-system/controllers/payment/clickController.js:27` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.ORDER_NOT_FOUND, error_note: 'Order not found' }) | `activation-system/controllers/payment/clickController.js:51` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.WRONG_AMOUNT, error_note: 'Wrong amount' }) | `activation-system/controllers/payment/clickController.js:65` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, merchant_prepare_id: attempt.id, error: CLICK_ERRORS.ALREADY_PAID, error_note: 'Already paid' }) | `activation-system/controllers/payment/clickController.js:76` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, merchant_prepare_id: attempt.id, error: CLICK_ERRORS.SUCCESS, error_note: 'Success' }) | `activation-system/controllers/payment/clickController.js:97` |
| res.json({ click_trans_id: params.click_trans_id, merchant_trans_id: params.merchant_trans_id, error: CLICK_ERRORS.SYSTEM_ERROR, error_note: 'System error' }) | `activation-system/controllers/payment/clickController.js:107` |


### controllers/payment/paymeController.js :: CancelTransaction

`activation-system/controllers/payment/paymeController.js:280`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| throw new PaymeError(PAYME_ERRORS.TRANSACTION_NOT_FOUND); | `activation-system/controllers/payment/paymeController.js:288` |
| !attempt | `activation-system/controllers/payment/paymeController.js:287` |
| attempt.status === 'FAILED' | `activation-system/controllers/payment/paymeController.js:297` |


Возвращаемые JSON-RPC result:

| Структура | Источник |
| --- | --- |
| { transaction: attempt.id.toString(), cancel_time: attempt.cancelTime ? attempt.cancelTime.getTime() : now.getTime(), state: attempt.paidAt ? -2 : -1 } | `activation-system/controllers/payment/paymeController.js:298` |
| { transaction: updated.id.toString(), cancel_time: now.getTime(), state: attempt.paidAt ? -2 : -1 } | `activation-system/controllers/payment/paymeController.js:317` |


### controllers/payment/paymeController.js :: CheckPerformTransaction

`activation-system/controllers/payment/paymeController.js:83`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| throw new PaymeError(PAYME_ERRORS.ORDER_NOT_FOUND, 'Namo'); | `activation-system/controllers/payment/paymeController.js:89` |
| !account &#124;&#124; !orderIdRaw | `activation-system/controllers/payment/paymeController.js:88` |
| isNaN(orderId) | `activation-system/controllers/payment/paymeController.js:94` |
| !attempt | `activation-system/controllers/payment/paymeController.js:103` |
| throw new PaymeError(PAYME_ERRORS.WRONG_AMOUNT); | `activation-system/controllers/payment/paymeController.js:110` |
| amount !== expectedAmountInTiyin | `activation-system/controllers/payment/paymeController.js:109` |
| throw new PaymeError(PAYME_ERRORS.ORDER_ALREADY_PAID); | `activation-system/controllers/payment/paymeController.js:115` |
| attempt.status === 'PAID' | `activation-system/controllers/payment/paymeController.js:114` |
| throw new PaymeError(PAYME_ERRORS.ORDER_CANCELLED); | `activation-system/controllers/payment/paymeController.js:119` |
| attempt.status === 'FAILED' &#124;&#124; attempt.status === 'EXPIRED' | `activation-system/controllers/payment/paymeController.js:118` |


Возвращаемые JSON-RPC result:

| Структура | Источник |
| --- | --- |
| { allow: true } | `activation-system/controllers/payment/paymeController.js:122` |


### controllers/payment/paymeController.js :: CheckTransaction

`activation-system/controllers/payment/paymeController.js:327`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| throw new PaymeError(PAYME_ERRORS.TRANSACTION_NOT_FOUND); | `activation-system/controllers/payment/paymeController.js:335` |
| !attempt | `activation-system/controllers/payment/paymeController.js:334` |
| attempt.status === 'FAILED' &#124;&#124; attempt.status === 'EXPIRED' | `activation-system/controllers/payment/paymeController.js:343` |
| attempt.status === 'PROCESSING' | `activation-system/controllers/payment/paymeController.js:341` |
| attempt.status === 'PAID' | `activation-system/controllers/payment/paymeController.js:339` |


Возвращаемые JSON-RPC result:

| Структура | Источник |
| --- | --- |
| { create_time: attempt.createdAt.getTime(), perform_time: attempt.paidAt ? attempt.paidAt.getTime() : 0, cancel_time: attempt.cancelTime ? attempt.cancelTime.getTime() : 0, transaction: attempt.id.toString(), state, reason: attempt.cancelReason &#124;&#124; null } | `activation-system/controllers/payment/paymeController.js:350` |


### controllers/payment/paymeController.js :: CreateTransaction

`activation-system/controllers/payment/paymeController.js:128`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| throw new PaymeError(PAYME_ERRORS.ORDER_NOT_FOUND, 'Namo'); | `activation-system/controllers/payment/paymeController.js:134` |
| !account &#124;&#124; !orderIdRaw | `activation-system/controllers/payment/paymeController.js:133` |
| isNaN(orderId) | `activation-system/controllers/payment/paymeController.js:139` |
| throw new PaymeError(PAYME_ERRORS.CANNOT_PERFORM, 'timeout'); | `activation-system/controllers/payment/paymeController.js:145` |
| Date.now() - time > 43200000 | `activation-system/controllers/payment/paymeController.js:144` |
| existing | `activation-system/controllers/payment/paymeController.js:153` |
| !attempt | `activation-system/controllers/payment/paymeController.js:167` |
| throw new PaymeError(PAYME_ERRORS.ORDER_HAS_TRANSACTION); | `activation-system/controllers/payment/paymeController.js:174` |
| attempt.externalPaymentId && attempt.externalPaymentId !== id | `activation-system/controllers/payment/paymeController.js:172` |
| throw new PaymeError(PAYME_ERRORS.WRONG_AMOUNT); | `activation-system/controllers/payment/paymeController.js:180` |
| amount !== expectedAmountInTiyin | `activation-system/controllers/payment/paymeController.js:179` |
| throw new PaymeError(PAYME_ERRORS.ORDER_ALREADY_PAID); | `activation-system/controllers/payment/paymeController.js:185` |
| attempt.status === 'PAID' | `activation-system/controllers/payment/paymeController.js:184` |
| throw new PaymeError(PAYME_ERRORS.ORDER_CANCELLED); | `activation-system/controllers/payment/paymeController.js:191` |
| attempt.status === 'FAILED' &#124;&#124; attempt.status === 'EXPIRED' | `activation-system/controllers/payment/paymeController.js:189` |


Возвращаемые JSON-RPC result:

| Структура | Источник |
| --- | --- |
| { create_time: existing.createdAt.getTime(), transaction: existing.id.toString(), state: existing.status === 'PAID' ? 2 : 1 } | `activation-system/controllers/payment/paymeController.js:155` |
| { create_time: updated.createdAt.getTime(), transaction: updated.id.toString(), state: 1 } | `activation-system/controllers/payment/paymeController.js:205` |


### controllers/payment/paymeController.js :: GetStatement

`activation-system/controllers/payment/paymeController.js:363`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| attempt.status === 'FAILED' &#124;&#124; attempt.status === 'EXPIRED' | `activation-system/controllers/payment/paymeController.js:388` |
| attempt.status === 'PROCESSING' | `activation-system/controllers/payment/paymeController.js:386` |
| attempt.status === 'PAID' | `activation-system/controllers/payment/paymeController.js:384` |


Возвращаемые JSON-RPC result:

| Структура | Источник |
| --- | --- |
| { id: attempt.externalPaymentId, time: attempt.createdAt.getTime(), amount: Math.round(attempt.amount * 100), account: { Namo: attempt.id.toString(), order_id: attempt.id.toString() }, create_time: attempt.createdAt.getTime(), perform_time: attempt.paidAt ? attempt.paidAt.getTime() : 0, cancel_time: attempt.cancelTime ? attempt.cancelTime.getTime() : 0, transaction: attempt.id.toString(), state, reason: attempt.cancelReason &#124;&#124; null } | `activation-system/controllers/payment/paymeController.js:394` |
| { transactions } | `activation-system/controllers/payment/paymeController.js:411` |


### controllers/payment/paymeController.js :: PerformTransaction

`activation-system/controllers/payment/paymeController.js:215`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| throw new PaymeError(PAYME_ERRORS.TRANSACTION_NOT_FOUND); | `activation-system/controllers/payment/paymeController.js:232` |
| !attempt | `activation-system/controllers/payment/paymeController.js:231` |
| attempt.status === 'PAID' && attempt.paidAt | `activation-system/controllers/payment/paymeController.js:236` |
| throw new PaymeError(PAYME_ERRORS.CANNOT_PERFORM, 'Transaction cancelled'); | `activation-system/controllers/payment/paymeController.js:247` |
| attempt.status === 'FAILED' &#124;&#124; attempt.status === 'EXPIRED' | `activation-system/controllers/payment/paymeController.js:246` |
| throw new PaymeError(PAYME_ERRORS.CANNOT_PERFORM, result.error); | `activation-system/controllers/payment/paymeController.js:260` |
| !result.success | `activation-system/controllers/payment/paymeController.js:258` |


Возвращаемые JSON-RPC result:

| Структура | Источник |
| --- | --- |
| { transaction: attempt.id.toString(), perform_time: attempt.paidAt.getTime(), state: 2 } | `activation-system/controllers/payment/paymeController.js:238` |
| { transaction: updated.id.toString(), perform_time: updated.paidAt.getTime(), state: 2 } | `activation-system/controllers/payment/paymeController.js:270` |


### controllers/payment/paymeController.js :: handlePayme

`activation-system/controllers/payment/paymeController.js:8`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { method, params, id } = req.body | `activation-system/controllers/payment/paymeController.js:9` |
| req.params.kassaId | `activation-system/controllers/payment/paymeController.js:15` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| throw new PaymeError(PAYME_ERRORS.SYSTEM_ERROR); | `activation-system/controllers/payment/paymeController.js:17` |
| !kassaId | `activation-system/controllers/payment/paymeController.js:16` |
| !kassa | `activation-system/controllers/payment/paymeController.js:20` |
| throw new PaymeError(PAYME_ERRORS.AUTH_ERROR); | `activation-system/controllers/payment/paymeController.js:26` |
| !checkPaymeAuth(req, kassa) | `activation-system/controllers/payment/paymeController.js:25` |
| throw new PaymeError(PAYME_ERRORS.METHOD_NOT_FOUND); | `activation-system/controllers/payment/paymeController.js:51` |
| error instanceof PaymeError | `activation-system/controllers/payment/paymeController.js:59` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.json({ result, id }) | `activation-system/controllers/payment/paymeController.js:54` |
| res.json({ error: { code: error.code, message: error.messageObj, data: error.data }, id }) | `activation-system/controllers/payment/paymeController.js:60` |
| res.json({ error: { code: PAYME_ERRORS.SYSTEM_ERROR.code, message: PAYME_ERRORS.SYSTEM_ERROR.message }, id }) | `activation-system/controllers/payment/paymeController.js:70` |


### controllers/public/qrPaymentController.js :: downloadReceipt

`activation-system/controllers/public/qrPaymentController.js:535`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { token, attemptId } = req.params | `activation-system/controllers/public/qrPaymentController.js:537` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !attempt &#124;&#124; attempt.status !== 'PAID' | `activation-system/controllers/public/qrPaymentController.js:546` |
| !receiptPath | `activation-system/controllers/public/qrPaymentController.js:553` |
| !fs.existsSync(absolutePath) | `activation-system/controllers/public/qrPaymentController.js:559` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).send('Чек недоступен') | `activation-system/controllers/public/qrPaymentController.js:547` |
| res.status(404).send('Чек не найден') | `activation-system/controllers/public/qrPaymentController.js:554` |
| res.status(404).send('Файл чека не найден') | `activation-system/controllers/public/qrPaymentController.js:560` |
| res.setHeader('Content-Type', 'application/pdf') | `activation-system/controllers/public/qrPaymentController.js:564` |
| res.setHeader('Content-Disposition', ˋattachment; filename=receipt-${attempt.id}.pdfˋ) | `activation-system/controllers/public/qrPaymentController.js:565` |
| res.status(500).send('Ошибка загрузки чека') | `activation-system/controllers/public/qrPaymentController.js:572` |


### controllers/public/qrPaymentController.js :: processCheckout

`activation-system/controllers/public/qrPaymentController.js:111`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { token } = req.params | `activation-system/controllers/public/qrPaymentController.js:113` |
| { phoneNumber, paymentMethod } = req.body | `activation-system/controllers/public/qrPaymentController.js:114` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phoneNumber &#124;&#124; !phoneRegex.test(phoneNumber) | `activation-system/controllers/public/qrPaymentController.js:118` |
| !paymentMethod &#124;&#124; !['click', 'payme', 'alif'].includes(paymentMethod) | `activation-system/controllers/public/qrPaymentController.js:126` |
| !link | `activation-system/controllers/public/qrPaymentController.js:141` |
| !alifToken | `activation-system/controllers/public/qrPaymentController.js:180` |
| !isAmountInAlifRange(priceInTiyin) | `activation-system/controllers/public/qrPaymentController.js:189` |
| !invoiceResult.success | `activation-system/controllers/public/qrPaymentController.js:212` |
| paymentMethod === 'alif' | `activation-system/controllers/public/qrPaymentController.js:174` |
| paymentMethod === 'payme' | `activation-system/controllers/public/qrPaymentController.js:170` |
| paymentMethod === 'click' | `activation-system/controllers/public/qrPaymentController.js:167` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(400).json({ success: false, error: 'Некорректный номер телефона' }) | `activation-system/controllers/public/qrPaymentController.js:119` |
| res.status(400).json({ success: false, error: 'Выберите способ оплаты' }) | `activation-system/controllers/public/qrPaymentController.js:127` |
| res.status(404).json({ success: false, error: 'Ссылка не найдена' }) | `activation-system/controllers/public/qrPaymentController.js:142` |
| res.status(400).json({ success: false, error: 'Alifpay не настроен для этой кассы' }) | `activation-system/controllers/public/qrPaymentController.js:185` |
| res.status(400).json({ success: false, error: 'Сумма вне допустимого диапазона Alifpay' }) | `activation-system/controllers/public/qrPaymentController.js:194` |
| res.status(502).json({ success: false, error: invoiceResult.error }) | `activation-system/controllers/public/qrPaymentController.js:218` |
| res.json({ success: true, redirectUrl: redirectUrl }) | `activation-system/controllers/public/qrPaymentController.js:233` |
| res.status(500).json({ success: false, error: 'Ошибка обработки' }) | `activation-system/controllers/public/qrPaymentController.js:240` |


### controllers/public/qrPaymentController.js :: showCheckoutPage

`activation-system/controllers/public/qrPaymentController.js:82`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { token } = req.params | `activation-system/controllers/public/qrPaymentController.js:84` |
| req.params.token | `activation-system/controllers/public/qrPaymentController.js:106` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !link &#124;&#124; !link.isActive | `activation-system/controllers/public/qrPaymentController.js:94` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect(ˋ/pay/${token}ˋ) | `activation-system/controllers/public/qrPaymentController.js:95` |
| res.render('pages/pay-checkout', { link, merchant: link.merchant, product: link.product, token }) | `activation-system/controllers/public/qrPaymentController.js:98` |
| res.redirect(ˋ/pay/${req.params.token}ˋ) | `activation-system/controllers/public/qrPaymentController.js:106` |


### controllers/public/qrPaymentController.js :: showProductPage

`activation-system/controllers/public/qrPaymentController.js:16`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { token } = req.params | `activation-system/controllers/public/qrPaymentController.js:18` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !link &#124;&#124; !link.isActive | `activation-system/controllers/public/qrPaymentController.js:33` |
| link.merchant.status !== 'active' | `activation-system/controllers/public/qrPaymentController.js:40` |
| link.product.status !== 'on' | `activation-system/controllers/public/qrPaymentController.js:47` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/pay-error', { error: 'link_invalid', message: 'Ссылка недействительна' }) | `activation-system/controllers/public/qrPaymentController.js:34` |
| res.render('pages/pay-error', { error: 'merchant_inactive', message: 'Продавец недоступен' }) | `activation-system/controllers/public/qrPaymentController.js:41` |
| res.render('pages/pay-error', { error: 'product_inactive', message: 'Товар снят с продажи' }) | `activation-system/controllers/public/qrPaymentController.js:48` |
| res.render('pages/pay-product', { link, merchant: link.merchant, product: link.product, hasVouchers: voucherCount > 0, oldPrice, discountPercent }) | `activation-system/controllers/public/qrPaymentController.js:64` |
| res.render('pages/pay-error', { error: 'server_error', message: 'Ошибка сервера' }) | `activation-system/controllers/public/qrPaymentController.js:74` |


### controllers/public/qrPaymentController.js :: showResultPage

`activation-system/controllers/public/qrPaymentController.js:478`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { token, attemptId } = req.params | `activation-system/controllers/public/qrPaymentController.js:480` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !attempt | `activation-system/controllers/public/qrPaymentController.js:495` |
| attempt.status !== 'PAID' | `activation-system/controllers/public/qrPaymentController.js:502` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/pay-error', { error: 'not_found', message: 'Запись не найдена' }) | `activation-system/controllers/public/qrPaymentController.js:496` |
| res.render('pages/pay-error', { error: attempt.status.toLowerCase(), message: errorMessages[attempt.status] &#124;&#124; 'Ошибка', token }) | `activation-system/controllers/public/qrPaymentController.js:510` |
| res.render('pages/pay-result', { attempt, merchant: attempt.link.merchant, product: attempt.link.product, sale: attempt.sale, voucherCode: attempt.voucherValue, token }) | `activation-system/controllers/public/qrPaymentController.js:517` |
| res.render('pages/pay-error', { error: 'server_error', message: 'Ошибка сервера' }) | `activation-system/controllers/public/qrPaymentController.js:527` |


### controllers/publicController.js :: handleEmailForm

`activation-system/controllers/publicController.js:12`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { email, phone } = req.body | `activation-system/controllers/publicController.js:13` |
| req.session.customer | `activation-system/controllers/publicController.js:20` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !email &#124;&#124; !phone | `activation-system/controllers/publicController.js:15` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/enter-email', { error: 'Введите email и номер телефона' }) | `activation-system/controllers/publicController.js:16` |
| res.redirect('/client/enter-voucher') | `activation-system/controllers/publicController.js:21` |


### controllers/publicController.js :: handleVoucherActivation

`activation-system/controllers/publicController.js:119`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.query.voucher | `activation-system/controllers/publicController.js:120` |
| req.session.user | `activation-system/controllers/publicController.js:126` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !code | `activation-system/controllers/publicController.js:122` |
| error instanceof ActivationError | `activation-system/controllers/publicController.js:147` |
| user?.vendorId && (user.role === 'vendor_user' &#124;&#124; user.role === 'vendor') | `activation-system/controllers/publicController.js:128` |
| !voucher | `activation-system/controllers/publicController.js:171` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.send('❌ Введите код ваучера в ссылке') | `activation-system/controllers/publicController.js:123` |
| res.render('pages/vendor/activate-qr-result', { user, result: { success: true, code: result.voucher.value, productName: result.productName, activationKey: result.activationKey, wasLinkedToClient: result.wasLinkedToClient, }, }) | `activation-system/controllers/publicController.js:136` |
| res.render('pages/vendor/activate-qr-result', { user, result: { success: false, message: error.message, }, }) | `activation-system/controllers/publicController.js:148` |
| res.render('pages/vendor/activate-qr-result', { user, result: { success: false, message: 'Не удалось активировать ваучер. Попробуйте ещё раз.', }, }) | `activation-system/controllers/publicController.js:157` |
| res.send('❌ Ваучер не найден') | `activation-system/controllers/publicController.js:172` |
| res.render('pages/activate-form', { voucherCode: code }) | `activation-system/controllers/publicController.js:176` |


### controllers/publicController.js :: handleVoucherSubmit

`activation-system/controllers/publicController.js:41`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { voucher } = req.body | `activation-system/controllers/publicController.js:42` |
| req.session.customer | `activation-system/controllers/publicController.js:43` |
| { name, email, phone } = req.session.customer &#124;&#124; {} | `activation-system/controllers/publicController.js:43` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !voucher &#124;&#124; !email &#124;&#124; !phone | `activation-system/controllers/publicController.js:45` |
| !voucherRecord &#124;&#124; voucherRecord.status !== 'active' | `activation-system/controllers/publicController.js:56` |
| voucherRecord.type === 'Telegram' | `activation-system/controllers/publicController.js:65` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/enter-voucher', { email, phone, name, error: 'Все поля обязательны' }) | `activation-system/controllers/publicController.js:46` |
| res.render('pages/activation-key', { error: 'Неверный или использованный ваучер', key: null }) | `activation-system/controllers/publicController.js:57` |
| res.render('pages/activation-key', { key: activationKey, error: null }) | `activation-system/controllers/publicController.js:92` |


### controllers/publicController.js :: showEmailForm

`activation-system/controllers/publicController.js:7`

Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/enter-email', { error: null }) | `activation-system/controllers/publicController.js:8` |


### controllers/publicController.js :: showVoucherForm

`activation-system/controllers/publicController.js:25`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.customer | `activation-system/controllers/publicController.js:26` |
| { email, phone } = req.session.customer &#124;&#124; {} | `activation-system/controllers/publicController.js:26` |
| req.session.customer.name | `activation-system/controllers/publicController.js:35` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !email &#124;&#124; !phone | `activation-system/controllers/publicController.js:28` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect('/email') | `activation-system/controllers/publicController.js:29` |
| res.render('pages/enter-voucher', { email, phone, name: req.session.customer.name &#124;&#124; '', error: null }) | `activation-system/controllers/publicController.js:32` |


### controllers/storeController.js :: activateVoucher

`activation-system/controllers/storeController.js:213`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { voucherCode, storeSlug } = req.body | `activation-system/controllers/storeController.js:214` |
| req.session.clientId | `activation-system/controllers/storeController.js:215` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !clientId | `activation-system/controllers/storeController.js:217` |
| !store | `activation-system/controllers/storeController.js:221` |
| failedAttempts >= 10 | `activation-system/controllers/storeController.js:240` |
| existingOrder && existingOrder.status === 'COMPLETED' | `activation-system/controllers/storeController.js:285` |
| existingRequest && existingRequest.status === 'COMPLETED' | `activation-system/controllers/storeController.js:311` |
| voucher.product.vendor.productType === 'MANUAL' | `activation-system/controllers/storeController.js:309` |
| voucher.product.vendor.productType === 'MANASTORE' | `activation-system/controllers/storeController.js:299` |
| voucher.product.vendor.productType === 'DRWEB' | `activation-system/controllers/storeController.js:291` |
| voucher.product.vendor.productType === 'ROKKY' | `activation-system/controllers/storeController.js:283` |
| voucher.type !== 'Vendor' | `activation-system/controllers/storeController.js:319` |
| voucher.status === 'pending' | `activation-system/controllers/storeController.js:280` |
| !['sold', 'pending'].includes(voucher.status) | `activation-system/controllers/storeController.js:276` |
| voucher.status === 'activated' | `activation-system/controllers/storeController.js:272` |
| voucher.status === 'active' | `activation-system/controllers/storeController.js:268` |
| voucher.product.storeId !== store.id | `activation-system/controllers/storeController.js:263` |
| !voucher | `activation-system/controllers/storeController.js:259` |
| error | `activation-system/controllers/storeController.js:324` |
| existingBinding && existingBinding.clientId !== clientId | `activation-system/controllers/storeController.js:341` |
| !existingBinding | `activation-system/controllers/storeController.js:346` |
| !manualRequest | `activation-system/controllers/storeController.js:373` |
| throw new Error('Product missing Rokky SKU'); | `activation-system/controllers/storeController.js:421` |
| !sku | `activation-system/controllers/storeController.js:421` |
| rokkyOrder.status === 'COMPLETED' && rokkyOrder.key | `activation-system/controllers/storeController.js:464` |
| throw new Error('Product missing DrWeb SKU'); | `activation-system/controllers/storeController.js:491` |
| drwebOrder.status === 'COMPLETED' && drwebOrder.key | `activation-system/controllers/storeController.js:534` |
| content.ready && content.codes.length > 0 | `activation-system/controllers/storeController.js:577` |
| manastoreOrder.status === 'COMPLETED' && !manastoreOrder.key | `activation-system/controllers/storeController.js:575` |
| throw new Error('Product missing ManaStore variant id'); | `activation-system/controllers/storeController.js:592` |
| !variantId | `activation-system/controllers/storeController.js:592` |
| manastoreOrder.status === 'COMPLETED' | `activation-system/controllers/storeController.js:622` |
| existingOrder && existingOrder.status === 'PENDING' | `activation-system/controllers/storeController.js:560` |
| manastoreOrder.status === 'COMPLETED' && manastoreOrder.key | `activation-system/controllers/storeController.js:656` |
| voucher.product.vendor.productType === 'VOUCHER' | `activation-system/controllers/storeController.js:677` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ error: 'Unauthorized' }) | `activation-system/controllers/storeController.js:217` |
| res.status(404).json({ error: 'Store not found' }) | `activation-system/controllers/storeController.js:221` |
| res.status(429).json({ error: 'Произошла ошибка. Пожалуйста обратитесь в службу поддержки.' }) | `activation-system/controllers/storeController.js:241` |
| res.status(400).json({ error: 'Произошла ошибка. Пожалуйста обратитесь в службу поддержки.' }) | `activation-system/controllers/storeController.js:336` |
| res.json({ success: true, message: 'Ваша заявка принята. Ожидайте SMS с ключом активации в течение нескольких минут.', pending: true }) | `activation-system/controllers/storeController.js:400` |
| res.status(500).json({ error: 'Произошла ошибка. Пожалуйста обратитесь в службу поддержки.' }) | `activation-system/controllers/storeController.js:412` |
| res.json({ success: true, key: rokkyOrder.key }) | `activation-system/controllers/storeController.js:480` |
| res.json({ success: true, message: 'Activation in progress. Please check back later.' }) | `activation-system/controllers/storeController.js:484` |
| res.json({ success: true, key: drwebOrder.key }) | `activation-system/controllers/storeController.js:549` |
| res.json({ success: true, key: manastoreOrder.key }) | `activation-system/controllers/storeController.js:671` |
| res.json({ success: true, message: 'Activation in progress. Please check back later.', pending: true }) | `activation-system/controllers/storeController.js:675` |
| res.json({ success: true, message: 'Voucher activated successfully', voucher: true }) | `activation-system/controllers/storeController.js:699` |
| res.status(500).json({ error: 'Internal Server Error' }) | `activation-system/controllers/storeController.js:713` |


### controllers/storeController.js :: generateOtp

`activation-system/controllers/storeController.js:10`

### controllers/storeController.js :: getHistoryStatus

`activation-system/controllers/storeController.js:725`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { storeSlug } = req.params | `activation-system/controllers/storeController.js:726` |
| req.session.clientId | `activation-system/controllers/storeController.js:727` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !clientId | `activation-system/controllers/storeController.js:729` |
| !store | `activation-system/controllers/storeController.js:733` |
| item.voucher.manualActivationRequest && item.voucher.manualActivationRequest.key | `activation-system/controllers/storeController.js:765` |
| item.voucher.manastoreOrder && item.voucher.manastoreOrder.key | `activation-system/controllers/storeController.js:763` |
| item.voucher.drwebOrder && item.voucher.drwebOrder.key | `activation-system/controllers/storeController.js:761` |
| item.voucher.rokkyOrder && item.voucher.rokkyOrder.key | `activation-system/controllers/storeController.js:759` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(401).json({ error: 'Unauthorized' }) | `activation-system/controllers/storeController.js:729` |
| res.status(404).json({ error: 'Store not found' }) | `activation-system/controllers/storeController.js:733` |
| res.json({ success: true, history: result }) | `activation-system/controllers/storeController.js:771` |
| res.status(500).json({ error: 'Internal Server Error' }) | `activation-system/controllers/storeController.js:774` |


### controllers/storeController.js :: getStorePage

`activation-system/controllers/storeController.js:12`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { storeSlug } = req.params | `activation-system/controllers/storeController.js:13` |
| req.session.clientId | `activation-system/controllers/storeController.js:30` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !store &#124;&#124; !store.isActive | `activation-system/controllers/storeController.js:21` |
| req.session.clientId | `activation-system/controllers/storeController.js:30` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(404).render('store/login', { store: null, error: 'Магазин не найден', layout: 'store/layout' }) | `activation-system/controllers/storeController.js:22` |
| res.render('store/activate', { store, client, history, layout: 'store/layout' }) | `activation-system/controllers/storeController.js:57` |
| res.render('store/login', { store, layout: 'store/layout' }) | `activation-system/controllers/storeController.js:66` |
| res.status(500).render('store/login', { store: null, error: 'Произошла ошибка', layout: 'store/layout' }) | `activation-system/controllers/storeController.js:73` |


### controllers/storeController.js :: logout

`activation-system/controllers/storeController.js:717`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { storeSlug } = req.params | `activation-system/controllers/storeController.js:718` |
| req.session.clientId | `activation-system/controllers/storeController.js:719` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.redirect(ˋ/${storeSlug}ˋ) | `activation-system/controllers/storeController.js:720` |


### controllers/storeController.js :: sendOtp

`activation-system/controllers/storeController.js:81`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { phone, storeSlug } = req.body | `activation-system/controllers/storeController.js:82` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !phone | `activation-system/controllers/storeController.js:86` |
| !result.success | `activation-system/controllers/storeController.js:124` |
| !isDemo | `activation-system/controllers/storeController.js:111` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(400).json({ error: 'Phone is required' }) | `activation-system/controllers/storeController.js:86` |
| res.status(500).json({ error: 'Failed to send SMS' }) | `activation-system/controllers/storeController.js:125` |
| res.json({ success: true, message: 'OTP sent' }) | `activation-system/controllers/storeController.js:140` |
| res.status(500).json({ error: 'Internal Server Error' }) | `activation-system/controllers/storeController.js:144` |


### controllers/storeController.js :: verifyOtp

`activation-system/controllers/storeController.js:148`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| { phone, code, storeSlug } = req.body | `activation-system/controllers/storeController.js:149` |
| req.session.clientId | `activation-system/controllers/storeController.js:202` |
| req.session.save | `activation-system/controllers/storeController.js:203` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !isDemo | `activation-system/controllers/storeController.js:171` |
| !log | `activation-system/controllers/storeController.js:182` |
| isDemo | `activation-system/controllers/storeController.js:196` |
| !client | `activation-system/controllers/storeController.js:194` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(400).json({ error: 'Invalid or expired code' }) | `activation-system/controllers/storeController.js:183` |
| res.json({ success: true, redirectUrl: ˋ/${storeSlug}ˋ }) | `activation-system/controllers/storeController.js:205` |
| res.status(500).json({ error: 'Internal Server Error' }) | `activation-system/controllers/storeController.js:209` |


### controllers/vendor/activationController.js :: buildViewModel

`activation-system/controllers/vendor/activationController.js:3`

### controllers/vendor/activationController.js :: handleActivation

`activation-system/controllers/vendor/activationController.js:19`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/vendor/activationController.js:20` |
| req.body?.code | `activation-system/controllers/vendor/activationController.js:27` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !vendorId | `activation-system/controllers/vendor/activationController.js:23` |
| !code | `activation-system/controllers/vendor/activationController.js:29` |
| error instanceof ActivationError | `activation-system/controllers/vendor/activationController.js:55` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(403).send('Не удалось определить вендора для сессии') | `activation-system/controllers/vendor/activationController.js:24` |
| res.render('pages/vendor/activate', buildViewModel({ user, form: { code }, error: 'Введите код ваучера', })) | `activation-system/controllers/vendor/activationController.js:30` |
| res.render('pages/vendor/activate', buildViewModel({ user, success: successPayload, })) | `activation-system/controllers/vendor/activationController.js:50` |
| res.render('pages/vendor/activate', buildViewModel({ user, form: { code }, error: error.message, })) | `activation-system/controllers/vendor/activationController.js:56` |
| res.render('pages/vendor/activate', buildViewModel({ user, form: { code }, error: 'Не удалось активировать ваучер. Попробуйте ещё раз или обратитесь к администратору.', })) | `activation-system/controllers/vendor/activationController.js:63` |


### controllers/vendor/activationController.js :: showActivationForm

`activation-system/controllers/vendor/activationController.js:14`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/vendor/activationController.js:15` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.render('pages/vendor/activate', buildViewModel({ user })) | `activation-system/controllers/vendor/activationController.js:16` |


### controllers/vendor/dashboardController.js :: formatChartLabel

`activation-system/controllers/vendor/dashboardController.js:21`

### controllers/vendor/dashboardController.js :: maskVoucherValue

`activation-system/controllers/vendor/dashboardController.js:5`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| raw.length <= 4 | `activation-system/controllers/vendor/dashboardController.js:7` |


### controllers/vendor/dashboardController.js :: showDashboard

`activation-system/controllers/vendor/dashboardController.js:25`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/vendor/dashboardController.js:26` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !vendorId | `activation-system/controllers/vendor/dashboardController.js:29` |
| trendMap.has(key) | `activation-system/controllers/vendor/dashboardController.js:132` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(403).send('Не удалось определить вендора для сессии') | `activation-system/controllers/vendor/dashboardController.js:30` |
| res.render('pages/vendor/dashboard', { user, vendor, metrics, activationTrend, recentActivations, pendingQueue: sanitizedQueue, trendRange: TREND_DAYS, }) | `activation-system/controllers/vendor/dashboardController.js:157` |
| res.render('pages/vendor/dashboard', { user, vendor: null, metrics: { balance: 0, pendingCount: 0, soldCount: 0, activationsLast7Days: 0, totalActivated: 0, }, activationTrend: [], recentActivations: [], pendingQueue: [], trendRange: TREND_DAYS, error: 'Не удалось загрузить данные дашборда', }) | `activation-system/controllers/vendor/dashboardController.js:168` |


### controllers/vendor/dashboardController.js :: startOfDay

`activation-system/controllers/vendor/dashboardController.js:15`

### controllers/vendor/settingsController.js :: showSettings

`activation-system/controllers/vendor/settingsController.js:4`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/vendor/settingsController.js:5` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !vendorId | `activation-system/controllers/vendor/settingsController.js:8` |
| vendor?.receiptTemplate | `activation-system/controllers/vendor/settingsController.js:55` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(403).send('Не удалось определить вендора для сессии') | `activation-system/controllers/vendor/settingsController.js:9` |
| res.render('pages/vendor/settings', { user, vendor, users, previewHtml, }) | `activation-system/controllers/vendor/settingsController.js:67` |
| res.render('pages/vendor/settings', { user, vendor: { name: '—', category: null, productType: null, description: null, balance: 0, defaultCommissionPercent: 0, }, users: [], previewHtml: '', error: 'Не удалось загрузить настройки вендора', }) | `activation-system/controllers/vendor/settingsController.js:75` |


### controllers/vendor/transactionController.js :: maskVoucherValue

`activation-system/controllers/vendor/transactionController.js:3`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| raw.length <= 4 | `activation-system/controllers/vendor/transactionController.js:5` |


### controllers/vendor/transactionController.js :: showTransactions

`activation-system/controllers/vendor/transactionController.js:13`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/vendor/transactionController.js:14` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !vendorId | `activation-system/controllers/vendor/transactionController.js:17` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(403).send('Не удалось определить вендора для сессии') | `activation-system/controllers/vendor/transactionController.js:18` |
| res.render('pages/vendor/transactions', { user, transactions: maskedTransactions, payments, summary, }) | `activation-system/controllers/vendor/transactionController.js:92` |
| res.render('pages/vendor/transactions', { user, transactions: [], payments: [], summary: { pendingAmount: 0, totalAmount: 0, lastPayment: null, }, error: 'Не удалось загрузить финансовые данные', }) | `activation-system/controllers/vendor/transactionController.js:100` |


### controllers/vendor/voucherController.js :: buildPagination

`activation-system/controllers/vendor/voucherController.js:16`

### controllers/vendor/voucherController.js :: listVouchers

`activation-system/controllers/vendor/voucherController.js:36`

Вход:

| Выражение в коде | Источник |
| --- | --- |
| req.session.user | `activation-system/controllers/vendor/voucherController.js:37` |
| { status = '', productId = '', search = '', page: rawPage = '1' } = req.query | `activation-system/controllers/vendor/voucherController.js:44` |


Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| !vendorId | `activation-system/controllers/vendor/voucherController.js:40` |
| status && STATUS_OPTIONS.includes(status) | `activation-system/controllers/vendor/voucherController.js:54` |
| !Number.isNaN(productIdNumber) | `activation-system/controllers/vendor/voucherController.js:60` |
| productId | `activation-system/controllers/vendor/voucherController.js:58` |
| search | `activation-system/controllers/vendor/voucherController.js:65` |


Ответы:

| Выражение в коде | Источник |
| --- | --- |
| res.status(403).send('Не удалось определить вендора для сессии') | `activation-system/controllers/vendor/voucherController.js:41` |
| res.render('pages/vendor/vouchers', { user, filters: { status, productId, search }, statusOptions: STATUS_OPTIONS, products, vouchers: sanitizedVouchers, pagination, }) | `activation-system/controllers/vendor/voucherController.js:128` |
| res.render('pages/vendor/vouchers', { user, filters: { status, productId, search }, statusOptions: STATUS_OPTIONS, products: [], vouchers: [], pagination: buildPagination(req, 0, page), error: 'Не удалось загрузить список ваучеров', }) | `activation-system/controllers/vendor/voucherController.js:138` |


### controllers/vendor/voucherController.js :: makeUrl

`activation-system/controllers/vendor/voucherController.js:20`

### controllers/vendor/voucherController.js :: maskVoucherValue

`activation-system/controllers/vendor/voucherController.js:6`

Проверки и ветвления:

| Выражение в коде | Источник |
| --- | --- |
| raw.length <= 4 | `activation-system/controllers/vendor/voucherController.js:8` |


## Неподключённый роутер

`routes/apiV1.js` не импортируется точкой входа. Его GET `/health`, GET `/wallet`, GET `/voucher/:id` и POST `/voucher/:id/events` не являются доступным API приложения; ответы содержат тестовые ваучеры. `activation-system/routes/apiV1.js:4` `activation-system/index.js:18`

## Структуры, собираемые вспомогательными функциями

`res.json(detail/result)` и render с переменной определяются следующими return-выражениями. Ссылки на модельные коллекции см. DATA_MODEL.

### controllers/client/api/voucherController.js :: formatSummary

| Возвращаемое значение | Источник |
| --- | --- |
| { id: raw.voucher.id, onlineVoucherId: raw.id, value: raw.voucher.value, displayValue: raw.voucher.value.slice(0, 4) + ' •••• ' + raw.voucher.value.slice(-4), status: raw.voucher.status, statusLabel: meta.label, statusColor: meta.color, productName: raw.voucher.product?.name &#124;&#124; raw.voucher.productName &#124;&#124; 'Ваучер', assignedAt: raw.assignedAt.toLocaleDateString('ru-RU'), assignedAtISO: raw.assignedAt.toISOString(), } | `activation-system/controllers/client/api/voucherController.js:66` |


### controllers/client/api/voucherController.js :: buildDetail

| Возвращаемое значение | Источник |
| --- | --- |
| base | `activation-system/controllers/client/api/voucherController.js:104` |
| { ...base, passSerial, qrUrl, qrPayload: qrUrl, barcodePayload: raw.voucher.value, qrDataUrl, barcodeDataUrl, terms: raw.voucher.product?.description &#124;&#124; 'Используйте этот ваучер согласно условиям продавца. Подробности уточняйте у поддержки.', brandColor: '#0A84FF', passUrl, } | `activation-system/controllers/client/api/voucherController.js:134` |


### services/walletActivationService.js :: resolveStatusPayload

| Возвращаемое значение | Источник |
| --- | --- |
| { status: 'ready', key } | `activation-system/services/walletActivationService.js:67` |
| { status: 'pending', requestId: req.id } | `activation-system/services/walletActivationService.js:72` |
| { status: 'error', message: 'Заявка отклонена оператором. Обратитесь в поддержку.' } | `activation-system/services/walletActivationService.js:73` |
| { status: 'pending' } | `activation-system/services/walletActivationService.js:77` |
| { status: 'error', message: 'Поставщик временно недоступен' } | `activation-system/services/walletActivationService.js:78` |
| { status: 'error', message: 'Поставщик временно недоступен' } | `activation-system/services/walletActivationService.js:81` |
| { status: 'error', message: 'Поставщик временно недоступен' } | `activation-system/services/walletActivationService.js:84` |
| { status: 'idle' } | `activation-system/services/walletActivationService.js:87` |


### services/walletActivationService.js :: requestVoucherCode

| Возвращаемое значение | Источник |
| --- | --- |
| existing | `activation-system/services/walletActivationService.js:141` |
| existing | `activation-system/services/walletActivationService.js:145` |
| { status: 'pending', requestId: manualRequest.id } | `activation-system/services/walletActivationService.js:176` |
| { status: 'ready', key: order.key } | `activation-system/services/walletActivationService.js:202` |
| { status: 'pending' } | `activation-system/services/walletActivationService.js:204` |
| { status: 'ready', key: order.key } | `activation-system/services/walletActivationService.js:230` |
| { status: 'pending' } | `activation-system/services/walletActivationService.js:232` |
| { status: 'ready', key: order.key } | `activation-system/services/walletActivationService.js:325` |
| { status: 'pending' } | `activation-system/services/walletActivationService.js:327` |


### services/walletActivationService.js :: getVoucherCodeStatus

| Возвращаемое значение | Источник |
| --- | --- |
| resolveStatusPayload(online.voucher) | `activation-system/services/walletActivationService.js:343` |


### services/telegramService.js :: sendNotificationToBot

| Возвращаемое значение | Источник |
| --- | --- |
| { success: false, error: 'Bot not found or inactive' } | `activation-system/services/telegramService.js:44` |
| { success: false, error: 'No authorized users' } | `activation-system/services/telegramService.js:58` |
| { success: successCount > 0, results, successCount, totalCount: results.length } | `activation-system/services/telegramService.js:71` |
| { success: false, error: error.message } | `activation-system/services/telegramService.js:79` |


### controllers/vendor/activationController.js :: buildViewModel

| Возвращаемое значение | Источник |
| --- | --- |
| { user, form: { code: form.code ?? '', }, error, success, } | `activation-system/controllers/vendor/activationController.js:4` |


### controllers/vendor/voucherController.js :: buildPagination

| Возвращаемое значение | Источник |
| --- | --- |
| `${req.path}?${params.toString()}` | `activation-system/controllers/vendor/voucherController.js:22` |
| { page, totalPages, total, hasPrev: page > 1, hasNext: page < totalPages, prevUrl: page > 1 ? makeUrl(page - 1) : null, nextUrl: page < totalPages ? makeUrl(page + 1) : null, } | `activation-system/controllers/vendor/voucherController.js:25` |


## Коды ошибок Payme

Имена, числовые коды и локализованные сообщения объявлены здесь; HTTP-handler возвращает их в error, обычно без изменения HTTP status. `activation-system/controllers/payment/paymeController.js:59`

| Определение | Источник |
| --- | --- |
| export const CLICK_ERRORS = { | `activation-system/utils/payment/paymeErrors.js:4` |
| SUCCESS: 0, | `activation-system/utils/payment/paymeErrors.js:5` |
| SIGN_CHECK_FAILED: -1, | `activation-system/utils/payment/paymeErrors.js:6` |
| WRONG_AMOUNT: -2, | `activation-system/utils/payment/paymeErrors.js:7` |
| ACTION_NOT_FOUND: -3, | `activation-system/utils/payment/paymeErrors.js:8` |
| ALREADY_PAID: -4, | `activation-system/utils/payment/paymeErrors.js:9` |
| ORDER_NOT_FOUND: -5, | `activation-system/utils/payment/paymeErrors.js:10` |
| TRANSACTION_NOT_FOUND: -6, | `activation-system/utils/payment/paymeErrors.js:11` |
| CANCEL_ERROR: -7, | `activation-system/utils/payment/paymeErrors.js:12` |
| ACTION_DISABLED: -8, | `activation-system/utils/payment/paymeErrors.js:13` |
| CANCELLED: -9, | `activation-system/utils/payment/paymeErrors.js:14` |
| SYSTEM_ERROR: -10 | `activation-system/utils/payment/paymeErrors.js:15` |
| }; | `activation-system/utils/payment/paymeErrors.js:16` |
| export const PAYME_ERRORS = { | `activation-system/utils/payment/paymeErrors.js:22` |
| TRANSPORT_ERROR: { | `activation-system/utils/payment/paymeErrors.js:24` |
| code: -32300, | `activation-system/utils/payment/paymeErrors.js:25` |
| message: { | `activation-system/utils/payment/paymeErrors.js:26` |
| uz: "Transport xatosi", | `activation-system/utils/payment/paymeErrors.js:27` |
| ru: "Ошибка транспорта", | `activation-system/utils/payment/paymeErrors.js:28` |
| en: "Transport error" | `activation-system/utils/payment/paymeErrors.js:29` |
| } | `activation-system/utils/payment/paymeErrors.js:30` |
| }, | `activation-system/utils/payment/paymeErrors.js:31` |
| SYSTEM_ERROR: { | `activation-system/utils/payment/paymeErrors.js:33` |
| code: -32400, | `activation-system/utils/payment/paymeErrors.js:34` |
| message: { | `activation-system/utils/payment/paymeErrors.js:35` |
| uz: "Tizim xatosi", | `activation-system/utils/payment/paymeErrors.js:36` |
| ru: "Системная ошибка", | `activation-system/utils/payment/paymeErrors.js:37` |
| en: "System error" | `activation-system/utils/payment/paymeErrors.js:38` |
| } | `activation-system/utils/payment/paymeErrors.js:39` |
| }, | `activation-system/utils/payment/paymeErrors.js:40` |
| METHOD_NOT_FOUND: { | `activation-system/utils/payment/paymeErrors.js:41` |
| code: -32601, | `activation-system/utils/payment/paymeErrors.js:42` |
| message: { | `activation-system/utils/payment/paymeErrors.js:43` |
| uz: "Metod topilmadi", | `activation-system/utils/payment/paymeErrors.js:44` |
| ru: "Метод не найден", | `activation-system/utils/payment/paymeErrors.js:45` |
| en: "Method not found" | `activation-system/utils/payment/paymeErrors.js:46` |
| } | `activation-system/utils/payment/paymeErrors.js:47` |
| }, | `activation-system/utils/payment/paymeErrors.js:48` |
| AUTH_ERROR: { | `activation-system/utils/payment/paymeErrors.js:49` |
| code: -32504, | `activation-system/utils/payment/paymeErrors.js:50` |
| message: { | `activation-system/utils/payment/paymeErrors.js:51` |
| uz: "Ushbu usulni bajarishga ruxsat yo'q", | `activation-system/utils/payment/paymeErrors.js:52` |
| ru: "Недостаточно прав для выполнения метода", | `activation-system/utils/payment/paymeErrors.js:53` |
| en: "Insufficient privilege to perform this method" | `activation-system/utils/payment/paymeErrors.js:54` |
| } | `activation-system/utils/payment/paymeErrors.js:55` |
| }, | `activation-system/utils/payment/paymeErrors.js:56` |
| WRONG_AMOUNT: { | `activation-system/utils/payment/paymeErrors.js:58` |
| code: -31001, | `activation-system/utils/payment/paymeErrors.js:59` |
| message: { | `activation-system/utils/payment/paymeErrors.js:60` |
| uz: "Noto'g'ri summa", | `activation-system/utils/payment/paymeErrors.js:61` |
| ru: "Недопустимая сумма", | `activation-system/utils/payment/paymeErrors.js:62` |
| en: "Invalid amount" | `activation-system/utils/payment/paymeErrors.js:63` |
| } | `activation-system/utils/payment/paymeErrors.js:64` |
| }, | `activation-system/utils/payment/paymeErrors.js:65` |
| TRANSACTION_NOT_FOUND: { | `activation-system/utils/payment/paymeErrors.js:66` |
| code: -31003, | `activation-system/utils/payment/paymeErrors.js:67` |
| message: { | `activation-system/utils/payment/paymeErrors.js:68` |
| uz: "Tranzaksiya topilmadi", | `activation-system/utils/payment/paymeErrors.js:69` |
| ru: "Транзакция не найдена", | `activation-system/utils/payment/paymeErrors.js:70` |
| en: "Transaction not found" | `activation-system/utils/payment/paymeErrors.js:71` |
| } | `activation-system/utils/payment/paymeErrors.js:72` |
| }, | `activation-system/utils/payment/paymeErrors.js:73` |
| CANNOT_PERFORM: { | `activation-system/utils/payment/paymeErrors.js:74` |
| code: -31008, | `activation-system/utils/payment/paymeErrors.js:75` |
| message: { | `activation-system/utils/payment/paymeErrors.js:76` |
| uz: "Operatsiyani bajarib bo'lmadi", | `activation-system/utils/payment/paymeErrors.js:77` |
| ru: "Невозможно выполнить операцию", | `activation-system/utils/payment/paymeErrors.js:78` |
| en: "Unable to perform operation" | `activation-system/utils/payment/paymeErrors.js:79` |
| } | `activation-system/utils/payment/paymeErrors.js:80` |
| }, | `activation-system/utils/payment/paymeErrors.js:81` |
| CANNOT_CANCEL: { | `activation-system/utils/payment/paymeErrors.js:82` |
| code: -31007, | `activation-system/utils/payment/paymeErrors.js:83` |
| message: { | `activation-system/utils/payment/paymeErrors.js:84` |
| uz: "Tranzaksiyani bekor qilib bo'lmaydi", | `activation-system/utils/payment/paymeErrors.js:85` |
| ru: "Невозможно отменить транзакцию", | `activation-system/utils/payment/paymeErrors.js:86` |
| en: "Unable to cancel transaction" | `activation-system/utils/payment/paymeErrors.js:87` |
| } | `activation-system/utils/payment/paymeErrors.js:88` |
| }, | `activation-system/utils/payment/paymeErrors.js:89` |
| ORDER_NOT_FOUND: { | `activation-system/utils/payment/paymeErrors.js:91` |
| code: -31050, | `activation-system/utils/payment/paymeErrors.js:92` |
| message: { | `activation-system/utils/payment/paymeErrors.js:93` |
| uz: "Biz sizning hisobingizni topolmadik", | `activation-system/utils/payment/paymeErrors.js:94` |
| ru: "Мы не нашли вашу учетную запись", | `activation-system/utils/payment/paymeErrors.js:95` |
| en: "We couldn't find your account" | `activation-system/utils/payment/paymeErrors.js:96` |
| } | `activation-system/utils/payment/paymeErrors.js:97` |
| }, | `activation-system/utils/payment/paymeErrors.js:98` |
| ORDER_CANCELLED: { | `activation-system/utils/payment/paymeErrors.js:99` |
| code: -31051, | `activation-system/utils/payment/paymeErrors.js:100` |
| message: { | `activation-system/utils/payment/paymeErrors.js:101` |
| uz: "Buyurtma bekor qilindi", | `activation-system/utils/payment/paymeErrors.js:102` |
| ru: "Заказ отменен", | `activation-system/utils/payment/paymeErrors.js:103` |
| en: "Order cancelled" | `activation-system/utils/payment/paymeErrors.js:104` |
| } | `activation-system/utils/payment/paymeErrors.js:105` |
| }, | `activation-system/utils/payment/paymeErrors.js:106` |
| ORDER_ALREADY_PAID: { | `activation-system/utils/payment/paymeErrors.js:107` |
| code: -31052, | `activation-system/utils/payment/paymeErrors.js:108` |
| message: { | `activation-system/utils/payment/paymeErrors.js:109` |
| uz: "Buyurtma allaqachon to'langan", | `activation-system/utils/payment/paymeErrors.js:110` |
| ru: "Заказ уже оплачен", | `activation-system/utils/payment/paymeErrors.js:111` |
| en: "Order already paid" | `activation-system/utils/payment/paymeErrors.js:112` |
| } | `activation-system/utils/payment/paymeErrors.js:113` |
| }, | `activation-system/utils/payment/paymeErrors.js:114` |
| ORDER_HAS_TRANSACTION: { | `activation-system/utils/payment/paymeErrors.js:115` |
| code: -31053, | `activation-system/utils/payment/paymeErrors.js:116` |
| message: { | `activation-system/utils/payment/paymeErrors.js:117` |
| uz: "Buyurtmada boshqa tranzaksiya mavjud", | `activation-system/utils/payment/paymeErrors.js:118` |
| ru: "По заказу уже есть другая транзакция", | `activation-system/utils/payment/paymeErrors.js:119` |
| en: "Order already has another transaction" | `activation-system/utils/payment/paymeErrors.js:120` |
| } | `activation-system/utils/payment/paymeErrors.js:121` |
| } | `activation-system/utils/payment/paymeErrors.js:122` |
| }; | `activation-system/utils/payment/paymeErrors.js:123` |
| export class PaymeError extends Error { | `activation-system/utils/payment/paymeErrors.js:128` |
| constructor(error, data = null) { | `activation-system/utils/payment/paymeErrors.js:129` |
| const msg = typeof error.message === 'object' ? error.message.en : error.message; | `activation-system/utils/payment/paymeErrors.js:130` |
| super(msg); | `activation-system/utils/payment/paymeErrors.js:131` |
| this.code = error.code; | `activation-system/utils/payment/paymeErrors.js:132` |
| this.messageObj = error.message; | `activation-system/utils/payment/paymeErrors.js:133` |
| this.data = data; | `activation-system/utils/payment/paymeErrors.js:134` |
| this.name = 'PaymeError'; | `activation-system/utils/payment/paymeErrors.js:135` |
| } | `activation-system/utils/payment/paymeErrors.js:136` |
| } | `activation-system/utils/payment/paymeErrors.js:137` |


## Проверки доступа и лимиты, общие для таблицы маршрутов

Отсутствие user в ensureAuthenticated даёт redirect /auth/login. Ролевые guards отвечают 403 текстом; authClient даёт 401 JSON для Accept application/json или originalUrl /api/*, иначе redirect /wallet. Имена middleware в строках маршрутов отсылают к следующей матрице.

| Middleware | Условие допуска | Отказ / источник |
| --- | --- | --- |
| ensureAuthenticated | req.session.user | redirect /auth/login; `activation-system/middleware/auth.js:2` |
| ensureAdmin | admin | 403 Access denied. Admins only.; `activation-system/middleware/auth.js:8` |
| allowOfficeDashboard | admin, financial_mgr, content_mgr, support_agent | 403 Access denied.; `activation-system/middleware/auth.js:16` |
| allowFinance | admin, financial_mgr, kassa_admin | 403 Finance role required; `activation-system/middleware/auth.js:23` |
| allowContent | admin, content_mgr | 403 Content role required; `activation-system/middleware/auth.js:30` |
| allowSupport | admin, support_agent | 403 Support role required; `activation-system/middleware/auth.js:37` |
| allowFinanceOrContent | admin, financial_mgr, content_mgr | 403 Finance or Content role required; `activation-system/middleware/auth.js:45` |
| ensureMerchant | merchant | 403 Merchants only; `activation-system/middleware/auth.js:51` |
| ensureVendor | vendor_user или vendor (последнего нет в enum) | 403 Vendors only; `activation-system/middleware/auth.js:56` |
| ensureKassa | kassa_admin/kassa_viewer и kassaId | 403 Kassa role required; `activation-system/middleware/auth.js:62` |
| ensureKassaAdmin | kassa_admin и kassaId | 403 Kassa admin role required; `activation-system/middleware/auth.js:69` |
| isClientAuthenticated | req.session.client | 401 {error: Unauthorized} либо redirect /wallet; `activation-system/middleware/authClient.js:1` |
| loginLimiter | 20 запросов / 15 минут | лимит задаётся в `activation-system/middleware/rateLimit.js:6` |
| activationLimiter | 10 запросов / час | лимит задаётся в `activation-system/middleware/rateLimit.js:17` |
| generationLimiter | 5 запросов / час | лимит задаётся в `activation-system/middleware/rateLimit.js:28` |
| otpVerifyLimiter | 5 запросов / 15 минут | лимит задаётся в `activation-system/middleware/rateLimit.js:38` |
| registerLimiter | 5 запросов / час | лимит задаётся в `activation-system/middleware/rateLimit.js:48` |


## Payme: параметры JSON-RPC

Общий body `{method, params, id}`; Basic Authorization проверяется по выбранной кассе. Все методы обслуживаются одним POST /api/payments/payme/:kassaId. `activation-system/controllers/payment/paymeController.js:8`

| method | params | Результат | Источник |
| --- | --- | --- | --- |
| CheckPerformTransaction | amount (целое, тийины), account.Namo или account.order_id | {allow:true} | `activation-system/controllers/payment/paymeController.js:83` |
| CreateTransaction | id, time (timestamp ms), amount, account.Namo/order_id | {create_time,transaction,state:1}; существующий может state:2 | `activation-system/controllers/payment/paymeController.js:128` |
| PerformTransaction | id внешней транзакции | {transaction,perform_time,state:2} | `activation-system/controllers/payment/paymeController.js:215` |
| CancelTransaction | id, reason | {transaction,cancel_time,state:-1 или -2} | `activation-system/controllers/payment/paymeController.js:280` |
| CheckTransaction | id | {create_time,perform_time,cancel_time,transaction,state,reason} | `activation-system/controllers/payment/paymeController.js:327` |
| GetStatement | from,to (временной интервал) | {transactions:[{id,time,amount,account:{Namo,order_id},create_time,perform_time,cancel_time,transaction,state,reason}]} | `activation-system/controllers/payment/paymeController.js:363` |


`account.Namo` читается до проверки наличия account в CheckPerform/CreateTransaction: отсутствующий account даёт исключение и общий SYSTEM_ERROR. `activation-system/controllers/payment/paymeController.js:87` `activation-system/controllers/payment/paymeController.js:132`.

Store history: `{success:true,history:[{id,key}]}`; key выбирается последовательно из Rokky/DrWeb/ManaStore/manual результата. `activation-system/controllers/storeController.js:749`.

## Multipart

Logo: поле logo, disk public/uploads, лимит 5 MiB. Product image: поле image, disk public/uploads/products, 5 MiB. Voucher import: поле voucherFile, memory storage, .txt/.csv, 2 MiB; дополнительно body productId/voucherList/type. Контроллер объединяет текст и файл, удаляет дубликаты, ограничивает число строк и createMany skipDuplicates. `activation-system/routes/adminRoutes.js:79` `activation-system/routes/adminRoutes.js:87` `activation-system/routes/adminRoutes.js:94` `activation-system/middleware/uploadMiddleware.js:35` `activation-system/middleware/uploadMiddleware.js:72` `activation-system/controllers/admin/voucherController.js:43`.
