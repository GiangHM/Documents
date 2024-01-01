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
