# Emlang Specification

![License](https://img.shields.io/github/license/emlang-project/spec)
![GitHub release](https://img.shields.io/github/v/release/emlang-project/spec)

Emlang is a YAML-based DSL for describing systems with Event Modeling patterns.

## Quick Reference

| Type      | Short | Acronym | Long         | Description            |
|-----------|-------|---------|--------------|------------------------|
| Trigger   | `t:`  | `trg:`  | `trigger:`   | Initiates an action    |
| Command   | `c:`  | `cmd:`  | `command:`   | Action/intent          |
| Event     | `e:`  | `evt:`  | `event:`     | Past fact              |
| Exception | `x:`  | `err:`  | `exception:` | Failure/error          |
| View      | `v:`  | —       | `view:`      | Projection/Read Model  |

See [SPEC.md](SPEC.md) for the complete specification and [schema.json](schema.json) for validation.

## Examples

### User Registration

A complete slice showing a user registering through a form:

```yaml
---
slices:
  RegisterUser:
    - t: Customer/RegistrationForm
    - c: RegisterUser
      props:
        email: string
        password: string
    - e: User/UserRegistered
      props:
        userId: uuid
        email: string
        registeredAt: iso8601
    - v: UserProfile
```

### E-Commerce Checkout

Multiple slices representing the checkout process:

```yaml
---
slices:
  StartCheckout:
    - t: Customer/Cart
    - c: StartCheckout
    - e: Order/CheckoutStarted
      props:
        orderId: uuid
        customerId: uuid
        items: array
    - v: CheckoutSummary

  SubmitPayment:
    - t: Customer/CheckoutSummary
    - c: SubmitPayment
      props:
        orderId: uuid
        paymentMethod: string
    - e: Order/PaymentSubmitted
    - e: Payment/PaymentProcessed
    - v: OrderConfirmation
```

### Slice with Attached Tests (Extended Form)

Tests are attached directly to a slice using the extended form with `steps:` and `tests:`:

```yaml
---
slices:
  RegisterUser:
    steps:
      - t: Customer/RegistrationForm
      - c: RegisterUser
        props:
          email: string
          password: string
      - e: User/UserRegistered
        props:
          userId: uuid
          email: string
      - v: UserProfile
    tests:
      RegistrationCreatesProfile:
        when:
          - c: RegisterUser
            props:
              email: alice@example.com
              password: secret123
        then:
          - e: User/UserRegistered

      DuplicateEmailRejected:
        given:
          - e: User/UserRegistered
            props:
              email: alice@example.com
        when:
          - c: RegisterUser
            props:
              email: alice@example.com
        then:
          - x: EmailAlreadyUsed
```

### Event Storming Notes

Partial slices are valid — useful for early exploration:

```yaml
---
slices:
  OrderLifecycle:
    - e: OrderPlaced
    - e: PaymentReceived
    - e: OrderShipped
    - e: OrderDelivered

  OrderCancellation:
    - e: OrderPlaced
    - e: OrderCancelled
    - e: RefundIssued
```

## Tools

A linter is currently in development. More tools coming soon.

## Contributing

Open an issue before submitting a pull request.

## License

[MIT](LICENSE)
