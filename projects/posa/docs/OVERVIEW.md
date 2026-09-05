# OVERVIEW — POSA

Последнее обновление: 2026-09-05 (первичное исследование, TASK-0001).

## Что это

Платформа дистрибуции цифровых товаров (игровых ключей, ваучеров, кодов
пополнения) под брендом **Namo** (`namo.uz` и поддомены). Два потока
пользователей:

1. **Мерчанты** (`merchant.namo.uz`) продают ваучеры конечным покупателям
   офлайн (с печатью чека и QR-кодом) или онлайн (по SMS/QR-платежу).
2. **Клиенты** (`wallet.namo.uz`) заходят на брендированные страницы магазинов
   или в PWA-кошелёк, авторизуются по номеру телефона (OTP по SMS) и
   активируют купленные ваучеры, получая цифровой код; код можно добавить в
   Apple/Google Wallet (pkpass).

Это подтверждено кодом: subdomain-based роутинг (`middleware/checkSubdomain.js`),
модели `Merchant`/`Client`/`Voucher`/`Sale` в Prisma-схеме, генерация pkpass в
`routes/wallet.js` (зависимость `passkit-generator`).

## Роли и поддомены (подтверждено кодом)

| Поддомен | Роль (`req.appRole`) | Назначение |
|---|---|---|
| `wallet.namo.uz` | `client` | Кошелёк клиента, активация ваучеров |
| `merchant.namo.uz` | `merchant` | Кабинет продавца: продажа, корзина, чеки |
| `vendor.namo.uz` | `vendor` | Кабинет поставщика товара (вендора) |
| `office.namo.uz` | `admin` | Админка/бэк-офис |
| `api.namo.uz` | — | Проксируется на тот же backend (порт 4001→4000), назначение по коду не обособлено отдельными роутами |
| прочее (`namo.uz` и др.) | `public` | Публичные страницы, ввод ваучера, форма email |

Источник: `middleware/checkSubdomain.js`, `/etc/nginx/sites-available/*`.

## Источники ваучеров/товаров (подтверждено `docs/BUSINESS_LOGIC.md` + Prisma-схемой)

Один и тот же `Product` может быть привязан к одному из типов вендора
(`VendorProductType` enum): `ROKKY`, `DRWEB`, `MANASTORE`, `MANUAL`, `VOUCHER`.
Для внешних API-поставщиков (Rokky, Dr.Web, ManaStore) в схеме есть
параллельные тройки моделей `*Order` / `*ApiLog` / `*Sku` — интеграция
запрашивает ключ у внешнего API в момент активации клиентом. Для `MANUAL`
создаётся `ManualActivationRequest`, обрабатываемая оператором вручную (с
уведомлением в Telegram через `TelegramBot`/`StoreTelegramBot`). Для `VOUCHER`
код уже загружен в БД заранее.

## Оплата (подтверждено кодом)

Поддержаны три платёжных провайдера как webhook-эндпоинты
(`controllers/payment/{clickController,paymeController,alifController}.js`,
смонтированы в `routes/paymentRoutes.js`), плюс собственный QR-платёж
(`routes/qrPaymentRoutes.js`, модели `MerchantProductLink`/`QrPaymentAttempt`).
Реквизиты провайдеров хранятся не только в `.env`, но и per-кассе в таблице
`Kassa` — см. [DATABASE.md](DATABASE.md).

## Что НЕ подтверждено кодом (было в других источниках)

- Проектная документация (`docs/copilot-instructions.md`,
  `.github/copilot-instructions.md`) упоминает "Python (Pyrogram) для одного
  Telegram-вендора" — в `/home/admin1/posa/` такого Python-модуля не найдено
  (нет каталога `telegram-key-bot/`, хотя он упомянут в `.gitignore`). Сам
  `docs/PRD.md` прямо утверждает обратное: "система не использует
  Python-скриптов для автоматизации Telegram". Расхождение зафиксировано в
  [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).
- `docs/PRD.md` датирован 2025-12-17 и описывает только интеграцию Rokky —
  на момент этого исследования в коде уже есть Dr.Web, ManaStore, Alif,
  QR-платежи и модуль "Касса" (`Kassa`), которых в PRD нет. PRD устарел
  относительно кода.
