# Bogus Usage Guide

## Overview

This project uses [Bogus](https://github.com/bchavez/Bogus) to generate realistic test data for unit and integration tests. Bogus is a powerful fake data generator for .NET that helps create consistent, reproducible test fixtures with minimal code.

## Why Bogus?

- **Realistic Data**: Generates contextually appropriate data (names, emails, phone numbers, dates, etc.)
- **Deterministic**: Uses seeded random generation for reproducible test data
- **Fluent API**: Clean, readable syntax for data generation
- **Extensible**: Easy to create custom fakers for domain-specific objects

## Core Patterns

### 1. Basic Faker Instance

Every test class that needs fake data creates a `Faker` instance:

```csharp
using Bogus;

public class NameTests
{
    private readonly Faker _faker = new();

    [Fact]
    public void create_with_valid_data_returns_success()
    {
        // Generate random names using Bogus
        var firstName = _faker.Name.FirstName();
        var lastName = _faker.Name.LastName();
        var middleName = _faker.Name.FirstName();
        
        // Use generated data in tests
        var result = Name.Create(firstName, lastName, middleName);
        
        result.IsSuccess.ShouldBeTrue();
    }
}
```

### 2. Built-in Generators

Bogus provides many built-in data generators:

```csharp
// Person data
_faker.Name.FirstName()         // "John"
_faker.Name.LastName()          // "Smith"
_faker.Name.FullName()          // "John Smith"

// Internet data
_faker.Internet.Email()         // "john.smith@example.com"
_faker.Internet.UserName()      // "john_smith123"

// Company data
_faker.Company.CompanyName()    // "Acme Corporation"

// Date/Time data
_faker.Date.Past(30, DateTime.Today.AddYears(-18))  // Random date in the past
_faker.Date.Future()            // Random date in the future

// Random utilities
_faker.Random.Bool()            // true or false
_faker.Random.Number(1, 100)    // Random number between 1-100
_faker.PickRandom(array)        // Pick random item from array
```

## Built-in Generators Reference

Bogus provides a comprehensive set of data generators out of the box. Below is a complete reference of available generators organized by category.

### Address

Generate address-related data:

```csharp
_faker.Address.ZipCode()                    // "90210"
_faker.Address.City()                       // "San Francisco"
_faker.Address.StreetAddress()              // "123 Main Street"
_faker.Address.CityPrefix()                 // "North"
_faker.Address.CitySuffix()                 // "town"
_faker.Address.StreetName()                 // "Main Street"
_faker.Address.BuildingNumber()             // "123"
_faker.Address.StreetSuffix()               // "Street"
_faker.Address.SecondaryAddress()           // "Apt. 123"
_faker.Address.County()                     // "Berkshire"
_faker.Address.Country()                    // "United States"
_faker.Address.CountryCode()                // "US"
_faker.Address.State()                      // "California"
_faker.Address.StateAbbr()                  // "CA"
_faker.Address.Latitude()                   // 37.7749
_faker.Address.Longitude()                  // -122.4194
_faker.Address.Direction()                  // "North"
_faker.Address.CardinalDirection()          // "N"
_faker.Address.OrdinalDirection()           // "NW"
```

### Commerce

Generate product and commerce data:

```csharp
_faker.Commerce.Department()                // "Electronics"
_faker.Commerce.Price(min: 1, max: 100)     // "49.99"
_faker.Commerce.ProductName()               // "Handcrafted Cotton Shoes"
_faker.Commerce.Color()                     // "Red"
_faker.Commerce.Product()                   // "Shoes"
_faker.Commerce.ProductAdjective()          // "Handcrafted"
_faker.Commerce.ProductMaterial()           // "Cotton"
_faker.Commerce.Ean8()                      // "12345678" (barcode)
_faker.Commerce.Ean13()                     // "1234567890123" (barcode)
```

### Company

Generate company and business data:

```csharp
_faker.Company.CompanyName()                // "Acme Corporation"
_faker.Company.CompanySuffix()              // "LLC"
_faker.Company.CatchPhrase()                // "User-friendly responsive infrastructure"
_faker.Company.Bs()                         // "monetize cross-platform platforms"
```

### Database

Generate database-related data:

```csharp
_faker.Database.Column()                    // "id"
_faker.Database.Type()                      // "int"
_faker.Database.Collation()                 // "utf8_unicode_ci"
_faker.Database.Engine()                    // "InnoDB"
```

### Date

Generate date and time data:

```csharp
_faker.Date.Past(yearsToGoBack: 1)                      // Past date within 1 year
_faker.Date.PastOffset(yearsToGoBack: 1)                // Past DateTimeOffset
_faker.Date.Soon(days: 7)                               // Date within next 7 days
_faker.Date.SoonOffset(days: 7)                         // DateTimeOffset within next 7 days
_faker.Date.Future(yearsToGoForward: 1)                 // Future date within 1 year
_faker.Date.FutureOffset(yearsToGoForward: 1)           // Future DateTimeOffset
_faker.Date.Between(start, end)                         // Date between two dates
_faker.Date.BetweenOffset(start, end)                   // DateTimeOffset between two dates
_faker.Date.Recent(days: 7)                             // Date within last 7 days
_faker.Date.RecentOffset(days: 7)                       // DateTimeOffset within last 7 days
_faker.Date.Timespan()                                  // Random TimeSpan
_faker.Date.Month()                                     // "January"
_faker.Date.Weekday()                                   // "Monday"
```

### Finance

Generate financial data:

```csharp
_faker.Finance.Account(length: 8)                       // "12345678" (account number)
_faker.Finance.AccountName()                            // "Home Loan Account"
_faker.Finance.Amount(min: 0, max: 1000)                // 549.99m
_faker.Finance.TransactionType()                        // "deposit"
_faker.Finance.Currency().Code                          // "USD"
_faker.Finance.Currency().Description                   // "US Dollar"
_faker.Finance.Currency().Symbol                        // "$"
_faker.Finance.CreditCardNumber()                       // "4532-1234-5678-9012"
_faker.Finance.CreditCardCvv()                          // "123"
_faker.Finance.BitcoinAddress()                         // "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"
_faker.Finance.EthereumAddress()                        // "0x1234567890123456789012345678901234567890"
_faker.Finance.RoutingNumber()                          // "123456789"
_faker.Finance.Bic()                                    // "DEUTDEFF"
_faker.Finance.Iban()                                   // "GB33BUKB20201555555555"
```

### Hacker

Generate tech/hacker culture phrases:

```csharp
_faker.Hacker.Abbreviation()                // "HTTP"
_faker.Hacker.Adjective()                   // "open-source"
_faker.Hacker.Noun()                        // "protocol"
_faker.Hacker.Verb()                        // "parse"
_faker.Hacker.IngVerb()                     // "parsing"
_faker.Hacker.Phrase()                      // "parsing the protocol won't do anything"
```

### Image

Generate image URLs (using Lorem Picsum service):

```csharp
_faker.Image.PicsumUrl()                                // "https://picsum.photos/640/480"
_faker.Image.PicsumUrl(width: 800, height: 600)         // "https://picsum.photos/800/600"
_faker.Image.PlaceholderUrl(width: 640, height: 480)    // Placeholder image URL
_faker.Image.DataUri(width: 640, height: 480)           // Base64 encoded data URI
```

### Internet

Generate internet-related data:

```csharp
_faker.Internet.Avatar()                                // "https://cloudflare-ipfs.com/ipfs/..."
_faker.Internet.Email()                                 // "john.smith@example.com"
_faker.Internet.Email(firstName, lastName)              // "john.smith@gmail.com"
_faker.Internet.ExampleEmail()                          // "john@example.com"
_faker.Internet.UserName()                              // "john_smith123"
_faker.Internet.UserName(firstName, lastName)           // "john.smith"
_faker.Internet.DomainName()                            // "example.com"
_faker.Internet.DomainWord()                            // "example"
_faker.Internet.DomainSuffix()                          // "com"
_faker.Internet.Ip()                                    // "192.168.1.1"
_faker.Internet.Port()                                  // 8080
_faker.Internet.Ipv6()                                  // "2001:0db8:85a3:0000:0000:8a2e:0370:7334"
_faker.Internet.UserAgent()                             // "Mozilla/5.0 (Windows NT 10.0; Win64; x64)..."
_faker.Internet.Mac()                                   // "01:23:45:67:89:ab"
_faker.Internet.Password(length: 10)                    // "aB3$dF7#jK"
_faker.Internet.Color()                                 // "#ffffff"
_faker.Internet.Protocol()                              // "https"
_faker.Internet.Url()                                   // "https://example.com"
_faker.Internet.UrlWithPath()                           // "https://example.com/path/to/page"
```

### Lorem

Generate Lorem Ipsum text:

```csharp
_faker.Lorem.Word()                         // "dolor"
_faker.Lorem.Words(count: 3)                // ["lorem", "ipsum", "dolor"]
_faker.Lorem.Letter(count: 5)               // "abcde"
_faker.Lorem.Sentence(wordCount: 5)         // "Lorem ipsum dolor sit amet."
_faker.Lorem.Sentences(sentenceCount: 3)    // "Lorem ipsum... Dolor sit... Amet consectetur..."
_faker.Lorem.Paragraph(sentenceCount: 3)    // "Lorem ipsum dolor sit amet..."
_faker.Lorem.Paragraphs(paragraphCount: 3)  // Multiple paragraphs
_faker.Lorem.Text()                         // Long text
_faker.Lorem.Lines(lineCount: 3)            // Multiple lines of text
_faker.Lorem.Slug(wordCount: 3)             // "lorem-ipsum-dolor"
```

### Name

Generate person names:

```csharp
_faker.Name.FirstName()                     // "John"
_faker.Name.FirstName(gender: Gender.Male)  // "Michael"
_faker.Name.LastName()                      // "Smith"
_faker.Name.LastName(gender: Gender.Female) // "Johnson"
_faker.Name.FullName()                      // "John Smith"
_faker.Name.FullName(gender: Gender.Female) // "Jane Doe"
_faker.Name.Prefix()                        // "Mr."
_faker.Name.Prefix(gender: Gender.Female)   // "Mrs."
_faker.Name.Suffix()                        // "Jr."
_faker.Name.FindName()                      // "John Smith"
_faker.Name.JobTitle()                      // "Senior Web Developer"
_faker.Name.JobDescriptor()                 // "Senior"
_faker.Name.JobArea()                       // "Web"
_faker.Name.JobType()                       // "Developer"
```

### Phone

Generate phone numbers:

```csharp
_faker.Phone.PhoneNumber()                  // "123-456-7890"
_faker.Phone.PhoneNumber("###-###-####")    // Custom format
_faker.Phone.PhoneNumberFormat(index: 0)    // Uses format from locale data
```

### Random

Generate random data of various types:

```csharp
// Numbers
_faker.Random.Number(min: 1, max: 100)      // Random int between 1-100
_faker.Random.Int(min: 1, max: 100)         // Random int
_faker.Random.UInt(min: 1, max: 100)        // Random uint
_faker.Random.Long(min: 1, max: 100)        // Random long
_faker.Random.ULong(min: 1, max: 100)       // Random ulong
_faker.Random.Short(min: 1, max: 100)       // Random short
_faker.Random.UShort(min: 1, max: 100)      // Random ushort
_faker.Random.Byte(min: 0, max: 255)        // Random byte
_faker.Random.SByte(min: -128, max: 127)    // Random sbyte
_faker.Random.Bytes(count: 10)              // Random byte array
_faker.Random.Decimal(min: 0, max: 100)     // Random decimal
_faker.Random.Double(min: 0, max: 100)      // Random double
_faker.Random.Float(min: 0, max: 100)       // Random float
_faker.Random.Even(min: 2, max: 100)        // Random even number
_faker.Random.Odd(min: 1, max: 100)         // Random odd number

// Strings and Characters
_faker.Random.Char(min: 'a', max: 'z')      // Random char
_faker.Random.Chars(min: 'a', max: 'z', count: 10)  // Random char array
_faker.Random.String(length: 10)            // Random string (letters)
_faker.Random.String2(length: 10)           // Random string with numbers
_faker.Random.String2(length: 10, chars: "abc123")  // Custom character set
_faker.Random.Hash(length: 40)              // Random hex hash
_faker.Random.Digits(count: 5)              // "12345"
_faker.Random.ReplaceNumbers("###-##-####") // "123-45-6789"
_faker.Random.ReplaceSymbols("???-***")     // Replace with random chars
_faker.Random.Replace("###-???")            // Replace # with number, ? with letter

// Collections
_faker.Random.Bool()                        // true or false
_faker.Random.ArrayElement(array)           // Random element from array
_faker.Random.ArrayElements(array, count: 3)// Random subset of array
_faker.Random.ListItem(list)                // Random item from list
_faker.Random.ListItems(list, count: 3)     // Random subset of list
_faker.Random.CollectionItem(collection)    // Random item from collection
_faker.Random.Enum<MyEnum>()                // Random enum value
_faker.Random.EnumValues<MyEnum>(count: 2)  // Random subset of enum values
_faker.PickRandom(items)                    // Pick random item
_faker.PickRandomParam(item1, item2, item3) // Pick from params

// Strings and GUIDs
_faker.Random.Guid()                        // Random GUID
_faker.Random.Uuid()                        // Random UUID (same as Guid)
_faker.Random.RandomLocale()                // Random locale code
_faker.Random.AlphaNumeric(length: 10)      // Random alphanumeric string
_faker.Random.Hexadecimal(length: 10)       // Random hex string
_faker.Random.WeightedRandom(items, weights)// Weighted random selection
_faker.Random.Shuffle(collection)           // Shuffle collection
_faker.Random.ClampString(str, min: 5, max: 10) // Clamp string length
```

### System

Generate system-related data:

```csharp
_faker.System.FileName()                    // "document.pdf"
_faker.System.FileName(ext: "txt")          // "document.txt"
_faker.System.DirectoryPath()               // "/usr/local/bin"
_faker.System.FilePath()                    // "/usr/local/bin/file.txt"
_faker.System.CommonFileName()              // "photo.jpg"
_faker.System.CommonFileName(ext: "png")    // "photo.png"
_faker.System.MimeType()                    // "application/json"
_faker.System.CommonFileType()              // "image"
_faker.System.CommonFileExt()               // "jpg"
_faker.System.FileType()                    // "application"
_faker.System.FileExt()                     // "pdf"
_faker.System.Semver()                      // "1.2.3"
_faker.System.Version()                     // System.Version object
_faker.System.Exception()                   // Random exception
_faker.System.AndroidId()                   // Android device ID
_faker.System.ApplePushToken()              // iOS push token
_faker.System.BlackBerryPin()               // BlackBerry PIN
```

### Vehicle

Generate vehicle-related data:

```csharp
_faker.Vehicle.Vin()                        // "1HGBH41JXMN109186" (Vehicle ID)
_faker.Vehicle.Manufacturer()               // "Toyota"
_faker.Vehicle.Model()                      // "Camry"
_faker.Vehicle.Type()                       // "Sedan"
_faker.Vehicle.Fuel()                       // "Gasoline"
```

### Person

Generate complete person data (combines multiple generators):

```csharp
var person = _faker.Person;
person.FirstName                            // "John"
person.LastName                             // "Smith"
person.FullName                             // "John Smith"
person.UserName                             // "john.smith123"
person.Email                                // "john.smith@example.com"
person.Gender                               // Gender.Male
person.DateOfBirth                          // DateTime
person.Phone                                // "123-456-7890"
person.Website                              // "https://example.com"
person.Avatar                               // "https://cloudflare-ipfs.com/ipfs/..."
person.Company.Name                         // "Acme Corporation"
person.Address.Street                       // "123 Main St"
person.Address.City                         // "San Francisco"
person.Address.State                        // "CA"
person.Address.ZipCode                      // "94102"
```

### Locale-Specific Generators

Bogus supports multiple locales for generating culturally appropriate data:

```csharp
// Create a faker with specific locale
var fakerDe = new Faker("de");              // German
var fakerFr = new Faker("fr");              // French
var fakerEs = new Faker("es");              // Spanish
var fakerPt = new Faker("pt_BR");           // Brazilian Portuguese
var fakerJa = new Faker("ja");              // Japanese

// Supported locales include:
// en, en_US, en_GB, en_AU, en_CA, de, fr, es, pt_BR, nl, sv, it, pl, ru, ja, ko, zh_CN, and many more
```

### Extension Methods for Specific Countries

Some countries have specialized generators available via extension methods:

```csharp
using Bogus.Extensions.Brazil;
using Bogus.Extensions.Canada;
using Bogus.Extensions.UnitedStates;

// Brazil
_faker.Person.Cpf()                         // Brazilian CPF
_faker.Company.Cnpj()                       // Brazilian CNPJ

// Canada
_faker.Person.Sin()                         // Social Insurance Number

// United States
_faker.Person.Ssn()                         // Social Security Number
_faker.Company.Ein()                        // Employer Identification Number

// United Kingdom
_faker.Finance.SortCode()                   // Banking Sort Code
_faker.Finance.Nino()                       // National Insurance Number
_faker.Finance.VatNumber()                  // VAT Number
```

### Usage Notes

1. **Seeding**: Set `Randomizer.Seed` for reproducible data across test runs
2. **Locale**: Default locale is "en" (English), can be changed via constructor
3. **Thread Safety**: Create separate `Faker` instances per thread for concurrent scenarios
4. **Extension Methods**: Some generators are available via extension methods in specific namespaces

## Custom Faker Pattern

For domain objects, we create custom Faker classes that encapsulate the logic for generating valid test data.

### Structure

Custom fakers follow this pattern:

1. **Record type** - Immutable, lightweight wrapper
2. **Constructor** - Takes a `Faker` instance for composition
3. **Create method** - Returns a valid domain object
4. **Extension method** - Provides fluent API access

### Example: NameFaker

```csharp
namespace Customer.Application.Tests.Helpers.Fakers;

using Bogus;
using Customer.Application.Lib.Domain;

/// <summary>
/// Test data generator for Name domain objects
/// </summary>
public record NameFaker(Faker Faker)
{
    /// <summary>
    /// Creates a valid Name instance with random data
    /// </summary>
    public Name Create()
    {
        var result = Name.Create(
            Faker.Name.FirstName(),
            Faker.Name.LastName(),
            Faker.Random.Bool() ? Faker.Name.FirstName() : null);
            
        return result.IsSuccess 
            ? result.Value 
            : throw new InvalidOperationException($"Failed to create valid Name: {result.Error}");
    }
}

/// <summary>
/// Extension to add NameFaker to Faker instances
/// </summary>
public static class NameFakerExtensions
{
    public static NameFaker NameFaker(this Faker faker) => new(faker);
}
```

### Usage in Tests

```csharp
public class ProfileTests
{
    private readonly Faker _faker = new();

    [Fact]
    public void create_with_valid_data_returns_success()
    {
        // Use custom faker via extension method
        var name = _faker.NameFaker().Create();
        var phoneNumbers = _faker.PhoneNumbersFaker().Create();
        
        // Generate other data directly
        var dateOfBirth = DateOfBirth.New(_faker.Date.Past(30, DateTime.Today.AddYears(-18)));
        var email = _faker.Internet.Email();
        var businessName = _faker.Company.CompanyName();

        var result = Profile.Create(name, dateOfBirth, email, phoneNumbers, businessName);

        result.IsSuccess.ShouldBeTrue();
    }
}
```

## Advanced Patterns

### 1. Parameterized Creation

Some fakers support optional parameters for specific test scenarios:

```csharp
public record PhoneNumbersFaker(Faker Faker)
{
    // Generate completely random phone numbers
    public PhoneNumbers Create() { ... }
    
    // Generate with specific values for testing
    public PhoneNumbers Create(string? home = null, string? business = null, string? mobile = null)
    {
        // Use provided values or generate random ones
        var homeResult = home != null
            ? PhoneNumber.New(PhonePattern, home).Map(PhoneNumber? (p) => p)
            : Result.Success<PhoneNumber?, IValidationError>(null);
            
        // ... similar for business and mobile
    }
}
```

Usage:

```csharp
// All random
var phoneNumbers = _faker.PhoneNumbersFaker().Create();

// Specific mobile, others random or null
var phoneNumbers = _faker.PhoneNumbersFaker().Create(null, null, "+64211111111");

// All specific
var phoneNumbers = _faker.PhoneNumbersFaker().Create("+6491234567", "+6497654321", "+64210111111");
```

### 2. Composable Fakers

Complex domain objects can compose simpler fakers:

```csharp
public record ProfileFaker(Faker Faker)
{
    public Profile Create()
    {
        // Compose other fakers
        var name = Faker.NameFaker().Create();
        var phoneNumbers = Faker.PhoneNumbersFaker().Create();
        
        // Mix with direct Bogus generators
        var dob = DateOfBirth.New(Faker.Date.Past(30, DateTime.Today.AddYears(-18)));
        var email = Faker.Internet.Email();
        var businessName = Faker.Company.CompanyName();
        
        var result = Profile.Create(name, dob, email, phoneNumbers, businessName);
        
        return result.IsSuccess
            ? result.Value
            : throw new InvalidOperationException($"Failed to create valid Profile: {result.Error}");
    }
}
```

### 3. Result Type Handling

Our domain objects use the `Result<T, Error>` pattern. Fakers handle this by:

1. Creating the domain object with generated data
2. Checking if creation succeeded
3. Returning the value or throwing an exception

```csharp
var result = Name.Create(
    Faker.Name.FirstName(),
    Faker.Name.LastName(),
    Faker.Random.Bool() ? Faker.Name.FirstName() : null);

return result.IsSuccess 
    ? result.Value 
    : throw new InvalidOperationException($"Failed to create valid Name: {result.Error}");
```

**Note**: This approach is acceptable in test code because:
- Tests should fail loudly if faker setup is incorrect
- The exception message includes the validation error for debugging
- It simplifies test arrangement code

## Best Practices

### 1. One Faker Instance Per Test Class

Create a single `Faker` instance as a field:

```csharp
public class MyTests
{
    private readonly Faker _faker = new();
    
    // Use _faker in all test methods
}
```

### 2. Use Custom Fakers for Domain Objects

Don't generate domain objects manually in tests. Create a reusable faker:

❌ **Bad**:
```csharp
[Fact]
public void some_test()
{
    var name = Name.Create(_faker.Name.FirstName(), _faker.Name.LastName()).Value;
    var phoneNumbers = PhoneNumbers.Create(...).Value;
    // Repeated in every test
}
```

✅ **Good**:
```csharp
[Fact]
public void some_test()
{
    var name = _faker.NameFaker().Create();
    var phoneNumbers = _faker.PhoneNumbersFaker().Create();
    // Reusable, consistent
}
```

### 3. Keep Fakers Simple and Focused

Fakers should generate **valid** data by default with minimal variations. Don't create a method for every possible test scenario.

#### The Two-Method Pattern

For complex objects (like events or DTOs), provide just two methods:
1. **Standard creation** - The most common valid scenario
2. **Alternative creation** - One key variation (e.g., deleted/obfuscated state)

Use C# record `with` expressions for other variations:

✅ **Good**:
```csharp
public record CustomerUpdatedEventFaker(Faker Faker)
{
    /// <summary>
    /// Creates a CustomerUpdatedEvent with full customer data (non-deleted)
    /// </summary>
    public CustomerUpdatedEvent Create()
    {
        return new CustomerUpdatedEvent(
            MessageId: Faker.Random.Guid(),
            CorrelationId: Faker.Random.Guid().ToString(),
            MessageType: "CustomerUpdated",
            SchemaVersion: "1.0",
            TimeGeneratedUtc: Faker.Date.RecentOffset(days: 1),
            SourceSystem: "nz.customer.api",
            Data: new CustomerDto(
                ScvId: Faker.Random.Guid(),
                FirstName: Faker.Name.FirstName(),
                // ... other fields
                IsDeleted: false,
                Status: Faker.PickRandom<CustomerApiCustomerStatus>()
            )
        );
    }

    /// <summary>
    /// Creates a CustomerUpdatedEvent with obfuscated customer data (IsDeleted = true)
    /// </summary>
    public CustomerUpdatedEvent CreateObfuscated()
    {
        return new CustomerUpdatedEvent(
            MessageId: Faker.Random.Guid(),
            // ... same structure but IsDeleted: true, Status: null
        );
    }
}

// In tests - use 'with' expressions for variations:
[Fact]
public void test_with_null_scv_id()
{
    var @event = _faker.CustomerUpdatedEventFaker().Create();
    var eventWithNullScvId = @event with { Data = @event.Data with { ScvId = null } };
    
    // Test with modified event
}

[Fact]
public void test_with_null_data()
{
    var @event = _faker.CustomerUpdatedEventFaker().Create();
    var eventWithNullData = @event with { Data = null };
    
    // Test with null data
}
```

❌ **Bad - Too Many Methods**:
```csharp
public record CustomerUpdatedEventFaker(Faker Faker)
{
    public CustomerUpdatedEvent Create() { ... }
    public CustomerUpdatedEvent CreateDeleted() { ... }
    public CustomerUpdatedEvent CreateNonDeleted() { ... }
    public CustomerUpdatedEvent CreateWithNullData() { ... }
    public CustomerUpdatedEvent CreateDeletedWithoutScvId() { ... }
    public CustomerUpdatedEvent CreateWithSpecificStatus(CustomerApiCustomerStatus status) { ... }
    // Don't do this! Use 'with' expressions instead
}
```

#### Benefits of This Pattern

1. **Simplicity**: Fewer methods to maintain
2. **Flexibility**: Tests can modify any property as needed
3. **Explicitness**: Test intent is clear from the modifications
4. **Immutability**: Leverages C# record features naturally

#### When to Add Parameters

Only add optional parameters if they're frequently needed across many tests:

```csharp
public Name Create()
{
    return Name.Create(
        Faker.Name.FirstName(),
        Faker.Name.LastName(),
        Faker.Random.Bool() ? Faker.Name.FirstName() : null
    ).Value;
}
```

For edge cases and invalid data, create them explicitly in the specific test:

```csharp
[Fact]
public void create_with_invalid_name_returns_error()
{
    // Explicitly create invalid data for this test
    var @event = _faker.CustomerUpdatedEventFaker().Create();
    var invalidEvent = @event with { Data = null };
    
    var result = ProcessEvent(invalidEvent);
    
    result.IsFailure.ShouldBeTrue();
}
```

### 4. Use Extension Methods for Discoverability

Extension methods make custom fakers discoverable via IntelliSense:

```csharp
public static class NameFakerExtensions
{
    public static NameFaker NameFaker(this Faker faker) => new(faker);
}
```

Now you can type `_faker.` and see all available custom fakers.

### 5. Document Complex Generation Logic

Add XML comments to explain non-obvious logic:

```csharp
/// <summary>
/// Creates a valid PhoneNumbers instance with realistic country codes.
/// Generates Australian (+61) or New Zealand (+64) phone numbers.
/// </summary>
public PhoneNumbers Create() { ... }
```

## Common Patterns Reference

### Conditional Optional Values

Generate optional fields randomly:

```csharp
Faker.Random.Bool() ? Faker.Name.FirstName() : null
```

### Realistic Phone Numbers

Generate valid phone numbers for specific countries:

```csharp
string[] countryCodes = ["61", "64"]; // AU: +61, NZ: +64
var countryCode = Faker.PickRandom(countryCodes);
var mobilePrefix = countryCode == "61" ? "4" : "2";
var mobileNumber = $"+{countryCode}{mobilePrefix}{Faker.Random.Number(10000000, 99999999)}";
```

### Date Ranges

Generate dates within specific ranges:

```csharp
// Adult (18+ years old)
_faker.Date.Past(30, DateTime.Today.AddYears(-18))

// Recent past (last 30 days)
_faker.Date.Recent(30)

// Future date
_faker.Date.Future()
```

## Testing with Bogus

### Arrange-Act-Assert with Fakers

```csharp
[Fact]
public void customer_can_update_mobile_number()
{
    // Arrange - Use fakers to set up test data
    var mobile = _faker.PhoneNumbersFaker().Create().Mobile;
    var customerId = Guid.NewGuid().ToString();
    
    // Act - Perform the operation
    var result = UpdateMobileNumber.Create(customerId, mobile);
    
    // Assert - Verify the outcome
    result.IsSuccess.ShouldBeTrue();
    result.Value.Mobile.ShouldBe(mobile);
}
```

### Testing Validation with Random Data

Bogus helps test validation logic thoroughly:

```csharp
[Fact]
public void create_with_valid_data_returns_success()
{
    // Generate random valid data
    var name = _faker.NameFaker().Create();
    var phoneNumbers = _faker.PhoneNumbersFaker().Create();
    
    // Validation should pass for any valid random data
    var result = Profile.Create(name, ...);
    
    result.IsSuccess.ShouldBeTrue();
}
```

## Additional Resources

- [Bogus GitHub Repository](https://github.com/bchavez/Bogus)
- [Bogus API Documentation](https://github.com/bchavez/Bogus#bogus-api-support)
- [Faker.js](https://fakerjs.dev/) - The JavaScript library that inspired Bogus

## Summary

Bogus provides a powerful, flexible way to generate test data that:
- ✅ Reduces test code duplication
- ✅ Improves test readability
- ✅ Generates realistic data automatically
- ✅ Makes tests more maintainable
- ✅ Enables comprehensive testing with varied data

By following the patterns in this guide, you can create consistent, reliable test fixtures that make your tests easier to write and understand.
