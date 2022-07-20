# SIMD - Single Input Multiple Data
SIMD describes the ability the process a single operation simultaneously on multiple data. This can give a big edge on independent data which can be processed in parallel. More information can be found [here](https://steven-giesel.com/blogPost/d80d9367-3a1f-407f-9bdb-067fae9ea527).

## Compare two lists with SIMD
To speed-up the process of two lists, we can vectorize those lists and compare multiple chunks in parallel. To vectorize a list we can use **SSE** (**S** IMD **S** treaming **E** xtensions) in combination with some helper tools from the .NET Framework itself.

❌ **Bad** Use LINQ queries, which can be slow.
```csharp
var isEqual = list1.SequenceEquals(list2);
```

✅ **Good** Use SSE to vectorize the list and check in parallel if the chunks are the same.
```csharp
public bool SIMDContains()
{
    // Lists are not equal when the count is differnet
    if (list1.Count != list2.Count)
        return false;

    // Create a Span<Vector<int>> from our original list.
    // We don't have to worry about the internal size, etc...
    var list1AsVector = MemoryMarshal.Cast<int, Vector<int>>(CollectionsMarshal.AsSpan(list1));
    var list2AsVector = MemoryMarshal.Cast<int, Vector<int>>(CollectionsMarshal.AsSpan(list2));

    for (var i = 0; i < list1AsVector.Length; i++)
    {
        // Compare the chunks of list1 and list2
        if (!Vector.EqualsAll(list1AsVector[i], list2AsVector[i]))
            return false;
    }

    return true;
}
```

> 💡 Info: Check with [`Vector.IsHardwareAccelerated`](https://docs.microsoft.com/en-us/dotnet/api/system.numerics.vector.ishardwareaccelerated?view=net-6.0) if SSE is available. If not the emulated / software version can be slower than LINQ due to the overhead.

### Benchmark
```csharp
public class Benchmark
{
    private readonly List<int> list1 = Enumerable.Range(0, 1_000).ToList();
    private readonly List<int> list2 = Enumerable.Range(0, 1_000).ToList();

    [Benchmark(Baseline = true)]
    public bool LINQSequenceEquals() => list1.SequenceEqual(list2);

    [Benchmark]
    public bool SIMDEquals()
    {
        if (list1.Count != list2.Count)
            return false;

        var list1AsVector = MemoryMarshal.Cast<int, Vector<int>>(CollectionsMarshal.AsSpan(list1));
        var list2AsVector = MemoryMarshal.Cast<int, Vector<int>>(CollectionsMarshal.AsSpan(list2));

        for (var i = 0; i < list1AsVector.Length; i++)
        {
            if (!Vector.EqualsAll(list1AsVector[i], list2AsVector[i]))
                return false;
        }

        return true;
    }
}
```

Results:
```csharp
|             Method |       Mean |    Error |   StdDev | Ratio |
|------------------- |-----------:|---------:|---------:|------:|
| LINQSequenceEquals | 3,487.4 ns | 13.72 ns | 10.71 ns |  1.00 |
|         SIMDEquals |   137.3 ns |  0.98 ns |  0.91 ns |  0.04 |
```

## Get the sum of a list or array
SIMD can be used to divide and conquer the problem of retrieving the sum of a list. The idea is to cut the list in smaller chunks and add them up via SIMD instuctions. This approach can faster the speed significantly.

❌ **Bad** Using plain old LINQ to get a sum of a list or array of integers.
```csharp
_list.Sum();
```

✅ **Good** Using SIMD instructions to divide and conquer and parallelize the addition of the individual vectors.
```csharp
public int SumSIMD()
{
    var accVector = Vector<int>.Zero;

    // For any array use
    // var spanOfVectors = MemoryMarshal.Cast<int, Vector<int>>(new Span<int>(myArray));
    var spanOfVectors = MemoryMarshal.Cast<int, Vector<int>>(CollectionsMarshal.AsSpan(_list));
    foreach (var vector in spanOfVectors)
    {
        accVector = Vector.Add(accVector, vector);
    }

    // Scalar-Product of our vector against the Unit vector is its sum
    var result = Vector.Dot(accVector, Vector<int>.One);
    return result;
}
```

### Benchmark
```csharp
public class Benchmark
{
    private readonly List<int> _list = Enumerable.Range(0, 1_000).ToList();
    
    [Benchmark(Baseline = true)]
    public int ListSort() => _list.Sum();

    [Benchmark]
    public int SumSIMD()
    {
        var accVector = Vector<int>.Zero;

        // For any array use
        // var spanOfVectors = MemoryMarshal.Cast<int, Vector<int>>(new Span<int>(myArray));
        var spanOfVectors = MemoryMarshal.Cast<int, Vector<int>>(CollectionsMarshal.AsSpan(_list));
        foreach (var vector in spanOfVectors)
        {
            accVector = Vector.Add(accVector, vector);
        }

        // Scalar-Product of our vector against the Unit vector is its sum
        var result = Vector.Dot(accVector, Vector<int>.One);
        return result;
    }
}
```
Results:
```csharp
|   Method |       Mean |    Error |   StdDev | Ratio |
|--------- |-----------:|---------:|---------:|------:|
| ListSort | 4,493.1 ns | 88.84 ns | 83.10 ns |  1.00 |
|  SumSIMD |   117.7 ns |  0.44 ns |  0.41 ns |  0.03 |
```