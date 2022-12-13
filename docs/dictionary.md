# `Dictionary`
This page will give some tips and tricks while handling the `Dictionary` type.

## Safe way of getting a key from a `Dictionary`
To get an element from a dictionary safely, we should check beforehand whether or not that element exists otherwise, we get an exception. Often times people use the `ContainsKey` method followed by the indexer of the dictionary. The problem here is that we do two lookups, even though we only need one (hashtable lookup). Also people find the second option more readable and it shows the intent better.

❌ **Bad** Use `ContainsKey` in combination with the indexer.
```csharp
Dictionary<string, string> mapping = GetMapping();
if (mapping.ContainsKey("some-key"))
{
  var value = mapping["some-key"];
  DoSomethingWithValue(value);
  // ...
}
```

✅ **Good** Avoid two lookups and get the value directly.
```csharp
Dictionary<string, string> mapping = GetMapping();
if (mapping.TryGetValue("some-key"), out var value)
{
  DoSomethingWithValue(value);
  // ...
}
```

## Define the initial size when creating a `Dictionary` when known
`Dictionary` works internally a bit similiar to a `List`. Once a certain threshold of elements is reached, the internal type holding the information has to be resized. Resizing in this context means, create a new object of that type, which is bigger and copy all entries from the old storage into the new one. This operation is costly. When you build a `Dictionary` and the amount of entries is known, it is a better option to provide that capacity for the `Dictionary`.

❌ **Bad** Don't define the capacity, if known.
```csharp
var dictionary = new Dictionary<string, string>();
foreach (var key in _keys)
    dictionary[key] = key;
```

✅ **Good** Defining the expected capacity.
```csharp
var dictionary = new Dictionary<string, string>(NumberOfKeys);
foreach (var key in _keys) 
    dictionary[key] = key;
```

### Benchmark
```csharp
[MemoryDiagnoser()]
public class Benchmark
{
    private List<string> _keys;

    [Params(5, 10, 100)]
    public int NumberOfKeys { get; set; }

    [GlobalSetup]
    public void Setup()
    {
        _keys = Enumerable.Range(0, NumberOfKeys).Select(n => n.ToString()).ToList();
    }

    [Benchmark(Baseline = true)]
    public Dictionary<string, string> NoSize()
    {
        var dictionary = new Dictionary<string, string>();
        foreach (var key in _keys) dictionary[key] = key;

        return dictionary;
    }

    [Benchmark]
    public Dictionary<string, string> WithSize()
    {
        var dictionary = new Dictionary<string, string>(NumberOfKeys);
        foreach (var key in _keys) dictionary[key] = key;

        return dictionary;
    }
}
```

Result:
```csharp
|   Method | NumberOfKeys |        Mean |     Error |   StdDev | Ratio |   Gen0 | Allocated | Alloc Ratio |
|--------- |------------- |------------:|----------:|---------:|------:|-------:|----------:|------------:|
|   NoSize |            5 |   150.09 ns |  1.953 ns | 1.731 ns |  1.00 | 0.2217 |     464 B |        1.00 |
| WithSize |            5 |    95.13 ns |  0.974 ns | 0.863 ns |  0.63 | 0.1568 |     328 B |        0.71 |
|          |              |             |           |          |       |        |           |             |
|   NoSize |           10 |   270.06 ns |  1.353 ns | 1.200 ns |  1.00 | 0.4740 |     992 B |        1.00 |
| WithSize |           10 |   161.27 ns |  1.288 ns | 1.205 ns |  0.60 | 0.2103 |     440 B |        0.44 |
|          |              |             |           |          |       |        |           |             |
|   NoSize |          100 | 2,538.13 ns | 10.519 ns | 9.325 ns |  1.00 | 4.8714 |   10192 B |        1.00 |
| WithSize |          100 | 1,574.53 ns |  8.705 ns | 7.717 ns |  0.62 | 1.4935 |    3128 B |        0.31 |
```