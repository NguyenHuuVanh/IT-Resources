# ASP.NET Core Web API

## Mục lục

- [Controllers](#controllers)
- [Routing](#routing)
- [Model Binding](#model-binding)
- [Validation](#validation)
- [Action Results](#action-results)
- [Filters](#filters)
- [Minimal APIs](#minimal-apis)
- [API Versioning](#api-versioning)
- [Swagger/OpenAPI](#swaggeropenapi)
- [Error Handling](#error-handling)
- [CORS](#cors)

---

## Controllers

### Basic Controller

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    // GET api/products
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Product>>> GetAll()
    {
        var products = await _productService.GetAllAsync();
        return Ok(products);
    }

    // GET api/products/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetById(int id)
    {
        var product = await _productService.GetByIdAsync(id);

        if (product == null)
            return NotFound();

        return Ok(product);
    }

    // POST api/products
    [HttpPost]
    public async Task<ActionResult<Product>> Create([FromBody] CreateProductDto dto)
    {
        var product = await _productService.CreateAsync(dto);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }

    // PUT api/products/5
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, [FromBody] UpdateProductDto dto)
    {
        if (id != dto.Id)
            return BadRequest();

        await _productService.UpdateAsync(dto);
        return NoContent();
    }

    // DELETE api/products/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        await _productService.DeleteAsync(id);
        return NoContent();
    }
}
```

### [ApiController] Attribute

```csharp
[ApiController]  // Enables:
// 1. Automatic model validation (returns 400 if invalid)
// 2. Binding source inference ([FromBody], [FromRoute], etc.)
// 3. Problem details for error responses
// 4. Multipart/form-data inference
```

---

## Routing

### Attribute Routing

```csharp
[Route("api/[controller]")]  // api/products
public class ProductsController : ControllerBase
{
    [HttpGet]                           // GET api/products
    [HttpGet("all")]                    // GET api/products/all
    [HttpGet("{id}")]                   // GET api/products/5
    [HttpGet("{id:int}")]               // GET api/products/5 (int only)
    [HttpGet("category/{category}")]    // GET api/products/category/electronics
    [HttpGet("search")]                 // GET api/products/search?name=laptop

    // Route constraints
    [HttpGet("{id:int:min(1)}")]        // id >= 1
    [HttpGet("{name:alpha}")]           // alphabetic only
    [HttpGet("{date:datetime}")]        // valid datetime
    [HttpGet("{slug:regex(^[a-z]+$)}")] // regex pattern

    // Multiple routes
    [HttpGet]
    [Route("~/api/all-products")]       // Override: api/all-products
    public IActionResult GetAll() { }
}

// Route constraints
// {id:int}          - Integer
// {id:bool}         - Boolean
// {id:datetime}     - DateTime
// {id:decimal}      - Decimal
// {id:double}       - Double
// {id:float}        - Float
// {id:guid}         - Guid
// {id:long}         - Long
// {id:minlength(4)} - Min string length
// {id:maxlength(8)} - Max string length
// {id:length(4,8)}  - String length range
// {id:min(1)}       - Min value
// {id:max(100)}     - Max value
// {id:range(1,100)} - Value range
// {id:alpha}        - Alphabetic
// {id:regex(...)}   - Regex pattern
// {id:required}     - Required
```

### Route Parameters

```csharp
// Route parameter
[HttpGet("{id}")]
public IActionResult Get(int id) { }

// Optional parameter
[HttpGet("{id?}")]
public IActionResult Get(int? id) { }

// Default value
[HttpGet("{page=1}")]
public IActionResult Get(int page) { }

// Catch-all parameter
[HttpGet("{**path}")]
public IActionResult Get(string path) { }
// /api/files/folder/subfolder/file.txt → path = "folder/subfolder/file.txt"
```

---

## Model Binding

### Binding Sources

```csharp
public class ProductsController : ControllerBase
{
    // [FromRoute] - URL path
    [HttpGet("{id}")]
    public IActionResult Get([FromRoute] int id) { }

    // [FromQuery] - Query string
    [HttpGet]
    public IActionResult Search(
        [FromQuery] string? name,
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10) { }
    // GET api/products?name=laptop&page=2&pageSize=20

    // [FromBody] - Request body (JSON)
    [HttpPost]
    public IActionResult Create([FromBody] CreateProductDto dto) { }

    // [FromForm] - Form data
    [HttpPost("upload")]
    public IActionResult Upload([FromForm] IFormFile file) { }

    // [FromHeader] - HTTP header
    [HttpGet]
    public IActionResult Get([FromHeader(Name = "X-Custom-Header")] string? customHeader) { }

    // [FromServices] - DI container
    [HttpGet]
    public IActionResult Get([FromServices] IProductService service) { }

    // Complex binding
    [HttpGet]
    public IActionResult Search([FromQuery] ProductSearchParams searchParams) { }
}

public class ProductSearchParams
{
    public string? Name { get; set; }
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
    public string? Category { get; set; }
    public int Page { get; set; } = 1;
    public int PageSize { get; set; } = 10;
    public string SortBy { get; set; } = "Name";
    public bool Descending { get; set; } = false;
}
```

### Custom Model Binder

```csharp
public class CommaSeparatedArrayBinder : IModelBinder
{
    public Task BindModelAsync(ModelBindingContext bindingContext)
    {
        var value = bindingContext.ValueProvider.GetValue(bindingContext.ModelName);

        if (value == ValueProviderResult.None)
            return Task.CompletedTask;

        var stringValue = value.FirstValue;
        if (string.IsNullOrEmpty(stringValue))
            return Task.CompletedTask;

        var result = stringValue.Split(',').Select(int.Parse).ToArray();
        bindingContext.Result = ModelBindingResult.Success(result);

        return Task.CompletedTask;
    }
}

// Sử dụng
[HttpGet]
public IActionResult Get(
    [ModelBinder(typeof(CommaSeparatedArrayBinder))] int[] ids)
{
    // GET api/products?ids=1,2,3,4,5
    return Ok(ids);
}
```

---

## Validation

### Data Annotations

```csharp
public class CreateProductDto
{
    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 3, ErrorMessage = "Name must be 3-100 characters")]
    public string Name { get; set; } = string.Empty;

    [Required]
    [Range(0.01, 999999.99, ErrorMessage = "Price must be between 0.01 and 999999.99")]
    public decimal Price { get; set; }

    [StringLength(500)]
    public string? Description { get; set; }

    [Required]
    [RegularExpression(@"^[A-Z]{2,5}$", ErrorMessage = "Invalid category code")]
    public string CategoryCode { get; set; } = string.Empty;

    [Url]
    public string? ImageUrl { get; set; }

    [EmailAddress]
    public string? ContactEmail { get; set; }

    [Phone]
    public string? ContactPhone { get; set; }

    [CreditCard]
    public string? PaymentCard { get; set; }

    [Compare(nameof(Password))]
    public string ConfirmPassword { get; set; }
}
```

### FluentValidation

```csharp
// Install: FluentValidation.AspNetCore

public class CreateProductDtoValidator : AbstractValidator<CreateProductDto>
{
    public CreateProductDtoValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Name is required")
            .Length(3, 100).WithMessage("Name must be 3-100 characters");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be positive")
            .LessThan(1000000);

        RuleFor(x => x.CategoryCode)
            .NotEmpty()
            .Matches(@"^[A-Z]{2,5}$").WithMessage("Invalid category code");

        RuleFor(x => x.Description)
            .MaximumLength(500)
            .When(x => !string.IsNullOrEmpty(x.Description));

        RuleFor(x => x.ImageUrl)
            .Must(BeAValidUrl).When(x => !string.IsNullOrEmpty(x.ImageUrl))
            .WithMessage("Invalid URL format");
    }

    private bool BeAValidUrl(string? url)
    {
        return Uri.TryCreate(url, UriKind.Absolute, out _);
    }
}

// Register
builder.Services.AddValidatorsFromAssemblyContaining<CreateProductDtoValidator>();
```

### Manual Validation

```csharp
[HttpPost]
public IActionResult Create([FromBody] CreateProductDto dto)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    // Custom validation
    if (dto.Price < 0)
    {
        ModelState.AddModelError(nameof(dto.Price), "Price cannot be negative");
        return BadRequest(ModelState);
    }

    // Process...
    return Ok();
}
```

---

## Action Results

### Common Results

```csharp
public class ProductsController : ControllerBase
{
    // 200 OK
    [HttpGet]
    public IActionResult GetAll()
    {
        return Ok(products);
    }

    // 201 Created
    [HttpPost]
    public IActionResult Create(Product product)
    {
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
        // hoặc
        return Created($"/api/products/{product.Id}", product);
    }

    // 204 No Content
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        return NoContent();
    }

    // 400 Bad Request
    [HttpPost]
    public IActionResult Create(Product product)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        return BadRequest("Invalid data");
    }

    // 401 Unauthorized
    public IActionResult Secure()
    {
        return Unauthorized();
    }

    // 403 Forbidden
    public IActionResult Forbidden()
    {
        return Forbid();
    }

    // 404 Not Found
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _service.GetById(id);
        if (product == null)
            return NotFound();
        return Ok(product);
    }

    // 409 Conflict
    public IActionResult Create(Product product)
    {
        if (_service.Exists(product.Id))
            return Conflict("Product already exists");
        return Ok();
    }

    // 500 Internal Server Error
    public IActionResult Error()
    {
        return StatusCode(500, "Internal server error");
    }

    // Custom status code
    public IActionResult Custom()
    {
        return StatusCode(418, "I'm a teapot");
    }

    // File result
    [HttpGet("download/{id}")]
    public IActionResult Download(int id)
    {
        byte[] fileBytes = _service.GetFileBytes(id);
        return File(fileBytes, "application/pdf", "document.pdf");
    }

    // Redirect
    public IActionResult Redirect()
    {
        return RedirectToAction(nameof(GetAll));
        return Redirect("https://example.com");
    }
}
```

### Typed Results

```csharp
// ActionResult<T> - Strongly typed
[HttpGet("{id}")]
public async Task<ActionResult<Product>> GetById(int id)
{
    var product = await _service.GetByIdAsync(id);

    if (product == null)
        return NotFound();  // Implicit conversion

    return product;  // Implicit conversion to Ok(product)
}

// IActionResult vs ActionResult<T>
// IActionResult - Flexible, any result type
// ActionResult<T> - Type-safe, better for Swagger documentation
```

---

## Filters

Filters cho phép chạy code trước/sau action execution.

```
┌─────────────────────────────────────────────────────────────┐
│  Authorization Filters                                      │
├─────────────────────────────────────────────────────────────┤
│  Resource Filters (before model binding)                    │
├─────────────────────────────────────────────────────────────┤
│  Action Filters (before/after action)                       │
├─────────────────────────────────────────────────────────────┤
│  Exception Filters                                          │
├─────────────────────────────────────────────────────────────┤
│  Result Filters (before/after result)                       │
└─────────────────────────────────────────────────────────────┘
```

### Action Filter

```csharp
public class LogActionFilter : IActionFilter
{
    private readonly ILogger<LogActionFilter> _logger;

    public LogActionFilter(ILogger<LogActionFilter> logger)
    {
        _logger = logger;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        _logger.LogInformation("Executing action {Action}",
            context.ActionDescriptor.DisplayName);
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        _logger.LogInformation("Executed action {Action}",
            context.ActionDescriptor.DisplayName);
    }
}

// Async version
public class AsyncLogActionFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        // Before
        var stopwatch = Stopwatch.StartNew();

        var result = await next();  // Execute action

        // After
        stopwatch.Stop();
        Console.WriteLine($"Action took {stopwatch.ElapsedMilliseconds}ms");
    }
}

