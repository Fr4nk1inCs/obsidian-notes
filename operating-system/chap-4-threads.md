#计算机 #软件 #底层 

# Multi-threading
## Motivation
### Application side
Most software applications are multi-threaded, each application is implemented as a process with several threads.
- Web browser
	- display image
	- retrieve data from network
- Text editor
	- display graphics
	- respond to keyboards
	- spelling & grammar checking
- Similar tasks in a single application (web browser)
	- Accept client requests, service the requests
	- Usually serve thousands of clients
	- ![[temp 17.svg|500]]

The reason why not create a process for each task
- Process creation is
	- Heavy weighted
	- Resource intensive
- **Many of the data can be shared between multiple tasks within an application**

### System side
Modern computers usually contain multicores
- But, each processor can **run only one process at a time**
- CPU is not fully utilized

Improve the efficiency
- Assign one task to each core
- Real parallelism (not just concurrency with interleaving on single core system)

![[temp 18.svg|400]]

## Thread Concept
### High-level Idea
![[temp 19.svg|500]]
### Internals
![[temp 20.svg|inlineR|200]]
- Code
	- All threads **share** the same code.
	- A thread starts with **one specific function**.
		- Named **thread function**
		- Functions A & B in the diagram
	- The thread function can invoke other functions or system calls
	- But, a thread could <u>never return to the caller of the thread function</u>
- Dynamically-allocated memory & Global variable
	- All threads **share** the same <u>global variable zone</u> and the same <u>dynamically allocated memory</u>
	- All threads can read from and write to both areas
- Local variables
	- Each thread has **its own memory range** for the local variables
	- So, the stack is the private zone for each stack

### Benefits
- Responsiveness and multi-tasking
	- Multi-threading design allows an application to do **parallel tasks simultaneously**
	- Example: Although a thread is blocking, the process can still depend on another thread to do other things.
	- Especially important for interactive applications (user interface)

> It’d be nice to assign **one thread for one blocking system/library call**

- Ease in data sharing
	- global variable
	- dynamically-allocated memory
- Processes share resources via shared memory or message passing, which must be explicitly arranged by the programmer

> This leads to the **mutual exclusion** & the **synchronization** problems 

- Economic
	- Allocating memory and resources for process creation is costly, dozens of times slower than creating threads
	- Context switch between processes is also costly, several times of slower
- Scalability
	- Threads may be running in parallel on different cores

### Programming Challenge
- Identifying tasks
	- Divide separate and concurrent tasks
- Balance
	- Tasks should perform equal work of equal value
- Data splitting
	- Data must be divided to run on separate cores
- Data dependency
	- Synchronization is needed
- Testing and debugging

## Thread Models
Thread should also include
- Data/resources in user space memory
- Structure in kernel

How to provide thread support?
- User thread
	- Implement in user space
- Kernel thread
	- Supported and managed by kernel

Thread models (relationship between user/kernel)
- Many-to-one
- One-to-one
- Many-to-many
### Many-to-One Model
![[temp 21.svg|inlineR|200]]
All the threads are mapped to one process structure in the kernel.
- Merit
	- Easy for kernel to implement
- Drawback
	- When a blocking system call is called, all the threads will be blocked
- Example
	- Old UNIX & green thread in some programming languages.

### One-to-One Model
![[temp 22.svg|inlineR|200]]
Each thread is mapped to a process or a thread structure
- Merit
	- Calling blocking system calls only block those calling threads
	- **A high degree of concurrency**
- Drawback
	- Cannot create too many threads as it is restricted by the size of the kernel memory
- Example
	- Linux
	- Windows

### One-to-More Model
![[temp 23.svg|inlineR|200]]
Multiple threads are mapped to multiple structures (group mapping)
- Merit:
	- Create as many threads as necessary
	- Also have a high degree of concurrency
# Programming
## Thread Libraries
- A thread library provides the programmer with an **API** for creating and managing threads
	- Two ways of implementation: User-level or Kernel-level
- Three main thread libraries
	- POSIX Pthreads (user-level or kernel-level)
	- Windows (kernel-level)
	- Java (implemented using Windows API or Pthreads)
