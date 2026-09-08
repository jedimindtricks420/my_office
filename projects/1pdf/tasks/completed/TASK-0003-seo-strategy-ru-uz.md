# TASK-0003 — SEO-стратегия 1PDF для RU/UZ

Статус: COMPLETED (Phase 0-2 реализованы и опубликованы в продакшн 2026-09-08; Phase 3-6 — будущая работа, см. «Реализация» в конце файла)
Создано: 2026-09-08
Завершено: 2026-09-08
Проект: 1PDF
Область: SEO / product growth / landing architecture

## Цель

Построить для 1PDF масштабируемую SEO-архитектуру, которая приводит пользователей из Google и других поисковых систем на отдельные страницы конкретных PDF-задач на русском и узбекском языках.

Основная идея: не пытаться ранжировать только главную страницу по запросу «PDF редактор», а создать отдельную индексируемую страницу для каждого подтвержденного или запланированного пользовательского intent: редактировать, добавить текст, добавить изображение, подписать, выделить текст, рисовать, организовать страницы и т.д. Для будущих PDF-инструментов (merge/split/compress/convert/OCR и др.) страницы публикуются только после появления реального функционала либо как явно информационные страницы без ложного CTA «использовать инструмент».

## Подтвержденный контекст продукта

По документации после независимого технического аудита 2026-09-08:

- production-сервис: `1pdf.uz`;
- текущий продукт — браузерный PDF-редактор на React/Konva/PDF.js/pdf-lib;
- интерфейс редактора поддерживает RU/UZ/EN;
- landing CMS поддерживает RU/UZ;
- обработка изменений до явного сохранения происходит в браузере;
- существуют учетные записи, документы/версии и PRO-подписка;
- весь `/editor/*` сейчас закрыт PRO-подпиской.

Источники внутри проекта: `docs/OVERVIEW.md`, `docs/STATUS.md`, `docs/VIEWS.md`, `docs/API.md`.

## Стратегическая модель

### 1. Один поисковый intent = одна landing page

Не создавать одну огромную страницу со всеми ключевыми словами. Для каждого достаточно самостоятельного намерения пользователя должна существовать отдельная страница с уникальными:

- URL;
- `<title>`;
- meta description;
- H1;
- вводным текстом;
- блоком «как это сделать»;
- FAQ;
- CTA;
- внутренними ссылками на соседние инструменты;
- RU/UZ hreflang-парой.

### 2. Основные SEO-кластеры

#### Cluster A — PDF editor / верх воронки

- PDF редактор онлайн
- редактировать PDF онлайн
- изменить PDF онлайн
- редактирование PDF в браузере
- бесплатный PDF редактор
- PDF muharriri
- PDF tahrirlash
- PDF faylni onlayn tahrirlash

#### Cluster B — конкретные действия редактора

- добавить текст в PDF
- изменить текст в PDF
- добавить изображение в PDF
- вставить картинку в PDF
- рисовать на PDF
- выделить текст в PDF
- добавить комментарий в PDF
- заполнить PDF
- подписать PDF
- добавить подпись в PDF
- удалить объект из PDF (только если реально поддерживается)
- переместить объект в PDF

Узбекские эквиваленты должны формироваться не механическим переводом, а по реальным формулировкам пользователей: `PDFga matn qo'shish`, `PDF faylni tahrirlash`, `PDFga rasm qo'shish`, `PDFga imzo qo'yish` и т.п.

#### Cluster C — управление страницами

Публиковать только по подтвержденному функционалу:

- повернуть страницы PDF;
- изменить порядок страниц PDF;
- удалить страницу PDF;
- добавить страницу PDF;
- извлечь страницы PDF;
- обрезать PDF.

Перед реализацией каждой страницы агент должен проверить код текущего редактора и пометить capability как `SUPPORTED`, `PARTIAL` или `NOT_IMPLEMENTED`.

#### Cluster D — файловые utilities

