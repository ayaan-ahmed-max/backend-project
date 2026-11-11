# D2 to-do lists for the team

**Written:** Wed 11 Nov 2026, from the group plan (`CMP5387_Group_Plan.md`, the revised version on GitHub) and the D2 progress tracker.

**Why this matters:** the brief says each group member must demonstrate how they contributed to the design, and must sign off the documentation. The group is marked mainly on the meeting, and the document is supplementary. So everyone writes their own rows, makes at least one design decision of their own, and has to be able to explain their slice out loud on Tue 17 Nov. Nobody else can do that part for you.

## The three dates

| Date | What is due |
|---|---|
| **Thu 12 Nov, 12:00** | Your own endpoint rows sent to Zeeshan, and your own table rows sent to Haarisah |
| **Fri 13 Nov** | Your D2 lead sections sent to Lee; consistency check and sign-off by all 7 |
| **Tue 17 Nov** | Progress meeting (all 7 must attend) and upload before 15:00 |

## Terms used in this file

| Term | Plain English |
|---|---|
| D2 | The design document and progress meeting due on Tue 17 Nov |
| Slice | Your feature from top to bottom: its tables, its endpoints and its tests |
| Endpoint | One URL plus method that the API answers, e.g. `GET /games/{id}` |
| Method (GET, POST, PUT, PATCH, DELETE) | The kind of request: read; create; create or replace at a known address; change some fields; remove |
| Status code | The number the API sends back to say what happened: 200 OK, 201 created, 204 deleted with nothing to return, and 400 to 422 for errors |
| JSON | The text format used for request and response bodies, e.g. `{"rating": 8}` |
| Happy path | A correct request and the successful response it gets |
| Sad path | A wrong request (bad data, wrong user, unknown id) and the error it gets |
| Error body | The one shape every error uses: `{"error": {"code": 422, "message": "...", "field": "..."}}` |
| Rows | Your lines in the two shared D2 tables: one row per endpoint in the endpoint inventory, and one row per table in the database design |
| Constraint | A rule the database itself enforces, so bad data can't get in even if the API has a bug |
| Primary key (PK) | The column, or columns, that uniquely identify a row in a table |
| Foreign key (FK) | A column that must point to a real row in another table, e.g. a review's `game_id` |
| ON DELETE RESTRICT / CASCADE | What happens to your rows when the row they point to is deleted: the delete is refused / your rows are deleted too |
| CHECK, UNIQUE, NOT NULL | Constraints: a value must pass a test; no two rows share the value; a value must be present |
| Index | A lookup structure the database keeps on a column so that searches on it are fast, like the index of a book |
| Idempotent | Sending the same request twice has the same effect as sending it once |
| ERD | Entity relationship diagram: the picture of the tables and how they link. The physical version shows real columns, types and keys |
| MoSCoW | Our priorities: Must, Should, Could, Won't |
| Bruno | The API client we use to send requests and save them as tests |
| Seed data | Sample games, users and records loaded into the database for testing and the demo |
| Traceability matrix | A table linking each requirement to its endpoints and its tests |
| Gantt chart | A chart with tasks as rows and dated bars showing when each one happens |
| F1 to F17 | The design fixes listed in section 3 of the progress tracker |
| FAQ | The module's assessment FAQ, which sets out the structure the D2 document should follow |

## How to write your rows (copy this format)

Use `ayaan_d2_rows.md` as the worked example. Copy its layout exactly and replace the content with your own slice.

