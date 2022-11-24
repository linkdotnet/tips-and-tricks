# Boxing / Unboxing
This chapter shows (hidden) pitfalls in terms of boxing and unboxing.

## Enumerate through `IList<struct>` vs `List<struct>`
When you call `List<struct>.GetEnumerator()` (which will be done in every foreach loop) you get a struct named `Enumerator`. When calling `IList<struct>.GetEnumerator()` you get a variable of type `IEnumerator<struct>`, which contains a boxed version of your value type. In the performance critical section, this can be important.

❌ **Bad** This will box every individual integer in the list when enumerated through.
```csharp
private int GetSum(IList<int> numbers)
{
    foreach (var number in numbers)
    // ...
}
```

✅ **Good** Using the `List<T>` implementation avoids the boxing.
```csharp
private int GetSum(List<int> numbers)
{
    foreach (var number in numbers)
    // ...
}
```

> 💡 Info: The same applies to `Collection<struct>` and `ICollection<struct>`. Basically everytime when you use the interface, changes are that the values get boxed.
> This also happens with LINQ queries.

### Benchmark
```csharp
public class Benchmark
{
    private readonly List<int> numbersList = Enumerable.Range(0, 10_000).ToList();
    private readonly IList<int> numbersIList = Enumerable.Range(0, 10_000).ToList();

    [Benchmark(Baseline = true)]
    public int GetSumOfList()
    {
        var sum = 0;
        foreach (var number in numbersList) { sum += number; }
        return sum;
    }
    
    [Benchmark]
    public int GetSumOfIList()
    {
        var sum = 0;
        foreach (var number in numbersIList) { sum += number; }
        return sum;
    }
}
```

Results:
```csharp
|        Method |      Mean |     Error |    StdDev | Ratio | RatioSD | Allocated | Alloc Ratio |
|-------------- |----------:|----------:|----------:|------:|--------:|----------:|------------:|
|  GetSumOfList |  9.392 us | 0.0186 us | 0.0165 us |  1.00 |    0.00 |         - |          NA |
| GetSumOfIList | 44.098 us | 0.3359 us | 0.3142 us |  4.69 |    0.03 |      40 B |          NA |
```

## Using `List<struct>` in LINQ queries will box the enumerator
As LINQ queries are built upon `IEnumerable<T>` passing a list of value types to a LINQ query will box the enumerator. This is especially unwanted in high performance path in your application.

❌ **Bad** Using LINQ query to get the sum of a `List<int>`.
```csharp
List<int> numbers = GetNumbers();

var sum = numbers.Sum(); // This will box the enumerator
```

✅ **Good** Use foreach with the enumerator of the list to avoid boxing.
```csharp
List<int> numbers = GetNumbers();

var sum = 0;
foreach (var number in numbers)
    sum += number;
```

### Benchmark
```csharp
public class Benchmark
{
    private readonly List<int> numbersList = Enumerable.Range(0, 10_000).ToList();

    [Benchmark(Baseline = true)]
    public int GetSumViaForeachList()
    {
        var sum = 0;
        foreach (var number in numbersList)
            sum += number;

        return sum;
    }

    [Benchmark]
    public int GetSumViaLINQ() => numbersList.Sum();
}

```

Results:
```csharp
|               Method |      Mean |     Error |    StdDev | Ratio | RatioSD | Allocated | Alloc Ratio |
|--------------------- |----------:|----------:|----------:|------:|--------:|----------:|------------:|
| GetSumViaForeachList |  9.391 us | 0.0210 us | 0.0176 us |  1.00 |    0.00 |         - |          NA |
|        GetSumViaLINQ | 43.778 us | 0.1257 us | 0.1176 us |  4.66 |    0.02 |      40 B |          NA |
```