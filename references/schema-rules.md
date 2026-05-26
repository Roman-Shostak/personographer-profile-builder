# Schema.org Rules — companion to the canonical template

`references/schema-template.json` is the **canonical schema the Personographer site expects** — it is the source of truth for *structure* (node types, `@id` patterns, who-points-to-whom). This file is the *companion*: it says which datum goes in which property, value formats, and the gotchas to avoid. Both the Mapper (`prompts/02-mapper.md`, Phase C) and the Validator (`prompts/03-validator.md`) follow the template first and this file second.

Goal: a graph that **matches the canonical template**, **filled with real facts**, with **empty things omitted** and **no invented data**. (The template deliberately carries an `additionalProperty` block on the `Person` to keep all editorial content in the structured data; a strict validator may show a soft notice on it — that is expected and accepted for this site.)

---

## 1. Graph-wide rules

- **One `@graph`** under `{"@context":"https://schema.org","@graph":[…]}`.
- **Conform to the template.** Reproduce its node types and relationship directions; fill, don't restructure. You may add a valid property the facts support, but never add new node types or re-route links.
- **Self-contained globals:** always include the `WebSite` (`#website`) and `Organization` (`#organization`) publisher nodes — `ProfilePage` references them via `isPartOf`/`publisher`, so they must be defined or the refs dangle.
- **Unique `@id`s; every reference resolves.** No `@id` referenced without a matching node.
- **No orphan Organizations.** Every `Organization` must be referenced by the Person via `worksFor` or `memberOf`. Founded-but-exited companies still get a role (with `endDate`).
- **Dates: ISO 8601** (`YYYY`, `YYYY-MM`, `YYYY-MM-DD`); use the most precise the facts support.
- **Omit, don't placeholder.** No `null`, empty strings, "TBD", "N/A", `No information available`, or `REPLACE_WITH_…` in the JSON-LD — drop the property/node. (Visible CMS text fields may display `No information available`; structured data never does.)
- **`inLanguage`: "en".**
- **No invention.** Every factual value traces to the Researcher data; the schema is *contextually* correct, not a verbatim mirror of the page HTML.
- **Field storage:** the final JSON is written to `schema-json-ld-3` wrapped in `<script type="application/ld+json">…</script>` (single line).

---

## 2. Node inventory & `@id` patterns

| Node | `@type` | `@id` |
|---|---|---|
| Site | `WebSite` | `https://personographer.com/#website` |
| Publisher | `Organization` | `https://personographer.com/#organization` |
| Page | `ProfilePage` | `…/profiles/{slug}#profilepage` |
| Breadcrumb | `BreadcrumbList` | `…/profiles/{slug}#breadcrumb` |
| Subject | `Person` | `…/profiles/{slug}#person` |
| Org (each) | `Organization` | `…/organizations/{org-slug}#organization` |
| Authored work | `Book`/`ScholarlyArticle`/`CreativeWork` | `…/works/{work-slug}#work` |
| Coverage | `Article`/`CreativeWork` | `…/articles/{slug}#article` (documentary → `#creativework`) |
| Event | `Event` | `…/events/{slug}#event` |
| Quote | `Quotation` | `…/profiles/{slug}#quote-N` |
| Philanthropy | `Project` | `…/projects/{slug}#project` |
| Video | `VideoObject` | `…/profiles/{slug}#video-{slug}` |
| Q&A | `FAQPage` | `…/profiles/{slug}#faq` |
| Related | `ItemList` | `…/profiles/{slug}#related-persons` |

Drop any node whose data is absent (no videos → no `VideoObject`; no philanthropy → no `Project`; first profile in a category → no `ItemList`).

---

## 3. Per-type property reference

