# Task 6 – D2 Design Contribution

**Name:** Hussain (Husray)  
**Student ID:** 25184802  
**Task:** Friends, Following and Activity Feed

## Database Design

My part of the project covers friend requests, friendships, following other users and the activity feed. I need two tables for the relationships between users. The feed will use the collections and reviews tables that Lee and Zeeshan are working on.

| FR | Table | Columns (type) | Keys and relationships | Constraints |
|---|---|---|---|---|
| FR10 | `friendships` | `id INTEGER`<br>`user_low_id INTEGER`<br>`user_high_id INTEGER`<br>`requested_by INTEGER`<br>`status VARCHAR(8)`<br>`created_at DATETIME` | `id` is the primary key. The three user ID columns reference `users.id`. Foreign keys use `ON DELETE RESTRICT`. | All fields are NOT NULL. `UNIQUE(user_low_id, user_high_id)` prevents duplicate pairs. `CHECK(user_low_id < user_high_id)` prevents self-friendship and keeps pairs in one order. `CHECK(requested_by IN (user_low_id, user_high_id))` ensures the sender belongs to the pair. Status is limited to `pending` or `accepted`. |
| FR10 | `follows` | `follower_id INTEGER`<br>`followee_id INTEGER`<br>`created_at DATETIME` | Both IDs reference `users.id` with `ON DELETE RESTRICT`. The primary key is `(follower_id, followee_id)`. | All fields are NOT NULL. `CHECK(follower_id <> followee_id)` stops users following themselves. The combined primary key prevents duplicate follows. |

Both tables use `CURRENT_TIMESTAMP` as the default for `created_at`.

### How the relationships work

I decided to store each friendship only once. For example, a friendship between users 7 and 8 is stored as `(7, 8)`, regardless of who sent the request.

The `requested_by` field identifies the sender. This lets the API check that only the other user can accept or decline the request.

Following is different because it only goes in one direction. User 7 can follow user 8 without user 8 having to follow them back.

## Endpoint Inventory

All endpoints use the shared error response format:

`{"error":{"code":422,"message":"Invalid value","field":"field_name"}}`

The IDs and usernames below are examples. They will need to match the seed data when the API is implemented.

### 1. Send Friend Request

**FR:** FR10, FR8  
**Priority:** Must  
**Route:** `/friends/requests`  
**Method:** POST  
**Description:** Sends a friend request to another user.  
**Users:** Logged-in players

**Happy input:**

Logged in as user 7.

`POST /friends/requests`

`{"user_id":8}`

**Happy output:**

201 Created

`{"id":14,"user_low_id":7,"user_high_id":8,"requested_by":7,"status":"pending"}`

**Sad inputs and outputs:**

- No authentication token: 401, Authentication required.
- Missing `user_id`: 400, user_id is required.
- Sending a request to yourself: 422, Cannot send a friend request to yourself.
- Sending a request to user 99999: 404, User not found.
- Sending another request when a pending request or friendship already exists: 409, Relationship already exists.

A failed request must not add a friendship record.

### 2. Accept Friend Request

**FR:** FR10, FR8  
**Priority:** Must  
**Route:** `/friends/requests/{id}`  
**Method:** PATCH  
**Description:** Accepts an existing friend request.  
**Users:** Only the user who received the request

**Happy input:**

User 8 is logged in and has received request 14.

`PATCH /friends/requests/14`

`{"action":"accept"}`

**Happy output:**

200 OK

`{"id":14,"user_low_id":7,"user_high_id":8,"requested_by":7,"status":"accepted"}`

**Sad inputs and outputs:**

- No token: 401, Authentication required.
- Missing action: 400, action is required.
- Invalid action: 422, Invalid action.
- Request 99999 does not exist: 404, Friend request not found.
- User 7 tries to accept their own outgoing request: 403, Not allowed to accept this request.
- Request has already been accepted: 409, Request already accepted.

Only the recipient can accept the request. The existing friendship row is updated rather than creating another one.

### 3. Decline Friend Request

**FR:** FR10, FR8  
**Priority:** Must  
**Route:** `/friends/requests/{id}`  
**Method:** DELETE  
**Description:** Declines a pending request by deleting it.  
**Users:** Only the recipient

**Happy input:**

User 8 is logged in.

`DELETE /friends/requests/14`

**Happy output:**

204 No Content

**Sad inputs and outputs:**

- No token: 401, Authentication required.
- Request does not exist: 404, Friend request not found.
- Sender tries to decline the request: 403, Not allowed to decline this request.
- Request is already accepted: 409, Cannot decline an accepted friendship.

Declining removes the pending request but does not affect other user data.

### 4. List Friends

**FR:** FR10  
**Priority:** Must  
**Route:** `/users/{id}/friends`  
**Method:** GET  
**Description:** Lists accepted friends, with pagination.  
**Users:** Anyone

**Happy input:**

`GET /users/7/friends?page=1&size=20`

**Happy output:**

200 OK

`{"items":[{"id":8,"username":"mira_plays"}],"page":1,"size":20,"total":1}`

**Sad inputs and outputs:**

- `page=abc`: 400, page must be an integer.
- `page=0`: 422, page must be at least 1.
- `size=500`: 422, size must not exceed 100.
- User 99999 does not exist: 404, User not found.

Pending requests are not included in the friends list. A user with no accepted friends gets 200 with an empty list.

