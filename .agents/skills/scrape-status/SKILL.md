---
name: scrape-status
description: Check today's scrape status across all pipelines — what's been scraped, what's in the DB, what's stranded in JSON files, what's been pushed to VPS. Use when the user asks about scrape progress, missing data, or push status.
argument-hint: "[date YYYY-MM-DD, default today]"
allowed-tools: Bash(python*) Bash(sqlite3*) Bash(ls*) Bash(wc*) Bash(curl*) Read Grep Glob
---

# Scrape Status Diagnostic

Check the scrape state for date `$ARGUMENTS` (default: today).

## 1. JSON files on disk

Check `JSON_data/` for the target date. For each pipeline:

| Pipeline | Merged file pattern | Shard file pattern |
|----------|--------------------|--------------------|
| CM Products | `collected_data_{date}.json` | `collected_data_{date}_*.json` |
| CM Singles | `collected_singles_{date}.json` | `collected_singles_{date}_*.json` |
| CM Graded | `collected_singles_graded_{date}.json` | `collected_singles_graded_{date}_*.json` |
| CM Boosters | `collected_boosters_{date}.json` | — |
| eBay Topsun | `ebay_topsun_{date}.json` | `ebay_topsun_{date}_*.json` |
| eBay Carddass | `ebay_carddass_{date}.json` | `ebay_carddass_{date}_*.json` |
| eBay Products | `ebay_products_{date}.json` | — |

For each, report:
- Does the merged file exist? How many records?
- Are there shard files? How many, and do they all have `.done` markers?
- **Stranded data**: shard files exist but merged file is missing/stale (fewer records than shards combined) — this means merge didn't run

Use: `python -c "import json; d=json.load(open('JSON_data/<file>')); print(len(d) if isinstance(d,list) else sum(len(v) for v in d.values() if isinstance(v,list)))"` to count records.

## 2. Database counts

Query `backend/tracker.db` for the target date:

```sql
SELECT 'price_entries' as t, COUNT(*) FROM price_entries WHERE collection_date = '{date}'
UNION ALL SELECT 'singles_price_entries', COUNT(*) FROM singles_price_entries WHERE collection_date = '{date}'
UNION ALL SELECT 'ebay_price_entries', COUNT(*) FROM ebay_price_entries WHERE collection_date = '{date}'
UNION ALL SELECT 'ebay_carddass_price_entries', COUNT(*) FROM ebay_carddass_price_entries WHERE collection_date = '{date}'
UNION ALL SELECT 'ebay_product_price_entries', COUNT(*) FROM ebay_product_price_entries WHERE collection_date = '{date}';
```

Also check `booster_price_entries` if it exists.

## 3. Expected vs actual

Compare DB counts against config sizes:
- Products: `from products_config import PRODUCTS` — count per language
- Singles: `from singles_config import SINGLES` — total count
- Check coverage percentage

## 4. VPS sync status

Check if data has been pushed to VPS by looking at the `scrape_runs` table:
```sql
SELECT * FROM scrape_runs WHERE run_date = '{date}' ORDER BY scraped_at DESC;
```

If the scrape_runs table doesn't exist locally, note that push status can only be checked on VPS.

## Output format

```
Scrape Status for {date}
========================

CM Products:   {json_count} scraped → {db_count} in DB  ({pct}% of {expected})
CM Singles:    {json_count} scraped → {db_count} in DB  ({pct}% of {expected})
CM Graded:     ...
CM Boosters:   ...
eBay Topsun:   ...
eBay Carddass: ...
eBay Products: ...

Issues:
  - [STRANDED] Singles: 3 shard files (3362 records) not merged — run: python daily/merge_collected_singles.py {date}
  - [PARTIAL] Graded: shard 3 missing .done marker — was interrupted mid-scrape
  - [NOT PUSHED] 1900 new records not on VPS — run: python deploy/push_to_vps.py --date {date}
```

Focus on actionable issues. If everything looks clean, say so briefly.
