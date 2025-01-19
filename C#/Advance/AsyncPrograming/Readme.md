# Asynchronous programming

- What is asynchronous programming
- The Asynchronous programming pattern
- CPU bound and I/O bound?
	- What is it and how to tackle these problems
- ASP. Net API with asynchronous: relationship, benefit when we use async/await in the controller... [methods](https://docs.microsoft.com/en-us/archive/msdn-magazine/2014/october/async-programming-introduction-to-async-await-on-asp-net)

![Async_Illustration](https://github.com/user-attachments/assets/0daadbaf-94d4-496b-a67c-57c8b76c013a)

- Benefits of asynchronous programming:
	- I/O bound: database, internet resource...
		- For client app (window desktop, WPF,..) -> make application responsive( non-blocking UI).
		- For server app, such as Web API... -> make application scalable.
		- use async and await keywords, don't use Task.Run()
		- Don't use Task parallel library
	- CPU bound: intensive computation
		- use Task.Run() along with async and await if you need responsive
		- Use Task parallel library if you want 
- A task is a promise (it like: I will complete the work later)
	- Async/await => use current task
	- Task.Run() => separate new task to work
- Asynchronous and Thread
	- [There is no thread](https://blog.stephencleary.com/2013/11/there-is-no-thread.html)
- How to implement asynchronous -> There some patterns:
	- Event-based Asynchronous Pattern (EAP) -> .NET framework 2.0 -> not recommend for new development.
	- Asynchronous Programming Model (APM) -> not recommend for new development
	- Task-based asynchronous pattern -> [.NET framework 4.0](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)		- 
	- Async/ Await keyword -> .NET 4.5		- 
- Asynchronous programming with .Net framework
	- Async delegate: BeginInovke and EndInvoke method
	- Async with Async and await keyword (useful) => just take the thread from the thread pool when it meets  the following conditions: 
		- Improve performance
		- There is await keyword in the method	
		
Some useful links:
https://docs.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2012/hh191443(v=vs.110)?redirectedfrom=MSDN
https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async
