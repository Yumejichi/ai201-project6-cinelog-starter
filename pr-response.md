# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:**
Changed the name of function save_to_watchlist() to add_to_watchlist() in services/watchlist_service.py

**How I verified:**
Searched all save_to_watchlist() in the folder and changed the corresponding function name. Alsi ran the app to check no error shown and ran the pytest tests/ -v to check nothing breaks.

## Comment 2 — Deduplication
**What I did:**
Add a duplicaate check logic in add_to_watchlist() in services/watchlist_service.py and a new AlreadyInWatchlistError in collection_service.py

**How I verified:**
Ran the app to see if the app rans correctly and ran the pytest tests/ -v to check nothing breaks.

## Comment 3 — Missing test
**What I did:**
Created tests/test_watchlist.py, modeled on test_add_to_collection_nonexistent_film_raises in test_collection.py. Wrote test_add_to_watchlist_nonexistent_film_raises, using the same app/sample_user fixtures, asserting add_to_watchlist() raises FilmNotFoundError for a fake film ID via pytest.raises.

**How I verified:**
Ran pytest tests/test_watchlist.py -v (3 passed) and then the full suite pytest tests/ -v (7 passed) to confirm nothing else broke.

## Comment 4 — Default visibility
**My position:**
Make the default visibility as public

**Reasoning:**
This app can be a personal log for films watched but I think it would be better to be implemented as a social app then the public visibility will make sense.

**Tradeoff acknowledged:**
We need to take care and consider more when implement the features due to it's public visibility such as adding logic to let the user choose if make their logs public or not.

## Comment 5 — Sort order
**My position:**
The newerest first should ne persist

**Reasoning:**
Most users see latest filmes they watched more impoertant and it can be better when we want to use this as future additional features such as recommending films.

**Engagement with reviewer's point:**
I agree with current date-added as the order for sorting.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->