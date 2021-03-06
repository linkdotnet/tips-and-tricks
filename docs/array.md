# Array
This page will look into arrays and their usage.

## Prefer jagged arrays over multidimensional
In a multidimensional array, each element in each dimension has the same, fixed size as the other elements in that dimension. In a jagged array, which is an array of arrays, each inner array can be of a different size. By only using the space that's needed for a given array, no space is wasted. Also from a performance point of view a jagged array performs generally better than a multidimensional array.

❌ **Bad** Multidimensional array performs less optimal and can waste space. 
```csharp
int[,] multiDimArray = { {1, 2}, {3, 4}};
```

✅ **Good** Jagged array performs generally better and only uses the really needed space.
```csharp
int[][] jaggedArray = { new int[] {1, 2}, new int [] {3, 4}};
```

### Benchmark
```csharp
[MemoryDiagnoser]
public class Arrays
{
    [Params(25, 100)]
    public int Size { get; set; }

    [Benchmark]
    public int[][] CreateAndFillJaggedArray()
    {
        int[][] jaggedArray = new int[Size][];
        for (var i = 0; i < Size; i++)
        {
            jaggedArray[i] = new int[Size];
            for (var j = 0; j < Size; j++) jaggedArray[i][j] = i + j;
        }

        return jaggedArray;
    }
    
    [Benchmark]
    public int[,] CreateAndFillMultidimensionalArray()
    {
        int[,] multidimArray = new int[Size, Size];
        for (var i = 0; i < Size; i++)
        {
            for (var j = 0; j < Size; j++) 
                multidimArray[i, j] = i + j;
        }

        return multidimArray;
    }
}
```

Result:
```csharp
|                             Method | Size |        Mean |     Error |    StdDev |      Median |   Gen 0 |  Gen 1 | Allocated |
|----------------------------------- |----- |------------:|----------:|----------:|------------:|--------:|-------:|----------:|
|           CreateAndFillJaggedArray |   25 |    717.7 ns |  15.51 ns |  43.50 ns |    700.3 ns |  0.8183 |      - |   3.34 KB |
| CreateAndFillMultidimensionalArray |   25 |  1,480.2 ns |  29.49 ns |  82.21 ns |  1,463.7 ns |  0.6065 |      - |   2.48 KB |
|           CreateAndFillJaggedArray |  100 |  9,895.6 ns | 143.22 ns | 126.96 ns |  9,900.8 ns | 10.3302 | 0.0153 |  42.21 KB |
| CreateAndFillMultidimensionalArray |  100 | 16,374.7 ns | 319.00 ns | 567.02 ns | 16,311.2 ns |  9.5215 | 0.0305 |   39.1 KB |
```

## Locality matters
Memory location matters. Accessing memory location sequentially as opposed to "jumping" around to read some memory address.

❌ **Bad** Do not access memory sequentially.
```csharp
public int NotLocal()
{
    var sum = 0;
    for (var i = 0; i < _table.GetLength(1); i++)
    {
        for (var j = 0; j < _table.GetLength(0); j++) 
            sum += _table[j, i];
    }

    return sum;
}
```

✅ **Good** Accessing memory sequentially.
```csharp
public int Local()
{
    var sum = 0;
    for (var i = 0; i < _table.GetLength(0); i++)
    {
        for (var j = 0; j < _table.GetLength(1); j++) 
            sum += _table[i, j];
    }

    return sum;
}
```

### Benchmark
Results for the given example above:
```csharp
|   Method |     Mean |   Error |  StdDev | Ratio | RatioSD |
|--------- |---------:|--------:|--------:|------:|--------:|
|    Local | 120.8 us | 2.35 us | 2.70 us |  1.00 |    0.00 |
| NotLocal | 158.6 us | 3.09 us | 3.56 us |  1.31 |    0.05 |
```