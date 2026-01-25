# Emlang Specification v0.3.0

Emlang is a YAML-based DSL for describing systems with Event Modeling patterns.

## Overview

Emlang uses standard YAML syntax, making it compatible with all YAML tooling (syntax highlighting, formatting,
validation). The domain-specific semantics come from the structure and element prefixes.

An Emlang file contains one or more YAML documents (separated by `---`), each with `flows:` and/or `tests:` top-level
keys.

```yaml
---
flows:
  # Direct form: list of elements
  FlowName:
    - t: Swimlane/TriggerName
    - c: DoSomething
    - e: SomethingDone

  # Extended form: steps with attached tests
  AnotherFlow:
    steps:
      - t: Swimlane/Trigger
      - c: DoAction
      - e: ActionDone
    tests:
      TestName:
        for: DoAction
        when:
          - c: DoAction
        then:
          - e: ActionDone

# Exploratory tests (not attached to a flow)
tests:
  ExploratoryTest:
    given:
      - e: PreviousEvent
    when:
      - c: DoSomething
    then:
      - e: ExpectedEvent
```

## Elements

Emlang has 5 element types, each with multiple prefix forms:

| Type      | Short | Acronym | Long         | Description           |
|-----------|-------|---------|--------------|------------------------|
| Trigger   | `t:`  | `trg:`  | `trigger:`   | Initiates an action    |
| Command   | `c:`  | `cmd:`  | `command:`   | Action/intent          |
| Event     | `e:`  | `evt:`  | `event:`     | Past fact              |
| Exception | `x:`  | `err:`  | `exception:` | Failure/error          |
| View      | `v:`  | —       | `view:`      | Projection/Read Model  |

All prefix forms are semantically equivalent. Choose based on readability preference.

### Element Format

Each element is a YAML list item with a type prefix and optional properties:

```yaml
- c: PlaceOrder
  props:
    orderId: uuid
    items: array
```

### Naming

Element names are free-form text. Spaces, accents, and special characters are allowed:

```yaml
- t: Customer/RegistrationForm
- c: RegisterUser
- e: Utilisateur enregistré
```

### Swimlanes

Swimlanes are optional and specified with `/` before the element name:

```yaml
- t: Customer/RegistrationForm   # Swimlane: Customer
- e: User/UserRegistered         # Swimlane: User
- c: RegisterUser                # No swimlane
```

### Properties

Properties are optional and defined under `props:`. The structure is free-form — consuming tools decide how to interpret
them.

```yaml
- c: RegisterUser
  props:
    email: string
    password: string

- e: User/UserRegistered
  props:
    userId: uuid
    email: string
    registeredAt: iso8601
```

Properties can also be a list:

```yaml
- c: SubmitOrder
  props:
    - orderId: uuid
    - customerId: uuid
    - items
```

## Flows

A flow is a named sequence of elements representing a business scenario.

### Direct Form

When a flow has no attached tests, use the direct form (list of elements):

```yaml
---
flows:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered
    - v: UserProfile
```

### Extended Form

When a flow has attached tests, use the extended form with `steps:` and `tests:`:

```yaml
---
flows:
  UserRegistration:
    steps:
      - t: Customer/RegistrationForm
      - c: RegisterUser
      - e: User/UserRegistered
      - v: UserProfile
    tests:
      EmailMustBeUnique:
        for: RegisterUser
        given:
          - e: User/UserRegistered
            props:
              email: joe@example.com
        when:
          - c: RegisterUser
            props:
              email: joe@example.com
        then:
          - x: EmailAlreadyInUse
```

The `for:` key attaches the test to a specific element in the flow for visual placement in diagrams. It references an element by name within the same flow.

Both forms are valid and can coexist in the same document. A parser detects the form by checking if the flow value is a list (direct) or a mapping with `steps:` (extended).

### Multiple Flows

Multiple flows can be defined in the same document:

```yaml
---
flows:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered

  UserLogin:
    - t: Customer/LoginForm
    - c: AuthenticateUser
    - e: User/UserAuthenticated
    - v: Dashboard
```

### Anonymous Flows

A flow without a name is valid but not referenceable:

```yaml
---
flows:
  - t: Customer/ContactForm
  - c: SubmitInquiry
  - e: InquiryReceived
```

## Tests

A test verifies behavior using a Given-When-Then structure.

Tests can be defined in two places:

| Location | Purpose |
|----------|---------|
| Inside a flow (`tests:` in extended form) | Attached tests, displayed in diagrams |
| At document root (`tests:`) | Exploratory tests, not yet attached to a flow |

Exploratory tests at the root level are useful during early modeling when flows are not yet well-defined. As the model matures, tests can be moved into their respective flows.

### Test Structure

Each test has three sections:

| Section  | Required | Contains                          |
|----------|----------|-----------------------------------|
| `given:` | No       | Pre-conditions (`e:`, `v:`)       |
| `when:`  | Yes      | Single command (`c:`)             |
| `then:`  | Yes      | Expected results (`e:`, `v:`, `x:`) |

