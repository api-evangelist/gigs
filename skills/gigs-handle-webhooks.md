---
name: Handle Gigs webhooks
description: Receive, verify, and process Gigs com.gigs.* events delivered via Svix.
api: Gigs Core API
base_url: https://api.gigs.com
auth: Bearer project API key (dashboard-managed webhook endpoints)
operations:
  - Configure webhook endpoint (Gigs Dashboard / Svix)
  - Verify webhook-id / webhook-signature / webhook-timestamp
  - Process com.gigs.* event payloads
source: https://developers.gigs.com/docs/core/events/events-webhooks
---

# Handle Gigs webhooks

Grounded in the documented Gigs events & webhooks surface. Gigs delivers webhooks through Svix.

## Steps

1. **Register an endpoint.** In the developers section of the Gigs Dashboard, open "Manage Webhooks" (Svix) and add your HTTPS endpoint URL. Prefer one endpoint for all event types you care about.

2. **Verify every request.** On receipt, verify the `webhook-id`, `webhook-signature`, and `webhook-timestamp` headers — either with a Svix client library or by manual verification. Reject unverified requests.

3. **Dispatch on `type`.** Every event has a `type` starting with `com.gigs.` and a `data` payload. Handle the events you subscribed to, e.g.:
   - `com.gigs.subscription.activated` — a subscription is live.
   - `com.gigs.subscription.renewed` — a subscription renewed.
   - `com.gigs.subscriptionChange.applied` — a scheduled change took effect.
   - `com.gigs.porting.requested` — an inbound number port started.
   - `com.gigs.usageThreshold.exceeded` — an allowance threshold was crossed.

4. **Handle updates precisely.** For `*.updated` events, compare the `previousData` field to `data` to see what changed; prefer business-logic events (e.g. `com.gigs.porting.declined`) over generic `*.updated` where available.

5. **Return 2xx fast.** Respond 2xx quickly; do heavy work asynchronously. Svix retries failures and disables an endpoint after ~5 days of continuous failure.

## Rules

- **Ordering is not guaranteed** — events may arrive out of order; make handlers idempotent on the event `id`.
- **Alerting:** configure Svix failure alerts (at most once/day) and replay failed messages from the Svix logs.
- See asyncapi/gigs-events-webhooks.yml for the captured event catalog.
