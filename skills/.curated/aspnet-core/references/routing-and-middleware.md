# Routing and Middleware in ASP.NET Core

## Overview

ASP.NET Core uses a pipeline-based middleware system and a flexible routing mechanism to handle HTTP requests. Understanding how these work together is essential for building robust web applications.

---

## Middleware Pipeline

Middleware components are assembled into a request pipeline. Each component can:
- Process an incoming request
- Pass the request to the next middleware
- Short-circuit the pipeline and return a response

### Registering Middleware

```csharp
var app = builder.Build();

// Built-in middleware
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Custom Middleware

```csharp
// Inline middleware using Use
app.Use(async (context, next) =>
{
    // Before next middleware
    Console.WriteLine($"Request: {context.Request.Path}");

    await next(context);

    // After next middleware
    Console.WriteLine($"Response: {context.Response.StatusCode}");
});

// Terminal middleware using Run
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello from terminal middleware!");
});
```

### Class-Based Middleware

```csharp
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
        _logger.LogInformation("Handling request: {Path}", context.Request.Path);
        await _next(context);
        _logger.LogInformation("Finished handling request.");
    }
}

// Register in Program.cs
app.UseMiddleware<RequestLoggingMiddleware>();
```

---

## Routing

### Conventional Routing (MVC)

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

### Attribute Routing

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]                        // GET api/products
    [HttpGet("{id:int}")]            // GET api/products/42
    [HttpPost]                       // POST api/products
    [HttpPut("{id:int}")]            // PUT api/products/42
    [HttpDelete("{id:int}")]         // DELETE api/products/42
}
```

### Route Constraints

| Constraint | Example | Description |
|---|---|---|
| `int` | `{id:int}` | Matches any integer |
| `guid` | `{id:guid}` | Matches a GUID |
| `minlength(n)` | `{name:minlength(3)}` | String min length |
| `range(min,max)` | `{age:range(1,120)}` | Integer in range |
| `regex(expr)` | `{code:regex(^[A-Z]{{3}}$)}` | Regex match |

### Minimal API Routing

```csharp
app.MapGet("/items", () => Results.Ok(new[] { "item1", "item2" }));

app.MapGet("/items/{id:int}", (int id) =>
    id > 0 ? Results.Ok($"Item {id}") : Results.NotFound());

app.MapPost("/items", (Item item) => Results.Created($"/items/{item.Id}", item));
```

---

## Route Groups (Minimal APIs)

Group related endpoints to share prefixes and middleware:

```csharp
var productsGroup = app.MapGroup("/api/products")
    .RequireAuthorization();

productsGroup.MapGet("/", GetAllProducts);
productsGroup.MapGet("/{id:int}", GetProductById);
productsGroup.MapPost("/", CreateProduct);
```

---

## See Also

- [APIs: Minimal and Controllers](./apis-minimal-and-controllers.md)
- [ASP.NET Core Middleware docs](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/)
- [Routing in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing)
