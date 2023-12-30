## I. Clean code(readability/ naming convention/ documentation)
- Are magic numbers explained? There should be no number in the code without at least a comment of why this is here. If the number is repetitive, is there a constant/enum or equivalent?
- Instead of using raw strings, are constants used in the main class? Or if these strings are used across files/classes, is there a static class for the constants?
- Documentation is in the source code???
- The comment is clear or we can rewrite the code to clean the comment?
- New files, new variables, and new methods are named consistently and spelled correctly.
- No complex boolean expressions?
- No negative named booleans?
- Null check to avoid null pointer exception in C#?
	
## II Troubleshoot(logging/ debugging/ exception and error handling)
- Handle exceptions correctly?
- Are the error messages, if any, informative?
- Is proper exception handling set up? Catching the exception base class(Exception) is generally not the right pattern. Instead, catch the specific exceptions that can happen(ex: IOException)
- Does the code include logs or traces?
	
## III. Design and Principles (Solid/ Dry)
- Does the DI injection correctly/ missing?
- There is a duplication code
- There is a hardcoded?
- Some parts can be replaced by standard lib or third-party lib (nuget package?)
- Can some parts be moved to a new util class?

## IV. Threading and asynchronous code
- Does this code make correct use of asynchronous programming constructs, including proper use of await and Task.WhenAll including CancellationTokens?
- Concurrency issues? Deadlock?
- If a method is asynchronous, is Task.Delay used instead of Thread.Sleep?

## V. Testing
- AAA pattern for a unit test?
- There are unit tests provided to accommodate the changes.
- The provided unit tests have a code coverage of at least 80%.
- The tests follow the correct styling (MethodName_Should_ExpectedBehaviour_When_Input)
- All tests pass locally on your machine
- All tests pass on CI.

## VI. Security
- All personal data inputs are checked (for the correct type, length/size, format, and range).
- Invalid parameter values are handled such that exceptions are not thrown (i.e; don't throw an exception if the user gives the wrong email and password combination).
- No sensitive information is logged or visible in a stack trace.

## VII. Performance
- Short-lived objects? How to optimize?
- Boxing and unboxing operations?
- Is the using pattern for streams and other disposable classes used? If not, better to have the Dispose method called explicitly
- String Builder is used to concatenate large string



