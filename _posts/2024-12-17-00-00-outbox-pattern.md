---
layout: post
title: "The Outbox Pattern"
date: 2024-12-17 00:00:00
categories: [C#, .NET, Patterns, Outbox, Distributed systems]
permalink: /OutboxPattern
---

**The Outbox Pattern** is a powerful technique used in distributed systems to ensure message delivery reliability when integrating with external systems or event-based architectures. In this guide, we will explore how to implement the Outbox Pattern in .NET.

![Outbox Pattern](/assets/img/posts/outbox-pattern.png)

## What is the Outbox Pattern? ##

The Outbox Pattern **ensures that database operations and message publishing are performed atomically**, meaning they are treated as a single, indivisible unit of work. This guarantees that either **all operations succeed together or none of them take effect**. This is especially important when working with event-driven systems to avoid inconsistencies caused by partial failures.

### How It Works ###

1. **Transactional Writing**: Events or messages to be sent are written to an `Outbox` table in the same database transaction as the business operation.
2. **Background Processing**: A separate process reads messages from the `Outbox` table and publishes them to the external system (e.g., a message broker).

## Setting Up the Outbox Pattern ##

### Step 1: Create the Outbox Table

Define an `OutboxMessage` table in your database:

```sql
CREATE TABLE OutboxMessage (
    Id UNIQUEIDENTIFIER PRIMARY KEY,
    EventType NVARCHAR(256) NOT NULL,
    Payload NVARCHAR(MAX) NOT NULL,
    CreatedAt DATETIME NOT NULL DEFAULT GETDATE(),
    ProcessedAt DATETIME NULL
);
```

### Step 2: Add the Outbox Entity ###

Create a corresponding model in your project:

```csharp
public class OutboxMessage
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public string EventType { get; set; }
    public string Payload { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? ProcessedAt { get; set; }
}
```

### Step 3: Save Outbox Messages in a Transaction ###

When performing business logic, save the event to the `OutboxMessage` table:

```csharp
public async Task SaveOrderAsync(Order order)
{
    using var transaction = await _dbContext.Database.BeginTransactionAsync();

    try
    {
        _dbContext.Orders.Add(order);

        var outboxMessage = new OutboxMessage
        {
            EventType = nameof(OrderCreatedEvent),
            Payload = JsonSerializer.Serialize(new OrderCreatedEvent(order.Id, order.TotalAmount))
        };

        _dbContext.OutboxMessages.Add(outboxMessage);

        await _dbContext.SaveChangesAsync();
        await transaction.CommitAsync();
    }
    catch
    {
        await transaction.RollbackAsync();
        throw;
    }
}
```

### Step 4: Process the Outbox Messages

Use a background service to read and publish messages from the `OutboxMessage` table:

```csharp
public class OutboxProcessor : BackgroundService
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<OutboxProcessor> _logger;

    public OutboxProcessor(IServiceProvider serviceProvider, ILogger<OutboxProcessor> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken cancellationToken)
    {
        while (!cancellationToken.IsCancellationRequested)
        {
            try
            {
                using var scope = _serviceProvider.CreateScope();
                var dbContext = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();

                var messages = await dbContext.OutboxMessages
                    .Where(m => !m.Processed)
                    .ToListAsync(stoppingToken);

                foreach (var message in messages)
                {
                    // Publish to message broker
                    await PublishMessageAsync(message);

                    // Mark as processed
                    message.Processed = true;
                }

                await dbContext.SaveChangesAsync(stoppingToken);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error processing outbox messages");
            }
        }
    }
}
```

### Step 5: Register the Background Service

Add the service to your `Program.cs` or `Startup.cs`:

```csharp
builder.Services.AddHostedService<OutboxProcessor>();
```

## Benefits of the Outbox Pattern

- **Atomicity**: Ensures that business operations and message publishing happen together.
- **Reliability**: Reduces the risk of lost messages in distributed systems.
- **Flexibility**: Allows asynchronous processing without blocking the main business logic.

## Conclusion

The Outbox Pattern is a robust solution for ensuring reliable messaging in distributed systems. By implementing it in .NET, you can build resilient and scalable applications that handle message delivery with consistency and efficiency.
