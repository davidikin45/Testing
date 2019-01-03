# Testing

## Types of Tests
1. Functional UI - Best to use Selenium and SpecFlow
2. Subcutaneous - Calling APIs from SOAPUI
3. Integration - Usually with DB/File System. Easiest to use TestServer. Catch regression
4. Unit - Low Level, Test all Paths, Quick to Execute. Lots of Mocking.  Entity Aggregate Root and Services.

Generally there should be more tests the lower the test type

## Books
* Clean Architecture
* Patterns of Enterprise Application Architecture

## SOLID
* Single Responsibility Principle. Do one thing and do it well.
* Dependency Inversion Principle. Interfaces rather than concrete types.

## Design Patterns
* MVC = UI
* Repository = Data Access
* Adapter = Mapping
* Strategy = Validations

## Unit Testing Data Access
* Hide everything behind IRepository interfaces
* Repository Pattern encapsulates persistence logic
* Focus on Mapping Testing rather than Repository Testing 
 
## Integration Tests (TestServer and WebApplicationFactory)
* https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-2.2

## ModelState.IsValid
* Hard to Unit Test
* Was Model Binding Successful?
* Is Object Valid?

## Validating Objects
Data Attributes. By default extending from these attribute won't give clientside validation.
* [Required]
* [StringLength]
* [MaxLength]/[MinLength]
* [EmailAddress]
* [Phone]
* [CreditCard]
* [Compare]
* [Range]
* [RegularExpression]
* [Url]
* [FileExtension]

Custom Attributes
* Extend from ValidationAttribute and implement IsValid()
* Implement IClientModelValidator for client side validation.
* https://docs.microsoft.com/en-us/aspnet/core/mvc/models/validation?view=aspnetcore-2.1

IValidatableObject
* Implement IValidatableObject
* Validate(ValidationContext) method

## Security Authorization
Role-based security = old
* Permissions tend to be broad

Claims-based security = new
* ClaimsIdentity : IIdentity
* ClaimsPrincipal : IPrincipal
* Key to remember about claim is that they are what the user “is” and not what the user can do.

Ways to implement Authorization
* [Authorize()] = User must be authenticated
* [Authorize(Roles = "Admin")] = Role-based authorization
* [Authorize(Policy = "AdminPolicy")] = Policy-based authorization. Policy has an IAuthorizationRequirement and a AuthorizationHandler<T>. Rather than creating policies it can be extended so policy = scope.
```
 public class ClaimsAuthorizationRequirement : AuthorizationHandler<ClaimsAuthorizationRequirement>, IAuthorizationRequirement
 {
     protected override Task HandleRequirementAsync(AuthorizationHandlerContext context ClaimsAuthorizationRequirement requirement)
    {

    }
 }
```
* when multiple policies are applied on a controller/action, they form AND logical operation
* Hard to unit test [Authorize] attribute so just check it's there using reflection.
* https://davidpine.net/blog/asp-net-core-security-unit-testing/

## Middleware
* Implement IMiddleware
* InvokeAsync(HttpContext context, RequestDelegate next)
* Configure Startup.cs

## MVC vs Web API
* MVC controllers inherit from Controller > ControllerBase. Returns HTML.
* API controllers inherit from ControllerBase. Return application/json or application/xml.

## xUnit
* xUnit.net
* https://xunit.github.io/
* [Fact] - Test
* [Theory] - Test with Data
* [InlineData] - Data

## Isolating Units with Moq > Mock Objects
* Fakes = Working Implementation, Not for prod
* Dummies = Passed around to satisfy parameter, never accessed or used
* Stubs = Provide answers to calls. E.g from property gets, method value returns
* Mocks = Expect/verify calls. Check if class testing calls property/method

## Styles
* BDD should not have any technical language
* Imperative Style (Complex, Explicity specify data)
* Declarative Style (Much simpler, Doesn't explicity specify data)
