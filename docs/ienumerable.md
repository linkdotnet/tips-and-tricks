# IEnumerable
All about `IEnumerable<T>`. [`IEnumerable<T>`](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1?view=net-6.0#:~:text=Without%20Iterator%20%3D%20206kb-,Remarks,%3E%20and%20ConcurrentStack.) are often used for deferred execution, which can naturally inherit some complexity.

## Possible multiple enumeration
As `IEnumerable<T>` is not a materialized collection but more cursor on the current element of the enumeration, using multiple functions on `IEnumerable<T>` can lead to possible multiple enumerations.
This becomes a penalty when retrieving the content is expensive as the work is done multiple times.

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