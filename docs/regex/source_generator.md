# Source Code Generators
This page will look at how source code generators can impact regular expressions.

## Source Code Generators and Regular Expressions
Since .NET 7 we have the possibility to use source code generators for regular expressions. The advantage over the regular approach is that we can get the same performance as `new Regex("...", RegexOptions.Compiled)` and the startup benefit of `Regex.CompileToAssembly`, but without the complexity of `CompileToAssembly`. As the code is generated it can be viewed and debugged.

So instead of this code:
```csharp
private static readonly Regex HelloOrWorldCompiled =
        new("Hello|World", RegexOptions.Compiled | RegexOptions.IgnoreCase | RegexOptions.CultureInvariant);
```

We can write it like this:
```csharp
[GeneratedRegex("Hello|World", RegexOptions.IgnoreCase | RegexOptions.CultureInvariant)]
private static partial Regex HelloOrWorldGenerator();
```