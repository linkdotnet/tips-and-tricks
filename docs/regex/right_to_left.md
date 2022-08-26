## Right to left search
The [`RegexOption.RightToLeft`](https://docs.microsoft.com/en-us/dotnet/api/system.text.regularexpressions.regexoptions?view=net-6.0) allows the regex engine to search from the end to the start. That can bring major gains if you expect the match at the end of the string rather than the beginning.

```csharp
public class RegexBenchmark
{
    private const string Sentence = "Hello this is an example of stock tips!";
    private const string RegexString = @"(\W|^)stock\stips(\W|$)";
    private static readonly Regex StockTips = new(RegexString, RegexOptions.Compiled);
    private static readonly Regex StockTipsRightToLeft = new(RegexString,
        RegexOptions.Compiled | RegexOptions.RightToLeft);

    [Benchmark(Baseline = true)]
    public bool HasMatch() => StockTips.IsMatch(Sentence);

    [Benchmark]
    public bool HasMatchRightToLeft() => StockTipsRightToLeft.IsMatch(Sentence);
}
```

Results:
```csharp
|              Method |      Mean |    Error |   StdDev | Ratio |
|-------------------- |----------:|---------:|---------:|------:|
|            HasMatch | 649.54 ns | 1.923 ns | 1.799 ns |  1.00 |
| HasMatchRightToLeft |  48.23 ns | 0.153 ns | 0.143 ns |  0.07 |
```