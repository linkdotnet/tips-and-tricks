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

> 💡 Info: Eliding the `async` keyword will also elide the whole state machine. In very hot paths that might be worth a consideration. In normal cases one should not elide the keyword. The allocations one is saving is depending on the circumstances but a normally very very small especially if only smaller objects are passed around. Also performance-wise there is no big gain when eliding the keyword (we are talking nano seconds). Please measure first and act afterwards.

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

## Favor `await` over synchronous calls
Using blocking calls instead of `await` can lead to potential deadlocks and other side effects like a poor stack trace in case of an exception and less scalability in web frameworks like ASP.NET core.

❌ **Bad** This call blocks the thread.
```csharp
public async Task SomeOperationAsync()
{
	await ...
}

public void Do()
{
	SomeOperationAsync().Wait();
}
```

✅ **Good** Use `async` & `await` in the whole chain
```csharp
public async Task SomeOperationAsync()
{
	await ...
}

public async Task Do()
{
	await SomeOperationAsync();
}
```

## Favor `GetAwaiter().GetResult()` over `Wait` and `Result`
`Task.GetAwaiter().GetResult()` is preferred over `Task.Wait` and `Task.Result` because it propagates exceptions rather than wrapping them in an AggregateException. 

❌ **Bad**
```csharp
string content = DownloadAsync().Result;
```

✅ **Good** 
```csharp
string content = DownloadAsync().GetAwaiter().GetResult();
```

## Don't use `Task.Delay` for small precise waiting times
[`Task.Delay`](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.delay?view=net-6.0#System_Threading_Tasks_Task_Delay_System_Int32_)'s internal timer is dependent on the underlying OS. On most windows machines this resolution is about 15ms.

So: `Task.Delay(1)` will not wait one millisecond but something between one and 15 milliseconds.

```csharp
var stopwatch = Stopwatch.StartNew();
await Task.Delay(1);
stopwatch.Stop(); // Don't account the Console.WriteLine into the timer
Console.WriteLine($"Delay was {stopwatch.ElapsedMilliseconds} ms");
```

Will print for example:
> Delay was 6 ms

## Properly awaiting concurrent tasks
Often times tasks are independent of each other and can be awaited independently.

❌ **Bad** The following code will run roughly 1 second.
```csharp

await DoOperationAsync();
await DoOperationAsync();

async Task DoOperationAsync()
{
	await Task.Delay(500);
} 
```

✅ **Good** When tasks or their data is independent they can be awaited independently for maximum benefits. The following code will run roughly 0.5 seconds.

```csharp
var t1 = DoOperationAsync();
var t2 = DoOperationAsync();
await t1;
await t2;

async Task DoOperationAsync()
{
	await Task.Delay(500);
} 
```

An alternative to this would be [`Task.WhenAll`](https://docs.microsoft.com/en-us/documentation/):
```csharp
var t1 = DoOperationAsync();
var t2 = DoOperationAsync();
await Task.WhenAll(t1, t2); // Can also be inlined

async Task DoOperationAsync()
{
	await Task.Delay(500);
} 
```

## `ConfigureAwait` with `await using` statement
Since C# 8 you can provide an `IAsyncDisposable` which allows to have asynchrnous code in the `Dispose` method. Also this allows to call the following construct:

```csharp
await using var dbContext = await dbContextFactory.CreateDbContextAsync().ConfigureAwait(false);
```

In this example `CreateDbContextAsync` uses the `ConfigureAwait(false)` but **not** the `IAsyncDisposable`. To make that work we have to break apart the statment like this:

```csharp
var dbContext = await dbContextFactory.CreateDbContextAsync().ConfigureAwait(false);
await using (dbContext.ConfigureAwait(false))
{
	// ...
}
```

The last part has the "ugly" snippet that you have to introduce a new "block" for the `using` statement. For that there is a easy workaround:
```csharp
var blogDbContext = await dbContextFactory.CreateDbContextAsync().ConfigureAwait(false);
await using var _ = blogDbContext.ConfigureAwait(false);
// You don't need the {} block here
```

## Avoid Async in Constructors
Constructors are meant to initialize objects synchronously, as their primary purpose is to set up the initial state of the object. When you need to perform asynchronous operations during object initialization, using async methods in constructors can lead to issues such as deadlocks or incorrect object initialization. Instead, use a static asynchronous factory method or a separate asynchronous initialization method to ensure safe and proper object initialization.

Using async operations in constructors can be problematic for several reasons like **Deadlocks**, **Incomplete Initialization**, and **Exception Handling**.

❌ **Bad** Calling async methods in constructors

```csharp
public class MyClass
{
    public MyClass()
    {
        InitializeAsync().GetAwaiter().GetResult();
    }

    private async Task InitializeAsync() => await LoadDataAsync();
```

✅ **Good**: Option 1 - Static asynchronous factory method:

```csharp
public class MyClass
{
    private MyClass() { }

    public static async Task<MyClass> CreateAsync()
    {
        var instance = new MyClass();
        await instance.InitializeAsync();
        return instance;
    }

    private async Task InitializeAsync() => await LoadDataAsync();
```

✅ **Good**: Option 2 - Separate asynchronous initialization method:

```csharp
public class MyClass
{
    public async Task InitializeAsync() => await LoadDataAsync();
```