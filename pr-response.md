# PR Response Doc — CineLog Watchlist Feature

---

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

---

**Codebase Exploration**: I used Claude Code to quickly get a quick but detailed overview of the codebase: `models.py`, `services/*.py`, `routes/*.py`, `tests/*.py`. Through Claude Code, I gathered the general purpose of modules, purpose of functions within modules, and how they relate to each other (how functions are called through different modules). Coupled verification of summaries by reading code, this method speeds up the process of familiarizing the codebase tremendously. 

**Verifying changes**: With Claude Code, I have been able to verify my fixes for all 6 comments quickly and reliably. It has checked if all instances of `save_to_watchlist()` has been replaced by `add_to_watchlist()` (Comment 1), verified my deduplication logic (Comment 2) and generated the nonexistent ID test (Comment 3).

**Presure-testing design decisions (Comment 4 and Comment 5)**: I used Claude Code to consider counter-arguments and think of consequent reinforcements to my design decision on Comment 4 and Comment 5. This makes my arguments and position on the matter sharper, more substantive and more concrete.

**Writing this .md**: I used Claude Code to rewrite and sharpen my sentences, without generating and replacing the main point of the sentence. I used Claude Code to polish my sentence formation, my flow of logic and word choices without replacing my original idea and intent.  

**Writing the PR description**: The PR description was written using Claude Code to make sure that everything written in this markdown, as well as modules and other files within this codebase, gets reflected comprehensively in a digestible paragraphs even for those who are unfamiliar to the new changes.

---

## Comment 1 — Rename

---

**Reviewer's comment:** `save_to_watchlist()` doesn't follow the project's `verb_to_noun` naming convention. Should be renamed to `add_to_watchlist()` to match `add_to_collection()`.

**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` (line 12). Updated the import statement in `routes/watchlist/watchlist.py` (line 8) and the function call in the `add_film()` endpoint (line 32).

**How I verified:**
1. Searched entire codebase for remaining references to `save_to_watchlist` using `grep -r "save_to_watchlist"`, found 0 code references (only appears in this PR response doc)
2. Verified all references to `add_to_watchlist` are correct in `routes/watchlist/watchlist.py` (import and usage)

---

## Comment 2 — Deduplication

---

**Reviewer's comment:** The function doesn't handle duplicate entries. If a user adds the same film twice to their watchlist, it creates duplicate database records instead of raising an error.

**What I did:** Added deduplication logic in `add_to_watchlist()` in `services/watchlist_service.py` (lines 20-25). Created a new `AlreadyInWatchlistError` exception class (lines 12-14). The function now queries for existing entries before creating a new one and raises the exception if a duplicate is found.

**How I verified:**
1. Verified that attempting to add a duplicate film raises `AlreadyInWatchlistError` exception
2. Confirmed database contains only 1 entry after duplicate attempt (verified with `WatchlistEntry.query.filter_by().count()`)

---

## Comment 3 — Missing test

---

**Reviewer's comment:** Need to add a test case for when `film_id` doesn't exist in the database. The pattern is in `test_collection.py`.

**What I did:** added `test_add_to_watchlist_nonexistent_film_raises()` in `tests/test_watchlist.py` to test if error raised when add nonexistent film. The test is modeled after existing test fixtures and structure found in `tests/test_collection.py` and `tests/test_watchlist.py`
**How I verified:** I ran the test and it passed: nonexistent film ID will raise `FilmNotFoundError`

---

## Comment 4 — Default visibility

---

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

---

## Comment 5 — Sort order

---

**Reviewer's comment:** Reviewer prefers "date added" order (newest first) rather than alphabetical. Most users want to see recent additions. Open to discussion but wants a documented decision.

**My position:** I agree with the reviewer — default sort should be **date added, newest first**.

**Reasoning:** I'm optimizing for the **primary watchlist use case: browsing for what to watch next**. When users open their watchlist, they're typically deciding "what should I watch tonight?" rather than searching for a specific film they remember saving. Recent additions have the highest intent to watch — they were added because of a fresh recommendation, trending conversation, or recent interest, making them more contextually relevant and top-of-mind. A film added yesterday after a friend's recommendation is far more likely to be watched than one added three months ago that has since been forgotten. Newest-first surfaces these high-intent items at the top, increasing the likelihood that users actually engage with their watchlist instead of feeling overwhelmed by stale entries they've lost interest in.

This also matches user mental models from similar features: email inboxes default to newest first, browser history shows recent items at the top, and "save for later" apps like Pocket and Instapaper use reverse chronological order. Users expect "the thing I just added is at the top."

**Engagement with reviewer's point:** The reviewer is correct that most users want to see recent additions. The main alternative — alphabetical sorting — optimizes for a different scenario: **finding a specific film by title when you remember what you saved**. Alphabetical order provides predictability and efficient scanning if browsing a long list for a known title. However, this "find a specific saved film" use case is better solved by **search functionality** rather than by sacrificing the default sort order. For a web app, users who remember they saved "Parasite" should use search rather than scroll through an alphabetically-sorted 50-film list. Chronological-by-default serves the primary "what should I watch" workflow, while search handles the edge case of retrieval by title.

**Tradeoff acknowledged:** Newest-first sorting creates a **"watchlist graveyard"** problem—older entries get pushed down and become functionally invisible, not because users lost interest, but because they're buried under newer additions. For users with large watchlists (100+ films), this means the bottom 70-80% may never be seen again, encouraging hoarding behavior rather than actual engagement. The sorting order itself creates the "staleness" I cited as justification. However, I still believe newest-first is the right *default* because it optimizes for the majority use case of smaller, actively-managed watchlists where recency genuinely correlates with intent. **Mitigation strategies** could address the visibility cliff without abandoning chronological default: (1) **periodic "rediscovery" prompts** that surface random older entries ("You saved this 6 months ago—still interested?"), (2) **smart sorting** that mixes recency with diversity signals (boost films you haven't been shown in a while), or (3) **user-configurable default sort** for power users who prefer alphabetical, random, or oldest-first. These features would help users who've accumulated large backlogs rediscover older interests while keeping the approachable newest-first default for typical users. I acknowledge that without these mitigations, newest-first can enable poor watchlist hygiene at scale—but the solution is better tooling for large lists, not forcing alphabetical on everyone. 

---

## Comment 6 — Rebase

---

**Reviewer's comment:** A refactor on main changed film IDs from integers to UUIDs. The watchlist code still references integer IDs. Need to rebase on main and update accordingly.

**What conflicted:** `models.py`, `services/watchlist_service.py`, `routes/watchlist/watchlist.py`

**How I resolved it:** Changed `WatchlistEntry.film_id` in `models.py` (line 80) from `db.Integer` to `db.String(36)` to match the new `Film.id` column. In `services/watchlist_service.py` and `routes/watchlist/watchlist.py`, no code changes were needed — the `film_id` parameter is passed through untouched, so Python's dynamic typing means it works with either an int or a UUID string. I updated the docstrings in `add_to_watchlist()` and the `add_film()` route to say `film_id (str): UUID of the film` instead of `int`, so the documented contract matches main.

**How I verified no conflict remains:**
1. Ran the full suite (`pytest tests/`) — 5 passed, no failures from the merge.
2. Grepped for `db.Integer` and `film_id` across `models.py`, `services/watchlist_service.py`, and `routes/watchlist/watchlist.py` to confirm no remaining integer-typed references to film IDs.
3. **Found a real gap:** `tests/test_watchlist.py` (line 61) still has `fake_film_id = 99999  # Non-existent integer ID`, left over from before the refactor. It currently passes, but only because SQLite's loose typing lets `db.session.get(Film, 99999)` silently fail to match any row — not because the test was actually adapted to the UUID scheme. `tests/test_collection.py` (line 104) shows the correct post-refactor pattern: `fake_film_id = "00000000-0000-0000-0000-000000000000"`. I'll update `test_watchlist.py` to match before merging, so the test asserts against a real UUID-shaped nonexistent ID rather than passing for an unrelated reason.

