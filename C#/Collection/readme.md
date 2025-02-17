# [Collection](https://learn.microsoft.com/en-us/dotnet/standard/collections/)

- Generic Collection (List<T>, Dictionary<Tkey, Tvalue>
- ReadOnlyCollection (ReadOnlyList, ReadOnlyDictionary)
  + Prevent modification (we cannot see the predefined method)
  + If we change the referenced collection -> the ReadOnlyCollection also change
- ImmutableCollection (ImmutableList, ImmutableDictionary)
  + Prevent changing the original collection
  + If add, modify an immutable collection -> instantinate a new collection
  + Cost at creation time
  + Underlying storage is binary tree -> better if we use foreach instead of for.
- From .NET 8, we have Frozen Dictionary: => Optimize for reading operation
- Concurrent Collections
  + Thread-safe collections
  + When use need to develop multi threading app (multi threads write/read operation)
  
## Indexable collection - IList

### List

- Internal array to store element -> fixed size
- Count/ Capacity properties
- Resize dynamically if the adding element > capacity
- [Performance consideration](https://learn.microsoft.com/en-us/dotnet/fundamentals/runtime-libraries/system-collections-generic-list%7Bt%7D#performance-considerations)
  + Contains, IndexOf, LastIndexOf, and Remove => use IEquatable<T> to compare
  + Sort, BinarySearch => use IComparable<T> to compare
  + Value Type should use IComparable/IEqualable to prevent boxing

### Queue
- FIFO
- Sequential access element

### Stack
- LIFO
- Sequential access element

### Linked List
- Sequential access element
- From Head to Tail
- From Tail to Head

## Key/Value pair collection - IDictionary
- Store in two arrays: one for buckets, one for value
- Hashing convert key to integer as index of buckets
- Capacity: resize if needed.
- Fast retrieval element by its key

# [How to choose a collection](https://learn.microsoft.com/en-us/dotnet/standard/collections/selecting-a-collection-class)
- Choose Generic Collection
- Do you need sequential list or access element in certain order?
- Access element by index or by its key
- Do you only need the collection for read only?
- Do you need the collection cannot be changed after intantiate it?

# [Code example](https://github.com/GiangHM/dotnet-collection-relearning)
