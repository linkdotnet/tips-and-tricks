## Don't track readonly entities
When loading models from the database Entity Framework will create proxies of your object for change detection. If you have load objects, which will never get updated this is an unnecessary overhead in terms of performance but also allocations as those proxies have their own memory footprint.

❌ **Bad** Change detection proxies are created even though it is only meant for reading.
```csharp
return await blogPosts.Where(b => b.IsPublished)
                      .Include(b => b.Tags)
                      .ToListAsync();
```

✅ **Good** Explicitly say that the entities are not tracked by EF.
```csharp
return await blogPosts.Where(b => b.IsPublished)
                      .Include(b => b.Tags)
                      .AsNoTracking()
                      .ToListAsync();
```

### Benchmark
A detailed setup and benchmark (which is referred below) can be found on the official https://docs.microsoft.com/en-us/ef/core/performance/efficient-querying#tracking-no-tracking-and-identity-resolution.

```csharp
|       Method | NumBlogs | NumPostsPerBlog |       Mean |    Error |   StdDev |     Median | Ratio | RatioSD |   Gen 0 |   Gen 1 | Gen 2 | Allocated |
|------------- |--------- |---------------- |-----------:|---------:|---------:|-----------:|------:|--------:|--------:|--------:|------:|----------:|
|   AsTracking |       10 |              20 | 1,414.7 us | 27.20 us | 45.44 us | 1,405.5 us |  1.00 |    0.00 | 60.5469 | 13.6719 |     - | 380.11 KB |
| AsNoTracking |       10 |              20 |   993.3 us | 24.04 us | 65.40 us |   966.2 us |  0.71 |    0.05 | 37.1094 |  6.8359 |     - | 232.89 KB |

```