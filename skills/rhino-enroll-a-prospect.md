---
name: Enroll a renter prospect with Rhino
description: Authenticate to the SayRhino Partner API, upsert a prospect for security-deposit-insurance
  eligibility, then poll until Rhino has priced the offer and return the enrollment URLs.
api: openapi/rhino-partner-api-openapi.json
base_url: https://api.prod.sayrhino.com
operations:
- POST /token
- POST /partners/{owner_slug}/prospects
- GET /partners/{owner_slug}/prospects/{source}/{source_prospect_id}
generated: '2026-08-02'
method: generated
source: openapi/rhino-partner-api-openapi.json
---

# Enroll a renter prospect with Rhino

Rhino's Partner API turns a renter application into a priced security-deposit-insurance
offer. This is the core flow: mint a token, submit the prospect, wait for pricing, hand
the renter an enrollment URL.

## Before you start

You need four things from Rhino partner onboarding, none of which are self-service:
`client_id`, `client_secret`, the `audience` identifier, and the `owner_slug` for the
property owner you are acting for. The `owner_slug` scopes **every** path.

## 1. Get an access token

`POST /token` — the only public operation.

```json
{ "grant_type": "client_credentials", "client_id": "...", "client_secret": "...", "audience": "..." }
```

Returns `access_token`, `token_type: Bearer`, `expires_in` (seconds) and `scope`. Cache
it until `expires_in` elapses; every other call needs
`Authorization: Bearer <access_token>`. A bad credential returns **401**
`{"error":"access_denied","error_description":"Unauthorized"}`.

## 2. Upsert the prospect

`POST /partners/{owner_slug}/prospects` with `Content-Type: application/json`.

Only two fields are required: `source` and `source_prospect_id`. Everything else
improves the eligibility decision — `first_name`, `last_name`, `email`, `phone`,
`birthdate`, `has_ssn`, `social_security_number`, `citizenship`
(`us_or_green_card` | `non_us`), `employment_status`, `education_level`,
`yearly_income`, `monthly_rent_cents`, `deposit_amount_cents`, `lease_start_date`,
`lease_end_date`, `screening_result` (`approved` | `conditional` | `denied`),
`terms_accepted_at`, and the property-management identifiers `pms_type` (`yardi`),
`pms_property_id`, `pms_unit_id`, `pms_prospect_id`.

**All money is integer cents.** `monthly_rent_cents: 200000` is $2,000.00.

**This call is safe to retry.** `(source, source_prospect_id)` is a natural key scoped
to the owner — resubmitting the same pair updates the existing prospect rather than
creating a duplicate. There is no `Idempotency-Key` header, and none is needed here.

A **422** returns per-field messages under a `prospect` key, e.g.
`{"error":"Prospect invalid.","prospect":{"source":"Source is not included in the list"}}`.
`source` must be one of the partner sources Rhino provisioned for that owner — fix the
value, do not retry the same body.

## 3. Poll until the prospect is priced

The create response usually comes back with `status: "processing"` and
`offered_products` carrying `null` enrollment URLs. Read it back with
`GET /partners/{owner_slug}/prospects/{source}/{source_prospect_id}` until `status`
reaches a terminal value:

| `status` | meaning | do |
|---|---|---|
| `incomplete` | Rhino needs more prospect data | resubmit step 2 with more fields |
| `processing` | pricing in flight | poll again |
| `ready` | offers priced | go to step 4 |
| `error` | Rhino could not evaluate | escalate to Partner Success |

Prefer subscribing to the `prospect.ready` and `prospect.incomplete` webhook events over
tight polling — see the *Receive Rhino webhook events* skill.

## 4. Read the offers

`offered_products[]` carries one entry per `product_type`:

- `security_deposit_insurance` — `monthly_premium_cents` + `monthly_enrollment_url`, and
  `upfront_premium_cents` + `upfront_enrollment_url`. Present both; let the renter choose.
- `cash_deposit` — `cash_deposit_amount_cents` + `enrollment_url`.

`effective_coverage_amount_cents` is what the owner is covered for, and
`percent_savings` is the headline saving versus the cash deposit. Send the renter to the
enrollment URL — the API does not complete enrollment for you. `application_submitted`
flips to `true` once they do.

## Rules

- Never invent an `owner_slug` or a `source`; both are provisioned by Rhino.
- Never log the request body — it carries SSN, birthdate and income.
- Treat `enrollment_url` as short-lived and re-read the prospect rather than caching it.
- Every error body is `{error, status_code}` plus, on 422, a per-field object. There are
  no stable machine-readable error codes — match on HTTP status, not on message text.
