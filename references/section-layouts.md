# RichText Section Layouts

Canonical reference for how each Webflow **RichText** CMS field must be structured so the published-site transform script renders it correctly. The Mapper (`prompts/02-mapper.md`, Phase B.4) writes HTML according to these layouts; this file is the source of truth for the exact structure and is expanded one section at a time as each layout is verified against the live site.

## How the live site renders RichText

The published Personographer site runs a JavaScript transform script (in the Webflow site/page custom code — **not** in this repo) that rewrites the DOM of RichText fields after load. It keys off:

- a `data-cms-layout` attribute on the rich-text element (set in the Webflow Designer template, not by the Mapper), and
- semantic markers in the HTML the Mapper writes.

### Global rules (mirror of Mapper B.0)

1. **Plain `<p>` is the default** for all body / descriptive text.
2. **A `<p>` whose entire content is `<strong>…</strong>` is a structural marker** (a label such as a role, period, or key) — the script lifts it into its own styled slot. Never use inline `<strong>` inside running text.
3. **`<ul>`/`<li>` lists are allowed in only three places** — Profile Overview, Publications & Talks (under the `Notable Mentions` subsection only), and Personal Interests. Everywhere else use plain `<p>` (one paragraph per item).
4. **Headings** (`<h3>`, `<h4>`) carry the grouping the script reads — use the exact levels each section specifies.

### Status legend

- ✅ **CONFIRMED** — verified rendering correctly on the live site.
- 🟡 **PENDING** — not yet verified; current Mapper template is a best guess and bold-marker placement may still change.

> `data-cms-layout` / `data-cms-custom` values below are the keys the transform script expects. The actual attribute on each rich-text element is set in the site's Webflow Designer; treat PENDING bindings as "to confirm against the live render".

---

## ✅ Career & Roles — `career-roles`

- **data-cms-layout:** `career-and-roles` (structure `h3-h4-nested`)
- Verified working — this is the reference layout confirmed on the live site.

Structure:
- `<h3>` = group (Career Timeline / Board & Committee Roles / Founded Organizations)
- `<h4>` = organization
- `<p><strong>{Role} / {Location}</strong></p>` = role marker
- `<p><strong>{period}</strong></p>` = period marker
- `<p>{description}</p>` = one or more plain description paragraphs
- Multiple role/period pairs may stack under one `<h4>` before the description paragraphs.

```html
<h3>Career Timeline</h3>
<h4>Nordlys Arts Platform</h4>
<p><strong>Founding Director / Global HQ</strong></p>
<p><strong>1997 – Present</strong></p>
<p>{description paragraph}</p>
<p>{description paragraph}</p>
<!-- a second role under the SAME organization stacks before the next <h4> -->
<p><strong>{Role} / {Location}</strong></p>
<p><strong>{period}</strong></p>
<p>{description paragraph}</p>

<h4>Meridian Cultural Trust</h4>
<p><strong>Executive Chair / Global HQ</strong></p>
<p><strong>2005 – Present</strong></p>
<p>{description paragraph}</p>

<h3>Board & Committee Roles</h3>
<h4>Industry Council</h4>
<p><strong>Board Member</strong></p>
<p><strong>2012 – Present</strong></p>
<p>{description paragraph}</p>

<h3>Founded Organizations</h3>
<h4>Nordlys Arts Platform</h4>
<p><strong>Founder</strong></p>
<p><strong>Founded in 1997</strong></p>
<p>{description paragraph}</p>
```

---

## ✅ Philanthropic Roles — `philanthropic-roles`

- **data-cms-layout:** `philanthropic-roles` (structure `h4-flat`)
- Flat version of Career & Roles: `<h4>` organization → bold role/location marker → bold period marker → plain description. No `<h3>` group level.

```html
<h4>{Organization}</h4>
<p><strong>{Role} / {Location}</strong></p>
<p><strong>{start_date} – {end_date}</strong></p>
<p>{description}</p>
```

## ✅ Economic Footprint — `economic-footprint`

