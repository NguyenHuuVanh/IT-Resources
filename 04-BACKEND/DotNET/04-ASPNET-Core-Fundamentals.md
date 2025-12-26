# ASP.NET Core Fundamentals

## Mục lục

- [Giới thiệu ASP.NET Core](#giới-thiệu-aspnet-core)
- [Project Structure](#project-structure)
- [Program.cs và Startup](#programcs-và-startup)
- [Middleware Pipeline](#middleware-pipeline)
- [Dependency Injection](#dependency-injection)
- [Configuration](#configuration)
- [Logging](#logging)
- [Environments](#environments)

---

## Giới thiệu ASP.NET Core

ASP.NET Core là framework **cross-platform, high-performance** để xây dựng web applications và APIs.

### Đặc điểm

```
┌─────────────────────────────────────────────────────────────┐
│                    ASP.NET Core                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐           │
│  │ Web API │ │   MVC   │ │ Razor   │ │ Blazor  │           │
│  │  REST   │ │  Pages  │ │  Pages  │ │   SPA   │           │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘           │
├─────────────────────────────────────────────────────────────┤
│  Middleware Pipeline                                        │
├─────────────────────────────────────────────────────────────┤
│  Dependency Injection | Configuration | Logging             │
├─────────────────────────────────────────────────────────────┤
│  Kestrel Web Server                                         │
├─────────────────────────────────────────────────────────────┤
│  .NET Runtime (Cross-platform)                              │
└─────────────────────────────────────────────────────────────┘
```

### Tạo Project

```bash
# Web API
dotnet new webapi -n MyApi

# MVC
dotnet new mvc -n MyMvc

# Minimal API
dotnet new web -n MyMinimalApi

# Blazor Server
dotnet new blazorserver -n MyBlazor

# Chạy
cd MyApi
dotnet run
```

---

## Project Structure

```
MyApi/
├── Controllers/           # API Controllers
│   └── WeatherController.cs
├── Models/               # Data models
│   └── WeatherForecast.cs
├── Services/             # Business logic
│   └── WeatherService.cs
├── Data/                 # Database context
│   └── AppDbContext.cs
├── Middleware/           # Custom middleware
├── Properties/
│   └── launchSettings.json
├── appsettings.json      # Configuration
├── appsettings.Development.json
├── Program.cs            # Entry point
└── MyApi.csproj          # Project file
```

### Project File (.csproj)

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
    <PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
  </ItemGroup>

</Project>
```

---

## Program.cs và Startup

### Minimal Hosting Model (.NET 6+)

```csharp
// Program.cs - Modern style
var builder = WebApplication.CreateBuilder(args);

// ============ SERVICES CONFIGURATION ============
// Add services to the container

// Controllers
builder.Services.AddControllers();

// Swagger
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Database
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Custom services
builder.Services.AddScoped<IWeatherService, WeatherService>();
builder.Services.AddSingleton<ICacheService, CacheService>();

// CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

// Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* config */ });

builder.Services.AddAuthorization();

var app = builder.Build();

// ============ MIDDLEWARE PIPELINE ============
// Configure the HTTP request pipeline

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseCors("AllowAll");

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Traditional Startup Class (Optional)

```csharp
// Startup.cs - Traditional style (vẫn hỗ trợ)
public class Startup
{
    public IConfiguration Configuration { get; }

    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    // Configure services (DI)
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        services.AddScoped<IMyService, MyService>();
    }

    // Configure middleware pipeline
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();
        app.UseAuthorization();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}

// Program.cs
var builder = WebApplication.CreateBuilder(args);
var startup = new Startup(builder.Configuration);
startup.ConfigureServices(builder.Services);

var app = builder.Build();
startup.Configure(app, app.Environment);
app.Run();
```

---

## Middleware Pipeline

Middleware xử lý HTTP requests và responses theo thứ tự pipeline.

```
Request  ──▶ Middleware 1 ──▶ Middleware 2 ──▶ Middleware 3 ──▶ Endpoint
                │                  │                  │              │
Response ◀── Middleware 1 ◀── Middleware 2 ◀── Middleware 3 ◀────────┘
```

### Built-in Middleware

```csharp
var app = builder.Build();

// Exception handling (đầu tiên)
app.UseExceptionHandler("/error");

// HSTS
app.UseHsts();

// HTTPS redirection
app.UseHttpsRedirection();

// Static files
app.UseStaticFiles();

// Routing
app.UseRouting();

// CORS (sau Routing, trước Auth)
app.UseCors();

// Authentication
app.UseAuthentication();

// Authorization
app.UseAuthorization();

// Response caching
app.UseResponseCaching();

// Endpoints (cuối cùng)
app.MapControllers();
```

### Custom Middleware

```csharp
// Inline middleware
app.Use(async (context, next) =>
{
    // Before next middleware
    Console.WriteLine($"Request: {context.Request.Path}");
    var stopwatch = Stopwatch.StartNew();

    await next();  // Call next middleware

    // After next middleware
    stopwatch.Stop();
    Console.WriteLine($"Response time: {stopwatch.ElapsedMilliseconds}ms");
});

// Terminal middleware (không gọi next)
app.Map("/health", app => app.Run(async context =>
{
    await context.Response.WriteAsync("Healthy");
}));

// Middleware class
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("Request: {Method} {Path}",
            context.Request.Method, context.Request.Path);

        var stopwatch = Stopwatch.StartNew();

        await _next(context);

        stopwatch.Stop();
        _logger.LogInformation("Response: {StatusCode} in {ElapsedMs}ms",
            context.Response.StatusCode, stopwatch.ElapsedMilliseconds);
    }
}

// Extension method
public static class MiddlewareExtensions
{
    public static IApplicationBuilder UseRequestLogging(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestLoggingMiddleware>();
    }
}

// Sử dụng
app.UseRequestLogging();
```

---

## Dependency Injection

ASP.NET Core có built-in DI container.

### Service Lifetimes

```csharp
// Transient - Tạo mới mỗi lần request service
builder.Services.AddTransient<IEmailService, EmailService>();

// Scoped - Tạo mới mỗi HTTP request
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

// Singleton - Tạo 1 lần, dùng suốt application lifetime
builder.Services.AddSingleton<ICacheService, CacheService>();
builder.Services.AddSingleton<IConfiguration>(builder.Configuration);
```

```
┌─────────────────────────────────────────────────────────────┐
│                    Service Lifetimes                        │
├─────────────────────────────────────────────────────────────┤
│  Transient:  Request 1 → Instance A                         │
│              Request 1 → Instance B (new)                   │
│              Request 2 → Instance C (new)                   │
├─────────────────────────────────────────────────────────────┤
│  Scoped:     Request 1 → Instance A                         │
│              Request 1 → Instance A (same)                  │
│              Request 2 → Instance B (new request)           │
├─────────────────────────────────────────────────────────────┤
│  Singleton:  Request 1 → Instance A                         │
│              Request 2 → Instance A (same)                  │
│              App restart → Instance B (new)                 │
└─────────────────────────────────────────────────────────────┘
```

### Registration Patterns

```csharp
// Interface → Implementation
builder.Services.AddScoped<IProductService, ProductService>();

// Self registration
builder.Services.AddScoped<ProductService>();

// Factory registration
builder.Services.AddScoped<IDbConnection>(sp =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    return new SqlConnection(config.GetConnectionString("Default"));
});

// Multiple implementations
builder.Services.AddScoped<INotificationService, EmailNotificationService>();
builder.Services.AddScoped<INotificationService, SmsNotificationService>();
// Inject IEnumerable<INotificationService> để lấy tất cả

// Keyed services (.NET 8+)
builder.Services.AddKeyedScoped<ICache, RedisCache>("redis");
builder.Services.AddKeyedScoped<ICache, MemoryCache>("memory");

// Generic registration
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));

// Options pattern
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));
```

### Injecting Services

```csharp
// Constructor injection (recommended)
public class ProductController : ControllerBase
{
    private readonly IProductService _productService;
    private readonly ILogger<ProductController> _logger;

    public ProductController(IProductService productService, ILogger<ProductController> logger)
    {
        _productService = productService;
        _logger = logger;
    }
}

// Method injection với [FromServices]
[HttpGet]
public IActionResult Get([FromServices] IProductService service)
{
    return Ok(service.GetAll());
}

// Primary constructor (C# 12)
public class ProductController(IProductService productService, ILogger<ProductController> logger)
    : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok(productService.GetAll());
}

// Inject trong Minimal API
app.MapGet("/products", (IProductService service) => service.GetAll());

// Service Locator (avoid if possible)
public class MyService
{
    private readonly IServiceProvider _serviceProvider;

    public void DoWork()
    {
        using var scope = _serviceProvider.CreateScope();
        var service = scope.ServiceProvider.GetRequiredService<IScopedService>();
    }
}
```

---

## Configuration

### appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;Trusted_Connection=true;"
  },
  "JwtSettings": {
    "Secret": "your-secret-key-here",
    "Issuer": "MyApp",
    "Audience": "MyApp",
    "ExpirationInMinutes": 60
  },
  "EmailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "Port": 587,
    "Username": "user@gmail.com"
  }
}
```

### Đọc Configuration

```csharp
// Trực tiếp từ IConfiguration
public class MyService
{
    private readonly IConfiguration _config;

    public MyService(IConfiguration config)
    {
        _config = config;
    }

    public void DoWork()
    {
        // Đọc giá trị đơn
        string connStr = _config.GetConnectionString("DefaultConnection");
        string secret = _config["JwtSettings:Secret"];
        int port = _config.GetValue<int>("EmailSettings:Port");

        // Đọc section
        IConfigurationSection section = _config.GetSection("JwtSettings");
    }
}

// Options Pattern (recommended)
public class JwtSettings
{
    public string Secret { get; set; } = string.Empty;
    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;
    public int ExpirationInMinutes { get; set; }
}

// Register
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection("JwtSettings"));

// Inject
public class AuthService
{
    private readonly JwtSettings _jwtSettings;

    public AuthService(IOptions<JwtSettings> options)
    {
        _jwtSettings = options.Value;
    }
}

// IOptionsSnapshot - Reload khi config thay đổi (scoped)
public AuthService(IOptionsSnapshot<JwtSettings> options)

// IOptionsMonitor - Reload + notification (singleton-safe)
public AuthService(IOptionsMonitor<JwtSettings> options)
{
    options.OnChange(settings => Console.WriteLine("Config changed"));
}
```

### Configuration Sources

```csharp
var builder = WebApplication.CreateBuilder(args);

// Thứ tự ưu tiên (sau override trước)
// 1. appsettings.json
// 2. appsettings.{Environment}.json
// 3. User secrets (Development)
// 4. Environment variables
// 5. Command line arguments

// Custom configuration
builder.Configuration
    .AddJsonFile("custom.json", optional: true)
    .AddEnvironmentVariables("MYAPP_")
    .AddCommandLine(args);

// User Secrets (Development only)
// dotnet user-secrets init
// dotnet user-secrets set "JwtSettings:Secret" "my-secret"
```

---

## Logging

### Built-in Logging

```csharp
public class ProductController : ControllerBase
{
    private readonly ILogger<ProductController> _logger;

    public ProductController(ILogger<ProductController> logger)
    {
        _logger = logger;
    }

    [HttpGet("{id}")]
    public IActionResult Get(int id)
    {
        _logger.LogTrace("Trace message");
        _logger.LogDebug("Debug message");
        _logger.LogInformation("Getting product {ProductId}", id);
        _logger.LogWarning("Product {ProductId} not found", id);
        _logger.LogError("Error getting product {ProductId}", id);
        _logger.LogCritical("Critical error!");

        // Structured logging
        _logger.LogInformation("User {UserId} accessed product {ProductId} at {Time}",
            userId, id, DateTime.UtcNow);

        // With exception
        try { }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error processing request");
        }

        return Ok();
    }
}
```

### Log Levels

```
┌─────────────────────────────────────────────────────────────┐
│  Level        │ Value │ Usage                               │
├───────────────┼───────┼─────────────────────────────────────┤
│ Trace         │   0   │ Detailed debugging                  │
│ Debug         │   1   │ Development debugging               │
│ Information   │   2   │ General flow                        │
│ Warning       │   3   │ Abnormal/unexpected                 │
│ Error         │   4   │ Failures, exceptions                │
│ Critical      │   5   │ System failures                     │
│ None          │   6   │ Disable logging                     │
└─────────────────────────────────────────────────────────────┘
```

### Serilog Integration

```csharp
// Install: Serilog.AspNetCore

// Program.cs
using Serilog;

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();

try
{
    Log.Information("Starting application");

    var builder = WebApplication.CreateBuilder(args);
    builder.Host.UseSerilog();

    // ... rest of setup

    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

---

## Environments

### Environment Detection

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Check environment
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
}
else if (app.Environment.IsStaging())
{
    // Staging specific
}
else if (app.Environment.IsProduction())
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

// Custom environment
if (app.Environment.IsEnvironment("Testing"))
{
    // Testing specific
}

// Trong service
public class MyService
{
    private readonly IWebHostEnvironment _env;

    public MyService(IWebHostEnvironment env)
    {
        _env = env;

        if (_env.IsDevelopment())
        {
            // Dev only logic
        }
    }
}
```

### Setting Environment

```bash
# Windows CMD
set ASPNETCORE_ENVIRONMENT=Development

# Windows PowerShell
$env:ASPNETCORE_ENVIRONMENT = "Development"

# Linux/macOS
export ASPNETCORE_ENVIRONMENT=Development

# launchSettings.json
{
  "profiles": {
    "Development": {
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### Environment-specific Configuration

```
appsettings.json                    # Base config
appsettings.Development.json        # Dev overrides
appsettings.Staging.json            # Staging overrides
appsettings.Production.json         # Production overrides
```

```json
// appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb_Dev;..."
  }
}
```
