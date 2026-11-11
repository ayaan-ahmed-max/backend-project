# Task 1 notes for the group: users, auth and profiles

**Owner:** Ayaan Ahmed

These notes go with `ayaan_d2_rows.md`. They are not part of the D2 document. The point numbers match the earlier draft, so they can be referred to by number.

---

## To confirm with the group

1. **JWT expiry:** tokens expire after 1 hour (`expires_in: 3600`). A request with an expired token gets 401, and the user logs in again. There are no refresh tokens.
2. **Logout:** logout happens on the client, which discards the token. There is no logout endpoint and no server-side token blacklist.
3. **Login identifier:** users log in with email and password, not username.
4. **Login response body:** `access_token`, `token_type`, `expires_in` and `user_id`. The plan only says "200 with a token". `user_id` is included so Bruno can save it for the `/users/{id}` routes.
5. **Unknown email at login:** returns 401 with the same message as a wrong password, not 404, so the API doesn't reveal which emails are registered.
6. **Register response:** returns the new profile (id, username, role, bio, created_at) with no token and no email, to match the rule that the profile never returns the email. The user logs in separately.
7. **The `field` key in the error body:** always present. It names the body field or path parameter at fault, and is `null` when no single field is to blame (401, and 403 for not being the owner). The plan's only example has a field.
8. **Role field:** any `role` field on register or PATCH is rejected, whatever its value (even `"player"` or an invalid value), because the role is never an accepted input. **Updated by fix F10:** register gives 400 ("unknown field: role"), the same as any other field an endpoint doesn't accept, and PATCH gives 403.
9. **Non-numeric id in the path** (e.g. `/users/abc`): 400, following the contract's "value of the wrong type" (like `page=abc`). Flask's default for this would be 404, so it needs handling in the shared code.
10. **Favourites inside the profile:** a plain array of `{"game_id", "title"}` on `GET /users/{id}`, not the paginated list shape, because it is part of one resource and not a list endpoint.
11. **Favourites response bodies:** PUT returns `{"user_id", "game_id"}` for both 201 and 200.
12. **DELETE of a game that is not in the favourites:** 404, not a silent 204.
13. **Profile ownership:** "Owner only" is taken literally, so a maintainer can't edit another user's profile or favourites.
14. **Other fields on PATCH:** **Resolved by fix 4.** `PATCH /users/{id}` accepts only `bio` and `username`. A body containing neither, or containing any other field apart from `role`, gives 400 (e.g. `{"email": "new@example.com"}` gives 400 "unknown field: email"). `role` is the exception: it passes the 400 check and is rejected with 403 at the ownership step. Email and password can't be changed (password reset is in the Won't list).
15. **Column sizes not in the plan:** email VARCHAR(254), password_hash VARCHAR(255), role VARCHAR(10), bio VARCHAR(500). Username VARCHAR(30) comes from the plan. A bio over 500 characters would be a 422.
16. **Username and email rules:** the only username rule is 1 to 30 characters, and the plan gives no allowed character set or minimum length. Whether emails and usernames are unique ignoring case is not specified; I propose storing emails in lower case.
17. **Timestamps:** returned as ISO 8601 in UTC, e.g. `2026-11-17T10:30:00Z`.

---

## Notes for Haarisah (database) and Zeeshan (inventory)

18. **`role` has no NOT NULL in the plan.** **Applied (fix 2).** The plan gives `role CHECK IN ('player','maintainer') default 'player'`. In SQL a CHECK passes on NULL, so a NULL role could be inserted directly. The users row now has `CHECK (role IN ('player', 'maintainer')) NOT NULL DEFAULT 'player'`.
19. **`created_at` has no constraint in the plan.** **Applied (fix 2).** The users row now has `created_at NOT NULL DEFAULT CURRENT_TIMESTAMP`.
20. **`favourite_games.user_id` has no ON DELETE rule.** Only `game_id` is given RESTRICT. No endpoint deletes users, so it doesn't matter yet, but the ERD needs one answer (I propose RESTRICT, the same as game_id).
21. **Order of checks hides the 403 in some cases.** Because 400, 422 and 404 are checked before ownership, a PATCH on another player's profile with an invalid value gets 422, not 403, and a PATCH on a user that doesn't exist gets 404 even for a non-owner. The Bruno 403 tests must therefore use a valid body and an existing user.
22. **403 on register.** **Resolved by fix F10.** The contract's 403 is described as "logged in, but not allowed", and nobody is logged in at registration. A `role` field on register now gives 400 ("unknown field: role"). 403 is kept for a `role` field on `PATCH /users/{id}`, because that user is logged in.
23. **Success code for login.** The shared rule says 200 is for "a read or update" and 201 for "a create". Login is a POST that is neither; the Task 1 table says 200, which I have followed.
24. **Unknown user on the favourites routes.** The plan lists only "favouriting an unknown game (404)". By the order of checks, `PUT /users/99999/favourites/3` would also be 404 (user not found). I have not added it as a row example.

**Applies to every slice: SQLite ignores VARCHAR lengths (from fix 3).** SQLite treats VARCHAR(n) as a label only, so `VARCHAR(30)` does not stop a 31-character value being inserted. Any length limit that the database should enforce needs its own CHECK, e.g. `CHECK (length(username) BETWEEN 1 AND 30)`. The users row now has length CHECKs for username, email and bio. Other slices with length limits (e.g. review body 2000, game title 200) need the same if the limit is meant to hold in the database as well as in the API's 422.
