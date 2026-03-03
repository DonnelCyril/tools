---
name: result-type-patterns
description: 'Enforce functional programming patterns using Result<T, E> types in C# Customer domain. Use when writing error handling, creating domain objects, implementing command handlers, repository methods, or API controllers. Ensures pattern matching over if-else, Map over Bind, proper error types (ValidationError, BusinessRuleError, DataIntegrityError, IoError), and Railway-Oriented Programming.'
---

# Result Type Patterns

Enforce strict adherence to Result type usage patterns in Customer domain projects. This skill ensures functional programming best practices for error handling using `Result<T, IError>` with the Customer.Error.Lib error hierarchy.

## When to Use This Skill

- Writing any code that returns `Result`, `Result<T>`, or `Result<T, E>`
- Implementing command handlers, repository methods, or domain logic
- Creating smart constructors for domain objects
- Handling errors in API controllers
- Performing validations or transformations
- Working with collections safely (dictionaries, lists)
- Composing multiple operations in pipelines

## Core Principles (ALWAYS ENFORCE)

### 1. Pattern Matching Over If-Else

**NEVER ALLOW:**
```csharp
if (result.IsFailure)
{
    return Result.Failure<Output, IError>(result.Error);
}
return ProcessValue(result.Value);
```

**ALWAYS REQUIRE:**
```csharp
var outcome = result switch
{
    { IsSuccess: true, Value: var value } => ProcessValue(value),
    _ => Result.Failure<Output, IError>(result.Error)
};
```

### 2. Use Map for Transformations

**NEVER ALLOW:**
```csharp
result.Bind(value => SomeOperation(value));
result.SelectMany(value => SomeOperation(value));
```

**ALWAYS REQUIRE:**
```csharp
result.Map(value => value.Transform())
```

For complex chaining, use pattern matching:
```csharp
var outcome = result switch
{
    { IsSuccess: true, Value: var value } => ComplexOperation(value),
    _ => Result.Failure<TargetType, IError>(result.Error)
};
```

### 3. No Maybe Type

**REJECT** any use of `Maybe<T>`. Use `Result<T, IError>` with appropriate error types instead.

## Required Patterns

### Smart Constructors

Domain objects MUST validate at creation:

```csharp
public record Email
{
    private Email(string address) => Address = address;
    
    public string Address { get; }
    
    public static Result<Email, ObjectValidationError> Create(string address)
    {
        return Validate<Email>
            .For(v => v.Address, address.IsValidEmail())
            .Create(validatedAddress => new Email(validatedAddress));
    }
}
```

### Command Handlers

```csharp
public async ValueTask<Result<Domain.Customer, IError>> Handle(
    UpdateCustomer command,
    CancellationToken cancellationToken)
{
    var updateResult = await _repository.UpdateAsync(command, cancellationToken);

    return updateResult switch
    {
        { IsSuccess: true, Value: var value } =>
            value.ToDomainCustomer(command.CustomerIdentifier, PpidEncoder, Logger)
                .MapError(IError (e) => e),
        _ => Result.Failure<Domain.Customer, IError>(updateResult.Error)
    };
}
```

### Repository Methods

```csharp
public async Task<Result<Customer, IError>> GetByIdAsync(Guid id)
{
    try
    {
        var entity = await _context.Customers.FindAsync(id);
        
        return entity switch
        {
            null => Result.Failure<Customer, IError>(
                new ValidationError { Message = $"Customer {id} not found" }),
            _ => Customer.Restore(entity)
                .MapError(error => new DataIntegrityError
                {
                    Operation = "RestoreCustomer",
                    Message = "Customer data failed validation",
                    EntityType = "Customer",
                    ValidationFailure = error
                })
        };
    }
    catch (Exception ex)
    {
        return Result.Failure<Customer, IError>(
            new IoError
            {
                Operation = "GetCustomerById",
                Target = "Database",
                Message = ex.Message,
                IsRetryable = IsTransientException(ex)
            }
        );
    }
}
```

### API Controllers

```csharp
[HttpPost]
public async Task<IActionResult> CreateCustomer(
    [FromBody] CreateCustomerRequest request,
    CancellationToken cancellationToken)
{
    var result = await _mediator.Send(
        new RegisterCustomer { /* ... */ },
        cancellationToken);

    return result switch
    {
        { IsSuccess: true, Value: var customer } => 
            Ok(new CustomerResponse(customer)),
        { IsFailure: true, Error: ValidationError error } => 
            BadRequest(error),
        { IsFailure: true, Error: BusinessRuleError error } => 
            UnprocessableEntity(error),
        { IsFailure: true, Error: DataIntegrityError error } => 
            StatusCode(500, error),
        { IsFailure: true, Error: IoError error } => 
            ServiceUnavailable(error),
        _ => StatusCode(500, "Unknown error occurred")
    };
}
```

## Error Type Decision Tree

When creating errors, ENFORCE this hierarchy:

1. **ValidationError** - Can the caller fix it now? (Invalid input, missing fields)
2. **BusinessRuleError** - Is this action invalid in current state? (Insufficient balance, duplicate registration)
3. **DataIntegrityError** - Is persisted/internal data inconsistent? (Failed to restore from DB)
4. **IoError** - Did a dependency/system call fail? (Database, HTTP, file system)
5. **CompositeError** - Did multiple failures occur together?

