# D2 sections led by Ayaan: title page, non-functional requirements, error contract

Each section below is ready to paste into the D2 document. This heading and the lines in *italics* are notes for Lee and are not part of the document.

---

## A. Title page

**CMP5387 Backend Web-services Development and Database Engineering**

**Deliverable 2 (D2): Design Documentation**

# Online Videogames Database with Social Features

| # | Name | Student ID | Task |
|---|---|---|---|
| 1 | Ayaan Ahmed | | Users, auth and profiles |
| 2 | Lakhan | | Catalogue browse and user content |
| 3 | Haarisah | | Catalogue maintenance |
| 4 | Lee | | Personal collections |
| 5 | Zeeshan | | Reviews and ratings |
| 6 | Husray | | Friends, following and activity feed |
| 7 | Victor | | Forums and integration |

**Date:** 17 November 2026

*Notes for Lee: full names (first name and surname) and student IDs still need adding for everyone. Check the module title and the submission date against Moodle.*

---

## B. Non-functional requirements

The problem statement describes what players and maintainers can do. It does not say how well the system must do it, but several qualities are implied. A community platform holds accounts and data that players have built up over time, so it has to be secure and it must never lose or corrupt that data. It is also built by seven people and marked by someone running it on their own machine, so it has to behave consistently and be easy to set up. We treat these implicit needs as non-functional requirements (NFRs). Each one has a planned source of evidence, so that we can show it has been met and not only claim it.

| ID | Requirement | Why it matters for this system | How we will evidence it |
|---|---|---|---|
| NFR1 | Security: passwords stored only as hashes; login needed for every change; maintainer-only routes checked | Players can only trust the platform with their collections and reviews if nobody else can log in as them or change the catalogue without permission. | 401 and 403 tests in Bruno; the users table shows only hashes |
| NFR2 | Integrity: rules enforced by the database as well as the API | A bug in one slice's API code must not be able to store an invalid rating, a duplicate record or a review of a game that doesn't exist. | CHECK, UNIQUE and foreign key constraints; bad data inserted directly with SQL is rejected |
| NFR3 | Consistency: a failed request never leaves a half-finished change | The problem statement requires rejected changes to leave existing data unchanged, including when a change has several steps. | Transactions; before-and-after checks on sad paths |
| NFR4 | Predictable API: consistent JSON, status codes and error format | Seven slices are written by seven people, and a client should not need to handle each one differently. | The same error body from every slice |
| NFR5 | Performance: lists are paginated and filtered columns are indexed | The catalogue and review lists grow as the community grows, so no request should return or scan everything at once. | Timed filtered requests on a seed large enough to show the effect of the indexes |
| NFR6 | Reproducibility: the system runs from the zip on a clean machine | A submission that the marker can't run can't be marked. | The clean-machine test at checkpoint CP5 |
| NFR7 | Maintainability: one module per slice, commented code, reviewed changes | Each member has to explain their own code, and the group has to merge seven slices without breaking each other's work. | Repo structure and pull request history |
| NFR8 | Documentation: README, OpenAPI docs and an organised Bruno collection | Bruno is the client, so the documentation is the only guide a marker has to using the system. | The README and the Bruno collection, with the OpenAPI docs as an extra if built |

NFR1 to NFR4, NFR6 and NFR8 are Must priorities. The "reject the change and leave the data unchanged" stories are not true without NFR1 to NFR4, and a system that can't be set up and used from its documentation can't be marked. NFR5 and NFR7 are Should priorities.

---

## C. Error contract

Every endpoint in every slice reports failure in the same way: one of six status codes, and one error body.

| Status | Meaning | Use it when | Example |
|---|---|---|---|
| 400 | Bad request | The request can't be read: not valid JSON, a value of the wrong type, a required field missing, or a field the endpoint doesn't accept | A review with no `rating` field; `page=abc` |
| 401 | Not logged in | No token, an invalid token, or a wrong password at login | A collection update with no token |
| 403 | Not allowed | Logged in, but not allowed to do this: changing someone else's data, a player on a maintainer-only route, or changing your own role | PUT on another player's collection; a player PATCHing a game |
| 404 | Not found | An id in the URL, or an id in the body, doesn't exist | A collection entry for game 99999 |
| 409 | Conflict | The change clashes with existing data | A second review of the same game; deleting a game that players have data about |
| 422 | Invalid value | The request was read, but a value breaks a rule: out of range, not an allowed option, empty, too long or the wrong format | Rating 11; status `borrowed`; an empty post |

**Success codes**

| Status | Used for | Body |
|---|---|---|
| 200 | A read or an update | The resource or list |
| 201 | A create | The new resource |
| 204 | A delete | No body |

**Error body (identical everywhere)**

`{"error": {"code": 422, "message": "rating must be between 1 and 10", "field": "rating"}}`

`code` repeats the status code, `message` says what went wrong in plain English, and `field` names the input at fault. `field` is `null` when no single input is to blame, for example when there is no token.

**Order of checks**

Checks always run in this order, and the first one that fails decides the response:

| Step | Check | Status |
|---|---|---|
| 1 | Not logged in | 401 |
| 2 | A player on a maintainer-only route | 403 |
| 3 | Unreadable request or a missing field | 400 |
| 4 | Invalid value | 422 |
| 5 | Unknown id | 404 |
| 6 | Not allowed, e.g. not the owner | 403 |
| 7 | Clash with existing data | 409 |

Only then is the change written, inside a transaction, so a failed request saves nothing.

**Why one shared format**

Seven people are writing endpoints, and without a contract each slice would report errors in its own way. With one format, a client handles every failure with the same code: it reads the status, shows the message and highlights the field. Our Bruno tests benefit in the same way, because every sad-path test asserts the same three keys. The fixed order of checks matters as much as the body. A request with two problems, such as an invalid rating on someone else's review, always gets the same answer whichever slice receives it, so the expected result of every test can be written down at design time. The contract is implemented once, as a shared error helper that every slice calls, so the format cannot drift between slices.
