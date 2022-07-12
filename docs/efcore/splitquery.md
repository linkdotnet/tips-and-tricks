## Split multiple queries - `SplitQuery`

The basic idea is to avoid "cartesian explosion". A cartesian explosion is when performing a JOIN on the one-to-many relationship then the rows of the one-side are being replicated N times (N = amount of rows on the many side).

With SplitQuery instead of having 1 query, you will now have 2 queries where first the "one" side is loaded and in a separate query the "many" part is loaded. Where the SplitQuery can bring improvement it also has some major drawbacks.

1. You go two times to your database instead of once.
2. From the database point of view these are two separate queries. So no guarantee of data consistency. There could be race conditions interfering with your result set.


❌ **Bad** Every include is resolved by a `LEFT JOIN` leading to duplicated entries.
```csharp
var blogPosts = await DbContext.Blog
 .Include(b => b.Posts)
 .Include(b => b.Tags)
 .ToListAsync();
```

✅ **Good** Get all related children by a separate query which gets resolved by an `INNER JOIN`.
```csharp
var blogPosts = await DbContext.Blog
 .Include(b => b.Posts)
 .Include(b => b.Tags)
 .AsSplitQuery();
 .ToListAsync();
```

> 💡 Info: There are database which support multiple result sets in one query (for example SQL Server with **M** ultiple **A** ctive **R** esult **S** et). Here the performance is even better. For more information checkout the official Microsoft page about [`SplitQuery`](https://docs.microsoft.com/en-us/ef/core/querying/single-split-queries).
