# struct
`struct`s can have a positive impact on performance due to their nature of living on the stack instead of the heap.
Of course `struct` will be put onto the heap if they outlive their stack frame.

## Passing by value can be faster than by reference
If the `struct` is small enough (at best as wide as a machine word, so 8 bytes on 64bit applications or 4 bytes on 32 bit applications) passing a `struct` by reference can be significant faster than any reference.
The reason is that copying a `struct` on the stack is a cheap operation when the `struct` is small enough. A reference in contrast is always as wide as the machine word but on top, for reading values, it has to be dereferences.  

> 💡 Info: In detail explanation can be found [here](https://steven-giesel.com/blogPost/d205e99c-e784-49be-a2e6-7f9c44ab890f).

## Allocation of small `struct`s are cheaper than reference types
Heap allocation is more expensive than creating an object on the stack.
If used in loops with only local usage of the (small) `struct` they can improve the performance.

❌ **Bad** Classes are relatively expensive
```csharp
 public void SlimClass()
{
    var array = new SlimClass[1000];
    for (var i = 0; i < 1000; i++)
        array[i] = new SlimClass();
}
```

✅ **Good** Structs are cheap to create
```csharp
 public void SlimStruct()
{
    var array = new SlimStruct[1000];
    for (var i = 0; i < 1000; i++)
        array[i] = new SlimStruct();
}
```

⚠️ **Warning** If the array of a struct is larger than 85kb it will be moved to the **L**arge **O**bject **H**eap which can give a major performance penalty.

### Benchmark
```csharp
[GroupBenchmarksBy(BenchmarkLogicalGroupRule.ByCategory), CategoriesColumn]
public class RegexBenchmark
{
    [Params(10, 1_000)]
    public int ObjectsToCreate { get; set; }

    [Benchmark(Baseline = true), BenchmarkCategory("Slim")]
    public void SlimClass()
    {
        var array = new SlimClass[ObjectsToCreate];
        for (var i = 0; i < ObjectsToCreate; i++)
            array[i] = new SlimClass();
    }

    [Benchmark, BenchmarkCategory("Slim")]
    public void SlimStruct()
    {
        var array = new SlimStruct[ObjectsToCreate];
        for (var i = 0; i < ObjectsToCreate; i++)
            array[i] = new SlimStruct();
    }
    
    [Benchmark(Baseline = true), BenchmarkCategory("Fat")]
    public void FatClass()
    {
        var array = new FatClass[ObjectsToCreate];
        for (var i = 0; i < ObjectsToCreate; i++)
            array[i] = new FatClass();
    }

    [Benchmark, BenchmarkCategory("Fat")]
    public void FatStruct()
    {
        var array = new FatStruct[ObjectsToCreate];
        for (var i = 0; i < ObjectsToCreate; i++)
            array[i] = new FatStruct();
    }
}

public class SlimClass { }
public struct SlimStruct { }

public class FatClass
{
    public int A, B, C, D, E, F, G, H, I, J, K, L, M, N, Q, R, S, T, U, V, W, X, Y, Z;
}

public struct FatStruct
{
    public int A, B, C, D, E, F, G, H, I, J, K, L, M, N, Q, R, S, T, U, V, W, X, Y, Z;
}
```

Results:
```csharp
|     Method | Categories | ObjectsToCreate |          Mean |       Error |      StdDev | Ratio | RatioSD |   Gen 0 |   Gen 1 |   Gen 2 | Allocated | Alloc Ratio |
|----------- |----------- |---------------- |--------------:|------------:|------------:|------:|--------:|--------:|--------:|--------:|----------:|------------:|
|   FatClass |        Fat |              10 |    248.246 ns |   7.1119 ns |  20.2908 ns |  1.00 |    0.00 |  0.2923 |       - |       - |    1224 B |        1.00 |
|  FatStruct |        Fat |              10 |     71.918 ns |   2.1169 ns |   6.0054 ns |  0.29 |    0.03 |  0.2352 |       - |       - |     984 B |        0.80 |
|            |            |                 |               |             |             |       |         |         |         |         |           |             |
|   FatClass |        Fat |            1000 | 11,551.343 ns | 231.0578 ns | 247.2293 ns |  1.00 |    0.00 | 28.6865 |  0.0305 |       - |  120024 B |        1.00 |
|  FatStruct |        Fat |            1000 | 54,708.078 ns | 362.9784 ns | 321.7709 ns |  4.73 |    0.12 | 30.2734 | 30.2734 | 30.2734 |   96034 B |        0.80 |
|            |            |                 |               |             |             |       |         |         |         |         |           |             |
|  SlimClass |       Slim |              10 |     60.762 ns |   1.1299 ns |   1.6562 ns |  1.00 |    0.00 |  0.0823 |       - |       - |     344 B |        1.00 |
| SlimStruct |       Slim |              10 |      9.246 ns |   0.1056 ns |   0.0936 ns |  0.15 |    0.01 |  0.0096 |       - |       - |      40 B |        0.12 |
|            |            |                 |               |             |             |       |         |         |         |         |           |             |
|  SlimClass |       Slim |            1000 |  5,968.366 ns | 111.5320 ns | 271.4843 ns |  1.00 |    0.00 |  7.6599 |       - |       - |   32024 B |        1.00 |
| SlimStruct |       Slim |            1000 |    484.738 ns |   8.7875 ns |   8.6305 ns |  0.08 |    0.00 |  0.2441 |       - |       - |    1024 B |        0.03 |

```

## Override equals and operator equals on value types
When comparing a custom `struct` with each other dotnet will use reflection to achieve comparison. In performance critical paths this might not be desirable.

❌ **Bad** Don't provide overrides for `Equals` and similar operations.
```csharp
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
}

var p1 = default(Point);
var p2 = default(Point);
var isSame = p1 == p2; // Uses reflection to achieve comparison
```

✅ **Good** Provide explicit implementations.
```csharp
public struct Point : IEquatable<Point> // Implementing the inteface is optional
{
    public int X { get; set; }
    public int Y { get; set; }

    public override int GetHashCode() => ...
    public override bool Equals(object obj) => ...
    public bool Equals(Point p2) => ...

    public static bool operator ==(Point point1, Point point2)
    {
        return point1.Equals(point2);
    }

    public static bool operator !=(Point point1, Point point2)
    {
        return !point1.Equals(point2);
    }
}
```

> 💡 Info: `record struct`, which were introduced with C# 10, automatically implement `IEquatable`. So by using a `record struct` you get some performance benefits. A more in-depth analysis can be found [here](https://steven-giesel.com/blogPost/073f2a14-ea0f-4241-9729-41ee0f30f90c).

### Benchmark
```csharp
public class ValueTypeEquals
{
    private readonly PointNoOverride _noOverrideP1 = new();
    private readonly PointNoOverride _noOverrideP2 = new();
    private readonly PointRecord _pointRecordP1 = new(0, 0);
    private readonly PointRecord _pointRecordP2 = new(0, 0);

    [Benchmark(Baseline = true)]
    public bool IsSameNoOverride() => _noOverrideP1.Equals(_noOverrideP2);

    [Benchmark]
    public bool IsSameOverride() => _pointRecordP1.Equals(_pointRecordP2);
}

public struct PointNoOverride
{
    public int X { get; init; }
    public int Y { get; init; }
}
// record struct implements IEquatable<T>
public record struct PointRecord(int X, int Y);
```

Results:
```csharp
|           Method |       Mean |     Error |    StdDev | Ratio |  Gen 0 | Allocated | Alloc Ratio |
|----------------- |-----------:|----------:|----------:|------:|-------:|----------:|------------:|
| IsSameNoOverride | 24.5980 ns | 0.7290 ns | 2.0682 ns |  1.00 | 0.0115 |      48 B |        1.00 |
|   IsSameOverride |  0.6466 ns | 0.0499 ns | 0.0555 ns |  0.03 |      - |         - |        0.00 |
```

## Override `GetHashCode` when used in a `Dictionary`
When a custom defined `struct` is used as a key in a `Dictionary` reflection is used to calculate the has of the current object. In performance critical paths that is not desirable.

❌ **Bad** Rely on the reflection to calculate the hash
```csharp
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
}

var dictionary = new Dictionary<Point, WorldObject>();
...
var worldObjectAtP1 = dictionary[p1];
```

✅ **Good** Provide `GetHashCode` implementation.
```csharp
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }

    public override int GetHashCode() => HashCode.Combine(X, Y);
}

var dictionary = new Dictionary<Point, WorldObject>();
...
var worldObjectAtP1 = dictionary[p1];
```

> 💡 Info: `record struct`, which were introduced with C# 10, automatically implement a non reflective `GetHashCode`. So by using a `record struct` you get some performance benefits. A more in-depth analysis can be found [here](https://steven-giesel.com/blogPost/073f2a14-ea0f-4241-9729-41ee0f30f90c).

### Benchmark
```csharp
[MemoryDiagnoser]
public class StructGetHashCode
{
    private static readonly PointNoOverride pointNoOverride = new();
    private static readonly PointRecord pointOverride = new();
    private readonly Dictionary<PointNoOverride, int> dictionaryNoOverride = new()
    {
        { pointNoOverride, 1 }
    };
    private readonly Dictionary<PointRecord, int> dictionaryOverride = new()
    {
        { pointOverride, 1 }
    };

    [Benchmark(Baseline = true)]
    public int GetFromNoOverride() => dictionaryNoOverride[pointNoOverride];

    [Benchmark]
    public int GetFromOverride() => dictionaryOverride[pointOverride];
}

public struct PointNoOverride
{
    public int X { get; init; }
    public int Y { get; init; }
}
// record struct implements GetHashCode
public record struct PointRecord(int X, int Y);
```

Results:
```csharp
|            Method |     Mean |    Error |   StdDev |   Median | Ratio | RatioSD |  Gen 0 | Allocated | Alloc Ratio |
|------------------ |---------:|---------:|---------:|---------:|------:|--------:|-------:|----------:|------------:|
| GetFromNoOverride | 57.35 ns | 1.303 ns | 3.801 ns | 56.31 ns |  1.00 |    0.00 | 0.0172 |      72 B |        1.00 |
|   GetFromOverride | 19.45 ns | 0.413 ns | 0.386 ns | 19.30 ns |  0.33 |    0.02 |      - |         - |        0.00 |
```