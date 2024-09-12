---
layout: post
title: "Introduction to Minimal APIs in .NET"
date: 2024-09-11
categories: [C#, .NET, Minimal APIs, REST, Web Development]
permalink: /MinimalApis
---
As modern web applications evolve, developers seek ways to create APIs with less complexity and more efficiency. This is where Minimal APIs in .NET come into play. Introduced with .NET 6, Minimal APIs provide a lightweight way to build HTTP APIs with minimal setup, allowing developers to focus on the core logic of their applications.

Minimal APIs reduce the amount of boilerplate code typically required in ASP.NET Core applications, streamlining the process of defining routes, handling requests, and returning responses. By offering a simple, yet powerful approach, Minimal APIs are ideal for building small, microservices-oriented applications, as well as prototyping larger systems.

## **Advantages of Minimal APIs**

- **Simplified Syntax:** Minimal APIs enable you to define routes and endpoints with minimal syntax, reducing the overhead of setting up a full ASP.NET Core project.
- **Faster Development:** With less code to write and maintain, you can develop and deploy APIs faster, making it easier to iterate and refine your application.
- **Performance:** Due to their lightweight nature, Minimal APIs can offer better performance, particularly in scenarios where speed and efficiency are critical.
- **Flexibility:** Minimal APIs are highly customizable and can be easily integrated with other ASP.NET Core features, such as middleware, dependency injection, and more.
- **Scalability:** While designed for simplicity, Minimal APIs can still scale to meet the demands of larger applications, especially when combined with modular patterns like Carter.
- **Great for Microservices:** The minimalistic approach aligns well with microservices architectures, where small, independently deployable services are the norm.

## **Examples of Usage**

### **1. Setting Up a Minimal API**

To get started with Minimal APIs, create a new .NET Web API project and add the necessary route definitions directly within the `Program.cs` file. Here’s an example:

```c#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/api/greet", () => "Hello, world!");

app.Run();
```

In this example, a simple GET endpoint is defined that returns a greeting. Notice how the setup is concise, with no need for controllers or additional routing configurations.

**NOTE**: You can use all existing verbs in Minimal APIs (Get, Post, Put, Patch, Delete)


### **2. Adding Parameters to Routes**

Minimal APIs support parameters in route definitions, making it easy to create dynamic endpoints:

```c#
app.MapGet("/api/greet/{name}", (string name) => $"Hello, {name}!");
```

This endpoint greets the user by name, demonstrating how Minimal APIs handle route parameters with ease.

Attributes can be used on parameters to provide additional metadata, control binding behavior, or apply validation:

```c#
app.MapGet("/api/resource", ([FromQuery] string param) =>
{
    return $"Query parameter: {param}";
});
```

Here are the attributes commonly used in ASP.NET Core Minimal APIs:

[FromQuery] – Binds a parameter to a query string value.  
[FromRoute] – Binds a parameter to a route value.  
[FromBody] – Binds a parameter to the body of the request.  
[FromHeader] – Binds a parameter to a value from the HTTP request headers.  
[FromForm] – Binds a parameter to form data.  

### **3. Integrating with Middleware(Optional)**

Minimal APIs can be easily integrated with ASP.NET Core middleware to add custom logic to the request processing pipeline:

```c#
public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;

    public RequestTimingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();

        await _next(context);

        stopwatch.Stop();
        var elapsedTime = stopwatch.ElapsedMilliseconds;

        context.Response.Headers.Add("X-Elapsed-Time", $"{elapsedTime}ms");
    }
}
```

To use the middleware, you need to register it in the request pipeline in Program.cs:
```c#
app.UseMiddleware<RequestTimingMiddleware>();
```
In this example, the middleware measures the time taken to process each request and adds this value as a X-Elapsed-Time header in the response. This can be useful for debugging or monitoring the performance of your application.

By leveraging middleware, you can manage cross-cutting concerns such as logging, authentication, and more, without complicating your endpoint definitions.

### **4. Using Filters**
Filters are used to handle concerns such as validation, error handling, logging, and other cross-cutting concerns.
```c#
app.MapPost("/users", async (User user) =>
{
    // .................... logic
    return Results.Created($"/users/{user.Id}", user);
})
.AddEndpointFilter(async (context, next) =>
{
    var user = (User)context.Arguments[0];

    if (string.IsNullOrWhiteSpace(user.Name) || !user.Email.Contains("@"))
    {
        return Results.BadRequest("Invalid user data");
    }

    return await next(context);
});
```

### **5. Dependency Injection**
In Minimal APIs, services registered in the application's service container (using builder.Services.Add...) can be injected as parameters into the endpoint handlers.
```c#
builder.Services.AddScoped<IMyService, MyService>();
```

```c#
app.MapGet("/greet", (IMyService service) =>
{
    return service.GetGreeting();
});
```

### **6. Documenting with Open API**
Used to automatically generate detailed API documentation. This is typically done through tools like Swagger, which generate an interactive interface based on your API's structure.

```c#
app.MapGet("/greet", () => "Hello, World!")
    .WithName("Greet")    // Assign a name to the operation
    .WithOpenApi();       // Automatically document this route
```
![Open Api 1](/assets/img/posts/swagger-open-api.png)

```c#
app.MapPost("api/AddGameConsole", (GameConsole gameConsole) =>
{
    // .................................... Logic

    return result.IsSuccess ?
        Results.Created(gameConsole) :
        Results.BadRequest(result.Error);
})
.WithOpenApi(operation =>
{
    operation.Summary = "Adds a new games console";
    operation.Description = "Creates a new games console entry in the system.";
    return operation;
})
.WithName(nameof(AddGameConsoleModule))
.WithTags(nameof(GameConsole))
.ProducesValidationProblem()
.Produces(StatusCodes.Status201Created)
.Produces(StatusCodes.Status400BadRequest)
.Produces(StatusCodes.Status500InternalServerError);
```
![Open Api 1](/assets/img/posts/swagger-open-api-detailed.png)

### **Conclusion**

Minimal APIs are a great addition to the .NET ecosystem, offering a straightforward and efficient way to build APIs. Whether you’re developing a small microservice or a larger application, Minimal APIs provide the flexibility and simplicity needed to get your project up and running quickly.
