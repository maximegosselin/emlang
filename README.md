# Emlang Specification

![License](https://img.shields.io/github/license/emlang-project/spec)
![GitHub release](https://img.shields.io/github/v/release/emlang-project/spec)

## Quick Reference

| Notation | Type      | Description           |
|----------|-----------|-----------------------|
| `/.../`  | Flow      | Business scenario     |
| `%...%`  | Trigger   | Initiates an action   |
| `[...]`  | Command   | Action/intent         |
| `<...>`  | Event     | Past fact             |
| `!...!`  | Exception | Failure/error         |
| `{...}`  | View      | Projection/Read Model |
| `?...?`  | Test      | Given-When-Then test  |

See [SPEC.md](SPEC.md) for the complete specification and [GRAMMAR.ebnf](GRAMMAR.ebnf) for the formal grammar.

## Examples

### User Registration

A complete flow showing a user registering through a form:

```emlang
/RegisterUser/
  %Customer:RegistrationForm%
  [RegisterUser]
    email
    password
  <User:UserRegistered>
    userId
    email
    registeredAt
  {UserProfile}
    userId
    email
```

### E-Commerce Checkout

Multiple sequences within a single flow, representing chronological breaks:

```emlang
/Checkout/
  %Customer:Cart%
  [StartCheckout]
  <Order:CheckoutStarted>
    orderId
    customerId
    items
  {CheckoutSummary}
  ---
  %Customer:CheckoutSummary%
  [SubmitPayment]
    orderId
    paymentMethod
  <Order:PaymentSubmitted>
  <Payment:PaymentProcessed>
  {OrderConfirmation}
  ---
  %Customer:OrderConfirmation%
```

### Multiple Flows with Tests

A document containing related flows and their tests:

```emlang
/RegisterUser/
  %Customer:RegistrationForm%
  [RegisterUser]
    email
    password
  <User:UserRegistered>
    userId
    email
  {UserProfile}
---
/LoginUser/
  %Customer:LoginForm%
  [LoginUser]
    email
    password
  <User:UserLoggedIn>
    userId
    sessionId
  {Dashboard}
---
?RegistrationCreatesProfile?
  [RegisterUser]
    email:alice@example.com
    password:secret123
  <User:UserRegistered>
    userId:user-1
    email:alice@example.com
---
?DuplicateEmailRejected?
  <User:UserRegistered>
    email:alice@example.com
  [RegisterUser]
    email:alice@example.com
    password:newpassword
  !EmailAlreadyUsed!
    email:alice@example.com
```

### Event Storming Notes

Partial flows are valid — useful for early exploration:

```emlang
/OrderLifecycle/
  <OrderPlaced>
  <PaymentReceived>
  <OrderShipped>
  <OrderDelivered>
---
/OrderCancellation/
  <OrderPlaced>
  <OrderCancelled>
  <RefundIssued>
```

## Tools

A linter is currently in development. More tools coming soon.

## Contributing

Open an issue before submitting a pull request.

## License

[MIT](LICENSE)
