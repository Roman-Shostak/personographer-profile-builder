# Personographer Profile Builder

A Claude skill that turns a person's name into a verified, structured, publication-ready editorial profile in the [Personographer](https://personographer.com) Webflow CMS.

Personographer is an editorial platform publishing institutional profiles of significant individuals in an FT / Bloomberg register. This skill automates the research → editorial → CMS pipeline for a single profile while keeping a human in the loop.

## What it does

A **three-stage pipeline with a mandatory human-review checkpoint after Stage 1**:

1. **Researcher** (`prompts/01-researcher.md`) — gathers verified facts from a strict source whitelist (Tier 1/2 publications, authoritative profiles, registries) and returns a structured JSON of raw facts, with full source citations for every claim. It never writes editorial copy and never invents — missing facts stay `null`.
2. **Editor / Mapper** (`prompts/02-mapper.md`) — takes the Researcher JSON and:
   - formulates narrative blocks (Professional Identity, Profile Overview, Expertise/Reputation/Economic summaries, Editorial Comment, Q&A) per the editorial rules in `prompts/editorial-rules.md`;
   - maps everything to the Webflow Profile CMS collection using exact field slugs (`references/webflow-fields.json`) and RichText structures (`references/section-layouts.md`);
   - generates a draft Schema.org JSON-LD (`references/schema-template.json`);
   - auto-creates any missing Profile Categories and Country Flags reference items.
3. **Validator** (`prompts/03-validator.md`) — audits and repairs the JSON-LD against the Schema.org vocabulary (valid type/property combinations, valid value targets, no orphan nodes, no fabricated values), using the Researcher facts as the source of truth. The validated JSON-LD — wrapped in a `<script type="application/ld+json">` tag — is what gets written to the CMS, after which the Profile is created as a **draft** item via the Webflow MCP.

Output is always a draft (`isDraft: true`) requiring human review before publishing.

## How to use

Trigger the skill in Claude by asking for a profile:

```
Build a Personographer profile for Richard Branson
```

```
Створи профіль для Bernard Arnault на Personographer
```

Flow:

1. **Researcher runs** → returns a JSON of verified facts with sources.
2. **Review checkpoint** → the skill shows you the facts and waits for your confirmation. It also asks two media questions:
   - **Videos?** — paste YouTube link(s) if any (the skill derives the rest).
   - **Photos?** — give the Webflow asset **name or URL** for the main portrait and/or gallery photos (you upload them to the Asset Manager yourself; Webflow does not expose asset IDs).
3. **Mapper + Validator run** → the Mapper generates narrative copy, maps to CMS fields, auto-creates missing references, and drafts the Schema.org JSON-LD; the Validator audits/repairs the JSON-LD against Schema.org. The validated schema is written and a draft Webflow item is created.
4. **You review** the draft in Webflow (and the schema validation report) and publish manually.

## Repository structure

```
.
├── SKILL.md                         # Main skill manifest — Claude reads this first
├── README.md                        # This file
├── CLAUDE.md                        # Working context for Claude Code (in Ukrainian)
├── .gitignore
├── prompts/
│   ├── 01-researcher.md             # Stage 1 — fact gathering (22 categories, citations)
│   ├── 02-mapper.md                 # Stage 2 — editorial + CMS mapping + draft Schema + MCP
│   ├── 03-validator.md              # Stage 3 — Schema.org QA / repair
│   └── editorial-rules.md           # Personographer voice rules (referenced by both stages)
├── references/
│   ├── webflow-fields.json          # Exact Profile collection field IDs, slugs, types, option IDs
│   ├── schema-template.json         # Reference Schema.org JSON-LD @graph structure
│   ├── section-layouts.md           # Per-section RichText layout contract for the live-site script
│   └── source-whitelist.md          # Allowed Tier 1/2 sources and forbidden ones
└── examples/                        # Reference I/O outputs — few-shot anchors (added during testing)
```

## Installation

### Option A — Claude.ai (web/desktop app)

1. Zip the project folder (or download a release `.zip`).
2. In Claude.ai, go to **Settings → Capabilities → Skills**.
3. Upload the zip and activate the skill.
4. Ensure the **Webflow MCP** is connected to your Claude account.

### Option B — Claude Code (CLI)

```bash
git clone git@github.com:Roman-Shostak/personographer-profile-builder.git ~/.claude/skills/personographer-profile-builder
```

## Webflow setup

The skill is hard-coded for the Personographer Webflow site. Key IDs:

