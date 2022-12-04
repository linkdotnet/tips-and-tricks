# strings
This chapter shows some tips, tricks and pitfalls in combination with `string`'s.

## Use `StringBuilder` when concatenating a lot of strings
As `string`'s are immutable in C# concatenating those will result in many allocations and loss of performance. In scenarios where many strings get concatenated a [`StringBuilder`](https://docs.microsoft.com/en-us/dotnet/api/system.text.stringbuilder?view=net-6.0) is preferred. The same applies to operations like `string.Join` or add a single character to a `string`.

❌ **Bad** Will use a lot of allocations and will result in a performance penalty.
```csharp
var outputString = "";

for (var i = 0; i < 25; i++)
{
    outputString += "test" + i;
}
```

✅ **Good** Usage of `StringBuilder` will reduce the allocations dramatically and also performs better.
Here is a comparison of both methods:

```
|              Method | Times |        Mean |     Error |      StdDev |      Median | Ratio | RatioSD |   Gen 0 | Allocated |
|-------------------- |------ |------------:|----------:|------------:|------------:|------:|--------:|--------:|----------:|
|        StringConcat |    10 |    298.7 ns |   1.86 ns |     1.45 ns |    298.3 ns |  1.00 |    0.00 |  0.4549 |      2 KB |
| StringBuilderAppend |    10 |    436.2 ns |   5.31 ns |     4.15 ns |    437.1 ns |  1.46 |    0.02 |  0.4206 |      2 KB |
|                     |       |             |           |             |             |       |         |         |           |
|        StringConcat |   100 | 15,025.7 ns | 739.33 ns | 2,011.40 ns | 14,579.0 ns |  1.00 |    0.00 | 39.5203 |    161 KB |
| StringBuilderAppend |   100 |  5,989.5 ns | 415.73 ns | 1,225.78 ns |  6,416.1 ns |  0.41 |    0.12 |  3.9063 |     16 KB |
```

