# DummyJSON API Test Automation

An API test automation suite built with Postman, covering authentication, full CRUD operations, negative testing, and an end-to-end purchase flow against the [DummyJSON](https://dummyjson.com/docs) REST API. The suite runs locally via Postman or headlessly via Newman, and is integrated into GitHub Actions for continuous testing on every push.

## Tech Stack

- **Postman** - request building and test scripting
- **JavaScript** - test assertions (Postman's built-in Chai-based test framework)
- **Newman** - command-line collection runner
- **GitHub Actions** - CI pipeline

## Project Structure
```
qa-api-automation/
├── collections/
│   └── DummyJSON API.postman_collection.json
├── environment/
│   └── DummyJSON Environment.postman_environment.json
├── .github/
│   └── workflows/
│       └── tests.yml
└── README.md
```
## Test Coverage

The collection is organized into the following folders:

- **Authentication** - login, token capture, and token refresh
- **Products** - full CRUD, search, and validation scenarios
- **Carts** - full CRUD, filtering, sorting, and merge-behavior scenarios
- **Users** - full CRUD, nested-field filtering, sorting, and relationship endpoints
- **Comments** - full CRUD and field-selection scenarios
- **Negative Tests** - invalid/missing credentials, missing authorization, and not-found/empty-result handling across resources
- **E2E - Purchase Flow** - a chained scenario simulating a real user session: login → discover the authenticated user's existing cart and product data → update the cart → delete the cart

### Testing techniques demonstrated

- Status code, response time, and JSON content-type validation (response time/content-type applied globally via collection-level scripts)
- Schema and type assertions
- Dynamic variable capture and request chaining
- Array validation and cross-item consistency checks
- Filter/search correctness (not just "got a result," but "got the *right* result")
- Nested object validation
- Business-logic assertions comparing two fields against each other (e.g. discounted total vs. total)
- Sort-order verification
- Negative/security-style assertions (e.g. confirming a field is *not* present)
- Field allow-listing (`select` parameter)
- Soft-delete behavior (`isDeleted` / `deletedOn`, since DummyJSON does not perform hard deletes)
- End-to-end request chaining using real, dynamically discovered data

## Running Locally

### With Postman
1. Import `collections/DummyJSON API.postman_collection.json`
2. Import `environment/DummyJSON Environment.postman_environment.json`
3. Select the environment, run `Authentication / Login` first to populate the access token, then run the collection

### With Newman
npm install -g newman
newman run "collections/DummyJSON API.postman_collection.json" -e "environment/DummyJSON Environment.postman_environment.json"
## Continuous Integration

A GitHub Actions workflow (`.github/workflows/tests.yml`) runs the full collection via Newman on every push and pull request to `main`.

## Known Issues / API Quirks

These are documented findings from testing against DummyJSON's real behavior, not defects in this test suite:

- **Refresh Token test may show as failing in CI/Newman.** The `Authentication / Refresh Token` test checks that a newly issued access token differs from the previous one. DummyJSON's JWTs use second-level timestamp precision, so when `Login` and `Refresh Token` execute in rapid succession (as they do in Newman/CI), they can produce an identical token. This is expected and does not indicate a functional problem with the API or the test.
- **`/auth/me` accepts authentication via cookie, not just Bearer token.** Testing the unauthenticated case for this endpoint requires disabling Postman's cookie jar for that specific request (`Settings → Disable Cookie Jar`), since a session cookie set during a prior `Login` will otherwise silently authenticate the request even with no Authorization header.
- **DummyJSON's write endpoints (POST/PUT/DELETE) do not persist data.** An id returned from a `POST` request cannot be used in a later, separate request (e.g. to fetch or delete it) — nothing was actually stored. This shaped the design of the E2E flow, which discovers and acts on the authenticated user's *existing* real cart data rather than attempting to verify a freshly created resource.
