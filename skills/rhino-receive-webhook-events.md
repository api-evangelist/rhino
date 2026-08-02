---
name: Receive and reconcile Rhino webhook events
description: Register an HTTPS webhook endpoint on the SayRhino Partner API, subscribe to the
  prospect, policy, policy-application and delinquency events, then inspect and retry failed
  deliveries.
api: openapi/rhino-partner-api-openapi.json
base_url: https://api.prod.sayrhino.com
operations:
- POST /token
- POST /partners/{owner_slug}/webhooks/endpoints
- GET /partners/{owner_slug}/webhooks/endpoints
- PUT /partners/{owner_slug}/webhooks/endpoints/{id}
- DELETE /partners/{owner_slug}/webhooks/endpoints/{id}
- GET /partners/{owner_slug}/webhooks/endpoints/{endpoint_id}/deliveries
- GET /partners/{owner_slug}/webhooks/endpoints/{endpoint_id}/deliveries/{id}
- POST /partners/{owner_slug}/webhooks/endpoints/{endpoint_id}/deliveries/{id}/retry
generated: '2026-08-02'
method: generated
source: openapi/rhino-partner-api-openapi.json
---

# Receive and reconcile Rhino webhook events

Rhino's webhook surface is API-managed: you create endpoints, pick events, and inspect
every delivery attempt through the same Partner API. Use it instead of polling prospects.

## 1. Register an endpoint

`POST /partners/{owner_slug}/webhooks/endpoints`

```json
{ "endpoint": { "name": "prod-listener", "destination_url": "https://example.com/hooks/rhino",
  "events": ["prospect.ready", "policy.created"] } }
```

`destination_url` **must** be HTTPS — plain HTTP returns **422**
`{"endpoint":{"destination_url":"Destination url must use HTTPS"}}`. The response carries
the endpoint `id`, its `status` (`active` | `suspended` | `disabled`), a
`suspension_reason`, and timestamps. Store the `id`.

## 2. Choose the events

Fifteen events across four domains:

- **prospect** — `prospect.created`, `prospect.updated`, `prospect.ready`, `prospect.incomplete`
- **policy_application** — `policy_application.created`, `policy_application.quoted`,
  `policy_application.submitted`, `policy_application.declined`
- **policy** — `policy.created`, `policy.updated`, `policy.canceled`, `policy.expired`,
  `policy.documents_updated`
- **delinquency** — `delinquency.created`, `delinquency.resolved`

Subscribe narrowly. `prospect.ready` is what replaces polling in the *Enroll a renter
prospect* skill; `policy.created` and `policy.canceled` are what keep a property
management system's coverage state in sync.

## 3. Handle the payload

Payloads are `{"event": "<name>", "data": { ... }}` — for example
`{"event":"policy.created","data":{"insurance_policy_id":1}}`. The `data` object carries
identifiers, not the full record; **re-read the resource** after receiving an event.

Rhino publishes **no signature or HMAC verification scheme**. Do not trust the payload
as authorization. Use an unguessable destination path, allowlist by source where you
can, and treat every payload as a hint to go re-read state over the authenticated API.

Return 2xx quickly and process asynchronously — a slow endpoint gets throttled
(deliveries land in `rate_limited`) and a persistently failing one gets the endpoint
`suspended` with a `suspension_reason`.

## 4. Reconcile failures

`GET /partners/{owner_slug}/webhooks/endpoints/{endpoint_id}/deliveries?status=failed&page=1&per_page=50`

Delivery `status` is one of `pending`, `delivering`, `delivered`, `retrying`,
`rate_limited`, `failed`. Each row carries `attempt_number`, `payload`, `duration_ms`,
`last_attempt_failure_reason`, `last_attempted_at`, `delivered_at` and `next_retry_at`.

`GET .../deliveries/{id}` for one attempt in full;
`POST .../deliveries/{id}/retry` to re-enqueue it — returns **204**.

**Retry is not idempotent.** Each call queues another attempt. Only retry after your
endpoint is confirmed healthy, and drive it from the failed-delivery list, never from a
blind loop.

## 5. Rotate or retire an endpoint

`PUT /partners/{owner_slug}/webhooks/endpoints/{id}` updates `destination_url`, `events`
or `status` (set `suspended` to pause without losing configuration).
`DELETE /partners/{owner_slug}/webhooks/endpoints/{id}` removes it permanently and
**cancels all pending deliveries** — drain the failed list first.

## Rules

- HTTPS only, always.
- No signature verification exists — re-read, never trust the payload's contents.
- List deliveries with `status` + pagination; there is no total count or next link.
- A 404 on an endpoint or delivery usually means a wrong `owner_slug`, not a deleted resource.