## Getting the printable length of a string or character
Retrieving the length of a string can often be done via `"my string".Length`, which in a lot of scenarios is good enough. Under the hood [`string.Length`](https://docs.microsoft.com/en-us/dotnet/api/system.string.length?view=net-6.0) will return the number of characters in this string object. Unfortunately that does not always map one to one with the printed characters on screen.

❌ **Bad** Assuming every printed characters has the same length.
```csharp
Console.Write("The following string has the length of 1: ");
Console.WriteLine("🏴󠁧󠁢󠁥󠁮󠁧󠁿".Length);
```

Output:
> The following string has the length of 1: 14

Emojis can consist out of "other" emojis making the length very variable. Also other charaters like the following are wider:
```csharp
Console.WriteLine("𝖙𝖍𝖎𝖘".Length); // Prints 8
```

✅ **Good** Take [`StringInfo.LengthInTextElements`](https://docs.microsoft.com/en-us/dotnet/api/system.globalization.stringinfo.lengthintextelements?view=net-6.0) to know the amount of printed characters.
```csharp
Console.WriteLine(new StringInfo("🏴󠁧󠁢󠁥󠁮󠁧󠁿").LengthInTextElements);
```

Output:
> 1

To summarize: `string.Length` will give return the internal array size not the length of printed characters. `StringInfo.LengthInTextElements` will return the amount of printed characters.

> 💡 Info: Some more information about Unicode, UTF-8, UTF-16 and UTF-32 can be found [here](https://medium.com/bobble-engineering/emojis-from-a-programmers-eye-ca65dc2acef0).

## Use `StringComparison` instead of `ToLowerCase` or `ToUpperCase` for insensitive comparison

Lots of code is using `"ABC".ToLowerCase() == "abc".ToLowerCase()` to compare two strings, when casing doesn't matter. The problem with that code is `ToLowerCase` as well as `ToUpperCase` creates a new string instance, resulting in unnecessary allocations and performance loss. 


❌ **Bad** Using new allocations for comparing strings.
```csharp
var areStringsEqual = "abc".ToUpperCase() == "ABC".ToUpperCase();
```

✅ **Good** Use of the `string.Equals` overload with the appropriate `StringComparison` technique.
```csharp
var areStringsEqual = string.Equals("ABC", "abc", StringComparison.OrdinalIgnoreCase);
```

### Benchmark
```csharp
[MemoryDiagnoser]
[HideColumns(Column.Arguments)]
public class StringBenchmark
{
    [Benchmark(Baseline = true)]
    [Arguments(
        "HellO WoRLD, how are you? You are doing good?",
        "hElLO wOrLD, how Are you? you are doing good?")]
    public bool AreEqualToLower(string a, string b) => a.ToLower() == b.ToLower();

    [Benchmark(Baseline = false)]
    [Arguments(
        "HellO WoRLD, how are you? You are doing good?",
        "hElLO wOrLD, how Are you? you are doing good?")]
    public bool AreEqualStringComparison(string a, string b) => string.Equals(a, b, StringComparison.OrdinalIgnoreCase);
}
```

Results:
```
|                   Method |     Mean |    Error |   StdDev | Ratio |   Gen0 | Allocated | Alloc Ratio |
|------------------------- |---------:|---------:|---------:|------:|-------:|----------:|------------:|
|          AreEqualToLower | 60.93 ns | 1.008 ns | 0.943 ns |  1.00 | 0.0356 |     224 B |        1.00 |
| AreEqualStringComparison | 16.10 ns | 0.030 ns | 0.028 ns |  0.26 |      - |         - |        0.00 |
```

## Prefer `StartsWith` over `IndexOf() == 0`
The problem with IndexOf is, that it will go through the whole string in the worst case. StartsWith on the contrary will directly abort one the first mismatch.

❌ **Bad** Using `IndexOf` which might run through the whole string.
```csharp
var startsWithHallo = "Hello World".IndexOf("Hallo") == 0;
```

✅ **Good** More readable, as well as more performant with `StartsWith`.
```csharp
var startsWithHallo = "Hello World".StartsWith("Hallo");
```

### Benchmark
```csharp
[Benchmark(Baseline = true)]
[Arguments("That is a sentence", "Thzt")]
public bool IndexOf(string haystack, string needle) => haystack.IndexOf(needle, StringComparison.OrdinalIgnoreCase) == 0;

[Benchmark]
[Arguments("That is a sentence", "Thzt")]
public bool StartsWith(string haystack, string needle) =>
    haystack.StartsWith(needle, StringComparison.OrdinalIgnoreCase);
```

Results:
```
|     Method |           haystack | needle |      Mean |     Error |    StdDev | Ratio |
|----------- |------------------- |------- |----------:|----------:|----------:|------:|
|    IndexOf | That is a sentence |   Thzt | 21.966 ns | 0.1584 ns | 0.1482 ns |  1.00 |
| StartsWith | That is a sentence |   Thzt |  3.066 ns | 0.0142 ns | 0.0126 ns |  0.14 |
```

## Prefer `AsSpan` over `Substring`
`Substring` always allocates a new string object on the heap. If you have a method that accepts a `Span<char>` or `ReadOnlySpan<char>` you can avoid these allocations. A prime example is `string.Concat` that takes a `ReadOnlySpan<char>` as an input parameter.

❌ **Bad** Creating new `string` objects that are directly discarded afterward.
```csharp
var output = Text.Substring(0, 5) + " - " + Text.Substring(11, 4);
```

✅ **Good** Directly use the underlying memory to avoid heap allocations.
```csharp
var output = string.Concat(Text.AsSpan(0, 5), " - ", Text.AsSpan(11, 4));
```

### Benchmark
```csharp
[MemoryDiagnoser]
public class Benchmark
{
    [Params("Hello dear world")]
    public string Text { get; set; }

    [Benchmark]
    public string Substring()
        => Text.Substring(0, 5) + " - " + Text.Substring(11, 4);

    [Benchmark]
    public string AsSpanConcat()
        => string.Concat(Text.AsSpan(0, 5), " - ", Text.AsSpan(11, 4));
}
```

Results:
```no-class
|       Method |             Text |     Mean |    Error |   StdDev |   Gen0 | Allocated |
|------------- |----------------- |---------:|---------:|---------:|-------:|----------:|
|    Substring | Hello dear world | 21.18 ns | 0.085 ns | 0.076 ns | 0.0179 |     112 B |
| AsSpanConcat | Hello dear world | 10.20 ns | 0.021 ns | 0.018 ns | 0.0076 |      48 B |
```