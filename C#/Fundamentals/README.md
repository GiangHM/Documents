## String
- Equality: Compare Reference type with these operators: the '==' and '!='; compare object memory of references => to compare value, **USE**: string.equal
- Case-sensitive:  => use StringComparison Enumeration
  + Ex(Should be a codesnippet): s1.Equals(s2, StringComparison.OrdinalIgnoreCase);
- String is immutable: cannot change the value of a string, ==> modify, it means to create a new string and assign it to a variable
  + Disadvantages = performance => use StringBuilder
- Interpolation: $"i can eat {var1}";
## Narrowing and Widening Data Type Conversions -> for value type
- Widening: implicit conversion: smaller value type to larger value type
- Narrowing: explicit conversion: larger value type to smaller value type
## Boxing and unboxing:
- Demonstrate conversion from value type to reference type and otherwise.
## Method Parameter Modifiers
- ref
- out: no need to declare before using them (from c# 7)
- Value type and Reference type
  + assignment operator:
    * value type: like copy value, store to a new stack
    * reference type: redirecting what the reference variable points to in memory
- [passing to a method](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/passing-value-type-parameters)
  + reference type:
    * pass value: possible to change object's state data, cannot change object memory that reference point to
    * pass ref: possible to change object's state data, and possible to change object memory
  + value type
    * pass value: pass a copy of data, any change  in called method, do not affect original data
    * pass ref: called method can change value.
## Tuples:
- (string, int, string)  values = ('a', 1, 'b')
- use as a method return value
- discard
- [tuple type](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
- [tuple class](https://docs.microsoft.com/en-us/dotnet/api/system.tuple?view=netcore-3.1)

# Some useful interfaces
## IEnumerable, IEnumerator
- support foreach statement
- Make class usable with foreach statement https://docs.microsoft.com/en-us/troubleshoot/dotnet/csharp/make-class-foreach-statement
- Use case: create class can integrate with foreach

## ICloneable
- Give custom type the ability to expose an identical copy of itself
- Must implement Clone method of this interface
- Shallow copy -->  can use 'Object.MemberwiseClone'. If a field is a value type, a bit-by-bit copy of the field is performed. If a field is a reference type, the reference is copied but the referred object is not; therefore, the original object and its clone refer to the same object
- [Deep copy](https://docs.microsoft.com/en-us/dotnet/api/system.object.memberwiseclone?view=netcore-3.1)
++ Use case: want to copy object in some cases

## IComparable / IComparer
- Allow array can sort
- Use in some Linq method

## Difference between IEnumrable and List and Array
https://medium.com/@ben.k.muller/c-ienumerable-vs-list-and-array-9f099f157f4f
- IEnumerable is read-only and List is not.
- IEnumerable types have a method to get the next item in the collection. It doesn’t need the whole collection to be in memory and doesn’t know how many items are in it, foreach just keeps getting the next item until it runs out.
- List implements IEnumerable, but represents the entire collection in memory.
- LINQ expressions return an enumeration, and by default the expression executes when you iterate through it using a foreach, but you can force it to iterate sooner using .ToList() or .ToArray().
- Code example
```C#
 var names = new List<string> {"sam", "alexia", "simon", "sumanth", "tony", "sam", "amr", "mark", "drew"};
 // case 1
  var moreThanFiveLetters = names.Where(w => w.Length > 5);
 // case 2
 var moreThanFiveLetters_case2 = names.Where(w => w.Length > 5).ToList();
 names[0] = "benjamin";
 foreach (var name in moreThanFiveLetters)
 {
    Console.WriteLine("Case 1: {0}", name);
 }
 foreach (var name in moreThanFiveLetters_case2)
 {
   Console.WriteLine("case 2: {0}", name);
 }
```
Output
![image](https://github.com/user-attachments/assets/37d9208f-ef3d-45df-bbda-91fc1795f2ea)


## Difference between IEnumerable and IQueryable
- IEnumerable ->  query data from in-memory collections like List, Array, etc
- IEnumerable -> execute select query on server side, load data in memory client side, then filter data.
- IEnumrable -> Linq to object, xml
- IQueryable -> query data from out-memory (database, services)
- IQueryable -> execute select on server side with all filter
- IQueryable -> Linq to sql