// Attribute-based filter
public class ValidateModelAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
        {
            context.Result = new BadRequestObjectResult(context.ModelState);
        }
    }
}

// Sử dụng
[ValidateModel]
[HttpPost]
public IActionResult Create(Product product) { }

// Global filter
builder.Services.AddControllers(options =>
{
    options.Filters.Add<LogActionFilter>();
});
```

### Exception Filter

```csharp
public class GlobalExceptionFilter : IExceptionFilter
{
    private readonly ILogger<GlobalExceptionFilter> _logger;

    public GlobalExceptionFilter(ILogger<GlobalExceptionFilter> logger)
    {
        _logger = logger;
    }

    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception, "Unhandled exception");

        var response = new
        {
            Message = "An error occurred",
            Detail = context.Exception.Message
        };

        context.Result = new ObjectResult(response)
        {
            StatusCode = 500
        };

        context.ExceptionHandled = true;
    }
}
```

---

## Minimal APIs

.NET 6+ hỗ trợ Minimal APIs - cách viết API ngắn gọn hơn.

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddScoped<IProductService, ProductService>();

var app = builder.Build();

// Basic endpoints
app.MapGet("/", () => "Hello World!");

app.MapGet("/products", async (IProductService service) =>
{
    return await service.GetAllAsync();
});

app.MapGet("/products/{id}", async (int id, IProductService service) =>
{
    var product = await service.GetByIdAsync(id);
    return product is not null ? Results.Ok(product) : Results.NotFound();
});

app.MapPost("/products", async (CreateProductDto dto, IProductService service) =>
{
    var product = await service.CreateAsync(dto);
    return Results.Created($"/products/{product.Id}", product);
});

app.MapPut("/products/{id}", async (int id, UpdateProductDto dto, IProductService service) =>
{
    await service.UpdateAsync(id, dto);
    return Results.NoContent();
});

app.MapDelete("/products/{id}", async (int id, IProductService service) =>
{
    await service.DeleteAsync(id);
    return Results.NoContent();
});

// Route groups
var productsGroup = app.MapGroup("/api/products")
    .WithTags("Products");

productsGroup.MapGet("/", GetAllProducts);
productsGroup.MapGet("/{id}", GetProductById);
productsGroup.MapPost("/", CreateProduct);

// With authorization
app.MapGet("/secure", [Authorize] () => "Secret data");

// With validation
app.MapPost("/products", async (
    [FromBody] CreateProductDto dto,
    IValidator<CreateProductDto> validator,
    IProductService service) =>
{
    var validationResult = await validator.ValidateAsync(dto);
    if (!validationResult.IsValid)
        return Results.ValidationProblem(validationResult.ToDictionary());

    var product = await service.CreateAsync(dto);
    return Results.Created($"/products/{product.Id}", product);
});

app.Run();
```

### TypedResults (Strongly Typed)

```csharp
app.MapGet("/products/{id}", async Task<Results<Ok<Product>, NotFound>> (
    int id,
    IProductService service) =>
{
    var product = await service.GetByIdAsync(id);
    return product is not null
        ? TypedResults.Ok(product)
        : TypedResults.NotFound();
});
```
