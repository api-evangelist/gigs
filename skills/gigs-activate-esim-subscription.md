---
name: Activate an eSIM subscription
description: Create a user and activate a new eSIM connectivity subscription for them on a Gigs plan.
api: Gigs Core API
base_url: https://api.gigs.com
auth: Bearer project API key (Authorization header)
operations:
  - GET /projects
  - GET /projects/{project}/plans
  - POST /projects/{project}/users
  - POST /projects/{project}/subscriptions
source: https://developers.gigs.com/docs/create-a-subscription
---

# Activate an eSIM subscription

Grounded in the documented Gigs Core API operations (HTTP method + path from the public docs). All requests use `Authorization: Bearer ${GIGS_TOKEN}` and `Accept: application/json`.

## Steps

1. **Find your project.** `GET https://api.gigs.com/projects` and read the `id` of the target project (the response is a list envelope: `object: list`, `items[]`, `moreItemsAfter`, `moreItemsBefore`). Use that id as `${GIGS_PROJECT}`.

2. **Select an eSIM-capable plan.** `GET https://api.gigs.com/projects/${GIGS_PROJECT}/plans` and pick a plan whose `simTypes` includes `eSIM`. Hold its `id` (form `pln_...`).

3. **Create the user.** `POST https://api.gigs.com/projects/${GIGS_PROJECT}/users` with a JSON body such as `{"email":"...","fullName":"..."}`. Save the returned user `id` (form `usr_...`).

4. **Create the subscription.** `POST https://api.gigs.com/projects/${GIGS_PROJECT}/subscriptions` referencing the user and the chosen plan. Poll or wait for the `com.gigs.subscription.activated` webhook to confirm the network provider has activated it.

## Rules

- **Auth first:** a missing/invalid token returns `401 unauthenticated` before any other validation (headers are checked first — see conventions/gigs-conventions.yml).
- **Errors:** handle the structured error object `{object:error, type, code, message, hint, documentation}`. Payment failures surface as `code` values like `paymentCardDeclined` (see errors/gigs-decline-codes.yml).
- **Pagination:** when listing, follow `moreItemsAfter` as the next `after` cursor; `limit` max is 200 (default 10).
- **Confirm via events, not polling loops:** subscribe to `com.gigs.subscription.activated` (Svix; verify `webhook-signature`).
