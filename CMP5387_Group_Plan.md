# CMP5387 Group Plan: Online Videogames Database with Social Features

**Assumptions (check on Moodle):**
- Teaching week 1 began Mon 21 Sept 2026, with no reading week. That puts week 9 in the week of 16 Nov.
- The D2 progress meeting is assumed to be in the Tuesday lab of week 9 (Tue 17 Nov). The document upload is due at 15:00; confirm the exact day on Moodle.
- D3 is due at 15:00 in the first week of the January assessment period (week of 4 Jan 2027). Confirm the exact date.
- Everyone must attend the progress meeting. If the group misses it, every D2 aspect is capped at 2/5.

---

## 1. Team

Every name and student ID must appear in the D2 document and the D3 video. Without them, the submission gets no marks.

| # | Task | Name | Student ID | D2 signed off |
|---|---|---|---|---|
| 1 | Users, auth and profiles | Ayaan Ahmed | | |
| 2 | Catalogue browse | Lakhan | | |
| 3 | Catalogue maintenance | Haarisah | | |
| 4 | Personal collections | Lee | | |
| 5 | Reviews and ratings | Zeeshan | | |
| 6 | Friends, following and activity | Husray | | |
| 7 | Forums, user content and integration | Victor | | |

All seven tasks are committed scope. Each one is a vertical slice: the owner designs and builds its tables, endpoints and Bruno tests. That lets every member state their contribution to the database, the web service and testing, which the D3 intro requires.

---

## 2. Decisions to lock at CP0 (Tue 3 Nov lab)

