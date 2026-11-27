# multi-agent-scraper-template

A three-level Claude Code template for collecting structured records from public web sources.

## Design

| Level | Agent | Model | Role |
|---|---|---|---|
| 1 | Manager (`CLAUDE.md`) | Opus | Dispatches, resolves escalations, merges and dedupes, writes final output, summarizes |
| 2 | `source-manager` (up to 6, one per source group) | Sonnet | Spawns scrapers, verifies and filters their results |
| 3 | `scraper` (up to 4 per manager) | Haiku | Fetches one page or query and returns structured records |

## Workflow

1. The manager reads `config/task.md` (what to collect, sources, filters, output format) and any list of already-collected items.
2. The manager dispatches one source-manager per source group.
3. Each source-manager sends scrapers to its pages or queries. Scrapers return only records they actually saw.
4. Each source-manager verifies every record (real source URL, matches the filters, not a duplicate) and returns a clean table.
5. The manager merges, dedupes across sources, and writes the output.
6. The manager reports counts, highlights, and any blocked searches.

## Bot-check ladder

1. A scraper that hits a CAPTCHA, bot check, or login wall stops and returns `BLOCKED`.
2. Its source-manager tries **one** legitimate alternative route. If that fails, it returns `ESCALATE`.
3. The manager makes one final attempt by a different route, or abandons the search.

Never solve or bypass CAPTCHAs. Blocked and abandoned searches are listed in the final report.

## Usage

1. Copy `config/task.example.md` to `config/task.md` and fill in the placeholders. `config/task.md` is gitignored.
2. Start Claude Code in the repo root with the manager on Opus: `claude --model opus`. `CLAUDE.md` cannot set the model itself.
3. Ask it to run the task. Set the output destination under `output/` so results stay out of git.

Requires Claude Code 2.1.172 or later, because source-managers spawn their own subagents.

The limits on agent counts, retries and nesting are instructions to the models, not enforced settings.

## Responsible use

- Respect each site's terms of service and `robots.txt`.
- Use official APIs where they are available.
- Do not commit personal data, IDs, emails, or credentials.
