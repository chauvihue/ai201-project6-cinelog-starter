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

**Mitigation strategy:** A **"followers only"** visibility option could help bridge this gap — allowing watchlists to be discoverable within a user's social network without broadcasting to the entire platform. This middle ground protects against the "public internet" exposure concern (strangers seeing your unvetted interests) while still enabling social discovery among trusted connections. Users comfortable sharing with friends but not the world get a safer sharing option, and the platform gains more visible content than pure private-by-default would provide. However, I acknowledge this approach **still constrains platform growth**: it limits viral discovery (strangers can't stumble upon great watchlists), reduces content available to new users with no followers yet, and maintains friction for creators who want maximum reach. If CineLog's core value proposition is broad public discovery rather than friend-network sharing, even "followers only" undermines that goal. The privacy-first stance I'm taking here implicitly prioritizes individual user protection over platform network effects, which may not align with CineLog's product strategy if it's positioning itself as a social discovery platform rather than a personal tracking tool.

## Comment 5 — Sort order
**Reviewer's comment:** Reviewer prefers "date added" order (newest first) rather than alphabetical. Most users want to see recent additions. Open to discussion but wants a documented decision.

**My position:** I agree with the reviewer — default sort should be **date added, newest first**.

**Reasoning:** I'm optimizing for the **primary watchlist use case: browsing for what to watch next**. When users open their watchlist, they're typically deciding "what should I watch tonight?" rather than searching for a specific film they remember saving. Recent additions have the highest intent to watch — they were added because of a fresh recommendation, trending conversation, or recent interest, making them more contextually relevant and top-of-mind. A film added yesterday after a friend's recommendation is far more likely to be watched than one added three months ago that has since been forgotten. Newest-first surfaces these high-intent items at the top, increasing the likelihood that users actually engage with their watchlist instead of feeling overwhelmed by stale entries they've lost interest in.

This also matches user mental models from similar features: email inboxes default to newest first, browser history shows recent items at the top, and "save for later" apps like Pocket and Instapaper use reverse chronological order. Users expect "the thing I just added is at the top."

**Engagement with reviewer's point:** The reviewer is correct that most users want to see recent additions. The main alternative — alphabetical sorting — optimizes for a different scenario: **finding a specific film by title when you remember what you saved**. Alphabetical order provides predictability and efficient scanning if browsing a long list for a known title. However, this "find a specific saved film" use case is better solved by **search functionality** rather than by sacrificing the default sort order. For a web app, users who remember they saved "Parasite" should use search rather than scroll through an alphabetically-sorted 50-film list. Chronological-by-default serves the primary "what should I watch" workflow, while search handles the edge case of retrieval by title.

**Tradeoff acknowledged:** Newest-first sorting creates a **"watchlist graveyard"** problem—older entries get pushed down and become functionally invisible, not because users lost interest, but because they're buried under newer additions. For users with large watchlists (100+ films), this means the bottom 70-80% may never be seen again, encouraging hoarding behavior rather than actual engagement. The sorting order itself creates the "staleness" I cited as justification. However, I still believe newest-first is the right *default* because it optimizes for the majority use case of smaller, actively-managed watchlists where recency genuinely correlates with intent. **Mitigation strategies** could address the visibility cliff without abandoning chronological default: (1) **periodic "rediscovery" prompts** that surface random older entries ("You saved this 6 months ago—still interested?"), (2) **smart sorting** that mixes recency with diversity signals (boost films you haven't been shown in a while), or (3) **user-configurable default sort** for power users who prefer alphabetical, random, or oldest-first. These features would help users who've accumulated large backlogs rediscover older interests while keeping the approachable newest-first default for typical users. I acknowledge that without these mitigations, newest-first can enable poor watchlist hygiene at scale—but the solution is better tooling for large lists, not forcing alphabetical on everyone. 

## Comment 6 — Rebase
**Reviewer's comment:** A refactor on main changed film IDs from integers to UUIDs. The watchlist code still references integer IDs. Need to rebase on main and update accordingly.

**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->