| Decision | Options | Recommendation | Agreed |
|---|---|---|---|
| Stack | Python version; Flask or FastAPI; DBMS | Use the framework and DBMS taught in the labs. Pin the Python version in the README. | |
| Collection status | Single enum (owned / played / wanted) or separate flags | Single enum. It matches the stakeholder wording ("change the status"), and changing status updates the one row. | |
| Rating scale | 1 to 5, or 1 to 10 integers | 1 to 10 integers, enforced at the API (422) and with a CHECK in the DB | |
| Friend model | Mutual friends only, follows only, or both | Both: mutual friend requests plus one-way follows (the topic asks for both) | |
| Auth | Session or token; hashing library | Token auth; passwords hashed with bcrypt or argon2, never stored in plain text | |
| Pagination | Page size | Default 20, maximum 100 | |
| Unknown filter value (e.g. a genre that doesn't exist) | 200 with an empty list, or 400 | 200 with an empty list for filters; 404 for an unknown id in the path | |
| Delete policy for games | CASCADE, RESTRICT or soft delete | ON DELETE RESTRICT. Maintainers correct entries with PATCH. | |

---

## 3. Stakeholders (draft for the D2 table)

| Stakeholder | Kind | Why they are interested | Key needs |
|---|---|---|---|
| Player | Primary user | Discover games and check platform availability | Browse and filter by title, genre and platform |
| Collector | Primary user | Track the games they own, have played or want | One record per game; status updates rather than duplicates |
| Reviewer | Primary user | Share opinions and see community ratings | Rate on an agreed scale; edit their own review only |
| Social user | Primary user | Connect with other gamers | Friends, follows, an activity feed, forums |
| Catalogue Maintainer | Privileged user | Keep catalogue data accurate | Correct game details and platforms without losing player data |
| Gaming community (client) | Organisation | Wants a trustworthy, community-driven platform | Data integrity; players can't damage other players' data |
| Development and maintenance team | Implicit | Has to run, test and extend the system | Reproducible setup, documented API, consistent error handling |

---

## 4. Requirements

### Functional (from the problem statement and topic)

| ID | Requirement | Task |
|---|---|---|
| FR1 | Browse and search games by title, genre and platform | 2 |
| FR2 | Game details include description, genres, release information per platform, and platforms | 2, 3 |
| FR3 | Players keep personal collections (owned / played / wanted), separate from the shared catalogue | 4 |
| FR4 | Changing a game's status updates the existing record and never creates a duplicate | 4 |
| FR5 | Players rate games on the agreed scale and write reviews; they can revise their own and never edit anyone else's | 5 |
| FR6 | Reviews from different players stay distinct, even for the same game; community rating is an aggregate | 5 |
| FR7 | Maintainers correct game details and platform links without erasing player collections or reviews | 3 |
| FR8 | Invalid ratings, unknown games and changes to another player's data are rejected, with existing data left unchanged | All |
| FR9 | Customisable profiles with favourite games | 1 |
| FR10 | Friend requests, follows and a feed of friends' gaming activity | 6 |
| FR11 | Forums per game, plus user-submitted guides and screenshot links | 7 |
| Out of scope | Hosting or running games; uploading video files | n/a |

### Implicit and non-functional

| ID | Requirement | How we evidence it |
|---|---|---|
| NFR1 | Security: hashed passwords, token auth, role checks on maintainer routes | Bruno 401 and 403 tests; show the hashed column in the DB |
| NFR2 | Integrity: constraints in the DB as well as validation in the API | CHECK, UNIQUE, FK constraints in the creation script |
| NFR3 | Consistency: multi-step writes are atomic | Transactions; GET before and after every sad path |
| NFR4 | Usability of the API: consistent JSON responses and the error contract | Same error shape on every endpoint |
| NFR5 | Performance: paginated lists; indexes on commonly filtered columns | Indexes on game title, genre and platform joins |
| NFR6 | Reproducibility: the system runs from the zip on a clean machine | CP5 clean-machine test |
| NFR7 | Maintainability: one slice per module, a README, comments on key code | Repo structure; code walkthrough in the video |
| NFR8 | Accessibility of the client: Bruno collection organised and documented | Bruno folders per slice, each request named |

---

## 5. Shared rules (everyone)

### Error contract (owned by Ayaan)

| Status | When to use | Example |
|---|---|---|
| 400 | Malformed body or a required field missing | Review with no `rating` field |
| 401 | Not logged in | Collection update with no token |
| 403 | Changing someone else's data, or not a maintainer | PUT on another user's collection; PATCH a game as a player |
| 404 | Unknown game, user, review or thread | PUT a collection entry for game 99999 |
| 409 | Duplicate or inconsistent change | Second review for the same game; deleting a game that has player records |
| 422 | Value outside the allowed range or set | Rating 11; status `borrowed` |

Error body shape (same everywhere): `{"error": {"code": 422, "message": "rating must be between 1 and 10", "field": "rating"}}`

### Rules for every endpoint
- Order of checks: authenticate, then validate input, then check the resource exists, then check ownership or role, then write inside a transaction.
- A failed request leaves the database unchanged.
- Foreign keys from player data (collections, reviews, favourites, threads) to games use ON DELETE RESTRICT.
- Bruno coverage per endpoint: one happy-path request, at least two sad-path requests, and for writes a GET before and after the sad path to prove nothing changed.
- Fill in your endpoint rows and table rows in the shared D2 templates yourself.
- Work on one branch per task; another member reviews each PR before it is merged.
- Any AI-generated code is cited in a comment with a sample prompt, as the brief requires, and logged in AI_USAGE.

### Endpoint inventory row format (D2)
| Route | Method | Description | Users | Happy input | Happy output | Sad inputs | Sad outputs |
|---|---|---|---|---|---|---|---|

### DB design row format (D2)
| Table | Columns (type) | Relationships | Constraints |
|---|---|---|---|

---

## 6. Tasks per person

Read the plain-English brief in the box first: it says what problem your slice solves and where to start. The spec below the box is the detail you build and put in your D2 rows. Technical terms are explained in section 15.

### Task 1: Users, auth and profiles
**Owner:** Ayaan Ahmed

> **The problem:** the system has to know who is making each request and what they are allowed to do. Without this, anyone could edit anyone's collection or change the catalogue.
>
> **What you build, in plain terms:**
> - Sign-up and log-in. Logging in gives the user a token they send with every request.
> - Two kinds of user: player and maintainer.
> - A profile with a bio and favourite games.
> - Two shared checks the rest of the group calls: "is this person logged in?" and "do they own this record, or are they a maintainer?"
> - The shared error format, so every slice fails in the same way.
>
> **Rules your slice enforces:** emails and usernames are unique; passwords are never stored in readable form; you can only edit your own profile.
>
> **What you show in the demo:** logging in; a request refused with no token; a player refused when editing someone else's profile.
>
> **Start here:** (1) agree the token approach at CP0; (2) create the users table; (3) build register and login; (4) give the group the two shared checks by the end of week 9, because five slices are waiting on them.

**Tables**
- `users`: id, username UNIQUE, email UNIQUE, password_hash, role CHECK IN ('player','maintainer'), bio, created_at
- `favourite_games`: user_id FK, game_id FK RESTRICT, PRIMARY KEY (user_id, game_id)

**Endpoints**
| Method | Route | Users |
|---|---|---|
| POST | /auth/register | Anyone |
| POST | /auth/login | Anyone |
| GET | /users/{id} | Anyone |
| PATCH | /users/{id} | Owner only |
| PUT / DELETE | /users/{id}/favourites/{game_id} | Owner only |

**Required sad paths:** duplicate email (409); wrong password (401); PATCH another user (403); favourite an unknown game (404).

**Shared work:** auth and role middleware used by every slice; the error contract and error helper.

**D2 lead:** title page, NFR section, error contract.

**D3 lead:** intro slide with every name and ID together, plus the slide deck template.

**Done when:** every other slice can protect a route with one decorator or dependency, and every error uses the shared format.

### Task 2: Catalogue browse
**Owner:** Lakhan

> **The problem:** players need to find games. The catalogue is the shared list of games that everyone sees, and it has to be searchable by title, genre and platform.
>
> **What you build, in plain terms:**
> - The games and genres tables.
> - A list of games that can be filtered and comes back in pages.
> - A single game's details: description, genres, platforms with release dates, and its average rating.
>
> **Rules your slice enforces:** anyone can read the catalogue but nobody edits it here (editing is Haarisah's slice); an unknown game gives 404; lists are paginated.
>
> **What you show in the demo:** searching by title, filtering by genre and platform, and opening one game.
>
> **Start here:** (1) create the games table first, because every other slice links to it; (2) agree the seed games with Victor; (3) build the single-game endpoint, then the list, then the filters.

**Tables**
- `games`: id, title NOT NULL, description, developer, publisher
- `genres`: id, name UNIQUE
- `game_genres`: game_id FK, genre_id FK, PRIMARY KEY (game_id, genre_id)

**Endpoints**
| Method | Route | Users |
|---|---|---|
| GET | /games?title=&genre=&platform=&page=&size= | Anyone |
| GET | /games/{id} (includes genres, platforms with release dates, average rating) | Anyone |
| GET | /genres | Anyone |
| GET | /platforms | Anyone |

**Required sad paths:** unknown game id (404); page 0 or size 500 (400); a filter with no matches (200 with an empty list, as agreed).

**D2 lead:** problem summary (explicit and implicit requirements) and the stakeholder table.

**D3 lead:** video editing and timing, keeping it to 20 minutes or less.

**Done when:** filters combine correctly, results are paginated, and the indexes are in the creation script.

### Task 3: Catalogue maintenance
**Owner:** Haarisah

> **The problem:** catalogue information will sometimes be wrong, such as a typo in a description or a missing platform. Maintainers have to fix it without destroying anything players have recorded about that game.
>
> **What you build, in plain terms:**
> - Platforms, and the link between a game and a platform, with a release date per platform.
> - Maintainer-only endpoints to add a game, edit a game, and add or remove a platform.
> - A change history recording who changed what, and when.
>
> **Rules your slice enforces:** only maintainers can make changes; a game that players have reviewed or collected cannot be deleted (409); every fix is written to the history.
>
> **What you show in the demo:** fixing a description, then showing the reviews and collections for that game are untouched; a player refused when trying to edit.
>
> **Start here:** (1) create the platform tables; (2) build the edit-game endpoint using Ayaan's maintainer check; (3) write to the history in the same transaction as the fix.

**Tables**
- `platforms`: id, name UNIQUE
- `game_platforms`: game_id FK, platform_id FK, release_date, PRIMARY KEY (game_id, platform_id)
- `catalogue_changes`: id, game_id FK, maintainer_id FK, field, old_value, new_value, changed_at

**Endpoints**
| Method | Route | Users |
|---|---|---|
| POST | /games | Maintainer |
| PATCH | /games/{id} | Maintainer |
| PUT | /games/{id}/platforms/{platform_id} | Maintainer |
| DELETE | /games/{id}/platforms/{platform_id} | Maintainer |
| DELETE | /games/{id} | Maintainer (409 if player records exist) |
| GET | /games/{id}/history | Anyone |

**Required sad paths:** player attempts a PATCH (403); delete a game that has reviews (409, with a before/after GET of the reviews); duplicate platform link (409); PATCH an unknown game (404).

**Key evidence:** fix a game description in the demo, then show that the reviews and collections for that game are untouched.

**D2 lead:** database design table and the physical ERD.

**D3 lead:** "design vs implementation" reflection section.

**Done when:** every maintainer write is logged in `catalogue_changes` in the same transaction.

### Task 4: Personal collections
**Owner:** Lee

> **The problem:** each player keeps their own list of games they own, have played or want to play. These lists are separate from the shared catalogue, and only the player can change their own.
>
> **What you build, in plain terms:**
> - One record per player per game, holding a status.
> - Endpoints to add a game to your list, change its status, and remove it.
> - A view of any player's list, filterable by status.
>
> **Rules your slice enforces:**
> - Changing a status updates the existing record and never adds a second one.
> - You can't change another player's list (403).
> - An unknown game gives 404.
> - The status must be owned, played or wanted (422).
>
> **What you show in the demo:** adding a game as wanted, changing it to played, and showing there is still only one record; trying to change another player's list and being refused.
>
> **Start here:** (1) create the table with its one-record-per-player-per-game rule; (2) build the endpoint that creates or updates; (3) write the sad-path tests.

**Tables**
- `collection_entries`: id, user_id FK, game_id FK RESTRICT, status CHECK IN ('owned','played','wanted'), hours_played CHECK >= 0, updated_at, UNIQUE (user_id, game_id)

**Endpoints**
| Method | Route | Users |
|---|---|---|
| PUT | /users/{id}/collection/{game_id} (upsert: 201 on create, 200 on update) | Owner only |
| GET | /users/{id}/collection?status= | Anyone |
| DELETE | /users/{id}/collection/{game_id} | Owner only |

**Required sad paths:**
- PUT on another user's collection (403, other user's list unchanged).
- Unknown game (404).
- Status `borrowed` (422).
- Second PUT with a new status updates the existing row and creates no duplicate.

**D2 lead:** Gantt chart, assembling the D2 document, and acting as the designated D2 uploader.

**D3 lead:** designated D3 uploader, including testing the OneDrive link with staff-level access before submitting.

**Done when:** the "no repeated entries" story is demonstrable, with a row count before and after.

### Task 5: Reviews and ratings
**Owner:** Zeeshan

> **The problem:** players want to share opinions and see what the community thinks of a game. Each player gets one review per game, can change it later, and can't touch anyone else's.
>
> **What you build, in plain terms:**
> - The reviews table.
> - Endpoints to write, edit and delete your own review.
> - A list of all reviews for a game.
> - The game's average rating and number of reviews.
>
> **Rules your slice enforces:** ratings are whole numbers from 1 to 10; one review per player per game, so a second one gives 409; only the author can edit or delete a review (403).
>
> **What you show in the demo:** two players reviewing the same game, with both reviews shown separately; a rating of 11 rejected.
>
> **Start here:** (1) create the reviews table with its rules; (2) build writing a review, then editing with the author check; (3) build the average-rating query.

**Tables**
- `reviews`: id, user_id FK, game_id FK RESTRICT, rating CHECK BETWEEN 1 AND 10, body (length limit), created_at, updated_at, UNIQUE (user_id, game_id)

**Endpoints**
| Method | Route | Users |
|---|---|---|
| POST | /games/{id}/reviews | Logged-in player |
| GET | /games/{id}/reviews?page= | Anyone |
| GET | /games/{id}/rating (average and count) | Anyone |
| PUT | /reviews/{id} | Author only |
| DELETE | /reviews/{id} | Author only |

**Required sad paths:** rating 11 or 0 (422); a second review for the same game (409); edit someone else's review (403, with a before/after GET); review an unknown game (404).

**D2 lead:** compiling the full endpoint inventory, so that every row matches the format and the error contract.

**D3 lead:** demo script, which is the 30% section (see section 11).

**Done when:** two users' reviews of the same game display separately, and the aggregate rating updates on create, edit and delete.

### Task 6: Friends, following and activity feed
**Owner:** Husray

> **The problem:** the topic is a social platform, so players want to connect and see what their friends are playing and reviewing.
>
> **What you build, in plain terms:**
> - Friend requests: send, accept, decline and remove.
> - Following, which is one-way and needs no acceptance.
> - A feed of recent collection and review activity from friends and people you follow.
>
> **Rules your slice enforces:** you can't friend or follow yourself; no duplicate requests; only the person who received a request can accept it.
>
> **What you show in the demo:** sending a request, accepting it, then seeing the friend's new review in the feed.
>
> **Start here:** (1) create the friendship and follow tables, which only need Ayaan's users; (2) build the request and accept endpoints; (3) leave the feed until last and write it against seed data until Lee's and Zeeshan's slices are ready.
>
> **Also:** you are the group's tester. You own the traceability matrix and run the clean-machine test.

**Tables**
- `friendships`: id, requester_id FK, addressee_id FK, status CHECK IN ('pending','accepted'), CHECK requester_id <> addressee_id, UNIQUE on the unordered pair
- `follows`: follower_id FK, followee_id FK, CHECK follower_id <> followee_id, PRIMARY KEY (follower_id, followee_id)

**Endpoints**
| Method | Route | Users |
|---|---|---|
| POST | /friends/requests | Logged-in player |
| PATCH | /friends/requests/{id} (accept or decline) | Addressee only |
| GET | /users/{id}/friends | Anyone |
| DELETE | /friends/{user_id} | Either friend |
| POST / DELETE | /users/{id}/follow | Logged-in player |
| GET | /feed?page= (friends' and followees' recent collection and review changes) | Logged-in player |

**Required sad paths:** friend or follow yourself (422); duplicate request (409); accept a request addressed to someone else (403); request to an unknown user (404).

**Note:** the feed is a derived query over `collection_entries` and `reviews`, with no separate activity table. Justify that in D2 as avoiding duplicated data.

**D2 lead:** evaluation ideas, the test plan, and the traceability matrix (each story mapped to its endpoints and to its Bruno tests).

**D3 lead:** evaluation reflection, including test results against FRs and NFRs.

**Done when:** the feed returns correct, paginated results for seeded data.

### Task 7: Forums, user content and integration
**Owner:** Victor

> **The problem:** players want to discuss games and share guides and screenshots. The group also needs one person who makes the whole system come together and run on any machine.
>
> **What you build, in plain terms:**
> - Discussion threads for each game, and posts in them.
> - Guides, and screenshot links.
> - The database creation script, seed data, README and final zip.
>
> **Rules your slice enforces:**
> - Only the author can edit a post.
> - The author or a maintainer can delete it.
> - Screenshots are stored as https links only, with no file uploads.
> - An empty post is rejected.
>
> **What you show in the demo:** creating a thread and a post; another user refused when trying to edit it.
>
> **Start here:** (1) set up the repo and a skeleton creation script in week 9 so others can add their tables; (2) write the seed data; (3) build the forums.

**Tables**
- `threads`: id, game_id FK RESTRICT, user_id FK, title NOT NULL, created_at
- `posts`: id, thread_id FK, user_id FK, body NOT NULL, created_at, updated_at
- `user_content`: id, user_id FK, game_id FK, type CHECK IN ('guide','screenshot'), title, body or url (CHECK url starts with https://), created_at

**Endpoints**
| Method | Route | Users |
|---|---|---|
| GET / POST | /games/{id}/threads | Anyone / logged-in |
| GET / POST | /threads/{id}/posts | Anyone / logged-in |
| PATCH | /posts/{id} | Author only |
| DELETE | /posts/{id} | Author or maintainer |
| GET / POST | /games/{id}/content | Anyone / logged-in |
| DELETE | /content/{id} | Author or maintainer |

**Required sad paths:** post to an unknown thread (404); edit someone else's post (403); a non-https screenshot URL (422); an empty post body (400).

**Integration work:**
- Repo setup.
- DB creation script for all tables.
- Seed data: at least 20 games, 5 platforms, 6 genres, and 6 users including 1 maintainer.
- requirements.txt.
- README.
- Assembling the final zip.

**D2 lead:** AI usage section.

**D3 lead:** README, zip contents and the AI_USAGE file.

**Done when:** a fresh clone reaches a working API using only the README steps.

---

## 7. Timeline

| Phase | W7 2 Nov | W8 9 Nov | W9 16 Nov | W10 23 Nov | W11 30 Nov | W12 7 Dec | W13 14 Dec | Break 21 Dec to 3 Jan | Jan 4 Jan |
|---|---|---|---|---|---|---|---|---|---|
| Lock decisions, stakeholders, requirements (everyone) | X | | | | | | | | |
| Endpoint and table rows per slice (everyone) | X | X | | | | | | | |
| ERD (Haarisah), Gantt (Lee), evaluation ideas (Husray), AI section (Victor) | X | X | | | | | | | |
| D2 assembly (Lee), sign-off and rehearsal (everyone) | | X | | | | | | | |
| D2 progress meeting (everyone), upload (Lee) | | | X | | | | | | |
| Repo, DB script, seed data (Victor) | | | X | | | | | | |
| Auth and errors (Ayaan), catalogue browse (Lakhan) | | | X | X | | | | | |
| Haarisah, Lee, Zeeshan, Husray, Victor build their slices | | | | X | X | X | | | |
| Integration, full Bruno run, bug fixing (everyone) | | | | | | X | X | | |
| Clean-machine test (Husray), README (Victor), recordings (everyone) | | | | | | | X | | |
| Video edit (Lakhan), final checks (Lee, Victor) | | | | | | | | X | |
| D3 submission (Lee) | | | | | | | | | X |

**Dependencies**
- Ayaan (auth) and Lakhan (catalogue browse) are the foundation and come first.
- Haarisah, Lee, Zeeshan and Victor need auth and the games table.
- Husray's feed needs Lee's collections and Zeeshan's reviews. Friends and follows only need auth, so Husray starts with those.
- Integration needs every slice merged.
- While waiting on dependencies, slices can build against seed data and the agreed endpoint contract from D2.

**Other dates:** D1 quizzes in the Tuesday labs on 10 Nov (week 8) and 1 Dec (week 11). These are individual, so plan around them.

---

## 8. Checkpoints

| CP | Date | Pass criteria | Checked by |
|---|---|---|---|
| CP0 | Tue 3 Nov | Section 2 decisions agreed; all student IDs collected; repo created | Lee |
| CP1 | Fri 13 Nov | D2 document complete; every endpoint row has happy and sad paths; every table has constraints; ERD is physical; all 7 signed off; under 4000 words | Zeeshan and Haarisah |
| CP2 | Tue 17 Nov | All 7 attend the meeting; each person can explain their own slice; document uploaded before 15:00 | Lee |
| CP3 | Fri 27 Nov | Fresh clone → create DB → seed → start server → Bruno register, login and browse all pass | Victor |
| CP4 | Fri 11 Dec | Feature freeze: every endpoint in the inventory implemented; full Bruno suite passes; every sad path shows data unchanged | Husray |
| CP5 | Fri 18 Dec | Clean-machine test, run by someone who didn't write the setup; README complete; every member's code walkthrough recorded | Husray runs, Victor fixes |
| CP6 | Thu 31 Dec | Video 20 minutes or less; intro slide shows all names and IDs; each member states their DB, web service and testing contribution; link opens from a non-team account | Lakhan and Lee |
| CP7 | Jan, by 15:00 | Video link and zip submitted. Aim for 1 to 2 days early. The video file isn't touched after submission. | Lee |

**Definition of done for any endpoint:** matches its inventory row; uses the error contract; has happy and sad Bruno requests; DB constraints are in the creation script; reviewed by one other member.

---

## 9. D2 submission checklist

- [ ] Title page with every name and student ID
- [ ] Problem summary covering explicit and implicit requirements (2%)
- [ ] Stakeholder table: kind of stakeholder and why they are interested (2%)
- [ ] Gantt chart covering all members, with timeline per subtask and a dependency analysis (4%)
- [ ] Endpoint inventory: route, description, method, happy and sad inputs and outputs, users involved (5%)
- [ ] DB design: tables with columns, relationships and constraints, plus the physical ERD (5%)
- [ ] Evaluation ideas: how we will prove each requirement is met (2%)
- [ ] AI usage section
- [ ] Contribution of each member clear; all 7 signed off
- [ ] Justification of each design decision (the meeting is marked mainly on this)
- [ ] Under 4000 words of body text (tables, ERD, Gantt and code don't count); font size 11; 1.5 spacing; cover sheet; Harvard references
- [ ] Meeting rehearsed to 15 minutes or less, with each person speaking about their own slice

---

## 10. D3 video plan (20 minutes maximum, target 18)

| Section | Marks | Time | Lead |
|---|---|---|---|
| Intro: each person gives name, ID, and their contribution to DB, web service and testing; all names and IDs on one slide | Compulsory | 2:00 | Ayaan |
| Problem and requirements identified | Part of the narrative | 1:00 | Lakhan |
| Live demo of the system working (section 11 script) | 30% | 7:00 | Zeeshan |
| Code-level explanation of key parts (about 40 s per person, own slice) | 10% | 4:30 | All |
| Design vs implementation: what changed and why, what we missed | 10% | 1:30 | Haarisah |
| Evaluation: testing approach, results against FRs and NFRs | 10% | 1:30 | Husray |
| Resources and AI reflection | Required content | 0:30 | Victor |

---

## 11. Demo script (follows the stakeholder stories)

1. Register and log in. Show a 401 with no token.
2. Browse games by title, genre and platform; open a game's details.
3. Add a game to the collection as `wanted`, then change it to `played`. Show there is still one row.
4. Try to change another player's collection. Show the 403 and that their list is unchanged.
5. Two players review the same game. Show both reviews are distinct and the aggregate rating.
6. Submit rating 11 (422), a review for an unknown game (404), and a second review (409). Show the data is unchanged each time.
7. Edit someone else's review (403). Edit your own (200).
8. As a maintainer, fix a description and a platform link. Show the existing collections and reviews are intact, and the change history.
9. As a player, attempt a maintainer edit (403). As a maintainer, try to delete a game with reviews (409).
10. Send a friend request, accept it, follow someone, and view the feed.
11. Create a forum thread and post, and add a guide. Show a 403 on editing someone else's post.

---

## 12. README and zip checklist

The README goes in the root of the zip:
- [ ] Design mapping (which design element maps to which code) and the finalised designs
- [ ] Python version assumptions
- [ ] Dependencies, plus a separate requirements.txt
- [ ] Installation steps for a clean machine
- [ ] Database creation and seeding commands
- [ ] Application startup commands
- [ ] How to open and run the Bruno collection, including the environment and token setup
- [ ] Custom frontend instructions, if we build one (optional enhancement)

The zip contains:
- [ ] Backend source code
- [ ] Bruno collection (all requests, organised by slice)
- [ ] DB creation script and seed data
- [ ] AI_USAGE file
- [ ] Anything else needed to reproduce the system on a clean install

---

## 13. AI usage log (D2 section and D3 AI_USAGE)

| Tool | Date | Purpose | Representative prompt | Output used or modified | How we verified it | Important changes made |
|---|---|---|---|---|---|---|
| | | | | | | |

Log every use as it happens, including planning help. Rebuilding this log from memory in January won't work.

---

## 14. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Foundation late (auth and catalogue) | Blocks five other slices | Ayaan and Lakhan finish by CP3 (Fri 27 Nov); others build against seed data meanwhile |
| D2 is two weeks away | Weak design marks | Rows drafted by CP1 (Fri 13 Nov); assembly owned by Lee |
| Husray's feed blocked on Lee's and Zeeshan's slices | Idle time, late feature | Build friends and follows first; write the feed query against seed data |
| Schema drift between slices | Integration breaks | One shared creation script owned by Victor; schema changes only via PR |
| Members away over Christmas | Video not recorded | All recordings done by CP5 (18 Dec) |
| Clean-machine setup fails | Reproducibility marks lost | CP3 and CP5 tests run by Husray, not Victor who wrote the setup |
| A member contributes little | Individual mark adjusted | Vertical slices, a visible PR history, and an own-slice segment in the meeting and video |
| Video link inaccessible to staff | Whole group fails | Test the link from a non-team account; don't edit the file after submission |

---

## 15. Glossary

| Term | Meaning |
|---|---|
| Slice | One person's feature from top to bottom: its tables, endpoints and tests |
| Endpoint | One URL plus method the API answers, e.g. `GET /games/{id}` |
| Happy path | A correct request, and the successful response it should get |
| Sad path | A wrong request (bad data, wrong user, unknown id), and the error it should get |
| Bruno | The API client we use to send requests and save them as tests |
| Token | A string returned at login that the client sends with every request to prove who it is |
| Player / maintainer | Normal user / user allowed to edit the catalogue |
| 400 / 401 / 403 / 404 / 409 / 422 | Bad request / not logged in / not allowed / doesn't exist / clashes with existing data / value not allowed (see section 5) |
| Constraint | A rule the database itself enforces, so bad data can't get in even if the API has a bug |
| UNIQUE | Constraint: no two rows can have the same value(s), e.g. one review per player per game |
| CHECK | Constraint: a value must pass a test, e.g. rating between 1 and 10 |
| Foreign key (FK) | A column that must point to a real row in another table, e.g. a review's `game_id` |
| ON DELETE RESTRICT | The database refuses to delete a row while other rows still point to it |
| Transaction | A group of changes that either all happen or none do, so a failed request leaves nothing half-done |
| Upsert | One request that creates the record if it doesn't exist, or updates it if it does |
| Pagination | Returning a list in pages (e.g. 20 at a time) instead of everything at once |
| Seed data | Sample games, users and reviews loaded into the database for testing and the demo |
| Unordered pair | A and B count as the same pair as B and A, so a friendship can't be stored twice |
| Clean machine | A computer that has never run the project, used to prove the README steps work |
