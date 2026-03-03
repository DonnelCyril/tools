# Result Type Usage Guide for Customer Domain

A comprehensive guide to using the `Result` type in Customer domain projects, following our established patterns and conventions for robust error handling and functional programming.

## Table of Contents
- [Overview](#overview)
- [Core Principles](#core-principles)
- [Basic Concepts](#basic-concepts)
- [Creating Results](#creating-results)
- [Pattern Matching (Preferred)](#pattern-matching-preferred)
- [Transforming Results](#transforming-results)
- [Error Handling](#error-handling)
- [Working with Collections](#working-with-collections)
- [Railway-Oriented Programming](#railway-oriented-programming)
- [Common Patterns](#common-patterns)
- [Testing](#testing)
- [Best Practices](#best-practices)

## Overview

The `Result` type is a functional programming pattern that makes error handling explicit and eliminates the need for exception-based control flow. In Customer domain projects, we use this pattern consistently with our error library (`Customer.Error.Lib`) to create robust, maintainable code.

**Benefits:**
- Makes success and failure paths explicit
- Eliminates null reference exceptions
- Enables method chaining (Railway-Oriented Programming)
- Improves code readability and maintainability
- Type-safe error handling
- Integrates seamlessly with our error types (ValidationError, BusinessRuleError, DataIntegrityError, IoError)

## Core Principles

### 1. Use Pattern Matching Over if-else

**❌ Don't use if-else with IsFailure/IsSuccess:**

```csharp
if (updateResult.IsFailure)
{
    return Result.Failure<Domain.Customer, IError>(updateResult.Error);
}

return updateResult.Value.ToDomainCustomer(command.CustomerIdentifier, PpidEncoder, Logger)
    .MapError(IError (e) => e);
```

**✅ Do use pattern matching:**

```csharp
var result = updateResult switch
{
    { IsSuccess: true, Value: var value } =>
        value.ToDomainCustomer(command.CustomerIdentifier, PpidEncoder, Logger)
            .MapError(IError (e) => e),
    _ => Result.Failure<Domain.Customer, IError>(updateResult.Error)
};
```

### 2. Use Map Method, Avoid Bind/SelectMany

Prefer `.Map()` for transformations. When you need `Bind` or `SelectMany` behavior, use pattern matching instead.

**❌ Avoid:**
```csharp
result.Bind(value => SomeOperation(value));
```

**✅ Prefer:**
```csharp
var outcome = result switch
{
    { IsSuccess: true, Value: var value } => SomeOperation(value),
    _ => Result.Failure<TargetType, IError>(result.Error)
};
```

### 3. No Maybe Type

We do not use the `Maybe` type in Customer domain projects. Use `Result` with appropriate error types instead.

## Basic Concepts

### Result Types

1. **`Result`** - Represents an operation that succeeds or fails without returning a value
2. **`Result<T>`** - Represents an operation that returns a value of type `T` on success
3. **`Result<T, E>`** - Represents an operation that returns a value of type `T` on success or an error of type `E` on failure

In Customer domain, we primarily use **`Result<T, IError>`** to work with our error library.

### Properties

- `IsSuccess` - Returns `true` if the operation succeeded
- `IsFailure` - Returns `true` if the operation failed
- `Value` - Gets the value (only available on success)
- `Error` - Gets the error object (only available on failure)

## Creating Results

### Explicit Construction

```csharp
// Success with value
Result<Customer, IError> customer = Result.Success<Customer, IError>(customerInstance);

// Failure with error
Result<Customer, IError> failed = Result.Failure<Customer, IError>(
    new ValidationError { /* ... */ }
);

// Simple success/failure without value
Result success = Result.Success();
Result failure = Result.Failure("Operation failed");
```

### Implicit Conversion

```csharp
// Directly assign a value to create success Result
Result<Customer, IError> customer = customerInstance;

// Directly assign an error to create failure Result
Result<Customer, IError> failed = new ValidationError { /* ... */ };
```


### From Functions (Exception Handling)

Wrap operations that might throw exceptions:

```csharp
// Synchronous
Result<Customer> customer = Result.Of(() => _service.CreateCustomer());

// Asynchronous
Result<Customer> customer = await Result.Of(() => _service.CreateCustomerAsync());
```

## Pattern Matching (Preferred)

### Basic Pattern Match

```csharp
var result = GetCustomer(id);

var outcome = result switch
{
    { IsSuccess: true, Value: var customer } => ProcessCustomer(customer),
    { IsFailure: true, Error: var error } => HandleError(error),
    _ => throw new InvalidOperationException("Unexpected result state")
};
```

### Pattern Match with Type Checking

```csharp
var outcome = result switch
{
    { IsSuccess: true, Value: var customer } => Ok(customer),
    { IsFailure: true, Error: ValidationError error } => BadRequest(error),
    { IsFailure: true, Error: BusinessRuleError error } => UnprocessableEntity(error),
    { IsFailure: true, Error: DataIntegrityError error } => InternalServerError(error),
    { IsFailure: true, Error: IoError error } => ServiceUnavailable(error),
    _ => InternalServerError("Unknown error occurred")
};
```

### Inline Pattern Match in Returns

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

### Pattern Match with Multiple Results

```csharp
var nameResult = Name.Create(firstName, lastName);
var emailResult = Email.Create(emailAddress);

return (nameResult, emailResult) switch
{
    ({ IsSuccess: true, Value: var name }, { IsSuccess: true, Value: var email }) =>
        CreateCustomer(name, email),
    ({ IsFailure: true }, _) =>
        Result.Failure<Customer, IError>(nameResult.Error),
    (_, { IsFailure: true }) =>
        Result.Failure<Customer, IError>(emailResult.Error),
    _ => throw new InvalidOperationException("Unexpected result state")
};
```

## Transforming Results

### Map - Transform Success Values

Transforms the inner value of a successful Result without checking success/failure:

```csharp
Result<Customer, IError> customerResult = GetCustomer(id);

Result<CustomerDto, IError> dtoResult = customerResult.Map(customer => 
    new CustomerDto
    {
        Id = customer.Id,
        Name = customer.Name.FullName,
        Email = customer.Email.Address
    }
);
```

### MapError - Transform Error Types

Transforms the error of a failed Result:

```csharp
Result<Customer, ObjectValidationError> validationResult = Customer.Create(data);

Result<Customer, IError> result = validationResult.MapError(IError (e) => e);
```

### Chaining Transformations

```csharp
var result = GetCustomer(id)
    .Map(customer => customer.UpdateEmail(newEmail))
    .Map(customer => customer.Promote())
    .MapError(IError (e) => e);
```

## Error Handling

### Combining Multiple Results

Handle multiple validations together:

```csharp
var nameResult = Name.Create(model.FirstName, model.LastName);
var emailResult = Email.Create(model.Email);
var phoneResult = PhoneNumber.New(pattern, model.Phone);

// Combine results - fails if any input fails
Result combinedResult = Result.Combine(nameResult, emailResult, phoneResult);

if (combinedResult.IsFailure)
{
    return Result.Failure<Customer, IError>(
        new CompositeError
        {
            Operation = "CreateCustomer",
            Errors = new List<IError>
            {
                nameResult.IsFailure ? nameResult.Error : null,
                emailResult.IsFailure ? emailResult.Error : null,
                phoneResult.IsFailure ? phoneResult.Error : null
            }.Where(e => e != null).ToList()
        }
    );
}

var customer = new Customer(nameResult.Value, emailResult.Value, phoneResult.Value);
```

### Pattern Match with Error Combining

```csharp
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

## Working with Collections

### TryFind - Safe Dictionary Access

```csharp
Dictionary<string, Customer> customerCache = /* ... */;

Result<Customer, IError> GetFromCache(string id)
{
    return customerCache.TryFind(id) switch
    {
        { HasValue: true, Value: var customer } => 
            Result.Success<Customer, IError>(customer),
        _ => Result.Failure<Customer, IError>(
            new ValidationError { Message = $"Customer {id} not found in cache" })
    };
}
```

### TryFirst / TryLast - Safe Collection Access

```csharp
IEnumerable<Address> addresses = customer.Addresses;

Result<Address, IError> GetPrimaryAddress()
{
    return addresses.TryFirst(a => a.IsPrimary) switch
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
}
```

## Railway-Oriented Programming

Chain multiple operations in a fluent manner, where each operation only executes if the previous one succeeded:

### Basic Pipeline

```csharp
return GetCustomerById(id)
    .Map(customer => customer.Promote())
    .Tap(customer => _emailGateway.SendPromotionNotification(customer.Email))
    .Tap(customer => _auditLog.LogPromotion(customer.Id))
    .MapError(IError (e) => e);
```

**Methods:**
- `Map` - Transforms the value on success
- `Tap` - Executes a side effect on success (doesn't change the Result)
- `MapError` - Transforms the error on failure


### Pattern Match in Pipeline

```csharp
var registrationResult = await GetRegistrationInfo(command.CustomerIdentifier);

var result = registrationResult switch
{
    { IsSuccess: true, Value: var info } =>
        CreateCustomer(info)
            .Map(customer => customer.Activate())
            .Tap(customer => _repository.SaveAsync(customer))
            .MapError(IError (e) => e),
    _ => Result.Failure<Customer, IError>(registrationResult.Error)
};

return result;
```

## Common Patterns

### Validation Pattern (Smart Constructors)

```csharp
public record Email
{
    private Email(string address)
    {
        Address = address;
    }
    
    public string Address { get; }
    
    public static Result<Email, ObjectValidationError> Create(string address)
    {
        return Validate<Email>
            .For(v => v.Address, address.IsValidEmail())
            .Create(validatedAddress => new Email(validatedAddress));
    }
}
```

### Repository Pattern

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
                    ValidationFailure = error,
                    Metadata = new Dictionary<string, object?> { ["customerId"] = id }
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

### Command Handler Pattern

```csharp
public async ValueTask<Result<Domain.Customer, IError>> Handle(
    RegisterCustomer command,
    CancellationToken cancellationToken)
{
    var registrationInfo = await Sender.Send(
        new GetCustomerRegistrationInfo(command.CustomerIdentifier),
        cancellationToken);

    var createResult = registrationInfo switch
    {
        { IsSuccess: true, Value: var info } =>
            await Sender.Send(
                new CreateCustomer
                {
                    CustomerIdentifier = command.CustomerIdentifier,
                    CustomerRegistrationInfo = info
                }, 
                cancellationToken),
        _ => Result.Failure<CustomerEntity, IError>(registrationInfo.Error)
    };

    return createResult switch
    {
        { IsSuccess: true, Value: var entity } =>
            entity.ToDomainCustomer(command.CustomerIdentifier, Logger)
                .MapError(IError (e) => e),
        _ => Result.Failure<Domain.Customer, IError>(createResult.Error)
    };
}
```

### API Controller Pattern

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

### HTTP Client with IoError

```csharp
public async Task<Result<UserDto, IError>> FetchUserAsync(
    Guid id,
    CancellationToken ct)
{
    try
    {
        var response = await _httpClient.GetAsync($"/users/{id}", ct);
        
        return response.IsSuccessStatusCode switch
        {
            true => Result.Success<UserDto, IError>(
                await response.Content.ReadFromJsonAsync<UserDto>(ct)),
            false => Result.Failure<UserDto, IError>(
                new IoError
                {
                    Operation = "FetchUser",
                    Target = "UserAPI",
                    Message = $"HTTP {(int)response.StatusCode}: {response.ReasonPhrase}",
                    IsRetryable = IsRetryableStatusCode(response.StatusCode),
                    Metadata = new Dictionary<string, object?>
                    {
                        ["http.statusCode"] = (int)response.StatusCode,
                        ["http.url"] = $"/users/{id}",
                        ["userId"] = id
                    }
                })
        };
    }
    catch (HttpRequestException ex)
    {
        return Result.Failure<UserDto, IError>(
            new IoError
            {
                Operation = "FetchUser",
                Target = "UserAPI",
                Message = ex.Message,
                IsRetryable = true,
                Metadata = new Dictionary<string, object?> { ["userId"] = id }
            }
        );
    }
}
```

## Testing

### Testing with SatisfyAllConditions (Recommended)

When asserting multiple conditions on a single result:

```csharp
[Fact]
public void Create_WithMultipleErrors_ReturnsAllValidationErrors()
{
    // Arrange
    var firstName = "";
    var lastName = "";

    // Act
    var result = Name.Create(firstName, lastName);

    // Assert
    result.ShouldSatisfyAllConditions(
        () => result.IsFailure.Should().BeTrue(),
        () => result.Error.Should().BeOfType<ObjectValidationError>(),
        () => result.Error.Errors.Should().ContainKey("FirstName"),
        () => result.Error.Errors.Should().ContainKey("LastName")
    );
}
```

## Best Practices

### 1. Always Use Pattern Matching

✅ **Do:**
```csharp
var result = updateResult switch
{
    { IsSuccess: true, Value: var value } => ProcessValue(value),
    _ => Result.Failure<Output, IError>(updateResult.Error)
};
```

❌ **Don't:**
```csharp
if (updateResult.IsFailure)
{
    return Result.Failure<Output, IError>(updateResult.Error);
}
return ProcessValue(updateResult.Value);
```

### 2. Use Map for Transformations

✅ **Do:**
```csharp
result.Map(customer => customer.ToDto())
```

❌ **Don't:**
```csharp
result.Bind(customer => Result.Success(customer.ToDto()))
```

### 3. Choose the Right Error Type

Use the error decision tree from `Customer.Error.Lib`:
1. Can the caller fix it now? → `ValidationError`
2. Is the action invalid in current state? → `BusinessRuleError`
3. Is persisted/internal data inconsistent? → `DataIntegrityError`
4. Did a dependency or system call fail? → `IoError`
5. Did multiple failures occur together? → `CompositeError`

### 4. Make Errors Descriptive

✅ **Do:**
```csharp
new ValidationError
{
    Message = "Email address is required",
    Metadata = new Dictionary<string, object?> { ["field"] = "email" }
}
```

❌ **Don't:**
```csharp
new ValidationError { Message = "Invalid input" }
```

### 5. Use Smart Constructors

Always validate domain objects at creation:

```csharp
public record Customer
{
    private Customer(/* validated params */) { }
    
    public static Result<Customer, ObjectValidationError> Create(/* raw params */)
    {
        return Validate<Customer>
            .For(/* validations */)
            .Create(/* constructor */);
    }
}
```

### 6. Compose Validations with Results

Pass `Result<T, E>` to parent validators:

```csharp
var profileResult = Profile.Create(
    Name.Create(firstName, lastName),
    Email.Create(email),
    PhoneNumber.New(pattern, phone)
);
```

### 7. Map Errors Appropriately in HTTP Context

```csharp
return result switch
{
    { IsSuccess: true, Value: var value } => Ok(value),
    { IsFailure: true, Error: ValidationError error } => BadRequest(error),
    { IsFailure: true, Error: BusinessRuleError error } => UnprocessableEntity(error),
    { IsFailure: true, Error: DataIntegrityError error } => StatusCode(500, error),
    { IsFailure: true, Error: IoError { IsRetryable: true } } => ServiceUnavailable(),
    _ => StatusCode(500, "Unknown error")
};
```

### 8. Avoid Accessing .Value Directly

✅ **Do:**
```csharp
result.Map(value => ProcessValue(value))
```

❌ **Don't:**
```csharp
if (result.IsSuccess)
{
    var value = result.Value;
    ProcessValue(value);
}
```

### 9. Don't Mix Exceptions with Result Pattern

✅ **Do:**
```csharp
try
{
    var data = await _service.GetDataAsync();
    return Result.Success<Data, IError>(data);
}
catch (Exception ex)
{
    return Result.Failure<Data, IError>(new IoError { /* ... */ });
}
```

❌ **Don't:**
```csharp
var result = GetData();
if (result.IsFailure)
{
    throw new InvalidOperationException(result.Error.Message);
}
```

## Summary

Key takeaways for using Result types in Customer domain:

1. **Always use pattern matching** over if-else for Result checking
2. **Prefer Map** for transformations, avoid Bind/SelectMany
3. **No Maybe types** - use Result with appropriate errors
4. **Smart constructors** - validate at creation time
5. **Typed errors** - use IError hierarchy from Customer.Error.Lib
6. **Railway-Oriented Programming** - chain operations with Map/Tap
7. **Pattern match for error handling** - explicit and type-safe
8. **Descriptive errors** - include context and metadata

Following these patterns ensures consistent, maintainable, and robust error handling across all Customer domain projects.

---

*Last Updated: January 2026*
