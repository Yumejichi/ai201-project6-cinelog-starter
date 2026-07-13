# PR Response Doc — CineLog Watchlist Feature

## AI Usage

I used Claude (Claude Code) throughout this project in a few specific ways:

- **Codebase orientation (Milestone 1):** Before looking at any review comments, I asked Claude to summarize `models.py` and `collection_service.py` and to walk through what `add_to_collection()` does step by step, including what it returns when `film_id` doesn't exist. This helped me recognize the `verb_to_noun` naming pattern and the dedup-check pattern before I read Comments 1 and 2, so I understood why the reviewer was asking for them.

- **Locating the real review comments:** My fork didn't show the PR (forking doesn't copy pull requests), so Claude used the GitHub CLI to find the upstream template PR and pull the exact text of all six `@dev-lead` comments, including the wording for Comment 5 ("Most users want to see what they added recently... I'm open to discussion if you see it differently"). I used that exact quote to write my Comment 5 response instead of guessing at the reviewer's reasoning.

- **Stress-testing Comments 4 and 5 (devil's advocate):** After I wrote my own first drafts, I asked Claude what counterargument a careful reviewer would raise against each position. For Comment 4, it pointed out that "not sensitive information" doesn't account for a watchlist revealing a pattern of interest over time, and that public-by-default and private-by-default aren't symmetric risks. For Comment 5, it caught that my first draft said "most recently watched" when the watchlist is about films not yet watched, and pointed out my "engagement" section was just restating agreement rather than responding to the reviewer's actual words. I rewrote both after that.

- **Debugging the rebase (Comment 6):** When my rebase made `WatchlistEntry` disappear from `models.py` with no conflict shown, I asked Claude to explain why. It traced the git history and showed that main's refactor commit had deleted the class outright, and since none of my own commits' diffs touched those same lines, git never saw a dispute to flag. I relied on that explanation to manually restore the model afterward.

I did not ask Claude to write the Comment 4 or Comment 5 reasoning itself — in both cases I wrote my own position first and used AI to find gaps or verify facts afterward.

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
Most users see will be willing to watch the latest film they added, so using the date_added sounds a solid logic.

**Engagement with reviewer's point:**
I agree with sorting by date_added would be better since a user would like to watch the films they adeed most recently.

## Comment 6 — Rebase
**What conflicted:**
Only .gitignore showed as a textual conflict (add/add — I and main independently created one). The real problem — main's refactor commit deleting the WatchlistEntry class entirely — never showed as a conflict at all, because none of my commits' diffs touched those same lines (the class was created in the very first shared commit, so my later commits never "added" it explicitly).

**How I resolved it:**
Merged .gitignore by combining both lists. Then manually re-added WatchlistEntry to models.py post-rebase with film_id as db.String(36) (UUID) to match Film.id, added the missing Film.watchlist_entries relationship, and updated stale "integer"/pre-refactor references in docstrings.

**How I verified no conflict remains:**
Ran pytest tests/ -v (7/7 passed) and confirmed git log --oneline --merges origin/main..HEAD shows no merge commits.

## PR Description

### What this feature does

Adds a watchlist to CineLog so users can save films they want to watch later, separate from their collection of films they've already watched. It introduces a `WatchlistEntry` model and two endpoints:

- `GET /watchlist/<user_id>` — returns the user's watchlist, sorted with the most recently added film first.
- `POST /watchlist/<user_id>/add` — adds a film to a user's watchlist. Body: `{ "film_id": "<uuid>" }`. Returns `404`-equivalent errors (`FilmNotFoundError`) if the film doesn't exist, and rejects duplicate entries (`AlreadyInWatchlistError`) instead of silently creating a second row.

### Design decisions

- **Default visibility (Comment 4):** Watchlist entries default to `public=True` — treating CineLog as a social film-logging app where sharing what you plan to watch is the norm, rather than a private log locked down by default. See Comment 4 above for the full reasoning and tradeoff.
- **Sort order (Comment 5):** `get_watchlist()` sorts by `date_added` descending (most recently added film first) instead of alphabetically, since a watchlist functions as a queue and users care most about what they just decided they want to watch. See Comment 5 above for the full reasoning.

### How to test this manually

1. Start the app: `python app.py`
2. Create a user and a film (or use existing seed data / the `/films/` endpoint).
3. Add a film to a user's watchlist:
   ```bash
   curl -X POST http://localhost:5000/watchlist/<user_id>/add \
     -H "Content-Type: application/json" \
     -d '{"film_id": "<film_id>"}'
   ```
4. View the watchlist and confirm the film appears:
   ```bash
   curl http://localhost:5000/watchlist/<user_id>
   ```
5. Add a second film and confirm it appears **first** in the list (newest-added-first ordering).
6. Try adding the same film again and confirm you get an "already in watchlist" error instead of a duplicate row.
7. Try adding a nonexistent `film_id` and confirm you get a "film not found" error.
8. Run the automated test suite: `pytest tests/ -v` — all 8 tests should pass.