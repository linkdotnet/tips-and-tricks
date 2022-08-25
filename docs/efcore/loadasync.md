## `LoadAsync` - Split queries into smaller chunks

`LoadAsync` in combination with a EF-Core `DbContext` can load related entities. That is useful when handling with big data sets, cartiasian explosion (wanted or unwanted), lots of joins or unions. Big data sets for the reason, that it can happen that the timeout will be reached easily. As with `LoadAsync` the query is split up into smaller chunks, those might not hit the timeout individually.

This code:
```csharp
return await _context.Books
    .Include(b => b.BookCategories)
    .ThenInclude(c => c.Category)
    .Include(c => x.Author.Biography)
    .ToListAsync();
```

Will be roughly translated to the following SQL statement:
```sql
SELECT [b].[Id], [b].[AuthorId], [b].[Title], [a].[Id], [a0].[Id], [t].[BookId], [t].[CategoryId], [t].[Id], [t].[CategoryName], [a].[FirstName], [a].[LastName], [a0].[AuthorId], [a0].[Biography], [a0].[DateOfBirth], [a0].[Nationality], [a0].[PlaceOfBirth]
FROM [Books] AS [b]
INNER JOIN [Authors] AS [a] ON [b].[AuthorId] = [a].[Id]
LEFT JOIN [AuthorBiographies] AS [a0] ON [a].[Id] = [a0].[AuthorId]
LEFT JOIN (
    SELECT [b0].[BookId], [b0].[CategoryId], [c].[Id], [c].[CategoryName]
    FROM [BookCategories] AS [b0]
    INNER JOIN [Categories] AS [c] ON [b0].[CategoryId] = [c].[Id]
) AS [t] ON [b].[Id] = [t].[BookId]
ORDER BY [b].[Id], [a].[Id], [a0].[Id], [t].[BookId], [t].[CategoryId]
```

With `LoadAsync`:
```csharp
var query = Context.Books;
await query.Include(x => x.BookCategories)
    .ThenInclude(x => x.Category).LoadAsync();
await query.Include(x => x.Author).LoadAsync();
await query.Include(x => x.Author.Biography).LoadAsync();
return await query.ToListAsync();;
```

Which will be translated into:

```sql
SELECT [b].[Id], [b].[AuthorId], [b].[Title], [t].[BookId], [t].[CategoryId], [t].[Id], [t].[CategoryName]
FROM [Books] AS [b]
LEFT JOIN (
    SELECT [b0].[BookId], [b0].[CategoryId], [c].[Id], [c].[CategoryName]
    FROM [BookCategories] AS [b0]
    INNER JOIN [Categories] AS [c] ON [b0].[CategoryId] = [c].[Id]
) AS [t] ON [b].[Id] = [t].[BookId]
ORDER BY [b].[Id], [t].[BookId], [t].[CategoryId]


SELECT [b].[Id], [b].[AuthorId], [b].[Title], [a].[Id], [a].[FirstName], [a].[LastName]
FROM [Books] AS [b]
INNER JOIN [Authors] AS [a] ON [b].[AuthorId] = [a].[Id]

SELECT [b].[Id], [b].[AuthorId], [b].[Title], [a].[Id], [a].[FirstName], [a].[LastName], [a0].[Id], [a0].[AuthorId], [a0].[Biography], [a0].[DateOfBirth], [a0].[Nationality], [a0].[PlaceOfBirth]
FROM [Books] AS [b]
INNER JOIN [Authors] AS [a] ON [b].[AuthorId] = [a].[Id]
LEFT JOIN [AuthorBiographies] AS [a0] ON [a].[Id] = [a0].[AuthorId]

SELECT [b].[Id], [b].[AuthorId], [b].[Title]
FROM [Books] AS [b]
```