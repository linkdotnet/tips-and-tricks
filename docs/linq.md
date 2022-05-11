# LINQ
This chapter will handle topics about <abbr title="Language-Integrated Query">**LINQ**</abbr>.

## Using `is` and `as` instead of `OfType`

Using `is` and `as` cast should not be used in LINQ as it degrades readability and to some extend also performance (neglactable small under normal circumstances).

❌ **Bad** Less readable and unnecessary casts.
```csharp
persons.Where(p => p is FullTimeEmployee).Select(p => p as FullTimeEmployee);
persons.Where(p => p is FullTimeEmployee).Select(p => (FullTimeEmployee)p);
persons.Select(p => p as FullTimeEmployee).Where(p => p != null);
```

✅ **Good** Using `OfType` is cleaner and more readable.
```csharp
persons.OfType<FullTimeEmployee>();
```

## Using `Count()` instead of `All` or `Any`
The problem with `Count()` is that LINQ has to enumerate every single entry in the enumeration to get the full count, where as `Any`/`All` would return immediately as soon as the condition is not satisfied anymore.
In general conditions (excluding LINQ to SQL) `Any`/`All` in the worst case have the same runtime as `Count` but in best and average cases they are faster and more readable.

❌ **Bad** Less readable and unnecessary enumerations 
```csharp
persons.Count() > 0
persons.Count(MyPredicate) > 0
persons.Count(MyPredicate) == 0
persons.Count(MyPredicate) == persons.Count()
```

✅ **Good** More readable and avoids unnecessary enumerations
```csharp
persons.Any()
persons.Any(MyPredicate)
!persons.Any(MyPredicate)
persons.All(MyPredicate)
```
