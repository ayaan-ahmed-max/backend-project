# CMP5387 D2 Progress Tracker

Tracks the D2 work against the D2 Progress Checklist (5-star marking guide). The design itself lives in `CMP5387_Group_Plan.md`; this file only tracks what is done and what is left.

**Last updated:** Wed 11 Nov 2026

**Key dates**
- **Fri 13 Nov (CP1):** D2 document finished, checked and signed off by all 7
- **Weekend 14 to 16 Nov:** mock meeting
- **Tue 17 Nov (CP2):** progress meeting (all 7 attend) and upload before 15:00 (confirm on Moodle)

If the group misses the meeting, every D2 aspect is capped at 2/5.

---

## 1. Blockers (everything else waits on these)

- [ ] All 7 student IDs collected (Lee). No names and IDs means no marks.
- [ ] Every member's own endpoint and table rows sent to Zeeshan (endpoints) and Haarisah (tables). Originally due Mon 9 Nov.
- [ ] CP0 decisions confirmed and the "Agreed" column in plan section 2 filled in
- [ ] `Videogames_DB_Task_Split_v2.xlsx` marked as superseded. It contradicts the plan (status flags, `/users/me` routes, forum priority, no friendships).

---

## 2. Tasks per person

Rows are in the plan's "D2 row formats" (section 5). Every endpoint row has realistic JSON happy and sad examples using the shared error body.

### Ayaan (Task 1)
- [x] Own rows drafted and fixed: 6 endpoints, 2 tables (users, favourite_games)
- [ ] Decide the rule for unexpected fields across all slices (fix F10)
- [ ] NFR section: correct the priorities and the NFR5 and NFR8 evidence
- [ ] Rows sent to Zeeshan and Haarisah
- [ ] Title page with all names and IDs
- [ ] Non-functional requirements section
- [ ] Error contract section
- [ ] AI usage log entry for the row drafting

### Lakhan (Task 2)
- [ ] Own rows: 6 endpoints, 4 tables (games, genres, game_genres, user_content)
- [ ] Problem summary in prose, each requirement marked explicit or implicit, each assumption justified
- [ ] Stakeholder table
- [ ] MoSCoW priority table with reasons and the Won't list

### Haarisah (Task 3)
- [ ] Own rows: 7 endpoints, 3 tables (platforms, game_platforms, catalogue_changes)
- [ ] Merged database design table, with a data type on every column
- [ ] Physical ERD
- [ ] Corrected index list (see fix F5)

