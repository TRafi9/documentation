# Dependency Injection

Dependency injection is a design pattern used to loosely couple components.
It allows you to pass dependencies (e.g. services, classes or objects) into a class without having the class create the dependency itself.

## Why use dependency injection?

1. Loose Coupling: Classes are not tightly bound to specific implementations of their dependencies, making them easier to modify, replace, or test.

2. Testability: You can pass mock or stub dependencies during unit testing

3. Flexibility: You can switch between different implementations of a dependency without changing the class that depends on it, e.g. If you have a logger, you can log to the console during local development, but have the logger log to a database in production.

## Example 1:

In the following example we have a logger that we can use dependency injection to pass either the database or the console version of the logger:

### Step 1: Define the Logger Interface

We'll create an ILogger interface that defines a `Log` method. This interface will be implemented by different loggers.

```C#
public interface ILogger
{
    void Log(string message)
}
```

### Step 2: Implement logger classes

We'll create two classes that implement ILogger which will both log out differently

```C#
public class ConsoleLogger: ILogger
{
    public void Log(string message){
        Console.WriteLine("blah blah blah")
    }
}

public class DatabaseLogger: ILogger
{
    public void Log(string message){
        Console.WriteLine($"DatabaseLogger:{message}")
    }
}

```

### Step 3: Create a service that depends on ILogger

Now we will create a `TransactionService` class that needs `ILogger` to log its activities. Notice how TransactionService does not care about which ILogger implementation its using, just the fact that it uses an ILogger.

```C#
public class TransactionService
{
    private readonly ILogger _logger

    // Constructor injection: ILogger is passed in through a constructor
    public TransactionService(ILogger logger)
    {
        _logger = logger
    }
    public void ProcessTransaction(string transactionDetails)
    {
        _logger.log($"Transaction Details: {transactionDetails}:")
    }
}
```

### Step 4: Setup Dependency Injection

Now that we have `TransactionService` setup, we can essentially choose either the console or the database logger for the `ProcessTransaction` method to use. We do this using dependency injection, where we can inject the type of logger it uses. We can do this through microsofts built in dependency inection library.

1. Install the Microsoft.Extensions.DependencyInjection NuGet package if needed:

```zsh
dotnet add package Microsoft.Extensions.DependencyInjection
```

2. Create a simple console application with the following code:

```C#
using System;
using Microsoft.Extensions.DependencyInjection;

namespace DependencyInjectionExample
{
    public class Program
    {
        public static void Main(string[] args)
        {
            // Step 5: Configure DI Container
            var serviceProvider = new ServiceCollection()
                .AddTransient<ILogger, ConsoleLogger>()    // Use ConsoleLogger as the implementation of ILogger
                //.AddTransient<ILogger, DatabaseLogger>() // Uncomment this line to use DatabaseLogger instead
                .AddTransient<TransactionService>()        // Register TransactionService
                .BuildServiceProvider();

            // Step 6: Use the Service with DI
            var transactionService = serviceProvider.GetService<TransactionService>();
            transactionService.ProcessTransaction("Order ID: 12345, Amount: $250.00");

            // Output will be:
            // Processing transaction...
            // ConsoleLogger: Transaction Details: Order ID: 12345, Amount: $250.00
        }
    }
}

```

## .NET implementation example

In .NET, dependencies are usually configured in `Program.cs` or `Startup.cs`, here is an example where it is setup in .NET but also uses an env variable to switch the logger type:

### Step 1 define the logger:

```csharp
public interface ILogger
{
    void LogMessage(string message);
}
```

### Step 2 Implement the interfaces

```csharp
public class FileLogger : ILogger
{
    public void LogMessage(string message)
    {
        Console.WriteLine($"File Logger: {message}");
    }
}

public class DatabaseLogger : ILogger
{
    public void LogMessage(string message)
    {
        Console.WriteLine($"Database Logger: {message}");
    }
}
```

### Step 3 Configure dependencies in Program.cs

Here we use an env variable to decide on logger type

```csharp
// Microsoft.AspNetCore.Builder namespace provides WebApplication

var builder = WebApplication.CreateBuilder(args);

// Step 3.1: Determine the environment or configuration for logger selection.
var environment = builder.Environment;

// Step 3.2: Register the appropriate logger implementation based on the environment.
if (environment.IsDevelopment())
{
    // Use FileLogger for development
    builder.Services.AddTransient<ILogger, FileLogger>();
}
else
{
    // Use DatabaseLogger for non-development environments
    builder.Services.AddTransient<ILogger, DatabaseLogger>();
}

// Step 3.3: Register the ReportGenerator class, which will use the injected logger.
builder.Services.AddTransient<ReportGenerator>();

var app = builder.Build();

app.MapGet("/", (ReportGenerator reportGenerator) =>
{
    reportGenerator.GenerateReport();
    return "Hello World!";
});

app.Run();

```

### Step 4: Inject the ILogger into ReportGenerator

```csharp
public class ReportGenerator
{
    private readonly ILogger _logger;

    // The logger dependency is injected via the constructor.
    public ReportGenerator(ILogger logger)
    {
        _logger = logger;
    }

    public void GenerateReport()
    {
        _logger.LogMessage("Report generated");
    }
}
```

In the example above we use an env variable to determine which logger to implement.

Alternatively we can use `appsettings.json` to determine the logger we use.

### Step 1 Add a setting to `appsettings.json`

```json
{
  "Logging": {
    "LoggerType": "File" // Change to "Database" for DatabaseLogger
  }
}
```

### Step 2 Read in the appsettings file into main

```csharp
var builder = WebApplication.CreateBuilder(args);

// Read configuration setting from appsettings.json.
var loggerType = builder.Configuration["Logging:LoggerType"];

if (loggerType == "File")
{
    builder.Services.AddTransient<ILogger, FileLogger>();
}
else if (loggerType == "Database")
{
    builder.Services.AddTransient<ILogger, DatabaseLogger>();
}

builder.Services.AddTransient<ReportGenerator>();

var app = builder.Build();

app.MapGet("/", (ReportGenerator reportGenerator) =>
{
    reportGenerator.GenerateReport();
    return "Logger is running!";
});

app.Run();

```
