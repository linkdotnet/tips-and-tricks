# ValueTuple
This section looks into tips and tricks of the `ValueTuple` type which was introduced in C# 7.

## Easy `IEquatable` implementation
`ValueTuple` can be leveraged to have an easy and readable implementation of [`IEquatable`](https://docs.microsoft.com/en-us/dotnet/api/system.iequatable-1?view=net-6.0).

```csharp
public class Dimension : IEquatable<Dimension>
{
    public Dimension(int height, int width)
        => (Height, Width) = (height, width);

    public int Width { get; }
    public int Height { get; }

    public bool Equals(Dimension other)
        => (Width, Height) == (other?.Width, other?.Height);

    public override bool Equals(object obj)
        => obj is Dimension dimension && Equals(dimension);

    public override int GetHashCode() 
        => (Width, Height).GetHashCode();
}
```

## Swap two values
`ValueTuple` can be used to swap two (or more) variables without the usage of a temporary variable.

❌ **Bad** Using temporary variable.
```csharp
int a = 10;
int b = 15;

var tmp = a;
a = b;
b = tmp;
```

✅ **Good** Using `ValueTuple`.
```csharp
int a = 10;
int b = 15;

(a, b) = (b, a);
```