### Lee (Task 4)
- [ ] Own rows: 3 endpoints, 1 table (collection_entries)
- [ ] Student IDs collected
- [ ] Gantt chart as a real chart (see section 4.3)
- [ ] Document assembled as a Word or PDF file in the FAQ's structure: title page; requirements analysis (problem summary, stakeholders, requirements, with a table of how each requirement will be evaluated); entity relationship modelling (conceptual, logical and physical ERDs with discussion); API design (protocol and resource URI inventory); timeline and workload planning (subtasks, dependencies, Gantt); MoSCoW; AI usage. No Moodle template exists
- [ ] Body text under 4000 words (tables, ERD, Gantt and code don't count)
- [ ] BCU Harvard references, cover sheet, font 11, 1.5 line spacing
- [ ] Upload before 15:00 on Tue 17 Nov

### Zeeshan (Task 5)
- [ ] Own rows: 5 endpoints, 1 table (reviews)
- [ ] Endpoint inventory compiled from everyone's rows
- [ ] Every row checked against the row format and the error contract

### Husray (Task 6)
- [ ] Own rows: 7 endpoints, 2 tables (friendships, follows), including fixes F1, F2 and F3
- [ ] Evaluation ideas
- [ ] Traceability matrix with Endpoint and Bruno test name columns added
- [ ] Decision on the performance test (fix F6)

### Victor (Task 7)
- [ ] Own rows: 6 endpoints plus `DELETE /threads/{id}` (fix F4), 2 tables (threads, posts)
- [ ] AI usage section
- [ ] Repo confirmed, with all 7 added

---

## 3. Design fixes from the checklist review

| ID | Fix | Owner | Why (checklist point) | Done |
|---|---|---|---|---|
| F1 | Following becomes `PUT /users/{id}/follow` (201 new, 200 repeat), or POST is kept with a written justification | Husray | Favourites use idempotent PUT for the same kind of pair; inconsistent method choice will be probed | [ ] |
| F2 | Declining a friend request becomes `DELETE /friends/requests/{id}` (204); PATCH is only for accepting | Husray | A PATCH that deletes the resource fails "why is this method appropriate?" | [ ] |
| F3 | Add `CHECK (requested_by IN (user_low_id, user_high_id))` to friendships | Husray | "What prevents invalid data being entered here?" | [ ] |
| F4 | Add `DELETE /threads/{id}` (author or maintainer, 204) | Victor | Stakeholder table says maintainers remove bad content; a bad thread can't be removed | [ ] |
| F5 | Index `game_genres(genre_id)`, `game_platforms(platform_id)` and `reviews(game_id)`, not `games.title` | Haarisah | A `LIKE '%x%'` search can't use an index; a marker asking "what does this index do?" exposes it | [ ] |
| F6 | Either generate a large seed (thousands of games) for the NFR5 timing test, or drop the 200 ms claim | Husray | Timings on 20 games prove nothing; evaluation must give real evidence | [ ] |
| F7 | State that users are never deleted through the API, so foreign keys to users use the default RESTRICT (threads are now deletable, see F13) | Haarisah | Every foreign key needs a stated delete behaviour | [ ] |
| F8 | Task 3 delete rule lists user content (guides and links) as player data that blocks deletion | Haarisah | Must match the database rules in plan section 5 | [ ] |
| F9 | Map `GET /games/{id}/history` to a stakeholder | Lakhan | Every endpoint should trace to a stakeholder need | [ ] |
| F10 | One rule for unexpected fields in every slice: any field an endpoint doesn't accept gives 400, so a `role` field at registration is 400, not 422. A `role` field on `PATCH /users/{id}` stays 403, because a logged-in user is attempting something not allowed. Add "a field the endpoint doesn't accept" to the 400 row of the error contract. | Ayaan | One rule in every slice is easier to defend than a different code per endpoint | [ ] |
| F11 | SQLite ignores VARCHAR lengths, so every length limit needs a CHECK, e.g. `CHECK (length(username) BETWEEN 1 AND 30)`, as well as the API's 422 | Everyone (Haarisah checks) | "What prevents invalid data being entered here?" Without the CHECK, nothing in the database does | [ ] |
| F12 | Every role, status and type column with a CHECK is also NOT NULL, because a CHECK passes on NULL | Everyone (Haarisah checks) | A row with no role, status or type could otherwise be inserted directly with SQL | [ ] |
| F13 | Decide what happens to a thread's posts when the thread is deleted (F4): `posts.thread_id` ON DELETE CASCADE or RESTRICT | Victor and Haarisah, recorded as their own design decision | Every foreign key needs a stated delete behaviour, and F4 makes threads deletable | [ ] |
| F14 | Get written approval from the tutor for SQLite (the FAQ's recommended DBMS is MariaDB/MySQL, and any change must be justified in D2 and approved before D2 is submitted). Add a justification paragraph to D2. Use Python 3.13 or later. | Ayaan emails; Haarisah writes the justification | Required by the FAQ before D2 is submitted | [ ] |
| F15 | Present the database design as conceptual, logical and physical ERDs, each with a short discussion | Haarisah (everyone checks their own tables) | The FAQ's document structure asks for all three ERDs with discussion | [ ] |
| F16 | State the protocol in the API design section: HTTP, JSON request and response bodies, bearer tokens, REST-style resource URIs | Zeeshan | The FAQ's API design section asks for the protocol as well as the resource URI inventory | [ ] |
| F17 | The contribution table shows every member contributing to database design, API design and planning, not one person per area | Lee | The brief requires each member to demonstrate their contribution to the design process | [ ] |

---

## 4. D2 sections against the 5-star checklist

Target: 4 to 5 stars in every section. "Plan estimate" is what the plan's content supports if it is written up properly.

| Section | Weight | Lead | Plan estimate | Status |
|---|---|---|---|---|
| 1. Problem specification | 2% | Lakhan | 4 | Not started |
| 2. Stakeholders | 2% | Lakhan | 5 | Not started |
| 3. Gantt chart | 4% | Lee | 3 (as a table) | Not started |
| 4. Endpoint inventory | 5% | Zeeshan | 4 to 5 | Rows outstanding |
| 5. Database design | 5% | Haarisah | 4 | Rows and ERD outstanding |
| 6. Evaluation ideas | 2% | Husray | 4 to 5 | Not started |

### 4.1 Problem specification (5-star needs)
- [ ] Problem and purpose explained in prose, not only as a requirements table
- [ ] Users identifiable from the description
- [ ] Each requirement marked explicit (from the problem statement) or implicit (derived by us)
- [ ] Data to be stored listed
- [ ] Functional and non-functional requirements both present
- [ ] Assumptions stated and justified
- [ ] Every later design decision traces back to something here

### 4.2 Stakeholders (5-star needs)
- [ ] Table with kind (direct or indirect), interest and needs
- [ ] Different player roles distinguished (player, collector, reviewer, social user, maintainer)
- [ ] Competing needs discussed, with how the design resolves them
- [ ] Each stakeholder mapped to endpoints and tables

### 4.3 Gantt chart (5-star needs)
- [ ] An actual chart, with tasks as rows and dated bars, not a weekly table per member
- [ ] All 7 members named against their tasks
- [ ] Stages covered: planning, database, web service, client (Bruno), testing, demo and submission preparation
- [ ] Milestones CP0 to CP7 marked
- [ ] Dependencies drawn (shared checks and seeded models lead into every slice's build)
- [ ] Parallel work visible (weeks 10 to 12)
- [ ] "If something runs late" plan included, following the MoSCoW priorities

### 4.4 Endpoint inventory (5-star needs)
- [ ] Every endpoint has route, method, description, users, FR and priority
- [ ] Realistic JSON happy input and output, with the success code (200, 201 or 204)
- [ ] At least 2 sad paths per endpoint, each with its status code and full error body
- [ ] Every functional requirement covered by at least one endpoint
- [ ] Naming consistent across slices; fixes F1, F2 and F4 applied
- [ ] Inputs and outputs match the database columns
- [ ] API design rationale included (why each method, why nesting, why one error shape)

### 4.5 Database design (5-star needs)
- [ ] Table summary with every column's data type
- [ ] Primary keys, foreign keys and ON DELETE behaviour for every relationship
- [ ] Cardinality table
- [ ] CHECK, UNIQUE and NOT NULL constraints listed; fixes F3, F5, F7, F8, F11, F12, F13 and F15 applied
- [ ] Physical ERD consistent with the table summary
- [ ] Every table serves a requirement
- [ ] Modelling alternatives considered, with reasons

### 4.6 Evaluation ideas (5-star needs)
- [ ] Chain shown: requirement, endpoint, test, expected result, evidence
- [ ] Bruno test names linked to inventory examples
- [ ] Happy and sad behaviour both tested
- [ ] Database integrity tested directly (NFR2)
- [ ] Non-functional requirements have real evidence (fix F6)
- [ ] States that "the program runs" is not evidence a requirement is met

---

## 5. Cross-document consistency check (before sign-off)

| Check | Yes / Not yet |
|---|---|
| Problem requirements appear in the stakeholder analysis | [ ] |
| Stakeholder needs shape the functionality | [ ] |
| Every functional requirement has an endpoint | [ ] |
| Every endpoint's data is supported by a table | [ ] |
| Every table exists for a stated requirement | [ ] |
| The Gantt chart includes the work to build the planned API and database | [ ] |
| The evaluation can show whether each important requirement is met | [ ] |
| Names (tables, routes, statuses, roles) match across every section | [ ] |
| The group can explain decisions, not just show diagrams | [ ] |
| All 7 members know the agreed design and plan | [ ] |

---

## 6. Pre-meeting self-assessment (do this at the mock meeting)

| Question | Yes | Partly | No |
|---|---|---|---|
| Can we clearly explain the problem we are solving? | [ ] | [ ] | [ ] |
| Can we identify the important explicit and implicit requirements? | [ ] | [ ] | [ ] |
| Can we explain who our stakeholders are and what they need? | [ ] | [ ] | [ ] |
| Does our Gantt chart clearly show who is doing what and when? | [ ] | [ ] | [ ] |
| Have we considered dependencies between tasks? | [ ] | [ ] | [ ] |
| Can we justify our important endpoints and HTTP methods? | [ ] | [ ] | [ ] |
| Have we designed both happy and sad path behaviour? | [ ] | [ ] | [ ] |
| Can we explain our entities, keys and relationships? | [ ] | [ ] | [ ] |
| Does our database actually support our API? | [ ] | [ ] | [ ] |
| Have we planned how to test whether requirements are met? | [ ] | [ ] | [ ] |
| Are all six D2 sections consistent with one another? | [ ] | [ ] | [ ] |
| Could every member explain the major design decisions? | [ ] | [ ] | [ ] |
| Can we explain why we made our decisions, not just what we made? | [ ] | [ ] | [ ] |

---

## 7. Meeting preparation

- [ ] Mock meeting held over the weekend
- [ ] Each member explains their slice and one design decision, with the alternative they rejected
- [ ] Each member practises one marker probe for their slice, for example:
  - Why is this a separate table rather than another column?
  - What would happen if this foreign key were removed?
  - Why this HTTP method and this status code?
  - What happens if the resource doesn't exist, or required input is missing?
- [ ] Rehearsed to 15 minutes or less
- [ ] Haarisah ready to note the tutor's feedback in the design change log
- [ ] All 7 confirmed attending on Tue 17 Nov