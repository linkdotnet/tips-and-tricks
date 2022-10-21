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

## Exception filtering with `when`
The `when` expression was introduced with C# 6 and allows a more readable way of filtering exceptions.

❌ **Bad** Less readable way.
```csharp
try
{
    await GetApiRequestAsync();
}
catch (HttpRequestException e)
{
    if (e.StatusCode == HttpStatusCode.BadRequest)
    {
        HandleBadRequest(e);
    }
    else if (e.StatusCode == HttpStatusCode.NotFound)
    {
        HandleNotFound(e);
    }
}
```

✅ **Good** More readable way.
```csharp
try
{
    await GetApiRequestAsync();
}
catch (HttpRequestException e) when (e.StatusCode == HttpStatusCode.BadRequest)
{
    HandleBadRequest(e);
}
catch (HttpRequestException e) when (e.StatusCode == HttpStatusCode.NotFound)
{
    HandleNotFound(e);
}
```

## Finalizers should not throw exceptions
Throwing an exception in a finalizer can lead to an immediate shutdown of the application without any cleanup.

❌ **Bad** Throwing an exception in the finalizer. 
```csharp
class MyClass
{
    ~MyClass()
    {
        throw new NotImplementedException();
    }
}
```

✅ **Good** Don't throw exceptions.
```csharp
class MyClass
{
    ~MyClass()
    {
    }
}
```

> 💡 Info: More details can be found [here.](https://steven-giesel.com/blogPost/3b55d5ac-f62c-4b86-bfa3-62670f614761)

## Preserving the stack trace of an exception
When catching an exception and re-throwing the exception via `throw exc` later the stack-trace gets altered, but sometimes capturing the exception can make sense. Since the .NET Framework 4.5 there is a small helper named [`ExceptionDispatchInfo`](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.exceptionservices.exceptiondispatchinfo?view=net-6.0), which let's you preserve that information. It offers a `Throw` method, which re-throws the saved exception with the given stack-trace.

❌ **Bad** Re-throwing the exception while losing the stack-trace
```csharp
Exception? exception = null;

try
{
    Throws();
}
catch (Exception exc)
{
    exception = exc;
}

// Do something here ...

if (exception is not null)
    throw exception;

void Throws() => throw new Exception("💣");
```

Produces the following output:
```no-class
Unhandled exception. System.Exception: 💣
   at Program.<Main>$(String[] args) in /Users/stgi/repos/Exception/Program.cs:line 15

```

✅ **Good** Capturing the context to re-throw it later.
```csharp
using System.Runtime.ExceptionServices;

ExceptionDispatchInfo? exceptionDispatchInfo;

try
{
    Throws();
}
catch (Exception exc)
{
    exceptionDispatchInfo = ExceptionDispatchInfo.Capture(exc);
}

// Do something here ...

if (exceptionDispatchInfo is not null)
    exceptionDispatchInfo.Throw();

void Throws() => throw new Exception("💣");
```

Produces the following output:
```no-class
Unhandled exception. System.Exception: 💣
   at Program.<<Main>$>g__Throws|0_0() in /Users/stgi/repos/Exception/Program.cs:line 19
   at Program.<Main>$(String[] args) in /Users/stgi/repos/Exception/Program.cs:line 7
--- End of stack trace from previous location ---
   at Program.<Main>$(String[] args) in /Users/stgi/repos/Exception/Program.cs:line 17
```