Это крупнейший потенциальный programmatic SEO-кластер, но нельзя обещать отсутствующие функции. План развития:

- объединить PDF;
- разделить PDF;
- сжать PDF;
- PDF в Word;
- Word в PDF;
- JPG в PDF;
- PNG в PDF;
- PDF в JPG;
- PDF в PNG;
- Excel в PDF;
- PowerPoint в PDF;
- OCR PDF;
- защитить PDF паролем;
- снять пароль с PDF — только если законно и продукт поддерживает работу с известным пользователю паролем;
- нумерация страниц PDF;
- добавить водяной знак.

Каждый такой utility следует рассматривать одновременно как product feature и SEO acquisition surface.

#### Cluster E — локальный Uzbekistan intent

Создать локальные страницы/семантику без doorway-spam:

- PDF редактор Узбекистан;
- PDF редактор онлайн на русском языке;
- PDF редактор на узбекском языке;
- PDF tahrirlash O'zbekiston;
- PDF muharriri o'zbek tilida;
- PDF online rus tilida;
- работа с PDF без установки программы;
- PDF инструменты для учебы и работы.

Не создавать сотни страниц под города (`/tashkent/`, `/samarkand/` и т.п.), если содержание и продукт идентичны: это создаст thin/doorway pages.

## Рекомендуемая URL-архитектура

Предпочтительный вариант:

```text
/ru/
/uz/

/ru/pdf-editor/
/uz/pdf-muharriri/

/ru/edit-pdf/
/uz/pdf-tahrirlash/

/ru/add-text-to-pdf/
/uz/pdfga-matn-qoshish/

/ru/add-image-to-pdf/
/uz/pdfga-rasm-qoshish/

/ru/sign-pdf/
/uz/pdfga-imzo-qoyish/
```

Когда появятся utilities:

```text
/ru/merge-pdf/          /uz/pdf-birlashtirish/
/ru/split-pdf/          /uz/pdf-bolish/
/ru/compress-pdf/       /uz/pdf-siqish/
/ru/pdf-to-word/        /uz/pdf-wordga/
/ru/word-to-pdf/        /uz/word-pdfga/
/ru/jpg-to-pdf/         /uz/jpg-pdfga/
```

Правило: URL должен быть стабильным, коротким и не зависеть от рекламного текста страницы.

## Первая волна страниц — P0

Это страницы, которые следует реализовать в первую очередь после проверки соответствия реальному функционалу.

| # | RU slug | RU Title | RU H1 | UZ slug | UZ Title | UZ H1 | Intent |
|---|---|---|---|---|---|---|---|
| 1 | `/ru/pdf-editor/` | PDF редактор онлайн — редактировать PDF в браузере | PDF редактор онлайн | `/uz/pdf-muharriri/` | PDF muharriri onlayn — PDF fayllarni tahrirlash | PDF muharriri onlayn | Core |
| 2 | `/ru/edit-pdf/` | Редактировать PDF онлайн — изменить PDF файл | Редактировать PDF онлайн | `/uz/pdf-tahrirlash/` | PDF tahrirlash onlayn | PDF faylni onlayn tahrirlash | Core |
| 3 | `/ru/add-text-to-pdf/` | Добавить текст в PDF онлайн | Как добавить текст в PDF | `/uz/pdfga-matn-qoshish/` | PDFga matn qo'shish onlayn | PDFga matn qo'shish | Feature |
| 4 | `/ru/add-image-to-pdf/` | Добавить изображение в PDF онлайн | Как вставить изображение в PDF | `/uz/pdfga-rasm-qoshish/` | PDFga rasm qo'shish onlayn | PDFga rasm qo'shish | Feature |
| 5 | `/ru/draw-on-pdf/` | Рисовать на PDF онлайн | Рисование на PDF онлайн | `/uz/pdfga-chizish/` | PDFga chizish onlayn | PDF faylga chizish | Feature |
| 6 | `/ru/highlight-pdf/` | Выделить текст в PDF онлайн | Как выделить текст в PDF | `/uz/pdf-matnini-belgilash/` | PDF matnini belgilash onlayn | PDF matnini belgilash | Feature |
| 7 | `/ru/annotate-pdf/` | Добавить комментарии и пометки в PDF | Аннотации и пометки в PDF онлайн | `/uz/pdfga-izoh-qoshish/` | PDFga izoh qo'shish onlayn | PDFga izoh va belgi qo'shish | Feature |
| 8 | `/ru/sign-pdf/` | Подписать PDF онлайн — добавить подпись | Добавить подпись в PDF | `/uz/pdfga-imzo-qoyish/` | PDFga imzo qo'yish onlayn | PDFga imzo qo'yish | Feature |
| 9 | `/ru/fill-pdf/` | Заполнить PDF онлайн | Заполнить PDF документ онлайн | `/uz/pdf-toldirish/` | PDF to'ldirish onlayn | PDF hujjatini to'ldirish | Feature |
| 10 | `/ru/pdf-editor-russian/` | PDF редактор на русском языке онлайн | PDF редактор на русском языке | `/uz/pdf-muharriri-ozbek-tilida/` | O'zbek tilida PDF muharriri | PDF muharriri o'zbek tilida | Localization |

