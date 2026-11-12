# CMP5387 Group Plan: Online Videogames Database with Social Features

**How to use this document**
- Start with your own row in section 1 and your own task in section 6. Then read the shared rules (section 5) and the dates (sections 8 and 9).
- Every technical term is explained in the glossary (section 18).
- Your task's endpoint table shows which of your endpoints are Must, Should or Could (the priorities are explained in section 4).
- Anything marked "proposed" needs the group's agreement at CP0 (Tue 3 Nov).

**Dates and admin to confirm on Moodle**
- Teaching week 1 began Mon 21 Sept 2026, with no reading week, so week 9 is the week of 16 Nov.
- The D2 progress meeting is assumed to be in the Tuesday lab of week 9 (Tue 17 Nov). The document upload is due at 15:00; confirm the exact day.
- D3 is due at 15:00 in the first week of the January assessment period (week of 4 Jan 2027). Confirm the exact date.
- Everyone must attend the progress meeting. If the group misses it, every D2 aspect is capped at 2/5.
- The D2 document must follow the template on Moodle. This plan has been checked against the D2 5-star checklist and the D3 30-mark checklist, but not yet against that template.

---

## 1. Team

Every name and student ID must appear in the D2 document and the D3 video. Without them, the submission gets no marks.

| # | Task | Name | Student ID | D2 signed off |
|---|---|---|---|---|
| 1 | Users, auth and profiles | Ayaan Ahmed | | |
| 2 | Catalogue browse and user content | Lakhan | | |
| 3 | Catalogue maintenance | Haarisah | | |
| 4 | Personal collections | Lee | | |
| 5 | Reviews and ratings | Zeeshan | | |
| 6 | Friends, following and activity feed | Husray | | |
| 7 | Forums and integration | Victor | | |

Each task is a vertical slice: its owner designs and builds the slice's tables, endpoints and tests. This lets every member show their own contribution to the database, the web service and testing, which the D3 introduction requires. All seven tasks are committed scope, and each contains Must work (section 4).

### Everyone's responsibilities at a glance

A D2 lead writes that part of the document and presents it at the meeting. A D3 lead scripts that part of the video. Everyone still writes their own slice's D2 rows and explains their own code.

