# ArrayPool
An `ArrayPool` is a reusable memory pool which tries to decrease garbage collection and memory pressure and therefore enables better performance. Using the array pool normally consists out of three parts:

 * Renting some array from the `ArrayPool`.
 * Doing something with the array.
 * Returning the rented array back to the `ArrayPool`.

As seen by the pattern it is most useful when a lot of smaller arrays are needed in a short time. The major downside of an `ArrayPool` is that the consumer is now responsible to give back the memory instead of the GC.

## Using the shared ArrayPool
`ArrayPool<T>.Shared` is a predefined array pool that can directly be used.

```csharp
var array = ArrayPool<int>.Shared.Rent(1024);
// Do something with the array here
ArrayPool<int>.Shared.Return(array);
```

## Not returning the rented array
❌ **Bad** Don't returning the rented array can lead to memory depletion and performance hits as the pool has to be regenerated. The following example uses the `ArrayPool` in the constructor but does not give it back afterward.
```csharp
public class MyArrayType
{
    private int[] _array;

    public MyArrayType()
    {
        _array = ArrayPool<int>.Shared.Rent(1024);
    }
}
```

✅ **Good** Return the array list in `Dispose` to guarantee the `ArrayPool` will not be depleted.
```csharp
public class MyArrayType : IDisposable
{
    private int[] _array;

    public MyArrayType()
    {
        _array = ArrayPool<int>.Shared.Rent(1024);
    }

    public void Dispose()
    {
        if (_array != null)
        {
            ArrayPool<int>.Shared.Return(_array);
        }
    }
}
```

## Assuming you get the exact length you are asking for

❌ **Bad** Assuming that you get the exact amount you asked for. [`Rent`](https://docs.microsoft.com/en-us/dotnet/api/system.buffers.arraypool-1.rent?view=net-6.0) only parameter is called `minimumLength`. That means you can get a bigger array than you are asking for.

```csharp
var array = ArrayPool<int>.Shared.Rent(1024);
for (var i = 0; i < array.Length; i++) // can be more than 1024 iterations
{
    // ...
}

ArrayPool<int>.Shared.Return(array);
```

✅ **Good** Use of a const variable.
```csharp
const int size = 1024;
var array = ArrayPool<int>.Shared.Rent(size);
for (var i = 0; i < size; i++)
{
    // ...
}

ArrayPool<int>.Shared.Return(array);
```

## Assuming it is null initialized
❌ **Bad** Renting array from the array is not necessarily zero-initialized.
```csharp
var array = ArrayPool<int>.Shared.Rent(3);
var zero = array[0] + array[1] + array[2]; // Is not necessary 0
```

The following example shows that the program rents an array from the pool and sets some values. Afterward, it returns this memory to the pool.
Once more retrieving these values, the program can read its old values.
```csharp
for(var i = 0; i < 4;i++)
{
    var array = ArrayPool<int>.Shared.Rent(4);
    array.SetValue(1, i);
    ArrayPool<int>.Shared.Return(array);
}

var rentedArray = ArrayPool<int>.Shared.Rent(8);
for (var i = 0; i < 4; i++)
{
    Console.WriteLine(rentedArray[i]);
}
``` 

Output:
```
1
1
1
1
```

## Performance

```csharp
[MemoryDiagnoser]
public class Benchmark
{
    const int NumberOfItems = 10000;
    
    [Benchmark]
    public void ArrayViaNew()
    {
        var array = new int[NumberOfItems];
    }
    [Benchmark]
    public void SharedArrayPool()
    {
        var array = ArrayPool<int>.Shared.Rent(NumberOfItems);
        ArrayPool<int>.Shared.Return(array);
    }
}
```

Results:
```
|          Method |        Mean |      Error |     StdDev |  Gen 0 | Allocated |
|---------------- |------------:|-----------:|-----------:|-------:|----------:|
|     ArrayViaNew | 3,387.04 ns | 263.598 ns | 773.088 ns | 9.5215 |  40,024 B |
| SharedArrayPool |    40.60 ns |   2.531 ns |   7.462 ns |      - |         - |
```