Примечание: пункты 5–9 должны пройти capability-check по коду перед публикацией. Названия сформированы как SEO backlog, а не утверждение, что каждая функция уже присутствует.

## Вторая волна — P1

После capability audit добавить отдельные страницы:

1. изменить текст в PDF;
2. переместить текст/изображение в PDF;
3. изменить размер текста в PDF;
4. изменить цвет текста в PDF;
5. добавить фигуру в PDF;
6. добавить ссылку в PDF;
7. повернуть PDF;
8. удалить страницы PDF;
9. изменить порядок страниц PDF;
10. добавить страницы в PDF;
11. извлечь страницы PDF;
12. обрезать PDF;
13. открыть и редактировать PDF в браузере;
14. PDF editor для Windows без установки;
15. PDF editor для Mac в браузере;
16. PDF editor для телефона;
17. PDF editor для Android;
18. PDF editor для iPhone;
19. PDF для учебы;
20. PDF для документов и работы.

Все страницы получают отдельную RU/UZ версию, но не обязательно буквальный slug-перевод.

## Третья волна — P2 / новый продуктовый функционал

Развивать инструменты, которые сами создают высокоинтентную поисковую страницу:

1. Merge PDF
2. Split PDF
3. Compress PDF
4. PDF → Word
5. Word → PDF
6. PDF → JPG
7. JPG → PDF
8. PDF → PNG
9. PNG → PDF
10. Excel → PDF
11. PPT → PDF
12. OCR PDF
13. Delete PDF pages
14. Rotate PDF
15. Extract PDF pages
16. Reorder PDF pages
17. Crop PDF
18. Add page numbers
19. Watermark PDF
20. Protect PDF

Для каждого инструмента создавать пару RU/UZ landing pages только вместе с реально работающим инструментом.

## Template каждой SEO tool page

Каждая страница должна иметь одинаковую информационную архитектуру, но уникальный текст.

```text
Breadcrumbs
H1: конкретное действие
1–2 предложения value proposition
Tool / upload zone / CTA

H2: Как <действие> PDF онлайн
3 простых шага

H2: Возможности инструмента
3–6 feature blocks

H2: Почему использовать 1PDF
скорость / браузер / язык / приватность — только подтвержденные свойства

H2: Частые вопросы
4–7 FAQ

Related PDF tools
Internal links
```

Главное правило: инструмент или CTA должны находиться выше информационного SEO-текста. Пользователь приходит выполнить действие, а не читать статью.

## Title / description formula

### RU

Title:
`{Основной запрос} онлайн — {короткое преимущество} | 1PDF`

