# Collections
This chapter looks closer to collections and its best practices.

## Passing collections as a method parameter 

If your method needs to accept a collection as a parameter, then generally `IEnumerable<T>` is ok and this offers the most flexibility to the caller. It doesn't matter what concrete collection type they use, and it even allows them to pass lazily evaluated sequences in, This is a `general` approach and for preventing multiple enumeration we can call `ToList()` in first of executing method or we can use `IReadOnlyCollection<T>` or `IReadOnlyList` for preventing multiple enumeration.

❌ **Bad** using `IEnumerable<T>` generally as input collection type is ok, but without using `ToList()` in first of executing could create multiple enumeration.
```csharp
void PrintProducts(IEnumerable<Product> products)
{
    // First Enumeration.
    Console.WriteLine($"Product Count is: {products.Count()}.");

    // Second Enumeration.
    foreach(var product in products)
    {
        Console.WriteLine($"Id: {product.Id} - Title: {product.Title}");
    }
}

void Caller() 
{
    PrintProducts(products.Where(p => p.Title == "IPhone")); 
}
```

✅ **Good** With using `IReadOnlyCollection<T>` in our parameter specifically, we can prevent unexpected behaviors and multiple enumerations.
```csharp
void PrintProducts(IReadOnlyCollection<Product> products)
{
    Console.WriteLine($"Product Count is: {products.Count}.");

    foreach(var product in products)
    {
        Console.WriteLine($"Id: {product.Id} - Title: {product.Title}");
    }
}

void Caller() 
{
    PrintProducts(products.where(p => p.Title == "IPhone").AsReadOnly()); 
}
```

✅ **Good** With using `IEnumerable<T>` in our parameter specifically and calling `ToList()` in first line of executing, we can prevent unexpected behaviors and multiple enumerations.
```csharp
void PrintProducts(IEnumerable<Product> products)
{
    var items = products.ToList();

    Console.WriteLine($"Product Count is: {items.Count}.");

    foreach(var product in items)
    {
        Console.WriteLine($"Id: {product.Id} - Title: {product.Title}");
    }
}

void Caller() 
{
    PrintProducts(products.Where(p => p.Title == "IPhone")); 
}
```

## Possible multiple enumeration with `IEnumerable<T>`
As `IEnumerable<T>` is not a materialized collection but more cursor on the current element of the enumeration, using multiple functions on `IEnumerable<T>` can lead to possible multiple enumerations.
This becomes a penalty when retrieving the content is expensive as the work is done multiple times, for example it could be an expensive operation that goes to a database multiple time with each enumeration, or it could even return different results in each enumeration.

❌ **Bad** Will enumerate the enumeration twice
```csharp
IEnumerable<int> integers = GetIntegers();
var min = integers.Min();
var max = integers.Max();

Console.Write($"Min: {min} / Max: {max}");

IEnumerable<int> GetIntegers()
{
    Console.WriteLine("Getting integer");
    yield return 1;
    yield return 2;
}
```

Will output:
> Getting integer  
Getting integer  
Min: 1 / Max: 2

✅ **Good** Enumeration get materialized once via `ToList`
```csharp
IEnumerable<int> integers = GetIntegers();
var integerList = integers.ToList();
var min = integerList.Min();
var max = integerList.Max();

Console.Write($"Min: {min} / Max: {max}");

IEnumerable<int> GetIntegers()
{
    Console.WriteLine("Getting integer");
    yield return 1;
    yield return 2;
}
```

Will output:
> Getting integer  
Min: 1 / Max: 2

## Returning null for `IEnumerable<T>`
A lot of **LINQ** operations are built upon the premise that `IEnumerable<T>`, even though it is a reference type, should not be `null` at any given time.

❌ **Bad** Instead of an empty enumeration a `null` will be returned which can confuse a consumer.
```csharp
public IEnumerable<MyDto> GetAll()
{
    if (...)
    {
        return null;
    }
    ...
}
```

✅ **Good** Instead return an empty enumeration.
```csharp
public IEnumerable<MyDto> GetAll()
{
    if (...)
    {
        return Enumerable.Empty<MyDto>();
    }
    ...
}
```

