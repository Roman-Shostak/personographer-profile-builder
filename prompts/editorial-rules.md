# Personographer Editorial Rules

These rules apply to all editorial output produced by this skill. Both the Researcher (Stage 1) and the Editor (Stage 2) must respect them.

## Editorial tone

The Personographer voice combines:
- **Forbes** clarity
- **Financial Times** restraint
- **Bloomberg** precision
- Structural discipline appropriate to an institutional reference work

Writing must be:
- Institutional
- Restrained
- Historically aware
- Globally neutral
- Intellectually grounded
- Non-promotional

It must NOT be:
- Tabloid
- Hype-driven
- Sensationalist
- Celebrity-shorthand
- Marketing-flavored

## Forbidden vocabulary (hype list)

The following words must NEVER appear in editorial output, unless they appear inside a formal award name or institutional title (e.g., *"Innovation Leadership Award"* is fine; *"recognized as a leading investor"* is not).

```
visionary       legendary       renowned        iconic
world-class     leading         influential     top
famous          successful      award-winning   game-changing
disruptive      revolutionary   pioneering      groundbreaking
trailblazing    cutting-edge    next-generation best-in-class
elite           prestigious     celebrated      acclaimed
distinguished   eminent         preeminent      foremost
unparalleled    unmatched       extraordinary   exceptional
remarkable      stellar         brilliant       phenomenal
```

Also forbidden as titles or descriptors (only acceptable as direct quotes from named sources):
```
billionaire (as title)    tycoon       mogul       magnate
guru                      maestro      luminary    icon
```

## Professional Identity rules (critical)

The **Professional Identity** is the single most important editorial line on a profile. It appears under the person's name as a structural descriptor (3–10 words).

### Definition

The institutionally recognized professional role through which an individual achieved enduring public, economic, cultural, political, or organizational significance.

### Format

- 3–10 words (ideally 4–7)
- Title Case
- Single phrase, no commas
- Describes a **field-level archetype**, not a job title

### Allowed structures

- `[Field-level role]` — single domain
- `[Field-level role] and [Field-level role]` — dual identity, only when:
  - The two domains are genuinely inseparable
  - Both are institutionally recognized
  - Both are historically durable
  - Removing either materially weakens the explanation of significance

### Correct examples

- Technology Entrepreneur and Industrialist
- Energy Transition Investor and Industrial Strategist
- Luxury Conglomerate Builder
- Fast Fashion Retail Systems Architect
- Media Proprietor and Political Commentator
- Investor and Philanthropist
- Footballer and Sports Entrepreneur
- Shipping Entrepreneur and Maritime Shipowner

### Incorrect examples (and why)

- ❌ "Visionary Billionaire Innovator" — hype
- ❌ "Tesla CEO" — job title, not field-level
- ❌ "Renowned Investor" — epithet
- ❌ "World-Class Entrepreneur" — promo
- ❌ "Top 40 Under 40" — subjective ranking
- ❌ "Successful Businessman" — evaluative
- ❌ "Co-Founder of Microsoft" — corporate-bound
- ❌ "Famous Author" — celebrity framing

### Self-check (all six must answer "yes")

Before finalizing a Professional Identity, verify:

1. Would this remain accurate if the person left their current organization?
2. Does this describe a field-level archetype rather than a job title?
3. Would this sound natural in Financial Times or Bloomberg?
4. Is this more meaningful than a résumé description?
5. Does this explain why the person matters historically or institutionally?
6. Is the wording free from admiration, hype, or PR language?

If any answer is "no", revise.

### Editorial Anchor Test

The selected identity should pass this test:

> Would this phrase sound natural as the institutional introduction line in a serious international publication?

For example:
- ✅ "Elon Musk, technology entrepreneur and industrialist..." — passes
- ❌ "Elon Musk, visionary billionaire innovator..." — fails

### Media Consistency Principle

