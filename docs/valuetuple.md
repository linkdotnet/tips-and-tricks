# ValueTuple
This section looks into tips and tricks of the `ValueTuple` type which was introduced in C# 7.

## Easy `IEquatable` implementation
`ValueTuple` can be leveraged to have an easy implementation of [`IEquatable`](https://docs.microsoft.com/en-us/dotnet/api/system.iequatable-1?view=net-6.0).

```csharp
public class Dimension : IEquatable<Dimension>
{
    public Dimension(int height, int width)
    {
        Height = height;
        Width = width;
    }

    public int Width { get; }
    public int Height { get; }

    public bool Equals(Dimension other)
        => (Width, Height) == (other?.Width, other?.Height);

    public override bool Equals(object obj)
        => obj is Dimension metrics && Equals(metrics);

    public override int GetHashCode() 
        => (Width, Height).GetHashCode();
}
```