- Multiple threads creating
	- Asynchronous threading
		- Parent resumes execution after creating a child
		- Parent and child execute **concurrently**
		- Each thread runs independently
			- **Little data share**
	- Synchronous threading
		- Fork join strategy: Parent waits for children to terminate
			- **Significant data sharing**

## The Pthreads Library
- **Pthreads**: POSIX standard defining an API for thread creation and synchronization.
	- Specification, not implementation

|         Operation          |         Process         |          Thread          |
|:--------------------------:|:-----------------------:|:------------------------:|
|          Creation          |        `fork()`         |    `pthread_create()`    |
|         I.D. Type          |    `PID`, an integer    | `pthread_t`, a structure |
|         Who am I?          |       `getpid()`        |     `pthread_self()`     |
|        Termination         |        `exit()`         |     `pthread_exit()`     |
| Wait for child termination | `wait()` or `waitpid()` |     `pthread_join()`     |
|            Kill            |        `kill()`         |     `pthread_kill()`     |


### `pthread_create()`
```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
// Thread function
void * hello(void *input)
{
	printf("%s\n", (char *) input);
	pthread_exit(NULL);
}
// Main function
int main()
{
	pthread_t tid;
	pthread_create(&tid, NULL, hello, "hello world");
	pthread_join(tid, NULL);
	return 0;
}
```

> The source code with `pthread` functions should be compiled with `-pthread` option!

```shell
> gcc -O2 -pthread pthread_create.c -o pthread_create
> ./pthread_create
hello world
```

### Passing parameter
```c
#include <pthread.h>
#include <stdlib.h>
#include <stdio.h>
// Thread function
void * do_your_job (void *input)
{
	printf("child = %d\n", *((int *)input));
	*((int *)input) = 20;
	printf("child = %d\n", *((int *)input));
	pthread_exit(NULL);
}
// Main function
int main()
{
	pthread_t tid;
	int input = 10;
	printf("main = %d\n", input);
	pthread_create(&tid, NULL, do_your_job, &input);
	pthread_join(tid, NULL);
	printf("main = %d\n", input);
	return 0;
}
```
```shell
> gcc -pthread -O2 passing_parameter.c -o passing_parameter
> ./passing_parameter
main = 10
child = 10
child = 20
main = 20    # the input in main() was changed!
```
- The local variable `input` is in <u>the stack for the `main()` thread</u>
- The stack for the new thread is <u>on the same piece of user space memory as the `main()` thread</u>
- The `pthread_create()` function only passes an **address** to the new thread, <u>pointing to a variable in the stack of the `main()` function</u>
- Therefore, the new thread can change the value in the main thread, and <u>vice versa</u>

### Multiple threads
```c
#include <pthread.h>
#include <stdlib.h>
#include <stdio.h>
// Thread function
void * do_your_job (void *input)
{
	int id = *((int *)input);
	printf("My ID number = %d\n", id);
	pthread_exit(NULL);
}
// Main function
int main()
{
	int i;
	int temp[5];
	pthread_t tid[5];
	// creating several threads
	for (int i = 0; i < 5; ++i)
	{
		temp[i] = i;
		pthread_create(&tid[i], NULL, do_your_job, &temp[i]);
	}
	// waiting on several theards
	for (int j = 0; j < 5; ++j)
		pthread_join(tid[j], NULL);
	return 0;
}
```
One possible output:
```shell
> ./multiple_threads
My ID number = 1
My ID number = 3
My ID number = 0
My ID number = 4
My ID number = 2
```
### Passing return value
```c
#include <pthread.h>
#include <stdlib.h>
#include <stdio.h>
#include <math.h>
#include <time.h>
// Thread function
void * do_your_job (void *input)
{
	int *output = (int *)malloc(sizeof(int));
	srand(time(NULL));
	*output = ((rand() % 10) + 1) * (*((int *)input));
	pthread_exit(output); // line 12
}
// Main function
int main()
{
	pthread_t tid;
	int input = 10;
	int *output;
	pthread_create(&tid, NULL, do_your_job, &input);
	pthread_join(tid, (void **) &output); // line 21
	printf("Output of child is %d\n", *output); 
	free(output);
	return 0;
}
```
One possible output
```shell
> ./passing_return_value
Output of child is 80
```

