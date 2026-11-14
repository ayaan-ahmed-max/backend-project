# D2 Evaluation Plan and Traceability Matrix

**Name:** Hussain (Husray)  
**Student ID:** 25184802  
**Module:** CMP5387  
**D2 responsibility:** Evaluation ideas and requirements traceability matrix

## Evaluation Approach

For the evaluation, we need to check whether the system meets the requirements we agreed on in D2.

The main approach is to test each requirement using the API endpoints in Bruno, then compare the response with what we expected. We will also use pytest for automated tests and check the database directly where needed.

Testing successful requests is not enough. We also need to check invalid inputs, missing records and permission errors. If a request fails, the database should remain unchanged.

The tables below show the tests we plan to use. These are expected results, not completed test results. The actual outcomes will be recorded during D3.

## Functional Requirements Traceability Matrix

| ID | Requirement | Endpoint(s) | Planned test | Expected result | Evidence |
|---|---|---|---|---|---|
| FR1 | Browse and filter games | GET /games | Search by title, genre and platform, including combined filters | 200; only matching games returned, with correct pagination | Bruno responses |
| FR2 | View game details | GET /games/{id} | Request a game that is available on multiple platforms | 200; correct description, genres, platforms, release dates and average rating | Bruno and database |
| FR3 | Manage personal collections | GET /users/{id}/collection, PUT /users/{id}/collection/{game_id}, DELETE /users/{id}/collection/{game_id} | Add a game as wanted, view the collection and remove it | 201 when added, 200 when listed and 204 when deleted | Bruno and database |
| FR4 | Update collection status without duplicates | PUT /users/{id}/collection/{game_id} | Add a game as wanted, change it to played and repeat the request | 201, then 200 and 200; only one collection record exists | Bruno and row count |
| FR5 | Review games and prevent unauthorised changes | POST /games/{id}/reviews, PUT /reviews/{id} | Add a review, edit it, submit rating 11 and attempt to edit another user's review | Valid changes succeed; rating 11 gives 422; editing another user's review gives 403 | Bruno and before/after GET |
| FR6 | Keep reviews separate and calculate average ratings | GET /games/{id}/reviews, GET /games/{id}/rating | Two users review the same game with different ratings | Both reviews remain separate and the average is calculated correctly | Bruno and database |
| FR7 | Maintain catalogue data without losing player records | POST /games, PATCH /games/{id}, PUT /games/{id}/platforms/{platform_id}, DELETE /games/{id} | Correct a description and release date; try deleting a game with reviews | Corrections succeed without changing player data; protected deletion gives 409 | Bruno and database |
| FR8 | Reject invalid or unauthorised changes | Relevant write endpoints across all tasks | Request an unknown game, change another player's collection and submit invalid data | Correct 404, 403 or 422 response; existing data unchanged | Bruno and before/after GET |
| FR9 | Register, log in and manage profiles | POST /auth/register, POST /auth/login, GET /users/{id}, PATCH /users/{id} | Register and log in, then edit own profile and attempt to edit another user's profile | Registration 201, login 200; owner edit allowed, other user edit gives 403 | Bruno |
| FR10 | Friends, following and activity feed | /friends/requests, /users/{id}/friends, /friends/{user_id}, /users/{id}/follow, /feed | Send and accept a request, follow a user and check their recent activity | Accepted friendship appears in the friends list; feed contains relevant activity once | Bruno and database |
| FR11 | Forum discussions | GET and POST /games/{id}/threads, GET and POST /threads/{id}/posts | Create a thread and post; try editing someone else's post if editing is implemented | Valid creation gives 201; unauthorised editing gives 403 | Bruno |
| FR12 | Guides and media links | GET and POST /games/{id}/content | Add a guide and try adding a non-HTTPS media link | Valid guide gives 201; invalid link gives 422 | Bruno |

## Non-Functional Requirements Traceability Matrix