## Consider `IReadOnlyCollection<T>` if you always return an in memory list

 Many developers assume that `IEnumerable` is the best type to return a collection from a method. It's not a bad choice, but it does mean that the caller cannot make any assumptions about the collection they have been given, If enumerating the return value will be an in-memory operation or a potentially expensive action. They also don't know if they can safely enumerate it more than once. So often the caller ends up doing a `.ToList()` or similar on the collection you passed, which is wasteful if it was already a `List<T>` already. This is actually another case in which `IReadOnlyCollection<T>` can be a good fit if you know your method is always going to return an `in-memory collection`.

 So It gives your caller the ability to access the count of items and iterate through as many times as they like.


 ❌ **Bad** With returning `IEnumerable<Product>`, We don't know the result is an in-memory operation or a potentially expensive action so It can be a risky operation with multiple enumeration.
```csharp
public IEnumerable<Product> GetAllProducts()
{
    // ...
}
```

✅ **Good** If we are sure, our result is a in-memory collection it is better we return a `IReadOnlyCollection<Product>` instead of `IEnumerable<Product>`.
```csharp
public IReadOnlyCollection<Product> GetAllProducts()
{
    // ...
}
```

## Don't use zero-length arrays
When initializing a zero-length array unnecessary memory allocation have to be made. Using `Array.Empty` can increase readability and reduce memory consumption as the memory is shared across all invocations of the method.

❌ **Bad** Return zero-initialized array.
```csharp
public int[] MyFunc(string input)
{
    if (input == null)
        return new int[0];
}
```

✅ **Good** Use `Array.Empty<int>` to increase readability and less allocations.
```csharp
public int[] MyFunc(string input)
{
    if (input == null)
        return Array.Empty<int>();
}
```

> 💡 Info: For every generic version of `Array.Empty<T>` the instance and memory is shared.
```csharp
ReferenceEquals(new int[0], new int[0]); // Is false
ReferenceEquals(Array.Empty<int>(), Array.Empty<int>()); // Is true
ReferenceEquals(Array.Empty<int>(), Array.Empty<float>()); // Is false
```

