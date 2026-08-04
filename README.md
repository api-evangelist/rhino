# Rhino

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Rhino (SayRhino) is a New York based insurtech that replaces the traditional cash security
deposit with low-cost security deposit insurance, alongside cash deposit management, renters
insurance and a renter guarantee product. Founded in 2017 and now operating alongside Jetty,
Rhino sells through property owners and managers across the United States and integrates with
the major property management systems (Yardi, RealPage, Entrata, Rent Manager, MRI, Salesforce).

- Website: https://www.sayrhino.com/
- API reference: https://api.prod.sayrhino.com/docs
- OpenAPI: https://api.prod.sayrhino.com/openapi.json
- Partner portal: https://portal.sayrhino.com/users/sign_in
- GitHub: https://github.com/sayrhino
- Secondary market listing: https://forgeglobal.com/rhino_stock/

## API

**SayRhino Partner API v2** — OpenAPI 3.0.3, 10 paths / 12 operations, served live from the API
host root at `https://api.prod.sayrhino.com/openapi.json` with a Redoc reference at `/docs`.
Partners authenticate with an OAuth 2.0 client-credentials grant (`POST /token`) and use a JWT
bearer token against owner-scoped paths to upsert insurance prospects, read eligibility offers
and coverage, and manage webhook endpoints across fifteen prospect, policy, policy-application
and delinquency events. Credentials are issued through partner onboarding — there is no public
developer sign-up.
