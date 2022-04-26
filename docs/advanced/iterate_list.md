# Iterate through a list

The page shows three different ways iterating through a List: `for`, `foreach` and via `Span<T>`.

Not all of them are the same in terms of behavior.

`foreach` is well known and shows the intend very well. It is slower than a `for` loop mainly because foreach has to check if the collection gets modified. A `for` loop can suffer from "one of" issues where you access an index out of bounds. How often does that happen with a `foreach`? So even though `foreach` is slower, it is the safest and most verbose option (when you don't need the index).

With [`CollectionMarshal.AsSpan`](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.collectionsmarshal.asspan?view=net-6.0)<sup>since .NET 5</sup> we can grab the internal array of List and use it as Span. This is the fastest thanks to a lot of optimizations. Like the `for` loop there are no checks if the original collection gets modified. You can also use the Span in a `foreach` struct and still the same holds true.
Be aware as the unsafe nature of that function things can get messy. For example the `Span<T>` can contain stale data, which in the original list is already removed.

## Performance comparison

```csharp
public class Benchmark
{
    [Params(10, 100, 1000)] 
    public int ArraySize { get; set; }
    
    private List<int> numbers;

    [GlobalSetup]
    public void Setup()
    {
        numbers = Enumerable.Repeat(0, ArraySize).ToList();
    }
    
    [Benchmark(Baseline = true)]
    public void ForEach() 
    {
        foreach (var num in numbers)
        {
        }
    }

    [Benchmark]
    public void For()
    {
        for (var i = 0; i < ArraySize; i++)
        {
            _ = numbers[i];
        }
    }

    [Benchmark]
    public void Span()
    {
        var collectionAsSpan = CollectionsMarshal.AsSpan(numbers);
        for (var i = 0; i < ArraySize; i++)
        {
            _ = collectionAsSpan[i];
        }
    }
}
```

Results:
```
|  Method | ArraySize |         Mean |      Error |     StdDev | Ratio | RatioSD |
|-------- |---------- |-------------:|-----------:|-----------:|------:|--------:|
| ForEach |        10 |    14.558 ns |  0.3226 ns |  0.8328 ns |  1.00 |    0.00 |
|     For |        10 |     6.053 ns |  0.1391 ns |  0.2166 ns |  0.41 |    0.03 |
|    Span |        10 |     3.906 ns |  0.0988 ns |  0.0924 ns |  0.27 |    0.01 |
|         |           |              |            |            |       |         |
| ForEach |       100 |   161.745 ns |  2.8664 ns |  3.4123 ns |  1.00 |    0.00 |
|     For |       100 |    67.268 ns |  1.1961 ns |  1.1188 ns |  0.41 |    0.01 |
|    Span |       100 |    50.935 ns |  1.0039 ns |  1.1158 ns |  0.31 |    0.01 |
|         |           |              |            |            |       |         |
| ForEach |      1000 | 1,508.880 ns | 29.7007 ns | 36.4751 ns |  1.00 |    0.00 |
|     For |      1000 |   636.547 ns | 12.6593 ns | 25.8595 ns |  0.42 |    0.02 |
|    Span |      1000 |   337.738 ns |  6.7882 ns | 16.6517 ns |  0.22 |    0.01 |

```
## Stale data
Be aware that getting the memory slice of a `List` and modifying the `List` afterwards will **not** update the `Span`.

```csharp
var myList = new List<int> { 1, 2 };
var listSlice = CollectionsMarshal.AsSpan(myList);
myList.Add(3);
Console.Write(listSlice.Length); // This will only print 2
```