## Specify capacity of collections
Often times collections like [`List<T>`](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.-ctor?view=net-6.0#system-collections-generic-list-1-ctor(system-int32)) offer constructors, which give the option to pass in the expected capacity of the list.
The `DefaultCapacity` of a list is **4**. If added another item it will grow by the factor of 2. As the underlying structure of a `List<T>` is a conventional array and arrays can't shrink or grow in size, a new array has to be created with the new size plus the content of the old array. This takes times and needs to unnecessary allocations. This is especially important for hot paths.

❌ **Bad** No capacity is given so the internal array has to be resized often.
```csharp
var numbers = new List<int>();
for (var i = 0; i < 20_000; i++)
    numbers.Add(i);
```

✅ **Good** Capacity is given and the internal array of the `List<T>` has the correct size.
```csharp
var numbers = new List<int>(20_000);
for (var i = 0; i < 20_000; i++)
    numbers.Add(i);
```

### Benchmark
Result of the given example above:
```csharp
|              Method |      Mean |    Error |   StdDev | Ratio | RatioSD |   Gen 0 |   Gen 1 |   Gen 2 | Allocated | Alloc Ratio |
|-------------------- |----------:|---------:|---------:|------:|--------:|--------:|--------:|--------:|----------:|------------:|
| ListWithoutCapacity | 102.64 us | 1.999 us | 3.340 us |  1.00 |    0.00 | 41.6260 | 41.6260 | 41.6260 | 256.36 KB |        1.00 |
|    ListWithCapacity |  39.55 us | 0.789 us | 1.698 us |  0.38 |    0.02 | 18.8599 |       - |       - |  78.18 KB |        0.30 |
```

## Use `HashSet` to avoid linear searches
If a collection is used often times to check whether or not an item is present a [`HashSet`](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.hashset-1?view=net-6.0) might be a better option than a `List<T>`. The initial cost to create a `HashSet` is more expensive than a `List<T>` but set-based operations are faster. In this case this holds true even for [`Any`](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.any?view=net-6.0), which is basically a linear search through the array.

❌ **Bad** If often checking for an item in a collection `Any` and `Contains` from `List<T>` can be slow.
```csharp
var userIds = Enumerable.Range(0, 1_000).ToList();
usersIds.Any(u => u == 999); // Basically goes through 999 elements
userIds.Contains(999); // Faster than Any but still slower than HashSet
```

✅ **Good** `HashSet` performs faster for set-based operations.
```csharp
var userIds = Enumerable.Range(0, 1_000).ToHashSet();
userIds.Contains(999);
```

### Benchmark
```csharp
public class ListVsHashSet
{
    private const int Count = 10_000;
    private readonly List<int> _list = Enumerable.Range(0, Count).ToList();
    private readonly HashSet<int> _hashSet = Enumerable.Range(0, Count).ToHashSet();

    [Benchmark(Baseline = true)]
    public bool ListContains() => _list.Contains(Count - 1);

    [Benchmark]
    public bool ListAny() => _list.Any(l => l == Count - 1);

    [Benchmark]
    public bool HashSetContains() => _hashSet.Contains(Count - 1);
}
```

Results:
```csharp
|          Method |          Mean |         Error |        StdDev |  Ratio | RatioSD |
|---------------- |--------------:|--------------:|--------------:|-------:|--------:|
|    ListContains |  1,279.002 ns |    10.5561 ns |     9.3577 ns |  1.000 |    0.00 |
|         ListAny | 92,892.521 ns | 1,848.0088 ns | 3,379.1886 ns | 75.344 |    2.48 |
| HashSetContains |      5.019 ns |     0.1364 ns |     0.1401 ns |  0.004 |    0.00 |
```

## Use `HashSet` for set based operations like finding the intersection of two collections
When performing set based operations like finding the intersection or union of two collection `HashSet` can be superior to a `List`.
Be aware that the API is a bit different with a `HashSet` as the `HashSet` is not pure and modifies the `HashSet` on which the operation is called on.

❌ **Bad** Using LINQ and `List<T>` to find the intersection of two collectionss 
```csharp
var evenNumbers = Enumerable.Range(0, 2_000).Where(i => i % 2 == 0).ToList();
var dividableByThree = Enumerable.Range(0, 2_000).Where(i => i % 3 == 0).ToList();

var evenNumbersDividableByThree = evenNumbers.Intersect(dividableByThree).ToList();
```

✅ **Good** Using a `HashSet` to find the intersection of two collections.
```csharp
var evenNumbers = Enumerable.Range(0, 2_000).Where(i => i % 2 == 0).ToHashSet();
var dividableByThree = Enumerable.Range(0, 2_000).Where(i => i % 3 == 0).ToHashSet();

var evenNumbersDividableByThree = new HashSet(evenNumbers);
evenNumbersDividableByThree.IntersectWith(dividableByThree);
```

### Benchmark
Runtimes for the given example above:

Result:
```csharp
|              Method |     Mean |    Error |   StdDev | Ratio |
|-------------------- |---------:|---------:|---------:|------:|
|    InterSectionList | 25.69 us | 0.416 us | 0.369 us |  1.00 |
| InterSectionHashSet | 11.88 us | 0.108 us | 0.090 us |  0.46 |
```

## Use `ArraySegment<T>` for efficient slicing of arrays
When slicing an array a new array is created. This is especially expensive when the array is large. `ArraySegment<T>` is a struct that holds a reference to the original array and the start and end index of the slice. This is much more efficient than creating a new one. Be aware that the `ArraySegment<T>` is a read-only view of the original array and any changes to the original array will be reflected in the `ArraySegment<T>`.

❌ **Bad** We create a new array when taking a slice of the original array, leading to extra memory allocations. 
```csharp

public int[] Slice(int[] array, int offset, int count)
{
    var result = new int[count];
    Array.Copy(array, offset, result, 0, count);
    return result;
}

var originalArray = new int[] { 1, 2, 3, 4, 5 };
var slicedArray = Slice(originalArray, 1, 3); // Creates a new array { 2, 3, 4 }
```

✅ **Good** Use `ArraySegment<T>` to create a view of the original array without creating a new array, avoiding the extra memory allocation.
```csharp
public ArraySegment<int> Slice(int[] array, int offset, int count)
{
    return new ArraySegment<int>(array, offset, count);
}

var originalArray = new int[] { 1, 2, 3, 4, 5 };
var slicedArray = Slice(originalArray, 1, 3); // A view over { 2, 3, 4 } in the original array
```
