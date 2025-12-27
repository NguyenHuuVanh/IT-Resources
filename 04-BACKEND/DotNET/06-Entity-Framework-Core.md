# Entity Framework Core

## Mục lục

- [Giới thiệu EF Core](#giới-thiệu-ef-core)
- [Setup và Configuration](#setup-và-configuration)
- [DbContext](#dbcontext)
- [Entity Configuration](#entity-configuration)
- [Migrations](#migrations)
- [CRUD Operations](#crud-operations)
- [Querying Data](#querying-data)
- [Relationships](#relationships)
- [Transactions](#transactions)
- [Performance Optimization](#performance-optimization)

---

## Giới thiệu EF Core

Entity Framework Core là **ORM (Object-Relational Mapper)** cho .NET, cho phép làm việc với database bằng C# objects.

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Code                         │
│                   (C# Classes/LINQ)                         │
├─────────────────────────────────────────────────────────────┤
│                   Entity Framework Core                     │
│              (DbContext, DbSet, Change Tracking)            │
├─────────────────────────────────────────────────────────────┤
│                   Database Provider                         │
│        (SQL Server, PostgreSQL, MySQL, SQLite...)           │
├─────────────────────────────────────────────────────────────┤
│                      Database                               │
└─────────────────────────────────────────────────────────────┘
```

### Cài đặt

```bash
# Core package
dotnet add package Microsoft.EntityFrameworkCore

# Database providers
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Pomelo.EntityFrameworkCore.MySql
dotnet add package Microsoft.EntityFrameworkCore.Sqlite

# Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-ef
```

---

## Setup và Configuration

### DbContext

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }

    // DbSets - Tables
    public DbSet<Product> Products => Set<Product>();
    public DbSet<Category> Categories => Set<Category>();
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<OrderItem> OrderItems => Set<OrderItem>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Fluent API configuration
        modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());

        base.OnModelCreating(modelBuilder);
    }
}
```

### Register in DI

```csharp
// Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
{
    // SQL Server
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"));

    // PostgreSQL
    // options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection"));

    // MySQL
    // options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString));

    // SQLite
    // options.UseSqlite("Data Source=app.db");

    // Development options
    if (builder.Environment.IsDevelopment())
    {
        options.EnableSensitiveDataLogging();
        options.EnableDetailedErrors();
    }
});

// Connection string in appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;Trusted_Connection=true;TrustServerCertificate=true;"
  }
}
```

---

## Entity Configuration

### Data Annotations

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

[Table("Products")]
public class Product
{
    [Key]
    public int Id { get; set; }

    [Required]
    [StringLength(100)]
    public string Name { get; set; } = string.Empty;

    [Column(TypeName = "decimal(18,2)")]
    public decimal Price { get; set; }

    [StringLength(500)]
    public string? Description { get; set; }

    [Required]
    public int CategoryId { get; set; }

    [ForeignKey(nameof(CategoryId))]
    public Category Category { get; set; } = null!;

    [NotMapped]  // Không map vào database
    public decimal PriceWithTax => Price * 1.1m;

    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;

    [Timestamp]  // Concurrency token
    public byte[] RowVersion { get; set; } = null!;
}

public class Category
{
    public int Id { get; set; }

    [Required]
    [StringLength(50)]
    public string Name { get; set; } = string.Empty;

    public ICollection<Product> Products { get; set; } = new List<Product>();
}
```

### Fluent API (Recommended)

```csharp
// Separate configuration class
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.ToTable("Products");

        builder.HasKey(p => p.Id);

        builder.Property(p => p.Name)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(p => p.Price)
            .HasColumnType("decimal(18,2)")
            .IsRequired();

        builder.Property(p => p.Description)
            .HasMaxLength(500);

        builder.Property(p => p.CreatedAt)
            .HasDefaultValueSql("GETUTCDATE()");

        builder.Property(p => p.RowVersion)
            .IsRowVersion();

        // Relationship
        builder.HasOne(p => p.Category)
            .WithMany(c => c.Products)
            .HasForeignKey(p => p.CategoryId)
            .OnDelete(DeleteBehavior.Restrict);

        // Index
        builder.HasIndex(p => p.Name);
        builder.HasIndex(p => new { p.CategoryId, p.Name }).IsUnique();

        // Ignore property
        builder.Ignore(p => p.PriceWithTax);
    }
}

// Apply in DbContext
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfiguration(new ProductConfiguration());
    // hoặc apply tất cả từ assembly
    modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());
}
```

---

## Migrations

```bash
# Tạo migration
dotnet ef migrations add InitialCreate

# Tạo migration với output directory
dotnet ef migrations add InitialCreate -o Data/Migrations

# Apply migrations
dotnet ef database update

# Revert migration
dotnet ef database update PreviousMigrationName

# Remove last migration (chưa apply)
dotnet ef migrations remove

# Generate SQL script
dotnet ef migrations script

# Generate SQL script từ migration cụ thể
dotnet ef migrations script FromMigration ToMigration

# List migrations
dotnet ef migrations list
```

### Migration trong Code

```csharp
// Apply migrations at startup
using (var scope = app.Services.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    db.Database.Migrate();
}

// Seed data trong migration
public partial class SeedInitialData : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.InsertData(
            table: "Categories",
            columns: new[] { "Id", "Name" },
            values: new object[] { 1, "Electronics" }
        );
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DeleteData(
            table: "Categories",
            keyColumn: "Id",
            keyValue: 1
        );
    }
}

// Seed data trong OnModelCreating
modelBuilder.Entity<Category>().HasData(
    new Category { Id = 1, Name = "Electronics" },
    new Category { Id = 2, Name = "Clothing" }
);
```

---

## CRUD Operations

### Create

```csharp
public class ProductService
{
    private readonly AppDbContext _context;

    public ProductService(AppDbContext context)
    {
        _context = context;
    }

    // Add single entity
    public async Task<Product> CreateAsync(CreateProductDto dto)
    {
        var product = new Product
        {
            Name = dto.Name,
            Price = dto.Price,
            CategoryId = dto.CategoryId
        };

        _context.Products.Add(product);
        await _context.SaveChangesAsync();

        return product;
    }

    // Add multiple entities
    public async Task CreateManyAsync(IEnumerable<Product> products)
    {
        _context.Products.AddRange(products);
        await _context.SaveChangesAsync();
    }
}
```

### Read

```csharp
// Get by ID
public async Task<Product?> GetByIdAsync(int id)
{
    return await _context.Products.FindAsync(id);
}

// Get by ID with includes
public async Task<Product?> GetByIdWithCategoryAsync(int id)
{
    return await _context.Products
        .Include(p => p.Category)
        .FirstOrDefaultAsync(p => p.Id == id);
}

// Get all
public async Task<List<Product>> GetAllAsync()
{
    return await _context.Products.ToListAsync();
}

// Get with filter
public async Task<List<Product>> GetByCategoryAsync(int categoryId)
{
    return await _context.Products
        .Where(p => p.CategoryId == categoryId)
        .ToListAsync();
}
```

### Update

```csharp
// Update tracked entity
public async Task UpdateAsync(int id, UpdateProductDto dto)
{
    var product = await _context.Products.FindAsync(id);

    if (product == null)
        throw new NotFoundException($"Product {id} not found");

    product.Name = dto.Name;
    product.Price = dto.Price;
    product.CategoryId = dto.CategoryId;

    await _context.SaveChangesAsync();
}

// Update untracked entity
public async Task UpdateAsync(Product product)
{
    _context.Products.Update(product);
    await _context.SaveChangesAsync();
}

// Partial update
public async Task UpdatePriceAsync(int id, decimal newPrice)
{
    var product = await _context.Products.FindAsync(id);
    if (product != null)
    {
        product.Price = newPrice;
        await _context.SaveChangesAsync();
    }
}

// Bulk update (EF Core 7+)
public async Task UpdatePricesByCategoryAsync(int categoryId, decimal multiplier)
{
    await _context.Products
        .Where(p => p.CategoryId == categoryId)
        .ExecuteUpdateAsync(s => s
            .SetProperty(p => p.Price, p => p.Price * multiplier));
}
```

### Delete

```csharp
// Delete by entity
public async Task DeleteAsync(int id)
{
    var product = await _context.Products.FindAsync(id);

    if (product != null)
    {
        _context.Products.Remove(product);
        await _context.SaveChangesAsync();
    }
}

// Delete without loading
public async Task DeleteByIdAsync(int id)
{
    var product = new Product { Id = id };
    _context.Products.Remove(product);
    await _context.SaveChangesAsync();
}

// Bulk delete (EF Core 7+)
public async Task DeleteByCategoryAsync(int categoryId)
{
    await _context.Products
        .Where(p => p.CategoryId == categoryId)
        .ExecuteDeleteAsync();
}
```

---

## Querying Data

### LINQ Queries

```csharp
// Basic queries
var products = await _context.Products
    .Where(p => p.Price > 100)
    .OrderBy(p => p.Name)
    .ToListAsync();

// Select specific columns
var productNames = await _context.Products
    .Select(p => new { p.Id, p.Name, p.Price })
    .ToListAsync();

// Pagination
var pagedProducts = await _context.Products
    .OrderBy(p => p.Id)
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();

// Aggregations
var count = await _context.Products.CountAsync();
var avgPrice = await _context.Products.AverageAsync(p => p.Price);
var maxPrice = await _context.Products.MaxAsync(p => p.Price);
var totalValue = await _context.Products.SumAsync(p => p.Price);

// Any/All
var hasExpensive = await _context.Products.AnyAsync(p => p.Price > 1000);
var allInStock = await _context.Products.AllAsync(p => p.Stock > 0);

// First/Single
var first = await _context.Products.FirstOrDefaultAsync();
var single = await _context.Products.SingleOrDefaultAsync(p => p.Id == 1);

// Grouping
var byCategory = await _context.Products
    .GroupBy(p => p.CategoryId)
    .Select(g => new
    {
        CategoryId = g.Key,
        Count = g.Count(),
        AvgPrice = g.Average(p => p.Price)
    })
    .ToListAsync();
```

### Include (Eager Loading)

```csharp
// Single include
var products = await _context.Products
    .Include(p => p.Category)
    .ToListAsync();

// Multiple includes
var orders = await _context.Orders
    .Include(o => o.Customer)
    .Include(o => o.OrderItems)
    .ToListAsync();

// Nested include (ThenInclude)
var orders = await _context.Orders
    .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product)
            .ThenInclude(p => p.Category)
    .ToListAsync();

// Filtered include (EF Core 5+)
var categories = await _context.Categories
    .Include(c => c.Products.Where(p => p.Price > 100))
    .ToListAsync();

// AsSplitQuery - Tách thành nhiều queries
var orders = await _context.Orders
    .Include(o => o.OrderItems)
    .AsSplitQuery()
    .ToListAsync();
```

### Explicit Loading

```csharp
var product = await _context.Products.FindAsync(id);

// Load related data explicitly
await _context.Entry(product)
    .Reference(p => p.Category)
    .LoadAsync();

await _context.Entry(product)
    .Collection(p => p.Reviews)
    .LoadAsync();

// Query related data
var reviewCount = await _context.Entry(product)
    .Collection(p => p.Reviews)
    .Query()
    .CountAsync();
```

### Raw SQL

```csharp
// FromSqlRaw
var products = await _context.Products
    .FromSqlRaw("SELECT * FROM Products WHERE Price > {0}", minPrice)
    .ToListAsync();

// FromSqlInterpolated (safer)
var products = await _context.Products
    .FromSqlInterpolated($"SELECT * FROM Products WHERE Price > {minPrice}")
    .ToListAsync();

// Execute raw SQL
await _context.Database.ExecuteSqlRawAsync(
    "UPDATE Products SET Price = Price * 1.1 WHERE CategoryId = {0}", categoryId);

// Stored procedure
var products = await _context.Products
    .FromSqlRaw("EXEC GetProductsByCategory @CategoryId = {0}", categoryId)
    .ToListAsync();
```

### No Tracking

```csharp
// Single query
var products = await _context.Products
    .AsNoTracking()
    .ToListAsync();

// Global setting
builder.Services.AddDbContext<AppDbContext>(options =>
{
    options.UseSqlServer(connectionString)
           .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
});
```

---

## Relationships

### One-to-Many

```csharp
// Entities
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Product> Products { get; set; } = new List<Product>();
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int CategoryId { get; set; }
    public Category Category { get; set; }
}

// Configuration
builder.HasOne(p => p.Category)
    .WithMany(c => c.Products)
    .HasForeignKey(p => p.CategoryId)
    .OnDelete(DeleteBehavior.Cascade);
```

### Many-to-Many

```csharp
// Entities
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Tag> Tags { get; set; } = new List<Tag>();
}

public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Product> Products { get; set; } = new List<Product>();
}

// EF Core 5+ auto creates join table
// Configuration (optional)
builder.HasMany(p => p.Tags)
    .WithMany(t => t.Products)
    .UsingEntity(j => j.ToTable("ProductTags"));

// With explicit join entity
public class ProductTag
{
    public int ProductId { get; set; }
    public Product Product { get; set; }
    public int TagId { get; set; }
    public Tag Tag { get; set; }
    public DateTime AddedAt { get; set; }
}
```

### One-to-One

```csharp
public class User
{
    public int Id { get; set; }
    public string Username { get; set; }
    public UserProfile Profile { get; set; }
}

public class UserProfile
{
    public int Id { get; set; }
    public string Bio { get; set; }
    public int UserId { get; set; }
    public User User { get; set; }
}

// Configuration
builder.HasOne(u => u.Profile)
    .WithOne(p => p.User)
    .HasForeignKey<UserProfile>(p => p.UserId);
```

---

## Transactions

```csharp
// Implicit transaction (SaveChanges)
_context.Products.Add(product);
_context.Categories.Add(category);
await _context.SaveChangesAsync();  // Single transaction

// Explicit transaction
using var transaction = await _context.Database.BeginTransactionAsync();
try
{
    _context.Products.Add(product);
    await _context.SaveChangesAsync();

    _context.OrderItems.Add(orderItem);
    await _context.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}

// Execution strategy (for retries)
var strategy = _context.Database.CreateExecutionStrategy();
await strategy.ExecuteAsync(async () =>
{
    using var transaction = await _context.Database.BeginTransactionAsync();
    // ... operations
    await transaction.CommitAsync();
});
```

---

## Performance Optimization

### 1. Use AsNoTracking

```csharp
// Read-only queries
var products = await _context.Products
    .AsNoTracking()
    .ToListAsync();
```

### 2. Select Only Needed Columns

```csharp
// Instead of loading entire entity
var productNames = await _context.Products
    .Select(p => new { p.Id, p.Name })
    .ToListAsync();
```

### 3. Use Pagination

```csharp
var products = await _context.Products
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();
```

### 4. Avoid N+1 Problem

```csharp
// Bad - N+1 queries
var categories = await _context.Categories.ToListAsync();
foreach (var category in categories)
{
    var products = category.Products;  // Lazy loading = N queries
}

// Good - Single query with Include
var categories = await _context.Categories
    .Include(c => c.Products)
    .ToListAsync();
```

### 5. Use Compiled Queries

```csharp
private static readonly Func<AppDbContext, int, Task<Product?>> GetProductById =
    EF.CompileAsyncQuery((AppDbContext context, int id) =>
        context.Products.FirstOrDefault(p => p.Id == id));

// Usage
var product = await GetProductById(_context, id);
```

### 6. Bulk Operations

```csharp
// EF Core 7+ ExecuteUpdate/ExecuteDelete
await _context.Products
    .Where(p => p.CategoryId == categoryId)
    .ExecuteUpdateAsync(s => s.SetProperty(p => p.Price, p => p.Price * 1.1m));

await _context.Products
    .Where(p => p.IsDeleted)
    .ExecuteDeleteAsync();
```

### 7. Use Indexes

```csharp
builder.HasIndex(p => p.Name);
builder.HasIndex(p => new { p.CategoryId, p.Name });
builder.HasIndex(p => p.Email).IsUnique();
```

---

## Tóm tắt

| Feature              | Mô tả                             |
| -------------------- | --------------------------------- |
| DbContext            | Unit of Work, manages connections |
| DbSet<T>             | Repository for entity type        |
| Migrations           | Database schema versioning        |
| Fluent API           | Preferred configuration method    |
| Include              | Eager loading related data        |
| AsNoTracking         | Better performance for read-only  |
| ExecuteUpdate/Delete | Bulk operations (EF Core 7+)      |
| Transactions         | Explicit transaction control      |
