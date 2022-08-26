## Use the correct `RegexOption` when defining the regular expression.

When defining a regular expression, one can define [`RegexOption`](https://docs.microsoft.com/en-us/dotnet/api/system.text.regularexpressions.regexoptions?view=net-6.0)s, which enable fine-grained control over the regular expression engine. That also works with the newly introduced [`RegexGenerator`](https://devblogs.microsoft.com/dotnet/regular-expression-improvements-in-dotnet-7/) introduced in .net 7, as they use the same attributes.

### `Compiled`
This flags (which is the default for `RegexGenerator` in .net7, so you do not have to provide that argument for them) enables the compilation of the regular expression into the MSIL code of the application. That brings performance improvements in trade-off in expense of a bigger startup time of your application. There are only small scenarios, where this flag does not make sense.

### `ExplicitCapture`
If you don't work with capture groups, you can safely provide this flag as this tells the regex engine **not** to look for implicit captures. Explicit captures are something like this: `?<groupname>`.

### Defining multiple values
As `RegexOption` are defined as flags, you can provide multiple arguments delimited by the `|` characters (bitwise or).

```csharp
public class RegexBenchmark
{
    private const string Sentence = "Hello this is an example of stock tips!";
    private const string RegexString = @"(\W|^)stock\stips(\W|$)";
    private static readonly Regex StockTips = new(RegexString);
    private static readonly Regex StockTipsCompiled = new(RegexString, RegexOptions.Compiled);
    private static readonly Regex StockTipsExplicitCapture = new(RegexString, RegexOptions.ExplicitCapture);
    private static readonly Regex StockTipsAllCombined = new(RegexString,
        RegexOptions.Compiled | RegexOptions.ExplicitCapture);

    [Benchmark(Baseline = true)]
    public bool HasMatch() => StockTips.IsMatch(Sentence);

    [Benchmark]
    public bool HasMatchCompiled() => StockTipsCompiled.IsMatch(Sentence);

    [Benchmark]
    public bool HasMatchExplicitCapture() => StockTipsExplicitCapture.IsMatch(Sentence);

    [Benchmark]
    public bool HasMatchCombined() => StockTipsAllCombined.IsMatch(Sentence);
}
```

Results:
```csharp
|                  Method |       Mean |    Error |   StdDev | Ratio |
|------------------------ |-----------:|---------:|---------:|------:|
|                HasMatch | 1,348.4 ns | 17.48 ns | 16.35 ns |  1.00 |
|        HasMatchCompiled |   650.0 ns |  3.59 ns |  3.36 ns |  0.48 |
| HasMatchExplicitCapture | 1,062.4 ns |  5.50 ns |  5.14 ns |  0.79 |
|        HasMatchCombined |   528.0 ns |  3.09 ns |  2.89 ns |  0.39 |
```