Description:
`{Выполните действие} с PDF онлайн в 1PDF. {1–2 реально доступные возможности}. Работает прямо в браузере.`

### UZ

Title:
`{Asosiy so'rov} onlayn — 1PDF`

Description:
`1PDF orqali PDF faylni onlayn {amal}. {Tasdiqlangan imkoniyat}. Brauzerda ishlaydi.`

Не добавлять слова «бесплатно», «без регистрации», «без загрузки на сервер», «без ограничений» автоматически. Каждое такое обещание должно соответствовать фактическому поведению текущей версии продукта.

## Canonical + hreflang

Для каждой пары:

```html
<link rel="canonical" href="https://1pdf.uz/ru/add-text-to-pdf/">
<link rel="alternate" hreflang="ru" href="https://1pdf.uz/ru/add-text-to-pdf/">
<link rel="alternate" hreflang="uz" href="https://1pdf.uz/uz/pdfga-matn-qoshish/">
<link rel="alternate" hreflang="x-default" href="https://1pdf.uz/ru/add-text-to-pdf/">
```

UZ использовать как `uz`, если продукт не ведет отдельные латиница/кириллица варианты. Не создавать две индексируемые копии одного узбекского текста в разных алфавитах без отдельной стратегии canonical/hreflang.

## Internal linking

Хабы:

```text
/ru/pdf-tools/
/uz/pdf-vositalari/
```

Категории:

```text
Edit PDF
Organize PDF
Convert PDF
Optimize PDF
Sign & Protect
```

На каждой инструментальной странице:

- 3–6 ссылок на соседние инструменты;
- ссылка на общий PDF tools hub;
- breadcrumbs;
- ссылки из главной страницы на P0-инструменты.

Пример graph:

`PDF editor → add text → add image → annotate → sign → fill PDF`

Когда появятся utilities:

`PDF tools → merge / split / compress / convert / organize`.

## Keyword strategy

Не собирать только high-volume head terms. Использовать три уровня:

### Head

- pdf редактор
- редактировать pdf
- pdf editor
- pdf muharriri
- pdf tahrirlash

### Mid-tail

- pdf редактор онлайн
- редактировать pdf онлайн
- добавить текст в pdf
- подписать pdf онлайн
- pdfga matn qo'shish
- pdfga imzo qo'yish

### Long-tail

- как изменить текст в pdf онлайн
- как добавить картинку в pdf без программы
- редактировать pdf в браузере на русском
- pdf faylga matn qanday qo'shiladi
- pdf faylni telefonda tahrirlash

Long-tail запросы должны обслуживаться не отдельной thin page на каждый вопрос, а FAQ/sections основной tool page или качественными guide pages.

## Контентная стратегия

После инструментальных страниц создавать guides, поддерживающие коммерческие landing pages:

RU:

- Как редактировать PDF онлайн
- Как добавить текст в PDF
- Как подписать PDF документ
- Как изменить PDF без Adobe Acrobat
- Как работать с PDF на телефоне
- Как заполнить PDF форму
- Чем открыть PDF для редактирования

UZ:

- PDF faylni qanday tahrirlash mumkin
- PDFga matn qanday qo'shiladi
- PDFga imzo qanday qo'yiladi
- PDF faylni telefonda tahrirlash
- PDF hujjatini qanday to'ldirish mumkin

Guide не должен конкурировать с tool landing: guide отвечает на «как», tool page — на «сделать сейчас». Ссылать guide → tool.

## Structured data

Применять только соответствующую содержимому schema markup:

- `SoftwareApplication` / `WebApplication` для основных tool pages, если поля schema корректны;
- `FAQPage` только когда FAQ реально видим пользователю и соответствует актуальным требованиям Google;
- `BreadcrumbList`;
- `Organization`/`WebSite` на глобальном уровне.

Не использовать fake ratings/reviews.

## Technical SEO requirements

