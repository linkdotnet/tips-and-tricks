## Don't use string interpolation when logging
String interpolation normally makes a string more readable but it interferes with structured logging.
The reason is that string interpolation gets serialized ahead of time. The underlying logger just sees "one" string instead of its individual components.
With that you will loose the ability to search for the format values.

❌ **Bad** String interpolation when calling logger methods.
```csharp
User user = GetUserFromAPI();
DateTime when = DateTime.UtcNow;

_logger.LogInformation($"Creating user: {user} at {when}");
```

✅ **Good** Use structured logging to preserve format information.
```csharp
User user = GetUserFromAPI();
DateTime when = DateTime.UtcNow;

_logger.LogInformation("Creating user: {User} at {When}", user, when);
```