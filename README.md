# Personographer Profile Builder

A Claude skill that turns a person's name into a verified, structured, publication-ready editorial profile in the [Personographer](https://personographer.com) Webflow CMS.

## What it does

Two-stage editorial pipeline:

1. **Researcher** — gathers verified facts from whitelisted sources (Tier 1/2 publications, authoritative profiles, registries), with full citations for every claim.
2. **Editor / Mapper** — formulates narrative blocks per Personographer's editorial rules (FT/Bloomberg tone, no hype), maps everything to the Webflow Profile CMS Collection, generates Schema.org JSON-LD, and creates a draft CMS item via the Webflow MCP.

Output is always a draft requiring human review before publishing.

## How to use

Trigger the skill in Claude by asking for a profile:

```
Build a Personographer profile for Richard Branson
```

```
Створи профіль для Bernard Arnault на Personographer
```

The skill will:
1. Run the Researcher → return a JSON of verified facts with sources
2. Wait for your confirmation
3. Run the Editor → generate narrative copy, map to CMS fields, generate Schema.org JSON-LD, create a draft Webflow item

## Repository structure

```
.
├── SKILL.md                         # Main skill manifest — Claude reads this first
├── README.md                        # This file
├── .gitignore
├── prompts/
│   ├── 01-researcher.md             # Stage 1 — fact gathering
│   ├── 02-mapper.md                 # Stage 2 — editorial + CMS + Schema
│   └── editorial-rules.md           # Personographer voice rules (referenced by both)
├── references/
│   ├── webflow-fields.json          # Exact Profile collection field IDs, slugs, types, options
│   ├── schema-template.json         # Reference Schema.org JSON-LD structure
│   └── source-whitelist.md          # Allowed Tier 1/2 sources and forbidden ones
└── examples/                        # Reference outputs (added during testing)
```

## Installation

### Option A — Claude.ai (web/desktop app)

1. Zip the project folder (or download a release `.zip`).
2. In Claude.ai, go to Settings → Capabilities → Skills.
3. Upload the zip.
4. Activate the skill.
5. Ensure the Webflow MCP is connected to your Claude account.

### Option B — Claude Code (CLI)

Clone this repo to your skills directory:

```bash
git clone git@github.com:<your-username>/personographer-profile-builder.git ~/.claude/skills/personographer-profile-builder
```

## Webflow setup

The skill is hard-coded for the Personographer Webflow site. Key IDs:

| Item | ID |
|---|---|
| Site | `69bbea57917a7a73aece58a9` |
| Profile collection | `69d69c16945c89eef261f630` |
| Profile Categories | `69d69c20f6cdce775944238b` |
| Country Flags | `69e24f513ec23d0170f8f18e` |

If the site ID changes, update `SKILL.md` and `references/webflow-fields.json`.

## Editorial principles

The skill enforces Personographer's editorial voice strictly:

- **No hype** — words like *visionary, renowned, iconic, world-class, leading* are forbidden in output.
- **Source discipline** — only Tier 1/2 publications and authoritative profiles are used. No blogs, tabloids, AI-generated wikis.
- **Never invent** — missing facts leave fields empty, not filled with guesses.
- **Always draft** — every item is created with `isDraft: true`. Human review before publishing.
- **Photo rights** — only Wikipedia Commons, official corporate About pages, or academic bios. No Reuters/Getty/AP.

Full rules in `prompts/editorial-rules.md`.

## Development workflow

This is a private skill in dev phase. Iteration cycle:

1. Run the skill on a test subject
2. Review the Researcher output → note issues
3. Review the Mapper output → note issues
4. Inspect the created Webflow draft → note rendering issues
5. Update prompts / references / SKILL.md
6. Commit and push
7. Repeat

When the skill is stable, the next phase is wrapping it into an API agent for automated, end-to-end profile generation without human checkpoints between stages.

## Notes

- Communication with the operator is in Ukrainian; the skill content is in English for triggering precision.
- The collection name "Contry Flags" contains a typo in Webflow — preserved for now until the client decides whether to fix it (would invalidate existing references).
