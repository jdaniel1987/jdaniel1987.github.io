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


### **2. Adding Parameters to Routes**

Minimal APIs support parameters in route definitions, making it easy to create dynamic endpoints:

```c#
app.MapGet("/api/greet/{name}", (string name) => $"Hello, {name}!");
```

This endpoint greets the user by name, demonstrating how Minimal APIs handle route parameters with ease.

### **3. Integrating with Middleware**

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

To use the middleware, you need to register it in the request pipeline in Program.cs
```c# Program.cs
app.UseMiddleware<RequestTimingMiddleware>();
```
In this example, the middleware measures the time taken to process each request and adds this value as a X-Elapsed-Time header in the response. This can be useful for debugging or monitoring the performance of your application.


By leveraging middleware, you can manage cross-cutting concerns such as logging, authentication, and more, without complicating your endpoint definitions.

Minimal APIs are a great addition to the .NET ecosystem, offering a straightforward and efficient way to build APIs. Whether you’re developing a small microservice or a larger application, Minimal APIs provide the flexibility and simplicity needed to get your project up and running quickly.
