# Schema.org Rules — the authoritative checklist

The single source of truth for how this skill builds and validates Schema.org JSON-LD. Both the Mapper (`prompts/02-mapper.md`, Phase C) and the Validator (`prompts/03-validator.md`) follow this file. `references/schema-template.json` is the worked example that conforms to these rules.

Goal: **valid** markup (passes the Schema Markup Validator and Google Rich Results Test with zero errors) and **high quality** (recommended fields populated where the facts support them), with **no invented data**.

---

## 1. Graph-wide rules

- **One `@graph`** under `{"@context":"https://schema.org","@graph":[…]}`.
- **Self-contained globals:** always include the `WebSite` (`#website`) and `Organization` (`#organization`) nodes — `WebPage` references them via `isPartOf`/`publisher`, so they must be defined or the refs dangle.
- **Unique `@id`s; every reference resolves.** No `@id` referenced without a matching node.
- **No orphan nodes.** Every `Organization` must be referenced by the Person (`worksFor` / `memberOf` / `alumniOf` / `owns`). Founded-but-exited companies still need a role (with `endDate`).
- **Dates: ISO 8601**, prefer full `YYYY-MM-DD`; a bare `YYYY` is a non-critical note for Article-family `datePublished`.
- **Omit, don't placeholder.** No `null`, empty strings, "TBD", "N/A", or `No information available` in the JSON-LD — drop the property/node. (The visible CMS text fields may display `No information available`; structured data never does.)
- **`inLanguage`: "en".**
- **No invention.** Every factual value traces to the Researcher data; schema must be *contextually* correct, not a verbatim mirror of the page HTML.
- **Field storage:** the final JSON is written to `schema-json-ld-3` wrapped in `<script type="application/ld+json">…</script>` (single line).

---

## 2. Per-type property reference

For each type: the properties this skill uses, value types, and which are **required** for a Google rich result.

### Person (central node)
Valid properties used: `name`, `givenName`, `familyName`, `additionalName`, `alternateName`, `honorificSuffix` (always "Fellow" — the Personographer tier, not real post-nominals), `description` (the Professional Identity), `disambiguatingDescription` (one concise reputation line), `image` (ImageObject), `gender` ("Male"/"Female"/Text), `birthDate`, `birthPlace` (Place), `nationality` (Country), `knowsLanguage` (Text/Language), `homeLocation` (Place), `jobTitle` (Text[]), `worksFor` (OrganizationRole[]), `memberOf` (OrganizationRole[]), `alumniOf` (EducationalOrganization[]), `hasCredential` (EducationalOccupationalCredential[]), `hasOccupation` (Occupation[]), `knowsAbout` (Text[]), `award` (Text[]), `netWorth` (MonetaryAmount), `owns` (Thing/Organization refs), `spouse` (Person), `children` (Person[] — named only, never a count), `subjectOf` (@id refs), `performerIn` (@id refs to Events), `sameAs` (URL[]), `identifier` (PropertyValue).
**INVALID on Person:** `additionalProperty` (allowed only on Place/Product/Offer/QuantitativeValue/QualitativeValue/MerchantReturnPolicy). There is **no** Schema.org property for marital-status text, number-of-children count, personal interests, profile category, or cultural origin — keep those in the CMS only.

### OrganizationRole (in `worksFor` / `memberOf`)
`{"@type":"OrganizationRole","worksFor"|"memberOf":{"@id":<org>},"roleName":<Text>,"startDate":<ISO>[,"endDate":<ISO>]}`. The nested property name must match the outer one. This is how org + role + dates are expressed validly.

### Occupation (in `hasOccupation`)
Valid: `name`, `occupationalCategory`, `occupationLocation` (**must be `AdministrativeArea`**, never `Place`), `description`, `skills`, `responsibilities`, `qualifications`, `estimatedSalary`.
**NEVER** put `hiringOrganization`, `startDate`, or `endDate` on `Occupation` (those caused validator errors — they belong on `OrganizationRole`).

### Organization (child nodes)
`name`, `url`, `address` (PostalAddress), `foundingDate`, `dissolutionDate`, `founder` (→ Person, for founded orgs), `parentOrganization` (→ Organization), `alternateName`, `sameAs` (Wikidata). Subtypes when accurate: `GovernmentOrganization`, `EducationalOrganization`, `NGO`. Must be referenced by the Person.

### EducationalOrganization (in `alumniOf` / `recognizedBy`)
`name`, `url`, `sameAs`.

