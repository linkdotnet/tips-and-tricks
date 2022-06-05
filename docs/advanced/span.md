# `Span<T>`
This chapter will look closely to the `Span<T>` type. The whole idea behind `Span<T>` is to not allocate additional memory as it operates on the memory slice behind the given data type.

## Substring from a `string`
Getting a substring of a `string` via the `Substring` method will create a new `string` and therefore a new allocation.

❌ **Bad** Using `Substring` to get a part of a string.
```csharp
var text = "Hello World";
var hello = text.Substring(0, 5);
var hello2 = text[..5]; // Range indexer uses Substring under the hood
```

✅ **Good** Using `AsSpan` to get the underlying memory.
```csharp
var text = "Hello World";
var hello = text.AsSpan().Slice(0, 5);
var hello2 = text.AsSpan()[..5]; 
```

### Benchmark
```csharp
[MemoryDiagnoser]
public class SubstringBenchmark
{
    private const string text = "Hello World";
    
    [Benchmark(Baseline = true)]
    public string Substring()
    {
        return text[..5];
    }
    
    [Benchmark]
    public ReadOnlySpan<char> SpanSlice()
    {
        return text.AsSpan()[..5];
    }
}
```

Results:
```csharp
|    Method |      Mean |     Error |    StdDev | Ratio |  Gen 0 | Allocated | Alloc Ratio |
|---------- |----------:|----------:|----------:|------:|-------:|----------:|------------:|
| Substring | 9.9252 ns | 0.3611 ns | 1.0534 ns |  1.00 | 0.0076 |      32 B |        1.00 |
| SpanSlice | 0.1921 ns | 0.0428 ns | 0.0653 ns |  0.02 |      - |         - |        0.00 |
```