| Name | D2 lead | D3 lead | Other responsibilities | Checkpoints they check |
|---|---|---|---|---|
| Ayaan | Title page; non-functional requirements; error contract | Introduction and slide template | Shared login and permission checks for every slice | None (foundation work) |
| Lakhan | Problem summary; stakeholder table; MoSCoW priorities | Topic and requirements; video editing | OpenAPI docs (E2); runs the clean-machine test | CP5, CP6 |
| Haarisah | Database design and physical ERD | Design vs implementation | Design change log; notes the tutor's feedback at the D2 meeting | CP1 (database) |
| Lee | Gantt chart; assembling the document; references; cover sheet; upload | Uploading the video link and zip | Collecting student IDs; checking the Moodle template | CP0, CP1 (document), CP2, CP6, CP7 |
| Zeeshan | Endpoint inventory (compiling everyone's rows) | Demo script | Shared pytest setup (E1) | CP1 (inventory), CP3 |
| Husray | Evaluation ideas; traceability matrix | Evaluation | Full test run at the feature freeze | CP4 |
| Victor | AI usage section | README, zip and AI_USAGE; resources and AI reflection | Repo, app structure, shared schema, seed data, Bruno collection | None (fixes setup problems found at CP3 and CP5) |

---

## 2. Decisions and assumptions

### Decisions to agree at CP0 (Tue 3 Nov lab)

| Decision | Options | Recommendation | Agreed |
|---|---|---|---|
| Stack | Flask or FastAPI; which database | Flask with Flask-SQLAlchemy (an ORM) and SQLite. These are the examples the module's own checklists use, and SQLite needs no database server, which keeps the clean-machine test simple. Everyone uses the Python version on the lab machines. | |
| Database type | Relational (SQL) or graph (Neo4j) | Relational. The catalogue, collections and reviews are tabular data with strict rules that SQL constraints enforce. The friend network is only used one step deep (your friends' activity), so a graph database would add tutor approval and setup risk for little gain. | |
| Collection status | One status column, or separate owned/played/wanted flags | One status column, limited to owned, played or wanted. It matches the stakeholder wording ("change the status"), and changing the status updates the one record. | |
| Rating scale | 1 to 5, or 1 to 10 | Whole numbers from 1 to 10, checked by the API (422) and by a CHECK constraint in the database | |
| Friend model | Mutual friends, one-way follows, or both | Both, because the topic asks for both | |
| Login | Sessions or tokens; how to hash passwords | Tokens (JWT, for example with Flask-JWT-Extended). Passwords hashed with Werkzeug's built-in functions, which come with Flask. | |
| Page size | Any | 20 per page by default, maximum 100 | |
| Filter with no matches | 200 with an empty list, or an error | 200 with an empty list. An unknown id in the URL path is still 404. | |
| Deleting games | CASCADE, RESTRICT or soft delete | Block deletion while players have data about the game (ON DELETE RESTRICT; the API returns 409). Maintainers correct entries with PATCH instead. | |
| Visibility | Public, or friends only | Anyone can read collections, reviews and profiles; only the owner can change them. Record the privacy trade-off in D2. | |
| Priorities | MoSCoW (Must, Should, Could, Won't) | Agree the table in section 4. Priorities are set inside each slice, so every member keeps Must work. | |
| Enhancements | See section 7 | E1 automated tests (Should) and E2 OpenAPI docs (Could); E3 browser client (Could) only if time allows after CP4 | |
| Workload (proposed) | Keep, or rebalance | Guides and media links move from Victor to Lakhan, to even out the slices | |

### Assumptions (state and justify these in the D2 problem summary)
- The service is a web API. Bruno is the client; a browser frontend is an optional extension (E3).
- Games are not hosted or run, and no files are uploaded. Guides are text; screenshots and videos are https links.
- Genres and platforms are fixed lists loaded with the seed data. Maintainers can link games to them but can't add new ones.
- Maintainer accounts are created in the seed data. There is no way to become a maintainer through the API.
- Maintainers also moderate: they can remove forum posts, guides and links.
- A collection record belongs to a game, not to a platform, and holds one status at a time. So removing a platform from a game never touches player data.
- Each player has one review per game. Revising it replaces the old version; no review history is kept.
- Profiles show a bio and favourite games. Achievements are out of scope because they would need data from the game platforms themselves.
- The system runs at small community scale, so SQLite is adequate and performance is judged on seed-data volumes.

---

## 3. Stakeholders

| Stakeholder | Kind | Why they are interested | What they need from the system |
|---|---|---|---|
| Player | Direct user | Discover games and check which platforms they are on | Browse and filter by title, genre and platform; game details |
| Collector | Direct user (a player keeping lists) | Track games they own, have played or want to play | One record per game; changing the status updates it rather than duplicating it |
| Reviewer | Direct user (a player sharing opinions) | Share opinions and see what the community thinks | Rate on an agreed scale; revise their own review; nobody else can edit it |
| Social user | Direct user (a player connecting with others) | Connect with other gamers and see what they play | Friends, follows, an activity feed, forums, guides and links |
| Catalogue Maintainer | Direct user with extra permissions | Keep the catalogue accurate and the community content usable | Correct games and platform links without losing player data; remove bad posts and content |
| Gaming community (the client) | Indirect stakeholder | Wants a trustworthy, community-driven platform | Data integrity; players can't damage each other's data |
| Development and maintenance team | Indirect stakeholder | Has to run, test and extend the system | Reproducible setup, documented API, consistent errors |

### Competing needs
| Tension | Between | How the design resolves it |
|---|---|---|
| Correcting vs preserving | Maintainers want to fix or remove bad entries; collectors and reviewers want their records kept | Corrections through PATCH; ON DELETE RESTRICT; 409 when deleting a game players have used; change history |
| Openness vs ownership | Players want to see others' collections and reviews; owners want sole control | Anyone can GET; only the owner can PUT, PATCH or DELETE (403 otherwise) |
| Social visibility vs privacy | Social users want activity feeds; some players may not want their activity shared | The feed only shows people you're friends with or follow; a privacy setting is listed as a future improvement |
| Free expression vs quality | Posters want to write freely; the community wants useful forums | Only the author can edit; maintainers can remove posts and content |
| Nuance vs comparability | Reviewers want nuance; players want a score they can compare | A 1 to 10 score plus free text; the community rating is the average and count |

### How stakeholder needs shape the design
| Stakeholder | Endpoints | Tables |
|---|---|---|
| Every user | /auth/register, /auth/login, /users/{id}, /users/{id}/favourites/{game_id} | users, favourite_games |
| Player | GET /games, GET /games/{id}, GET /genres, GET /platforms | games, genres, game_genres, platforms, game_platforms |
| Collector | /users/{id}/collection | collection_entries |
| Reviewer | /games/{id}/reviews, /reviews/{id}, /games/{id}/rating | reviews |
| Social user | /friends/requests, /users/{id}/friends, /users/{id}/follow, /feed, /games/{id}/threads, /threads/{id}/posts, /games/{id}/content | friendships, follows, threads, posts, user_content |
| Catalogue Maintainer | POST /games, PATCH /games/{id}, /games/{id}/platforms/{platform_id}, DELETE /games/{id}, DELETE /posts/{id}, DELETE /content/{id} | games, game_genres, game_platforms, catalogue_changes |
| Development team | Every endpoint (shared error format) | The setup and seed script |

---

## 4. Requirements

### Functional requirements
| ID | Requirement | Task |
|---|---|---|
| FR1 | Browse and search games by title, genre and platform, one page at a time | 2 |
| FR2 | See a game's details: description, genres, platforms with their release dates, and average rating | 2 (using data from 3 and 5) |
| FR3 | Keep a personal collection of games marked owned, played or wanted, separate from the shared catalogue | 4 |
| FR4 | Changing a game's status updates the existing record and never creates a duplicate | 4 |
| FR5 | Rate a game on the agreed scale and write a review; revise your own review but never anyone else's | 5 |
| FR6 | Reviews from different players stay separate, even for the same game; the community rating is their average | 5 |
| FR7 | Maintainers add games and correct game details and platform links without erasing player collections or reviews | 3 |
| FR8 | Invalid ratings, unknown games and changes to another player's data are rejected, leaving existing data unchanged | All |
| FR9 | Register, log in, and keep a profile with a bio and favourite games | 1 |
| FR10 | Send and accept friend requests, follow players, and see a feed of their recent activity | 6 |
| FR11 | Discuss games in forum threads and posts | 7 |
| FR12 | Share guides, screenshot links and video links for games | 2 |
| Out of scope | See the Won't list under Priorities below | n/a |

### Data the system must store
- Users, their role (player or maintainer), profile and favourite games
- Games, their descriptive details, genres, platforms and the release date on each platform
- Each player's collection records and statuses
- Each player's reviews and ratings
- Friendships and follows
- Forum threads and posts; guides, screenshot links and video links
- A history of catalogue changes

### Non-functional and implicit requirements
| ID | Requirement | How we will evidence it |
|---|---|---|
| NFR1 | Security: passwords stored only as hashes; login needed for every change; maintainer-only routes checked | 401 and 403 tests; the users table shows only hashes |
| NFR2 | Integrity: rules enforced by the database as well as the API | CHECK, UNIQUE and foreign key constraints; bad data inserted directly with SQL is rejected |
| NFR3 | Consistency: a failed request never leaves a half-finished change | Transactions; before-and-after checks on sad paths |
| NFR4 | Predictable API: consistent JSON, status codes and error format | The same error body from every slice |
| NFR5 | Performance: lists are paginated and filtered columns are indexed | Response times on the seed data |
| NFR6 | Reproducibility: the system runs from the zip on a clean machine | The CP5 clean-machine test |
| NFR7 | Maintainability: one module per slice, commented code, reviewed changes | Repo structure and pull request history |
| NFR8 | Documentation: README, OpenAPI docs and an organised Bruno collection | Markers can run every request using the README alone |

### Priorities (MoSCoW)

MoSCoW comes from the DSDM Agile Project Framework (Agile Business Consortium, n.d.). It sorts every requirement into one of four groups:
- **Must:** the minimum usable subset. Without it, the system doesn't meet the problem statement or the brief. Never dropped.
- **Should:** important but not vital. The system still works without it.
- **Could:** wanted but less important. The first thing dropped if a deadline is at risk.
- **Won't (this time):** agreed not to be delivered in this project, with a reason.

Priorities are set inside each slice, not between slices, so every member has Must work and keeps their database, web service and testing evidence. Anyone who falls behind drops their own Could items first, then their Should items; a Must at risk gets help from the rest of the group. DSDM recommends keeping Musts to no more than about 60% of the effort, with about 20% in Coulds, so that dropping Coulds absorbs problems. Check this when agreeing the table at CP0.

| Priority | Requirement or feature | Covers | Owner | Why this priority |
|---|---|---|---|---|
| Must | Register and log in, with player and maintainer roles | FR9 | Ayaan | Players can only change their own records if the system knows who they are, and maintainers need a role |
| Must | Shared login and permission checks, and the shared error format | FR8, NFR1, NFR4 | Ayaan | Every "reject the request" story depends on them |
| Must | Browse and search games by title, genre and platform, in pages, with the genre and platform lists | FR1 | Lakhan (platform list: Haarisah) | Explicit in the problem statement and the Player story |
| Must | Game details: genres, platforms with release dates, average rating | FR2 | Lakhan | The problem statement lists title, genre, release information and platform availability |
| Must | Add games; correct details and genres; link and unlink platforms with release dates | FR7 | Haarisah | The Catalogue Maintainer story |
| Must | Block deleting a game that players have data about (409) | FR7, FR8 | Haarisah | The problem statement's "changes that would otherwise leave the system in an inconsistent state" |
| Must | Personal collections: add, change status, remove, list by status, one record per game | FR3, FR4 | Lee | The Collector story |
| Must | Write, revise and list reviews; ratings 1 to 10; one per player per game; average rating | FR5, FR6 | Zeeshan | The Reviewer and Player stories |
| Must | Friend requests: send, accept, decline; list friends | FR10 | Husray | The project is defined by user-based social features, and this is the core of the friend system |
| Must | Forum threads and posts: create and read | FR11 | Victor | The topic's core social interaction feature |
| Must | Invalid ratings, unknown games and changes to other players' data rejected, with data left unchanged | FR8 | Everyone | Explicit in the problem statement |
| Must | Hashed passwords, database constraints, transactions and consistent errors | NFR1 to NFR4 | Everyone | Without them, the "reject and leave unchanged" story isn't actually true |
| Must | Setup script, seed data, requirements.txt, README, Bruno collection and zip that work on a clean machine | NFR6, NFR8 | Victor | Required by the brief; a submission that won't run can't be marked |
| Must | Bruno happy-path and sad-path tests for every Must endpoint | All FRs | Everyone | The D3 demo and evaluation are judged on this evidence |
| Should | Profile with a bio, viewable by anyone and editable by its owner | FR9 | Ayaan | Part of the topic's customisable profiles; no problem statement story depends on it |
| Should | Change history of catalogue corrections | FR7 | Haarisah | Proves that corrections kept player data, but corrections work without it |
| Should | Deleting your own review | FR5 | Zeeshan | Revising a review already lets players change their opinion |
| Should | Removing a friend; following and unfollowing players | FR10 | Husray | Completes the friend system; requests alone already make it social |
| Should | Activity feed of friends' and followed players' collections and reviews | FR10 | Husray | The topic's "see their gaming activities"; the hardest query, so it carries the most risk |
| Should | Editing and deleting posts (authors edit; authors or maintainers delete) | FR11 | Victor | Forums work without editing; moderation matters but comes second |
| Should | Guides, screenshot links and video links | FR12 | Lakhan | The topic's user-generated content; not in the problem statement |
| Should | Indexes on filtered columns | NFR5 | Lakhan and Haarisah | The seed data is small enough without them, but they show database understanding |
| Should | One module per slice, commented code, reviewed pull requests | NFR7 | Everyone | Keeps the code explainable and contributions visible; the system runs without it |
| Should | Automated pytest tests (E1) | NFR3, NFR7 | Everyone (setup: Zeeshan) | Stronger, repeatable evidence; Bruno already covers the Must tests |
| Could | Favourite games on profiles | FR9 | Ayaan | Nice for profiles; no story depends on it |
| Could | OpenAPI (Swagger) documentation (E2) | NFR8 | Lakhan | An enhancement; the README and Bruno collection already document the API |
| Could | Browser client in JavaScript (E3), only after CP4 | None | Unassigned | An extension task in the D3 checklist; Bruno is enough as the client |

**Won't (this time)**

| Feature | Why not |
|---|---|
| Hosting or running games | Explicitly excluded by the problem statement |
| Uploading image or video files | File storage, size limits and security add risk; https links meet the need |
| Achievements | Would need data from the game platforms themselves |
| Notifications (e.g. "your request was accepted") | Needs its own table and delivery method; the friends list and feed show the same information |
| Privacy settings for collections and the feed | Listed as a future improvement (see the competing needs in section 3) |
| Custom lists beyond owned, played and wanted | The problem statement defines these three statuses |
| Review history and "helpful" votes on reviews | Not asked for; revising a review replaces it |
| Adding genres or platforms through the API | Fixed lists in the seed data keep the catalogue consistent |
| Password reset and email verification | Would need an email service |

Reference for D2: Agile Business Consortium (n.d.) *MoSCoW prioritisation*. Available at: https://www.agilebusiness.org/?p=1907 (Accessed: 3 November 2026).

---

## 5. Shared rules (everyone)

### Error contract (owned by Ayaan)
| Status | Meaning | Use it when | Example |
|---|---|---|---|
| 400 | Bad request | The request can't be read: not valid JSON, a value of the wrong type, or a required field missing | A review with no `rating` field; `page=abc` |
| 401 | Not logged in | No token, an invalid token, or a wrong password at login | A collection update with no token |
| 403 | Not allowed | Logged in, but not allowed to do this: changing someone else's data, a player on a maintainer-only route, or changing your own role | PUT on another player's collection; a player PATCHing a game |
| 404 | Not found | An id in the URL, or an id in the body, doesn't exist | A collection entry for game 99999 |
| 409 | Conflict | The change clashes with existing data | A second review of the same game; deleting a game that players have data about |
| 422 | Invalid value | The request was read, but a value breaks a rule: out of range, not an allowed option, empty, too long or the wrong format | Rating 11; status `borrowed`; an empty post |

Success codes: 200 for a read or update, 201 for a create (with the new resource in the body), 204 for a delete (no body).

Error body (identical everywhere):
`{"error": {"code": 422, "message": "rating must be between 1 and 10", "field": "rating"}}`

### Rules for every endpoint
Checks always run in this order, so every slice behaves the same way:
1. Not logged in: 401 (shared decorator).
2. A player on a maintainer-only route: 403 (shared decorator).
3. Unreadable request or a missing field: 400.
4. Invalid value: 422.
5. Unknown id: 404.
6. Not allowed, e.g. not the owner: 403.
7. Clash with existing data: 409.

Only then is the change written.

- Every write runs inside a transaction: if any step fails, nothing is saved.
- Every list uses the same shape: `{"items": [...], "page": 1, "size": 20, "total": 57}`.
- Bruno: each endpoint has one happy-path request and at least two sad-path requests. Each request checks the status code and the key fields of the response body. For a sad path on a write, a GET before and after shows that nothing changed.
- Every happy and sad example in the D2 endpoint inventory becomes a Bruno request with the same name, so the inventory turns straight into tests.
- Comment every route, validation step, error branch and database query. In the D3 code explanation, markers are pointed to these comments for detail.
- Work on one branch per task. Another member reviews each pull request before it is merged into main.
- Cite any AI-generated code in a comment with a sample prompt, and log it in AI_USAGE, as the brief requires.

### API design rationale (be ready to explain this in the meeting)
| Choice | Reason |
|---|---|
| Plural nouns, nested where data belongs to something (`/games/{id}/reviews`, `/users/{id}/collection`) | The URL shows what the data belongs to, so ownership checks and 404s follow naturally from the path |
| GET reads, POST creates, PUT creates or replaces at a known address, PATCH changes some fields, DELETE removes | Clients can predict what a request does from its method, and GET never changes data |
| PUT for collection entries, favourites and platform links | Idempotent: repeating the request leaves the same single record (201 the first time, 200 after), which is exactly the "no repeated entries" story |
| POST for reviews, with 409 for a second review | A review is a new resource with its own id, so a repeated POST is a clash rather than a silent overwrite |
| PATCH for corrections and profile edits | Only the changed fields are sent, so a correction can't accidentally blank other fields |
| Distinct success codes (200, 201, 204) | The status tells the client what happened without it having to read the body |
| One list shape and one error shape | Clients and Bruno tests handle every slice in the same way |

### Database rules
- **One shared schema.** All tables are SQLAlchemy models in one place, built by one script (`create_db.py`) that also loads the seed data. Each owner writes their own models; Victor merges them.
- **Foreign keys must be switched on.** SQLite ignores foreign keys unless `PRAGMA foreign_keys = ON` is run for every connection. Victor's database setup does this. Without it, every rule below silently does nothing.
- **Player data is protected.** Collections, reviews, favourites, threads and user content point at games with ON DELETE RESTRICT, so a game with player data can't be deleted.
- **Catalogue links go with their game.** game_genres and game_platforms use ON DELETE CASCADE, because they only describe the game.
- **History is kept.** catalogue_changes uses ON DELETE SET NULL, so its rows survive when a game is deleted.
- **Notation in the table specs:** PK = primary key; FK = foreign key; RESTRICT, CASCADE and SET NULL = what happens when the referenced row is deleted; CHECK, UNIQUE and NOT NULL = constraints.

### D2 row formats
Endpoint inventory (one row per endpoint; the FR and Priority columns link it to the requirements and the MoSCoW table):

| FR | Priority | Route | Method | Description | Users | Happy input | Happy output (status and body) | Sad inputs | Sad outputs (status and body) |
|---|---|---|---|---|---|---|---|---|---|

Database design (one row per table):

| FR | Table | Columns (type) | Keys and relationships | Constraints |
|---|---|---|---|---|

### Suggested data types
- Ids: INTEGER PRIMARY KEY.
- Text: VARCHAR with a length, e.g. title 200, username 30, review body 2000.
- release_date: DATE; timestamps: DATETIME.
- status, role and type: VARCHAR with a CHECK listing the allowed values; rating: INTEGER with a CHECK.

### Relationships and cardinality
| Relationship | Cardinality | Implemented by |
|---|---|---|
| User to collection entries | One to many | collection_entries.user_id |
| Game to collection entries | One to many | collection_entries.game_id |
| User to reviews; game to reviews | One to many each | reviews.user_id, reviews.game_id |
| Game to genres | Many to many | game_genres (link table) |
| Game to platforms | Many to many, with a release date | game_platforms (link table) |
| User to favourite games | Many to many | favourite_games (link table) |
| User to user (friends) | Many to many, self-referencing | friendships |
| User to user (follows) | Many to many, self-referencing | follows |
| Game to threads; thread to posts | One to many each | threads.game_id, posts.thread_id |
| User to threads, posts and user content | One to many each | threads.user_id, posts.user_id, user_content.user_id |
| Game to user content | One to many | user_content.game_id |
| Game to catalogue changes; maintainer to catalogue changes | One to many each | catalogue_changes.game_id, catalogue_changes.maintainer_id |

### Modelling alternatives considered
| Our choice | Alternative | Why we chose ours |
|---|---|---|
| Release date on game_platforms | Release date on games | Release dates differ by platform, so one date on games would be wrong for multi-platform games |
| One status column with a CHECK | Separate owned, played and wanted flags | Matches the stakeholder wording and guarantees one status per record; flags allow contradictory combinations unless extra CHECKs are added |
| Feed worked out from collections and reviews | A separate activities table | No duplicated data, so the feed can never disagree with the source records; the trade-off is a heavier query |
| Genres in their own table with a link table | Genre as a text column on games | Games have several genres, and a text column would break filtering and invite spelling variants |
| Each friendship stored once, smaller id first | Two rows per friendship (A to B and B to A) | One row can't fall out of step with a mirror row, and CHECK plus UNIQUE stop duplicates and self-friendship in the database itself |
| A role column on users | A separate maintainers table | Only two roles with no extra data, so one CHECK-limited column is simpler |
| A change history table | No history | Supports the maintainer story and proves that corrections kept player data |
| ON DELETE RESTRICT on player data | CASCADE | CASCADE would silently delete players' reviews and collections when a game is removed, which breaks the problem statement |

---

## 6. Tasks per person

Each task starts with a plain-English brief: the problem your slice solves, what you build, the rules it enforces, what you show in the demo and where to start. The spec underneath it is the detail you build and put into your D2 rows.

Everyone also writes their own D2 inventory and table rows (by Mon 9 Nov), adds Bruno requests and pytest tests as they build, and explains their own code in the video.

In each brief, items marked (Should) or (Could) are lower priority; everything else is a Must. Build your Musts first.

### Task 1: Users, auth and profiles
**Owner:** Ayaan Ahmed

> **The problem:** the system has to know who is making each request and what they're allowed to do. Without this, anyone could change anyone's collection or the catalogue.
>
> **What you build:**
> - Registration and login. Login returns a token, which Bruno sends with every later request.
> - Two kinds of user: player and maintainer.
> - A profile with a bio (Should) and favourite games (Could).
> - Two shared checks that every other slice uses: "is this person logged in?" and "is this their record, or are they a maintainer?"
> - The shared error helper, so every slice fails in the same way.
>
> **Rules your slice enforces:** emails and usernames are unique; passwords are never stored readable; you can only edit your own profile; nobody can change their own role through the API.
>
> **What you show in the demo:** logging in, editing your profile and adding a favourite; a request with no token refused (401); editing someone else's profile refused (403).
>
> **Start here:** (1) agree the login approach at CP0; (2) write the users model in week 9; (3) build register and login; (4) give everyone the shared checks and error helper by Fri 20 Nov, because every other slice's write endpoints need them.

**Tables**
- `users`: id PK, username UNIQUE NOT NULL, email UNIQUE NOT NULL, password_hash NOT NULL, role CHECK IN ('player','maintainer') default 'player', bio, created_at
- `favourite_games`: user_id FK, game_id FK RESTRICT, PK (user_id, game_id)

**Endpoints**

| Method | Route | Users | Priority | Notes |
|---|---|---|---|---|
| POST | /auth/register | Anyone | Must | 201 |
| POST | /auth/login | Anyone | Must | 200 with a token |
| GET | /users/{id} | Anyone | Should | Profile and favourites; never returns the email or password hash |
| PATCH | /users/{id} | Owner only | Should | Bio and username only; the role can't be changed |
| PUT | /users/{id}/favourites/{game_id} | Owner only | Could | 201 when added, 200 if already a favourite |
| DELETE | /users/{id}/favourites/{game_id} | Owner only | Could | 204 |

**Required sad paths and edge cases:** registration with no password (400); a password shorter than 8 characters (422); a duplicate email or username (409); a wrong password (401); editing another player's profile (403); trying to change your own role (403); favouriting an unknown game (404).

**Shared code:** `login_required` and owner-or-maintainer checks written as Flask decorators; the error helper. The Bruno login request saves the token in the Bruno environment so that every other request reuses it.

**D2 lead:** title page; the non-functional requirements part of the problem summary; the error contract.

**D3 lead:** the introduction, with every name and ID on one slide; the slide template.

**Done when:** any slice can protect a route by adding one decorator, and every error in the system uses the shared format.

### Task 2: Catalogue browse and user content
**Owner:** Lakhan

> **The problem:** players need to find games. The catalogue is the shared list of games everyone sees, and it has to be searchable by title, genre and platform. Players also want to share guides and links for each game.
>
> **What you build:**
> - The games and genres tables.
> - A list of games that can be filtered and comes back in pages.
> - A single game's details: description, genres, platforms with release dates, and average rating.
> - Guides, screenshot links and video links attached to games (Should; proposed: moved from Victor).
> - The OpenAPI documentation (E2, Could).
>
> **Rules your slice enforces:** anyone can read the catalogue, but nobody changes it through your endpoints (that's Haarisah's slice); an unknown game gives 404; lists come in pages; links must be https; only the author or a maintainer can delete content.
>
> **What you show in the demo:** searching by title, filtering by genre and platform, opening one game, and adding a guide.
>
> **Start here:** (1) write the games, genres and game_genres models in week 9, because every slice links to games; (2) agree the seed games with Victor; (3) build game details, then the list, then the filters, then user content.

**Tables**
- `games`: id PK, title NOT NULL, description, developer, publisher. Haarisah's maintainer endpoints write to this table.
- `genres`: id PK, name UNIQUE NOT NULL
- `game_genres`: game_id FK CASCADE, genre_id FK, PK (game_id, genre_id)
- `user_content` (proposed): id PK, user_id FK, game_id FK RESTRICT, type CHECK IN ('guide','screenshot','video'), title NOT NULL, body, url CHECK (url LIKE 'https://%'), created_at. A CHECK also makes sure a guide has a body and a screenshot or video has a url.

**Endpoints**

| Method | Route | Users | Priority | Notes |
|---|---|---|---|---|
| GET | /games?title=&genre=&platform=&page=&size= | Anyone | Must | Title matches part of the title, ignoring case; genre and platform take ids; filters combine |
| GET | /games/{id} | Anyone | Must | Genres, platforms with release dates, average rating |
| GET | /genres | Anyone | Must | |
| GET | /games/{id}/content | Anyone | Should | |
| POST | /games/{id}/content | Logged-in player | Should | 201 |
| DELETE | /content/{id} | Author or maintainer | Should | 204 |

**Required sad paths and edge cases:** an unknown game (404); `page=abc` (400); page 0 or size 500 (422); a filter with no matches (200 with an empty list); a link that isn't https (422); a content type of `meme` (422); deleting someone else's content (403).

**Uses other slices' data:** game details read platform links from Haarisah's table and reuse Zeeshan's average-rating function, so the rating is worked out in one place. Until their code is merged, read the seed data directly.

**E2:** an OpenAPI spec (`openapi.yaml`) written from the endpoint inventory and served as Swagger UI docs. It doesn't change how anyone else writes their code.

**D2 lead:** problem summary (explicit and implicit requirements); stakeholder table.

**D3 lead:** the project topic and requirements; video editing and timing.

**Done when:** filters combine correctly (e.g. RPGs on Switch), results come in pages, the filtered columns are indexed, and the content rules hold.

### Task 3: Catalogue maintenance
**Owner:** Haarisah

> **The problem:** catalogue information will sometimes be wrong: a typo in a description, a missing platform, a wrong release date. Maintainers have to fix it without destroying anything players have recorded about that game.
>
> **What you build:**
> - Platforms, and the link between a game and a platform, with a release date for each platform.
> - Maintainer-only endpoints to add a game, correct a game (including its genres), link or unlink platforms, and delete a game nobody has used.
> - A change history recording who changed what, and when (Should).
>
> **Rules your slice enforces:** only maintainers can change the catalogue; a game that players have collected, reviewed, favourited or discussed can't be deleted (409); every change is written to the history in the same transaction (Should).
>
> **What you show in the demo:** correcting a description and a release date, then showing that the game's reviews and collections are untouched and the change is in the history; a player refused when trying to edit (403).
>
> **Start here:** (1) write the platforms, game_platforms and catalogue_changes models in week 9; (2) build PATCH /games/{id} using Ayaan's maintainer check; (3) add the history write inside the same transaction.

**Tables**
- `platforms`: id PK, name UNIQUE NOT NULL
- `game_platforms`: game_id FK CASCADE, platform_id FK, release_date DATE, PK (game_id, platform_id)
- `catalogue_changes`: id PK, game_id FK SET NULL, maintainer_id FK, field, old_value, new_value, changed_at

**Endpoints**

| Method | Route | Users | Priority | Notes |
|---|---|---|---|---|
| POST | /games | Maintainer | Must | 201; the body includes `genre_ids` |
| PATCH | /games/{id} | Maintainer | Must | Any of title, description, developer, publisher, genre_ids |
| PUT | /games/{id}/platforms/{platform_id} | Maintainer | Must | Body: release_date. 201 when the link is new, 200 when it is updated |
| DELETE | /games/{id}/platforms/{platform_id} | Maintainer | Must | 204 |
| DELETE | /games/{id} | Maintainer | Must | 204, or 409 if players have data about the game |
| GET | /games/{id}/history | Anyone | Should | |
| GET | /platforms | Anyone | Must | |

**Required sad paths and edge cases:** a player attempting a PATCH (403); deleting a game that has reviews (409, with a GET of the reviews before and after); linking an unknown platform (404); a release date that isn't a real date (422); PATCHing an unknown game (404).

**Notes:**
- Repeating the same PUT on a platform link sets the same release date again and never creates a duplicate (idempotent).
- When a game is deleted, its genre and platform links go with it (CASCADE). Its history rows stay, with game_id set to NULL, after a final "deleted" entry that records the title.

**D2 lead:** database design table and physical ERD, combining everyone's table rows.

**D3 lead:** design vs implementation, using the change log (section 11).

**Done when:** every maintainer change appears in the history, and no correction or deletion can remove player data.

### Task 4: Personal collections
**Owner:** Lee

> **The problem:** each player keeps their own list of games they own, have played or want to play. These lists are separate from the shared catalogue, and only the player can change their own.
>
> **What you build:**
> - One record per player per game, holding a status.
> - Endpoints to add a game to your list, change its status and remove it.
> - A view of any player's list, which can be filtered by status.
>
> **Rules your slice enforces:**
> - Changing a status updates the existing record and never adds a second one.
> - You can't change another player's list (403).
> - An unknown game gives 404.
> - The status must be owned, played or wanted (422).
>
> **What you show in the demo:** adding a game as wanted, changing it to played, repeating the request, and showing there is still only one record; trying to change another player's list and being refused.
>
> **Start here:** (1) write the collection_entries model, with its one-record-per-player-per-game rule, in week 9; (2) build the PUT endpoint that creates or updates; (3) write the sad-path tests.

**Tables**

| FR | Table | Columns (type) | Keys and relationships | Constraints |
|---|---|---|---|---|
| FR3, FR4 | `collection_entries` | `id (INTEGER)`<br>`user_id (INTEGER)`<br>`game_id (INTEGER)`<br>`status (VARCHAR(6))`<br>`added_at (DATETIME)`<br>`updated_at (DATETIME)` | PK: `id`<br>FK: `user_id → users.id` with `ON DELETE RESTRICT`<br>FK: `game_id → games.id` with `ON DELETE RESTRICT`<br>One user and one game per entry. A user or game can have many entries. | `user_id` and `game_id` are `NOT NULL`.<br>`status NOT NULL` with `CHECK (status IN ('owned', 'played', 'wanted'))`.<br>`UNIQUE (user_id, game_id)` keeps one entry per player per game.<br>`added_at` and `updated_at` are `NOT NULL DEFAULT CURRENT_TIMESTAMP`. The API keeps `added_at` fixed and updates `updated_at` when the status changes. |

**Endpoints**

| FR | Priority | Route | Method | Description | Users | Happy input | Happy output (status and body) | Sad inputs | Sad outputs (status and body) |
|---|---|---|---|---|---|---|---|---|---|
| FR3, FR4, FR8 | Must | `/users/{id}/collection/{game_id}` | PUT | Adds a game to the player's collection or updates its status. Repeating the request keeps one entry. | Owner only | Signed in as player 7.<br>1. `PUT /users/7/collection/3` with `{"status":"wanted"}`<br>2. `PUT /users/7/collection/3` with `{"status":"played"}`<br>3. Repeat request 2 exactly. | 1. **201** `{"user_id":7,"game_id":3,"status":"wanted"}`<br>2. **200** `{"user_id":7,"game_id":3,"status":"played"}`<br>3. **200** `{"user_id":7,"game_id":3,"status":"played"}` (still one entry) | Signed in as player 7.<br>1. `PUT /users/7/collection/3` with `{"status":"borrowed"}`<br>2. `PUT /users/7/collection/99999` with `{"status":"wanted"}`<br>3. `PUT /users/2/collection/3` with `{"status":"wanted"}` | 1. **422** `{"error":{"code":422,"message":"status must be owned, played or wanted","field":"status"}}`<br>2. **404** `{"error":{"code":404,"message":"Game not found","field":"game_id"}}`<br>3. **403** `{"error":{"code":403,"message":"You can only change your own collection","field":null}}` (the collection is unchanged) |
| FR3 | Must | `/users/{id}/collection` | GET | Lists a player's collection, with an optional status filter and pagination. | Anyone | `GET /users/7/collection?status=wanted&page=1` (no token or request body) | **200** `{"items":[{"user_id":7,"game_id":3,"status":"wanted"}],"page":1,"size":20,"total":1}` | 1. `GET /users/7/collection?status=borrowed`<br>2. `GET /users/99999/collection` | 1. **422** `{"error":{"code":422,"message":"status must be owned, played or wanted","field":"status"}}`<br>2. **404** `{"error":{"code":404,"message":"User not found","field":"id"}}` |
| FR3, FR8 | Must | `/users/{id}/collection/{game_id}` | DELETE | Removes a game from the player's collection. | Owner only | `DELETE /users/7/collection/3` (signed in as player 7, no request body) | **204** (no response body) | 1. `DELETE /users/7/collection/3` while signed out<br>2. `DELETE /users/2/collection/3` while signed in as player 7 (the entry exists) | 1. **401** `{"error":{"code":401,"message":"Authentication required","field":null}}`<br>2. **403** `{"error":{"code":403,"message":"You can only change your own collection","field":null}}` (the collection is unchanged) |

**Required sad paths and edge cases:** a PUT on another player's collection (403, their list unchanged); an unknown game (404); status `borrowed` (422); filtering by `?status=borrowed` (422); repeating the same PUT (200, still one record).

**D2 lead:** Gantt chart; assembling the document; compiling the Harvard reference list; cover sheet; uploading the document.

**D3 lead:** uploading the video link and zip, after checking that staff can open and download the video.

**Done when:** the "no repeated entries" story can be shown with a row count before and after.

### Task 5: Reviews and ratings
**Owner:** Zeeshan

> **The problem:** players want to share opinions and see what the community thinks of a game. Each player gets one review per game, can change it later, and can't touch anyone else's.
>
> **What you build:**
> - The reviews table.
> - Endpoints to write and edit your own review, and to delete it (Should).
> - A list of all reviews for a game.
> - The game's average rating and number of reviews.
>
> **Rules your slice enforces:** ratings are whole numbers from 1 to 10; one review per player per game, so a second one gives 409; only the author can edit or delete a review (403).
>
> **What you show in the demo:** two players reviewing the same game, with both reviews shown separately and the average; a rating of 11 rejected.
>
> **Start here:** (1) write the reviews model with its rules in week 9; (2) build writing a review, then editing with the author check; (3) build the average-rating function, which Lakhan's game details also use.

**Tables**
- `reviews`: id PK, user_id FK, game_id FK RESTRICT, rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 10), body VARCHAR(2000), created_at, updated_at, UNIQUE (user_id, game_id)

**Endpoints**

| Method | Route | Users | Priority | Notes |
|---|---|---|---|---|
| POST | /games/{id}/reviews | Logged-in player | Must | 201 |
| GET | /games/{id}/reviews?page= | Anyone | Must | |
| GET | /games/{id}/rating | Anyone | Must | Average and count |
| PUT | /reviews/{id} | Author only | Must | Replaces the rating and text |
| DELETE | /reviews/{id} | Author only | Should | 204 |

**Required sad paths and edge cases:** no rating field (400); a rating of 11 or 0 (422); a second review of the same game (409); editing someone else's review (403, with a GET before and after); reviewing an unknown game (404).

**E1 setup:** in week 10, set up the shared pytest fixtures (a fresh test database and logged-in test users) that every slice's tests use, and check at CP4 that every slice has tests.

**D2 lead:** compiling the endpoint inventory from everyone's rows, and checking that every row matches the format and the error contract.

**D3 lead:** the demo script (section 14).

**Done when:** two players' reviews of one game show separately, and the average updates after a create, an edit and a delete.

### Task 6: Friends, following and activity feed
**Owner:** Husray

> **The problem:** the topic is a social platform: players want to connect with each other and see what their friends are playing and reviewing.
>
> **What you build:**
> - Friend requests: send, accept, decline, and list your friends.
> - Removing a friend (Should).
> - Following, which is one-way and needs no acceptance (Should).
> - A feed of recent collection changes and reviews from your friends and the people you follow (Should).
>
> **Rules your slice enforces:** you can't friend or follow yourself; two players can only have one friendship or request between them; only the person who received a request can accept or decline it.
>
> **What you show in the demo:** sending a request, accepting it as the other player, then seeing the friend's new review in the feed.
>
> **Start here:** (1) write the friendships and follows models in week 9 (they only need users); (2) build requests, accepting and following; (3) write the feed query last, against the seed data, while Lee and Zeeshan build their endpoints.
>
> **You also lead evaluation:** you own the traceability matrix and the full test run at CP4.

**Tables**
- `friendships`: id PK, user_low_id FK, user_high_id FK, requested_by FK, status CHECK IN ('pending','accepted'), created_at, CHECK (user_low_id < user_high_id), UNIQUE (user_low_id, user_high_id). Each pair is stored once with the smaller id first, so A-B and B-A can't both exist and nobody can friend themselves.
- `follows`: follower_id FK, followee_id FK, created_at, CHECK (follower_id <> followee_id), PK (follower_id, followee_id)

**Endpoints**

| Method | Route | Users | Priority | Notes |
|---|---|---|---|---|
| POST | /friends/requests | Logged-in player | Must | Body: user_id. 201 |
| PATCH | /friends/requests/{id} | The player who received it | Must | Body: accept or decline. Accept gives 200; decline deletes the request (204) |
| GET | /users/{id}/friends | Anyone | Must | Accepted friends only |
| DELETE | /friends/{user_id} | Either friend | Should | 204 |
| POST | /users/{id}/follow | Logged-in player | Should | 201 |
| DELETE | /users/{id}/follow | Logged-in player | Should | 204 |
| GET | /feed?page= | Logged-in player | Should | Newest first |

**Required sad paths and edge cases:** friending or following yourself (422); a second request between the same two players (409); accepting a request sent to someone else (403); a request to an unknown user (404); following someone twice (409).

**Feed:** a query over collection_entries and reviews, with no separate activity table, so it can never disagree with the source data. Someone who is both a friend and followed appears once. Justify this in D2.

**D2 lead:** evaluation ideas and the traceability matrix (each requirement mapped to its endpoints and tests).

**D3 lead:** the evaluation section.

**Done when:** the feed returns the right items, newest first, in pages, with no duplicates.

### Task 7: Forums and integration
**Owner:** Victor

> **The problem:** players want to discuss games with each other. The group also needs one person who makes the whole system fit together and run on any machine.
>
> **What you build:**
> - Discussion threads for each game, and posts in them; editing and deleting posts (Should).
> - The shared project setup: repo, app structure, database setup, seed data, requirements.txt, README, Bruno collection and final zip.
>
> **Rules your slice enforces:** only the author can edit a post; the author or a maintainer can delete it; empty posts are rejected.
>
> **What you show in the demo:** creating a thread and a post; another player refused when trying to edit it.
>
> **Start here:** (1) create the empty repo and add everyone today (CP0); (2) in week 9, set up the app skeleton and database setup, merge everyone's models and load the seed data by Fri 20 Nov; (3) then build the forums.

**Tables**
- `threads`: id PK, game_id FK RESTRICT, user_id FK, title NOT NULL, created_at
- `posts`: id PK, thread_id FK, user_id FK, body NOT NULL, created_at, updated_at

**Endpoints**

| Method | Route | Users | Priority | Notes |
|---|---|---|---|---|
| GET | /games/{id}/threads | Anyone | Must | |
| POST | /games/{id}/threads | Logged-in player | Must | 201 |
| GET | /threads/{id}/posts | Anyone | Must | |
| POST | /threads/{id}/posts | Logged-in player | Must | 201 |
| PATCH | /posts/{id} | Author only | Should | |
| DELETE | /posts/{id} | Author or maintainer | Should | 204 |

**Required sad paths and edge cases:** posting to an unknown thread (404); a post with no body field (400); an empty post (422); editing someone else's post (403).

**Integration work:**
- The repo, with one branch per task and reviewed pull requests into main.
- The Flask app structure, with one module (blueprint) per slice.
- The database setup, including `PRAGMA foreign_keys = ON` for every connection.
- `create_db.py`, which builds every table from the models and loads the seed data: at least 20 games, 6 genres, 5 platforms and 6 users (including 1 maintainer), plus enough collections, reviews, friendships and posts to demo every slice.
- requirements.txt with exact versions.
- Configuration, including the secret key the login tokens need, set up so that a clean machine works (e.g. a `.env.example` file to copy).
- The Bruno collection: one folder per slice, and an environment holding the base URL and token.
- The README and the final zip (section 15).

**D2 lead:** the AI usage section (section 16).

**D3 lead:** README, zip contents and AI_USAGE file; the resources and AI reflection in the video.

**Done when:** a fresh clone reaches a working, seeded API by following only the README.

---

## 7. Enhancements

The D3 bands above 65% need "meaningful enhancements" beyond the taught examples. The D3 checklist is just as clear that a modest system that is coherent, robust, well tested and well defended beats a bigger one. So these enhancements improve quality rather than adding features.

| ID | Enhancement | Priority | Owner | Why it adds marks |
|---|---|---|---|---|
| E1 | Automated pytest tests for each slice's key happy and sad paths, run against a fresh test database | Should | Each owner for their slice; Zeeshan sets up the shared fixtures | Repeatable test evidence for the evaluation, including things Bruno can't test, such as a failure halfway through a transaction |
| E2 | OpenAPI (Swagger) documentation for the whole API, written from the endpoint inventory | Could | Lakhan | A browsable service contract; comparing it with the final code feeds the design vs implementation reflection |
| E3 (only after CP4) | A small browser client in JavaScript for browsing, collections and reviews | Could (lowest) | Unassigned | The checklist treats a custom client as an extension task. It needs CORS switching on in the API. |

In the video, explain why each enhancement was chosen and what it costs. An enhancement nobody can defend adds nothing.

---

## 8. Timeline

### Key dates
- **Tue 3 Nov (week 7):** CP0, decisions agreed.
- **Mon 9 Nov:** everyone's own D2 inventory and table rows done.
- **Fri 13 Nov:** CP1, D2 document finished, checked and signed off.
- **Tue 17 Nov (week 9):** CP2, progress meeting and upload.
- **Fri 20 Nov:** all models merged and seeded (Victor); shared checks ready (Ayaan).
- **Fri 27 Nov:** CP3, the foundation works on a fresh clone.
- **Weeks 10 to 12:** all slices build in parallel.
- **Fri 11 Dec:** CP4, feature freeze.
- **Fri 18 Dec:** CP5, clean-machine test passed and code segments recorded.
- **Christmas break (21 Dec to 3 Jan):** video editing; CP6 on Thu 31 Dec.
- **January:** CP7, submission.
- **D1 quizzes (individual):** in the Tuesday labs on 10 Nov and 1 Dec. Missing the lab means 0 for that attempt.

### Gantt chart (per member, for the D2 document)

"Own rows" means your D2 endpoint inventory and table rows. "Break" is the Christmas break.

| Member | W7 (2 Nov) | W8 (9 Nov) | W9 (16 Nov) | W10 (23 Nov) | W11 (30 Nov) | W12 (7 Dec) | W13 (14 Dec) | Break | Jan |
|---|---|---|---|---|---|---|---|---|---|
| Ayaan | Decisions; NFRs; error contract; own rows | Title page; start login code early | Meeting; users model; register, login, shared checks (by Fri 20 Nov) | Profiles and favourites | Tests; help other slices | Integration; bug fixes | Introduction; record own segment | Slide deck | |
| Lakhan | Problem summary; stakeholders; own rows | Final problem summary; rehearsal | Meeting; games and genres models; game details | List, filters, pagination | User content; OpenAPI docs | Integration; tests | Clean-machine test (CP5); record | Video editing; check (CP6) | |
| Haarisah | ERD draft; own rows | Physical ERD; database table (CP1) | Meeting (tutor's notes); platform and history models | Add and correct games; history | Platform links; delete rules; sad paths | Integration; tests; compare models with ERD | Design vs implementation script; record | | |
| Lee | CP0; Gantt draft; own rows | Assemble D2; references; sign-off (CP1) | Meeting; upload (CP2); collections model | Collection PUT | List, filter, delete; sad paths | Integration; tests | Record | Video link check (CP6) | Submit (CP7) |
| Zeeshan | Inventory template; own rows | Compile inventory (CP1) | Meeting; reviews model | Write and edit reviews; test fixtures; CP3 check | Rating, delete; sad paths | Integration; demo script; compare routes with inventory | Demo rehearsal; record | | |
| Husray | Evaluation plan; own rows | Traceability matrix | Meeting; friendships and follows models | Requests, accept, follow | Feed query | Full test run (CP4) | Evaluation script; record | | |
| Victor | Empty repo (CP0); AI usage section; own rows | Final AI section | Meeting; app skeleton; merge models; seed data; Bruno setup (by Fri 20 Nov) | Threads and posts | Post rules; sad paths | Merge slices; requirements.txt; tidy Bruno | README; zip; record | AI_USAGE; final zip | |

Bruno requests and pytest tests are written in the same week as the endpoint they test.

### Dependencies
- All models are merged and seeded by Fri 20 Nov (Victor). After that, nobody waits for anyone else's tables, because every slice can read the seed data.
- Every write endpoint needs Ayaan's shared checks, also due Fri 20 Nov.
- Lakhan's game details reuse Zeeshan's rating function and read Haarisah's platform links; until those are merged, he reads the seed data directly.
- Husray's feed reads Lee's and Zeeshan's tables, so it can be written against the seed data from week 10.
- Zeeshan's test fixtures (week 10) come before the other slices write their pytest tests.
- The demo, recordings and clean-machine test need every slice merged by CP4.

### Running in parallel
- Weeks 7 and 8: the D2 sections are written in parallel, and each person writes their own rows.
- Weeks 10 to 12: all seven slices are built in parallel against the shared schema.

### If something runs late
- Anything more than a week behind its checkpoint is raised at the stand-up, and the owner pairs with one helper until it catches up.
- If the shared checks are late, other slices use a fixed test user and keep building.
- If the feed is late, ship friends and follows, show the feed with seed data, and record the gap in the evaluation.
- Follow the priorities in section 4: drop Could items first (E3, then E2, then favourites), then the late slice's own Should items. Must items and Bruno sad-path tests are never dropped; if a Must is at risk, someone with spare time helps.
- Record anything dropped in the change log (section 11), because the D3 reflection has to acknowledge missing functionality.
- The CP4 feature freeze is fixed: after it, only bug fixes.

**Weekly stand-up:** 15 minutes after each Tuesday 5387 lab, covering what's done, what's next and what's blocked.

---

## 9. Checkpoints

| CP | Date | Pass criteria | Checked by |
|---|---|---|---|
| CP0 | Tue 3 Nov | Section 2 decisions agreed, including the priorities in section 4; all student IDs collected; workload and enhancements confirmed; empty repo created with everyone added; Moodle D2 template checked | Lee |
| CP1 | Fri 13 Nov | D2 document follows the template; every endpoint row has happy and sad paths; every table has keys and constraints; physical ERD done; references compiled; under 4000 words; consistency check done (section 12); mock meeting held; all 7 signed off | Zeeshan (inventory), Haarisah (database), Lee (document) |
| CP2 | Tue 17 Nov | All 7 attend; each person explains their own slice; tutor feedback noted in the change log; document uploaded before 15:00 | Lee |
| CP3 | Fri 27 Nov | On a fresh clone: create and seed the database, start the server, and pass the Bruno register, login and browse requests | Zeeshan |
| CP4 | Fri 11 Dec | Feature freeze: every Must built; every Should built or dropped and recorded in the change log; the full Bruno collection and pytest suite pass; every sad path leaves data unchanged | Husray |
| CP5 | Fri 18 Dec | Clean-machine test using only the README, on a lab PC or another computer that has never run the project; every member's code segment recorded | Lakhan runs it; Victor fixes problems |
| CP6 | Thu 31 Dec | Video 20 minutes or less; all names and IDs on one slide; every member states their DB, web service and testing contribution; the link opens and downloads from a non-team account | Lakhan and Lee |
| CP7 | January, by 15:00 | Video link and zip submitted, ideally 1 to 2 days early; the video file isn't edited after submission | Lee |

### Definition of done (every endpoint)
- It matches its inventory row and uses the error contract.
- It has happy and sad Bruno requests that check the status and the response body.
- It has pytest tests for its key happy and sad paths.
- Its constraints are in the models.
- Its route, validation, errors and queries are commented.
- One other member has reviewed it before merging.

---

## 10. Evaluation plan (D2 evaluation ideas, completed in D3)

The chain is: requirement → test → expected result → evidence. In D3, add **Actual result** and **Verdict** (met, partly met, not met) columns, and discuss failed tests and limitations honestly. "The program runs" is not evidence that a requirement is met. Report the verdicts by priority: every Must should be met, and any Should or Could that was dropped or only partly met is discussed as a limitation.

| ID | Requirement | Test | Expected result | Evidence |
|---|---|---|---|---|
| FR1 | Browse by title, genre and platform | Bruno filter requests; pytest filter tests | 200; only matching games; correct page details | Bruno results; response bodies |
| FR2 | Game details | GET a seeded multi-platform game | Its genres, each platform with its own release date, and the same average as /rating | Response compared with the seed data |
| FR3, FR4 | One record per game | PUT `wanted` (new), PUT `played` (change), PUT `played` again (repeat) | 201, 200, 200; one record throughout | Bruno; row count before and after |
| FR5 | Rating scale and ownership | POST a rating of 11; PUT another player's review | 422; 403; review unchanged | GET before and after |
| FR6 | Separate reviews | Two players review the same game | Two separate reviews; correct average | Bruno; database view |
| FR7 | Corrections keep player data | PATCH a description and PUT a new release date, then GET the game's reviews and collections | 200; player data unchanged; history entries added | GET before and after; history |
| FR8 | Invalid changes rejected | Unknown game; another player's list; deleting a game with reviews | 404; 403; 409; database unchanged | GET before and after |
| FR9 | Accounts and profiles | Register; log in; edit own profile; edit another's | 201; 200 with a token; 200; 403 | Bruno |
| FR10 | Friends and feed | Request, accept, friend reviews a game, GET /feed | The review appears once, newest first | Bruno |
| FR11 | Forums | Create a thread and post; edit someone else's post | 201; 403 | Bruno |
| FR12 | Guides and links | Add a guide; add an http (not https) link | 201; 422 | Bruno |
| NFR1 | Security | No token; a player on a maintainer route; inspect the users table | 401; 403; only hashes stored | Bruno; database screenshot |
| NFR2 | Integrity in the database | With foreign keys on, insert a rating of 11 and a review for a missing game directly with SQL | The database rejects both, even with the API bypassed | SQL error messages |
| NFR3 | No half-finished changes | A pytest test that forces the history write to fail during a correction | The game is unchanged | pytest output |
| NFR4 | Predictable API | One sad-path request per slice | The same error body shape from every slice | Bruno |
| NFR5 | Performance | Time filtered GET /games requests on the seed data | Under 200 ms locally (our own target) | Bruno timings |
| NFR6 | Reproducibility | Clean-machine install using only the README | Running API; Bruno collection passes | CP5 record |
| NFR7 | Maintainability | Review the repo at CP4 | One module per slice; commented code; every change merged through a reviewed pull request | Repo and pull request history |
| NFR8 | Documentation and client | Run the whole Bruno collection in order with the Bruno runner; open the OpenAPI docs | Every request passes in one run; the docs match the endpoints | Bruno run report |

---

## 11. Design change log (feeds the D3 design vs implementation section)

Haarisah owns this log. Everyone adds a row whenever their implementation differs from the D2 design, including any Should or Could item that was dropped. The first rows are the tutor's feedback from the D2 meeting. In week 12, Haarisah compares the final models with the D2 ERD and Zeeshan compares the final routes with the D2 inventory; every difference goes in the log.

| Date | Requirement | D2 design | Final implementation | Reason and evidence | Consequence and lesson | Logged by |
|---|---|---|---|---|---|---|
| | | | | | | |

---

## 12. D2 submission checklist

**Required by the brief**
- [ ] Follows the Moodle D2 template
- [ ] Title page with every name and student ID
- [ ] Problem summary with explicit and implicit requirements (2%)
- [ ] Stakeholder table: kind of stakeholder and why they are interested (2%)
- [ ] Gantt chart covering all members, with a timeline per subtask and its dependencies (4%)
- [ ] Endpoint inventory: route, description, method, happy and sad inputs and outputs, users involved (5%)
- [ ] Database design: table summary and physical ERD (5%)
- [ ] Evaluation ideas (2%)
- [ ] MoSCoW priority table (requested by our tutor), with a reason for each priority and a Won't list
- [ ] AI usage section
- [ ] Each member's contribution is clear (reuse the responsibilities table in section 1), and all 7 have signed off
- [ ] Under 4000 words of body text (+10% tolerance, beyond which nothing is marked; tables, ERD, Gantt and code don't count); font size 11; 1.5 line spacing; cover sheet; BCU Harvard references

**What lifts each section to 4 or 5 stars (from the D2 checklist)**
- [ ] Assumptions stated and justified; the data to be stored identified
- [ ] Competing stakeholder needs discussed and linked to endpoints and tables
- [ ] The Gantt shows client (Bruno) work, testing, submission preparation, parallel work and the "if something runs late" plan, which follows the MoSCoW priorities
- [ ] The inventory shows success codes as well as errors, and the API design rationale explains the choices
- [ ] The database design shows keys, data types, cardinality and the alternatives considered
- [ ] The evaluation plan follows requirement → test → expected result → evidence
- [ ] Consistency check: every FR has an endpoint, every endpoint's data has a table, every table serves an FR, and names match everywhere

**The meeting**
- [ ] Mock meeting held: each member explains their slice and one design decision, with the alternative they rejected
- [ ] Rehearsed to 15 minutes or less

---

## 13. D3 video plan (20 minutes maximum)

| Section | Marks | Time | Lead |
|---|---|---|---|
| Introduction: each person gives their name, student ID and their contribution to DB, web service and testing; all names and IDs together on one slide | Compulsory | 2:00 | Ayaan |
| The project topic and the requirements we identified | Required content | 0:45 | Lakhan |
| Live demo (section 14) | 30% | 6:30 | Zeeshan; each owner narrates their own step |
| Code explanation: each person traces one request through their slice (client → route → validation → database → response), ideally the same request they showed in the demo, using terms such as ORM, idempotent and foreign key; plus how the Bruno collection is organised | 10% | 4:30 | Everyone |
| Design vs implementation, from the change log: what changed, why, the consequence and the lesson | 10% | 2:00 | Haarisah |
| Evaluation: how we tested, the results table against the FRs and NFRs (by priority), failed tests, dropped items, limitations and improvements | 10% | 2:00 | Husray |
| Resources and AI reflection | Required content | 0:30 | Victor |

Total: 18:15, which leaves a margin under the 20-minute limit.

---

## 14. Demo script

Before starting, open the database in DB Browser for SQLite next to Bruno, so every change can be shown landing in the database. Say which requirement each step proves. If a Should or Could item was dropped, skip that part of the step and cover it in the evaluation instead.

1. Register and log in; edit your profile and add a favourite game (FR9). Show a 401 with no token, and a 403 when editing someone else's profile.
2. Browse games by title, genre and platform, then open one game's details (FR1, FR2). Show in the database that its genres and platforms come from the link tables.
3. Add a game to your collection as `wanted`, change it to `played`, then repeat the same request. Show there is still one record (FR3, FR4).
4. Try to change another player's collection. Show the 403 and that their list is unchanged (FR8).
5. Two players review the same game. Show both reviews and the average rating (FR5, FR6).
6. Submit a rating of 11 (422), review an unknown game (404), and post a second review (409). Show the data is unchanged each time (FR5, FR8).
7. Edit someone else's review (403), then edit your own (200) (FR5).
8. As a maintainer, correct a description and a release date. Show the collections and reviews are intact, and the change history (FR7).
9. As a player, attempt a maintainer edit (403). As a maintainer, try to delete a game that has reviews (409) (FR7, FR8).
10. Send a friend request, accept it as the other player, follow someone, and view the feed (FR10).
11. Create a forum thread and a post; add a guide and a screenshot link. Show a 403 when editing someone else's post (FR11, FR12).

---

## 15. README, zip and pre-recording checks

**The README (in the root of the zip) covers:**
- [ ] Design mapping (which part of the design is in which code) and the finalised designs
- [ ] The Python version
- [ ] Dependencies, with a separate requirements.txt
- [ ] Installing on a clean machine
- [ ] Configuration, including how the secret key is set
- [ ] Creating and seeding the database (`create_db.py`)
- [ ] Starting the application
- [ ] Running the Bruno collection, including the environment and token
- [ ] Running the pytest tests
- [ ] Test accounts from the seed data, with usernames and passwords (including the maintainer)
- [ ] Opening the OpenAPI docs
- [ ] Using the browser client, if E3 is built

**The zip contains:**
- [ ] Backend source code
- [ ] The Bruno collection
- [ ] The database setup and seed script
- [ ] The tests
- [ ] Configuration files needed to run (e.g. `.env.example`)
- [ ] The AI_USAGE file
- [ ] Anything else needed to reproduce the system on a clean machine, and nothing that isn't (no virtual environment folder or cache files)

**Before recording (from the D3 checklist):**
- [ ] The backend runs reliably and is visibly using our database
- [ ] The important create, read, update and delete operations, happy paths and sad paths can all be shown
- [ ] Each person can trace a request from the client to the database and back, explaining the code rather than reading it out
- [ ] We can say where the D2 design changed and why
- [ ] We can show test evidence and link each test to a requirement
- [ ] We can name the system's weaknesses and limitations
- [ ] We can defend why this approach suits the problem
- [ ] Every member can explain their own contribution to DB, web service and testing
- [ ] The zip reproduces the system on a clean machine

---

## 16. AI usage log (D2 section and D3 AI_USAGE file)

| Tool | Date | Purpose | Representative prompt | Output used or modified | How we verified it | Important changes made |
|---|---|---|---|---|---|---|
| | | | | | | |

Log every use as it happens, including help with planning. Rebuilding this log from memory in January won't work.

---

## 17. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Shared checks or schema late | Blocks every other slice | Both due Fri 20 Nov; other slices use a fixed test user and seed data meanwhile |
| D2 is two weeks away | Weak design marks | Own rows by Mon 9 Nov; document finished by CP1 (Fri 13 Nov); assembly owned by Lee |
| Feed blocked by collections and reviews | Late feature | Build friends and follows first; write the feed query against seed data |
| Slices drift apart | Integration breaks | One shared schema merged by Victor; schema changes only through reviewed pull requests, noted in the change log |
| SQLite foreign keys left off | Delete rules silently fail | `PRAGMA foreign_keys = ON` in Victor's setup; the NFR2 test proves it |
| Secret key or config missing on a clean machine | The app won't start for markers | `.env.example` in the zip; config steps in the README; tested at CP5 |
| Members away over Christmas | Video not recorded | Every code segment recorded by CP5 (Fri 18 Dec) |
| Clean-machine setup fails | Reproducibility marks lost | CP3 and CP5 run by Zeeshan and Lakhan, not Victor, who wrote the setup |
| A member contributes little | Individual mark adjusted | Vertical slices, a visible pull request history, and each person's own segment in the meeting and video |
| Video link not accessible to staff | The whole group fails | Test the link from a non-team account; never edit the file after submitting |
| Scope too large to defend | Lower marks despite more code | The checklists reward coherent, tested, well-defended systems; drop Coulds first (section 4) and polish the Musts before adding anything |

---

## 18. Glossary

| Term | Meaning |
|---|---|
| Slice | One person's feature from top to bottom: its tables, endpoints and tests |
| MoSCoW | A way of prioritising requirements: Must have, Should have, Could have, Won't have this time (section 4) |
| FR, NFR | Functional requirement (what the system does); non-functional requirement (how well it does it: security, speed, reliability) |
| CP | Checkpoint: a date when the group checks something is genuinely done (section 9) |
| D2 lead, D3 lead | The person who writes that part of the D2 document, or scripts that part of the video |
| Stakeholder | Anyone with an interest in the system, whether or not they use it directly |
| Traceability matrix | A table linking each requirement to its endpoints and its tests |
| Endpoint | One URL plus method that the API answers, e.g. `GET /games/{id}` |
| GET, POST, PUT, PATCH, DELETE | The HTTP methods: read; create; create or replace at a known address; change some fields; remove |
| JSON | The text format the API uses for request and response bodies |
| Happy path, sad path | A correct request and its successful response; a wrong request (bad data, wrong user, unknown id) and the error it should get |
| Bruno | The API client we use to send requests and save them as tests. An environment stores shared values such as the token; the runner runs a whole collection in one go |
| Token, JWT | A string returned at login that the client sends with every request to prove who it is; JWT is a common token format |
| Hashing | Turning a password into a scrambled value that can be checked but not reversed |
| Player, maintainer, owner | A normal user; a user allowed to edit the catalogue and remove content; the player a record belongs to |
| Status codes | 200 OK, 201 created, 204 deleted; 400 to 422 are errors (section 5) |
| Idempotent | Sending the same request twice has the same effect as sending it once |
| Upsert | One request that creates a record if it doesn't exist, or updates it if it does |
| Pagination | Returning a list in pages (e.g. 20 at a time) instead of everything at once |
| Constraint | A rule the database itself enforces, so bad data can't get in even if the API has a bug |
| PK (primary key) | The column, or columns, that uniquely identify a row |
| FK (foreign key) | A column that must point to a real row in another table, e.g. a review's `game_id` |
| UNIQUE, CHECK, NOT NULL | No two rows share the value(s); a value must pass a test; a value must be present |
| ON DELETE RESTRICT, CASCADE, SET NULL | When a referenced row is deleted: refuse the delete; delete these rows too; keep these rows but empty the reference |
| PRAGMA foreign_keys = ON | The SQLite setting that makes it enforce foreign keys; it's off by default |
| Transaction | A group of changes that either all happen or none do |
| Link table | A table that connects two others in a many-to-many relationship, e.g. game_genres (also called a junction table) |
| Self-referencing relationship | A relationship between rows of the same table, e.g. one user following another |
| Cardinality | How many rows on each side of a relationship can be linked: one to one, one to many, many to many |
| ERD, physical ERD | Entity relationship diagram; the physical version shows the real tables, columns, types and keys |
| ORM, SQLAlchemy model | Object relational mapping: Python classes that map to database tables, so code reads and writes rows as objects. A model is one such class |
| Flask, blueprint, decorator | Flask is the Python web framework; a blueprint is one module of routes (one per slice); a decorator is a line above a function that adds behaviour, such as `@login_required` |
| Aggregate | A value calculated from many rows, such as an average rating |
| Seed data | Sample games, users and records loaded into the database for testing and the demo |
| pytest, fixture | A Python testing tool; a fixture is shared setup for tests, such as a fresh test database |
| OpenAPI, Swagger UI | A standard format describing an API's endpoints; Swagger UI shows it as interactive documentation |
| CORS | A browser security rule; the API has to allow it before a web page on another address can call it |
| Repo, branch, pull request, main | The shared Git project; a separate line of work; a request to merge that work, reviewed by a teammate; the shared, working version |
| Feature freeze | The point after which no new features are added, only fixes |
| Clean machine | A computer that has never run the project, used to prove the README steps work |
| Change log | Our record of where the final system differs from the D2 design, and why |
