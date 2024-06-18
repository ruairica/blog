---
title: C# Fluent Assertions
date: 2024-06-18
---
Fluent Assertions is a test framework agnostic Nuget package which provides a set of extension methods that allow you to more naturally specify the expected outcome of unit tests.


Get started with `dotnet add package FluentAssertions`

## Syntax
The general format of a a Fluent Assertion is that the value being asserted on comes first, then the Should() method is called on it, followed by one of the built in assertion methods. There are many useful built in assertion methods which make it easy read and write assertions. Let's start simple:

```csharp
var result = 5;
result.Should().Be(6);
```

In this example we expect the value provided to the `Be` method to equal the variable we're asserting on.
Most of the assertion methods also have an opposite equivalent, which often begin with `Not`, for example `result.Should().NotBe(6);`

`Should().Be(...)` works for any value type so it can always be used to compare a variable to an expected value, but depending on the type of the variable we're asserting on, Fluent Assertions provides some helper methods to make asserting easier.

### Boolean Assertion Methods
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

There is also
```csharp
x.Should().BeInRange(3, 4); // inclusive on lower and upper bound
```

and

```csharp
x.Should().BeCloseTo(4, 1); // x should be anywhere from 3 to 5
```

### Strings
```csharp
var x = "foo";
x.Should().BeLowerCased();
x.Should().BeUpperCased();
x.Should().NotBeNullOrWhiteSpace();
x.Should().HaveLength(5);
x.Should().StartWith("a");
x.Should().EndWith("a");
x.Should().ContainAny("Z", "A"); // string should contain any of these expected values
x.Should().ContainAll("ABC", "BCD"); // string should contain all of these expected values
```

### Collections
```csharp
int[] x = [1, 2, 3, 4];

x.Should().BeEmpty();
x.Should().ContainSingle(); // collection has only one element

x.Should().HaveCount(2);

x.Should().HaveCountGreaterThan(1);
x.Should().HaveCountGreaterThanOrEqualTo(1);
// There is also LessThan equivalents for these ^

x.Should().HaveElementAt(0, 1); // at index 0, there should be a 1

x.Should().BeEquivalentTo([1, 2, 3, 4]); // collections compared by value, not reference

x.Should().Contain(x => x > 10); // should contain at least one element matching the predicate
x.Should().OnlyContain(e => e > 3); // all values should match the predicate
x.Should().NotContain(e => e > 5); // no values should match the predicate
x.Should().BeInAscendingOrder(e => e);
```

`ContainSingle` can sometimes catch people out when asserting on a collection of strings.

```csharp
string[] res = ["abc"];
res.Should().ContainSingle("abc");
```

You might look at this and think it means `res` should have one element which has value "abc" but this is not the case.
ContainsSingle only checks if there is one element in the collection.
`ContainSingle("abc")` as used here checks there is only one element, the "abc" argument is the `because` parameter, which only aids in providing a more complete exception message. See about exception messages [here](#error-messaging)


To check if there is a single element "abc" in the collection you could use one of
```csharp
res.Should().ContainSingle().Which.Should().Be("abc");
res.Should().BeEquivalentTo(["abc"]);
res.Should().ContainSingle().And.AllBe("abc");
```

See more about how `And` and `Which` [here](#and--which)

### Reference Types

Classes have a couple of useful helpers, which are also available for collections as expected.

```csharp
public class Thing(int a) : IThing
{
    public int Number { get; set; } = a;
}
public interface IThing {};


IThing foo = new Thing(3);
foo.Should().BeOfType(typeof(Thing));
foo.Should().BeEquivalentTo(new Thing(3)); // compares by value
```

### And / Which

`And` and `Which` can be used to chain assertions together. `And` is used for multiple separate assertions, `Which` is used to specify a condition for the assertion, usually when dealing with collections or properties of an object. It allows you to filter the collection or object properties based on a specific condition before applying the assertion.

```csharp
string actual = "ABCDEFGHI";
actual.Should().StartWith("AB").And.EndWith("HI").And.Contain("EF").And.HaveLength(9);

response.Should().ContainSingle().Which.Should()
    .Match<ErrorModel>(x =>
        x.ErrorMessage ==
        "Required parameter Id was not provided from query string.");

dictionary.Should().ContainValue(myClass).Which.SomeProperty.Should().BeGreaterThan(0);
```
These could all be written as separate assertions, but this syntax is more succinct.
I haven't mentioned the `Match` method, it allows you to assert by predicate on any type.
There are many methods that haven't been covered here, like asserting on dictionaries
includes `Should().ContainKey(...)` and `ContainValue(...)`

## Multiple Assertions
Normally, a test will stop when the first assertion fails (because an exception is thrown) but if you want the test
to continue, and collect all failed assertions regardless of whether an earlier one failed, you can do so by creating a new `AssertionScope` in combination with a `using` statement, now any and all assertions that failed, are thrown only when code execution leaves the scope of the `using` statement

```csharp
using (new AssertionScope())
{
    var text = "abc";
    text.Should().HaveLength(5);
    text.Should().BeUpperCased();
}
// gives exception message:
// Expected text with length 5, but found string "abc" with length 3.Expected all characters in text to be upper cased, but found "abc".
```

## Error Messaging

By default I think the error messsages provided by Fluent Assertions are better than test framework defaults because they are easier to read at a glance, for example

```csharp
var sut = 5;
sut.Should().Be(6);
/* Exception message:
Expected sut to be 6, but found 5. */

// NUnit assert
Assert.That(sut, Is.EqualTo(6));
/* Exception message:
  Assert.That(sut, Is.EqualTo(6))
  Expected: 6
  But was:  5  */
```

All the assertions provided by Fluent Assertions, come with an optional "because" parameter than can be supplied to enhance
the error message.

```csharp
sut.Should().Be(6, "if it is not 6, the application blows up");
/* Exception message:
Expected sut to be 6 because if it is not 6, the application blows up, but found 5.
*/
```