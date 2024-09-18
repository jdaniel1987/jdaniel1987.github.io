---
layout: post
title: "Introduction to FluentAssertions"
date: 2024-09-18 00:00:00
categories: [C#, .NET, Testing, FluentAssertions]
permalink: /FluentAssertions
---

FluentAssertions is a popular library for .NET applications that allows developers to write assertions in a more natural and expressive manner. By using a fluent API, your tests become easier to read and maintain, enabling you to focus on the actual behavior being tested, rather than the implementation details.

## **Advantages of FluentAssertions**

- **Improved Readability:** The fluent API provides a more human-readable way to write assertions.
- **Detailed Failure Messages:** When tests fail, FluentAssertions gives you clear and informative messages about why the test failed.
- **Strongly Typed Assertions:** It supports a wide range of types and provides strong typing to catch errors at compile time.
- **Custom Assertions:** You can extend the library with your own custom assertions for specific use cases.
- **Extensive Features:** FluentAssertions supports collections, exceptions, objects, strings, dates, and much more.

## **Examples of Usage**
### **1. Install the nuget package**

To get started with FluentAssertions, install the FluentAssertions NuGet package. Open your project, right-click -> Manage NuGet Packages, search for 'FluentAssertions', and click Install.

### **2. Basic Assertions**
Instead of the traditional Assert.AreEqual, FluentAssertions allows you to use the more readable Should().Be(), which clearly communicates the intent of the test.
```c#
[Fact]
public void BasicAssertionsExample()
{
    int result = 5 + 5;

    // Instead of using Assert.AreEqual or Assert.IsTrue
    result.Should().Be(10);
}
```

### **3. Asserting Exceptions**
FluentAssertions makes it easy to assert that an exception is thrown in specific scenarios:
```c#
[Fact]
public void ExceptionAssertionsExample()
{
    Action act = () => throw new InvalidOperationException("Invalid operation");

    act.Should().Throw<InvalidOperationException>()
        .WithMessage("Invalid operation");
}
```

### **4. Asserting Collections**
luentAssertions provides powerful capabilities for asserting conditions on collections:
```c#
[Fact]
public void CollectionAssertionsExample()
{
    var fruits = new List<string> { "apple", "banana", "orange" };

    fruits.Should().Contain("apple")
          .And.HaveCount(3)
          .And.ContainInOrder("apple", "banana", "orange");
}
```
This example asserts that:  

- The collection contains "apple".  
- The collection has exactly 3 items.  
- The items appear in the specified order.  

### **5. Asserting Object Properties**
You can assert that the properties of an object have expected values:
```c#
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

[Fact]
public void ObjectAssertionsExample()
{
    var person = new Person { Name = "John", Age = 30 };

    person.Should().BeEquivalentTo(new { Name = "John", Age = 30 });
}
```
The BeEquivalentTo method is particularly useful for asserting object properties **without needing to compare each one manually**.

### **6. Custom Assertions (Optional)**
You can extend FluentAssertions by creating custom assertions for your domain-specific objects. For example:
```c#
public static class CustomAssertions
{
    public static void ShouldHaveValidName(this Person person)
    {
        person.Name.Should().NotBeNullOrEmpty()
                    .And.MatchRegex("^[a-zA-Z]+$");
    }
}

[Fact]
public void CustomAssertionExample()
{
    var person = new Person { Name = "John", Age = 30 };

    person.ShouldHaveValidName();
}
```

### **7. Asserting Asynchronous Operations**
FluentAssertions also supports asserting async methods:
```c#
[Fact]
public async Task AsyncAssertionsExample()
{
    Func<Task> act = async () => await Task.Delay(500);

    await act.Should().CompleteWithinAsync(TimeSpan.FromSeconds(1));
}
```
This asserts that the asynchronous task completes within the specified time limit.

### Conclusion
FluentAssertions is a versatile and expressive library that greatly improves the readability of your unit tests. By using its fluent interface, you can write clear and concise assertions, leading to better maintainability and stronger test suites. Start integrating FluentAssertions into your tests to make them more readable, descriptive, and powerful!
