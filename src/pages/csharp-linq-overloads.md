---
title: C# Linq overloads
href: /csharp-linq-overloads
date: 2023-12-29
layout: ../layouts/PostLayout.astro
---

# TODO, write this

```

    [Test]
    public void linq_tests()
    {
        // anything sepecial to do with range?
        //Enumerable.Range(1, 10).Skip(1).Select(val => (val, Math.Pow(val, 2)))
        //  .Where((_, i) => i % 2 == 0).Dump();

        //select with index, useful for enumerate style functions, also select many with index
        // where with index, not that useful
        // maybe a simple selectmany with selector functiion
        // selectmany with selector function? , explain this.....

        var range = Enumerable.Range(1, 4).ToList();
        var combinations = range.SelectMany(
            (_, i) => range.Skip(i + 1),
            (parent, child) => Tuple.Create(parent, child));

        combinations.Dump();

        // select many with index..., figure this out
        // aggregate with and without seed ?



        //this type of group by...
        /*
         *    var query = petsList.GroupBy(
        pet => Math.Floor(pet.Age),
        pet => pet.Age,
        (baseAge, ages) => new
        {
            Key = baseAge,
            Count = ages.Count(),
            Min = ages.Min(),
            Max = ages.Max()
        });
         *
         */

        // firtordefault/singleOD provide default value
        //var f = Enumerable.Range(1, 10).FirstOrDefault(x => x % 2 == 0, -1);
        //var l = Enumerable.Range(1, 10).LastOrDefault(x => x % 2 == 0, -1);
        //var s = Enumerable.Range(1, 10).SingleOrDefault(x => x % 2 == 0, -1);

        //take with range, can be a shorter version of  skip then take
        Enumerable.Range(1, 10).Take(2..7);
        Enumerable.Range(1, 10).Take(..^1);

        var mins = Enumerable.Range(1, 9).Select(x => (10 - x) * x).Dump().Max();
        var min = Enumerable.Range(1, 9).Max(x => (10 - x) * x);


        mins.Dump();
        min.Dump();

    }```