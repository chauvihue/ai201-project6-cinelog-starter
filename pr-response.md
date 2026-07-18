# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**Reviewer's comment:** `save_to_watchlist()` doesn't follow the project's `verb_to_noun` naming convention. Should be renamed to `add_to_watchlist()` to match `add_to_collection()`.

**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` (line 12). Updated the import statement in `routes/watchlist/watchlist.py` (line 8) and the function call in the `add_film()` endpoint (line 32).

**How I verified:**
1. Searched entire codebase for remaining references to `save_to_watchlist` using `grep -r "save_to_watchlist"`, found 0 code references (only appears in this PR response doc)
2. Verified all references to `add_to_watchlist` are correct in `routes/watchlist/watchlist.py` (import and usage)

## Comment 2 — Deduplication
**Reviewer's comment:** The function doesn't handle duplicate entries. If a user adds the same film twice to their watchlist, it creates duplicate database records instead of raising an error.

**What I did:** Added deduplication logic in `add_to_watchlist()` in `services/watchlist_service.py` (lines 20-25). Created a new `AlreadyInWatchlistError` exception class (lines 12-14). The function now queries for existing entries before creating a new one and raises the exception if a duplicate is found.

**How I verified:**
1. Verified that attempting to add a duplicate film raises `AlreadyInWatchlistError` exception
2. Confirmed database contains only 1 entry after duplicate attempt (verified with `WatchlistEntry.query.filter_by().count()`)

## Comment 3 — Missing test
**Reviewer's comment:** Need to add a test case for when `film_id` doesn't exist in the database. The pattern is in `test_collection.py`.

**What I did:** added `test_add_to_watchlist_nonexistent_film_raises()` in `tests/test_watchlist.py` to test if error raised when add nonexistent film
**How I verified:** I ran the test and it passed!

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