| ID | Requirement | Planned test | Expected result | Evidence |
|---|---|---|---|---|
| NFR1 | Security | Access a protected route without a token and try using a player account on a maintainer route | 401 without authentication and 403 without permission; passwords stored as hashes | Bruno and database |
| NFR2 | Database integrity | Try inserting an invalid rating, duplicate relationship or missing foreign key directly into the database | Database constraints reject the invalid records | SQLite constraint errors |
| NFR3 | Consistent transactions | Force a failure during a multi-step update | No partial changes are saved | pytest and database checks |
| NFR4 | Consistent API errors | Run invalid requests against endpoints from different team members | Errors use the same JSON structure and appropriate HTTP status codes | Bruno responses |
| NFR5 | Performance | Measure filtered game queries using the seed database | Requests meet the group's local target of under 200 ms | Bruno timings |
| NFR6 | Reproducibility | Install the project on another computer using only the README | Database can be created, server starts and test requests work | Clean-machine test record |
| NFR7 | Maintainability | Check project structure, code comments and pull requests | Features are organised into separate modules and changes are reviewed | GitHub repository |
| NFR8 | Documentation | Follow the README to run the API and Bruno collection; check OpenAPI documentation if implemented | Instructions are accurate and the documented requests match the API | README, Bruno and documentation review |

## Task 6 – Detailed Evaluation

My own part is FR10, covering friend requests, following and the activity feed. I will focus on checking that relationships are stored correctly and that users cannot change relationships they do not have permission to manage.

### Test 1 – Sending a Friend Request

**Steps:**
1. Log in as user 7.
2. Send a friend request to user 8.
3. Check the friendships table.

**Expected result:**

The API returns 201 and creates one pending relationship. The `requested_by` field should contain user 7.

### Test 2 – Preventing Duplicate Requests

**Steps:**
1. Send a friend request from user 7 to user 8.
2. Try sending the same request again.
3. Try sending another request from user 8 to user 7.

**Expected result:**

The duplicate requests return 409. There should still be only one friendship row for the pair.

### Test 3 – Accepting a Friend Request

**Steps:**
1. User 7 sends a request to user 8.
2. Log in as user 8.
3. Accept the request.
4. View both users' friends lists.

**Expected result:**

The request becomes accepted, and both users appear in each other's friends lists. No second friendship row is created.

### Test 4 – Checking Permissions

**Steps:**
1. User 7 sends a friend request to user 8.
2. Log in as another user.
3. Try accepting the request.
4. Check the friendship record again.

**Expected result:**

The API returns 403 because only the recipient should be allowed to accept it. The request should remain pending.

### Test 5 – Self-Friendship

**Steps:**
1. Log in as user 7.
2. Try sending a friend request to user 7.

**Expected result:**

The API returns 422 and no friendship is created.

The database CHECK constraint should also prevent a friendship row with the same user on both sides.

### Test 6 – Declining a Request

**Steps:**
1. User 7 sends a request to user 8.
2. Log in as user 8.
3. Decline the request.
4. Check the friendships table.

**Expected result:**

The pending request is removed and the API returns 204.

This test uses the proposed DELETE endpoint, which must be confirmed with the group.

### Test 7 – Following Users

**Steps:**
1. Log in as user 7.
2. Follow user 8.
3. Repeat the same request.
4. Check the follows table.

**Expected result:**

The first request returns 201 and the repeated PUT returns 200. Only one follow record exists.

Following yourself should return 422.

### Test 8 – Activity Feed

**Steps:**
1. Create an accepted friendship between users 7 and 8.
2. User 8 adds a game to their collection or writes a review.
3. Log in as user 7.
4. Request the activity feed.

**Expected result:**

The activity appears in user 7's feed, sorted from newest to oldest.

If user 7 also follows user 8, the same activity should not appear twice.

The feed must also return the correct page number, page size and total.

### Test 9 – Failed Requests Leave Data Unchanged

For each rejected write request, I will check the relevant record before and after sending it.

For example, if an unrelated user tries to accept a friend request, the API should return 403 and the friendship should still be pending.

This is important because returning the correct error code is not enough if the database has already been changed.

## Recording the Results

During D3, the evaluation table will be updated with two more columns:

- **Actual result:** What happened when the test was run.
- **Verdict:** Met, Partly Met or Not Met.

For every failed test, we should record what went wrong, whether it was fixed and any remaining limitations.

The Must requirements will be checked first because these are the main features the system needs to deliver.

Should and Could features will also be evaluated if implemented. If anything is not completed, it should be recorded in the design change log rather than marked as passed.

## Dependencies and Limitations

The Task 6 tests need the users table and shared authentication from Ayaan. The feed also needs collection data from Lee and review data from Zeeshan.

The exact response fields and test user IDs will be checked once the shared backend and seed data are available.

A limitation of the proposed feed is that it uses the current collection and review records. It will not show every previous edit or deleted activity because there is no separate activity history table.

At this stage, this document is the testing plan. No Bruno or pytest results are being claimed until the actual tests are run.
