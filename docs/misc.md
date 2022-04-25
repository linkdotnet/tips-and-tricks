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