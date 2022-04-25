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

## Pokemon Exception Handling
Indiscriminately catching exceptions. "Pokemon - gotta catch 'em all"

❌ **Bad** Catching every exception and gracefully swallowing it.
```csharp
try
{
    MyOperation();
}
catch (Exception exc)
{
    // Do nothing
}
```

Also:
```csharp
try
{
    MyOperation();
}
catch
{
    // Do nothing
}
```

✅ **Good** Catch the specific exception one would expect and handle it.
```csharp
try
{
    MyOperation();
}
catch (InvalidOperationException exc)
{
    logger.Log(exc);
    // More logic here and if wished re-throw the exception
    throw; 
}
```

## Throwing `NullReferenceException`, `IndexOutOfRangeException`, and `AccessViolationException`
This exceptions should not be thrown in public API's. The reasoning is that those exceptions should only be thrown by the runtime and normally indicate a bug in the software.
For example one can avoid `NullReferenceException` by checking the object if it is `null` or not. On almost any case there different exceptions one can utilize, which have more semantic.

❌ **Bad** Throw `NullReferenceException` when checking a parameter.
```csharp
public void DoSomething(string name)
{
    if (name == null)
    {
        throw new NullReferenceException();
    }
```

✅ **Good** Indicating the argument is null and use the proper exception.
```csharp
public void DoSomething(string name)
{
    ArgumentNullException.ThrowIfNull(name);
}
```

> 💡 Info: More details can be found [here.](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/using-standard-exception-types)

## Don't throw `StackOverflowException` or `OutOfMemoryException`
Both exceptions are meant only to be thrown from the runtime itself. Under normal circumstances recovering from a `StackOverflow` is hard to impossible. Therefore catching a `StackOverflowException` should also be avoided.
And can catch `OutOfMemoryException` as they can also occur if a big array gets allocated but no more free space is available. That does not necessarily mean recovering from this is impossible.

> 💡 Info: More details can be found [here.](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/using-standard-exception-types)