1. SSR, prerender или статически отдаваемый HTML для SEO landing pages. Не полагаться на пустой client-side shell.
2. Уникальный title, description и H1.
3. 200 status для индексируемых страниц.
4. Никаких soft-404.
5. XML sitemap отдельно для RU/UZ либо единый sitemap index.
6. `robots.txt` с sitemap.
7. canonical self-reference.
8. hreflang RU/UZ.
9. Open Graph и Twitter metadata.
10. Breadcrumbs.
11. Чистые стабильные URLs.
12. Не индексировать user documents, portal, admin, auth callbacks и приватные routes.
13. Проверить `/editor/*`: поскольку редактор требует PRO, не использовать приватный editor URL как единственную SEO landing page.
14. Core Web Vitals: минимальный JS для SEO landing, lazy-load тяжелого editor bundle/tool UI при необходимости.
15. Изображения: WebP/AVIF где возможно, корректные dimensions и alt.

## Важная архитектурная рекомендация для 1PDF

SEO landing pages и сам editor должны быть разделены логически:

```text
/ru/add-text-to-pdf/     <- индексируемая public landing + entry point
/editor/...              <- приложение / рабочее пространство
```

Причина: текущий `/editor/*` закрыт PRO через nginx, поэтому Google и новый пользователь не должны зависеть от приватного приложения для понимания назначения страницы.

## Conversion flow

Для страницы инструмента:

```text
Google search
→ SEO tool page
→ Upload / Start editing
→ если действие доступно бесплатно: выполнить
→ если требуется PRO: ясно показать paywall до потери работы пользователя
→ signup/payment
→ editor
```

SEO-текст не должен скрывать ограничения PRO.

## Измерение

Подключить/проверить:

- Google Search Console;
- sitemap submission;
- indexing reports;
- GA4 или privacy-compatible analytics;
- события: landing_view, upload_start, editor_start, signup_start, signup_complete, checkout_start, subscription_success;
- source landing slug в conversion event.

Основные KPI:

1. indexed tool pages;
2. impressions;
3. non-brand clicks;
4. average position по кластеру;
5. tool-start rate;
6. signup conversion;
7. paid conversion;
8. SEO CAC/ROI при наличии затрат на контент/разработку.

## План реализации

### Phase 0 — capability audit

Перед созданием текстов пройти по редактору и сформировать таблицу:

```text
feature | supported | route/component | free/pro | SEO page
```

Нельзя публиковать страницы, утверждающие поддержку отсутствующего инструмента.

### Phase 1 — foundation

- language routing RU/UZ;
- SEO layout/template;
- metadata API/fields;
- canonical/hreflang;
- sitemap;
- robots;
- breadcrumbs;
- PDF tools hub;
- Search Console.

### Phase 2 — первые 10 пар RU/UZ

Реализовать P0 pages после capability audit.

### Phase 3 — expand current editor intents

20–30 высококачественных страниц по реально доступным editor features и device/use-case intents.

### Phase 4 — utilities

Добавлять standalone PDF utilities и одновременно создавать соответствующие RU/UZ SEO pages.

### Phase 5 — guides

Создать supporting content cluster и внутренние ссылки.

### Phase 6 — optimization

Каждые 4–8 недель использовать Search Console:

- queries с impressions, но position 8–30 → улучшать страницу;
- несколько страниц ранжируются по одному запросу → устранять cannibalization;
- новые long-tail запросы → добавлять FAQ/sections;
- страницы без impressions после достаточного времени → объединять/удалять, если они thin.

## Definition of Done для TASK-0003

Документальная задача считается завершенной, когда:

- утверждена RU/UZ SEO information architecture;
- выполнен capability audit реального editor code;
- сформирован окончательный P0/P1 URL map;
- для P0 подготовлены final title/H1/meta descriptions RU/UZ;
- описан template tool page;
- сформированы требования canonical/hreflang/sitemap/robots/schema;
- определена внутренняя перелинковка;
- сформирован implementation backlog для кода;
- после реализации проверена индексация и Search Console.