**Endpoint rows** (one per endpoint in your task's table in plan section 6):
- Columns: FR, Priority, Route, Method, Description, Users, Happy input, Happy output (status and body), Sad inputs, Sad outputs (status and body).
- Happy input and output are realistic example JSON, with the success code (200, 201 or 204).
- At least two sad paths per endpoint, and every "required sad path" listed for your task in the plan. Each one shows its status code and the full error body.
- Number the sad inputs and outputs so each input lines up with its output, and keep them in the plan's order of checks: 401, then 400, then 422, then 404, then 403, then 409.
- Any field your endpoint doesn't accept gives 400 ('unknown field: <name>'). Add one sad path for it on each endpoint that takes a body.

**Table rows** (one per table you own):
- Columns: FR, Table, Columns (type), Keys and relationships, Constraints.
- Every column gets a data type from the plan's "Suggested data types" (plan section 5).
- SQLite does not enforce the length in `VARCHAR(30)`. If a length limit matters, add a CHECK for it, as the users row does.

---

## Lakhan

**Slice:** Task 2, catalogue browse and user content. **D2 lead:** problem summary, stakeholder table, MoSCoW priority table.

- [ ] **By Thu 12 Nov at 12:00:** own rows. 6 endpoint rows to Zeeshan; 4 table rows (games, genres, game_genres, user_content) to Haarisah.
- [ ] **By Thu 12 Nov at 12:00, fix F11:** SQLite (our database) ignores the length in a type such as `VARCHAR(200)`, so it would store a longer value without complaint. For every column in your tables that has a length limit, add a CHECK, e.g. `CHECK (length(title) <= 200)`, so the database enforces the limit as well as the API's 422.
- [ ] **By Thu 12 Nov at 12:00, fix F12:** a CHECK passes when the value is missing (NULL). Any role, status or type column that has a CHECK must also be NOT NULL, or a row with no value could be inserted directly with SQL. In your slice this is `user_content.type`.
- [ ] **By Fri 13 Nov:** problem summary, written in prose and not only as a table. Mark each requirement as explicit (stated in the problem statement) or implicit (worked out by us), list the data to be stored, and justify each assumption. Ayaan supplies the non-functional requirements part (`ayaan_d2_sections.md`).
- [ ] **By Fri 13 Nov:** stakeholder table, including the competing needs and the table mapping each stakeholder to endpoints and tables.
- [ ] **By Fri 13 Nov:** MoSCoW priority table with a reason for each priority, and the Won't list.
- [ ] **By Fri 13 Nov, fix F9:** `GET /games/{id}/history` (the list of corrections made to a game) is not linked to any stakeholder in the mapping table. Add it to a stakeholder, so that every endpoint traces back to someone's need.
- [ ] **Design contribution:** make and record at least one design decision of your own on your slice: the alternatives you considered, the one you chose, and why. Write it under your rows. For example, how the title search matches (whole title or part of it, case sensitive or not), or whether the genre filter takes an id or a name.
- [ ] **Be ready to answer:**
  - "Why are genres a separate table with a link table, and not just a text column on games?"
  - "What comes back if the game doesn't exist, if `page=abc`, or if a filter matches nothing, and why are those three different?"

## Haarisah

**Slice:** Task 3, catalogue maintenance. **D2 lead:** database design table and physical ERD.

- [ ] **By Thu 12 Nov at 12:00:** own rows. 7 endpoint rows to Zeeshan; 3 table rows (platforms, game_platforms, catalogue_changes) for your own merged table.
- [ ] **By Thu 12 Nov at 12:00, fix F11:** SQLite (our database) ignores the length in a type such as `VARCHAR(200)`, so it would store a longer value without complaint. For every column in your tables that has a length limit, add a CHECK, e.g. `CHECK (length(name) <= 50)`, so the database enforces the limit as well as the API's 422.
- [ ] **By Thu 12 Nov at 12:00, fix F12:** a CHECK passes when the value is missing (NULL). Any role, status or type column that has a CHECK must also be NOT NULL, or a row with no value could be inserted directly with SQL. Your own tables have no such column, but check it in everyone's rows when you merge them, along with the length CHECKs from F11.
- [ ] **By Thu 12 Nov at 12:00, fix F13:** with Victor, decide what happens to a thread's posts when the thread is deleted (fix F4 makes threads deletable). The foreign key `posts.thread_id` is either ON DELETE CASCADE (the posts are deleted with the thread) or ON DELETE RESTRICT (a thread can't be deleted while it has posts). Record the options and the reason; this counts as a design decision of your own.
- [ ] **By Fri 13 Nov:** merged database design table from everyone's table rows, with a data type on every column, the keys, and the ON DELETE behaviour of every foreign key. Include the cardinality table and the "alternatives considered" table from plan section 5.
- [ ] **By Fri 13 Nov:** physical ERD that matches the merged table exactly (same table names, column names and types).
- [ ] **By Fri 13 Nov, fix F5:** correct the index list. Index `game_genres(genre_id)`, `game_platforms(platform_id)` and `reviews(game_id)`, and do not index `games.title`. The title search looks for text anywhere inside the title, which an index can't speed up, so a marker asking "what does this index do?" would expose it.
- [ ] **By Fri 13 Nov, fix F7:** state in the database design that users are never deleted through the API, so the foreign keys pointing at users use the default (RESTRICT). Every foreign key needs a stated delete behaviour. Threads are now deletable, so their rule is decided in fix F13.
- [ ] **By Fri 13 Nov, fix F14:** write a short paragraph for D2 justifying SQLite. The FAQ recommends MariaDB or MySQL, and any other database must be justified in D2 and approved by the tutor before D2 is submitted. Ayaan is emailing the tutor for the written approval; the reasons are in plan section 2 (no database server to install, which keeps the clean-machine test simple).
- [ ] **By Fri 13 Nov, fix F15:** present the database design as three ERDs, each with a short discussion: conceptual (the things the system stores and how they relate, with no columns), logical (adds the attributes and keys, without database-specific types) and physical (the real tables, columns, types and constraints). Ask everyone to check their own tables in each one.
- [ ] **By Thu 12 Nov at 12:00, fix F8:** in your `DELETE /games/{id}` row, the list of player data that blocks deletion must include user content (guides and links), to match the database rules in plan section 5.
- [ ] **Design contribution:** make and record at least one design decision of your own on your slice, with the alternative you rejected and why. For example, what the change history stores when a game's genres change, or what a repeated platform link request does to the release date.
- [ ] **Be ready to answer:**
  - "What would happen if the foreign key from collections or reviews to games used CASCADE, or were removed altogether?"
  - "Why is the release date on game_platforms and not on games, and why is linking a platform a PUT?"

