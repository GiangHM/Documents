#Multi Thread

![Multi_Thread](https://github.com/user-attachments/assets/3ffb1e9b-2487-4f5a-938e-5dad9c31310d)

- Multithreading programming is all about concurrent execution of different functions. Async programming is about non-blocking execution between functions.
-  Multithreading is about workers, Asynchronous is about tasks
-   Multithreading is one form of Asynchronous
- Thread
- Ways that .Net provide to implement an application under multithreaded
	- Thread: created manually by developers --> heavyweight --> very expensive, resource CPU
	- Thread pool ->  allocates threads and manage them for developers, some kinds of :
		- ThreadPool.QueueUserWorkItem
			- only receive one parameter -> use: lambda expression to bypass this limitation
			- cannot signal when all-thread is finished -> use: CountdownEvent 
		- BackgroundWorker
		- Asynchronous delegate: BeginInvoke, EndInvoke --> .Net core dosen't support 
		- TPL: task/ task factory -> developers handle their business, .Net manage creating thread... on their behalf  --> Lightweight, cause it's based on Thread pool concept

- Thread: background and foreground
	- Background thread: shut down app --> Background thread quit
	- Foreground thread: shut down app -> foreground thread still process, when foreground thread is finish --> app will shut down.
	- Thread pool -> always background thread

- Difference between Task and Thread
	- A task is more abstract than threads. It is always advised to use tasks instead of thread as it is created on the thread pool which has already system-created threads to improve the performance.
	- The task can return a result. There is no direct mechanism to return the result from a thread.
	- Task supports cancellation through the use of cancellation tokens. But Thread doesn't.
	- A task can have multiple processes happening at the same time. Threads can only have one task running at a time.
	- You can attach a task to the parent task, thus you can decide whether the parent or the child will exist first.
	- While using thread if we get the exception in the long-running method it is not possible to catch the exception in the parent function but the same can be easily caught if we are using tasks.
	- You can easily build chains of tasks. You can specify when a task should start after the previous task and you can specify if there should be a synchronization context switch. That gives you the great opportunity to run a long-running task in the background and after that a UI refreshing task on the UI thread.
		- A task is by default a background task. You cannot have a foreground task. On the other hand, a thread can be background or foreground.
		- The default TaskScheduler will use thread pooling, so some Tasks may not start until other pending Tasks have completed. If you use Thread directly, every use will start a new Thread.
- [Thread vs ThreadPool and Task](https://blog.slaks.net/2013-10-11/threads-vs-tasks/)

References:

Books: C# pro, C# multi thread, and parallel programming

Links:

 - https://medium.com/@letienthanh0212/asynchronous-and-parallel-programming-in-c-net-1e0f14e1db80
 - https://www.baeldung.com/cs/async-vs-multi-threading
 - https://medium.com/devtechblogs/overview-of-c-async-programming-with-thread-pools-and-task-parallel-library-7b18c9fc192d
