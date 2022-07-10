# Boxing / Unboxing
This chapter shows (hidden) pitfalls in terms of boxing and unboxing.

## Enumerate through `IList<struct>` vs `List<struct>`
When you call `List<struct>.GetEnumerator()` (which will be done in every foreach loop) you get a struct named `Enumerator`. When calling `IList<struct>.GetEnumerator()` you get a variable of type `IEnumerable<struct>`, which contains a boxed version of your value type. Each individual enumeration step is slower due to the boxing nature. In performance critical section this can be important.

❌ **Bad** This will box every individual integer in the list when enuemrated through.
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
> LINQ queries like `Sum()` have special overloads for a bunch of primitive datatypes like `int`, `float` and so on to avoid exactly that boxing behavior.
> Be aware if you have your custom value type, that also then boxing can apply.

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
        foreach (var number in numbersList)
            sum += number;

        return sum;
    }
    
    [Benchmark]
    public int GetSumOfIList()
    {
        var sum = 0;
        foreach (var number in numbersIList)
            sum += number;

        return sum;
    }
}
```

Results:
```csharp
|        Method |      Mean |     Error |    StdDev | Ratio | RatioSD |
|-------------- |----------:|----------:|----------:|------:|--------:|
|  GetSumOfList |  9.425 us | 0.0380 us | 0.0355 us |  1.00 |    0.00 |
| GetSumOfIList | 44.019 us | 0.1565 us | 0.1464 us |  4.67 |    0.02 |
```