- `void pthread_exit(void *return_value);`
	- Together with termination, a pointer to **a global variable or a piece of dynamically allocated memory** is returned to the main thread. (line 12)
	- Using pass by reference, a pointer to the result is received in the main thread. (line 21)

## Implicit Threading
- Applications are containing hundreds or even thousands of threads
	- Program correctness is more difficult with explicit threads
- How to address the programming difficulties?
	- Transfer the creation and management of threading from programmers to compilers and run time libraries
	- Implicit threading
- Introduce two methods
	- Thread Pools
	- OpenMP

### Thread Pools
- Problems with multithreaded servers
	- Time required to create threads, which will be discarded once completed their work
	- Unlimited threads could exhaust the system resources
- Thread pool
	- Idea
		- Create a number of threads in a pool where they wait for work
	- Procedure
		- Awakens a thread if necessary
		- Returns to the pool after completion
		- Waits until one become free if the pool contains no available thread
- Advantages
	- Usually slightly faster to service a request with an existing thread than create a new thread
	- Allows the number of threads in the application(s) to be bound to the size of the pool

### OpenMP
- Provides support for parallel programming in shared memory environments
- Set of compiler directives and an API for C, C++, FORTRAN
- Identifies parallel regions - blocks of code that can run in parallel
	- When OpenMP encounters the directive, it creates as many threads as there are processing cores

```c
#include <omp.h>
#include <stdio.h>

int main()
{
	// parallel region
	#pragma omp parallel
	{
		printf("I am a parallel region.\n");
	}
	return 0;
}
```
> compile with `-fopenmp` option!
 - parallel for loop
```c
#include <omp.h>
#include <stdio.h>

int main()
{
	#pragma omp parallel for
	for (int i = 0; i < 10; ++i)
		printf("%d", i);
	return 0;
}
```
One possible output
```shell
> gcc -fopenmp openMP.c -o openMP
> ./openMP
7238014965
```

## Threading Issues
### Semantics of `fork()` and `exec*()`
- `fork()`: Some UNIX systems have two versions
	- The new process duplicates all threads, or
	- Duplicates only the thread that invoked `fork()`
- `exec*()`: usually works as normal
	- Replace the running process - including **all threads**

### Signal Handling
- **Signals** are used in UNIX systems to notify a process that a particular event has occurred
	- Synchronous signal and asynchronous signal
	- Default handler or user defined handler
- Where should a signal be delivered in multi threaded program?
	- Deliver the signal to the thread to which the signal applies
	- Deliver the signal to every thread in the process
	- Deliver the signal to certain threads in the process
	- Assign a specific thread to receive all signals for the process
- Deliver a signal to a specified thread with Pthread
	- `pthread_kill(pthread_t tid, int signal)`

### Thread Cancelling
- Terminating a thread before it has finished
- Two general approaches
	- **Asynchronous cancellation** terminates the target thread immediately
		- Problem: Troublesome when canceling a thread which is updating data shared by other threads
	- **Deferred cancellation** allows the target thread to periodically check if it should be cancelled (can be canceled safely)
- Pthreads
	- `pthread_cancel(pthread_t tid)`
		- Indicates only a request
	- Three cancelation modes
		- can be changed by 
			- `int pthread_setcancelstate(int state, int *oldstate);`
				- States: `PTHREAD_CANCEL_ENABLE` & `PTHREAD_CANCEL_DISABLE`
			- `int pthread_setcanceltype(int type, int *oldtype);`
				- Types: `PTHREAD_CANCEL_DEFERRED` & `PTHREAD_CANCEL_ASYNCHRONOUS`
		- Default: deferred
			- Cancelation occurs only when it reaches a cancelation point, can be established by `pthread_testcancel()`

		  		
|     Mode     |  State   |     Type     |
|:------------:|:--------:|:------------:|
|     Off      | Disabled |              | 
|   Deferred   | Enabled  |   Deferred   |
| Asynchronous | Enabled  | Asynchronous |
### Thread-Local Storage
- Some applications, each thread may need its own copy of certain data
	- Transaction processing system: service each transaction (with a unique identifier) in a thread
	- Local variables visible only during a single function invocation
- Thread local storage (TLS) allows each thread to have its own copy of data
	- TLS is **visible across function invocations**
	- Similar to `static` data
	- TLS data are unique to each thread