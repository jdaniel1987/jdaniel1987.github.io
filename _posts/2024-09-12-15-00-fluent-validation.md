---
layout: post
title: "Introduction to FluentValidation"
date: 2024-09-12 15:00:00
categories: [C#, .NET, FluentValidation, Validation]
permalink: /FluentValidation
---
In the world of application development, ensuring that your data is valid before processing it is crucial. This is where **FluentValidation** comes in, offering a powerful and flexible way to handle validation in .NET applications.

**FluentValidation** is a popular library that provides a fluent interface for defining validation rules for your objects. It helps you keep your validation logic separate from your business logic, making your code cleaner and more maintainable.

## **Advantages of FluentValidation**

- **Fluent Interface:** It uses a fluent interface, which allows you to build validation rules in a clear and expressive manner.
- **Separation of Concerns:** Validation rules are separated from your business logic, promoting a clean and organized codebase.
- **Extensibility:** FluentValidation is highly extensible and allows you to create custom validators and rules tailored to your needs.
- **Integration:** It integrates smoothly with ASP.NET Core and other .NET frameworks, providing built-in support for dependency injection.
- **Testability:** Validation logic can be tested independently of your business logic, improving the reliability of your application.

## **Examples of Usage**

### **1. Install the nuget package**

To get started with FluentValidation, install the FluentValidation NuGet package. Open your project, right-click -> Manage NuGet Packages, search for 'FluentValidation', and click Install.

### **2. Create the Request Object**
Define the object you want to validate. For example, create a 'CreateUserRequest' class:

```csharp
public class CreateUserRequest
{
    public string Name { get; set; }
    public string Email { get; set; }
}
```

### **3. Define a Validator** 
Create a validator by inheriting from AbstractValidator<T>, where T is the type of object you want to validate. Define your rules in the constructor.
```c#
using FluentValidation;

public class CreateUserRequestValidator : AbstractValidator<CreateUserRequest>
{
    public CreateUserRequestValidator()
    {
        RuleFor(x => x.Name).NotEmpty().WithMessage("Name is required.");
        RuleFor(x => x.Email).NotEmpty().EmailAddress().WithMessage("A valid email is required.");
    }
}
```

### **4. Register FluentValidation in Program.cs** 
To integrate FluentValidation with your ASP.NET Core application, register it in the Program.cs file using the extension method provided by FluentValidation.AspNetCore.
```c#
builder.Services.AddControllers()
    .AddFluentValidation(fv => fv.RegisterValidatorsFromAssemblyContaining<CreateUserRequestValidator>());
```

### **5. Using FluentValidation in a Controller/Minimal API Endpoint** 
In your controllers, validation is applied automatically when you use model binding. For example:
```c#
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult CreateUser([FromBody] CreateUserRequest request)
    {
        // If the validation fails, a 400 Bad Request response is automatically returned
        // with the validation errors.
        return Ok();
    }
}
```

For Minimal APIs, you need to validate the request manually:
```c#
app.MapPost("/api/users", async (CreateUserRequest request, IValidator<CreateUserRequest> validator) =>
{
    var result = await validator.ValidateAsync(request);
    if (!result.IsValid)
    {
        return Results.BadRequest(result.Errors);
    }

    // Process the valid request
    return Results.Ok();
})
.WithName("CreateUser")
.WithOpenApi();
```
### Tip on Minimal APIs
If you are using Minimal APIs along with MediatR, you can import MediatR.Extensions.FluentValidation.AspNetCore, and it will automatically validate the command/query/request you send via _mediator.Send(...).
```c#
services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly()));
services.AddValidatorsFromAssemblyContaining<CreateUserRequestValidator>();
services.AddFluentValidation([typeof(CreateUserRequestValidator).Assembly]);
```

### **6. Custom Validation Rules (Optional)**
FluentValidation allows you to create custom validation rules for more complex scenarios.
```c#
public class CustomValidation : AbstractValidator<CreateUserRequest>
{
    public CustomValidation()
    {
        RuleFor(x => x.Name).Must(BeAValidName).WithMessage("Invalid name.");
    }

    private bool BeAValidName(string name)
    {
        // Custom validation logic
        return !string.IsNullOrWhiteSpace(name);
    }
}
```

### Conclusion
**FluentValidation** provides a robust and flexible way to handle validation in your .NET applications. By keeping validation rules separate from business logic, it helps maintain a clean and organized codebase. Start leveraging FluentValidation in your projects to ensure your data is valid and your code is maintainable!
