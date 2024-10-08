---
layout: post
title: "A Complete Guide to Clean Architecture"
date: 2024-10-17 00:00:00
categories: [C#, .NET, Clean Architecture, Architecture, Software Design, Separation of Concerns, Dependency Inversion]
permalink: /CleanArchitecture
---

Clean Architecture, popularized by Robert C. Martin (Uncle Bob), is a design approach that emphasizes separating software systems into layers with clear boundaries, ensuring that the business logic remains independent from external concerns like UI, databases, or frameworks. This separation allows systems to be more maintainable, testable, and scalable. In this post, we’ll explore the core concepts of Clean Architecture and demonstrate how they can be applied to .NET projects.  
![Clean Architecture basic graph](/assets/img/posts/clean-architecture-raw.png)

## Core Principles of Clean Architecture ##

### 1. Separation of Concerns ###
Clean Architecture promotes a clear division between different parts of an application. This separation is often achieved by organizing the code into layers. Each layer has a specific responsibility, and dependencies between them should flow in one direction: from the outer layers (infrastructure, UI) to the innermost core (business logic).

### 2. Dependency Inversion Principle (DIP) ###
The Dependency Inversion Principle states that high-level modules should not depend on low-level modules. Both should depend on abstractions. In Clean Architecture, this is implemented by introducing interfaces and Mediator pattern to decouple layers.  

Example:
```csharp
// High-level module
public class OrderService
{
    private readonly IOrderRepository _orderRepository;

    public OrderService(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }

    public void PlaceOrder(Order order)
    {
        // Business logic for placing an order
        _orderRepository.Save(order);
    }
}
```
```csharp
// Low-level module (implementation)
public class SqlOrderRepository : IOrderRepository
{
    public void Save(Order order)
    {
        // Save order to SQL database
    }
}
```
### 3. Entities ###
Entities are the core business objects that have a distinct identity and are responsible for encapsulating business logic. **<u>They should be completely isolated from external services, databases, and UI concerns</u>**.

Example:
```csharp
public class Order
{
    public Guid Id { get; set; }
    public DateTime OrderDate { get; set; }
    public Customer Customer { get; set; }
    public List<OrderItem> Items { get; set; }

    public decimal TotalAmount => Items.Sum(item => item.Price * item.Quantity);
}
```

### 4. Use Cases (Application layer) ###
Use cases represent the application-specific logic. They orchestrate the interaction between domain entities and external services (like repositories, notifications, or external APIs) **through abstractions**, ensuring they remain agnostic to the implementation details of these external services.

Example:
```csharp
public class PlaceOrderUseCase
{
    private readonly IOrderRepository _orderRepository;
    private readonly IPaymentService _paymentService;

    public PlaceOrderUseCase(IOrderRepository orderRepository, IPaymentService paymentService)
    {
        _orderRepository = orderRepository;
        _paymentService = paymentService;
    }

    public void Execute(Order order)
    {
        // Business logic for placing an order
        _paymentService.ProcessPayment(order);
        _orderRepository.Save(order);
    }
}
```

### 5. Interface Adapters ###
Interface adapters are responsible for converting data from the format most convenient for the use case and entities to the format needed for the outer layers (UI, database) and viceversa. They help maintain the independence of the core business logic.

Example:
```csharp
public class OrderDto
{
    public Guid Id { get; set; }
    public DateTime OrderDate { get; set; }
    public string CustomerName { get; set; }
}
```
```csharp
public OrderDto ToOrderDto(Order order)
{
    return new OrderDto()
    {
        Id = order.Id,
        OrderDate = order.OrderDate,
        CustomerName = order.Customer.Name
    };
}
```

### 6. Frameworks and External Tools ###
The outermost layer in Clean Architecture consists of frameworks, databases, and other external tools. This layer should not contain business logic but rather implement the interfaces defined by the inner layers.

Example:
```csharp
public class SqlOrderRepository : IOrderRepository
{
    public void Save(Order order)
    {
        // Save order to SQL database
    }
}
```

## Layers in Clean Architecture ##
![Clean Architecture dependencies](/assets/img/posts/clean-architecture-dependencies.png)

### Presentation ###
The Presentation Layer in software architecture is responsible for **interacting with the end user**. It handles everything related to the user interface (UI), including rendering views, capturing user input, and sending it to the appropriate use cases or services within the application.  
It includes **Controllers and Endpoints, Views, View-Models and User Interactions**.

### Infrastructure ###
The Infrastructure Layer in software architecture **provides the technical capabilities and services needed to support the application’s operations**. It serves as the backbone of the system, handling communication with external systems, data storage, and other lower-level concerns that are necessary for the application to function effectively.  
It includes **Data Access, External services, Messaging brokers, Logging and Monitoring**.

### Application ###
The Application Layer in software architecture encapsulates the **application-specific logic and serves as the bridge between the Presentation Layer and the Domain Layer**. It orchestrates the flow of data and commands, ensuring that the core business logic is executed correctly while remaining agnostic to the details of user interaction and data persistence.  
It includes **Use Cases / Mediator Handlers, Services, Commands, Queries, Mappers, and interfaces for repositories and external services (which are implemented in the Infrastructure Layer)**.

### Domain ###
The Domain Layer in software architecture encapsulates the **core business logic and rules that govern the application's behavior**. It represents the heart of the application, where fundamental entities and their relationships are defined. This layer is focused on the domain model, which reflects the real-world scenarios that the application aims to address.  
It includes **Entities, Value Objects, Aggregates, Domain Services, Domain Events, and business logic classes such as Specifications and Policies**.

## Flow of Data ##
It is important not to confuse the previous diagram showing the direction of dependencies with the direction of data flow. While dependencies illustrate how different layers and components interact with one another, the flow of data emphasizes how information moves through the application from one layer to another.
![Clean Architecture dependencies vs data flow](/assets/img/posts/ca-dependencies-vs-data-flow.png)

## Benefits of Clean Architecture ##
- **Maintainability**: By separating concerns and decoupling layers, it becomes easier to maintain and extend the application over time.
- **Testability**: Since the business logic is decoupled from frameworks and external dependencies, it's easier to write unit tests for the core logic.
- **Flexibility**: The architecture allows you to switch frameworks, databases, or UI layers with minimal impact on the core business logic.

## Challenges of Clean Architecture ##
- **Complexity**: For smaller projects, the additional layers and abstractions introduced by Clean Architecture may feel like overkill.
- **Learning Curve**: Developers unfamiliar with the principles of Clean Architecture may find it challenging to implement initially.

## Relationship with SOLID Principles ##
Clean Architecture closely aligns with the SOLID principles, especially the Dependency Inversion Principle. It ensures that the core business logic is protected from changes in the outer layers, promoting a more stable and flexible system.

## Conclusion ##
Clean Architecture offers a structured way to build software that is maintainable, testable, and flexible. While it introduces complexity, especially for smaller projects, the benefits for large-scale systems make it a powerful design approach. By applying these principles in .NET projects, you can create systems that are easier to maintain and extend over time.

## Want more? ##
Explore **Uncle Bob**'s book *Clean Architecture: A Craftsman's Guide to Software Structure and Design* to dive deeper into the principles and practices of Clean Architecture.

## Link to example project ##
[![GitHub](/assets/icons/icons8-github.svg)](https://github.com/jdaniel1987/GameShop.CQRS) [Clean Architecture](https://github.com/jdaniel1987/GameShop.CQRS)  
This project provides a robust solution using Clean Architecture, along with other patterns such as CQRS and DDD.  
**Please note the following dependencies:**
- API projects depend on Application projects.  
- Infrastructure projects depend on Application projects.  
- Application projects depend on Domain projects.  