- **data-cms-layout:** `economic-footprint` (structure `h3-h4-economic`)
- One `<h3>` group ("Estimated Net Worth"). The net-worth value is an `<h4>` (rendered large), the year + source a plain `<p>` under it (rendered gold). `Known Assets` and `Economic Impact` are standalone **bold** labels, each followed by a plain `<p>`.

```html
<h3>Estimated Net Worth</h3>
<h4>{net worth value}</h4>
<p>{year}, {source}</p>
<p><strong>Known Assets</strong></p>
<p>{known assets text}</p>
<p><strong>Economic Impact</strong></p>
<p>{economic impact text}</p>
```

## ✅ Awards & Recognition — `awards-recognition`

- **data-cms-layout:** `awards-and-recognition`
- `Awards and Honors`: the year is a standalone **bold** marker, the award name a plain `<p>` below it. `Reputation Summary`: one plain `<p>` per sentence (not one big paragraph).

```html
<h3>Awards and Honors</h3>
<p><strong>{year}</strong></p>
<p>{award_name}</p>
<!-- repeat the year + award pair per award -->

<h3>Reputation Summary</h3>
<p>{sentence 1}</p>
<p>{sentence 2}</p>
```

## ✅ Publications & Talks — `publications-talks`

- **data-cms-layout:** `publications-and-talks`
- Confirmed structure. Each subsection is grouped under an `<h3>`. The script reads bold/plain pairs (a standalone `<p><strong>…</strong></p>` label followed by a plain `<p>`); `<li>` is used ONLY under `Notable Mentions`.

```html
<h3>Authored Works</h3>
<p><strong>— {Type}, {publisher}, {year}</strong></p>
<p>{title}</p>
<!-- repeat the bold line + title per work -->

<h3>Profiled In</h3>
<p><strong>{Type}</strong></p>
<p>{publication}, {year}</p>
<!-- repeat per item -->

<h3>Notable Mentions</h3>
<ul>
<li>{mention}</li>
</ul>
<!-- the ONLY list allowed in this field -->

<h3>Public Appearances</h3>
<p><strong>{Event Name}, {date}</strong></p>
<p>{role / what they did}</p>
<!-- repeat: bold line = event + readable date (e.g. "June 20, 2025"), plain line = the role -->
```

## ✅ Philanthropy & Impact — `philanthropy-impact`

- **data-cms-layout:** `philanthropy-and-impact`
- No lists — each item is its own plain `<p>` under its `<h3>` (Areas of Influence / Impact Initiatives / Patronage & Sponsorship). No bold markers.

```html
<h3>Areas of Influence</h3>
<p>{area}</p>

<h3>Impact Initiatives</h3>
<p>{initiative}</p>

<h3>Patronage & Sponsorship</h3>
<p>{text}</p>
```

## ✅ Current Positions — `current-positions`

- One plain `<p>` per active role — **no bold**.

```html
<p>{role} at {organization}</p>
<!-- repeat per active role -->
```

---

## ✅ Custom-rendered fields (carousels / accordion)

These are RichText fields the script transforms into interactive components via a `data-cms-custom` attribute. Live-site render confirmed.

- **Selected Quotes — `selected-quotes`** (`data-cms-custom="quotes"`): one `<blockquote>` per quote → Swiper carousel.
- **Q&A — `profile-qa`** (`data-cms-custom="profile-qa"`): `<h3>` question immediately followed by a `<p>` answer → accordion.
- **Videos — `videos`** (`data-cms-custom="videos"`): one `<a href="{youtube_url}">` per link → Swiper carousel; the script fetches each title from YouTube. Filled only if the editor provided links at the review checkpoint.

## Plain fields (no transform)

Standard RichText, plain paragraphs (lists allowed where the global rules permit):

- **Profile Overview — `profile-overview`** ✅ each point is its own `<li>` inside a single `<ul>` (no bold): `<ul><li>{point}</li>…</ul>`
- Expertise Summary — `expertise-summary`
- Education Highlights — `education-highlights-2` (plain `<p>` per highlight)
- Personal Interests — `personal-interests` (list allowed here)