### Person (central node)
- **Identity:** `@id`, `url`, `mainEntityOfPage` (→ `#profilepage`), `name`, `givenName`, `familyName`, `alternateName`, `description` (the Professional Identity). (No `additionalName`/`honorificSuffix` — "Fellow" goes in `additionalProperty` → "Profile Status".)
- **`identifier`** (array of `PropertyValue`): Wikidata `{propertyID:"Wikidata", value, url}` (omit if no QID) **+ always** `{propertyID:"Personographer ID", value:"personographer:person:{slug}"}`.
- **`sameAs`** (URL[]): the person's external links (Wikidata, Forbes, LinkedIn, X, Instagram, Facebook, official site) — only those that exist.
- **Bio:** `image` (ImageObject — only if a portrait was provided), `gender`, `birthDate`, `birthPlace` (Place + PostalAddress), `nationality` (Country), `workLocation` (Place), `knowsLanguage` (Text[]), `homeLocation` (Place).
- **Roles:** `jobTitle` (Text[]); `worksFor` (array of **plain `{"@id"}` org refs**, employment/operating roles); `memberOf` (array of **plain `{"@id"}` org refs**, board/committee/foundation/advisory roles). **No dates on the Person** — they live on the org's `member`/`OrganizationRole`.
- **Education/expertise:** `alumniOf` (EducationalOrganization[]), `hasCredential` (EducationalOccupationalCredential[]), `hasOccupation` (a **single** `Occupation`), `knowsAbout` (Text[]), `award` (Text[], `"Name, Year"`).
- **Money:** `netWorth` (MonetaryAmount).
- **Coverage/appearances:** `subjectOf` (refs to coverage nodes only), `performerIn` (refs to Event nodes).
- **`additionalProperty`** (array of `PropertyValue`) — the editorial catch-all (see §4). Expected on the Person in this template.

### OrganizationRole (on the **Organization** node's `member`)
`{"@type":"OrganizationRole","member":{"@id":<person>},"roleName":<Text>,"startDate":<ISO>[,"endDate":<ISO>],"description":<Text>}`. This is where org + role + dates are expressed — on the org side, pointing back to the person. `member` may be a single object or an array (multiple roles at one org).

### Occupation (in `hasOccupation`, single object)
Valid props only: `name` (= Professional Identity), `occupationalCategory` (= Profile Category), `skills` (Text[] of expertise areas), optional `description`. **NEVER** `startDate`/`endDate`/`hiringOrganization`.

### Organization (child nodes)
`name`, `location` (Place), `founder` (→ Person, founded orgs), `foundingDate`, `member` (OrganizationRole — see above). Must be referenced by the Person via `worksFor`/`memberOf`.

### EducationalOrganization (in `alumniOf` / `recognizedBy`)
`name`, `url`, `sameAs`.

### EducationalOccupationalCredential (in `hasCredential`)
`name`, `credentialCategory` (`"degree"` or `"honorary degree"`), `educationalLevel`, `recognizedBy` (→ EducationalOrganization). One entry per degree **and** per honorary degree — add all the person holds.

### Authored works — `Book` / `ScholarlyArticle` / `CreativeWork`
`name`, `author` (→ Person `@id`), `datePublished` (ISO; year acceptable), `publisher` (Organization), `genre`. Type: Book → `Book`; Research Paper → `ScholarlyArticle`; Essay/Op-ed/White Paper/Report/other → `CreativeWork`. Optional `headline` (= title).

### Coverage — `Article` / `CreativeWork` (profiled-in)
`name` (real title if known, else type label), `datePublished` (ISO; year acceptable), `about` (→ Person), `publisher` (Organization). Optional additions when facts support: `headline` (= real title), `author` (byline Person, else publication Organization). Documentary → `CreativeWork`; else `Article`. These are the nodes in `Person.subjectOf`.

### Event (in `performerIn`)
`name`, `startDate` (ISO; year acceptable), `description`, `performer` (→ Person); optional `location` (Place).

### Quotation (selected quotes)
`text` (verbatim, no attribution inside), `spokenByCharacter` (→ Person). The live-site quotes carousel reads the `<blockquote>` text — keep attribution out of `text`.

### Project (philanthropy)
`name`, `description`, `about` (the area of influence, as a string), and **either** `funder` (→ Person, funded initiatives/research) **or** `sponsor` (→ Person, patronage/sponsorship).

### VideoObject (editor-provided videos)
**Required:** `name`, `description`, `thumbnailUrl` (`https://i.ytimg.com/vi/{id}/hqdefault.jpg`), **`uploadDate`** (ISO `YYYY-MM-DD` — fetch from the watch page, never skip). Plus `embedUrl` (`https://www.youtube.com/embed/{id}`), `contentUrl` (watch URL), `about` (→ Person).