## Ограничения и риски

- `my_office` не содержит исходный код 1PDF; source of truth для кода — `/home/admin1/1pdf/`. Эта задача пока не изменяет код.
- Текущий editor PRO-only. Нельзя писать «бесплатный PDF редактор» на всех страницах до определения бесплатного user journey.
- Не создавать сотни автоматически переведенных thin pages.
- Не использовать keyword stuffing.
- Не делать doorway city pages.
- Не публиковать tool landing для отсутствующей функции как будто инструмент уже работает.
- Следить за cannibalization между `/pdf-editor/`, `/edit-pdf/` и guide pages.

## Следующее действие агента

Следующая техническая задача должна начаться с read-only capability audit `/home/admin1/1pdf/editor/src` и построения точной матрицы функций. После этого обновить этот TASK конкретными `SUPPORTED/PARTIAL/NOT_IMPLEMENTED`, скорректировать P0 и только затем создавать production landing pages.

## Capability audit (выполнен 2026-09-08, read-only чтение `/home/admin1/1pdf/editor/src`)

| Feature | Статус | Файл:строка | Free/PRO | SEO page |
|---|---|---|---|---|
| Добавить текст | SUPPORTED | `types/edits.ts` (`TextEdit`), размещение — компоненты `editor/` | PRO (весь `/editor/*` за paywall) | `/ru/add-text-to-pdf/` |
| Изменить/заменить существующий текст | SUPPORTED — «cover-and-replace»: существующий текст находится через `lib/pdf/textRuns.ts:38-90` (`extractTextRuns`/`findRunAt`, PDF.js), визуально перекрывается `CoverEdit` и поверх кладётся новый `TextEdit`; подтверждено и локализацией `editor.replaceHint` в `i18n/locales/uz.json` | PRO | `/ru/edit-pdf/` |
| Добавить изображение | SUPPORTED | `types/edits.ts` (`ImageEdit`) | PRO | `/ru/add-image-to-pdf/` |
| Подпись (нарисовать или загрузить) | SUPPORTED | `components/tools/SignatureTool.tsx` (canvas-подпись + upload PNG, `place()` создаёт `ImageEdit`) | PRO | `/ru/sign-pdf/` |
| Выделение текста | SUPPORTED | `types/edits.ts` (`HighlightEdit`) | PRO | `/ru/highlight-pdf/` |
| Рисование (линия/от руки) | SUPPORTED | `types/edits.ts` (`PolylineEdit`, `type: 'line' \| 'freehand'`) | PRO | `/ru/draw-on-pdf/` |
| Фигуры (прямоугольник) и штампы (галочка/крестик) | SUPPORTED | `types/edits.ts` (`RectEdit`, `StampEdit`) | PRO | входит в `/ru/annotate-pdf/` |
| Комментарии в духе «sticky note» (отдельная от разметки сущность) | **NOT_IMPLEMENTED** — в `EditObject` нет типа `comment`/заметки, есть только визуальная разметка (highlight/rect/polyline/stamp) | — | — | `/ru/annotate-pdf/` переформулирован как «разметка и пометки» (highlight+фигуры+рисование+штампы), а не «комментарии» в духе Adobe/Word |
| Заполнение форм (автоопределение AcroForm-полей) | **NOT_IMPLEMENTED** — нет `getForm()`/`AcroForm`/`PDFTextField` нигде в `editor/src` | — | — | `/ru/fill-pdf/` переформулирован честно: «впишите текст в любое место документа вручную», не «умное распознавание полей» |
| Повернуть страницу | SUPPORTED | `lib/pdf/pageOperations.ts:65-78` (`rotatePage`), UI — `components/editor/ThumbnailStrip.tsx:115` | PRO | не в P0 этой волны, см. рекомендацию ниже |
| Переставить/удалить страницы | SUPPORTED | `pageOperations.ts:19-46` (`movePage`/`deletePages`), UI — `ThumbnailStrip.tsx:127,139,151` | PRO | не в P0 |
| Объединить PDF (добавить страницы из другого файла в открытый документ) | SUPPORTED — не отдельный «merge-only» флоу, а функция внутри редактора: `pageOperations.ts:81-90` (`mergePdfs`), UI — `ThumbnailStrip.tsx:192-199` (`onMergeFiles`) | PRO | не в P0, честная формулировка: «добавить страницы из другого PDF» |
| Разделить/извлечь страницы (выбрать → скачать отдельным файлом) | SUPPORTED | `pageOperations.ts:93-99` (`splitPages`), UI — `ThumbnailStrip.tsx:202-209` (`onSplitDownload`) | PRO | не в P0 |
| Отмена операций со страницами (undo) | SUPPORTED | `ThumbnailStrip.tsx` (`undoPageOp`, `pageOpHistory`) | PRO | упоминается как деталь UX, не отдельная страница |
| Пользовательские шрифты | SUPPORTED | `types/edits.ts` (`FontId` допускает `custom-<uuid>`), `store/fontStore.ts` | PRO | упоминается как деталь на `/ru/add-text-to-pdf/` |
| Экспорт без водяного знака, обработка в браузере (файл не уходит на сервер до нажатия «Сохранить») | SUPPORTED | `lib/pdf/exportPdf.ts`, `components/editor/ExportButton.tsx` | — | упоминается как «почему 1PDF» на всех страницах |
| Конвертация (Word/Excel/PPT/JPG/PNG ↔ PDF), OCR, сжатие, водяной знак (добавление), пароль/шифрование | **NOT_IMPLEMENTED** — ни одного совпадения по `watermark`(добавление)/`compress`/`encrypt`/`setPassword` как реальной функции нигде в `editor/src` (только в маркетинговом тексте «no watermark» про экспорт без чужого водяного знака) | — | — | Cluster D остаётся P2/P3 — публикуется только вместе с реальной реализацией |

