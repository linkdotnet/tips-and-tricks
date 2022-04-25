# stackalloc

The following chapter will show some tips and tricks around the `stackalloc` and its usage.
> A `stackalloc` expression allocates a block of memory on the stack. A stack allocated memory block created during the method execution is automatically discarded when that method returns. <sup>Taken from [here](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/stackalloc)</sup>

## Allocate only small amounts of memory
The **stack** has a limited size and attempting to allocate a huge<sup>relatively speaking</sup> amount of memory and if taken too much a `StackOverflowException` is thrown which is not recoverable. Linux might have a big stack size but embedded devices can limit the size to the kilobyte range.

❌ **Bad** Allocate a huge amount.
```csharp
Span<byte> = stackalloc byte[1024 * 1024];
```

Also be careful if taken user input which can crash the program:
```csharp
Span<byte> = stackalloc byte[userInput];
```

✅ **Good** Take small amounts of memory.
```csharp
Span<byte> = stackalloc byte[1024];
```

✅ **Good** Fallback to traditional heap allocation if over a specific threshold.
```csharp
Span<byte> = userInput <= 1024 ? stackalloc byte[1024] : new byte[userInput];
```

## `stackalloc` is not zero initialized
In contrast to arrays allocated via `new` `stackalloc` does not zero initialize arrays. Especially if one in the caller chain uses the [`SkipLocalsInitAttribute`](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.skiplocalsinitattribute?view=net-6.0).

❌ **Bad** Don't relay on zero initialized arrays.
```csharp
Span<int> buffer = stackalloc int[3];
buffer[0] = 1;
buffer[1] = 1;

int result = buffer[0] + buffer[1] + buffer[2]; // Is not necessarily 2
```

✅ **Good**  Call `Clear` or `Fill` to initialize the array if needed.
```csharp
Span<int> buffer = stackalloc int[3];
buffer.Clear(); // or buffer.Fill(0);
buffer[0] = 1;
buffer[1] = 1;

int result = buffer[0] + buffer[1] + buffer[2]; // Is not necessarily 2
```