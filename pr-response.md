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

**What I did:** added `test_add_to_watchlist_nonexistent_film_raises()` in `tests/test_watchlist.py` to test if error raised when add nonexistent film. The test is modeled after existing test fixtures and structure found in `tests/test_collection.py` and `tests/test_watchlist.py`
**How I verified:** I ran the test and it passed: nonexistent film ID will raise `FilmNotFoundError`

## Comment 4 — Default visibility
**Reviewer's comment:** Watchlists default to `public=True`. Need to document the reasoning for this decision — we should be intentional about default visibility, not just inherit a default.

**My position:** I set the default to `public=False` (models.py:80) and advocate keeping it that way.

**Reasoning:** I'm optimizing for the **privacy-conscious majority use case**. A watchlist is fundamentally a personal bookmark system — it captures what a user *intends* to watch, not what they've already seen and are willing to endorse. This includes:
- Films they're curious about but haven't researched yet
- Guilty pleasures they're exploring privately
- Work-related content (documentaries, training films) that reveals professional interests
- Recommendations they're vetting before sharing with others

Defaulting to public would expose this exploratory, unfiltered list without explicit consent. Most users expect bookmarks and "save for later" features to be private by default (see: browser bookmarks, YouTube's "Watch Later", reading lists). A user who wants to share their watchlist can easily toggle it public; a user who didn't realize their private interests were public has already suffered the privacy breach.

**Tradeoff acknowledged:** The `public=False` default creates friction for **curators and influencers** who intentionally build watchlists as public recommendations for their followers. These users want discovery, not privacy — they're creating content, not consuming it. For them, an extra toggle click on every watchlist is annoying overhead. However, this use case is the **minority**: most CineLog users are film enthusiasts tracking their own viewing, not content creators building public playlists. The safer, privacy-respecting default protects the majority at the cost of a minor inconvenience for power users who can (a) quickly learn to toggle the setting, or (b) benefit from a future "default to public" account-level preference if this cohort grows.

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