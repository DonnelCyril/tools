# Test Constitution for Copilot Agent 

This document defines the **constitutional rules and preferences** that guide how Copilot (and humans) should write tests in this codebase. It is intentionally opinionated. When in doubt, **opt for clarity, determinism, and property-focused correctness over example-heavy testing**.

---

## Core Principles

1. **Tests describe properties, not examples**

   * Prefer asserting *invariants* that must always hold rather than single happy-path examples.
   * Ask: *“What must be true for all valid inputs?”* and *“What must never be true?”*

2. **Deterministic randomness beats handcrafted fixtures**

   * Use randomized data generation (Bogus) with deterministic seeding so failures are reproducible.

3. **Functional style over procedural setup**

   * Prefer pure functions, immutable inputs, and expression-based assertions.
   * Avoid mutable shared state and complex setup/teardown.

4. **Tests are executable documentation**

   * Naming, structure, and grouping should make intent obvious without comments.

---

## Technology Baseline

* **.NET:** .NET 10
* **Test framework:** xUnit
* **Data generation:** Bogus
* **Assertion style:** Shouldly (preferred)
* **Mocking library:** Moq (preferred)
* **Property-style testing:** xUnit `[Theory]` + data generators (not FsCheck unless explicitly required)

---

## Property-Based Testing Approach

### When to Use Property-Based Tests

Use property-based tests whenever:

* Input space is large or unbounded
* Validation rules exist (e.g., nulls, ranges, formats)
* Transformations must preserve invariants
* Edge cases matter more than specific values

Examples of properties:

* "All invalid tenants report fully-qualified error paths"
* "Serialization followed by deserialization preserves equality"

---

### xUnit Usage

* Prefer `[Theory]` over `[Fact]`
* Use:

  * `[MemberData]`
  * `[ClassData]`
  * `TheoryData<T>` factories

These should stream **dozens of permutations per run**.

```csharp
public class AddressValidation
{
    [Theory]
    [MemberData(nameof(addresses_with_missing_required_fields))]
    public void address_is_invalid_when_any_required_field_is_missing(CustomerAddress address)
    {
        Validate(address)
            .IsSuccess
            .Should()
            .BeFalse();
    }

    public static IEnumerable<object[]> addresses_with_missing_required_fields()
    {
        var faker = AddressFaker.Valid().UseSeed(42);

        foreach (var address in faker.Generate(50))
        {
            yield return new object[]
            {
                address with { Line1 = null }
            };

            yield return new object[]
            {
                address with { City = null }
            };

            yield return new object[]
            {
                address with { Country = null }
            };
        }
    }
}

```

---

### Deterministic Randomness

When randomness matters:

```csharp
var faker = new Faker("en").UseSeed(12345);
```

Rules:

* Every property-focused theory **must be reproducible**
* Seeds may be:

  * Hard-coded per test, or
  * Derived from a test identifier

Failing inputs must be re-creatable without guesswork.

---

### Valid + Invalid Specimens

Each property test **should generate both**:

* ✅ Valid specimens (to assert success invariants)
* ❌ Invalid specimens (to assert failure modes)

Avoid writing separate tests when a single property can cover both.

---

## Faker-Based Generators

### Use Bogus based text fixtures where possible

* Do **not** hand-roll test data, unless it is test specific corner cases.
* Do **not** use magic constants unless asserting boundary behavior

All fixtures must originate from **Bogus fakers**.

---

### Chainable Faker Pattern

Adopt a **chainable, immutable faker style**.

```csharp
var valid = CustomerFaker.Valid();

var missing_email = valid with { Email = null };
```

Rules:

* Prefer `with` expressions for mutations
* Avoid bespoke builder APIs
* Keep fakers small and composable

---

### Faker Placement

* Fakers live alongside tests 
* Never embed faker logic inline inside tests

---


## Functional Style of Testing

Tests should resemble **expressions**, not scripts.

Prefer:

```csharp
Validate(input)
    .Errors
    .Should()
    .Contain(e => e.Code == ErrorCodes.Required);
```

When you need to assert multiple conditions on a single object or result, use **Shouldly's `SatisfyAllConditions`** for clarity and atomicity:

```csharp
result.ShouldSatisfyAllConditions(
    () => result.IsFailure.ShouldBeTrue(),
    () => result.Error.Errors.ShouldContainKey("Property1"),
    () => result.Error.Errors["Property1"].ShouldBeOfType<Required>()
);
```

**Why?**
- `SatisfyAllConditions` runs all assertions and reports all failures together, making test failures easier to diagnose.
- It keeps related assertions grouped, improving readability and intent.

See: https://docs.shouldly.org/documentation/satisfyallconditions

Avoid:

* Mutable setup objects
* Step-by-step imperative flows
* Excessive Arrange blocks

A test should read top-to-bottom as a **single logical assertion** or grouped assertion.

---

## Test Naming Conventions

### snake_case Test Names

Use lower `snake_case` for all test methods. eg: create_with_valid_data_returns_success

Reason:

* Long test names remain readable
* Natural language flows better

```csharp
public void invalid_address_missing_line1_returns_validation_error()
```

---

### Nested Test Classes

Group related tests using nested classes.

Benefits:

* Shorter test names
* Stronger contextual grouping
* Shared setup when required

```csharp
public class add_customer_address
{
    public class when_address_is_invalid
    {
        [Theory]
        public void missing_line1_fails(...) { }
    }
}
```

This structure applies to **unit, integration, and application-layer tests** alike.

---

## Use of `[Fact]`

Reserve `[Fact]` for:

* Structural assertions
* Smoke tests
* Non-parametric guarantees

Examples:

* "Validate returns original reference on success"
* "Handler does not allocate on hot path"

If a test *could* be a `[Theory]`, it probably **should be**.

---

## General Testing Guidelines

### ✅ Avoid Environment-Specific Data

Tests must run consistently across:

* Local development
* CI
* UAT

Rules:

* No environment variables
* No real URLs, tenants, or credentials

---

### ✅ Keep Tests Self-Contained

* Each test manages its own setup
* No reliance on execution order
* No hidden global state

---

### ✅ Test at the Appropriate Level

Tests should live **as close to the code they validate as possible**.

Examples:

* Application-layer logic → application-layer tests
* Domain validation → domain tests
* Avoid testing application logic via GraphQL resolvers

Reference concept: **Depth of Test**

---

### ✅ Minimize Duplication Across Layers

* Trust lower-layer coverage when appropriate
* Do not re-test domain rules at API boundaries unless behavior diverges

Collaborate early with QA to avoid redundant or overlapping test scenarios.

---

## What Copilot Should Do By Default

When generating tests, Copilot should:

1. Start with properties, not examples
2. Reach for `[Theory]` + Bogus immediately
3. Seed all randomness deterministically
4. Prefer immutable `with` mutations
5. Use snake_case test names
6. Group behavior via nested classes
7. Avoid environment coupling
8. Keep tests expressive and minimal

If these rules conflict, **favor determinism and readability over cleverness**.

---

## Non-Goals

This constitution intentionally does **not**:

* Mandate a specific assertion library (though FluentAssertions is preferred)
* Replace end-to-end or contract testing strategies
* Cover performance or load testing

Those concerns belong in separate guidance.

---

**This document is living guidance. Update it when the codebase teaches us something new.**
