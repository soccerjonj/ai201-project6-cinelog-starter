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
The default should remain public=True
**Reasoning:**
CineLog is described as "a community film tracking app", so its value depends on users being able to see each other's activity. I'm optimizing for discovery and social engagement. New users seeing what films others in their community are saving is the whole point of a shared tracking app. Defaults matter here because most users never change them: if watchlists were private by default, the community features would sit empty because almost no one would opt in. Public-by-default is what makes the "community" in the app's own description actually work.
**Tradeoff acknowledged:**
The cost of this choice is a potential privacy surprise: adding a film to a watchlist feels like a private "save for later," and some users won't expect that action to be visible to others. The more conservative option, private by default, avoids that surprise, since users can't accidentally over-share and can opt into visibility when they want it. I'm accepting the public default because the app is explicitly social and the feature loses most of its value if lists are hidden, but it's a real tradeoff. It's mitigated by the fact that the public field still exists per-entry, so a user can mark an entry private, and we can revisit the default if users report feeling exposed.

## Comment 5 — Sort order
**My position:**
Watchlists should default to "date added" order rather than alphabetical.
**Reasoning:**
Users want to see what they have added recently and be able to scroll down and see what they may have added in the past. Having it be chronological solves that problem whereas having them sorted alphabetically makes it harder for users to find movies they've recently added to their watchlist.
**Engagement with reviewer's point:**
I agree with the reviewer's preference for "date added" order, and with their reasoning that most users want to see what they added recently. Beyond the user-experience point they raised, sorting by date added also makes the watchlist consistent with `get_collection()`, which already orders by `date_added` descending. Aligning the two features means the app behaves the same way across collections and watchlists rather than surprising users with different sort orders, and it's easier to maintain. If implemented, this would be a small change to `get_watchlist()`: switching `.order_by(Film.title.asc())` to `.order_by(WatchlistEntry.date_added.desc())` (which also removes the now-unnecessary join to `Film`).

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->