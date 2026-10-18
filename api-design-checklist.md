# API Design Checklist

This note is a quick guide for writing backend APIs that are easier to debug, safer to use, and simpler to maintain.

## 1. Keep the contract clear

- Use descriptive resource names and consistent URL patterns.
- Prefer nouns for endpoints: `/users`, `/orders`, `/posts/{id}`.
- Keep HTTP methods semantically correct: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`.

## 2. Validate input early

- Validate request data at the API boundary.
- Reject invalid payloads with clear errors.
- Return 400 for client input issues and 422 when validation is stricter.

## 3. Think about idempotency

- `GET`, `PUT`, and `DELETE` are often expected to be idempotent.
- Repeating a request should not cause duplicate side effects.
- When a client retries a payment or order action, make sure the backend can detect duplicates safely.

## 4. Handle errors consistently

- Use predictable status codes: `200`, `201`, `204`, `400`, `401`, `403`, `404`, `409`, `429`, `500`.
- Return a machine-readable error payload with message and code when useful.
- Avoid leaking internal stack traces or database details to users.

## 5. Secure by default

- Use auth and authorization checks on every protected route.
- Store secrets in environment variables or secret managers.
- Enable HTTPS and secure cookies for authenticated sessions.
- Add rate limiting and request-size limits where needed.

## 6. Make it observable

- Log meaningful request IDs and correlation IDs.
- Track latency, error count, and throughput.
- Add health endpoints and basic metrics for monitoring.

## 7. Design for scale

- Keep business logic separate from transport code.
- Use pagination for large collections.
- Add caching only where it helps and where invalidation is clear.
- Move long-running work to background jobs when needed.

## 8. Practical checklist

- Does every route have clear auth and permission rules?
- Are validation and error handling consistent?
- Is the API safe to retry?
- Are logs and metrics useful when debugging?
- Can the system recover from partial failures without corrupting state?

## Common mistakes

- Returning 200 for failed actions.
- Accepting user input without validation.
- Ignoring idempotency and duplicate requests.
- Logging secrets or sensitive user data.
- Mixing database logic with API handlers.

## Quick takeaway

A good backend API is predictable, safe, and observable. If a client can understand the contract and a developer can reason about failures, the API is usually in a healthy state.
