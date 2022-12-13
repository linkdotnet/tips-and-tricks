# Misc
A collection of different topics which didn't fit in the other categories.

## `new Random` in a loop
> 💡 Info: This part is only valid for the old .NET Framework and not the newer .NET Core / .NET 5 or 6 versions.

In the .NET Framework creating the [`Random`](https://docs.microsoft.com/en-us/dotnet/api/system.random?view=net-6.0) class without any parameter in very short time spans will create identical outputs.
Especially in for loops this can result in the same number over and over again.
If no seed is provided for the `Random` class, it will create a seed which is time dependent. The resolution of the timer is depending on the implementation of the underlying OS. On most windows system the resolution is 15ms.

❌ **Bad** Will produce the same number 3 times.
```csharp
for(var i = 0; i < 3; i++)
{
	var random = new Random();
	var number = random.Next(6);
	Console.WriteLine(number);
}
```

Prints:
> 4  
> 4  
> 4

✅ **Good** Extract the creation of the `Random` class outside the loop.
```
var random = new Random();

for(var i = 0; i < 3; i++)
{	
	var number = random.Next(6);
	Console.WriteLine(number);
}
```

Prints:
> 5  
> 3  
> 5

## Implementing `GetHashCode`
`GetHashCode` is mainly used for `Dictionary`. Mainly two things are important to say an object (at least from an dictionary point of view) is equal:
 * `GetHashCode` for two objects are the same **and:**
 * `Equals` is the same.

Dotnet does the first because its cheaper to calculate hashes and only if they match to check if the object is really the same.

❌ **Bad** GetHashCode which can cause collision
```csharp
public class Point
{
	public int X { get; set; }
	public int Y { get; set; }

	public override int GetHashCode() => X + Y; // Can lead to massive collisions
}
```

✅ **Good** Since .netcore there is a helper class called: [`HashCode.Combine`](https://docs.microsoft.com/en-us/dotnet/api/system.hashcode.combine?view=net-6.0).
```csharp
public class Point
{
	public int X { get; set; }
	public int Y { get; set; }

	public override int GetHashCode() => HashCode.Combine(X, Y);
}
```

✅ **Good** Also `ValueTuple` can be used for that purpose (since C# 7).
```csharp
public class Point
{
	public int X { get; set; }
	public int Y { get; set; }

	public override int GetHashCode() => (X, Y).GetHashCode();
}
```

## Virtual member calls in constructor
Calling a virtual member in a constructor can lead to unexpected behavior. The reason is that initializers run from most derived to base type but constructors run from base constructor to most derived.

❌ **Bad** This example will lead to a `NullReferenceException`
```csharp
public class Parent
{
	public Parent() { Console.WriteLine(GetType().Name); PrintHello(); }
	public virtual void PrintHello() {}
}

public class Child : Parent
{
	private string foo; 
	public Child()
	{
		foo = "Something";
	}
	
	public override void PrintHello()
	{
		// foo will be null as it gets called in the base constructor before "our"
		// Child constructor is called and initializes the state
		Console.WriteLine(foo.ToUpperInvariant());
	}
}
```
Output:
> Child  
Unhandled exception. System.NullReferenceException: Object reference not set to an instance of an object.

✅ **Good** Avoid virtual member calls via sealing the class or don't make the method virtual.
```csharp
public class Parent
{
	public Parent() { Console.WriteLine(GetType().Name); PrintHello(); }
	public void PrintHello() {}
}

public class Child : Parent
{
	private string foo; 
	public Child()
	{
		
		foo = "Something";
	}
}
```

> 💡 Info: It is perfectly fine to call a virtual member if the caller is the most derived type in the chain and there can't be another derived type (therefore the class is `sealed`).

## Don't have empty finalizers
An empty finalizer does not offer any functionality and also decreases performance. Objects have to live at least one more generation before they can be removed.

❌ **Bad** Empty finalizer defined.
```csharp
public class MyClass
{
	~MyClass() {}
}
```

✅ **Good** Don't define an empty finalizer.
```csharp
public class MyClass
{

}
```
> 💡 Info: More information can be found [here](https://steven-giesel.com/blogPost/3b55d5ac-f62c-4b86-bfa3-62670f614761).

### Benchmark
Results for the given example above:
```csharp
|             Method |       Mean |     Error |    StdDev | Ratio | RatioSD |  Gen 0 |  Gen 1 | Allocated |
|------------------- |-----------:|----------:|----------:|------:|--------:|-------:|-------:|----------:|
|   WithoutFinalizer |   2.205 ns | 0.1230 ns | 0.1416 ns |  1.00 |    0.00 | 0.0057 |      - |      24 B |
| WithEmptyFinalizer | 144.038 ns | 2.9594 ns | 6.1773 ns | 65.32 |    4.19 | 0.0057 | 0.0029 |      24 B |
```

## `Enum.TryParse` unexpected behavior

[`Enum.TryParse`](https://docs.microsoft.com/en-us/dotnet/api/system.enum.tryparse?view=net-6.0) parses a string and tries to convert it into a valid representation of a given enum.
If the operation is successful it returns `true`, otherwise `false`. Have a look at the following code. What will be the output of that code? 

```csharp
var couldParse = Enum.TryParse("3", out WeekendDay weekendDay);

Console.WriteLine($"Could parse: {couldParse}");
Console.WriteLine($"Value: {weekendDay}");

public enum WeekendDay
{
    Saturday = 0,
    Sunday = 1,
}
```

`3` is not a valid `WeekendDay` therefore `false` sure thing. But unfortunately no. The output is 
> Could parse: true  
> Value: 3

You can try this on out [sharplab.io](https://sharplab.io/#v2:EYLgtghglgdgNAFxAJwK7wCYgNQB8ACATAIwCwAUBQG4TIAEAxgPaoA2GACrQM4CmdAXjoBRGKjAA6ACrIAnl2R8AFACIAzCrh0WCOgHVevANa8YGACIRZdAO6GTZy7ICUAbgoV8xAJxKAJCoAwizsdAAOPLwgdADezGyckQC+Km6ePv4qAGoQrKhRsXbGphZWKWmU5PhqdKbi+vYlThQxFHTtdADKEAioyBhWgnQADHBtHZ3oA9ZCxGPkSUA===) if you want.

✅ **Good** Use `Enum.IsDefined` to check if a given value is in the correct range.
With [`Enum.IsDefined`](https://docs.microsoft.com/en-us/dotnet/api/system.enum.isdefined?view=net-6.0) we can check if the value is correct for a given enum.

```csharp
var couldParse = Enum.TryParse("3", out WeekendDay weekendDay);
if (couldParse && Enum.IsDefined(typeof(WeekendDay), weekendDay))
{
	// Only here we have a valid value for WeekendDay
}
```

## Be careful of closures
Closures are "attached" to their parent to close over a certain variable. Basically the anonymous function / lambda is taking an argument outside of his own scope.
The following code demonstrates such behavior:
```csharp
var list = new List<Action>();

for(var i = 0; i < 5; i++)
    list.Add(() => Console.WriteLine(i));

list.ForEach(action => action());
```

And here is the pitfall. The snippet might print something unexpecting:
```no-class
5
5
5
5
5
```
The reason is that behind the scenes the closures creates an anonymous class, which holds a reference to `i` (instead if a copy). That will lead to that all `Action`s point to the same reference of `i`, which after the for loop is `5`. The way to fix this is the following:

```csharp
var list = new List<Action>();

for (var i = 0; i < 5; i++)
{
    var t = i;
    list.Add(() => Console.WriteLine(t));
}

list.ForEach(action => action());
```

This works because copying an interger to a temporary variable will result in a new reference behind the scenes. You can fiddle around with that example on [sharplab.io](https://sharplab.io/#v2:D4AQDABCCMAsDcBYAUCATNFKBuBDAThADYCWAzgC4QC8EAdgKYDuEAMuRQDwxoB8AFAEokyFADMA9oX55CJGhDDwI8zhACsykgGptglAG8UEE8Q4A6AIIATa/yE1eUaAE5+JQcJQBfLMlKU5gBiUgCiuADGABb8kRQkEnSOEHEJdELCQA===).

Additionally adding the `static` keyword to an anonymous function will prohibit any closures. So `list.Add(() => Console.WriteLine(i));` will result in an compiler error.