### EducationalOccupationalCredential (in `hasCredential`)
`name`, `credentialCategory` ("degree"/"honorary degree"/…), `educationalLevel`, `recognizedBy` (→ EducationalOrganization).

### Book / Article / ScholarlyArticle / Report (authored works)
`name`, **`headline`** (= title), **`author`** (→ Person `@id`), `datePublished` (full ISO), `publisher` (Organization), `genre` (for Essay/Op-ed/White Paper). Google Article wants `headline`, `image`, `author`, `datePublished` — supply all but `image` (optional).

### Article / TVSeries (profiled-in / external coverage)
`name`, **`headline`** (= the piece's title), `datePublished` (full ISO), `publisher` (Organization), `about` (→ Person), **`author`** (byline `Person` if known, else the publication `Organization`). `image` optional. Documentary → `TVSeries`.

### Event (in `performerIn`)
`name`, `startDate` (ISO), `location` (Place), `performer` (→ Person), `description`. Optional: `endDate`, `organizer`.

### Quotation (selected quotes)
`text` (verbatim), `spokenByCharacter` (→ Person). Note: the live-site quotes carousel reads the `<blockquote>` text — keep attribution out of the quote `text` value.

### VideoObject (editor-provided videos)
**Required by Google:** `name`, `description`, `thumbnailUrl`, **`uploadDate`** (ISO `YYYY-MM-DD` — fetch from the watch page, never skip). Plus `embedUrl` (`https://www.youtube.com/embed/{id}`), `contentUrl` (watch URL), `thumbnailUrl` (`https://i.ytimg.com/vi/{id}/hqdefault.jpg`), `about` (→ Person).

### FAQPage (Q&A)
`about` (→ Person), `mainEntity`: array of `Question` `{name, acceptedAnswer: {"@type":"Answer","text":…}}`.

### WebPage
`@id` (`…#webpage`), `url`, `name` ("{full_name} — {Professional Identity}"), `isPartOf` (→ `#website`), `about` + `mainEntity` (→ Person), `publisher` (→ `#organization`), `inLanguage`, `datePublished`, `dateModified`, `breadcrumb` (BreadcrumbList).

### WebSite (`#website`) — global
`url`, `name`, `description`, `publisher` (→ `#organization`), `inLanguage`.

### Organization (`#organization`) — global publisher
`name`, `url`, `logo` (ImageObject), `publishingPrinciples`, `correctionsPolicy`, `contactPoint` (ContactPoint).

### Value objects
- **MonetaryAmount** (`netWorth`): `currency` + `value`, or `currency` + `minValue`/`maxValue` for a range.
- **ImageObject** (`image`): `url`, `caption`; `width`/`height` if known.
- **Place** / **AdministrativeArea**: `name` + optional `address` (PostalAddress). `occupationLocation` uses **AdministrativeArea**.
- **PostalAddress**: `addressLocality`, `addressRegion`, `addressCountry` (ISO 2-letter code or Country).
- **Country**: `name`.
- **BreadcrumbList**: `itemListElement` of `ListItem {position, name, item}`; the last (current page) item may omit `item`.

---

## 3. Gotchas / anti-patterns (all observed in real runs — do not repeat)

| Symptom in validator | Cause | Fix |
|---|---|---|
| "property not recognized for type Person" | `additionalProperty` on Person | map to `netWorth`/`owns`/`spouse`/`disambiguatingDescription`/`knowsAbout`/`hasCredential`; drop vocab-less editorial fields (keep in CMS) |
| "startDate/endDate/hiringOrganization not recognized for Occupation" | dated employment modeled on `Occupation` | move org+dates to `OrganizationRole`; keep `Occupation` for the profession only |
| "Place is not a valid target for occupationLocation" | `occupationLocation` typed `Place` | use `AdministrativeArea` |
| Orphan orgs shown as separate top-level items | founded/past companies not referenced by Person | add a `worksFor`/`memberOf` `OrganizationRole` (with `endDate` if past) |
| `#website`/`#organization` shown as bare `CreativeWork`/`Thing` | global nodes omitted from page graph | always include both global nodes |
| VideoObject "Missing field uploadDate" | `uploadDate` skipped | fetch the publish date from the watch page; ISO `YYYY-MM-DD` |
| Article "missing headline/author", "invalid date-time" | reference nodes left sparse / year-only date | add `headline` (= title) + `author`; use full `YYYY-MM-DD` |
