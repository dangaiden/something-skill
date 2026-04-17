# Pokemon Tracker Skills for Claude Code

Custom [Claude Code](https://claude.ai/code) skills for the [Pokemon Price Tracker](https://github.com/YOUR_USERNAME/pokemon-tracker) project — a multi-source price tracking pipeline for Pokemon TCG cards.

## Skills

### `/review-pending`

Audits the project's `doc/PENDING_REVIEW.md` backlog and checks which items are still actionable.

For each pending item it:
- Queries the database to check if referenced data now exists
- Verifies file/script existence for items that depend on them
- Flags stale items (older than 2 weeks with no activity)
- Identifies dependency chains between items

**Usage:**
```
/review-pending           # review all pending items
/review-pending 18        # review a specific item by number
```

### `/scrape-status`

Diagnostic that checks the scrape state across all pipelines for a given date.

It checks:
- **JSON files on disk** — merged files, shard files, `.done` markers
- **Database record counts** — per table, compared against expected config sizes
- **Stranded data** — shard files that were never merged (e.g., interrupted overnight runs)
- **VPS sync status** — whether data has been pushed to the remote server

**Usage:**
```
/scrape-status              # check today
/scrape-status 2026-04-16   # check a specific date
```

## Installation

### With `npx` (one-liner)

```bash
npx @anthropic-ai/claude-code skills add /review-pending https://github.com/YOUR_USERNAME/something-skill
npx @anthropic-ai/claude-code skills add /scrape-status https://github.com/YOUR_USERNAME/something-skill
```

### Manual

Copy the skill directories into your project's `.claude/skills/`:

```bash
# From your project root
mkdir -p .claude/skills
cp -r review-pending .claude/skills/
cp -r scrape-status .claude/skills/
```

## Context

These skills are designed for a specific project structure:

- `doc/PENDING_REVIEW.md` — backlog of items needing manual review
- `backend/tracker.db` — SQLite database with price entries
- `JSON_data/` — scraped data files (merged + shard files)
- `backend/*_config.py` — product/singles/expansion configurations
- `deploy/push_to_vps.py` — VPS sync script

Adapt the table names, file patterns, and paths in the `SKILL.md` files to match your own project if forking for a different scraping pipeline.

## License

MIT
