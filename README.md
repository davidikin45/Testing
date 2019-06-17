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

## Entity Framework Core InMemory and Sqlite InMemory
* [Testing with EF Core](https://app.pluralsight.com/library/courses/ef-core-testing/table-of-contents)
* Microsoft.EntityFrameworkCore.InMemory (Can't test migrations)
* Microsoft.EntityFrameworkCore.Sqlite (Can test migrations)

```
public class SqliteInMemoryConnectionFactory : IDisposable
{
	protected DbConnection _connection;

	//cant create and seed using the same context
	public async Task<DbConnection> GetConnection(CancellationToken cancellationToken = default)
	{
		if (_connection == null)
		{
			_connection = new SqliteConnection("DataSource=:memory:");
			await _connection.OpenAsync(cancellationToken);
		}

		return _connection;
	}

	public void Dispose()
	{
		if (_connection != null)
		{
			_connection.Dispose();
			_connection = null;
		}
	}
}
```
```
public class SqliteInMemoryDbContextFactory<TDbContext> : SqliteInMemoryConnectionFactory
	where TDbContext : DbContext
{
	private bool _created = false;

	private DbContextOptions<TDbContext> CreateOptions()
	{
		return new DbContextOptionsBuilder<TDbContext>()
			.UseSqlite(_connection)
			.EnableSensitiveDataLogging()
			.Options;
	}

	//cant create and seed using the same context
	public async Task<TDbContext> CreateContextAsync(bool create = true, CancellationToken cancellationToken = default)
	{
		await GetConnection(cancellationToken);

		if (!_created && create)
		{
			using (var context = (TDbContext)Activator.CreateInstance(typeof(TDbContext), CreateOptions()))
			{
				await context.Database.EnsureCreatedAsync(cancellationToken);
			}
			_created = true;
		}

		return (TDbContext)Activator.CreateInstance(typeof(TDbContext), CreateOptions());
	}
}
```
```
 using (var factory = new SqliteInMemoryDbContextFactory<DND.Data.AppContext>())
{
	using (var context = await factory.CreateContextAsync())
	{
		
	}
}
``` 

## Entity Framework Core Test Logging
* xUnit ITestOutputHelper

```
public static ILoggerFactory CommandLoggerFactory(Action<string> logger)
  => new ServiceCollection().AddLogging(builder =>
  {
	  builder.AddAction(logger).AddFilter(DbLoggerCategory.Database.Command.Name, LogLevel.Information);
  }).BuildServiceProvider()
  .GetService<ILoggerFactory>();
```

```
var connectionString = $"Data Source={dbName}.db;";

var builder = new DbContextOptionsBuilder<TContext>();
builder.UseSqlite(connectionString);
builder.UseLoggerFactory(CommandLoggerFactory(logAction));
builder.EnableSensitiveDataLogging();
return builder.Options;
```

```
 public static class ActionLoggerFactoryExtensions
{
	public static ILoggingBuilder AddAction(this ILoggingBuilder builder, Action<string> logAction)
	{
		builder.Services.AddSingleton<ILoggerProvider>(new ActionLoggerProvider(logAction));
		return builder;
	}
}

[ProviderAlias("Action")]
public class ActionLoggerProvider : ILoggerProvider
{
	private readonly Action<string> _logAction;

	public ActionLoggerProvider(Action<string> logAction)
	{
		_logAction = logAction;
	}

	public ILogger CreateLogger(string name)
	{
		return new ActionLogger(name, _logAction);
	}

	public void Dispose()
	{
	}
}
public partial class ActionLogger : ILogger
{
	private readonly Action<string> _logAction;
	private readonly string _name;

	public ActionLogger(string name, Action<string> logAction) : this(name, filter: null, logAction: logAction)
	{
	}

	public ActionLogger(string name, Func<string, LogLevel, bool> filter, Action<string> logAction)
	{
		_name = string.IsNullOrEmpty(name) ? nameof(ActionLogger) : name;
		_logAction = logAction;
	}

	public IDisposable BeginScope<TState>(TState state)
	{
		return NoopDisposable.Instance;
	}

	public bool IsEnabled(LogLevel logLevel)
	{
		return _logAction != null;
	}

	/// <inheritdoc />
	public void Log<TState>(LogLevel logLevel, EventId eventId, TState state, Exception exception, Func<TState, Exception, string> formatter)
	{
		if (!IsEnabled(logLevel))
		{
			return;
		}

		if (formatter == null)
		{
			throw new ArgumentNullException(nameof(formatter));
		}

		var message = formatter(state, exception);

		if (string.IsNullOrEmpty(message))
		{
			return;
		}

		message = $"{ logLevel }: {message}";

		if (exception != null)
		{
			message += Environment.NewLine + Environment.NewLine + exception.ToString();
		}

		_logAction(message);
	}

	private class NoopDisposable : IDisposable
	{
		public static NoopDisposable Instance = new NoopDisposable();

		public void Dispose()
		{
		}
	}
}
```

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
