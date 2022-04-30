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

> 💡 Info: Some more information about Unicode, UTF-8, UTF-16 and UTF-32 can be found [here](https://medium.com/bobble-engineering/emojis-from-a-programmers-eye-ca65dc2acef0).