### FAQPage (Q&A)
`mainEntity`: array of `Question` `{name, acceptedAnswer:{"@type":"Answer","text":…}}`. Every answer carries real text — no placeholders.

### ProfilePage (the page)
`@id` (`…#profilepage`), `url`, `name` ("{full_name} Profile"), `headline` ("{full_name} — {Professional Identity}"), `description`, `inLanguage`, `isPartOf` (→ `#website`), `publisher` (→ `#organization`), `publishingPrinciples`, `mainEntity` + `about` (→ Person), `breadcrumb` (→ `#breadcrumb`).

### BreadcrumbList (separate node)
`@id` (`…#breadcrumb`), `itemListElement` of `ListItem {position, name, item}` — three levels: Main → Persons → {full_name} (the last one **does** carry its `item` URL in this template).

### WebSite (`#website`) — global
`url`, `name`, `description`, `inLanguage`, `publisher` (→ `#organization`), `publishingPrinciples`.

### Organization (`#organization`) — global publisher
`name`, `url`, `logo` (ImageObject), `publishingPrinciples`, `correctionsPolicy`, `contactPoint` (ContactPoint), `sameAs` (Personographer's own socials — only real ones; drop placeholders). Site constants: reproduce verbatim from the template.

### ItemList (`#related-persons`) — optional
`name` "Leaders in the same category", `itemListElement` of `ListItem {position, item:{Person @id, name, url}}` for up to 4 other same-category profiles. Omit the whole node if none exist.

### Value objects
- **MonetaryAmount** (`netWorth`): `currency` + `value`, or `currency` + `minValue`/`maxValue`; optional `description` ("Estimated $1–3B").
- **ImageObject** (`image`): `url`, `caption`; `width`/`height` if known.
- **Place** / **PostalAddress** / **Country**: `name`; address uses `addressLocality`/`addressRegion`/`addressCountry` (ISO code).
- **PropertyValue** (`identifier`, `additionalProperty`): `propertyID`/`name` + `value`.

---

## 4. `additionalProperty` catalog (on the Person)

Array of `{"@type":"PropertyValue","name":"…","value":"…"}`, plain text, in this order. Emit an entry **only if its data exists**:

`Profile Category`, `Profile Status` (= "Fellow", always), `Origin`, `Notable Mentions`, `Education Highlights`, `Primary Field of Expertise`, `Expertise Summary`, `Reputation Summary`, `Known Assets`, `Economic Impact`, `Areas of Influence` (`"A; B; C"`), `Impact Initiatives` (`"A; B"`), `Patronage & Sponsorship`, `Marital Status`, `Number of Children`, `Personal Interests` (`"A; B; C"`), `Editorial Comment`.

Never write `No information available` into a `value` — omit the entry instead.

---

## 5. Gotchas / anti-patterns (do not repeat)

| Symptom | Cause | Fix |
|---|---|---|
| `REPLACE_WITH_…` shows up in output | template placeholder not substituted | replace with the real value, or omit that property/array entry |
| `#website`/`#organization` shown as bare `CreativeWork`/`Thing` | global nodes omitted from page graph | always include both global nodes |
| Orphan org shown as a separate top-level item | org not referenced by Person | add it to `worksFor`/`memberOf`; keep the `OrganizationRole` on the org's `member` |
| Dates on the Person's `worksFor`/`memberOf` | roles modeled on the Person side | move role+dates to the **Organization** node's `member`/`OrganizationRole`; keep only `{"@id"}` refs on the Person |
| "startDate/hiringOrganization not recognized for Occupation" | dated employment put on `Occupation` | keep `Occupation` to `name`/`occupationalCategory`/`skills`; dates live on `OrganizationRole` |
| VideoObject "Missing field uploadDate" | `uploadDate` skipped | fetch the publish date from the watch page; ISO `YYYY-MM-DD` |
| FAQ answer reads "should be added…" | placeholder answer left in | write the real answer from the facts, or drop the Question |
| `No information available` inside a schema value | visible-CMS label leaked into JSON-LD | remove it — placeholders live only in visible CMS fields |
