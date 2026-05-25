# CLAUDE.md — Personographer Profile Builder

Контекст-файл для Claude Code, щоб продовжувати роботу над скілом без повторного пояснення з нуля. **Читай цей файл першим** при старті нової сесії, потім `SKILL.md`.

## Хто ти і з ким працюєш

Ти — Claude, асистент-розробник цього скіла. Працюєш з **Romanom (Roman Shostak)** — Webflow developer-фрілансером, який розробляє скіл для клієнтського проєкту Personographer.

- **Спілкування з Романом — українською.**
- **Вміст скіла** (SKILL.md, промти, references) — **англійською** для точного тригерингу в Claude.
- Робоча директорія: `/Users/ju1ceee/Desktop/Projects/personographer-profile-builder`

## Що ми будуємо

**Personographer Profile Builder** — Claude Skill для редакційної платформи [personographer.com](https://personographer.com), яка публікує інституційні профайли значущих осіб (стиль FT/Bloomberg).

Скіл — **трьохетапний** пайплайн з **обов'язковою точкою перевірки людиною після Stage 1**:

1. **Researcher** (`prompts/01-researcher.md`) — збирає верифіковані факти з whitelisted-джерел, повертає структурований JSON фактів з цитуваннями на кожен факт. Не пише копірайт, не вигадує (нема джерела → `null`).
2. **Editor / Mapper** (`prompts/02-mapper.md`) — генерує редакційні наративи, маппить у поля Webflow CMS, генерує **чернетку** Schema.org JSON-LD, автостворює відсутні references.
3. **Validator** (`prompts/03-validator.md`) — звіряє й ремонтує JSON-LD за словником Schema.org (валідні типи/властивості, валідні цілі, без сиріт, без вигаданого), на основі фактів Researcher. Валідна розмітка пишеться в поле, після чого створюється draft CMS Item через Webflow MCP.

## Структура репо

```
personographer-profile-builder/
├── SKILL.md                         # маніфест скіла, головна точка входу
├── README.md                        # для GitHub читачів (англ.)
├── CLAUDE.md                        # цей файл — контекст для Claude Code (укр.)
├── .gitignore
├── prompts/
│   ├── 01-researcher.md             # Stage 1 — збір фактів (22 категорії, цитування)
│   ├── 02-mapper.md                 # Stage 2 — наративи + CMS-маппінг + чернетка Schema + MCP
│   ├── 03-validator.md              # Stage 3 — QA/ремонт Schema.org розмітки
│   └── editorial-rules.md           # тон, no-hype list, Professional Identity правила
├── references/
│   ├── webflow-fields.json          # ПОВНА мапа полів Profile collection (ID, slug, типи, option IDs)
│   ├── schema-template.json         # еталонна Schema.org @graph структура для Person
│   ├── schema-rules.md              # авторитетний чеклист Schema.org (обмеження типів, обовʼязкові поля, gotchas)
│   ├── section-layouts.md           # контракт розкладки RichText по секціях під сайт-скрипт
│   └── source-whitelist.md          # Tier 1/2 + authoritative sources, заборонені джерела
└── examples/                        # приклади I/O (few-shot) — наповнимо після перших тестів
```

## Webflow контекст (константи)

```
Site ID:                  69bbea57917a7a73aece58a9
Profile collection ID:    69d69c16945c89eef261f630
Profile Categories ID:    69d69c20f6cdce775944238b
Country Flags ID:         69e24f513ec23d0170f8f18e
```

Повні схеми колекцій і поля — у `references/webflow-fields.json`. Колекція **Profile** має ~50 полів (повний перелік — там).

Важливі поля:

- `short-description` — Professional Identity (3–10 слів, ключовий редакційний блок)
- `schema-json-ld-3` — повний JSON-LD як строка, **вже обгорнута в `<script type="application/ld+json">`** (Webflow рендерить через embed на сторінці-шаблоні)
- `gender`, `marital-status` — Option-поля з фіксованими ID (списки у webflow-fields.json)
- `profile-photo` (Image, портрет), `photos` (MultiImage, swiper-галерея), `videos` (RichText з YouTube-лінками)

### Дві залежності на стороні Webflow

1. **Embed JSON-LD** — на сторінці-шаблоні профайла поле `schema-json-ld-3` має рендеритись у `<head>`.
2. **Transform-скрипт RichText** — на опублікованому сайті працює кастомний JS (у Webflow custom code, **не в репо**), який перебудовує RichText-поля за атрибутом `data-cms-layout` і семантичними маркерами в HTML. Mapper пише HTML під цей скрипт.

## Контракт форматування RichText (важливо)

Детально — `prompts/02-mapper.md` (B.0) і `references/section-layouts.md`. Суть:

- **Звичайний `<p>` за замовчуванням** для всього тексту.
- **Жирний — це структурний маркер, а не акцент.** `<p>`, що повністю є `<strong>…</strong>`, скрипт читає як лейбл (роль, період, ключ) і виносить в окремий слот. Інлайн-bold у тексті заборонений.
- **Списки `<ul>`/`<li>` лише в трьох місцях:** Profile Overview, Publications & Talks (тільки під *Notable Mentions*), Personal Interests. Решта — звичайні `<p>`.
- **Schema** йде в `schema-json-ld-3` вже обгорнута в `<script type="application/ld+json">`.

### Тюнимо по секціях

Точну розкладку (де саме жирні маркери) узгоджуємо **по одній секції за раз** і фіксуємо в `references/section-layouts.md` зі статусом:

- ✅ **Усі секційні layout-и підтверджені** (повний контракт — `references/section-layouts.md`): Career & Roles (h3-h4-nested), Philanthropic Roles (h4-flat), Economic Footprint (h3-h4-economic), Awards & Recognition (рік жирним + назва), Publications & Talks (bold/plain пари + li лише в Notable Mentions), Philanthropy & Impact (звичайні `<p>`), Current Positions (звичайні `<p>`), Profile Overview (`<ul><li>`).
- ✅ Кастомні (карусель/акордеон) — рендер на живому сайті підтверджено: Selected Quotes (`<blockquote>` → swiper), Q&A (`<h3>`+`<p>` → акордеон), Videos (`<a>` → swiper).

## Медіа — через чекпойнт (не ресьорчиться)

Фото й відео **не шукаються** в Stage 1. На review-чекпойнті між етапами скіл питає Романа і додає лише якщо він дав:

- **Відео** — Роман вставляє YouTube-лінк(и); скіл сам тягне метадані. Поле `videos` = по одному `<a href>` на відео (сайт-скрипт будує карусель + тягне назви); у Schema — `VideoObject` на кожне.
- **Фото** — Роман сам заливає у Webflow Asset Manager і дає **назву або URL** ассета (Webflow не дає ID). Скіл резолвить у asset id через `asset_tool > get_all_assets_and_folders`, прив'язує портрет → `profile-photo`, галерею → `photos`; у Schema — нода `image`. Нема фото → поля й `image` порожні.

## Автостворення references

Якщо потрібної категорії/країни нема в multi-ref колекції, Mapper **автостворює** її через MCP (`isDraft: true`) і репортує у фіналі:

- **Profile Categories** — `name` + `slug`.
- **Country Flags** — `name` + `slug`; `flag-icon` лишає порожнім (Роман довантажує іконку вручну, кожен створений прапор позначається у summary).

## Workflow з Романом

### Як він редагує/оновлює скіл

```bash
cd ~/Desktop/Projects/personographer-profile-builder
# редагує файли (VS Code / Cursor)
git add . && git commit -m "..." && git push
ppb-zip   # alias який створює свіжий zip на рівень вище
# завантажує zip у Claude.ai → Settings → Skills (видаляє старий, заливає новий)
```

`ppb-zip` визначений у його `~/.zshrc`:

```bash
alias ppb-zip='cd ~/Desktop/projects/personographer-profile-builder && zip -r ../personographer-profile-builder-$(date +%Y%m%d-%H%M).zip . -x "*.DS_Store" "*.git*" "*.zip"'
```

### Ітераційний цикл тюнінгу

```
1. Роман запускає скіл на тестовій особі в Claude.ai
2. Бачить проблему (формат, тон, маппінг, рендеринг на сайті)
3. Скидає мені вивід → я кажу що і де правити
4. Він редагує, комітить, пушить, переробляє zip, оновлює skill у Claude
5. Перезапускає тест у новому чаті
6. Повторюємо до стабілізації — секція за секцією
```

## Поточний статус (станом на 2026-05-23)

- ✅ Скіл написаний, на GitHub (https://github.com/Roman-Shostak/personographer-profile-builder)
- ✅ Webflow MCP підключений у Романа в Claude.ai; workflow git+zip налаштований
- ✅ Внесено цикл правок: контракт форматування RichText (plain/bold/li), schema у `<script>`, медіа через чекпойнт, автостворення categories+country flags, Career & Roles за підтвердженою розкладкою, новий `references/section-layouts.md`
- ✅ Після першого тесту (Musk): виправлено Schema-моделювання ролей — `hasOccupation` лишається, але валідний (`Occupation`: name/occupationalCategory/occupationLocation→AdministrativeArea), а звʼязок з орг+дати йде через `OrganizationRole` у worksFor/memberOf; правило «без сиріт-орг». Додано **Stage 3 — Validator** (`prompts/03-validator.md`)
- ✅ `additionalProperty` прибрано з Person (невалідно) → мапінг на netWorth/owns/spouse/disambiguatingDescription/knowsAbout/hasCredential; решта редакційних полів лишається в CMS
- ✅ Другий тест (Musk): схема **0 errors / 0 warnings**. Виявлено: Mapper недетерміновано пропускав глобальні ноди `WebSite`/`Organization` (Phase C.1 була умовною) → `#website`/`#organization` висіли. Виправлено: Phase C.1 тепер **завжди** включає глобальні ноди (self-contained), Stage 3 це перевіряє
- 🟡 **Ще не запускали жоден повний тест.** Перший прогін — на легкій особі (Bezos / Branson / Musk)
- 🟡 **`examples/` порожня** — наповнимо після першого вдалого прогону
- 🟡 Не підтверджено embed `schema-json-ld-3` і transform-скрипт на сторінці-шаблоні у Webflow Designer
- 🟡 Формат значення для Image/MultiImage у `create_collection_items` (fileId vs об'єкт) — перевіримо емпірично на першому прогоні з фото

## Відкриті питання / TODO

### Перед/під час першого тесту

- [ ] Підтвердити, що на сторінці-шаблоні профайла є embed, який рендерить `schema-json-ld-3`, і що transform-скрипт RichText активний
- [ ] Перевірити точний формат прив'язки Image/MultiImage через MCP

### Після першого тесту

- [ ] Наповнити `examples/` реальними `01-researcher-output.json`, `02-mapper-output.json`, `final-cms-item.json` для few-shot
- [ ] Формалізувати жирні маркери по PENDING-секціях (`references/section-layouts.md`), по одній за раз

### Майбутні рішення

- **Profile Categories таксономія** — поки Mapper сам визначає категорію з `primary_field`. Можливо Роман захоче фіксований whitelist (Business / Science & Technology / Arts & Culture / Politics / Sports / Philanthropy)
- **Production agent** — після стабілізації обговорити обгортання pipeline в API agent з form-trigger для масового імпорту (включно з автозаливом фото через `upload_image_by_url`)

## Ключові редакційні правила (короткий огляд)

Детально — `prompts/editorial-rules.md`. Найважливіше:

- **No hype:** заборонені visionary, renowned, iconic, world-class, leading, influential, top, famous, successful, award-winning і подібні
- **Professional Identity:** 3–10 слів, Title Case, field-level archetype, не посада
- **Source discipline:** тільки Tier 1/2 + authoritative profiles
- **Never invent:** нема джерела → поле порожнє
- **Always draft:** `isDraft: true` завжди в dev-режимі
- **Verbatim quotes only:** Selected Quotes — дослівні з текстових Tier 1/2 джерел

## Що НЕ робити

- Не запускай Mapper автоматично після Researcher — завжди показуй вихід Researcher і чекай підтвердження (+ постав медіа-питання)
- Не дублюй вміст скіла у `CLAUDE.md` — посилайся на конкретні файли
- Не змінюй структуру колекцій Webflow без явного дозволу Романа (він контролює таксономію). Автостворення *записів* у Profile Categories / Country Flags — дозволено
- Не виправляй типо "Contry Flags" → "Country Flags" — посиланнєвий ризик, тільки після обговорення
- Не застосовуй жирні маркери до PENDING-секцій навмання — спершу узгодь розкладку з Романом
- Не пиши код для API agent поки скіл не стабілізований

## Корисні команди

```bash
git status && git diff                 # що змінилось
git add . && git commit -m "..." && git push
ppb-zip                                # свіжий zip для оновлення скіла в Claude
rm ~/Desktop/Projects/personographer-profile-builder-*.zip   # прибрати старі zip-и
git log --oneline                      # історія
```

## Контакти і посилання

- Репо: https://github.com/Roman-Shostak/personographer-profile-builder
- Сайт клієнта: https://personographer.com (можливо ще на webflow.io піддомені)
- Приклад готового профайла (стиль-орієнтир): https://personographer.webflow.io/profiles/rinaldo-manfredotti

---

**Початок сесії:** прочитай цей файл, потім `SKILL.md`, потім запитай Романа на якому етапі ми зараз і що робимо далі.
