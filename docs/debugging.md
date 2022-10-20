# Debugging
This chapter looks into features, which can make debugging an application easier.

## Using `DebuggerDisplayAttribute`
The [`DebuggerDisplayAttribute`](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.debuggerdisplayattribute?view=net-6.0) determines how a class or structure is displayed in the debugger. This can save time if you have complex object, where you have a few key information, which are the most significant.

```csharp
var steven = new Person("Steven", 31, Array.Empty<string>());
Console.WriteLine(steven);

[DebuggerDisplay("{Name} is {Age} years old with {PetNames.Count} pets.")]
public record Person(string Name, int Age, IReadOnlyCollection<string> PetNames);
```

In the debugger window or if you hover over the `steven` object the debugger will show:
```no-class
Steven is 31 years old with 0 pets.
```