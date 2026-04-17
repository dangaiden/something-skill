---
name: review-pending
description: Review pending items from doc/PENDING_REVIEW.md — check which are still actionable, which can be resolved, and prioritize. Use when the user asks about pending reviews, what needs checking, or what's left to do.
argument-hint: "[item number or 'all']"
allowed-tools: Bash(python*) Bash(sqlite3*) Bash(ls*) Bash(wc*) Read Grep Glob
---

# Review Pending Items

Read `doc/PENDING_REVIEW.md` and audit which items are still actionable.

## For each non-resolved item, check:

1. **Data freshness**: Query `backend/tracker.db` to see if relevant data exists
   - For "needs first scrape" items: `SELECT COUNT(*) FROM <table> WHERE collection_date >= '<item_date>'`
   - For singles: check `singles_price_entries` count for the expansion
   - For products: check `price_entries` count
   - For eBay items: check `ebay_price_entries`

2. **File existence**: Check if referenced JSON files, scripts, or config entries exist
   - Use Glob/ls to verify paths mentioned in the item

3. **Staleness**: Items older than 2 weeks with no activity are likely stale — flag them

4. **Dependencies**: Some items depend on others (e.g., "run after graded scraper fix") — note if the dependency is resolved

## Output format

For each item, report:
- **Status**: Actionable / Likely resolved / Stale / Blocked
- **Evidence**: What you checked and found (DB counts, file existence, etc.)
- **Action needed**: What the user should do next (if anything)

If `$ARGUMENTS` is a specific item number, focus on just that item.
If `$ARGUMENTS` is empty or "all", review everything.

Sort by priority: Actionable items first, then stale, then likely resolved.

At the end, suggest which items can be moved to the Resolved table.
