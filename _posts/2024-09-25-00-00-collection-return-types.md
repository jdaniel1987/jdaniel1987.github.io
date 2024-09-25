---
layout: post
title: "Understanding Return Types: <br />IEnumerable<T>, IReadOnlyCollection<T>, and List<T>"
date: 2024-09-25 00:00:00
categories: [C#, .NET, Collections]
tags: [C#, IEnumerable, IReadOnlyCollection, List, .NET]
permalink: /CollectionReturnTypes
---

When working with collections in C#, choosing the appropriate return type for your methods can greatly affect performance, design, and flexibility. In this post, we'll explore three commonly used return types: `IEnumerable<T>`, `IReadOnlyCollection<T>`, and `List<T>`. We'll break down their differences, use cases, advantages, and disadvantages to help you decide when to use each one.

## 1. IEnumerable<T> ##
`IEnumerable<T>` is the most basic interface for collections in .NET. It allows you to iterate over a sequence of elements, without guaranteeing that the data is stored in memory or readily accessible.

### Example ###
```csharp
public static IEnumerable<int> GetNumbersDeferredExecution() =>
    // Deferred execution, DANGEROUS
    Enumerable.Range(1, 5).Select(_ => new Random().Next(1, 100));
```

### Advantages ###
- **Deferred execution**: The data is generated only when needed, which can be more efficient in terms of memory and performance if you're working with large datasets or expensive operations.
- **Flexibility**: `IEnumerable<T>` works with any collection type that implements it, including arrays, lists, or even custom collections.
- **Low memory footprint**: Since data isn't necessarily stored in memory, you avoid materializing large collections until absolutely necessary.

### Disadvantages ###
- **No access to `Count` or index**: You can’t directly get the size of the collection or access elements by index.
- **Potential inefficiency**: Accessing the data multiple times can be inefficient because **<u>the sequence is re-evaluated every time it’s iterated</u>** unless you explicitly materialize it (e.g., using `ToList()`):  

```csharp
var numbers = GetNumbersDeferredExecution();
// First iteration
Console.WriteLine("First iteration of random numbers:");
Console.WriteLine(string.Join(" ", numbers)); // OUTPUT: 15 37 65 59 38

Console.WriteLine();
Console.WriteLine("Second iteration of random numbers:");
Console.WriteLine(string.Join(" ", numbers)); // OUTPUT: 75 14 74 27 44
```

### When to use ###
- When you're working with large datasets or operations that you want to be evaluated lazily.
- When you're sure that the consumer will only need to iterate over the collection once.

---

## 2. IReadOnlyCollection<T> ##
`IReadOnlyCollection<T>` is a more specialized interface that represents a read-only collection with a known size. It guarantees that consumers cannot modify the collection, but you still have access to the `Count` property.

### Example:

```csharp
public static IReadOnlyCollection<int> GetReadOnlyNumbers() =>
    // Materialized collection, immutable
    Enumerable.Range(1, 5).Select(_ => new Random().Next(1, 100)).ToList();
```

### Advantages:
- **Read-only safety**: Ensures that the consumer of the collection cannot modify its contents.
- **Access to `Count`**: Unlike `IEnumerable<T>`, you can directly retrieve the number of elements in the collection.
- **Encapsulation**: Hides the implementation details (e.g., whether it’s a `List<T>` or `Array`) while still providing read-only access.

### Disadvantages:
- **Materialized data**: Unlike `IEnumerable<T>`, the collection is materialized in memory, which may not be ideal for very large datasets.
- **Limited functionality**: You can’t modify the collection or access it by index, limiting its use in scenarios where those operations are needed.

### When to use:
- When you want to expose a collection to consumers, but you need to guarantee it won't be modified.
- When the collection’s size is relevant, and you need to ensure it's materialized.

---

## 3. List<T> ##
`List<T>` is the most commonly used collection type in .NET. It provides full access to the collection, including the ability to modify it and access elements by index.

### Example:

```csharp
public static List<int> GetNumbersAsList() =>
    // Full list access
    Enumerable.Range(1, 5).Select(_ => new Random().Next(1, 100)).ToList();
```

### Advantages:
- **Full control**: You can add, remove, and modify elements, and access them by index.
- **Efficient for smaller collections**: When working with relatively small collections that need to be modified, `List<T>` is the go-to option.
- **Rich API**: Provides a wide range of methods for manipulating the data (`Add`, `Remove`, `Insert`, etc.).

### Disadvantages:
- **Higher memory usage**: Since the data is fully materialized in memory, it can be less efficient when working with large datasets.
- **Potential overexposure**: Returning a `List<T>` allows consumers to modify the data, which may not always be desirable. In such cases, exposing too much control can lead to unintended side effects.

### When to use:
- When you need full control over the collection and expect to modify or access elements by index.
- When working with small to moderately sized datasets that won't significantly impact memory.

---

## BONUS: Using `ToList()` with `IEnumerable<T>` ##
If you call `ToList()` on an `IEnumerable<T>`, the collection will be materialized immediately, and the benefits of deferred execution are lost. However, it can improve efficiency if you're going to access the collection multiple times.

```csharp
public static IEnumerable<int> GetNumbersMaterialized() =>
    // Materialized collection, flexibility
    Enumerable.Range(1, 5).Select(_ => new Random().Next(1, 100)).ToList();
```

By converting to a list, you **avoid the inefficiencies of multiple iterations** when calling GetNumbersAsList(). You also **retain flexibility**, such as being able to later convert the collection to a different type (e.g., an Array). However, this approach **sacrifices the potential benefits of deferred execution**.

## Conclusion ##

Each of these return types has its place depending on your specific requirements:

- **`IEnumerable<T>`**: Use when you need flexibility, deferred execution, or when you're working with large datasets.
- **`IReadOnlyCollection<T>`**: Ideal for scenarios where you want to expose a collection but prevent modification, while also giving access to the collection’s size.
- **`List<T>`**: Best for cases where you need full control over the collection, including modifications and random access.

Choosing the right return type can have a big impact on the performance, safety, and flexibility of your code. Make sure to understand the trade-offs of each option to optimize for your particular use case.

## Link to example project: ##
[![GitHub](/assets/icons/icons8-github.svg)](https://github.com/jdaniel1987/CollectionTypes) [CollectionTypes](https://github.com/jdaniel1987/CollectionTypes)