⚠️ **Обнаружено серьёзное противоречие в продуктовом тексте, не создано этой задачей**: `editor/src/app/routes/Home.tsx:14,20,75,89` (домашний экран самого редактора, унаследован от open-source freePDF) утверждает *«No upload. No paywall. No watermark»* и «every edit and export is free... no signup» — это прямо противоречит реальному развёртыванию, где весь `/editor/*` закрыт PRO-подпиской на уровне nginx `auth_request` (см. `TASK-0002`, `docs/ARCHITECTURE.md`). Эта задача **не изменяет** `Home.tsx` (вне области «SEO landing pages»), но использует только подтверждённо честные формулировки на новых страницах и фиксирует находку здесь и в `docs/GAPS.md` my_office.

### Скорректированный P0 (после аудита)

Исходные 10 пар из таблицы выше подтверждены как совместимые с реальным функционалом (с переформулировкой пп. 7 и 9, отмеченной в таблице аудита). Реализованы в этой задаче.

**Рекомендация для следующей волны (не реализовано в этой задаче, требует отдельного решения о приоритете)**: добавить `/ru/rotate-pdf/`, `/ru/merge-pdf/`, `/ru/split-pdf/`, `/ru/reorder-pdf-pages/`, `/ru/delete-pdf-pages/` (+ UZ-пары) — все пять подтверждены как **SUPPORTED** этим аудитом, в отличие от предположения черновика задачи (Cluster C/D), где они числились как «публиковать только по подтверждённому функционалу» без proведённой проверки. Формулировки должны честно описывать UX «внутри редактора одного документа», а не отдельный upload-only merge/split инструмент.

## Реализация (2026-09-08)

