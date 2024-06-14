---
title: C# Fluent Assertions
date: 2024-06-14
---


TODO fluent assertions vs that
nicer syntax, nicer error message
test framework agnostic
multiple assertions

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


Collections

watch out for containsSingle("a")

HaveLength(3), also useful for strings

what you actually want is
        responseBody.Should().ContainSingle().Which.ErrorMessage.Should()
            .Be("You do not have access to this resource");


ContainsSingle just verifies it has one value

        responseBody.Should().OnlyContain(x => x > 5)


use which and "and" for multiple assertions in on single line