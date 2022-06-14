## Define maximum length for strings
When creating a SQL database via code first it is important to tell EF Core how long a string can be otherwise it will always be translated to `NVARCHAR(MAX)`. This has [performance implications](https://hungdoan.com/2017/04/13/nvarcharn-vs-nvarcharmax-performance-in-ms-sql-server/) as well as other problems like not being able to create an index on that column. Also a rogue application could flood the database.

Having this model:
```csharp
public class BlogPost
{
  public int Id { get; private set; }
  public string Title { get; private set; }
  public string Content { get; private set; }
}
```

❌ **Bad** Not defining the maximum length of a string will lead to `NVARCHAR(max)`.
```csharp
public class BlogPostConfiguration : IEntityTypeConfiguration<BlogPost>
{
    public void Configure(EntityTypeBuilder<BlogPost> builder)
    {
        builder.HasKey(c => c.Id);
        builder.Property(c => c.Id).ValueGeneratedOnAdd();
    }
}
```

Will lead to generation of this SQL table:  
```sql
CREATE TABLE BlogPosts
(
  [Id] [int] NOT NULL,
  [Title] [NVARCHAR](MAX) NULL,
  [Content] [NVARCHAR](MAX) NULL
)
```

✅ **Good** Defining the maximum length will reflect also in the database table.
```csharp
public class BlogPostConfiguration : IEntityTypeConfiguration<BlogPost>
{
    public void Configure(EntityTypeBuilder<BlogPost> builder)
    {
        builder.HasKey(c => c.Id);
        builder.Property(c => c.Id).ValueGeneratedOnAdd();

        // Set title max length explicitly to 256
        builder.Property(c => c.Title).HasMaxLength(256);
    }
}
```

Will lead to generation of this SQL table:
```sql
CREATE TABLE BlogPosts
(
  [Id] [int] NOT NULL,
  [Title] [NVARCHAR](256) NULL, -- Now it only can hold 256 characters
  [Content] [NVARCHAR](MAX) NULL
)
```