- Foundation (Phase 1) и первая волна P0 (Phase 2, 10 RU/UZ пар + PDF tools hub) реализованы и опубликованы в продакшн `1pdf.uz` в этой же задаче. Код закоммичен и запушен в `1pdf_v1@main` (коммит `654c09c`).
- Генератор `scripts/generate-seo-pages.mjs` (source of truth для контента и capability-ограничений) + `scripts/generate-sitemap.mjs` — статические prerendered HTML-страницы в `admin/public/seo/`, без фреймворка, в стиле существующих `admin/public/*.html`.
- `nginx.freepdf.conf` — новый `location ~ ^/(ru|uz)/([a-z0-9-]+)/?$`, публичный, без `auth_request` (в отличие от `/editor/*`), проксирует на соответствующий статический файл.
- `admin/public/robots.txt` (новый) + `admin/public/sitemap.xml` (23 URL, сгенерирован) — `/portal`, `/admin`, `/editor/`, `/api/`, `/uploads/` исключены.
- `admin/public/index.html` — добавлена ссылка «Инструменты»/«Vositalar» в навигацию и блок из 5 ссылок на инструменты + хаб в футере, с переключением по языку (RU/UZ) через существующий JS-механизм страницы.
- **Опубликованные URL** (все проверены `curl` через продовый `1pdf-nginx`, код ответа 200): `/ru/pdf-editor/`, `/ru/edit-pdf/`, `/ru/add-text-to-pdf/`, `/ru/add-image-to-pdf/`, `/ru/draw-on-pdf/`, `/ru/highlight-pdf/`, `/ru/annotate-pdf/`, `/ru/sign-pdf/`, `/ru/fill-pdf/`, `/ru/pdf-editor-russian/`, `/ru/pdf-tools/` и их UZ-пары (`/uz/pdf-muharriri/`, `/uz/pdf-tahrirlash/`, `/uz/pdfga-matn-qoshish/`, `/uz/pdfga-rasm-qoshish/`, `/uz/pdfga-chizish/`, `/uz/pdf-matnini-belgilash/`, `/uz/pdfga-izoh-qoshish/`, `/uz/pdfga-imzo-qoyish/`, `/uz/pdf-toldirish/`, `/uz/pdf-muharriri-ozbek-tilida/`, `/uz/pdf-vositalari/`) — итого 22 страницы.
- Каждая страница: уникальные `<title>`/description/H1, canonical (self), взаимный `hreflang` (ru/uz/x-default), Open Graph + Twitter meta, `BreadcrumbList` + `FAQPage` JSON-LD (все 48 JSON-LD блоков синтаксически провалидированы), хаб-страницы дополнительно несут `CollectionPage`; `pdf-editor`/`edit-pdf` — `WebApplication`.
- **Обнаружена и исправлена операционная проблема при деплое**: `nginx.freepdf.conf` смонтирован в `docker-compose.yml` как bind-mount ОДНОГО файла (`:ro`); правка файла на хосте инструментом создала новый inode, и контейнер `1pdf-nginx` продолжал обслуживать старую (несуществующую) версию конфига до `docker restart 1pdf-nginx` (не rebuild). После рестарта md5 хоста и контейнера совпали, `nginx -t` прошёл, все новые маршруты вернули 200. Это известный нюанс bind-mount'ов одиночных файлов в Docker, стоит иметь в виду при будущих правках `nginx.freepdf.conf`/`docker-compose.yml` этого проекта.
- Google Search Console / Yandex.Webmaster: файлы/разметка на стороне сайта, необходимые для верификации (мета-теги/файлы подтверждения), не добавлялись — для этого нужны токены верификации из соответствующих аккаунтов владельца продукта, которых нет у агента. Требуется отдельное действие владельца (см. `docs/OPEN_QUESTIONS.md`).
- Не реализовано в этой задаче (осознанно, вне P0): Phase 3 (20-30 доп. страниц под остальные device/use-case intent), Phase 4 (utilities — merge/split/rotate/reorder/delete как отдельные landing, хотя функционал подтверждён SUPPORTED, см. таблицу выше), Phase 5 (guides), Phase 6 (оптимизация по данным Search Console — недоступно без верификации).
