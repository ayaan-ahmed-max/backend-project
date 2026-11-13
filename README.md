# backend-project

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
| 1 | Users, auth and profiles | Ayaan Ahmed | 25178079 | |
| 2 | Catalogue browse | Lakhan | | |
| 3 | Catalogue maintenance | Haarisah | 25132920 | |
| 4 | Personal collections | Lee | | |
| 5 | Reviews and ratings | Zeeshan | | |
| 6 | Friends, following and activity | Hussain | 25184802 | |
| 7 | Forums, user content and integration | Victor | 25176535 | |

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
| Development and maintenance team | Implicit | Has to run, test and extend the system | Reproducible setup, documented API, consistent error
