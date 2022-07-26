## Compie-time logging source generators

.NET 6 introduced a new of defining how to log messages with source generators.

```csharp
// The class has to be partial so we can extend the functionality
public partial class MyService
{
    private ILogger logger;

    public MyService(ILogger logger) => this.logger = logger;
    
    private void SomeActivity()
    {
        LogSomeActivity("Steven", "Giesel");
    }

    // The source generator will automatically find the "logger" private field and use it
    [LoggerMessage(Message = "Hello {lastName}, {firstName}", Level = LogLevel.Information)]
    private partial void LogSomeActivity(string firstName, string lastName);
}
```

It is also possible to have a static version, where the user passes in the `ILogger`.
```csharp
public partial class MyService
{
    private ILogger logger;

    public MyService(ILogger logger) => this.logger = logger;
    

    // The source generator will automatically find the "logger" private field and use it
    [LoggerMessage(Message = "Hello {lastName}, {firstName}", Level = LogLevel.Information)]
    public static partial void LogSomeActivity(ILogger logger, string firstName, string lastName);
}
```

> 💡 Info: A detailed explanation can be found [here](https://steven-giesel.com/blogPost/48697958-4aee-474a-8920-e266d1d7b8fa).