## Retry on failure
Entity Framework Core supports automatic retry on failure when saving changes to the database. This can be useful when working with a database that is subject to transient failures, such as a database on Azure SQL Database. The examples uses SQL Server but other providers, like PostgreSQL, also support this feature.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<MyDbContext>(options =>
        options.UseSqlServer("your_connection_string",
            sqlServerOptions =>
            {
                sqlServerOptions.EnableRetryOnFailure(
                    maxRetryCount: 3,
                    maxRetryDelay: TimeSpan.FromSeconds(5),
                    errorNumbersToAdd: null);
            }));
}
```