### Error Creation Examples

```csharp
// ValidationError - caller can fix
new ValidationError
{
    Message = "Email address is required",
    Metadata = new Dictionary<string, object?> { ["field"] = "email" }
}

// BusinessRuleError - state-dependent
new BusinessRuleError
{
    Operation = "WithdrawFunds",
    Rule = "SufficientBalance",
    Message = "Account balance insufficient for withdrawal"
}

// DataIntegrityError - data corruption
new DataIntegrityError
{
    Operation = "RestoreCustomer",
    Message = "Customer data failed validation",
    EntityType = "Customer",
    ValidationFailure = validationError
}

// IoError - external failure
new IoError
{
    Operation = "GetCustomerById",
    Target = "Database",
    Message = ex.Message,
    IsRetryable = true
}
```

## Railway-Oriented Programming

Chain operations using Map and Tap:

```csharp
return GetCustomerById(id)
    .Map(customer => customer.Promote())
    .Tap(customer => _emailGateway.SendPromotionNotification(customer.Email))
    .Tap(customer => _auditLog.LogPromotion(customer.Id))
    .MapError(IError (e) => e);
```

**Methods:**
- `Map` - Transform value on success, skip on failure
- `Tap` - Execute side effect on success, skip on failure (returns original Result)
- `MapError` - Transform error type

## Safe Collection Access

### TryFind (Dictionaries)

```csharp
Dictionary<string, Customer> cache = /* ... */;

var result = cache.TryFind(id) switch
{
    { HasValue: true, Value: var customer } => 
        Result.Success<Customer, IError>(customer),
    _ => Result.Failure<Customer, IError>(
        new ValidationError { Message = $"Customer {id} not found" })
};
```

### TryFirst / TryLast

```csharp
var result = addresses.TryFirst(a => a.IsPrimary) switch
{
    { HasValue: true, Value: var address } => 
        Result.Success<Address, IError>(address),
    _ => Result.Failure<Address, IError>(
        new BusinessRuleError 
        { 
            Operation = "GetPrimaryAddress",
            Rule = "MustHavePrimaryAddress",
            Message = "Customer must have a primary address" 
        })
};
```

## Combining Multiple Results

When validating multiple inputs:

```csharp
var nameResult = Name.Create(firstName, lastName);
var emailResult = Email.Create(email);
var phoneResult = PhoneNumber.New(pattern, phone);

return (nameResult, emailResult, phoneResult) switch
{
    ({ IsSuccess: true }, { IsSuccess: true }, { IsSuccess: true }) =>
        Customer.Create(nameResult.Value, emailResult.Value, phoneResult.Value),
    _ => Result.Failure<Customer, IError>(
        new CompositeError
        {
            Operation = "CreateCustomer",
            Errors = new[]
            {
                nameResult.IsFailure ? nameResult.Error : null,
                emailResult.IsFailure ? emailResult.Error : null,
                phoneResult.IsFailure ? phoneResult.Error : null
            }.Where(e => e != null).Cast<IError>().ToList()
        })
};
```

## Testing Patterns

Use `ShouldSatisfyAllConditions` for multiple assertions:

```csharp
[Fact]
public void Create_WithInvalidData_ReturnsValidationError()
{
    // Arrange & Act
    var result = Email.Create("");

    // Assert
    result.ShouldSatisfyAllConditions(
        () => result.IsFailure.Should().BeTrue(),
        () => result.Error.Should().BeOfType<ObjectValidationError>(),
        () => result.Error.Errors.Should().ContainKey("Address")
    );
}
```

## Code Review Checklist

When reviewing code, REJECT if:

- [ ] Uses `if (result.IsFailure)` instead of pattern matching
- [ ] Uses `Bind` or `SelectMany` instead of `Map`
- [ ] Uses `Maybe<T>` type
- [ ] Accesses `.Value` directly without pattern matching
- [ ] Throws exceptions for control flow
- [ ] Creates errors without proper metadata
- [ ] Doesn't use error decision tree
- [ ] Missing smart constructors for domain objects
- [ ] Repository methods don't wrap exceptions in IoError
- [ ] API controllers don't map error types to HTTP status codes

When reviewing code, REQUIRE:

- [x] Pattern matching for all Result checks
- [x] Map for value transformations
- [x] Proper error types with descriptive messages
- [x] Smart constructors for domain objects
- [x] Railway-Oriented Programming for pipelines
- [x] IoError for all external dependencies
- [x] CompositeError for multiple failures
- [x] Safe collection access with TryFind/TryFirst/TryLast

## Quick Reference

| Anti-Pattern | Correct Pattern |
|--------------|-----------------|
| `if (result.IsFailure)` | `result switch { ... }` |
| `result.Bind(x => ...)` | `result.Map(x => ...)` |
| `result.Value` | Pattern match with `{ Value: var v }` |
| `throw new Exception()` | `Result.Failure<T, IError>(error)` |
| `Maybe<T>` | `Result<T, IError>` |
| `dict[key]` | `dict.TryFind(key)` |
| `list.First()` | `list.TryFirst()` |

## References

See `references/result-type-usage-full.md` for comprehensive examples and detailed explanations.
