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
        return await client.GetStringAsync(url);
}
```

> 💡 Info: Eliding the `async` keyword will also elide the whole state machine. In very hot paths that might be worth a consideration. In normal cases one should not elide the keyword. The allocations one is saving is depending on the circumstances but a normally very very small especially if only smaller objects are passed around. Also performance-wise there is no big gain when eliding the keyword (we are talking nano seconds). Please measure first and act afterwards.

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

## `async void`
The problem with `async void` is first they are not awaitable and second they suffer the same problem with exceptions and stack trace as discussed a bit earlier. It is basically fire and forget.

❌ **Bad** Not awaited
```csharp
public async void DoAsync()
{
	await SomeAsyncOp();
}
```

✅ **Good** return `Task` instead of `void`
```csharp
public async Task DoAsync()
{
	await SomeAsyncOp();
}
```

> 💡 Info: There are valid cases for `async void` like top level event handlers.

## `List<T>.ForEach` with `async`
`List<T>.ForEach` and in general a lot of LINQ methods don't go well with `async` `await`:

❌ **Bad** Is the same as `async void`
```csharp
var ids = new List<int>();
// ...
ids.ForEach(id => _myRepo.UpdateAsync(id));
```

One could thing adding `async` into the lamdba would do the trick:

❌ **Bad** Still the same as `async void` because `List<T>.ForEach` takes an `Action` and not a `Func<Task>`.
```csharp
var ids = new List<int>();
// ...
ids.ForEach(async id => await _myRepo.UpdateAsync(id));
```

✅ **Good** Enumerate through the list via `foreach`
```csharp
foreach (var id in ids)
{
	await _myRepo.UpdateAsync(id);
}
```