---
layout: post
title: "Comprehensive Guide to Domain-Driven Design (DDD)"
date: 2024-09-23 00:00:00
categories: [C#, .NET, DDD, Architecture, Domain-Driven Design, Software Design, Bounded Contexts, CQRS, Event Sourcing, Factories, Specifications, Anti-Corruption Layer, Domain Events, Repositories, Value Objects]
permalink: /DomainDrivenDesign
---

Domain-Driven Design (DDD) is a strategic approach to software development that centers on building a deep understanding of the business domain. It provides tools to manage complexity by aligning the software model with business concepts. In this post, we will explore various advanced concepts of DDD, providing examples for each, and examine how they can be applied in .NET projects.

## Core Concepts of DDD ##
### 1. Ubiquitous Language ###
The Ubiquitous Language is a **shared vocabulary used by both developers and domain experts** (i.e Producy Owner). It ensures that all parties involved in the development process are on the same page, making communication more efficient. For example:
- **Customer** means the same thing to both developers and business users.
- **Order Status** has the same life cycle across the application: Pending, Shipped, Delivered.

### 2. Entities ###
Objects that **have a distinct identity**, such as OrderItem, which persist over time.
```c#
public record OrderItem(
    Guid Id,
    string ProductName,
    decimal Price,
    int Quantity);
```


### 3. Value Objects ###
Immutable objects that are defined by their properties rather than a unique identity. Common examples include Address or Price, which are considered equal if all their properties match. Unlike entities, Value Objects **do not have an Id** and focus on structural equality rather than identity.
```c#
public record Price
{
    public decimal Value { get; init; }

    private Price(decimal value)
    {
        if (value < 0)
        {
            throw new ArgumentException("Price cannot be negative", nameof(value));
        }

        Value = value;
    }

    public static Price Create(decimal value) =>
        new(value);

    // Operator overloading to perform operations with prices
    public static Price operator +(Price a, Price b) => new Price(a.Value + b.Value);
    public static Price operator -(Price a, Price b) => new Price(a.Value - b.Value);

    public override string ToString() => Value.ToString();
}
```

### 4. Aggregates ###
An Aggregate is a **cluster of related objects** treated as a single unit.  
Example of an Order aggregate with OrderItems:
```c#
public record Order(
    Guid Id,
    DateTime OrderDate,
    Customer Customer,
    List<OrderItem> Items);
```

### 5. Repositories ###
A Repository acts as a **collection of aggregate roots** and provides an abstraction over data access. It isolates the domain model from the persistence layer.
```c#
public class IOrderRepository
{
    Task<Order> GetById(int id);

    Task Save(Order order);
}
```
### 6. Domain Services ###
Domain Services encapsulate **domain logic** that doesn’t naturally belong to an entity or value object, often because they span across multiple aggregates.
```c#
public class PaymentService
{
    public bool ProcessPayment(Order order, PaymentDetails paymentDetails)
    {
        // Domain logic to process payment
    }
}
```

## Advanced DDD Concepts ##
### 1. Bounded Contexts ###
A Bounded Context defines clear **boundaries around a specific domain model**, preventing ambiguities and inconsistencies across the system. Each context focuses on a particular aspect of the business, ensuring models don’t overlap.

For example, the Order aggregate in the **Sales Context might contain customer and payment details**.
```c#
public record Order(
    Guid Id,
    DateTime OrderDate,
    Customer Customer,
    List<OrderItem> Items,
    Payment Payment);
```

While the Order aggregate in the **Shipping Context might only include shipping-related information**.
```c#
public record Order(
    Guid Id,
    DateTime OrderDate,
    List<OrderItem> Items,
    ShippingAddress ShippingAddress,
    string TrackingNumber);
```

### 2. Domain Events ###
Domain events notify the system of **significant changes in the business logic**. They are used to trigger actions in other parts of the application without creating tight coupling.
```c#
public record OrderPlacedEvent(
    int OrderId,
    DateTime PlacedOn) : IDomainEvent;
```

```c#
public class OrderPlacedHandler : IEventHandler<OrderPlacedEvent>
{
    public Task Handle(OrderPlacedEvent domainEvent)
    {
        // Logic to handle the order placed event
    }
}
```

### 3. Factories ###
A Factory is used to encapsulate the **logic of creating complex aggregates**. It ensures that aggregates are constructed following all domain rules.
```c#
public class OrderFactory
{
    public static Order CreateOrder(Customer customer, List<OrderItem> items)
    {
        var order = new Order(customer.Id);
        foreach (var item in items)
        {
            order.AddItem(item);
        }
        return order;
    }
}
```

### 4. Event Sourcing ###
With Event Sourcing, the **state of an aggregate is represented as a series of domain events**. Instead of storing the current state directly, the system maintains a **log of all the events** that have led to the current state. The current state is reconstructed by replaying these events.
```c#
public class Order
{
    public OrderStatus Status { Get; Set; }
    private List<IDomainEvent> _events = new List<IDomainEvent>();

    public void Apply(OrderPlacedEvent @event)
    {
        // Update the internal state based on the event
        this.Status = OrderStatus.Placed;

        // Store the event for later persistence
        _events.Add(@event);
    }

    public IEnumerable<IDomainEvent> GetUncommittedEvents()
    {
        return _events;
    }
}
```
**Key Points:**
- **Event Storage**: Those events should be persisted somewhere, like databases, blob storage, or event stores. This allows for future reconstruction of the state and enables features like auditing and temporal queries.

- **Replaying Events**: To reconstruct the state of an aggregate, you can replay the stored events. This means that your domain events must be immutable and represent state changes effectively.

- **Handling Uncommitted Events**: It’s often useful to have a method to clear uncommitted events after they have been persisted to avoid duplicating them.

### 5. Anti-Corruption Layer (ACL) ###
The Anti-Corruption Layer is a boundary that protects the internal model of a bounded context from being polluted by external systems. It **translates external models into the domain's models**.
```c#
public class ExternalCustomerAdapter
{
    public Customer Convert(ExternalCustomer externalCustomer)
    {
        // Logic to map external customer to domain customer
    }
}
```
You can use external mappers like Mapster, AutoMapper,...

### 6. Specifications ###
A Specification is a pattern used to encapsulate **business rules into reusable objects**. It can be used to validate objects or filter collections based on specific criteria.
```c#
public class EligibleForDiscountSpecification : ISpecification<Order>
{
    public bool IsSatisfiedBy(Order order)
    {
        return order.TotalAmount > 100;
    }
}
```
```c#
if (discountSpec.IsSatisfiedBy(order))
{
    Console.WriteLine("Order is eligible for a discount.");
}
```

### 7. Policies ###
A Policy encapsulates **complex business rules that might affect different parts of the system**. Policies can be executed in response to domain events or when certain conditions in the domain are met.
```c#
public class DiscountPolicy
{
    public decimal ApplyDiscount(Order order)
    {
        if (order.TotalAmount > 500)
        {
            return order.TotalAmount * 0.10m; // 10% discount
        }
        return 0;
    }
}
```

## Benefits of DDD ##
- **Clarity**: DDD aligns the domain model with the business, making the software design intuitive.
- **Scalability**: By splitting large domains into bounded contexts and leveraging CQRS and Event Sourcing, DDD allows the system to scale both in terms of complexity and performance.
- **Flexibility**: Changes in the business domain are easier to manage and reflect in the software.

## Challenges of DDD ##
- **Learning Curve**: Understanding and applying DDD can be difficult for teams unfamiliar with the patterns and concepts.
- **Complexity**: DDD introduces additional complexity in terms of layers and abstractions, which can be unnecessary for smaller or less complex domains.

## Conclusion ##
Domain-Driven Design offers a systematic approach to managing complexity in software development by aligning the design with business concepts. While it introduces complexity, especially for new teams, the long-term benefits in terms of scalability, maintainability, and business alignment make it an excellent choice for large and intricate domains.

## Want more? ##
Dive into **Eric Evans**' groundbreaking book: Domain-Driven Design: Tackling Complexity in the Heart of Software. Published in 2003, this seminal work lays the foundation for the DDD methodology, offering a comprehensive guide to managing complexity in software systems. By focusing on the core business domain and fostering collaboration between developers and domain experts, Evans provides the blueprint for building more maintainable and scalable software solutions.
