# strings
This chapter shows some tips, tricks and pitfalls in combinations with `string`'s.

## Use `StringBuilder` when concatenating a lot of strings
As `string`'s are immutable in C# concatenating those will result in many allocations and loss of performance. In scenarios where many strings get concatenated a [`StringBuilder`](https://docs.microsoft.com/en-us/dotnet/api/system.text.stringbuilder?view=net-6.0) is preferred. The same applies for operations like `string.Join` or adding a single character to a `string`.

❌ **Bad** Will use a lot of allocations and will result in performance penalty.
```csharp
var outputString = "";

for (var i = 0; i < 25; i++)
{
    outputString += "test" + i;
}
```

✅ **Good** Usage of `StringBuilder` will reduce the allocations dramatically and also performs better.
```csharp
var stringBuilder = new StringBuilder();

for (var i = 0; i < 25; i++)
{
    stringBuilder.Append("test" + i);
}
```

Here a comparison of both methods:

```
|              Method | Times |        Mean |     Error |      StdDev |      Median | Ratio | RatioSD |   Gen 0 | Allocated |
|-------------------- |------ |------------:|----------:|------------:|------------:|------:|--------:|--------:|----------:|
|        StringConcat |    10 |    298.7 ns |   1.86 ns |     1.45 ns |    298.3 ns |  1.00 |    0.00 |  0.4549 |      2 KB |
| StringBuilderAppend |    10 |    436.2 ns |   5.31 ns |     4.15 ns |    437.1 ns |  1.46 |    0.02 |  0.4206 |      2 KB |
|                     |       |             |           |             |             |       |         |         |           |
|        StringConcat |   100 | 15,025.7 ns | 739.33 ns | 2,011.40 ns | 14,579.0 ns |  1.00 |    0.00 | 39.5203 |    161 KB |
| StringBuilderAppend |   100 |  5,989.5 ns | 415.73 ns | 1,225.78 ns |  6,416.1 ns |  0.41 |    0.12 |  3.9063 |     16 KB |
```