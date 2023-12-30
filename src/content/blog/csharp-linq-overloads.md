---
title: C# Linq Overloads
date: 2023-12-29
---
Linq in C# provides extension methods to the IEnumerable interface that let you work with enumerables using a concise and declarative syntax. Whilst every C# developer is familiar with Linq syntax, there are some lesser known overloads of the popular methods that can be useful to have in your back pocket.

### Select/SelectMany/Where with index
```Select```, ```SelectMany```, and ```Where``` all have an overload that provide access to the index of the element currently being evaluated. Take Select for instance, the standard parameter is a ```Func<T, U>```, but the ```Func<T, int, U>``` overload can be used when the index is required in the selector function.

##### Usage
```csharp
Enumerable.Range(1, 5).Select((value, index) => (value, index))
// (1,0), (2,1), (3,2), (4,3), (5,4)

Enumerable.Range(1, 5).Where((_, index) => index % 2 ==0)
//filters the IEnumerable leaving values where the index is even:
// 1, 3, 5

Enumerable.Range(1, 5).SelectMany((value, index) => new List<(int,int)>{(value,index)});
// (1,0), (2,1), (3,2), (4,3), (5,4)
```
I find the overload of ```Select``` to be the most useful out of the three, If I need to loop over a collection and need the index as well as the element I prefer to do
```csharp
foreach(var (el, i) in someCollection.Select((el,i) => (el,i)))
{}
```
as opposed to
```csharp
for(var i = 0; i < someCollection.Count; i++)
{
    var el = someCollection[i]
}
```

I found myself needing this a lot recently so I've created an extension method


```csharp
public static IEnumerable<(T val, int index)> Enumerate<T>(this IEnumerable<T> list) =>
            list.Select((x, i) => (x, i));
```

This gives me functionality close to the enumerate function from Python. Using this I can loop through a collection with the index in a concise way:

```csharp
foreach(var (el, i) in someCollection.Enumerate()){}
```


### SelectMany with result selector
You're probably familiar with using SelectMany to flatten nested lists. But you can use the overload with the selector function parameter where you'll have access to the "parent" and "child" and be able to create the element that will end up in the returned enumerable.

##### Usage
```csharp
class PetOwner
{
    public string Name { get; set; }
    public List<string> Pets { get; set; }
}

PetOwner[] petOwners =
{
    new () { Name = "Higa", Pets = new List<string> { "Scruffy", "Sam" } },
    new () { Name = "Hines", Pets = new List<string> { "Dusty" } }
};

petOwners
    .SelectMany(x => x.Pets, (petOwner, pet) => $"{petOwner.Name} - {pet}")
// "Higa - Scruffy", "Higa - Sam", "Hines - Dusty"
```


#### Combining index and selector function on SelectMany
Say you wanted to get all possible combinations of a collection of numbers, eg for the integers 1, 2, 3, you want (1, 2), (1, 3), (2, 3). This can be achived by combining the two techniques above, and some usage of ```Skip```.

```csharp
var collection = Enumerable.Range(1, 3).ToList();

var combinations = collection.SelectMany(
            (_, i) => collection.Skip(i + 1),
            (parent, child) => (parent, child));
// (1, 2), (1, 3), (2, 3)
```
Here I'm using ```SelectMany``` where the collection selector will be everything after the current element (```.Skip(i + 1)```), then the result selector is used to combine the original number with each number after it in the collection as a tuple, finally SelectMany will flatten this into one ```IEnumerable<(int, int)>```. This can be simplified again use Tuple.Create method group:
```csharp
var combinations = collection.SelectMany(
            (_, i) => collection.Skip(i + 1),
            Tuple.Create);
```


### FirstOrDefault / SingleOrDefault / LastOrDefault with default value
```FirstOrDefault```, ```SingleOrDefault```, and ```LastOrDefault``` are commonly used to try and find an element you're not certain will be in a collection, if your predicate is not satisifed the default value will be the default value of whatever type your collection is made up of, eg ```null``` for reference types or ```0``` for ```int```. However you can specify a default value with the (Func<T, bool> predicate, T defaultValue) overload.

##### Usage
```csharp
new List<int>{1, 3, 5}
    .FirstOrDefault(x => x % 2 == 0, -1);
// -1
```

### Min/Max with selector
Min/Max are usually used with no arguments, and return the minimum/maximum value of a collection of numbers (int, double, long etc). However you can use the overload with the selector function to transform the numbers, and then these new numbers are used for comparison.

##### Usage

```csharp
Enumerable.Range(1, 4).Max(x => (5 - x) * x);
// 6
```

This is a more consise and efficient way to write:

```csharp
Enumerable.Range(1, 4).Select(x => (5 - x) * x).Max();
// 6
```

### Take with Range
Normally you provide ```Take``` with an ```int``` argument but there is also an overload that takes a [Range](https://learn.microsoft.com/en-us/dotnet/api/system.range?view=net-8.0) argument. ```Take``` is often used in combination with ```Skip``` to provide pagination functionality, but using the Range overload you can remove the skip entirely

##### Usage
```csharp
var pageNumber = 2;
var pageSize = 10;
// using traditional Skip/Take for pagination
Enumerable.Range(1, 50).Skip((pageNumber -1) * pageSize).Take(pageSize);
// 11, 12, 13, 14, 15, 16, 17, 18, 19, 20

// using Take Range overload
Enumerable.Range(1, 50).Take(((pageNumber - 1) * pageSize)..(pageNumber * pageSize));
// 11, 12, 13, 14, 15, 16, 17, 18, 19, 20
```



### GroupBy with selector function
There is an overload of GroupBy if you want to tranform your groupings without having to use an additional select statement,
```csharp
class Pet
{
    public string Name { get; set; }
    public double Age { get; set; }
}

var petsList =
    new List<Pet>
    {
        new() { Name="Barley", Age=8.3 },
        new() { Name="Boots", Age=4.9 },
        new() { Name="Whiskers", Age=1.5 },
        new() { Name="Daisy", Age=4.3 }
    };

petsList.GroupBy(
            pet => Math.Floor(pet.Age), // grouping key
            pet => pet.Age, // value
            (key, values) => $"Key: {key}, Count: {values.Count()}, Minimum: {values.Min()}, Maximum: {values.Max()}"); // selector function

// "Key: 8, Count: 1, Minimum: 8.3, Maximum: 8.3", "Key: 4, Count: 2, Minimum: 4.3, Maximum: 4.9", "Key: 1, Count: 1, Minimum: 1.5, Maximum: 1.5"
```