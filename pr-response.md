# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:**
I renamed the function save_to_watchlist() to add_to_watchlist() in the function definition and call sites.
**How I verified:**
Grepped the whole project for the old name and none remain; app imports cleanly; pytest passes 4/4.

## Comment 2 — Deduplication
**What I did:**
I added deduplication to `add_to_watchlist()` in `services/watchlist_service.py`, following the same pattern as `add_to_collection()`. Specifically I (1) defined a new `AlreadyInWatchlistError` exception (mirroring `AlreadyInCollectionError`), and (2) added a check that queries for an existing `WatchlistEntry` with the same `user_id` and `film_id` before inserting — if one exists, it raises `AlreadyInWatchlistError` instead of creating a duplicate row.
**How I verified:**
Note that the existing test suite does NOT cover the duplicate path (the test I wrote for Comment 3 exercises a *nonexistent* film, not a *duplicate* one), so simply running pytest does not verify this logic. I verified it directly instead: added the same film to a user's watchlist twice. The first call succeeded, the second raised `AlreadyInWatchlistError`, and a `WatchlistEntry.query.filter_by(...).count()` confirmed exactly one row existed in the database — no duplicate was created.
## Comment 3 — Missing test
**What I did:**
I created a new file tests/test_watchlist.py. Writing the equivalent test of test_add_to_collection_nonexistent_film_raises but now as test_add_to_watchlist_nonexistent_film_raises following the same fixture and assertion structure.
**How I verified:**
I ran the new pytest as well as the whole pytest suite to verify it worked.
## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->