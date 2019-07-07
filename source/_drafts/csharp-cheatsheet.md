---
title: csharp_cheatsheet
tags:
---

### LINQ, Where() vs FindAll()

FindAll() can only return List<T>

FindAll() is a function on the List<T> type, it's not a LINQ extension method like Where. The LINQ extension methods work on any type that implements IEnumerable, whereas FindAll can only be used on List<T> instances (or instances of classes that inherit from it, of course).
Additionally, they differ in actual purpose. Where returns an instance of IEnumerable that is executed on-demand when the object is enumerated. FindAll returns a new List<T> that contains the requested elements. FindAll is more like calling Where(...).ToList() on an instance of IEnumerable.

FindAll() ≈ Where().ToList()
List  			Any


### string to int array
var idlist = input.Split(',').Select(long.Parse).ToList();
> https://stackoverflow.com/questions/1938204/linq-where-vs-findall


### is safe to call return in using block
totally safe, using block is just the try/catch block
```
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
```
using ( var x = new Something() ) { 
  // not a good idea
  return x;
}
```
```
Something y;
using ( var x = new Something() ) {
  y = x;
}
```
> https://stackoverflow.com/questions/662773/returning-in-the-middle-of-a-using-block
> https://stackoverflow.com/questions/11776945/is-it-a-good-approach-to-call-return-inside-using-statement