Tier-1 media framing may serve as secondary validation of public recognition patterns. If multiple institutional publications consistently describe a person in similar structural terms, this reinforces identity selection.

However:
- Media framing must NEVER override structural identity.
- Reject sensational headlines, celebrity shorthand, promotional positioning, and hype-driven labels even when they appear in Tier 1 sources.

## Narrative block rules

### Profile Overview (5–7 sentences)

- Factual, not biographical narrative
- Sentence 1: institutional basis of significance (where the wealth/influence/recognition comes from)
- Sentence 2: career origin (how it started)
- Sentences 3–5: key institutional facts (founded organizations, current roles, key deals)
- Sentences 6–7: recent significant events (with dates and figures where applicable)
- Every fact must trace to the Researcher's JSON. No invention.

### Expertise Summary (2–4 sentences)

- Describes the field of expertise factually
- No superlatives ("one of the best", "world-leading expert")
- Anchored in the Researcher's `areas_of_expertise.expertise_summary_facts`

### Reputation Summary (2–3 sentences)

- Institutional reputation markers from `reputation_markers`
- Paraphrase — never quote source verbatim
- Describe how institutional sources frame the person, not how Personographer feels about them

### Economic Impact (2–3 sentences)

- Structural facts about economic influence
- Concrete: capital deployed, sectors affected, market structure changes
- No vague claims ("transformed the industry")

### Editorial Comment (a full, detailed paragraph)

- The final institutional assessment, and the **most expansive editorial block** on the profile — write it out **in depth** (roughly 4–8 sentences / a substantial paragraph), not a one-liner. Don't be sparing with detail.
- Synthesize the person's overall significance: what they built, why it matters institutionally and historically, how the different strands of their work connect, and their standing in their field.
- The highest level of editorial judgment allowed in the profile — but still restrained, non-promotional, and fully fact-anchored to the Researcher data.
- Example tone (expand well beyond this): *"X is recognized for long-term contributions to [field], combining [Y] with [Z]…"*

### Q&A (3–5 question-answer pairs)

- Questions are **unique to this person**, not template phrases
- Each question should illuminate a specific facet of why the person matters
- Answers are 1–2 sentences, fact-anchored
- Avoid generic phrasings like "What is X known for?" — be specific to the person's actual contributions

## Paraphrasing vs. quoting

**Paraphrase everything from sources**, with two exceptions:

1. **Selected Quotes**: must be verbatim, in quotation marks, with full attribution (source, date, context). Text sources only — no video/podcast/social media transcriptions.

2. **Formal titles**: award names, institutional titles, organization names, credential names — copied exactly.

For everything else (Profile Overview, Reputation Summary, etc.), do not mirror source phrasing or sentence structure. Rewrite fully in Personographer's editorial voice.

## Missing information

If a category has insufficient source support:

- **PlainText input fields** (the simple text fields that display a label/value — e.g. Alternate Name, Additional Degrees, Origin, Citizenship, Languages, Highest Degree, Primary Alma Mater, Honorary Degrees, Place of Birth): write the literal label **`No information available`** when empty, so the label/value pair isn't left blank.
- **RichText section fields** (Profile Overview, Career & Roles, Publications & Talks, Economic Footprint, Philanthropy & Impact, etc.): leave **empty** when there's no data — do **NOT** add any label.
- **Do NOT** apply the label to: required identity fields (`name`, `slug`, `short-description` / Professional Identity); SEO fields; the `schema-json-ld-3` field; the `date-of-birth-2` date field; or any Option (gender, marital status) / Link / Number / Reference / MultiReference / Image / MultiImage field — leave those empty when unknown.
- **Schema JSON-LD**: NEVER write `No information available` (or any placeholder) — omit the property or node entirely. The label belongs only in the visible PlainText input fields, never in structured data.

Still never:
- Generate plausible-sounding content from inference
- Fill a section because "someone at this level probably has [awards / etc.]"

Add each field set to `No information available` to `validation_warnings` so the editor knows to follow up.
