# Process vs. Thread

Threads represent multiple independent execution contexts within the same address space

![Process vs Thread](assets/P2L2/process_vs_thread.png)

A multithreaded process will have a more complex process control block structure, as these thread specific execution contexts needs to be incorporated.

# Benefits of Multithreading

1. Different threads can work in parallel on different components of the program's workload, which speeds up the program's execution.

   - For example, each thread may be processing a different component of the program's input.

2. Specialization

   If we designate certain threads to accomplish only certain tasks or certain types of task, we can take a specialized approach with how we choose to manage those threads. For instance, we can give higher priority to tasks that handle more important tasks or service higher paying customers.

   Performance of a thread depends on how much information can be stored in the processor cache. By having threads that are more specialized - that work on small subtasks within the main application - we can potentially have each thread keep it's entire state within the processor cache (hot cache), further enhancing the speed at the thread continuously performs it task.

![Why Thread](assets/P2L2/why_thread.png)

## Why not just write a multiprocess application?

A multiprocess application requires a new address space for each process, while a multithreaded application requires only one address space

As a result, a multithreaded application is more likely to fit in memory, and not requires as many swaps from disk, which is another performance improvement.

As well, passing data between processes - inter process communication (IPC) - is more costly than inter thread communication, which consists primarily of reading/writing shared variables.

# Benefits of Multithreading: Single CPU

Consider the situation where a single thread makes a disk request. The disk needs some amount of time to respond to the request, during which the thread cannot do anything useful. If the time that the thread spends waiting greatly exceeds the time it takes to context switch (twice), then it makes sense to switch over to a new thread.

Now, this is true for both processes and threads. One of the most time consuming parts of context switching for processes is setting up the virtual to physical mappings. Thankfully, when we are context switching with threads, we are using the same mappings because we are within the same process. This brings down the total time to context switch, which brings up the number of opportunities in which switching threads can be useful.

![Why Thread](assets/P2L2/thread_single_cpu.png)

# Benefits of Multithreading: Apps and OS Code

By multithreading the operating system kernel, we allow the operating system to support multiple execution contexts, which is particularly useful when we do have multiple CPUs, which allows the execution contexts to operate concurrently. The OS threads may run on behalf of different application or OS-level services like daemons and device drivers.

![os](assets/P2L2/os.png)

<br>

# Basic Thread Mechanisms

![what do we need to support threads](assets/P2L2/support_threads.png)

When processes run concurrently, they operate within their own address space

Threads share the same virtual to physical address mappings, since they share the same address space. Naturally, this can introduce some problems. For example, one thread can try to read the data while another modifies it, which can lead to inconsistencies. This is an example of a data race problem.

![problems](assets/P2L2/problems.png)

## mutual exclusion

A mechanism that allows threads to operate on data in an exclusive manner

Only one thread at a time is granted access to some data. The remaining threads must wait their turn

We accomplish mutual exclusion through the use of a **mutex**

![mutex](assets/P2L2/mutex.png)

## Condition Variable

Inter thread communication

A thread can wait on another thread, and to be able to exactly specify what condition the thread is waiting on

Both _mutexes_ and _condition variables_ are examples of **synchronization mechanisms**

# Thread Creation

We need some data structure to represent a thread

- Thread ID
- Program counter
- Stack pointer
- Register values
- Stack
- Other attributes

## fork

- not the UNIX fork call
- takes two arguments
  - proc (run when the thread is created)
  - args (pass to proc)

## join

- Return its result or communicate its status to the forking thread

- Ensure that a forking thread doesn't not exit before its forked thread completes work (as child threads exit when parent threads do)

- When **join** returns, the child thread exits the system and all resources associated with it are deallocated

![thread creation](assets/P2L2/thread_creation.png)

# Thread Creation Example

## data race

As we do not know which thread will run at which time in this example, we cannot be certain of the ordering of the elements in the list

![thread creation example](assets/P2L2/thread_creation_example.png)

# Mutexes

Many steps required to add an element to the list. Expected behavior is as following:

![update list expected](assets/P2L2/update_list_expected.png)

But there might be problem as data race. One example:

1. Thread A reads list and list.p_next
2. Thread B reads list and list.p_next
3. Thread A sets e.pointer to list.p_next
4. Thread B sets e.pointer to list.p_next
5. Thread A sets list.p_next to e
6. Thread B sets list.p_next to e

So one thread add successfully but another lost

![update list problem](assets/P2L2/update_list_problem.png)

# Mutual Exclusion