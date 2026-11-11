# Shared checks proposal (Task 1, Ayaan)

This is the shared code I will build by **Fri 20 Nov**. Please agree the names now, so that you can write your D2 rows and plan your routes against them before the code exists. Nothing here is implemented yet, and the examples only show how a route would use each one.

Reply in the group chat by **Thu 12 Nov at 12:00** if you want a name or behaviour changed. After that, I will treat these as fixed.

## The four pieces

| Name | What it does | On failure | How your slice uses it |
|---|---|---|---|
| `@login_required` | Checks that the request carries a valid token and makes the logged-in user available to your route as `g.current_user` (with `id` and `role`). | 401 | Put `@login_required` on the line above any route that changes data, e.g. above `PUT /users/{id}/collection/{game_id}`. |
| `@maintainer_required` | Checks that the request carries a valid token and that the logged-in user is a maintainer. It includes the login check, so you don't need both. | 401 if not logged in; 403 if logged in as a player | Put `@maintainer_required` above any maintainer-only route, e.g. above `PATCH /games/{id}`. |
| `check_owner(owner_id, allow_maintainer=False)` | Checks that the logged-in user is the user the record belongs to; with `allow_maintainer=True`, a maintainer also passes. | 403 | Call it inside the route after you have found the record, e.g. `check_owner(review.user_id)` for "author only", or `check_owner(post.user_id, allow_maintainer=True)` for "author or maintainer". |
| `error(code, message, field=None)` | Builds the shared error response, `{"error": {"code": ..., "message": ..., "field": ...}}`, with the matching status code. | Not applicable (it is the failure response) | Return it from any failed check in your route, e.g. `return error(422, "rating must be between 1 and 10", "rating")`. |

## How they fit the order of checks

| Step in the plan's order | Status | Who does it |
|---|---|---|
| 1. Not logged in | 401 | `@login_required` or `@maintainer_required` |
| 2. A player on a maintainer-only route | 403 | `@maintainer_required` |
| 3. Unreadable request or a missing field | 400 | Your route, using `error(400, ...)` |
| 4. Invalid value | 422 | Your route, using `error(422, ...)` |
| 5. Unknown id | 404 | Your route, using `error(404, ...)` |
| 6. Not allowed, e.g. not the owner | 403 | `check_owner(...)` |
| 7. Clash with existing data | 409 | Your route, using `error(409, ...)` |

## Points to agree

1. **The owner check is a function, not a decorator.** The plan describes it as a decorator. A decorator runs before your route, but the plan's order of checks puts the ownership check (step 6) after the 400, 422 and 404 checks, and for reviews and posts the owner is only known once the record has been found. A function called inside the route keeps the agreed order. `@login_required` and `@maintainer_required` stay as decorators, because they are steps 1 and 2.
2. **"Owner only" is the default.** Most routes in the plan are "owner only" or "author only" (profiles, collections, reviews, editing posts). Only deleting posts, threads and user content is "author or maintainer", so those routes pass `allow_maintainer=True`.
3. **`field` is optional.** Leave it out when no single input is to blame (401, and 403 for not being the owner), and the error body will contain `"field": null`.
4. **Until Fri 20 Nov,** build against a fixed test user, as the plan's "if something runs late" section says. Swapping in the real decorators later is a one-line change per route.
