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
    PrintProducts(products.where(p => p.Title == "IPhone")); 
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
    PrintProducts(products.where(p => p.Title == "IPhone")); 
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