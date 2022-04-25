# Exceptions
This chapter looks closer to exceptions and how exceptions are handled.

## Re-throwing exceptions incorrectly
If done the wrong way, the stack trace including valuable information is gone.

❌ **Bad** Throwing the original object again will create a new stack trace
```csharp
try
{
    // logic which can throw here
}
catch (Exception exc)
{
    throw exc;
}
```

⚠️ **Warning** Creating a new exception object with the original as `InnerException` but a new stack trace which can make debugging harder.
```csharp
try
{
    // logic which can throw here
}
catch (Exception exc)
{
    throw new Exception("A message here", exc);
}
```

✅ **Good** Simply `throw` the exception to preserve the original stack trace.
```csharp
try
{
    // logic which can throw here
}
catch (Exception exc)
{
    // Logic like logging here
    throw;
}
```

> 💡 Info: Sometimes hiding the original stack trace might be the goal due to for example security related reasons. In the average case re-throwing via `throw;` is preferred.