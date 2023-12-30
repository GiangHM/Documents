## I/ Clean code(readability/ naming convention/ documentation)

	1/ Are magic numbers explained? There should be no number in the code without at least a comment of why this is here. If the number is repetitive, is there a constant/enum or equivalent?
	2/ Instead of using raw strings, are constants used in the main class? Or if these strings are used across files/classes, is there a static class for the constants?
	3/ Documentation is in source code???
	4/ Comment is clear or we can rewrite the code to clean the comment?
	5/ New files, new variables, and new methods are named consistently and spelled correctly.
	6/ No complex boolean expressions?
	7/ No negative named booleans?
	8/ Null check to avoid null pointer exception in C#?
	
**II/ Troubleshoot(logging/ debugging/ exception and error handling)**

	1/ Handle exception correctly?
	2/  Are the error messages, if any, informative?
	3/ Is proper exception handling set up? Catching the exception base class(Exception) is generally not the right pattern. Instead, catch the specific exceptions that can happen(ex: IOException)
	4/ Code includes logs or traces?
	
**III/ Design and Principles (Solid/ Dry)**

	1/ DI injection correctly/ missing?
	2/ There is a duplication code
	3/ There is a hardcoded?
	4/ Some parts can be replaced by standard lib or third-party lib (nuget package?)
	5/ Can some parts be moved to a new util class?

**IV/ Threading and asynchronous code**

	1/ Does this code make correct use of asynchronous programming constructs, including proper use of await and Task.WhenAll including CancellationTokens?
	2/ Concurrency issues? Deadlock?
	3/ If a method is asynchronous, is Task.Delay used instead of Thread.Sleep?

**V/ Testing:**

	1/ AAA pattern for unit test?
	2/ There are unit tests provided to accommodate the changes.
	3/ The provided unit tests have a code coverage of at least 80%.
	4/ The tests follow the correct styling (MethodName_Should_ExpectedBehaviour_When_Input)
	5/ All tests pass locally on your machine
	6/ All tests pass on CI.

**VI/ Security:**

	1/ All personal data inputs are checked (for the correct type, length/size, format, and range).
	2/ Invalid parameter values are handled such that exceptions are not thrown (i.e; don't throw an exception if the user gives the wrong email and password combination).
	3/ No sensitive information is logged or visible in a stack trace.

**VII/ Performance**

	1/ Short-lived objects? How to optimize?
	2/ Boxing and unboxing operations?
	3/ Is the using pattern for streams and other disposable classes used? If not, better to have the Dispose method called explicitly
	4/ String Builder is used to concatenate large string