## Lee

**Slice:** Task 4, personal collections. **D2 lead:** Gantt chart, assembling the document, references, cover sheet, upload.

- [ ] **Now:** collect all 7 student IDs and full names. Without names and IDs the submission gets no marks.
- [ ] **By Thu 12 Nov at 12:00:** own rows. 3 endpoint rows to Zeeshan; 1 table row (collection_entries) to Haarisah.
- [ ] **By Thu 12 Nov at 12:00, fix F11:** SQLite (our database) ignores the length in a type such as `VARCHAR(200)`, so it would store a longer value without complaint. For every column in your tables that has a length limit, add a CHECK, e.g. `CHECK (length(username) BETWEEN 1 AND 30)` on the users table, so the database enforces the limit as well as the API's 422.
- [ ] **By Thu 12 Nov at 12:00, fix F12:** a CHECK passes when the value is missing (NULL). Any role, status or type column that has a CHECK must also be NOT NULL, or a row with no value could be inserted directly with SQL. In your slice this is `collection_entries.status`, which the plan already marks NOT NULL.
- [ ] **By Fri 13 Nov:** Gantt chart as a real chart (tasks as rows, dated bars), not a weekly table. It needs all 7 members named, the checkpoints CP0 to CP7 marked, dependencies drawn, the parallel work in weeks 10 to 12 visible, Bruno and testing work shown, and the "if something runs late" plan.
- [ ] **By Fri 13 Nov:** document assembled from everyone's sections as a Word or PDF file, in the FAQ's structure: title page; requirements analysis (problem summary, stakeholders, requirements, with a table of how each requirement will be evaluated); entity relationship modelling (conceptual, logical and physical ERDs with discussion); API design (protocol and resource URI inventory); timeline and workload planning (subtasks, dependencies, Gantt); MoSCoW; AI usage. No Moodle template exists. Body text under 4000 words (tables, ERD, Gantt and code don't count), font size 11, 1.5 line spacing, cover sheet, BCU Harvard references.
- [ ] **By Fri 13 Nov, fix F17:** the contribution table must show every member contributing to database design, API design and planning, not one person per area. Each of us designed our own tables and endpoints and led a planning section, so list all three for everyone.
- [ ] **Fri 13 Nov:** run the consistency check (tracker section 5) and collect sign-off from all 7.
- [ ] **Tue 17 Nov:** upload before 15:00.
- [ ] **Design contribution:** make and record at least one design decision of your own on your slice, with the alternative you rejected and why. For example, what the PUT returns in its body, or whether `added_at` changes when the status changes.
- [ ] **Be ready to answer:**
  - "Why is adding a game to a collection a PUT and not a POST, and why does it return 201 the first time and 200 after?"
  - "If the API had a bug, what in the database would still stop two records for the same player and game?"

## Zeeshan

**Slice:** Task 5, reviews and ratings. **D2 lead:** endpoint inventory (compiling everyone's rows).

- [ ] **By Thu 12 Nov at 12:00:** own rows. 5 endpoint rows for your own inventory; 1 table row (reviews) to Haarisah.
- [ ] **By Thu 12 Nov at 12:00, fix F11:** SQLite (our database) ignores the length in a type such as `VARCHAR(200)`, so it would store a longer value without complaint. For every column in your tables that has a length limit, add a CHECK, e.g. `CHECK (length(body) <= 2000)`, so the database enforces the limit as well as the API's 422.
- [ ] **By Thu 12 Nov at 12:00, fix F12:** a CHECK passes when the value is missing (NULL). Any role, status or type column that has a CHECK must also be NOT NULL, or a row with no value could be inserted directly with SQL. Your reviews table has no such column; `rating` is already NOT NULL in the plan, so just confirm it in your row.
- [ ] **Thu 12 Nov, from 12:00:** chase anyone whose endpoint rows have not arrived.
- [ ] **By Fri 13 Nov:** endpoint inventory compiled from everyone's rows. Check every row against the row format and the error contract: success code shown, at least two sad paths, full error bodies, sad paths in the order of checks, and route and field names that match the database columns.
- [ ] **By Fri 13 Nov:** include the API design rationale table from plan section 5 (why each method, why nested routes, why one error shape), and check that fixes F1, F2 and F4 appear in Husray's and Victor's rows.
- [ ] **By Fri 13 Nov, fix F16:** state the protocol (the agreed rules for how the client and the API talk to each other) at the start of the API design section: HTTP; JSON request and response bodies; bearer tokens (the login token sent in the `Authorization` header of each request); and REST-style resource URIs (each URL names a thing, such as `/games/3/reviews`, and the method says what to do with it).
- [ ] **By Fri 13 Nov:** confirm every functional requirement (FR1 to FR12) has at least one endpoint.
- [ ] **Design contribution:** make and record at least one design decision of your own on your slice, with the alternative you rejected and why. For example, whether the review text is optional, or whether editing a review must send both the rating and the text.
- [ ] **Be ready to answer:**
  - "Why is writing a review a POST that gives 409 the second time, when collections use a PUT that just updates?"
  - "A review with no rating gives 400 but a rating of 11 gives 422. Why are they different, and what would happen if the CHECK on rating were removed?"

## Husray

**Slice:** Task 6, friends, following and activity feed. **D2 lead:** evaluation ideas and the traceability matrix.

- [ ] **By Thu 12 Nov at 12:00:** own rows, with fixes F1, F2 and F3 applied. 7 endpoint rows to Zeeshan (8 if you take fix F2 as written); 2 table rows (friendships, follows) to Haarisah.
- [ ] **By Thu 12 Nov at 12:00, fix F1:** following is currently a POST, but favourites use an idempotent PUT for the same kind of thing (linking two records). Either change it to `PUT /users/{id}/follow` (201 the first time, 200 on a repeat), or keep POST and write down why. If you choose PUT, the "following someone twice (409)" sad path no longer exists, so replace it with another one.
- [ ] **By Thu 12 Nov at 12:00, fix F2:** declining a friend request is currently a PATCH that deletes the request. A PATCH is meant to change a record, not remove it. Make declining `DELETE /friends/requests/{id}` (204), and keep PATCH for accepting only.
- [ ] **By Thu 12 Nov at 12:00, fix F3:** add `CHECK (requested_by IN (user_low_id, user_high_id))` to friendships, so the database refuses a request that claims to come from someone who is not one of the two players.
- [ ] **By Thu 12 Nov at 12:00, fix F11:** SQLite (our database) ignores the length in a type such as `VARCHAR(200)`, so it would store a longer value without complaint. For every column in your tables that has a length limit, add a CHECK, e.g. `CHECK (length(username) BETWEEN 1 AND 30)` on the users table, so the database enforces the limit as well as the API's 422.
- [ ] **By Thu 12 Nov at 12:00, fix F12:** a CHECK passes when the value is missing (NULL). Any role, status or type column that has a CHECK must also be NOT NULL, or a row with no value could be inserted directly with SQL. In your slice this is `friendships.status`.
- [ ] **By Fri 13 Nov:** evaluation ideas, following the chain requirement, endpoint, test, expected result, evidence. State that "the program runs" is not evidence that a requirement is met.
- [ ] **By Fri 13 Nov:** traceability matrix, with Endpoint and Bruno test name columns added.
- [ ] **By Fri 13 Nov, fix F6:** timing requests on 20 seed games proves nothing about performance. Decide between generating a large seed (thousands of games) for the timing test, or dropping the "under 200 ms" claim. Tell Victor if you want the large seed.
- [ ] **Design contribution:** fix F1 is a real design decision and it is yours: record both options and your reason. If you would like a different one, record another choice on your slice in the same way.
- [ ] **Be ready to answer:**
  - "Why is each friendship stored once with the smaller id first, and not as two rows?"
  - "Why this method for following, and what happens if you follow yourself or send a request to a user that doesn't exist?"

## Victor

**Slice:** Task 7, forums and integration. **D2 lead:** AI usage section.

- [ ] **Now:** confirm the repo exists with all 7 added, and that everyone has pulled the revised plan.
- [ ] **By Thu 12 Nov at 12:00:** own rows, with fix F4 applied. 7 endpoint rows to Zeeshan (your 6 plus the new one); 2 table rows (threads, posts) to Haarisah.
- [ ] **By Thu 12 Nov at 12:00, fix F4:** add `DELETE /threads/{id}` (author or maintainer, 204). The stakeholder table says maintainers remove bad content, but at the moment a bad thread can't be removed. What happens to the thread's posts is fix F13, below.
- [ ] **By Thu 12 Nov at 12:00, fix F13:** with Haarisah, decide what happens to a thread's posts when the thread is deleted. The foreign key `posts.thread_id` is either ON DELETE CASCADE (the posts are deleted with the thread) or ON DELETE RESTRICT (a thread can't be deleted while it has posts). Record the options and the reason; this counts as a design decision of your own.
- [ ] **By Thu 12 Nov at 12:00, fix F11:** SQLite (our database) ignores the length in a type such as `VARCHAR(200)`, so it would store a longer value without complaint. For every column in your tables that has a length limit, add a CHECK, e.g. `CHECK (length(title) <= 200)`, so the database enforces the limit as well as the API's 422.
- [ ] **By Thu 12 Nov at 12:00, fix F12:** a CHECK passes when the value is missing (NULL). Any role, status or type column that has a CHECK must also be NOT NULL, or a row with no value could be inserted directly with SQL. Your tables have no such column, so just confirm it.
- [ ] **By Fri 13 Nov:** AI usage section, using the log format in plan section 16. Ask everyone to send you their entries (tool, date, purpose, a representative prompt, what was used, how it was checked, what was changed).
- [ ] **Design contribution:** make and record at least one design decision of your own on your slice, with the alternative you rejected and why. For example, whether creating a thread also creates its first post, or whether a thread title can be edited.
- [ ] **Be ready to answer:**
  - "Why are posts a separate table from threads, and what would happen if the foreign key from threads to games were removed?"
  - "A post with no body field gives 400 but an empty post gives 422. Why are they different, and what happens when you post to a thread that doesn't exist?"

---

## Whole group

- [ ] **Now:** send your student ID and full name to Lee.
- [ ] **By Thu 12 Nov:** confirm the CP0 decisions, and fill in the "Agreed" column in plan section 2.
- [ ] **By Thu 12 Nov:** agree the names in `shared_checks_proposal.md`, so everyone can write against them.
- [ ] **By Thu 12 Nov:** mark `Videogames_DB_Task_Split_v2.xlsx` as superseded. It contradicts the plan (status flags, `/users/me` routes, forum priority, no friendships), so nobody should work from it.
- [ ] **By Thu 12 Nov:** send your AI usage log entries to Victor.
- [ ] **From now:** use Python 3.13 or later (fix F14). Ayaan is emailing the tutor for written approval to use SQLite, which is needed before D2 is submitted.
- [ ] **Fri 13 Nov:** consistency check (tracker section 5), then sign-off by all 7. The document follows the FAQ's structure (see Lee's list), because there is no Moodle template.
- [ ] **Weekend of 14 and 15 Nov:** mock meeting. Each person explains their slice and one design decision, with the alternative they rejected, and practises their two questions. Rehearse to 15 minutes or less.
- [ ] **Tue 17 Nov:** all 7 attend the progress meeting. If the group misses it, every D2 aspect is capped at 2 out of 5.
- [ ] **Tue 17 Nov:** Lee uploads the document before 15:00.

