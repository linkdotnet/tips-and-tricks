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