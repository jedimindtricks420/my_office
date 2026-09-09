---
id: TASK-0003
title: "Contador: SEO-лендинги и полезные страницы на русском и узбекском"
status: in-progress
created: 2026-09-09
owner: azureuser
priority: high
type: implementation-spec
---

# TASK-0003 — SEO-страницы Contador на русском и узбекском

## Назначение и границы

Создать публичный раздел Contador для органического привлечения пользователей из Узбекистана: 30 тем × 2 языка = 60 целевых URL, включая главные страницы, каталоги, функциональные и аудиторные лендинги, инструкции, три калькулятора и интерактивный чек-лист. Если соответствующая страница уже существует, улучшить её и сохранить сильный существующий URL, а не создавать дубль.

Этот файл — готовое задание следующему агенту-разработчику. На этапе постановки выполнены исследование и подготовка ТЗ; реализация страниц НЕ начата. Статус задачи остаётся planned. Виральность, позиции и объём трафика не гарантируются: задача создаёт измеримую основу для поискового роста, самостоятельного использования материалов и добровольного распространения ссылок.

Рабочий каталог документации, заданный пользователем: `/home/azureuser/work/my_office`.
Файл задачи: `projects/contador/tasks/active/TASK-0003-seo-landings-ru-uz.md`.
Код в документации проекта находится отдельно: `/home/admin1/contador/`; доступный GitHub-репозиторий кода — `jedimindtricks420/contador_v1`. Перед реализацией сверить `git remote -v`, ветку и фактический каталог на сервере. Не считать каталог my_office каталогом исходников приложения.

В рамках реализации разрешены маркетинговые страницы, их контент, маршруты, стили, SEO, локальные калькуляторы, аналитика и необходимые изолированные изменения общей навигации. Не менять бухгалтерский движок, налоговые формулы продукта, БД, оплату и права доступа ради маркетинга. Деплой в production — отдельный шаг по правилам репозитория; подготовить проверяемый PR и инструкцию выпуска.

## 1. Что изучено и насколько подтверждено

Дата исследования: 9 сентября 2026 года.

- Прочитаны корневые [AGENTS.md](../../../../AGENTS.md), [CONTRIBUTING.md](../../../../CONTRIBUTING.md), [инструкции Contador](../../AGENTS.md), [STATUS](../../docs/STATUS.md), [OVERVIEW](../../docs/OVERVIEW.md), [OPEN_QUESTIONS](../../docs/OPEN_QUESTIONS.md), [VIEWS](../../docs/VIEWS.md), [DEPLOYMENT](../../docs/DEPLOYMENT.md) и [предыдущий аудит](../completed/TASK-0002-full-technical-reaudit.md).
- Снимок my_office: commit `0d63fab348046607e636be4bd64910c13477cfc6`.
- Непосредственно прочитаны файлы кода в contador_v1/main: `AGENTS.md`, `v2/package.json`, `v2/next.config.ts`, `v2/src/proxy.ts`, `v2/src/app/layout.tsx`, `v2/src/app/page.tsx`. SHA дерева/снимка, возвращённый GitHub: `ec57169e5d1e6d3a3dcbcae2c524cfd1dbf17ea5`. Версия сервера может отличаться.
- Код подтверждает Next.js 16.2.1 / React 19.2.4, `basePath: /v2`, статический `lang="ru"`, общий title «Contador v2», редирект корня v2 в dashboard и проверку сессии для большинства путей. Простое добавление SEO-страниц внутрь v2 не сделает их публичными.
- Функции ниже подтверждены документацией аудита 08.09.2026, но их бизнес-логика в этой задаче повторно целиком не аудировалась. Перед публикацией лендинга исполнитель обязан проверить соответствующий UI и сценарий.
- Доступ к production `contador.uz` через веб-инструмент получить не удалось. Текущая главная, её хостинг, индексируемые URL, robots.txt, метаданные и позиции не подтверждены. Ошибка получения страницы не доказывает, что сайт не работает.
- Главная `www.kontur-extern.ru` и несколько её разделов отвечали 403. Анализ Экстерна основан на доступных официальных страницах того же продукта на `kontur.ru/extern`. Не утверждать, что проведён полный crawl заблокированного домена.
- Объём поискового спроса, частотности и позиции конкурентов не измерены: доступа к их Search Console/аналитике нет. Семантика ниже — редакционная карта интентов, а приоритеты — продуктовые гипотезы.

### Подтверждённая продуктовая основа и ограничения обещаний

| Тема | Основание для лендинга | Ограничение |
| --- | --- | --- |
| Банковские выписки | Аудит описывает Excel и 1CClientBankExchange, дедупликацию, откат импорта | Не обещать API-подключение ко всем банкам или импорт любых Excel-файлов |
| Классификация | Правила и AI-классификатор через Azure OpenAI | Не обещать безошибочность, полностью автономного бухгалтера или фиксированную экономию |
| Проводки | Шаблоны, двойная запись, контроль равенства дебета и кредита | Не обещать поддержку любых хозяйственных операций |
| Закрытие периода | Семишаговый мастер, проверки, финализация | Не обещать закрытие за конкретное время или отсутствие бухгалтерской проверки |
| Отчёты | Баланс, P&L, ДДС, ОСВ, журнал, карточка счёта | Не подменять управленческую/учётную информацию обещанием принятия отчёта государством |
| Открытые позиции | Авансы, подотчёт, депозиты, займы и сроки | Не представлять как полнофункциональную CRM взыскания долгов |
| Совместная работа | Организации и роли OWNER/ADMIN/ACCOUNTANT | Не заявлять сертифицированную безопасность, неограниченные организации или места без проверки |
| Оплата | Подписки; Payme/Click/Alifpay описаны в аудите | Это способы оплаты самого сервиса, а не интеграция учёта продаж клиента |
| Налоговые функции | В аудите есть налоговые режимы и расчёт налога на прибыль | В стартовый набор не входят налоговые калькуляторы, автоматическая отправка в Soliq и юридические инструкции |

Не публиковать утверждения «заменяет 1С целиком», «официальный партнёр Soliq», «отправляет ЭСФ», «ведёт зарплату и склад», «работает на узбекском во всём кабинете», «поддержка 24/7», «бесплатно навсегда», если это не подтверждено. Маркетинговый перевод не равен локализации приложения. Не переносить в Contador отзывы, клиентские цифры, банковские интеграции и налоговые обещания конкурентов.

## 2. Подходы конкурентов и их адаптация

Это наблюдения о публичной структуре страниц; причинная связь с трафиком или ростом не доказана.