```yaml
---
tests:
  EmailMustBeUnique:
    given:
      - e: User/UserRegistered
        props:
          email: joe@example.com
    when:
      - c: RegisterUser
        props:
          email: joe@example.com
    then:
      - x: EmailAlreadyInUse
```

### Attaching Tests to Flow Elements

Tests defined inside a flow (extended form) can use the `for:` key to attach to a specific element for visual placement in diagrams:

```yaml
---
flows:
  UserRegistration:
    steps:
      - t: Customer/RegistrationForm
      - c: RegisterUser
      - e: User/UserRegistered
    tests:
      EmailMustBeUnique:
        for: RegisterUser
        given:
          - e: User/UserRegistered
            props:
              email: joe@example.com
        when:
          - c: RegisterUser
            props:
              email: joe@example.com
        then:
          - x: EmailAlreadyInUse
```

The `for:` key is only valid for tests inside a flow. It references an element by name within the same flow. The diagram generator uses this to position the test visually below the referenced element.

### Test with Exception

```yaml
---
tests:
  RateLimitExceeded:
    given:
      - e: User/PasswordResetRequested
        props:
          userId: 123
          requestedAt: 2024-01-24T10:00:00Z
    when:
      - c: RequestPasswordReset
        props:
          userId: 123
    then:
      - x: TooManyResetAttempts
```

### Test with View

Verify both events and view projections:

```yaml
---
tests:
  ProfileUpdatedAfterRegistration:
    when:
      - c: RegisterUser
        props:
          email: joe@example.com
    then:
      - e: UserRegistered
      - v: UserProfile
        props:
          email: joe@example.com
          status: pending
```

### Multiple Tests

```yaml
---
tests:
  EmailMustBeUnique:
    given:
      - e: UserRegistered
        props:
          email: joe@example.com
    when:
      - c: RegisterUser
        props:
          email: joe@example.com
    then:
      - x: EmailAlreadyInUse

  EmailMustBeValid:
    when:
      - c: RegisterUser
        props:
          email: not-an-email
    then:
      - x: InvalidEmailFormat
```

## Document Structure

### Single Document

A document can contain both flows and tests:

```yaml
---
flows:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered
    - v: UserProfile

tests:
  EmailMustBeUnique:
    given:
      - e: UserRegistered
        props:
          email: joe@example.com
    when:
      - c: RegisterUser
        props:
          email: joe@example.com
    then:
      - x: EmailAlreadyInUse
```

### Multiple Documents

Use `---` to separate YAML documents in a single file:

```yaml
---
flows:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered

---
flows:
  UserDeletion:
    - t: Admin/UserManagement
    - c: DeleteUser
    - e: User/UserDeleted

---
tests:
  CannotDeleteActiveUser:
    given:
      - e: User/UserAuthenticated
    when:
      - c: DeleteUser
    then:
      - x: UserCurrentlyActive
```

## Comments

Standard YAML comments with `#`:

```yaml
---
flows:
  OrderPlacement:
    - t: Customer/ProductPage    # User clicks "Buy Now"
    - c: PlaceOrder              # Validates inventory
      props:
        productId: uuid
        quantity: int
    - e: Order/OrderPlaced       # Core domain event
    - v: OrderConfirmation       # Shows order details
```

## Complete Example

```yaml
---
flows:
  # Extended form: flow with attached tests
  UserRegistration:
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
          registeredAt: iso8601
      - e: User/WelcomeEmailSent
      - v: UserProfile
    tests:
      EmailMustBeUnique:
        for: RegisterUser
        given:
          - e: User/UserRegistered
            props:
              email: joe@example.com
        when:
          - c: RegisterUser
            props:
              email: joe@example.com
        then:
          - x: EmailAlreadyInUse

      PasswordMustBeStrong:
        for: RegisterUser
        when:
          - c: RegisterUser
            props:
              email: jane@example.com
              password: "123"
        then:
          - x: PasswordTooWeak

  # Direct form: flow without attached tests
  EmailVerification:
    - t: User/VerificationLink
    - c: VerifyEmail
      props:
        token: string
    - e: User/EmailVerified
    - v: UserProfile

---
flows:
  OrderPlacement:
    steps:
      - t: Customer/Cart
      - c: PlaceOrder
        props:
          customerId: uuid
          items: array
      - e: Order/OrderPlaced
        props:
          orderId: uuid
          totalAmount: decimal
      - e: Inventory/StockReserved
      - e: Payment/PaymentRequested
      - v: OrderConfirmation
    tests:
      CannotCancelShippedOrder:
        for: PlaceOrder
        given:
          - e: Order/OrderShipped
            props:
              orderId: order-123
        when:
          - c: CancelOrder
            props:
              orderId: order-123
        then:
          - x: OrderAlreadyShipped

  # Direct form
  OrderCancellation:
    - t: Customer/OrderDetails
    - c: CancelOrder
      props:
        orderId: uuid
        reason: string
    - e: Order/OrderCancelled
    - e: Inventory/StockReleased
    - e: Payment/RefundInitiated
    - v: OrderDetails

# Exploratory tests at root level (not yet attached to a flow)
tests:
  SomeEarlyIdea:
    when:
      - c: RefundOrder
    then:
      - e: Order/OrderRefunded
```