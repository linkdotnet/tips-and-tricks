# Tips and tricks
A collection of tips and tricks with smaller code snippets and explanation.

- [Tips and tricks](#tips-and-tricks)
- [async await](#async-await)
  - [Elide await keyword - Exceptions](#elide-await-keyword---exceptions)
  - [Elide await keyword - using block](#elide-await-keyword---using-block)
  - [Return `null` `Task` or `Task<T>`](#return-null-task-or-taskt)
- [Blazor](#blazor)
  - [`CascadingValue` fixed](#cascadingvalue-fixed)
  - [`@key` directive](#key-directive)
- [Exceptions](#exceptions)
  - [Rethrowing exceptions incorrectly](#rethrowing-exceptions-incorrectly)

# async await
The following chapter will deep dive into tips, tricks and pitfalls when it comes down to `async` and `await` in C#.

## Elide await keyword - Exceptions
Eliding the await keyword can lead to a less traceable stacktrace due to the fact that every `Task` which doesn't get awaited, will not be part of the stack trace.

❌ **Bad**
```csharp
using System;
using System.Threading.Tasks;

try
{
	await DoWorkWithoutAwaitAsync();
}
catch (Exception e)
{
	Console.WriteLine(e);
}

static Task DoWorkWithoutAwaitAsync()
{
	return ThrowExceptionAsync();
}

static async Task ThrowExceptionAsync()
{
	await Task.Yield();
	throw new Exception("Hey");
}
```

Will result in

```
System.Exception: Hey  
 at Program.<<Main>$>g__ThrowExceptionAsync|0_1()  
 at Program.<Main>$(String[] args)  
``` 

✅ **Good** If speed and allocation is not very crucial, add the `await` keyword.
```csharp
using System;
using System.Threading.Tasks;

try
{
	await DoWorkWithoutAwaitAsync();
}
catch (Exception e)
{
	Console.WriteLine(e);
}

static async Task DoWorkWithoutAwaitAsync()
{
	await ThrowExceptionAsync();
}

static async Task ThrowExceptionAsync()
{
	await Task.Yield();
	throw new Exception("Hey");
}
```

Will result in:
```
System.Exception: Hey
   at Program.<<Main>$>g__ThrowExceptionAsync|0_1()
   at Program.<<Main>$>g__DoWorkWithoutAwaitAsync|0_0()
   at Program.<Main>$(String[] args)
```

💡 Info: Eliding the `async` keyword will also elide the whole state machine. In very hot paths that might be worth a consideration. In normal cases one should not elide the keyword. The allocations one is saving is depending on the circumstances but a normally very very small especially if only smaller objects are passed around. Also performance-wise there is no big gain when eliding the keyword (we are talking nano seconds). Please measure first and act afterwards.

## Elide await keyword - using block
Eliding inside an `using` block can lead to a disposed object before the `Task` is finished.

❌ **Bad** Here the download will be aborted / the `HttpClient` gets disposed:
```csharp
public Task<string> GetContentFromUrlAsync(string url)
{
    using var client = new HttpClient();
        return client.GetStringAsync(url);
}
```

✅ **Good**
```csharp
public async Task<string> GetContentFromUrlAsync(string url)
{
    using var client = new HttpClient();
        return await lient.GetStringAsync(url);
}
```

💡 Info: Eliding the `async` keyword will also elide the whole state machine. In very hot paths that might be worth a consideration. In normal cases one should not elide the keyword. The allocations one is saving is depending on the circumstances but a normally very very small especially if only smaller objects are passed around. Also performance-wise there is no big gain when eliding the keyword (we are talking nano seconds). Please measure first and act afterwards.

## Return `null` `Task` or `Task<T>`
When returning directly `null` from a synchronous call (no `async` or `await`) will lead to `NullReferenceException`:

❌ **Bad** Will throw `NullReferenceException`
```csharp
await GetAsync();

static Task<string> GetAsync()
{
	return null;
}
```

✅ **Good** Use `Task.FromResult`:

```csharp
await GetAsync();

static Task<string> GetAsync()
{
	return Task.FromResult(null);
}
```

# Blazor
The following chapter show some tips and tricks in regards to **Blazor**.

## `CascadingValue` fixed
CascadingValues are used, as the name implies, the cascade a parameter down the hierarchy. The problem with that is, that every child component will listen to changes to the original value. This can get expensive. If your component does not rely on updates, you can pin / fix this value. For that you can use the IsFixed parameter.

❌ **Bad** 
```razor
<CascadingValue Value="this">
    <SomeOtherComponents>
</CascadingValue>
```

✅ **Good** If the component does **not** need further updates (or the value never changes anyway) we can **fix** the value.
```razor
<CascadingValue Value="this" IsFixed="true">
    <SomeOtherComponents>
</CascadingValue>
```

## `@key` directive
Blazor diff engine determines which elements have to be re-rendered on every render cycle. It does that via sequence numbers. That works in most of cases very well, but adding or removing items in the middle or at the beginning of the list, will lead to re-rendering the whole list instead of the single entry. 

❌ **Bad** 
```razor
<ul>
    @foreach (var item in items)
    {
        <li>@item</li>
    }
</ul>
```

✅ **Good** Helping Blazor how it should find the difference between two render cycles.
```razor
<ul>
    @foreach (var item in items)
    {
        @* With the @key attribute we define what makes the element unique
           Based on this Blazor can determine whether or not it has to re-render
           the element
        *@
        <li @key="item">@item</li>
    }
</ul>
```

💡 Info: If you want to know more about `@key` [here](https://steven-giesel.com/blogPost/a8772410-847d-4fe7-ba93-3e03ab7748c0) is article about that topic.

# Exceptions
This chapter looks closer to exceptions and how exceptions are handled.

## Rethrowing exceptions incorrectly
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

❌ **Bad** Creating a new exception object with the original as `InnerException` can make debugging harder.
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

💡 Info: Sometimes hiding the original stack trace might be the goal due to for example security related reasons. In the average case re-throwing via `throw;` is preferred.