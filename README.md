# HasData Agent Skills

A collection of [agent skills](https://agentskills.io/) for working with [HasData](https://hasdata.com) — a cloud platform for extracting public web data via SERP APIs, pre-parsed Scraper APIs (Amazon, Zillow, Maps, Indeed, Instagram, …), and async Scraper Jobs (recursive crawling, contact enrichment, SEC filings).

These skills teach AI coding agents — Claude Code, Cursor, Codex, Gemini CLI, and any tool that loads `SKILL.md` files — how to call HasData from real code without guessing endpoints, parameter shapes, or response schemas.

## Skills

| Skill | What it does |
|---|---|
| [`hasdata`](skills/hasdata) | Wire HasData APIs directly into your code (Python, TypeScript, Go). Covers `POST /scrape/web`, every Scraper API (Google SERP / AI Mode / Maps / Amazon / Zillow / Redfin / Airbnb / Yelp / YellowPages / Indeed / Glassdoor / Instagram / Shopify / Trends / Flights / Bing), and the async Scraper Job lifecycle (submit → poll → results, webhooks). Schemas verified against the live API. |
| [`hasdata-cli`](skills/hasdata-cli) | Use the [`hasdata` CLI](https://github.com/HasData/hasdata-cli) for real-time data lookups from the terminal — same APIs, but driven by `hasdata <subcommand> --flag value` and piped through `jq`. Best for one-off queries, shell pipelines, and CI jobs. |

Pick `hasdata` when you're building an application or service that calls the API. Pick `hasdata-cli` when you're scripting in bash or want quick interactive lookups.

## Install

The fastest way (works with any agent harness that supports the [Agent Skills](https://agentskills.io/) format):

```
npx skills add hasdata/agent-skills
```

### Claude Code

```
/plugin marketplace add https://github.com/HasData/agent-skills
/plugin install hasdata@hasdata-agent-skills
/plugin install hasdata-cli@hasdata-agent-skills
```

### Cursor / Windsurf / VS Code with agent extensions

Clone the repo into your project (or globally) and point your agent at the `skills/` directory:

```bash
git clone https://github.com/HasData/agent-skills ~/.agent-skills/hasdata
```

Most agent harnesses auto-discover any `SKILL.md` file under a configured skills root.

### Codex / Gemini CLI / generic markdown-context tools

Reference the individual skill files directly:

- `skills/hasdata/SKILL.md`
- `skills/hasdata-cli/SKILL.md`

Each `SKILL.md` is self-contained with progressive references under `skills/<name>/references/`.

## Prerequisites

1. A HasData account — sign up at [app.hasdata.com](https://app.hasdata.com).
2. An API key (Dashboard → API). Store it in `HASDATA_API_KEY`:
   ```bash
   export HASDATA_API_KEY="..."
   ```
3. For the `hasdata-cli` skill: the [HasData CLI](https://github.com/HasData/hasdata-cli) installed and configured (`hasdata configure`).

## Skill structure

```
skills/
├── hasdata/
│   ├── SKILL.md              # entry point — orientation, decision rules, gotchas
│   └── references/           # progressive disclosure — agent loads on demand
│       ├── web-scraping.md   #   POST /scrape/web parameters, JS scenarios, AI extraction
│       ├── search.md         #   Google SERP / AI Mode / News / Bing / Trends
│       ├── ecommerce.md      #   Amazon, Shopify
│       ├── real-estate.md    #   Zillow, Redfin, Airbnb
│       ├── local-business.md #   Maps, Yelp, YellowPages
│       ├── jobs.md           #   Indeed, Glassdoor
│       ├── scraper-jobs.md   #   async submit/poll/results, webhooks, crawler, contacts, SEC
│       └── code-recipes.md   #   Python / TypeScript clients, retry, backoff, polling
└── hasdata-cli/
    ├── SKILL.md
    └── references/
```

## Resources

- [HasData docs](https://docs.hasdata.com/)
- [HasData CLI](https://github.com/HasData/hasdata-cli)
- [Dashboard](https://app.hasdata.com)
