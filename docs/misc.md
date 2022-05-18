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