### 5. Remove Friend

**FR:** FR10  
**Priority:** Should  
**Route:** `/friends/{user_id}`  
**Method:** DELETE  
**Description:** Removes an accepted friendship.  
**Users:** Either person in the friendship

**Happy input:**

User 7 is logged in.

`DELETE /friends/8`

**Happy output:**

204 No Content

**Sad inputs and outputs:**

- No token: 401, Authentication required.
- Trying to remove yourself: 422, Cannot remove yourself as a friend.
- Unknown user: 404, User not found.
- No accepted friendship exists: 404, Friendship not found.

Removing a friend does not delete either user's account, reviews or collections.

### 6. Follow User

**FR:** FR10, FR8  
**Priority:** Should  
**Route:** `/users/{id}/follow`  
**Method:** PUT  
**Description:** Follows another user without needing approval.  
**Users:** Logged-in players

**Happy input:**

User 7 is logged in.

`PUT /users/8/follow`

**Happy output:**

201 Created

`{"follower_id":7,"followee_id":8}`

Repeating the same request returns 200 with the same relationship, rather than creating a duplicate.

**Sad inputs and outputs:**

- No token: 401, Authentication required.
- Unexpected fields in the request body: 400, Unknown field.
- Following yourself: 422, Cannot follow yourself.
- Target user does not exist: 404, User not found.

### 7. Unfollow User

**FR:** FR10  
**Priority:** Should  
**Route:** `/users/{id}/follow`  
**Method:** DELETE  
**Description:** Stops following another user.  
**Users:** Logged-in players

**Happy input:**

User 7 is logged in.

`DELETE /users/8/follow`

**Happy output:**

204 No Content

**Sad inputs and outputs:**

- No token: 401, Authentication required.
- Trying to unfollow yourself: 422, Cannot unfollow yourself.
- Unknown user: 404, User not found.
- No follow relationship exists: 404, Follow relationship not found.

Unfollowing someone does not remove an accepted friendship.

### 8. Activity Feed

**FR:** FR10  
**Priority:** Should  
**Route:** `/feed`  
**Method:** GET  
**Description:** Shows recent collection updates and reviews from friends and followed users.  
**Users:** Logged-in players

**Happy input:**

User 7 is logged in.

`GET /feed?page=1&size=20`

**Happy output:**

200 OK

`{"items":[{"type":"review","user_id":8,"game_id":3,"review_id":12,"rating":9,"occurred_at":"2026-11-17T10:08:00Z"}],"page":1,"size":20,"total":1}`

**Sad inputs and outputs:**

- No authentication token: 401, Authentication required.
- `page=abc`: 400, page must be an integer.
- `page=0`: 422, page must be at least 1.
- `size=500`: 422, size must not exceed 100.

The feed sorts activities from newest to oldest.

If someone is both a friend and followed, their activity should only appear once.

The feed reads from the existing `collection_entries` and `reviews` tables. It does not have its own activity table.

This means it can display the latest stored changes but not a full history of every previous update.

## Individual Design Decisions

### 1. Storing Friendships Once

I chose to store one row for each pair of users instead of having two separate rows, one for each direction.

The alternative could cause problems if one row was updated but the other was not. For example, one side might say the users are friends while the other still says the request is pending.

Using `user_low_id` and `user_high_id`, together with a UNIQUE constraint, avoids that problem.

### 2. Keeping Friends and Follows Separate

I used two tables because friendship and following work differently.

Friends have to accept a request, but following is one-way and does not need approval.

I considered storing both relationships in one table, but that would make the status checks more complicated.

### 3. Using PUT for Following

I chose PUT because the relationship has a specific URL and sending the same request again should not create another record.

POST was another option, but it would need extra handling for duplicate requests. PUT matches how the group is handling similar operations.

### 4. Accepting and Declining Requests

I used PATCH to accept a request because it changes the status from pending to accepted.

For declining, I used DELETE because the pending request is removed completely.

This avoids keeping declined relationships that the system does not currently need.

### 5. Activity Feed Without a Separate Table

I decided to use the collections and reviews tables for the feed.

This avoids storing the same information twice, although the query will be more complicated.

The disadvantage is that the feed cannot show every previous status change or activity that has since been deleted.

A separate activity history table would be needed if we wanted that functionality in the future.

## Dependencies

My database tables depend on the users table from Ayaan's section.

The friend request endpoints will also use the shared authentication and error handling.

The feed depends on Lee's `collection_entries` table and Zeeshan's `reviews` table, so that part will be implemented once the shared database is available.

The endpoint examples and error messages are part of the proposed D2 design. The final responses and tests will be checked during implementation.

## D2 Testing Plan

For my section, I plan to test:

- Sending and accepting a friend request.
- Declining a pending request.
- Checking that pending requests do not appear in the accepted friends list.
- Preventing duplicate friend requests.
- Preventing users from sending requests to themselves.
- Making sure only the recipient can accept or decline a request.
- Following and unfollowing users.
- Preventing duplicate follows and self-following.
- Checking the feed order and pagination.
- Making sure a friend who is also followed does not appear twice.
- Checking that failed requests leave the database unchanged.

I will use Bruno to test the API responses and pytest for the main behaviour and database constraints once the implementation is ready.

The results will be recorded during testing rather than assumed at the design stage.