| Источник | Наблюдение | Конкретное решение для Contador |
| --- | --- | --- |
| [Экстерн: главная продукта](https://kontur.ru/extern) | Разделение по аудиториям и функциям, переходы к тарифам и материалам | Построить навигацию по задачам, добавить страницы бухгалтеру, руководителю и сервисной компании |
| [Экстерн: учёт](https://kontur.ru/extern/parts/uchet) | Отдельная продуктовая страница объясняет пригодность, сценарии и совместную работу | Один функциональный лендинг на один интент, показать входные данные и результат в интерфейсе |
| [Экстерн: справочная](https://kontur.ru/extern/spravka) | Тематические статьи с понятными вопросами и отдельными URL | Создать небольшую полезную базу инструкций с переходом к связанным функциям |
| [Kick: главная](https://www.kick.co/) | Короткое позиционирование, демонстрации интерфейса, CTA и отзывы | Первый экран с понятным результатом, настоящие скриншоты Contador и действие «Создать аккаунт» |
| [Kick: бухгалтеры](https://www.kick.co/accountants) | Отдельное предложение для профессиональной аудитории | Лендинг для бухгалтера: импорт → проверка классификации → проводки → отчёт |
| [Kick: тарифы](https://www.kick.co/pricing) | Тарифы связаны с составом возможностей и ограничениями | Показывать реальные цены и лимиты из единого подтверждённого источника |
| [Kick: материалы](https://www.kick.co/resources) | На доступной странице преобладают новости компании и клиентские истории | Не приписывать Kick массовую SEO-генерацию. Использовать его подход к продуктовой конверсии |
| [Kick: клиентский кейс](https://www.kick.co/resources/how-elevated-tax-strategies-10xd-their-practice-with-kick) | Кейс построен вокруг исходной проблемы, внедрения и заявленных результатов | Подготовить компонент для будущего подтверждённого кейса; до появления данных заменить примером на демонстрационных данных |

Архитектура «справочная + продуктовые страницы» вдохновлена Экстерном; подача продукта и аудиторные CTA — Kick. Три математических калькулятора и интерактивный чек-лист ниже — наша гипотеза привлечения и распространения, а не заявление о точно таких же инструментах у конкурентов. Тексты, код и визуальные материалы сделать оригинальными.

## 3. Порядок работы следующего агента

1. В `/home/azureuser/work/my_office` прочитать инструкции в указанном выше порядке и этот файл. Проверить активные задачи и отсутствие дублирующей SEO-работы.
2. Определить актуальный исходный репозиторий и каталог Contador. Прочитать применимые инструкции исходников, проверить ветку, чистоту рабочей копии, существующие маркетинговые маршруты и дизайн.
3. Проверить, какой сервис обслуживает корень contador.uz, существующую маршрутизацию и SEO. Использовать доступные конфиги и HTTP-запросы; не выводить секреты. Не останавливать legacy-контейнеры для исследования.
4. Создать ветку реализации. В журнале этой задачи записать commit исходников и карту «старый URL → новый/сохраняемый URL». Сначала сверить существующие страницы и redirects; массовый перенос URL не является целью.
5. Для каждой функциональной страницы выполнить минимальный сценарий на тестовых данных и записать evidence: путь к UI/коду, что реально доступно, ограничения, разрешённый CTA. Если функция отсутствует, пометить только её страницу blocked с причиной; остальные продолжать.
6. Реализовать основу, пилотную волну, проверить обе локали и публичный HTML. Затем доделать остальную матрицу, инструменты и аналитические события.
7. Передать PR с preview/скриншотами, таблицей проверок и инструкцией выпуска. Обновить STATUS и связанные документы только по фактически выполненным изменениям. Не закрывать задачу после одной пилотной волны.

## 4. URL, языки и архитектура

Целевой origin — `https://contador.uz`, после проверки действующего canonical-host. Новый публичный раздел: `/ru/` и `/uz/`, без `/v2` в маркетинговых URL. Кабинет сохраняет `/v2`.

Начальное архитектурное решение: использовать существующее приложение главной страницы, если его исходники доступны. Если такого поддерживаемого приложения нет, создать отдельное `marketing/` внутри исходного репозитория Contador: Next.js App Router, TypeScript, существующие визуальные токены, статически создаваемые публичные страницы. Выбор отражать ADR после проверки сервера. Новый CMS и отдельная БД не нужны.

Для отдельного marketing-приложения:

- `src/app/[locale]/[[...slug]]/page.tsx` — маршруты из белого списка manifest, статическая генерация, неизвестные locale/slug дают настоящий 404.
- `src/app/[locale]/layout.tsx` — корневой layout с корректным `html lang`; не вкладывать второй html в существующий root layout.
- `src/content/seo-pages.ts` — единый реестр тем/URL/связей/публикации.
- `src/content/ru/` и `src/content/uz/` — статические локализованные тексты.
- `src/components/marketing/` — Header, Footer, Hero, ProductScreenshot, FeatureSteps, FAQ, RelatedPages, CTA.
- `src/components/tools/` — клиентские интерактивные островки; основной текст остаётся в HTML.
- `src/app/sitemap.ts`, `src/app/robots.ts`, `public/og/`, `public/screenshots/`.
- На существующем хосте подготовить маршрутизацию к marketing только для утверждённых публичных префиксов, robots/sitemap и его assets. Не использовать новый catch-all, который заберёт `/v2`, admin, callback оплаты или действующие пути.
- `/v2`, его `basePath`, `assetPrefix` и auth proxy не менять ради публичного маркетинга. Приложение marketing не импортирует Prisma, пользовательские сессии и компоненты кабинета с fetch приватных данных.

Для существующего публичного приложения сохранить его стек и структуру; логические требования к manifest/HTML/локалям те же. Указанные пути marketing являются целевой схемой для варианта нового приложения, а не утверждением об их наличии.

Корень `/`: сохранить существующее поведение до аудита. Если главная уже индексируется и соответствует RU-home, использовать её вместо дубля `/ru/` и изменить manifest/hreflang/ссылки. Если корень свободен для нового решения — настроить один постоянный redirect в `/ru/`, обе языковые домашние страницы self-canonical. Выбранный вариант зафиксировать до кодинга; не дублировать главный интент на двух индексируемых URL.

Правила локализации:

- RU — русский для Узбекистана; UZ — узбекский на латинице. `lang="ru"` / `lang="uz"`; `hreflang="ru-UZ"` / `hreflang="uz-UZ"`.
- Переключатель языка ведёт на перевод той же темы по ID из manifest, а не всегда на главную.
- Не применять принудительные IP/Accept-Language-редиректы: оба языка открываются напрямую.
- Canonical каждой языковой страницы указывает на неё саму. Hreflang содержит обе версии и обратные ссылки с абсолютными URL. `x-default` — RU-версия той же темы; если используется самостоятельный языковой selector, x-default домашней пары указывает на него.
- Тексты, title, description, H1/H2, FAQ, alt, формы, валидация и кнопки переведены. Не выпускать узбекский URL с русским fallback-текстом.
- Терминология: бухгалтерский учёт — buxgalteriya hisobi; выписка — bank ko‘chirmasi; проводка — buxgalteriya o‘tkazmasi; закрытие месяца — oyni yopish; ОСВ — aylanma-saldo qaydnomasi; ДДС — pul oqimi; начальные остатки — boshlang‘ich qoldiqlar.
- В URL — ASCII, lowercase, дефисы, без апострофов и лет. В тексте единообразно использовать o‘ и g‘. Суммы форматировать по локали; в примерах UZS/сум/so‘m.
- Узбекский контент обязан пройти языковую проверку компетентным редактором перед production. До проверки хранить как draft; не считать технический перевод подтверждением качества.

Основание hreflang: [Google Search Central: локализованные версии](https://developers.google.com/search/docs/specialty/international/localized-versions).

## 5. Матрица страниц и точные SEO-поля

Ниже 30 тем, у каждой два URL и готовые title, description, H1. Ключевые фразы — основной интент, а не список для тега meta keywords. Теги meta keywords не добавлять.

Title и description — стартовые редакционные варианты. Можно сократить после проверки ширины сниппета, сохраняя интент и смысл; не добавлять обещания. Ориентир title 45–65, description 130–170 знаков — редакционный ориентир, не лимит поисковика и не обязательное количество символов. Все метаданные уникальны. Один H1 на страницу — требование проекта.

P0 — пилот и основа: 01–08, 14–15, 17–18, 26.
P1 — расширение: 09–13, 16, 19–25, 27–30.
P0/P1 вместе образуют полное задание. Страница тарифов 17 публикуется только после проверки фактических тарифов; другие отсутствующие функции блокируют только соответствующие карточки, с явным отчётом.

### 01. Главная

Тип: home. Связанные темы: 02, 14, 15, 17.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/ | /uz/ |
| Title | Онлайн-бухгалтерия в Узбекистане \| Contador | O‘zbekistonda onlayn buxgalteriya \| Contador |
| H1 | Бухгалтерский учёт для бизнеса в Узбекистане | O‘zbekistondagi biznes uchun buxgalteriya hisobi |
| Description | Ведите учёт в Contador: загружайте банковские выписки, проверяйте операции и формируйте отчёты. Познакомьтесь с возможностями сервиса для вашего бизнеса. | Contador bilan bank ko‘chirmalarini yuklang, operatsiyalarni tekshiring va hisobotlarni shakllantiring. Biznesingiz uchun hisob yuritish imkoniyatlarini ko‘ring. |
| Основной запрос | онлайн бухгалтерия Узбекистан | onlayn buxgalteriya O‘zbekiston |
| H2 по порядку | От выписки к отчёту; Возможности для вашей работы; Как начать | Ko‘chirmadan hisobotgacha; Ishingiz uchun imkoniyatlar; Ishni qanday boshlash mumkin |

Обязательное содержание: Показать три шага на реальном UI: импорт, проверка, отчёт. Сделать карточки двух аудиторий и трёх основных функций, ссылку на тарифы. Не создавать отдельный конкурирующий лендинг «онлайн-бухгалтерия».

### 02. Каталог функций

Тип: hub. Связанные темы: 03, 04, 06, 07.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/ | /uz/imkoniyatlar/ |
| Title | Возможности бухгалтерской программы \| Contador | Buxgalteriya dasturi imkoniyatlari \| Contador |
| H1 | Возможности Contador для бухгалтерского учёта | Contador buxgalteriya imkoniyatlari |
| Description | Изучите возможности Contador: импорт выписок, классификация операций, проводки, закрытие месяца и отчёты. Выберите инструмент для своей задачи. | Bank ko‘chirmalari importi, operatsiyalar tasnifi, o‘tkazmalar, oyni yopish va hisobotlar bilan tanishing. Vazifangizga mos Contador imkoniyatini tanlang. |
| Основной запрос | возможности Contador | Contador imkoniyatlari |
| H2 по порядку | Подготовка данных; Учёт и проверки; Отчёты | Ma’lumotlarni tayyorlash; Hisob va tekshiruvlar; Hisobotlar |

Обязательное содержание: Сгруппировать 03–13 по этапам работы. Каждая карточка имеет короткое объяснение результата и обычную HTML-ссылку. Не копировать объёмные тексты дочерних страниц.

### 03. Импорт выписок

Тип: feature. Связанные темы: 19, 04, 06.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/import-bankovskih-vypisok/ | /uz/imkoniyatlar/bank-kochirmalarini-import-qilish/ |
| Title | Импорт банковских выписок в бухгалтерию \| Contador | Bank ko‘chirmalarini import qilish \| Contador |
| H1 | Загружайте банковские выписки в Contador | Bank ko‘chirmalarini Contador’ga yuklang |
| Description | Импортируйте поддерживаемые выписки Excel и 1CClientBankExchange в Contador. Проверьте операции после загрузки и подготовьте данные для дальнейшего учёта. | Qo‘llab-quvvatlanadigan Excel va 1CClientBankExchange ko‘chirmalarini Contador’ga yuklang. Operatsiyalarni tekshirib, keyingi hisob bosqichiga tayyorlang. |
| Основной запрос | импорт банковской выписки Excel | bank ko‘chirmasini import qilish |
| H2 по порядку | Какие файлы поддерживаются; Загрузка и проверка операций; Что делать с повторным импортом | Qaysi fayllar qo‘llab-quvvatlanadi; Yuklash va tekshirish; Takroriy import bilan ishlash |

Обязательное содержание: Показать поддерживаемый образец с вымышленными строками и названия фактических полей. Снять UI импорта. Объяснить ограничения формата, дедупликацию и откат только после проверки сценария.

### 04. AI-классификация

Тип: feature. Связанные темы: 20, 05, 03.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/ai-klassifikaciya-operacij/ | /uz/imkoniyatlar/ai-operatsiyalar-tasnifi/ |
| Title | AI-классификация банковских операций \| Contador | AI yordamida operatsiyalar tasnifi \| Contador |
| H1 | Проверяйте операции с помощью правил и AI | Operatsiyalarni qoidalar va AI yordamida tasniflang |
| Description | Используйте правила и AI-классификацию Contador для обработки банковских операций. Проверяйте предложенные категории и уточняйте неоднозначные назначения. | Contador’da bank operatsiyalarini qoidalar va AI yordamida tasniflang. Tavsiya etilgan toifalarni tekshiring va noaniq operatsiyalarni aniqlashtiring. |
| Основной запрос | AI классификация банковских операций | AI bank operatsiyalarini tasniflash |
| H2 по порядку | Как работают правила и AI; Как проверить результат; Когда нужно уточнение | Qoidalar va AI qanday ishlaydi; Natijani tekshirish; Qachon aniqlashtirish kerak |

Обязательное содержание: Сопоставить 3 демонстрационные операции с предложенными категориями, показать исправление. Чётко отделить предположение AI от подтверждённой операции. Не изображать чат, если его нет.

### 05. Проводки

Тип: feature. Связанные темы: 11, 10, 04.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/buhgalterskie-provodki/ | /uz/imkoniyatlar/buxgalteriya-otkazmalari/ |
| Title | Бухгалтерские проводки по операциям \| Contador | Buxgalteriya o‘tkazmalarini yuritish \| Contador |
| H1 | Формируйте и проверяйте бухгалтерские проводки | Buxgalteriya o‘tkazmalarini yarating va tekshiring |
| Description | Работайте с проводками в Contador: используйте доступные типы документов, проверяйте дебет и кредит и переходите к журналу операций для контроля учёта. | Contador’da mavjud hujjat turlari asosida o‘tkazmalar bilan ishlang. Debet va kreditni tekshirib, hisobni nazorat qilish uchun jurnalga o‘ting. |
| Основной запрос | программа для бухгалтерских проводок | buxgalteriya o‘tkazmalari dasturi |
| H2 по порядку | От операции к проводке; Проверка дебета и кредита; Просмотр в журнале | Operatsiyadan o‘tkazmagacha; Debet va kreditni tekshirish; Jurnalda ko‘rish |

Обязательное содержание: На одном проверенном сценарии показать документ и проводку. Счета брать из проекта, не придумывать номера по памяти. Ссылкой отделить объяснение метода от продукта.

### 06. Закрытие месяца

Тип: feature. Связанные темы: 21, 03, 10.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/zakrytie-mesyaca/ | /uz/imkoniyatlar/oyni-yopish/ |
| Title | Закрытие месяца в бухгалтерии онлайн \| Contador | Buxgalteriyada oyni yopish \| Contador |
| H1 | Проходите закрытие месяца по шагам | Oyni yopish bosqichlarini izchil bajaring |
| Description | Используйте мастер закрытия месяца в Contador: подготовьте операции, пройдите доступные проверки и проверьте результат перед финализацией периода. | Contador’da oyni yopish ustasi bilan operatsiyalarni tayyorlang, mavjud tekshiruvlardan o‘ting va davrni yakunlashdan oldin natijalarni tekshiring. |
| Основной запрос | закрытие месяца программа | buxgalteriyada oyni yopish |
| H2 по порядку | Этапы закрытия периода; Незавершённые операции; Проверка перед финализацией | Davrni yopish bosqichlari; Tugallanmagan operatsiyalar; Yakunlashdan oldingi tekshiruv |

Обязательное содержание: Перечислить действующие семь шагов из UI; показать статус шага и причину, мешающую завершению. Дать бесплатный чек-лист 21. Не обещать автоматическое урегулирование расхождений Soliq.

### 07. Баланс

Тип: feature. Связанные темы: 24, 10, 08.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/buhgalterskiy-balans/ | /uz/imkoniyatlar/buxgalteriya-balansi/ |
| Title | Бухгалтерский баланс онлайн \| Contador | Buxgalteriya balansini shakllantirish \| Contador |
| H1 | Формируйте бухгалтерский баланс в Contador | Contador’da buxgalteriya balansini shakllantiring |
| Description | Получайте бухгалтерский баланс по данным учёта в Contador. Выбирайте дату отчёта, проверяйте показатели и используйте результат для анализа состояния бизнеса. | Contador’dagi hisob ma’lumotlari asosida buxgalteriya balansini oling. Hisobot sanasini tanlang, ko‘rsatkichlarni tekshiring va biznes holatini tahlil qiling. |
| Основной запрос | бухгалтерский баланс онлайн | buxgalteriya balansi onlayn |
| H2 по порядку | Что показывает баланс; Как выбрать дату; Как проверить исходные данные | Balans nimani ko‘rsatadi; Sanani tanlash; Boshlang‘ich ma’lumotlarni tekshirish |

Обязательное содержание: Показать собственный пример активов и обязательств на одну дату, основанный на демо-данных. Указать, что полнота зависит от введённых операций и начальных остатков.

### 08. P&L

Тип: feature. Связанные темы: 22, 09, 15.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/otchet-o-pribylyah-i-ubytkah/ | /uz/imkoniyatlar/foyda-va-zarar-hisoboti/ |
| Title | Отчёт о прибылях и убытках онлайн \| Contador | Foyda va zarar hisoboti onlayn \| Contador |
| H1 | Анализируйте прибыль и убытки бизнеса | Biznes foydasi va zararini tahlil qiling |
| Description | Формируйте отчёт о прибылях и убытках в Contador за выбранный период. Изучайте доходы, расходы и финансовый результат на основе ваших учётных данных. | Contador’da tanlangan davr uchun foyda va zarar hisobotini shakllantiring. Hisob ma’lumotlari asosida daromad, xarajat va moliyaviy natijani ko‘ring. |
| Основной запрос | отчет о прибылях и убытках программа | foyda va zarar hisoboti |
| H2 по порядку | Доходы и расходы за период; Финансовый результат; Почему прибыль не равна деньгам | Davr daromadlari va xarajatlari; Moliyaviy natija; Nega foyda pul qoldig‘iga teng emas |

Обязательное содержание: Показать таблицу отчёта с проверенным примером и фильтром периода. Не называть банковский остаток прибылью. Вывести ссылку на объяснение различий с cash flow.

### 09. ДДС

Тип: feature. Связанные темы: 22, 29, 08.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/dvizhenie-denezhnyh-sredstv/ | /uz/imkoniyatlar/pul-oqimi/ |
| Title | Движение денежных средств бизнеса \| Contador | Biznes pul oqimini kuzatish \| Contador |
| H1 | Следите за поступлениями и выплатами | Pul tushumlari va chiqimlarini kuzating |
| Description | Изучайте движение денежных средств в Contador: поступления, выплаты и данные выбранного периода. Сопоставляйте денежный поток с финансовыми результатами. | Contador’da tanlangan davrdagi pul tushumlari va chiqimlarini ko‘ring. Pul oqimini moliyaviy natijalar bilan solishtirib, biznes hisobini tahlil qiling. |
| Основной запрос | отчет движение денежных средств онлайн | pul oqimi hisoboti |
| H2 по порядку | Поступления и выплаты; Период и банковские счета; Связь с прибылью | Pul tushumlari va chiqimlari; Davr va bank hisobvaraqlari; Foyda bilan bog‘liqlik |

Обязательное содержание: Показать реальный отчёт и его фильтры. Не обещать прогнозирование, если отчёт отражает только прошлые операции.

### 10. ОСВ

Тип: feature. Связанные темы: 23, 12, 07.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/oborotno-saldovaya-vedomost/ | /uz/imkoniyatlar/aylanma-saldo-qaydnomasi/ |
| Title | Оборотно-сальдовая ведомость онлайн \| Contador | Aylanma-saldo qaydnomasi onlayn \| Contador |
| H1 | Проверяйте остатки и обороты по счетам | Hisobvaraqlar qoldiqlari va aylanmalarini tekshiring |
| Description | Формируйте оборотно-сальдовую ведомость в Contador. Просматривайте начальные остатки, обороты и итоговое сальдо для проверки данных бухгалтерского учёта. | Contador’da aylanma-saldo qaydnomasini shakllantiring. Hisobni tekshirish uchun boshlang‘ich qoldiqlar, aylanmalar va yakuniy saldoni ko‘rib chiqing. |
| Основной запрос | оборотно сальдовая ведомость онлайн | aylanma saldo qaydnomasi |
| H2 по порядку | Остатки на начало периода; Дебетовые и кредитовые обороты; Сальдо на конец периода | Davr boshidagi qoldiqlar; Debet va kredit aylanmalari; Davr oxiridagi saldo |

Обязательное содержание: Скриншот ОСВ и пояснения к фактическим колонкам. Пример ошибки неполных исходных данных без обещания, что равенство итогов доказывает корректность всей бухгалтерии.

### 11. Журнал

Тип: feature. Связанные темы: 05, 12, 10.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/zhurnal-provodok/ | /uz/imkoniyatlar/otkazmalar-jurnali/ |
| Title | Журнал бухгалтерских проводок \| Contador | Buxgalteriya o‘tkazmalari jurnali \| Contador |
| H1 | Просматривайте проводки в едином журнале | O‘tkazmalarni yagona jurnalda ko‘ring |
| Description | Проверяйте бухгалтерские записи в журнале проводок Contador. Изучайте даты, суммы и счета, чтобы разбирать операции и контролировать отражение документов. | Contador o‘tkazmalar jurnalida buxgalteriya yozuvlarini tekshiring. Operatsiyalarni tushunish uchun sana, summa va hisobvaraqlarni ko‘rib chiqing. |
| Основной запрос | журнал бухгалтерских проводок | buxgalteriya o‘tkazmalari jurnali |
| H2 по порядку | Какие записи видны в журнале; Как проверить операцию; Переход к отчётам | Jurnalda qanday yozuvlar ko‘rinadi; Operatsiyani tekshirish; Hisobotlarga o‘tish |

Обязательное содержание: Показать 3 демонстрационные записи и доступные фильтры. Не заявлять неизменяемый юридический аудиторский журнал: это журнал учётных проводок.

### 12. Карточка счёта

Тип: feature. Связанные темы: 10, 11, 23.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/kartochka-scheta/ | /uz/imkoniyatlar/hisobvaraq-kartochkasi/ |
| Title | Карточка бухгалтерского счёта онлайн \| Contador | Buxgalteriya hisobvarag‘i kartochkasi \| Contador |
| H1 | Разбирайте операции по отдельному счёту | Alohida hisobvaraq operatsiyalarini ko‘rib chiqing |
| Description | Открывайте карточку счёта в Contador для проверки движений за период. Изучайте записи и суммы, чтобы понимать, как сформировались показатели учёта. | Contador’da hisobvaraq kartochkasini ochib, davr harakatlarini tekshiring. Hisob ko‘rsatkichlari qanday shakllanganini yozuvlar va summalar orqali ko‘ring. |
| Основной запрос | карточка бухгалтерского счета | hisobvaraq kartochkasi |
| H2 по порядку | Выбор счёта и периода; Движения по счёту; Проверка итогов | Hisobvaraq va davrni tanlash; Hisobvaraq harakatlari; Natijalarni tekshirish |

Обязательное содержание: Отличить бухгалтерский счёт от банковского счёта. Показать один подтверждённый пример без вымышленных функций экспорта или детализации.

### 13. Открытые позиции

Тип: feature. Связанные темы: 06, 21, 15.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/vozmozhnosti/avansy-i-podotchet/ | /uz/imkoniyatlar/avanslar-va-hisobdor-summalar/ |
| Title | Учёт авансов и подотчётных сумм \| Contador | Avanslar va hisobdor summalar hisobi \| Contador |
| H1 | Контролируйте авансы и подотчётные суммы | Avanslar va hisobdor summalarni nazorat qiling |
| Description | Просматривайте открытые позиции в Contador: авансы, подотчётные суммы и сроки. Проверяйте незакрытые операции и следите за их состоянием в учёте. | Contador’da ochiq pozitsiyalar, avanslar, hisobdor summalar va muddatlarni ko‘ring. Yopilmagan operatsiyalarni tekshirib, hisobdagi holatini kuzating. |
| Основной запрос | учет авансов подотчетных сумм | avanslar hisobdor summalar hisobi |
| H2 по порядку | Какие позиции остаются открытыми; Сроки и статусы; Проверка перед закрытием | Qaysi pozitsiyalar ochiq qoladi; Muddatlar va holatlar; Yopishdan oldin tekshirish |

Обязательное содержание: Сделать пример выданного аванса и проверки его закрытия. Раскрыть только реальные статусы и механизмы; не рекламировать автоматические напоминания, если их нет.

### 14. Бухгалтеру

Тип: audience. Связанные темы: 03, 04, 06, 21.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/dlya-buhgalterov/ | /uz/buxgalterlar-uchun/ |
| Title | Программа для бухгалтеров Узбекистана \| Contador | O‘zbekiston buxgalterlari uchun dastur \| Contador |
| H1 | Рабочие инструменты бухгалтера в одном сервисе | Buxgalter uchun ish vositalari bitta servisda |
| Description | Импортируйте выписки, проверяйте классификацию и проводки, готовьте отчёты в Contador. Изучите последовательность работы бухгалтера с данными организации. | Contador’da ko‘chirmalarni yuklang, tasnif va o‘tkazmalarni tekshiring, hisobotlarni tayyorlang. Tashkilot ma’lumotlari bilan ishlash bosqichlarini ko‘ring. |
| Основной запрос | программа для бухгалтера Узбекистан | buxgalterlar uchun dastur |
| H2 по порядку | Подготовьте данные; Проверьте операции; Сформируйте отчёты | Ma’lumotlarni tayyorlang; Operatsiyalarni tekshiring; Hisobotlarni shakllantiring |

Обязательное содержание: Сценарий рабочего дня бухгалтера вместо повторения главной. Роли и несколько организаций описывать только в подтверждённых пределах. Вторичный CTA — чек-лист закрытия.

### 15. Руководителю

Тип: audience. Связанные темы: 08, 09, 22, 29.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/dlya-rukovoditeley/ | /uz/rahbarlar-uchun/ |
| Title | Учёт и финансовые отчёты для руководителя \| Contador | Rahbar uchun hisob va moliyaviy hisobotlar \| Contador |
| H1 | Понимайте финансовые результаты своего бизнеса | Biznesingiz moliyaviy natijalarini tushuning |
| Description | Смотрите прибыль, денежные потоки и бухгалтерские отчёты в Contador. Разбирайте результаты компании вместе с бухгалтером на основе данных учёта. | Contador’da foyda, pul oqimi va buxgalteriya hisobotlarini ko‘ring. Kompaniya natijalarini buxgalter bilan birga hisob ma’lumotlari asosida tahlil qiling. |
| Основной запрос | финансовый учет для руководителя | rahbar uchun moliyaviy hisobot |
| H2 по порядку | Прибыль и деньги на счёте; Вопросы бухгалтеру; Какие отчёты открыть | Foyda va hisobdagi pul; Buxgalterga savollar; Qaysi hisobotlarni ko‘rish kerak |

Обязательное содержание: Показать 3 вопроса руководителя и связанный отчёт. Без заявления о прогнозном бюджете, консолидированной отчётности или замене финансового директора.

### 16. Сервисным компаниям

Тип: audience. Связанные темы: 13, 08, 06.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/dlya-kompaniy-uslug/ | /uz/xizmat-korsatish-korxonalari-uchun/ |
| Title | Учёт для компаний сферы услуг \| Contador | Xizmat ko‘rsatish korxonalari hisobi \| Contador |
| H1 | Ведите учёт сервисной компании в Contador | Xizmat ko‘rsatish korxonasi hisobini yuriting |
| Description | Работайте с банковскими операциями, доходами, расходами и авансами сервисной компании. Изучите подходящий сценарий учёта и отчётности в Contador. | Contador’da xizmat ko‘rsatish korxonasining bank operatsiyalari, daromad, xarajat va avanslari bilan ishlash hamda hisobot imkoniyatlarini ko‘ring. |
| Основной запрос | бухгалтерская программа для услуг | xizmat ko‘rsatish korxonasi hisobi |
| H2 по порядку | Оплата услуг и расходы; Работа с авансами; Проверка результата за месяц | Xizmat to‘lovlari va xarajatlar; Avanslar bilan ishlash; Oy natijasini tekshirish |

Обязательное содержание: Один конкретный пример агентства/сервисного ООО на вымышленных данных. Не заявлять отраслевой модуль, CRM, тайм-трекинг, склад или особые льготы.

### 17. Тарифы

Тип: pricing. Связанные темы: 02, 14, 01.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/tarify/ | /uz/tariflar/ |
| Title | Тарифы и стоимость Contador для бизнеса | Contador tariflari va biznes uchun narxlar |
| H1 | Выберите подходящий тариф Contador | O‘zingizga mos Contador tarifini tanlang |
| Description | Изучите актуальную стоимость Contador, доступные возможности и ограничения тарифов. Сравните условия использования сервиса перед созданием аккаунта. | Contador’ning amaldagi narxlari, tarif imkoniyatlari va cheklovlari bilan tanishing. Hisob yaratishdan oldin servisdan foydalanish shartlarini solishtiring. |
| Основной запрос | Contador тарифы стоимость | Contador tariflari narxi |
| H2 по порядку | Стоимость и период оплаты; Что входит в тариф; Вопросы об оплате | Narx va to‘lov davri; Tarifga nimalar kiradi; To‘lov bo‘yicha savollar |

Обязательное содержание: Блокирует публикацию отсутствие подтверждённых цен. Получить фактический состав подписки и лимиты из кода/админ-конфигурации без секретов; не брать чужие цены и не подставлять 0.

### 18. Каталог инструкций

Тип: hub. Связанные темы: 19, 21, 22, 30.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/rukovodstva/ | /uz/qollanmalar/ |
| Title | Инструкции по учёту и отчётам \| Contador | Hisob va hisobotlar bo‘yicha qo‘llanmalar \| Contador |
| H1 | Практические руководства по работе с учётом | Hisob bilan ishlash bo‘yicha amaliy qo‘llanmalar |
| Description | Разберитесь с банковскими выписками, закрытием месяца, ОСВ и финансовыми отчётами. Пошаговые материалы Contador с примерами для работы с учётом. | Bank ko‘chirmalari, oyni yopish, aylanma-saldo qaydnomasi va moliyaviy hisobotlarni o‘rganing. Contador qo‘llanmalaridagi amaliy misollar bilan tanishing. |
| Основной запрос | Contador инструкции бухгалтерский учет | hisob yuritish qo‘llanmalari |
| H2 по порядку | Начало работы; Проверки и закрытие; Понимание отчётов | Ishni boshlash; Tekshiruvlar va yopish; Hisobotlarni tushunish |

Обязательное содержание: Каталог 19–25 и 30. Добавить дату реального обновления и тематические группы, без индексируемых пустых тегов и внутренних поисковых результатов.

### 19. Как подготовить выписку

Тип: guide. Связанные темы: 03, 20, 21.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/rukovodstva/podgotovka-bankovskoy-vypiski/ | /uz/qollanmalar/bank-kochirmasini-tayyorlash/ |
| Title | Как подготовить банковскую выписку к импорту | Bank ko‘chirmasini importga qanday tayyorlash kerak |
| H1 | Подготовка банковской выписки: пошаговая инструкция | Bank ko‘chirmasini importga tayyorlash qo‘llanmasi |
| Description | Узнайте, что проверить в банковской выписке перед импортом: формат, период, суммы и повторные операции. Практическая инструкция для загрузки в Contador. | Importdan oldin bank ko‘chirmasining formati, davri, summalari va takroriy operatsiyalarini tekshirishni o‘rganing. Contador uchun amaliy yo‘riqnoma. |
| Основной запрос | как загрузить банковскую выписку в программу | bank ko‘chirmasini yuklash yo‘riqnomasi |
| H2 по порядку | Проверьте формат и период; Сопоставьте суммы; Разберите ошибку загрузки | Format va davrni tekshiring; Summalarni solishtiring; Yuklash xatosini aniqlang |

Обязательное содержание: Точный пошаговый путь по текущему UI, образец поддерживаемых колонок, таблица 3 реальных ошибок и исправлений. Отличается от 03 обучающим интентом.

### 20. Как проверять AI

Тип: guide. Связанные темы: 04, 05, 19.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/rukovodstva/proverka-ai-klassifikacii/ | /uz/qollanmalar/ai-tasnifini-tekshirish/ |
| Title | Как проверять AI-классификацию операций \| Contador | AI operatsiyalar tasnifini tekshirish \| Contador |
| H1 | Проверка операций после AI-классификации | AI tasnifidan keyin operatsiyalarni tekshiring |
| Description | Разберите, как проверять категории операций, уточнять назначение платежа и исправлять неоднозначные результаты AI-классификации в Contador. | Operatsiya toifalarini tekshirish, to‘lov maqsadini aniqlashtirish va AI tasnifidagi noaniq natijalarni tuzatishni Contador misolida o‘rganing. |
| Основной запрос | как проверить AI классификацию операций | AI tasnifini qanday tekshirish |
| H2 по порядку | Какие данные сверять; Разбор неоднозначного платежа; Подтверждение результата | Qaysi ma’lumotlarni solishtirish; Noaniq to‘lovni tahlil qilish; Natijani tasdiqlash |

Обязательное содержание: Разобрать 3 вымышленных примера: перевод между счетами, аванс, обычная оплата. Не выдумывать уровень confidence или кнопку, которых нет в UI.

### 21. Чек-лист закрытия

Тип: checklist. Связанные темы: 06, 13, 10.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/rukovodstva/chek-list-zakrytiya-mesyaca/ | /uz/qollanmalar/oyni-yopish-tekshiruv-royxati/ |
| Title | Чек-лист закрытия месяца в бухгалтерии \| Contador | Oyni yopish uchun tekshiruv ro‘yxati \| Contador |
| H1 | Проверьте готовность учёта к закрытию месяца | Hisobning oyni yopishga tayyorligini tekshiring |
| Description | Пройдите интерактивный чек-лист закрытия месяца: выписки, операции, открытые позиции и отчёты. Отмечайте проверки и распечатайте список для работы. | Ko‘chirmalar, operatsiyalar, ochiq pozitsiyalar va hisobotlarni interaktiv ro‘yxat bilan tekshiring. Bajarilgan bandlarni belgilang va ro‘yxatni chop eting. |
| Основной запрос | чек лист закрытия месяца | oyni yopish tekshiruv ro‘yxati |
| H2 по порядку | Подготовьте данные; Проверьте незавершённые операции; Сверьте отчёты | Ma’lumotlarni tayyorlang; Tugallanmagan operatsiyalarni tekshiring; Hisobotlarni solishtiring |

Обязательное содержание: 10 пунктов, локальное сохранение отметок, сброс, печать, ссылка на сам чек-лист. Не считать прохождение юридическим подтверждением полноты отчётности. Реализация в разделе 7.

### 22. Прибыль и cash flow

Тип: guide. Связанные темы: 08, 09, 15.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/rukovodstva/pribyl-i-denezhnyy-potok/ | /uz/qollanmalar/foyda-va-pul-oqimi-farqi/ |
| Title | Прибыль и денежный поток: в чём разница \| Contador | Foyda va pul oqimi: qanday farq bor \| Contador |
| H1 | Почему прибыль не равна деньгам на счёте | Nega foyda hisobdagi pulga teng emas |
| Description | Разберите разницу между прибылью и денежным потоком на простом примере. Узнайте, зачем смотреть P&L и движение денег вместе при анализе бизнеса. | Oddiy misolda foyda va pul oqimi o‘rtasidagi farqni o‘rganing. Biznesni tahlil qilishda foyda-zarar hisoboti va pul harakatini birga ko‘rish sabablarini biling. |
| Основной запрос | прибыль и денежный поток разница | foyda va pul oqimi farqi |
| H2 по порядку | Два разных показателя; Пример с отсрочкой оплаты; Какие отчёты смотреть вместе | Ikki xil ko‘rsatkich; Kechiktirilgan to‘lov misoli; Qaysi hisobotlarni birga ko‘rish kerak |

Обязательное содержание: Оригинальный пример оказанной услуги с оплатой позже: разнести признание дохода и движение денег, обозначить упрощённость примера и отсутствие налогового расчёта.

### 23. Как читать ОСВ

Тип: guide. Связанные темы: 10, 12, 24.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/rukovodstva/kak-chitat-osv/ | /uz/qollanmalar/aylanma-saldo-qaydnomasini-oqish/ |
| Title | Как читать оборотно-сальдовую ведомость \| Contador | Aylanma-saldo qaydnomasini qanday o‘qish kerak |
| H1 | Как разобраться в оборотно-сальдовой ведомости | Aylanma-saldo qaydnomasini tushunish qo‘llanmasi |
| Description | Изучите колонки ОСВ на понятном примере: начальное сальдо, дебет, кредит и остаток на конец периода. Разберитесь, какие данные нужно проверить. | Boshlang‘ich saldo, debet, kredit va davr oxiridagi qoldiqni oddiy misolda o‘rganing. Aylanma-saldo qaydnomasida nimalarni tekshirish kerakligini biling. |
| Основной запрос | как читать оборотно сальдовую ведомость | aylanma saldo qaydnomasini o‘qish |
| H2 по порядку | Что означает каждая колонка; Пример движения по счёту; Почему равенства итогов недостаточно | Har bir ustun nimani anglatadi; Hisobvaraq harakati misoli; Nega jami tengligi yetarli emas |

Обязательное содержание: Аннотированная собственная таблица без реальных данных. Не смешивать правила активных/пассивных счетов и проверять пример по документации проекта.

### 24. Начальные остатки

Тип: guide. Связанные темы: 07, 23, 30.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/rukovodstva/nachalnye-ostatki/ | /uz/qollanmalar/boshlangich-qoldiqlar/ |
| Title | Начальные остатки: подготовка данных \| Contador | Boshlang‘ich qoldiqlarni tayyorlash \| Contador |
| H1 | Подготовьте начальные остатки для начала учёта | Hisobni boshlash uchun qoldiqlarni tayyorlang |
| Description | Узнайте, какие данные собрать для ввода начальных остатков в Contador. Проверьте дату начала учёта, счета и суммы перед дальнейшей работой с отчётами. | Contador’da boshlang‘ich qoldiqlarni kiritish uchun kerakli ma’lumotlarni o‘rganing. Hisobotlar bilan ishlashdan oldin sana, hisobvaraqlar va summalarni tekshiring. |
| Основной запрос | как ввести начальные остатки Contador | boshlang‘ich qoldiqlarni kiritish |
| H2 по порядку | Выберите дату начала; Соберите данные по счетам; Проверьте введённые остатки | Boshlanish sanasini tanlang; Hisobvaraqlar ma’lumotlarini yig‘ing; Kiritilgan qoldiqlarni tekshiring |

Обязательное содержание: Использовать действующую страницу settings/opening-balance. Чек-лист подготовки и демо-скриншот, не обещать автоматический перенос всей базы 1С.

### 25. Выбор программы

Тип: guide. Связанные темы: 02, 17, 19.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/rukovodstva/kak-vybrat-buhgalterskuyu-programmu/ | /uz/qollanmalar/buxgalteriya-dasturini-tanlash/ |
| Title | Как выбрать бухгалтерскую программу в Узбекистане | O‘zbekistonda buxgalteriya dasturini tanlash |
| H1 | Проверьте программу на задачах вашего бизнеса | Dasturni biznesingiz vazifalarida tekshiring |
| Description | Используйте список критериев для выбора бухгалтерской программы в Узбекистане: импорт, отчёты, проверка операций, доступы и стоимость обслуживания. | O‘zbekistonda buxgalteriya dasturini tanlash mezonlarini ko‘ring: import, hisobotlar, operatsiyalarni tekshirish, kirish huquqlari va xizmat narxi. |
| Основной запрос | как выбрать бухгалтерскую программу Узбекистан | buxgalteriya dasturini qanday tanlash |
| H2 по порядку | Составьте список задач; Проверьте сценарий на примере; Сравните условия и ограничения | Vazifalar ro‘yxatini tuzing; Jarayonni misolda tekshiring; Shartlar va cheklovlarni solishtiring |

Обязательное содержание: Таблица критериев с пустой колонкой для оценки пользователем. Не делать вымышленный рейтинг и неподтверждённое сравнение конкурентов. Обозначить авторство Contador.

### 26. Каталог инструментов

Тип: hub. Связанные темы: 27, 28, 29, 21.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/instrumenty/ | /uz/vositalar/ |
| Title | Бесплатные калькуляторы для бизнеса \| Contador | Biznes uchun bepul kalkulyatorlar \| Contador |
| H1 | Полезные инструменты для расчётов бизнеса | Biznes hisob-kitoblari uchun foydali vositalar |
| Description | Рассчитайте маржу, наценку, точку безубыточности и запас денежных средств. Бесплатные инструменты Contador с формулами и примерами без регистрации. | Marja, ustama, zararsizlik nuqtasi va pul zaxirasining yetish muddatini hisoblang. Formulalar va misollar bilan bepul vositalar, ro‘yxatdan o‘tish shart emas. |
| Основной запрос | бесплатные калькуляторы для бизнеса | biznes uchun bepul kalkulyatorlar |
| H2 по порядку | Маржа и наценка; Безубыточность; Запас денежных средств | Marja va ustama; Zararsizlik nuqtasi; Pul zaxirasi |

Обязательное содержание: Карточки 27–29 плюс ссылка на чек-лист 21. Обещание бесплатности относится к этим самостоятельным инструментам и должно быть реализовано.

### 27. Маржа и наценка

Тип: tool. Связанные темы: 28, 08, 26.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/instrumenty/kalkulyator-marzhi-i-nacenki/ | /uz/vositalar/marja-va-ustama-kalkulyatori/ |
| Title | Калькулятор маржи и наценки онлайн \| Contador | Marja va ustama kalkulyatori onlayn \| Contador |
| H1 | Рассчитайте маржу и наценку по цене и себестоимости | Narx va tannarx asosida marja va ustamani hisoblang |
| Description | Введите себестоимость и цену продажи, чтобы рассчитать маржу и наценку. Бесплатный калькулятор Contador с формулами, пояснениями и примером расчёта. | Marja va ustamani hisoblash uchun tannarx va sotuv narxini kiriting. Contador’ning bepul kalkulyatori, formulalar, izohlar va hisoblash misoli. |
| Основной запрос | калькулятор маржи наценки | marja ustama kalkulyatori |
| H2 по порядку | Введите цену и себестоимость; Чем отличаются проценты; Формулы и пример | Narx va tannarxni kiriting; Foizlar o‘rtasidagi farq; Formulalar va misol |

Обязательное содержание: Реальный вычисляющий инструмент, два процента и валовая прибыль на единицу. Точные формулы/ошибки в разделе 7. Не называть валовую прибыль чистой.

### 28. Безубыточность

Тип: tool. Связанные темы: 27, 08, 26.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/instrumenty/tochka-bezubytochnosti/ | /uz/vositalar/zararsizlik-nuqtasi/ |
| Title | Калькулятор точки безубыточности \| Contador | Zararsizlik nuqtasi kalkulyatori \| Contador |
| H1 | Рассчитайте объём продаж для безубыточности | Zararsizlik uchun zarur savdo hajmini hisoblang |
| Description | Укажите постоянные расходы, цену и переменные затраты на единицу. Рассчитайте точку безубыточности по упрощённой модели и изучите формулу расчёта. | Doimiy xarajatlar, birlik narxi va o‘zgaruvchan xarajatlarni kiriting. Soddalashtirilgan modelda zararsizlik nuqtasini hisoblab, formula bilan tanishing. |
| Основной запрос | калькулятор точки безубыточности | zararsizlik nuqtasi kalkulyatori |
| H2 по порядку | Задайте исходные значения; Продажи для покрытия расходов; Ограничения модели | Boshlang‘ich qiymatlarni kiriting; Xarajatlarni qoplash uchun savdo; Model cheklovlari |

Обязательное содержание: Показать непрерывный порог и округлённое число единиц, выручку. Объяснить одинаковый период расходов и допущение об одном продукте.

### 29. Запас денег

Тип: tool. Связанные темы: 09, 22, 26.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/instrumenty/zapas-denezhnyh-sredstv/ | /uz/vositalar/pul-zaxirasi-muddati/ |
| Title | На сколько хватит денег бизнесу: калькулятор | Pul zaxirasi qancha vaqtga yetadi: kalkulyator |
| H1 | Рассчитайте запас денежных средств в месяцах | Pul zaxirasi necha oyga yetishini hisoblang |
| Description | Введите доступные деньги, среднемесячные поступления и выплаты. Оцените срок до исчерпания запаса при неизменном денежном потоке и сравните сценарии. | Mavjud pul, o‘rtacha oylik tushum va chiqimlarni kiriting. Pul oqimi o‘zgarmasa, zaxira qancha vaqtga yetishini hisoblab, ssenariylarni solishtiring. |
| Основной запрос | на сколько хватит денег бизнесу калькулятор | pul zaxirasi qancha vaqtga yetadi |
| H2 по порядку | Деньги и ежемесячный поток; Расчёт срока; Что изменит результат | Pul va oylik oqim; Muddatni hisoblash; Natijaga nima ta’sir qiladi |

Обязательное содержание: Детерминированный сценарий с объяснением допущений. Не прогноз финансовой устойчивости и не персональная инвестиционная рекомендация. Без сбора пользовательских сумм.

### 30. Переход из таблиц

Тип: guide. Связанные темы: 24, 19, 25.

| Поле | Русский | O‘zbekcha |
| --- | --- | --- |
| URL | /ru/rukovodstva/perehod-iz-excel/ | /uz/qollanmalar/exceldan-otish/ |
| Title | Как перейти от Excel к учёту в программе \| Contador | Excel’dan hisob dasturiga o‘tish \| Contador |
| H1 | Подготовьте переход от таблиц к Contador | Jadvallardan Contador’ga o‘tishni tayyorlang |
| Description | Составьте план перехода от таблиц к учёту в Contador: выберите дату, соберите остатки, подготовьте поддерживаемые выписки и проверьте первые отчёты. | Contador’ga o‘tish rejasini tuzing: sanani tanlang, qoldiqlarni yig‘ing, mos bank ko‘chirmalarini tayyorlang va dastlabki hisobotlarni tekshiring. |
| Основной запрос | переход с Excel на бухгалтерскую программу | Excel’dan buxgalteriya dasturiga o‘tish |
| H2 по порядку | Определите границы переноса; Подготовьте остатки и выписки; Сверьте первый период | Ko‘chirish chegaralarini belgilang; Qoldiqlar va ko‘chirmalarni tayyorlang; Birinchi davrni tekshiring |

Обязательное содержание: Показать таблицу «что переносится поддерживаемым способом / что требует подготовки». Не обещать конвертацию произвольной Excel-бухгалтерии одним кликом.

## 6. Контент, дизайн и конверсия

### Общие компоненты

Сохранить бренд Contador: логотип, шрифты и цвета сначала извлечь из актуального проекта. Оформление — светлая рабочая область, читаемые таблицы, крупный скриншот продукта, заметный основной CTA. Не копировать дизайн конкурентов пиксель-в-пиксель. На мобильном порядок: H1 → ценность → CTA → визуальный пример → детали. Проверить ширины 360, 768 и 1440 px, клавиатурную навигацию и читаемость таблиц.

Меню RU: «Возможности», «Для бухгалтеров», «Для руководителей», «Руководства», «Инструменты», «Тарифы», «Войти». UZ: «Imkoniyatlar», «Buxgalterlar uchun», «Rahbarlar uchun», «Qo‘llanmalar», «Vositalar», «Tariflar», «Kirish». Скрывать только непубликованные пункты; не оставлять ссылки на 404.

| Тип страницы | Обязательный порядок блоков | Основное действие |
| --- | --- | --- |
| home | H1 и обещание → 3 шага → скриншот → функции → аудитории → реальные условия → FAQ → CTA | Создать аккаунт |
| feature | H1 и результат → входные данные → скриншот/пример → 3–5 шагов → ограничения → FAQ → связанные страницы → CTA | Создать аккаунт |
| audience | Проблема аудитории → рабочий сценарий → 3 подходящих функции → пример → ограничения → FAQ → CTA | Создать аккаунт |
| pricing | Название → подтверждённые цены/лимиты → сравнение → оплата → FAQ | Выбрать доступный тариф/создать аккаунт по действующему сценарию |
| guide | Краткий ответ → содержание → пошаговая инструкция → собственный пример → ошибки → связанные функции → CTA | Открыть соответствующую возможность |
| tool | H1 → рабочий калькулятор → результат → формула → пример → ограничения → связанные материалы → CTA | Посчитать, затем посмотреть отчёты Contador |
| checklist | Краткая цель → интерактивные пункты → прогресс → сброс/печать/поделиться → мастер закрытия | Пройти чек-лист |
| hub | H1 и пояснение → полезные категории → дочерние карточки | Перейти к выбранной теме |

Ориентиры объёма без искусственного растягивания: feature/audience 450–800 слов; guide 700–1200; инструмент 300–600 плюс функциональность. Для узбекского — эквивалентная полнота, не буквальное количество слов. Критерий — раскрыть конкретную задачу, привести пример и ограничения. Общий шаблон не заменяет уникальную основную часть.

На feature/audience написать 4–6 содержательных FAQ. Минимально ответить: какие данные нужны; что делает пользователь; что получается; каких ограничений ожидать. Не повторять один и тот же FAQ на 15 страницах. Отображать FAQ в доступном HTML, можно через details/summary. Специальные FAQ-rich-results не входят в обещания или критерии успеха.

Использовать реальные скриншоты своего продукта на искусственных данных, с объясняющим alt на языке страницы. Скриншоты конкурентов в production не использовать. Если кабинет пока русский, на UZ-странице прямо и кратко указать язык демонстрируемого интерфейса, не рисовать несуществующий перевод. Не показывать реальные ИНН, банковские счета, суммы клиента или email. Размеры изображений заданы; WebP/AVIF по поддержке; первая крупная картинка без lazy load, нижние — lazy.

### CTA и доверие

- Базовый CTA RU «Создать аккаунт», UZ «Hisob yaratish». Под ним объяснить следующий шаг. Не использовать «Начать бесплатно», пока тариф/пробный период не подтверждён.
- Вторичный CTA «Посмотреть пример» / «Misolni ko‘rish» прокручивает к настоящему примеру.
- Вход ведёт в действующий `/v2/login`, регистрация — в `/v2/register`, после проверки этих URL на целевом хосте.
- Минимальный список допустимых целей перехода задаётся в коде; не добавлять произвольный returnUrl/open redirect.
- Не придумывать демо-звонок, телефон поддержки, юридическое лицо, адрес, отзывы или клиентские логотипы. Использовать только подтверждённые контакты и правовые страницы текущего сервиса.
- Существующие политики и контакты включить в footer; проверку их актуальности обозначить перед выпуском. Новые юридические документы не генерировать вместо фактов.
- Для будущего кейса подготовить схему «задача → исходный процесс → использование → измеренный результат → ограничения». Пока клиента и разрешённых данных нет, компонент не рендерить и страницу кейса не индексировать.

### Перелинковка и отсутствие каннибализации

Все страницы доступны за 2–3 перехода от главной через меню/каталоги. ID-ссылки из матрицы использовать для 3–4 контекстных ссылок в той же локали; не вставлять их одинаковым набором на каждую страницу.

Каталог функций → 03–13. Каталог руководств → 19–25, 30. Инструменты → 27–29 и 21. Каждая дочерняя страница ссылается на родительский каталог; аудитории связаны с подходящими функциями. Хлебные крошки и кликабельная навигация должны соответствовать реальной иерархии.

Разделить пары интентов: 03 продаёт импорт / 19 учит подготовить файл; 06 показывает мастер / 21 даёт самостоятельный чек-лист; 10 показывает ОСВ в продукте / 23 учит читать её; 08 показывает отчёт / 22 объясняет разницу с потоком денег.

Не создавать варианты для каждого города Узбекистана с заменой одного слова, отдельные страницы под синонимы одного интента, массовые bank-name/industry-name страницы без проверенной функции и самостоятельного содержания. Это также соответствует [правилам Google о doorway и scaled content](https://developers.google.com/search/docs/essentials/spam-policies).

## 7. Полезные инструменты и механика распространения

Все три калькулятора и чек-лист работают без аккаунта. Вычисления на клиенте, без обращения к бухгалтерским API и без сохранения данных на сервере. Формулы — общие математические модели, не налоговый расчёт. Явно указывать допущения рядом с результатом. Денежные значения допускают ноль в оговорённых случаях; отрицательные, нечисловые, бесконечные и чрезмерно большие значения валидировать. При ошибке показывать локализованный текст возле поля, не NaN/Infinity.

Денежные расчёты проводить Decimal или целочисленной арифметикой минимальных единиц; не показывать артефакты floating-point. Проценты и дробные месяцы округлять только для вывода. Для полей разрешить привычный десятичный разделитель/пробелы разрядов, после нормализации проверить значение. Установить явный верхний предел поддерживаемого ввода и протестировать его.

### 27. Маржа и наценка

Вход: себестоимость C ≥ 0, цена P > 0, в одной валюте и на одинаковой основе включения налогов.

- Валовая прибыль на единицу = P − C.
- Маржа = (P − C) / P × 100%.
- Наценка = (P − C) / C × 100% при C > 0.
- При C = 0 наценка «не определена», маржа считается при P > 0. При P = 0 — ошибка поля цены.
- При P < C вывести отрицательные значения и нейтральное объяснение продажи ниже себестоимости.
- Пример: C = 100 000, P = 125 000 → прибыль 25 000, маржа 20%, наценка 25%.
- Никакой оценки НДС, прибыли после всех расходов или рекомендации по установлению цены.

### 28. Безубыточность

Вход: постоянные расходы F ≥ 0 за выбранный период, цена единицы P > 0, переменные расходы на единицу V ≥ 0. Единый период и валюта.

- Маржинальный доход на единицу D = P − V.
- Если D > 0: непрерывный порог Q = F / D; выручка порога R = Q × P.
- Дополнительно для неделимых единиц вывести Qceil = ceil(Q) и Rceil = Qceil × P, явно отличая от непрерывного порога.
- Если D ≤ 0: «При этих условиях продажи не обеспечивают положительного вклада в покрытие постоянных расходов»; не делить на ноль и не выдавать отрицательный объём продаж.
- Пример F = 1 000 000, P = 150 000, V = 100 000 → Q = 20, R = 3 000 000.
- Пример округления F = 1 025 000 при тех же P,V → Q = 20,5; Qceil = 21; R = 3 075 000; Rceil = 3 150 000.
- Модель одного продукта с постоянной ценой/затратами; не учитывать автоматически налоги, сезонность и изменение ассортимента.

### 29. Запас денег

Вход: доступные деньги B ≥ 0, среднемесячные поступления I ≥ 0 и выплаты O ≥ 0.

- Чистый отток N = O − I.
- При N > 0: запас в месяцах M = B / N.
- При N ≤ 0: «В этой модели запас не сокращается при заданных поступлениях и выплатах»; не обещать бесконечную устойчивость бизнеса.
- B = 0 и N > 0 → 0 месяцев.
- Пример B = 30 000 000, I = 8 000 000, O = 13 000 000 → 6 месяцев.
- Не рассчитывать календарную дату исчерпания с ложной точностью. Объяснить, что внутри месяца платежи могут распределяться неравномерно; будущие изменения не прогнозируются.

### 21. Чек-лист

10 пунктов с полноценным переводом:

| RU | UZ |
| --- | --- |
| Выбран правильный период учёта | To‘g‘ri hisob davri tanlangan |
| Собраны выписки по нужным счетам | Kerakli hisobvaraqlar ko‘chirmalari yig‘ilgan |
| Проверена полнота загруженных операций | Yuklangan operatsiyalar to‘liqligi tekshirilgan |
| Разобраны повторные и ошибочные строки | Takroriy va xato satrlar tekshirilgan |
| Проверены предложенные категории | Tavsiya etilgan toifalar tekshirilgan |
| Уточнены неоднозначные операции | Noaniq operatsiyalar aniqlashtirilgan |
| Проверены проводки по документам | Hujjatlar bo‘yicha o‘tkazmalar tekshirilgan |
| Просмотрены авансы и открытые позиции | Avanslar va ochiq pozitsiyalar ko‘rib chiqilgan |
| Проверены остатки и доступные отчёты | Qoldiqlar va mavjud hisobotlar tekshirilgan |
| Результаты проверены перед закрытием периода | Davrni yopishdan oldin natijalar tekshirilgan |

Хранить только булевы отметки и версию списка в localStorage, без свободных полей с данными компании. Прогресс X/10, сброс с локальным подтверждением, печатная версия через print CSS. Подпись: «Рабочий список проверок; состав обязательных действий зависит от вашего учёта» / «Ishchi tekshiruv ro‘yxati; zarur amallar hisobingizga bog‘liq».

### Добровольное распространение

- На 21 и 27–29 добавить «Поделиться ссылкой» / «Havolani ulashish», Copy Link и пользовательскую кнопку Telegram share.
- Делиться каноническим URL и нейтральным названием инструмента. Не включать значения полей, результаты и состояние чек-листа в query string, fragment, OG-картинку или аналитику.
- Для «Скопировать результат» — отдельная кнопка, копирующая только по явному действию пользователя локальный текст результата, входные значения и название модели. Не отправлять автоматически.
- OG-карточки 1200×630 с названием, языком и брендом, без пользовательских данных. Не нужно 60 уникальных иллюстраций: допустим типовой дизайн с точным заголовком.
- Печать результата/чек-листа даёт удобную сохраняемую копию средствами браузера. Самостоятельная генерация PDF/XLSX не обязательна.
- Под калькулятором естественная связка: «Проверяйте фактические данные в отчётах Contador» → релевантный feature. Регистрация не закрывает доступ к результату.
- Не добавлять реферальные выплаты, рассылки, массовые публикации или обязательные ссылки с пользовательских сайтов в эту задачу.

## 8. Техническое SEO

### Индексируемый HTML

Заголовки, основной текст, ссылки, примеры и метаданные доступны в исходном ответе сервера до JS. Статическая генерация предпочтительна; SSR допустим при обосновании. Не выдавать поисковикам отдельный текст по user-agent. Публичные страницы возвращают 200 без сессии и не перенаправляют на login.

Manifest содержит: topicId, type, priority, locale, slug, title, description, h1, sections, faq, relatedIds, primaryCta, featureEvidence, contentStatus, updatedAt. Схема валидирует уникальность topicId+locale/URL/title, существование relatedIds, пару переводов и обязательные поля. Роуты и sitemap строятся только из publication-ready записей. Draft не доступен как обычная публичная страница; preview защищён и noindex.

Metadata: self-canonical, hreflang, title, description, Open Graph, Twitter summary_large_image, абсолютные URL и корректный production origin. Preview не должен получать production-индексацию. Не добавлять год в title автоматически, не обновлять дату содержимого на каждом build.

### Sitemap и robots

- Root `/sitemap.xml` содержит только канонические индексируемые страницы со статусом 200; `lastmod` — дата содержательного изменения. Никаких login, register, кабинетов, API, search/filter и draft.
- Root `/robots.txt` ссылается на sitemap, разрешает публичные страницы и нужные CSS/JS/assets. Для draft/noindex нельзя рассчитывать только на robots Disallow: робот должен увидеть noindex, если URL уже доступен ему.
- Кабинет остаётся защищён авторизацией. На служебных auth-страницах проверить noindex; sitemap не является механизмом доступа или защиты.
- Варианты HTTP/HTTPS, www/non-www и завершающего слэша свести к одному выбранному URL через один redirect, без цепочек. Существующие старые URL перенаправлять только на смысловой эквивалент.
- Неизвестный slug — 404, не 200 с пустым шаблоном. Удалённая без замены страница — 404/410 по принятому процессу.
- UTM и произвольные параметры не создают новую каноническую страницу. Не генерировать страницы под результаты калькулятора.
- Sitemap не означает гарантированную индексацию.

### Структурированные данные

JSON-LD формировать из тех же видимых данных: Organization/WebSite на главной при подтверждённых реквизитах, BreadcrumbList на дочерних, Article на руководствах с реальным авторством/датами, WebPage для остальных. SoftwareApplication добавлять только при корректно заполненных подтверждённых полях; не создавать фиктивные ratings/reviews/offers и не подставлять цену 0. Разметка не должна обещать rich result. Валидировать синтаксис JSON-LD и соответствие видимому контенту; актуальные условия поисковых расширений сверить по официальной документации на дату реализации.

### Скорость и доступность

Цели по реальным полевым данным на 75-м перцентиле: LCP ≤ 2,5 с, INP ≤ 200 мс, CLS ≤ 0,1, отдельно mobile/desktop. При отсутствии полевых данных честно указать «данных пока нет», провести лабораторную проверку и измерить после накопления трафика. Эти пороги не являются гарантией ранжирования. Основание: [Google о Core Web Vitals](https://developers.google.com/search/docs/appearance/core-web-vitals).

Не загружать тяжёлые библиотеки кабинета ради одной диаграммы на лендинге. Предпочесть статический скриншот и текстовую таблицу. Калькуляторы подключать только на нужных страницах. Явные размеры изображений, доступные labels/errors, видимый focus, контраст WCAG AA, отсутствие layout shift при загрузке шрифтов.

## 9. Аналитика и оценка результатов

До выпуска зафиксировать baseline, если доступен: Search Console за последние 28 дней, страна Узбекистан, RU/UZ URL отдельно, брендовые/небрендовые запросы отдельно. Если данных нет — «baseline отсутствует», без вымышленных цифр и прогнозов. Доступ к аккаунтам не блокирует реализацию: подготовить настройки и инструкции подключения.

Использовать существующую аналитику, без второго дублирующего счётчика. Если её нет — подготовить адаптер и документировать требуемый measurement ID; не зашивать выдуманный ID и не активировать внешние сервисы без настроек владельца.

| Событие | Когда | Допустимые параметры |
| --- | --- | --- |
| seo_cta_click | Нажат основной CTA | topic_id, locale, cta_location, target_type |
| guide_to_feature_click | Переход из инструкции к функции | source_topic_id, target_topic_id, locale |
| tool_calculation_success | Получен валидный результат | tool_id, locale |
| checklist_complete | Впервые достигнуты 10/10 за сессию | topic_id, locale |
| share_link_click | Пользователь нажал поделиться | topic_id, locale, channel |
| registration_success | Фактическая успешная регистрация, если событие уже доступно | source_topic_id, locale, без email и других данных учёта |

Событие share_link_click не равно реально отправленному сообщению или новому посетителю. Суммы, категории транзакций, реквизиты и содержимое выписок не отправлять в аналитику. Событие регистрации фиксировать только после подтверждённого успеха, не по нажатию кнопки. Передача source_topic_id через разрешённые статические значения допустима; проверить, что сохранение источника не ломает существующую авторизацию.

Основные метрики после публикации: индексируемые/обнаруженные URL, ошибки canonical/hreflang/404, небрендовые показы и клики из Узбекистана, CTR по теме/локали, переходы к регистрации, подтверждённые регистрации, использование инструментов и добровольные share-clicks.

Оценка на 14-й день — обход и технические ошибки; на 30-й — запросы, CTR, конверсии; на 60–90-й — полезность тематических кластеров и приоритеты расширения. Это план измерения после выпуска, а не автоматическая задача/обязательство ждать 90 дней до сдачи кода. Метрики бизнеса не включать как гарантированные результаты разработчика.

## 10. Этапы и результаты сдачи

| Этап | Что сделать | Проверяемый результат |
| --- | --- | --- |
| A. Аудит | Проверить источник корня сайта, URL, код и claims | Карта существующих маршрутов, evidence функций, решение по root/marketing |
| B. Основа | Manifest, локали, шаблоны, метаданные, навигация | Публичный SSR/SSG HTML, корректная пара языков и 404 |
| C. P0 | Темы 01–08, 14–15, 17–18, 26 в двух языках | До 26 URL, скриншоты, доступность без логина, подтверждённые тарифы |
| D. P1 | Все оставшиеся темы и работающие инструменты | Полная матрица 60 URL либо явный список заблокированных страниц |
| E. QA | Обход URL, локализация, расчёты, regressions | Отчёт проверок с фактическим числом опубликованных/готовых URL |
| F. Передача | PR кода, ссылки из my_office, release/rollback-инструкция | Другой агент/владелец может воспроизвести сборку, preview и выпуск |

Hub 18/26 в P0 заполнять уже готовыми дочерними страницами; если дочерние страницы ещё не готовы, каталог держать в preview, не выпускать пустой индексируемый hub. 26 URL P0 — верхняя граница готовности пилота, не обязательство индексировать пустые страницы. Коммерческое расширение после пилота не останавливать без конкретного блокера.

Пакет сдачи:

- PR в исходном репозитории с работающими страницами и локальным preview.
- Один канонический manifest, RU/UZ-контент, исходники локальных калькуляторов, SEO assets, sitemap/robots.
- Отчёт проверки всех целевых URL: URL, locale, HTTP, canonical, hreflang, title, H1, состояние контента.
- Скриншоты каждого типа шаблона на desktop/mobile для RU/UZ, плюс все обнаруженные отклонения.
- Команды build/lint/tests из реального package.json, окружение тестирования, результаты и известные ограничения.
- Release plan с перечнем конкретных reverse-proxy изменений, порядком проверки и откатом только маркетингового релиза. Не переносить и не перезапускать бухгалтерские БД.
- Обновления этой задачи и STATUS, ссылки на code PR/commit; ARCHITECTURE/ADR — только если решение реализовано.

## 11. Критерии приёмки

Все пункты ниже проверяет исполнитель и перечисляет в PR. Непроверенные пункты не отмечать выполненными.

- [ ] Инструкции обоих репозиториев прочитаны; реальные каталог, ветка и commit записаны.
- [ ] Существующие страницы/URL учтены, новые дубли и ненужная миграция отсутствуют.
- [ ] Все 30 тем имеют полноценные RU/UZ версии; итоговый URL manifest содержит 60 уникальных адресов с учётом выбранного корня. Отдельно перечислены реальные блокеры, если матрица неполная.
- [ ] Каждая опубликованная функция проверена в продукте; title/description/CTA не обещают отсутствующие возможности.
- [ ] Все публичные страницы доступны анонимно, возвращают 200 и содержат основной контент в исходном HTML.
- [ ] Уникальные title/description/H1, корректные lang и взаимные hreflang, self-canonical.
- [ ] Переключение языка сохраняет тему; узбекский текст проверен, русских fallback-блоков нет.
- [ ] Sitemap совпадает с publication-ready manifest; служебные и черновые URL исключены.
- [ ] Все ссылки навигации/перелинковки работают, нет orphan pages, неизвестные адреса дают 404.
- [ ] Калькуляторы проходят числовые примеры и граничные случаи из раздела 7; локализованные ошибки понятны.
- [ ] Чек-лист сохраняет только отметки, сбрасывается и печатается; инструменты работают без регистрации.
- [ ] Кнопки поделиться не раскрывают значения пользователя; OG-карточка соответствует языку и теме.
- [ ] CTA ведут в действующую регистрацию; успешной конверсией не считается клик.
- [ ] Метаданные и JSON-LD не содержат вымышленных отзывов, рейтингов, цен или авторов.
- [ ] Проверены mobile/desktop, клавиатура, labels, контраст, отсутствие горизонтальной прокрутки всей страницы.
- [ ] Проведены build/lint и целевые проверки, результаты зафиксированы. Для нового приложения тесты запускаются в нём; не запускать миграции БД для SEO.
- [ ] Изменения общего роутинга, если есть, проверены: login, register, кабинет, API и callbacks сохранили прежнюю доступность и назначение. Не выполнять реальные оплаты ради проверки.
- [ ] Нет новых обращений публичного маркетинга к приватным финансовым API; авторизация v2 не ослаблена.
- [ ] Получен preview, подготовлены release/rollback-процедуры, обновлены документация и ссылки на PR.

Минимальные содержательные автоматические проверки: обход всех URL по manifest с парсингом HTML и hreflang; неизвестные slug/locale → 404; формулы и нулевые/отрицательные/нечисловые входы инструментов; smoke-тесты CTA и language switch; регрессия auth/маршрутов только если затронута общая маршрутизация. Полный финансовый test suite не обязателен, если бухгалтерский код не изменён.

Завершение: код и контент приняты по проверяемым критериям; если production не выпускался, явно написать «готово к выпуску, не опубликовано». Не считать план, шаблоны без содержимого, 30 русских страниц без узбекских или один pilot выполнением всей задачи.

## 12. Журнал постановки и текущая передача

2026-09-09: по запросу пользователя изучены актуальные инструкции и документация my_office, выборочно проверены реальные файлы contador_v1, разобраны доступные официальные страницы Экстерна и Kick и рекомендации Google. Подготовлена новая задача с 30 темами на RU/UZ, конкретными метаданными, содержанием, техническими ограничениями, математическими инструментами и приёмкой. Код Contador и production не изменялись.

Следующее действие агента-разработчика: этап A. Первым проверить источник публичной главной и сопоставление `/home/admin1/contador/` с актуальной рабочей копией. Неподтверждённые production-URL и тарифы не подставлять по памяти.

При работе дополнять этот раздел краткими фактическими записями: дата, этап, что сделано, code commit/PR, проверка, блокер/следующий шаг. Не переносить в документацию предположения как реализованные функции.

2026-09-09 (этап A, аудит источника — выполнено): По запросу пользователя начата реализация. Подтверждено прямой проверкой: `/home/admin1/contador` — git remote `jedimindtricks420/contador_v1`, ветка `main`, чистое дерево, HEAD `ec57169e5d1e6d3a3dcbcae2c524cfd1dbf17ea5` (совпадает с SHA, зафиксированным в постановке задачи). Корень `contador.uz/` сейчас обслуживает `nginx` (`/etc/nginx/sites-available/contador.uz`, `location /`) из статического файла `/home/admin1/landing/index.html` — самостоятельный HTML без фреймворка, без git, без `robots.txt`/`sitemap.xml`; это НЕ поддерживаемое приложение → по правилу раздела 4 принято архитектурное решение создавать отдельное `marketing/` внутри `contador_v1`, а не встраивать SEO-страницы в `/home/admin1/landing`. `/v2` (Next.js 16.2.1, `basePath: /v2`, порт 3032) и `/admin` (порт 3031) обслуживаются nginx отдельными `location`-блоками и не менялись. В `v2/next.config.ts` обнаружено `experimental.cpus: 1` с комментарием о падении всей VM от параллельной сборки 2026-08-19 — учтено как обязательное ограничение для новой сборки; на хосте не запускался ни один dev-сервер (соответствует [[feedback_no_dev_server_verification]]).

Собраны и подтверждены реальные данные для пилотных тем (не по памяти): маршруты `/v2/login`, `/v2/register`, `/v2/dashboard`, `/v2/settings/opening-balance`, `/v2/closing`, `/v2/balance`, `/v2/pnl`, `/v2/cashflow`, `/v2/reports`, `/v2/transactions`, `/v2/accounts`, `/v2/open-positions`; реальные 7 шагов мастера закрытия месяца (`v2/src/app/closing/ClosingWizard.tsx:154-164`); реальные тарифы FREE/PRO (`v2/src/lib/constants.ts` — `BILLING.DEFAULT_PRO_PRICE_YEARLY=299000`, `DEFAULT_PRO_DURATION_DAYS=365`), подтверждённые вживую запросом к `http://localhost:3031/admin/api/payment-info` → `{"pro_price_yearly":299000}`, что разблокировало тему 17 (Тарифы) для пилота. Реальные коды счетов НСБУ взяты из `v2/src/lib/constants.ts`.

Этап B (основа) и пилотная волна темы 01, 02, 06, 17, 21, 26, 27 (RU+UZ, 14 URL из 60) отправлены в работу отдельному агенту в изолированном git worktree репозитория `contador_v1`, ветка `feature/seo-landings-ru-uz` — код ещё не смержен, не задеплоен, production не менялся. Следующая запись в этом журнале будет сделана после получения результата и ревью.

2026-09-09 (этап B, основа + пилот — выполнено и независимо перепроверено): Пилотная волна получена (коммиты `e61af70`, `53ff6ef` в worktree `/home/admin1/contador/.claude/worktrees/agent-adfeb0b207ad07498`, ветка `feature/seo-landings-ru-uz`, база — реальный `main`@`ec57169`). Новое приложение `marketing/` (Next.js 16.2.1, TypeScript, App Router) внутри `contador_v1`: единый manifest (`src/content/seo-pages.ts`) с валидацией на этапе сборки, whitelist-роутинг (`generateStaticParams` + `dynamicParams=false` → настоящий 404 на неизвестные адреса), hreflang/canonical/OG/Twitter-метаданные на каждой странице (`x-default` → RU), JSON-LD (WebPage/BreadcrumbList/Organization+WebSite) только из подтверждённых данных, `sitemap.ts`/`robots.ts` только из `contentStatus:"published"`, компоненты Header/Footer/LanguageSwitcher (переключение по topicId), CTA с точным текстом кнопок и событием `seo_cta_click`, no-op-safe адаптер аналитики без зашитого ID.

Реализованы темы 01 (главная), 02 (каталог функций — на этом этапе карточка только для 06, остальные помечены «скоро»), 06 (закрытие месяца — реальные 7 шагов из кода, + условный 8-й шаг ЭСФ), 17 (тарифы — серверный запрос к `http://localhost:3031/admin/api/payment-info` с fallback на `299000`, ISR `revalidate: 3600`, а не статический текст), 21 (чек-лист — 10 пунктов дословно из ТЗ, только localStorage), 26 (каталог инструментов), 27 (калькулятор маржи/наценки — `decimal.js`, проверен на контрольном примере и граничных случаях из ТЗ). Итого 7 тем × 2 языка = 14 URL.

**Независимая перепроверка (не только отчёт агента):** лично перезапущены `npx vitest run` (18/18 тестов), `npx tsc --noEmit` (чисто), `npx eslint .` (0 ошибок/предупреждений) и один `next build` (успешно, `experimental.cpus:1`, память проверена `free -h` до/после — не опускалась ниже ~700 МБ свободной/7.8 ГБ available, сборка не уронила хост). Отдельно проверено техническое устройство сборки: конкретные HTML-файлы 14 URL не пишутся на диск при `next build` (в `.next/server/app` только generic-манифесты и роут-шаблон) — это ожидаемое поведение ISR из-за `fetch(..., {revalidate: 3600})` на странице тарифов (Next.js рендерит и кеширует HTML по каждому пути при первом запросе, а не только на этапе сборки), а не дефект; полный HTML в исходном ответе сервера подтверждён тестом `ssr-content.test.tsx`, который рендерит те же серверные компоненты через `renderToStaticMarkup` без поднятия процесса/порта — реальную задержку первого запроса по каждому URL предстоит проверить на реальном preview-деплое (см. следующий этап).

Не реализовано в этом проходе (зафиксировано агентом честно, не выдано за готовое): реальные скриншоты продукта (заглушка), дизайн OG-картинки (плейсхолдер 1200×630), реальный analytics ID, узбекский текст не прошёл проверку компетентным редактором (риск отмечен явно), нет отдельного черновик/preview-режима для тем со статусом draft (не нужен пока все опубликованные темы реально готовы), футер с юридическими контактами оставлен нейтральным (не найдено подтверждённых публичных юридических страниц — не выдумано).

Этап C (продолжение P0 — выполнено и независимо перепроверено): Темы 03 (импорт — честно отмечено, что доступен только внутри мастера закрытия, отдельного экрана нет), 04 (AI-классификация — реальный порог доверия 70, только очередь уточнения, чат не рисуется), 05 (проводки — реальный экран `/v2/documents`, коды счетов намеренно не называются без дополнительной сверки), 07 (баланс, `/v2/balance`), 08 (P&L, `/v2/pnl`), 14 (бухгалтеру), 15 (руководителю) — коммит `5ea2b53`. Каталог функций (02) переведён из «скоро» в реальные карточки 03–08. `main`, `v2/`, `docker-compose.yml`, nginx и `.env` не тронуты (проверено `git diff --stat` вне `marketing/` — пусто).

**Независимая перепроверка**: `npx vitest run` (26/26), `npx tsc --noEmit` (чисто), `npx eslint .` (чисто), `free -h` стабильна (~7.8 ГБ available). P0 из раздела 5 задачи (13 тем: 01–08, 14–15, 17–18, 26) теперь закрыт на 12 из 13 — не хватает только 18 (каталог инструкций), сознательно отложен до появления хотя бы одной темы-инструкции (19–25/30, все P1), как и предписывает раздел 10 задачи для пустых хабов.

Этап D (P1, продолжение — выполнено и независимо перепроверено): Темы 09 (ДДС, `/v2/cashflow`), 10 (ОСВ, `/v2/reports/osv`), 11 (журнал проводок, `/v2/reports/journal`), 12 (карточка счёта, `/v2/reports/account-card`), 13 (открытые позиции, `/v2/open-positions`), 16 (сервисным компаниям) — коммит `c94907b`. Итого 20 тем × 2 языка = 40 URL из 60. `npx vitest run` (34/34), `tsc`/`eslint` чисто, `free -h` стабильна.

**Ревью и правка после этапа D**: агент этапа D самостоятельно добавил тему 16 в верхнее меню (`nav.ts`), явно пометив это отклонением от буквального списка §6 задачи (7 фиксированных пунктов, 16 в списке нет — в отличие от 14/15, которые названы буквально). При ревью решение отклонено: меню возвращено к 7 пунктам задачи (коммит `1d96a3e`), тема 16 остаётся достижимой через relatedIds-ссылки со страниц 06/08/13 и через sitemap, без изменения фиксированного меню. Соответствующий тест в `ssr-content.test.tsx` обновлён на противоположное утверждение. Это иллюстрирует ценность проверки решений субагента, а не только доверия его самоотчёту.

Этап E (P1, гайды + хаб 18): в работу тому же агенту в том же worktree/ветке отправлены темы 18 (каталог инструкций — активируется впервые, все дочерние готовы одновременно), 19, 20, 22, 23, 24, 25, 30 (все guide). Ожидается 28 тем × 2 языка = 56 URL из 60 — не 60, как ошибочно указано в первом черновике инструкции агенту (полные 60 URL потребуют ещё калькуляторов 28 и 29, отдельный этап F). Меню пополняется пунктом «Руководства»/«Qo'llanmalar» — это единственный оставшийся пункт из буквального списка §6, поэтому в отличие от темы 16 это не отклонение, а прямое соответствие задаче.