---

## Open issues

Problems spotted while writing this file. Nothing in the plan's design has been changed.

1. **Fixes F1, F2 and F4 change endpoints that are written in the plan.** Plan section 6 (Tasks 6 and 7), the stakeholder mapping in section 3 and the MoSCoW table still show the old routes. They need updating so the names match in every section.
2. **The rows deadline has moved from Mon 9 Nov to Thu 12 Nov at 12:00.** That leaves Zeeshan and Haarisah a day and a half to compile the inventory, the merged table and the ERD before sign-off on Fri 13 Nov.
3. **The plan in some local copies is out of date.** The revised plan is on GitHub. Anyone who has not pulled since Tue 3 Nov has the old version, which has no priorities, no row formats with FR columns, and different table designs. The tracker is not on GitHub yet.
4. **Team names don't match.** The plan lists Task 6 as Husray; the README on GitHub lists Hussain, and still uses the old task titles. The title page needs everyone's full name as registered.
5. **The owner check can't be a decorator and keep the order of checks.** The plan describes the owner-or-maintainer check as a decorator, but the order of checks puts ownership after the 400, 422 and 404 checks. `shared_checks_proposal.md` proposes a function called inside the route. To agree.
6. **The plan's NFR8 wording still mentions the OpenAPI docs.** It should be updated to match the D2 section, where the evidence is the README and the Bruno collection, with the OpenAPI docs as an extra if built.
7. **Open points on Task 1** are listed in `ayaan_d2_notes_for_group.md`, including the login response body, the `field` key being `null` when no input is at fault, and the delete rule for `favourite_games.user_id`.
