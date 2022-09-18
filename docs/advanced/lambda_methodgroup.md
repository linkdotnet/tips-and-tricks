# Lambda vs method group
This article will show the differences between a lamdbda expression and a method group.

## Prefer lambda expression over method groups
Lambda expression can be cached by the .net runtime where as methods groups are not. This can lead to additional allocations if you use .net6 or below (it was improved with .net7).

❌ **Bad** Use method group.
```csharp
_ints.Where(IsEven);
```

✅ **Good** Use lambda expression.
```csharp
_ints.Where(i => IsEven(i));
```

### Benchmark
```csharp
public class LambdaVsMethodGroup
{
    private IEnumerable<int> _ints = Enumerable.Range(0, 100);

    [Benchmark(Baseline = true)]
    public List<int> MethodGroup() => _ints.Where(IsEven).ToList();

    [Benchmark]
    public List<int> Lambda() => _ints.Where(i => IsEven(i)).ToList();

    private static bool IsEven(int i) => i % 2 == 0;
}
```

Results:
```csharp
|      Method |     Mean |    Error |   StdDev | Ratio | RatioSD |
|------------ |---------:|---------:|---------:|------:|--------:|
| MethodGroup | 570.4 ns | 10.88 ns | 11.18 ns |  1.00 |    0.00 |
|      Lambda | 517.6 ns |  6.20 ns |  5.80 ns |  0.91 |    0.02 |
```