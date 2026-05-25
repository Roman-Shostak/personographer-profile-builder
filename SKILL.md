---
name: personographer-profile-builder
description: Build editorial profiles of notable individuals for Personographer (personographer.com). Use this skill whenever the user mentions building, creating, researching, or generating a profile for a person, biographical research on a public figure, a Personographer CMS item, or asks to "make a profile for [name]" — even when they don't explicitly say "Personographer". Three-stage workflow: (1) Researcher gathers verified facts from Tier 1/2 sources with citations, (2) Editor formulates narrative blocks per Personographer editorial rules (FT/Bloomberg tone, no hype), maps data to the Webflow Profile CMS Collection, and generates Schema.org JSON-LD, (3) Validator audits and repairs the JSON-LD against Schema.org before a draft CMS item is created via the Webflow MCP. Always operate in draft mode and surface the research output for user review before mapping.
---

# Personographer Profile Builder

A three-stage editorial pipeline that turns a person's name into a verified, structured, publication-ready profile in the Personographer Webflow CMS.

## When to trigger this skill

Trigger whenever the user:
- Provides a person's name and asks for a profile, biography, or background research with editorial framing
- Says things like "build a profile for [name]", "research [name]", "add [name] to Personographer", "create a CMS item for [name]"
- References Personographer, profile pages, or notable-individual editorial content
- Asks to generate Schema.org markup for a person profile

If the request is just a casual factual question about a person ("when was X born?") — do NOT trigger this skill. This skill is for full profile construction, not lookups.

## Workflow

This is a **three-prompt pipeline with a mandatory human review checkpoint after Stage 1**. Stages 2 and 3 run together to produce the validated draft.

### Stage 1 — Researcher

Read `prompts/01-researcher.md` and follow it precisely. The Researcher:
- Takes only a name as input (asks for disambiguation if multiple notable people share the name)
- Gathers verified facts from a strict source whitelist (see `references/source-whitelist.md`)
- Returns a structured JSON of raw facts with citations for every claim
- Never formulates editorial copy — that's Stage 2's job
- Never invents — missing data stays `null`

After Stage 1, **stop and show the JSON to the user**. Do not proceed to Stage 2 until the user confirms the facts look right (or asks for corrections / additional research).

At this checkpoint, also ask the user two media questions (their answers feed Stage 2):
- **Videos?** — if yes, the user pastes YouTube link(s). The skill looks up any metadata it needs from the links.
- **Photos?** — if yes, the user gives the Webflow asset **name or URL** for the main portrait and/or gallery photos (the user uploads them to the Asset Manager; Webflow does not expose asset IDs). If no, the photo/video fields and their Schema nodes stay empty.

### Stage 2 — Editor / Mapper

Read `prompts/02-mapper.md` and follow it precisely. The Editor:
- Takes the Researcher's JSON as input
- Generates narrative blocks (Professional Identity, Profile Overview, Expertise Summary, Reputation Summary, Economic Impact, Editorial Comment, Q&A) per the editorial rules in `prompts/editorial-rules.md`
- Maps everything to the Webflow Profile collection using exact field slugs from `references/webflow-fields.json` and the RichText structure in `references/section-layouts.md`
- Generates a draft Schema.org JSON-LD per the template in `references/schema-template.json`
- Auto-creates missing Profile Categories and Country Flags items

Photos and videos are provided by the user at the review checkpoint, not researched — see Stage 1. Stage 2 includes them only if provided.

### Stage 3 — Schema Validator / QA

Read `prompts/03-validator.md` and follow it precisely. The Validator takes the draft JSON-LD from Stage 2 and the Researcher facts, then audits and repairs the schema against the Schema.org vocabulary so it is fully valid and high quality (no invalid type/property combinations, no orphan nodes, no fabricated values). The **validated** JSON-LD is what gets written (wrapped in `<script type="application/ld+json">`) to the `schema-json-ld-3` field.

Only after the schema is validated does the skill create the Profile CMS item as a draft (`isDraft: true`) via the Webflow MCP.

After Stage 3, present the user with:
- A summary of generated narrative blocks
- The Webflow item URL (draft mode) for review
- The schema validation report (what was fixed/removed, any residual warnings)
- Any validation warnings or missing-data notes
- Any reference items the skill auto-created (Profile Categories, Country Flags) — and a reminder that each new Country Flag still needs its flag-icon uploaded manually

## Files in this skill

```
SKILL.md                            # this file
prompts/
  01-researcher.md                  # Stage 1 prompt (Researcher)
  02-mapper.md                      # Stage 2 prompt (Editor/Mapper)
  03-validator.md                   # Stage 3 prompt (Schema Validator / QA)
  editorial-rules.md                # Personographer editorial principles (FT/Bloomberg tone, no-hype list, Professional Identity rules)
references/
  webflow-fields.json               # Exact Webflow collection IDs, field slugs, types, and option IDs
  schema-template.json              # Reference Schema.org JSON-LD structure for a Person profile
  schema-rules.md                   # Authoritative Schema.org checklist (type→property constraints, required fields, gotchas)
  source-whitelist.md               # Tier 1 / Tier 2 / authoritative profile sources
  section-layouts.md                # Per-section RichText layout contract for the live-site transform script
examples/                           # Reference example outputs (populated during testing)
```

## Webflow context (constants)

```
Site ID:                       69bbea57917a7a73aece58a9
Profile collection ID:         69d69c16945c89eef261f630
Profile Categories ID:         69d69c20f6cdce775944238b
Country Flags ID:              69e24f513ec23d0170f8f18e
```

Full field schemas and option IDs are in `references/webflow-fields.json`.

## Critical rules (enforced in both stages)

1. **No hype language**: Never use words like *visionary, renowned, iconic, legendary, world-class, leading, influential, top, famous, successful, award-winning* unless they appear inside a formal award title. Full list in `prompts/editorial-rules.md`.

2. **Source discipline**: Researcher only uses sources from `references/source-whitelist.md`. No blogs, no Medium, no tabloids, no AI-generated wikis, no data aggregators without cross-check.

3. **Never invent**: If a fact has no source, the field stays empty. Don't fill placeholders like "TBD" or "N/A".

4. **Always draft mode**: Every CMS item created during the dev phase has `isDraft: true`. The user reviews and publishes manually.

5. **Mandatory human review between stages**: Never run Stage 2 automatically after Stage 1. Show the Researcher output, wait for user confirmation.

6. **Quotes are verbatim**: Selected Quotes must be exact text from Tier 1/2 written sources only — no paraphrasing, no video/podcast/social media quotes.

7. **Photos & videos**: Not researched in Stage 1. At the review checkpoint the skill asks the user; if the user provides YouTube links (videos) or Webflow asset names/URLs (photos), Stage 2 includes them — videos via the `videos` field + `VideoObject` Schema, photos via `profile-photo`/`photos` fields (asset resolved by name/URL) + Schema `image`. If the user provides nothing for one, that field and its Schema node(s) stay empty.

## Communication language

The user (Roman) communicates in Ukrainian. The skill content (this file, prompts, references) is in English for triggering precision, but Claude responds to the user in Ukrainian throughout the workflow.

## Future production mode

In the current dev phase, the skill stops at "draft created, ready for human review". In production, the workflow may be wrapped into an API agent that runs end-to-end without human checkpoints. The prompts are written to support both modes; the human-review gates are enforced by this SKILL.md, not by the prompts themselves.
