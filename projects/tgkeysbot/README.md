# tgkeysbot

Telegram-бот и Django back-office для партнёров (дилеров) KeysPro.uz:
партнёр покупает лицензионные ключи (Microsoft Office/Windows, Dr.Web) в
Telegram-боте в счёт баланса, ключ доставляется автоматически — либо через
эмуляцию пользователя Telegram, отправляющего команды другому боту-
источнику, либо через внешний HTTP-сервис Dr.Web.

Это каталог документации в `my_office`. Исходный код — отдельно, вне этого
репозитория, см. [AGENTS.md](AGENTS.md).

## С чего начать

1. [AGENTS.md](AGENTS.md) — инструкции для агентов, где код и что нельзя делать.
2. [docs/STATUS.md](docs/STATUS.md) — текущее подтверждённое состояние.
3. [docs/OVERVIEW.md](docs/OVERVIEW.md) — что это за продукт и как работает поток заказа.
4. [docs/OPEN_QUESTIONS.md](docs/OPEN_QUESTIONS.md) — что не удалось подтвердить, включая находки безопасности.
