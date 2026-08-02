---
name: Get insurance quotes for a customer
description: >-
  Use the Super Studio quoting API to generate insurance quotes (life, home, renters,
  landlord) for a customer and return a prefilled link they can use to buy on super.mx.
api: openapi/super-seguros-openapi.json
operations:
  - BobbyWeb.Api.V2.QuoteController.index
---

# Get insurance quotes for a customer

Super Studio (by Super Seguros / super.mx) lets partners quote insurance for their
customers. The customer completes the purchase on super.mx via a prefilled link
returned in each quote.

## Authenticate
- Send your key in the `X-Api-Key` request header on every call.
- Use the sandbox host `https://sandbox.super.mx` while testing and
  `https://app.super.mx` in production. Keys are issued during partner onboarding.

## Request quotes
- Call `POST /api/v2/{product}/quotes` (operationId `BobbyWeb.Api.V2.QuoteController.index`).
- `{product}` is a path parameter — currently `life`.
- Send a JSON `LifeQuoteRequest` body. Required fields: `date_of_birth` (ISO8601) and
  `sex_at_birth` (`male`|`female`). Other fields (`plan_type` = `term` |
  `return_of_premium`, `monthly_income`, `smoker`, `height`, `weight`, `address`,
  contact fields) refine the returned quotes.
- Mexican formats matter: `mobile` is E.164 (`+52` + 13 digits), `rfc` is the tax id
  pattern, addresses use Google-geocoding-style fields.

## Handle the response
- On `200`, read `data.quotes[]`. Each `LifeQuote` has `monthly_premium`,
  `annual_premium`, `term_years`, `coverages`, and a `link` to complete purchase on
  super.mx.
- On `401`, the `X-Api-Key` is missing or invalid — fix auth and retry.
- On `422`, read the `errors[]` envelope (field name -> list of messages) and correct
  the input; do not blind-retry. Errors are `application/json`, not RFC 9457.

## Notes
- No idempotency key is supported; quoting is a read/idempotent-in-effect operation.
- Purchase endpoints (selling directly via API) require separate partner approval.
