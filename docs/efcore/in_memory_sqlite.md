## In-Memory database with SQLite

SQLite can be used as an in-memory database for your code. This brings big advantage in testing. The database is transient, that means as soon as the connection gets closed the memory is freed. One downside is that the in-memory database is not thread-safe by default. This is achieved with the special `:memory:` data source.
The advantage over the In-Memory database package provided via `Microsoft.EntityFrameworkCore.InMemory` that the SQLite version behaves closer to a real rational database. Also [Microsoft disencourages](https://docs.microsoft.com/en-us/ef/core/testing/testing-without-the-database#in-memory-provider) the use of the `InMemory` provider.

```csharp
var connection = new SqliteConnection("DataSource=:memory:");
connection.Open();

services.AddDbContext<MyDbContext>(options =>
{
options.UseSqlite(connection);
});
```

To make it work with multiple connections at a time, we can utilize the `` identifier for the data source. More information can be found on the [official website](https://www.sqlite.org/inmemorydb.html#sharedmemdb).

```csharp
var connection = new SqliteConnection("DataSource=myshareddb;mode=memory;cache=shared");
connection.Open();

services.AddDbContext<MyDbContext>(options =>
{
options.UseSqlite(connection);
});
```

The database gets cleaned up when there is no active connection anymore.

> 💡 Info: You have to install the [`Microsoft.EntityFrameworkCore.Sqlite`](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/) package to use the `UseSqlite` method.