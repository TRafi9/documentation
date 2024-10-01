# Interfaces in C#

Interfaces are used to declare a set of members (methods, properties, etc.) that an implementing class must provide. When you define an interface, you specify the methods and properties it must have, but you do not define the implementation. This means that when a class implements an interface, it must define what those methods actually do. Interfaces usually start with the letter `I` (e.g., `ILogger`).

## Why Use Interfaces?

Interfaces are beneficial for several reasons:

- **Decoupling**: They allow you to separate the declaration of members (methods, properties, etc.) from the implementation details. This separation helps reduce dependencies and makes it easier to change implementations without affecting other parts of the code.
- **Abstraction**: Interfaces provide a way to work with different implementations. For example, you can define a logger interface (`ILogger`) that can be implemented as a database logger or a local file logger. The consuming code only needs to work with the `ILogger` interface and does not need to know the underlying implementation details. This abstraction is typically set up during configuration (e.g., in a startup file).

## Example

The following example demonstrates how to define and implement an interface in C#:

```csharp
public interface ILogger
{
    void LogMessage(string message);
}

public class FileLogger : ILogger
{
    public void LogMessage(string message)
    {
        // Write the message to a file
        System.IO.File.AppendAllText("log.txt", message + Environment.NewLine);
    }
}

public class DatabaseLogger : ILogger
{
    public void LogMessage(string message)
    {
        // Save the message to a database
        Console.WriteLine("Logged to database: " + message);
    }
}
```

In the above example:

- `ILogger` is an interface that declares a `LogMessage` method.
- `FileLogger` and `DatabaseLogger` are two different classes that implement the `ILogger` interface with their own definitions of the `LogMessage` method.

## Usage Example

To instantiate and use one of these classes through the `ILogger` interface, you can do the following:

```csharp
public interface ILogger
{
    void LogMessage(string message);
}

public class DatabaseLogger : ILogger
{
    public void LogMessage(string message)
    {
        Console.WriteLine("Logged to database: " + message);
    }
}

public class Program
{
    public static void Main()
    {
        // Create an instance of DatabaseLogger and use it through the ILogger interface
        ILogger myDBLogger = new DatabaseLogger();

        // Log a message using the DatabaseLogger's implementation of LogMessage
        myDBLogger.LogMessage("Hello, world!");
    }
}
```

In this example:

- `myDBLogger` is created as an instance of `DatabaseLogger` but is referred to using the `ILogger` interface.
- This allows for flexibility; you can later swap out `DatabaseLogger` for another implementation (e.g., `FileLogger`) without changing the rest of your code.
