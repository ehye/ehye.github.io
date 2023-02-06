---
title: C# Cheatsheet
categorties: Note
tags:
    - C#
    - Linq
    - Lambda expression
---

C# Cheatsheet

---

## Basic

### Static readonly vs const

`const` can be faster, but if you change the value of const, you need to rebuild all the clients

If the value will **never** change, then const is fine, other than that, `static` properties are more common.

### Compare structs and classes

Classes and Structs in C# do have a few things in common, namely:

Are compound data types
Can contain methods and events
Can support interfaces
But there are a number of differences. Here’s a comparison:

*Classes:*

- Support inheritance
- Are reference (pointer) types
- The reference can be null
- Have memory overhead per new instance
*Structs:*

- Do not support inheritance
- static (value) types
- Cannot have a null reference (unless Nullable is used)
- Do not have memory overhead per new instance (unless “boxed”)

> [c# - Static readonly vs const - Stack Overflow](https://stackoverflow.com/questions/755685/static-readonly-vs-const)

## LINQ

### Where() vs FindAll()

FindAll() ≈ Where().ToList()

List            Any

FindAll() can only return List\<T>

> FindAll() is a function on the List\<T> type, it's not a LINQ extension method like Where. The LINQ extension methods work on any type that implements IEnumerable, whereas FindAll can only be used on List\<T> instances (or instances of classes that inherit from it, of course).
> Additionally, they differ in actual purpose. Where returns an instance of IEnumerable that is executed on-demand when the object is enumerated. FindAll returns a new List\<T> that contains the requested elements. FindAll is more like calling Where(...).ToList() on an instance of IEnumerable.

### string to int array

```csharp
var idlist = input.Split(',').Select(long.Parse).ToList();
```

> [c# - LINQ, Where() vs FindAll() - Stack Overflow](https://stackoverflow.com/questions/1938204/linq-where-vs-findall)

### convert-dataset-to-iqueryablet-or-ienumerablet

```csharp
table.AsEnumerable();
```

```csharp
table.AsEnumerable().AsQueryable();
```

> [linq - Convert Dataset to IQueryable\<T> or IEnumerable\<T> - Stack Overflow](https://stackoverflow.com/questions/505054/convert-dataset-to-iqueryablet-or-ienumerablet)

### Use Async with ForEach

```chsarp
var tasks = list.Select(i => DoSomething(i));
var values = await Task.WhenAll(tasks);
```

*C#8.0中有`await foreach`*

> [c# - How can I use Async with ForEach? - Stack Overflow](https://stackoverflow.com/questions/18667633/how-can-i-use-async-with-foreach)

## Thread

### is safe to call return in using block

totally safe, using block is just the try/catch block

```csharp
var scope = new TransactionScope())
try
{
    // my core logic
    return true; // if condition met else
    return false;
    scope.Complete();
}
finally
{
    if( scope != null)
        ((IDisposable)scope).Dispose();
}
```

better not keep reference

```csharp
using ( var x = new Something() ) {
    // not a good idea
    return x;
}
```

```csharp
Something y;
using (var x = new Something()) {
    y = x;
}
```

> [c# - returning in the middle of a using block - Stack Overflow](https://stackoverflow.com/questions/662773/returning-in-the-middle-of-a-using-block)
> [c# - Is it a good approach to call return inside using {} statement? - Stack Overflow](https://stackoverflow.com/questions/11776945/is-it-a-good-approach-to-call-return-inside-using-statement)

---
