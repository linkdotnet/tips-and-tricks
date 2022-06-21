# `Lazy<T>`
This section will look closer into the [`Lazy<T>`](https://docs.microsoft.com/en-us/dotnet/api/system.lazy-1?view=net-6.0) type.

## Use the correct `LazyThreadSafetyMode` option
If you are using `Lazy` only in a single-threaded manner, you can save some resources when providing either the `isThreadSafe` parameter or pass in the correct [`LazyThreadSafetyMode`](https://docs.microsoft.com/en-us/dotnet/api/system.threading.lazythreadsafetymode?view=net-6.0). The default is `LazyThreadSafetyMode.ExecutionAndPublication` which creates locks to ensure the synchronization.

❌ **Bad** Use the multi-thread safe version in a single-threaded application.
```csharp
var instance = new Lazy<MyClass>(() => new MyClass());
```

✅ **Good** Pass the `isThreadSafe` parameter.
```csharp
var instance = new Lazy<MyClass>(() => new MyClass(), isThreadSafe: false);
```

✅ **Good** Pass the correct `LazyThreadSafetyMode` parameter.
```csharp
var instance = new Lazy<MyClass>(() => new MyClass(), LazyThreadSafetyMode.None);
```

### Benchmark
```csharp
public class LazyBenchmark
{
    [Params(
        LazyThreadSafetyMode.None,
        LazyThreadSafetyMode.PublicationOnly,
        LazyThreadSafetyMode.ExecutionAndPublication)]
    public LazyThreadSafetyMode Option { get; set; }

    [Benchmark]
    public int Lazy()
    {
        var lazy = new Lazy<MyClass>(() => new MyClass(), isThreadSafe: false);
        return lazy.Value.MyFunc();
    }
}

public class MyClass
{
    public int MyFunc() => 2;
}
```

Results:
```csharp
| Method |               Option |     Mean |    Error |   StdDev |
|------- |--------------------- |---------:|---------:|---------:|
|   Lazy |                 None | 28.06 ns | 0.641 ns | 1.797 ns |
|   Lazy |      PublicationOnly | 32.25 ns | 0.676 ns | 1.013 ns |
|   Lazy | Execu(...)ation [23] | 41.82 ns | 0.829 ns | 1.655 ns |

```