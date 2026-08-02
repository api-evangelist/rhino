# Rhino

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
