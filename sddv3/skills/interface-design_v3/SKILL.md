---
name: interface-design_v3
description: Interface design rules for testability — accept dependencies, return results, keep surfaces small. Use this when designing components to ensure they are naturally testable. Loaded by the plan_v3 skill at design time.
version: 0.1.0
---

# Interface Design for Testability

Three rules that make components naturally testable.

## 1. Accept dependencies, don't create them

Pass external dependencies in. Don't construct them inside the function.

> *Testable*: `function processOrder(order, paymentGateway) { ... }`
>
> *Hard to test*: `function processOrder(order) { const gateway = new StripeGateway(); ... }`

The first lets a test substitute a fake gateway. The second buries the dependency where no test can reach it.

## 2. Return results, don't produce side effects

Prefer functions that return values over functions that mutate.

> *Testable*: `function calculateDiscount(cart): Discount { ... }`
>
> *Hard to test*: `function applyDiscount(cart): void { cart.total -= discount; }`

The first is tested by asserting on the return value. The second forces tests to inspect mutated state, which couples them to the internal shape.

## 3. Small surface area

Fewer methods means fewer tests. Fewer parameters means simpler test setup. A component with one public method and three parameters is dramatically easier to test than one with six methods and a dozen parameters. Keep contracts minimal.
