---
name: mocking_v3
description: Mocking rules — where test doubles are appropriate and where they aren't. Mock at system boundaries only, never your own modules. Use this when writing tests that need mocks, fakes, or stubs.
version: 0.1.0
---

# When to Mock

Mock at **system boundaries** only:
- External APIs (payment, email, third-party services)
- Databases (sometimes — prefer a real test DB)
- Time and randomness
- File system (sometimes)

Don't mock:
- Your own classes or modules
- Internal collaborators
- Anything you control

A test that mocks an internal collaborator is testing the test, not the code. The warning sign: a refactor that doesn't change behavior breaks the test. If that happens, the mock was wrong.

## Designing for Mockability

When you do mock at a boundary, the interface shape matters.

### 1. Use dependency injection

Pass external dependencies in rather than creating them internally.

> *Easy to mock*: `function processPayment(order, paymentClient) { ... }`
>
> *Hard to mock*: `function processPayment(order) { const client = new StripeClient(env.STRIPE_KEY); ... }`

### 2. Prefer SDK-style over generic fetchers

Specific functions per external operation, not one generic function with conditional logic.

> *Good* — each function is independently mockable:
> ```
> api = {
>   getUser: (id) => fetch(`/users/${id}`),
>   getOrders: (userId) => fetch(`/users/${userId}/orders`),
>   createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
> }
> ```
>
> *Bad* — mocking requires conditional logic inside the mock:
> ```
> api = {
>   fetch: (endpoint, options) => fetch(endpoint, options),
> }
> ```

The SDK approach gives each mock one specific shape, no conditional logic in test setup, and clear visibility of which endpoints a test exercises.
