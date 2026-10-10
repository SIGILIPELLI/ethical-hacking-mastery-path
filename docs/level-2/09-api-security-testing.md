---
description: "Testing REST and GraphQL APIs: authentication and tokens, broken object-level authorization, mass assignment, rate limiting and data exposure."
---

# API Security Testing

Modern apps are mostly APIs — a thin front end calling JSON endpoints that do the real work. APIs
have their own top-risk list (the OWASP **API Security Top 10**) because they fail in
characteristic ways: they expose objects directly, trust the client too much, and often lack the
UI's accidental guard-rails. This lesson covers testing REST and GraphQL APIs. The web-app
principles (trust boundary, access control, injection) all still apply; this is the API-shaped
version. Test APIs you run.

## Why APIs fail differently

- There's no UI hiding endpoints — every operation is a documented (or discoverable) HTTP call.
- Clients are often trusted to send object IDs, roles and even whole objects.
- Auth is token-based (API keys, JWTs, OAuth), with its own pitfalls.
- Automated clients make rate limiting and resource exhaustion central.
- Responses frequently return *more* data than the UI shows, assuming the client will filter.

## Mapping the API

Before testing, enumerate the endpoints:

- **Documentation** — Swagger/OpenAPI (`/swagger.json`, `/openapi.json`, `/v2/api-docs`), GraphQL
  introspection, Postman collections. Often exposed and a complete map.
- **Traffic** — proxy the app's own front end and watch the API calls it makes.
- **Patterns** — REST resources are predictable (`/users/{id}`, `/orders/{id}`); try the siblings.

```bash
# Is an OpenAPI spec exposed? (lab target)
curl -s http://api.lab/openapi.json | head
```

## The high-impact API tests

### Broken Object-Level Authorization (BOLA / API-IDOR)

The #1 API risk — the IDOR of Level 2 lesson 6, applied to API objects. Change the object ID in an
API call and see if you get an object you don't own:

```bash
# As USER A's token, request USER B's object
curl -s -H "Authorization: Bearer <USER_A_TOKEN>" http://api.lab/api/users/1002
```

If A's token returns user 1002's data, that's BOLA. APIs are especially prone because the ID is
right there in the path and there's no UI to constrain you.

### Broken authentication / token handling

- **JWT pitfalls:** is the signature actually verified? Try tampering with the payload (flip
  `"role":"user"` to `"admin"`) and resending — a server that doesn't verify the signature, or
  accepts `alg: none`, will honour it. Are tokens expired and revocable?
- **API keys in the client** — keys embedded in mobile/JS clients are extractable; test what one
  key can do.
- **Missing auth** — some endpoints simply forget to require a token. Call them without one.

### Broken Object Property-Level Authorization / Mass Assignment

APIs often bind a whole JSON body onto an object. If the app blindly accepts every field, you can
set properties you shouldn't:

```bash
# The form only lets you change your name; the API accepts more:
curl -s -X PATCH -H "Authorization: Bearer <TOKEN>" -H 'Content-Type: application/json' \
  -d '{"name":"Alice","isAdmin":true,"accountBalance":999999}' http://api.lab/api/users/me
```