---

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->

---

### Add watchlist feature

**What this does**

Adds a watchlist so users can bookmark films they intend to watch later, separate from their collection of already-watched films.

- `POST /watchlist/<user_id>/add` — add a film (by `film_id` UUID) to a user's watchlist. Rejects films that don't exist (`FilmNotFoundError`, 4xx) and rejects duplicates (`AlreadyInWatchlistError`) instead of silently creating a second row.
- `GET /watchlist/<user_id>` — return a user's watchlist as a list of film dicts, each annotated with `date_added` and `public`.

New `WatchlistEntry` model (`models.py`) tracks `user_id`, `film_id`, `date_added`, and a per-entry `public` flag. Business logic lives in `services/watchlist_service.py`; routes are thin wrappers in `routes/watchlist/watchlist.py`.

**Design decisions**

1. **Default visibility is private (`public=False`)** — a watchlist reflects unfiltered, exploratory intent-to-watch (curiosity picks, guilty pleasures, recommendations still being vetted), not vetted opinions a user has chosen to endorse publicly. Defaulting private avoids exposing that without consent, matching the convention of similar "save for later" features (browser bookmarks, YouTube Watch Later, Pocket/Instapaper). Users who want a public watchlist can toggle it. Full reasoning and tradeoffs (curator/influencer use case, possible "followers only" middle ground) are in Comment 4 above.
2. **Default sort order is date added, newest first** — the primary watchlist workflow is "what should I watch next," and the most recently added film is the most likely to reflect current intent. This matches user expectations from inboxes, browser history, and other reverse-chronological "save for later" tools. Finding a specific older title by name is better solved by search than by defaulting to alphabetical order for everyone. Full reasoning and tradeoffs (the "watchlist graveyard" problem for large lists, and mitigation ideas) are in Comment 5 above.

**Manual testing steps**

1. Start the app and open a shell/REPL or API client (e.g. `curl`, Postman) pointed at the running server.
2. Create or grab an existing user ID and film ID (e.g. from `/films` or the seed data).
3. **Add a film to the watchlist:**
   ```
   POST /watchlist/<user_id>/add
   Content-Type: application/json

   { "film_id": "<valid-film-uuid>" }
   ```
   Expect `201` with the new entry (including `date_added`) in the response body.
4. **Verify duplicate rejection:** repeat step 3 with the same `user_id`/`film_id`. Expect a rejection (not a second row) since the film is already on the watchlist.
5. **Verify nonexistent film handling:** repeat step 3 with a made-up UUID (e.g. `00000000-0000-0000-0000-000000000000`). Expect a `FilmNotFoundError`-driven error response, not a 201.
6. **Add a second, different film** to the same user's watchlist so there are at least two entries with different `date_added` timestamps.
7. **Fetch the watchlist:**
   ```
   GET /watchlist/<user_id>
   ```
   Expect a `200` with a JSON list of film dicts. Confirm:
   - The most recently added film appears **first** (newest-first ordering).
   - Each entry includes `date_added` and `public` fields.
   - `public` is `false` by default for entries you didn't explicitly mark public.
8. Run the automated suite as a final check: `pytest tests/` — all watchlist tests (including duplicate and nonexistent-film cases) should pass.

---