## Enable Detailed logging in debug builds
You can enable detailed logging in debug builds by adding the `EnableDetailedErrors` method to the `DbContextOptionsBuilder` in the `ConfigureServices` method of the `Startup` class.

```csharp
services.AddDbContext<AppDbContext>(options =>
{
    options.UseSqlServer(connectionString)
#if DEBUG
        .EnableDetailedErrors()
#endif
        ;
});

```

This can give you vital insights, for example if you misconfigured some mappings/configurations.

!!! warning
    This is only recommended for development and debugging purposes. It is not recommended to use this in production.