If `isAdmin` or `accountBalance` is honoured, that's **mass assignment** — the server bound
attacker-controlled fields it should have ignored. The inverse, **excessive data exposure**, is the
API returning sensitive fields (password hashes, tokens, other users' PII) and expecting the client
to hide them.

### Rate limiting & resource consumption

Without limits, an attacker can brute-force, scrape, or exhaust resources. Test whether endpoints
throttle rapid repeated calls (as in the auth lesson). For GraphQL, deeply **nested queries** can be
a denial-of-service amplifier (below).

### Injection, still

APIs are just as injectable as web pages — the JSON value `{"name": "'; DROP TABLE..."}` reaches the
same database. Test API parameters for SQL/NoSQL/command injection exactly as in lesson 3.

## GraphQL specifics

GraphQL puts one endpoint in front of a typed schema, which changes testing:

- **Introspection** — if enabled, a single query returns the entire schema (every type, field and
  mutation). Great for mapping; often should be disabled in production.
- **Authorization per field/resolver** — each field can leak independently; BOLA/BFLA must be
  checked per field, not per endpoint.
- **Query depth/complexity** — a maliciously nested query (`a { b { a { b … }}}`) can force huge
  work. Test whether depth/complexity limits exist.
- **Batching** — some servers accept arrays of operations, which can bypass per-request rate limits.

The deeper GraphQL mechanics (resolvers, N+1, DataLoader, federation) are covered in the
[GraphQL Mastery Path](https://sigilipelli.github.io/graphql-mastery-path/); here the focus is the
security surface.

## The defences

- **Enforce object- and property-level authorization server-side on every endpoint and field** —
  the BOLA/mass-assignment fix is the same deny-by-default ownership check from lesson 6.
- **Verify JWT signatures properly**; reject `alg: none`; keep tokens short-lived and revocable;
  don't trust client-sent roles.
- **Bind only allow-listed fields** (explicit DTOs / serializers), never the whole request body;
  return only the fields the caller needs (no excessive exposure).
- **Rate-limit and throttle**; for GraphQL, add depth/complexity limits and control batching.
- **Disable introspection and verbose errors in production**; don't expose internal specs publicly.

## How It Actually Works

Why do APIs suffer *more* from authorization bugs than the server-rendered web apps they replaced?
Because moving logic to the client removed a layer of accidental protection and shifted trust in the
wrong direction. In a classic server-rendered app, the server decided what HTML you got — it *had*
to make an access decision to render your page, so authorization was at least on the happy path. An
API, by design, is a set of primitive operations the client composes. The client asks for object
1002; the API's job, as the developer often conceives it, is to *return object 1002*. The access
decision ("may this caller have 1002?") is an extra step that the primitive operation doesn't
inherently require to function — so it gets skipped, and the endpoint works perfectly for honest
callers who only ever request their own IDs. It is the same invisibility that makes web IDOR
common, intensified because APIs expose the raw object references directly and in bulk.

Mass assignment and excessive data exposure are two faces of the same convenience: frameworks make
it trivial to map a whole JSON object to a database row and back. Binding the entire request body
onto the model (mass assignment) and serialising the entire model into the response (excessive
exposure) are one line of code each, and both silently include fields the developer never meant to
expose on this endpoint. The vulnerability is the *gap between the object's full shape and the shape
this caller should see or set* — and because the easy code path ignores that gap, the easy code path
is insecure. The fix in every case is to make the trust boundary explicit again: a per-caller,
per-field decision about what may be read and written, re-stated on the server for every operation,
because — as always — the client's request is untrusted and only the server knows the true
authorization and the true safe shape of the data.

## Common mistakes and pitfalls

- **Testing only what the front end calls.** The API usually has more endpoints and fields than the
  UI uses. Enumerate from the spec/introspection, not just from traffic.
- **Assuming a JWT is verified.** Always test tampering and `alg: none`; plenty of servers decode
  without verifying.
- **Missing mass assignment because the form hides the field.** The API may accept fields the form
  never sends. Add them to the JSON and check.
- **Ignoring excessive data exposure** because the UI hides the extra fields — the raw response
  still leaked them to anyone who reads it.
- **Forgetting GraphQL authorization is per-field.** One endpoint, many independent leak points.
- **Running automated API fuzzers against systems you don't own, or without RoE approval.** Lab and
  authorised only; API fuzzing is noisy and can be destructive.

## Exercise

1. Find and retrieve a lab API's OpenAPI/Swagger spec (or run GraphQL introspection). List the
   endpoints/fields the UI does *not* use.
2. Perform a BOLA test: with user A's token, request one of user B's objects. Record the result.
3. Attempt mass assignment: add a privileged field (`isAdmin`, a balance, a role) to a PATCH/POST
   body and check whether the server honours it.
4. Take a JWT from a practice app, decode it, flip a claim, and resend. Does the server detect the
   tampering? Explain what that tells you about signature verification.
5. Explain, in your own words, why moving from server-rendered pages to APIs tends to *increase*
   authorization bugs, referencing where the access decision used to live.
