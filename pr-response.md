# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**Reviewer's comment:** `save_to_watchlist()` doesn't follow the project's `verb_to_noun` naming convention. Should be renamed to `add_to_watchlist()` to match `add_to_collection()`.

**What I did:** changed the name of the function as described; references found in `routes/watchlist/watchlist.py` in the `add_film()` function
**How I verified:** I ran tests

## Comment 2 — Deduplication
**Reviewer's comment:** The function doesn't handle duplicate entries. If a user adds the same film twice to their watchlist, it creates duplicate database records instead of raising an error.

**What I did:** added a depulication logic in `add_to_watchlist()` in `services/watchlist_service.py` by a test query of the WatchlistEntry database.
**How I verified:**

## Comment 3 — Missing test
**Reviewer's comment:** Need to add a test case for when `film_id` doesn't exist in the database. The pattern is in `test_collection.py`.

**What I did:**
**How I verified:**

## Comment 4 — Default visibility
**Reviewer's comment:** Watchlists default to `public=True`. Need to document the reasoning for this decision — we should be intentional about default visibility, not just inherit a default.

**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**Reviewer's comment:** Reviewer prefers "date added" order (newest first) rather than alphabetical. Most users want to see recent additions. Open to discussion but wants a documented decision.

**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**Reviewer's comment:** A refactor on main changed film IDs from integers to UUIDs. The watchlist code still references integer IDs. Need to rebase on main and update accordingly.

**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->