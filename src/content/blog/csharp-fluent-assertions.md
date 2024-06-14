---
title: C# Fluent Assertions
date: 2024-06-14
---
Fluent Assertions is a N???uget package which provides a set of extension methods that allow you to more naturally specify the expected outcome of unit tests. It's test framework agnostic
Get started with ```dotnet add package FluentAssertions````

## Syntax

The general format of a a Fluent Assertion is that the value being asserted on comes first, then the Should() method is called on it, followed by one of the built in assertion methods, there are many useful built in assertion methods which make it easy read and write assertions. Let's start simple:

```csharp
var result = 5;
result.Should().Be(6);
```

This is the general format where we expect the value provided to the `Be` method to equal the variable we're asserting on.
Most of the assertion methods also have an opposite equivalent, which often begin with `Not` for example `result.Should().NotBe(6);`

`Should().Be(...)` works for any type so it can always be used to compare a variable to an expected value, but depending on the type of the variable we're asserting on, Fluent Assertions provides some helper methods to make asserting easier.

### Boolean Assertion Methods

Pretty straight forward here...

```csharp
var result = true;
result.Should().BeTrue();
result.Should().BeFalse();
```


### Ints/Floats etc

```csharp
var x = 5;
x.Should().BeNegative();
x.Should().BePositive();
x.Should().BeGreaterThan(2);
x.Should().BeGreaterThanOrEqualTo(2);
x.Should().BeLessThan(2);
x.Should().BeLessThanOrEqualTo(2);
```

These are all self explanitory. There is also
```csharp
x.Should().BeInRange(3, 4); // inclusive on lower and upper bound
```

and

```csharp
x.Should().BeCloseTo(4, 1);
```


### Strings

```csharp
var x = "foo";
x.Should().BeLowerCased();
x.Should().BeUpperCased();
x.Should().NotBeNullOrWhiteSpace();
x.Should().HaveLength(5);

x.Should().ContainAny("Z", "A"); // string should contain any of these expected values
x.Should().ContainAll("ABC", "BCD"); // string should contain all of these expected values
```

### Collections

There are many

multiple assertions
throwing exceptions, sync and async
should().Be(value)
BeNull()

ints         x.Should().BeGreaterThan()

strings
        y.Should().BeNullOrWhiteSpace();
        y.Should().BeUpperCased()


Match example
responseBody[0].Should().Match(x => x.As<AsosErrorModel>().ErrorMessage == "5");

should().BeEQuivalentTo

There are also Not variations for most of these


### Collections

```csharp
int[] x = [1, 2, 3, 4];

x.Should().BeEmpty();
x.Should().ContainSingle(); // collection has only one element

x.Should().HaveCount(2);

x.Should().HaveCountGreaterThan(1);
x.Should().HaveCountGreaterThanOrEqualTo(1);
// There is also LessThan equivalents for these

x.Should().HaveElementAt(0, 1); // index to be looked at, value expected

x.Should().BeEquivalentTo([1, 2, 3, 4]); // collections compared by value, not reference

x.Should().Contain(x => x > 10); // should contain at least one element matching the predicate
x.Should().OnlyContain(e => e > 3);
x.Should().NotContain(e => e > 5);
x.Should().BeInAscendingOrder(e => e);
```

`ContainSingle` can sometimes catch people up when asserting on a collection of strings.

```csharp
string[] res = ["abc"];
res.Should().ContainSingle("abc");
```

You might look at this and think it means `res` should have one element which has value "abc" but this is not the case.
ContainsSingle only checks if there is one element
`ContainSingle("abc")` here checks there is only one element, the "abc" argument is a `because` parameter, which only aids in providing a more complete exception message. See about exception messages hereTODO


To check if there is a single element "abc" in the collection you could use one of


```csharp
res.Should().BeEquivalentTo(["abc"]);
res.Should().ContainSingle().And.AllBe("abc");
```

See more about how `And` can be used hereTODO

### Reference Types

Classes and Records have a couple of useful helpers, which are also available for collections as expected.

```csharp
// class definition
public class Thing(int a);

b.Should().BeOfType(typeof(Thing));
b.Should().BeEquivalentTo(new Thing(3));
```

TODO exceptions

### And / Which

`And` and `Which` can be used to chain assertions together. `And` is used for multiple separate assertions, and `Which` is used to specify a condition for the assertion, usually when dealing with collections or properties of an object. It allows you to filter the collection or object properties based on a specific condition before applying the assertion.


```csharp
string actual = "ABCDEFGHI";
actual.Should().StartWith("AB").And.EndWith("HI").And.Contain("EF").And.HaveLength(9);
```
These could all be written as separate assertions, but it is cleaner this way.

```csharp
responseBody.Should().ContainSingle().Which.Should()
    .Match<ErrorModel>(x =>
        x.ErrorMessage ==
        "Required parameter Id was not provided from query string.");

dictionary.Should().ContainValue(myClass).Which.SomeProperty.Should().BeGreaterThan(0);
```

I haven't mentioned the Match method, it allows you to assert by predicate on any type.




what you actually want is
        responseBody.Should().ContainSingle().Which.ErrorMessage.Should()
            .Be("You do not have access to this resource");

        x.Should().Match(x => x) can be used on anything

cointainOnly
Containany
ContainsSingle just verifies it has one value

        responseBody.Should().OnlyContain(x => x > 5)


use which and "and" for multiple assertions in on single line

dictionary.Should().ContainValue(myClass).Which.SomeProperty.Should().BeGreaterThan(0);

There are many methods that have no been covered and vary depending on the type, like asserting on dictionaries
includes `Should().ContainKey(...)`

## Multiple Assertions
Normally, a test will stop when the first assertion fails (because an exception is throw) but if you want to the test
to continue, and collect all failed assertions regardless of whether an earlier one failed, you can do so by creating a new `AssertionScope` in combination with a `using` statement. Now any and all assertions that failed, only are thrown when code execution leaves the scope of `using` statement

```csharp
using (new AssertionScope())
{
    var text = "abc";
    text.Should().HaveLength(5);
    text.Should().BeUpperCased();
}
// gives exception message
// Expected text with length 5, but found string "abc" with length 3.Expected all characters in text to be upper cased, but found "abc".
```



## Error Messaging

By default I think the exception messsage provided by Fluent Assertions are better than test framework defaults.


TODO fluent assertions vs that
nicer syntax, nicer error mess

Fluent Assertions automatically detects what framework you're using and throws the correct exception for that framework, this means it's easier to migrate testing frameworks in the future,
but also it provides a syntax which is easier to read, write but also provides cleaner error messages.

        Assert.That(testVariable, Is.EqualTo(6));
        testVariable.Should().Be(6);
        write error messages


        can also provide a message, in both, compare these