| Item | ID |
|---|---|
| Site | `69bbea57917a7a73aece58a9` |
| Profile collection | `69d69c16945c89eef261f630` |
| Profile Categories | `69d69c20f6cdce775944238b` |
| Country Flags | `69e24f513ec23d0170f8f18e` |

Full field schemas and option IDs are in `references/webflow-fields.json`. If the site ID changes, update `SKILL.md`, `CLAUDE.md`, and `references/webflow-fields.json`.

Two things must be set up on the Webflow side for output to render correctly:

1. **JSON-LD embed** — the profile template page must render the `schema-json-ld-3` field into `<head>` (the field already contains the full `<script type="application/ld+json">…</script>`).
2. **RichText transform script** — the published site runs a custom-code JavaScript that restructures RichText fields based on a `data-cms-layout` attribute and semantic markers in the HTML (see "RichText formatting contract" below). The Mapper writes HTML to match this script.

## Editorial principles

The skill enforces Personographer's editorial voice strictly (full rules in `prompts/editorial-rules.md`):

- **No hype** — words like *visionary, renowned, iconic, world-class, leading, influential, top, famous, successful, award-winning* are forbidden in output (unless inside a formal award title).
- **Source discipline** — only Tier 1/2 publications and authoritative profiles. No blogs, Medium, tabloids, AI-generated wikis, or unverified data aggregators.
- **Never invent** — missing facts leave fields empty, never filled with guesses or "TBD".
- **Always draft** — every item is created with `isDraft: true`. Human review before publishing.
- **Verbatim quotes only** — Selected Quotes are exact text from Tier 1/2 written sources; no paraphrasing, no video/podcast/social-media quotes.

## RichText formatting contract

Most multi-entry data (career, awards, publications, etc.) is stored in **RichText** fields, and the live site transforms them with a custom script. The Mapper must follow these rules (defined in `prompts/02-mapper.md` B.0 and `references/section-layouts.md`):

- **Plain `<p>` is the default** for all body text.
- **Bold is a structural marker, not emphasis** — a `<p>` that is entirely `<strong>…</strong>` is read as a label (role, period, key). Never inline-bold inside running text.
- **`<ul>`/`<li>` lists only in**: Profile Overview, Publications & Talks (under *Notable Mentions* only), and Personal Interests. Everywhere else uses plain `<p>`.
- **Schema JSON-LD** goes into `schema-json-ld-3` already wrapped in `<script type="application/ld+json">…</script>`.

`references/section-layouts.md` documents the exact per-section structure and tracks which sections are CONFIRMED on the live site vs. PENDING verification. Career & Roles is confirmed; the rest are being formalized one section at a time.

## Media handling

Photos and videos are **not researched** — they are supplied by the operator at the review checkpoint and included only if provided:

- **Videos** — operator pastes YouTube link(s). The Mapper writes one `<a href>` per video into the `videos` field (the site script builds the carousel and fetches titles) and generates a Schema `VideoObject` per link.
- **Photos** — operator uploads to the Webflow Asset Manager and gives the asset **name or URL**. The Mapper resolves it to an asset id via the MCP and binds the portrait to `profile-photo` and gallery images to `photos`, plus a Schema `image` node. No photo → those fields and the Schema `image` are omitted.

## Auto-created references

When a needed multi-reference target is missing, the Mapper auto-creates it via the MCP (`isDraft: true`) and reports it in the final output:

- **Profile Categories** — created with `name` + `slug`.
- **Country Flags** — created with `name` + `slug`; the `flag-icon` is left empty for the operator to upload manually (each created flag is flagged in the summary).

## Development workflow

This is a private skill in its dev phase. Iteration cycle:

1. Run the skill on a test subject in Claude.ai.
2. Review the Researcher output → note issues.
3. Review the Mapper output and the created Webflow draft → note formatting / mapping / rendering issues.
4. Update prompts / references / `SKILL.md`.
5. Commit, push, repackage the zip, re-upload to Claude.
6. Repeat in a fresh chat until stable.

When the skill is stable, the next phase is wrapping it into an API agent for automated, end-to-end profile generation (including photo upload via `asset_tool > upload_image_by_url`) without human checkpoints between stages.

## Notes

- Communication with the operator is in Ukrainian; the skill content is in English for triggering precision.
- The Country Flags collection is named "Contry Flags" (typo) in Webflow — preserved as-is until the client decides whether to fix it (renaming would invalidate existing references).
