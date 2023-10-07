#计算机 #软件 #底层 

# Process Concept
## Program
> Informally, a process is a program in execution.

A program is just **a piece of code**.

Which code?
- High-level language code: C or C++?
- Low-level language code: assembly code?
- Not yet an executable: object code?
- Executable: machine code?

### Flow of building a program
$$
\text{C code}\xrightarrow{\text{Pre-processor}}\text{Expanded C code}\xrightarrow{\text{Compiler \& Optimizer}}\text{Assembly code}\xrightarrow{\text{Assembler}}\text{Object code}
$$
$$
\left.
\begin{matrix}
\text{Object code}\\
\text{Static/Dynamic library}
\end{matrix}
\right\}
\xrightarrow{\text{Linker}}\text{Executable}
$$
![[temp 1.svg|400]]![[temp 2.svg|400]]

#### Pre-processor
The pre-processor expands `#define`, `#include`, `#ifdef`, `#ifndef`, `#endif`, etc. 

Try `gcc -E hello.c`, it can show the expanded code.

- `#define`
	- raw code
		```c
		#define TXT "hello"

		int main(void)
		{
			printf("%s\n", TXT);
			return 0;
		}
		```
	- expanded code
		```c
		int main(void)
		{
		    printf("%s\n", "hello");
		    return 0;
		}
		```
- macro
	- raw code
		```c
		#define SWAP(a,b) { int c; c = a; a = b; b = c; }

		int main(void)
		{
			int i = 10, j = 20;
			printf("before swap: i = %d, j = %d\n", i, j);
			SWAP(i, j);
			printf("after swap: i = %d, j = %d\n", i, j);
		}
		```
	- expanded code
		```c
		int main(void)
		{
		    int i = 10, j = 20;
		    printf("before swap: i = %d, j = %d\n", i, j);
		    { int c; c = i; i = j; j = c; };
		    printf("after swap: i = %d, j = %d\n", i, j);
		}
		```
- `#include`
	- raw code
		- `include.c`
			```c
			#include "header.h"

			int main(void)
			{
			    add_fun(1,2);
			    return 0;
			}
			```
		- `header.h`
			```c
			int add_fun(int a, int b)
			{
			    return (a + b);
			}
			```
	- expanded code
		```c
		int add_fun(int a, int b)
		{
		    return (a + b);
		}
		
		int main(void)
		{
		    add_fun(1,2);
		    return 0;
		}
		```

#### Compiler and Optimizer
- The compiler performs
	- Syntax checking and analyzing
	- If there is no syntax error, construct intermediate codes, i.e., <u>assembly codes</u>
- The optimizer optimizes codes
	- **It improves stupid codes!**
	- Check the parameter of gcc
	>The parameter `-O` means to optimize. The number followed is the optimization level. Max is level 3, i.e., `-O3`. Default is level is `-O1`. `-O0` means no optimization.
	>
	>For example, `gcc -O3 hello.c`

#### Assembler and Linker
- The assembler assembles `hello.s` and generates an object code `hello.o`
	- A step closer to machine code
	- Try `as hello.s -o hello.o`
- The **linker** puts together all abject files as well as the libraries
	- There are two kinds of libraries
		- statically-linked
		- dynamically-linked

#### Library files
- just a bunch of function implementations.
- for the linker to look for the function(s) that the target C program needs.

![[temp 3.svg|500]]
![[temp 4.svg|500]]

#### How to compile multiple files?
- `gcc` by default hides all the intermediate steps.
	- *Executable*: `gcc -o hello hello.c` generates `hello` directly.
	- *Object code*: `gcc -c hello.c` generates `hello.o` directly.
- Working with multiple files
	1. Prepare all the source files. 
	
		**Important**: there must be one and only one file containing the `main` function
	2. Compile them into object codes one by one, i.e., `gcc -c *.c`
	3. Construct the program together with all the object codes, i.e., `gcc -o program *.o`

### Conclusion
A program is an **executable** file.
- It is static
- It may be associated with dynamically-linked files
	- `*.so` in Linux and `*.dll` in Windows
- It may be compiled from more than one file

## Process
A process is a program in execution
- A program (an executable file) becomes process when it is **loaded into memory**
- **Active**
### Process in Memory
![[Pasted image 20220313134433.png|inlineR|200]]
- Text section
	- Program code
- Data section
	- Global variables
- Heap
	- Dynamically allocated memory during process run time
- Stack
	- Temporary data
		- function parameters
		- return addresses
		- local variables
- Program counter
- Contents of registers
### Process State
As a process executes, it changes **state**, which is defined in part by the **current activity**
- States
	- **new**: The process is being created.
	- **running**: Instructions are being executed
	- **waiting**: The process is waiting for some event to occur
		- I/O completion
		- Reception of a signal
	- **ready**: The process is waiting to be assigned to a processor
	- **terminated**: The process has finished execution
- State diagram
	![[Pasted image 20220313144513.png|400]]
- Only one process can be running on any processor at any instant
- Many processes may be ready or waiting

### Processes in PCB
Each process is represented in the operating system by a **process control block (PCB, also called task control block)**

![[Pasted image 20220313144854.png|inlineR|250]]
- Locate / Represent a process
	- Process state
	- Program counter
		- location of next instruction to execute
	- CPU registers
		- contents of all process-centric registers
	- CPU scheduling information
		- priorities
		- scheduling queue pointers
	- Memory-management information
		- memory allocated to the process
	- I/O status information
		- I/O devices all allocated to process
		- list of open files
	- Accounting information
		- CPU used
		- clock time elapsed since start
		- time limits
- Process switch
	![[Pasted image 20220313145851.png|600]]

### Process Data
- Process data structure in Linux
	- Represented by C structure `task_struct` in `<linux/sched.h>`. Here are some members in the structure
	```c
	pid t_pid;                  /* process identifier */
	long state;                 /* state of the process */
	struct sched_entity se;     /* scheduling information */
	struct task_struct *parent; /* this process's parent */
	struct list_head children;  /* this process's children */
	struct files_struct *files; /* list of open files */
	struct mm_struct *mm;       /* address space of this process */
	```
	![[Pasted image 20220313150441.png|400]] ^60cf32
- Relationship between process data and PCB
	![[temp 6.svg|500]]

### Conclusion
- A process is a program in execution
	- $\text{process (active entity)}\neq\text{program (static entity)}$
	- **active**
		- A program counter specifying the next instruction to execute
		- A set of associated resources
- Only one process can be running on any processor at any instant
- Two processes maybe associated with the same program (Two users are running the same program)
	- Example
		- The same user invokes two copies of the web browser
	- Separate execution sequences
		- The text section may be equivalent
		- The data, heap, and stack sections vary
- A process can be an execution environment for other code
	- Example: Java programming environment - JVM (Java Virtual Machine)

# Processes Operations (Programmer's Perspective)
- Process
	- It associates with <u>all the files opened</u> by that process.
	- It attaches to all the memory that is allocated for it.
	- It contains every <u>accounting information</u>
		- running time
		- current memory usage
		- process owner
		- etc.
- System must provide mechanisms for:
	- process **identification**
	- process **creation**
	- program **execution**
	- process **termination**
- Some *basic* and *important* system calls
	- `getpid()`
	- `fork()`
	- `exec*()`
	- `wait()`
	- `exit()`
## Process Identification
- Each process is given an unique ID number, and is called the **process ID**, or the **PID**.
- The system call, `getpid()`, prints the PID of the calling process.

```c
#include <stdio.h>  // printf()
#include <unistd.h> // getpid()

int main()
{
	printf("My PID is %d\n", getpid());
}
```
Run the program in shell be like:
```plaintext
> ./test
My PID is 124
> ./test
My PID is 130
> ./test
My PID is 136
```
## Process Creation
### Process Blossoming
- A process may create several new processes
	- **Parent** process: the creating process
	- **Children** processes: the new processes
- The first process
	- The kernel, while it is booting up, creates the first process `init`
	- `init` process
		- `PID` = 1
		- is running the program code `/sbin/init`
		```plaintext
		$ ps -ef
		UID          PID    PPID  C STIME TTY          TIME CMD
		root           1       0  0 xxxxx ?        00:00:01 /sbin/init
		....
		```
		- Its first task is to **create more processes**
- *Tree hierarchy*
	- Each of the new process may in turn create other processes, and form a tree hierarchy![[temp 7.svg|600]]
	- You can view the tree with the command `pstree` or `pstree -A` (ASCII-character-only display)
		```plaintext
		$ pstree
		init─┬─init───init───zsh───pstree
		     ├─init───init───zsh─┬─bash
		     │                   └─zsh
		     └─2*[{init}]
		```
- Orphan process
	- All the resources are deallocated to OS when a process terminates
	- A process may become an orphan when its parent terminated
	- An orphan turns the hierarchy from a **tree** into a **forest**
	- Plus, no one would know the termination of the orphan.
- Orphan re-parent ^e64097
	- In Linux, we have the **re-parent operation**. The `init` process will become the step-mother of all orphans.
	- Windows maintains a *forest-like* hierarchy.
- **Summary**
	- Observation
		1. The processes in Linux is always organized as a tree. Because of the re parent operation, there is always **only one process tree**.
		2. The re parent operation allows processes running **without the need of a parent terminal**. Thus, the **background jobs** survive even though the hosting terminal is closed.
- Parent-child relationship
	- Resource sharing options
		- Parent and children share all resources
		- Children share subset of parent's resources
		- Parent and child share no resources
	- Execution options
		- Parent and children execute concurrently
		- Parent waits until children terminate
	- Address space options
		- Child is a duplicate of parent
		- Child has a new program loaded into it
### UNIX example `fork()`
- To create a process, we use the system call `fork()`

![[temp 8.svg|500]]

#### Example 1
```c
#include <stdio.h>	// printf()
#include <unistd.h> // getpid()

int main(void)
{
	printf("Ready (PID = %d)\n", getpid());
	fork();
	// This line (below) will be executed twice.
	printf("My PID is %d\n", getpid()); 
	return 0;
}
```

```plaintext
$ ./fork_example_1
Ready (PID = 386)
My PID is 386            # the original process(parent process).
My PID is 387            # created by fork(), the child process
```

- Both the parent and the child execute **the same program before and after `fork()`**
- The child process starts its execution **at the location that `fork()` is returned**, *not from the beginning of the program*.

#### Example 2
- Assumption
	- <u>Only one process</u> is allowed to be executed at one time.
	- However, we can’t predict which process will be chosen by the OS, which is the **process scheduling** mechanism.

```c
#include <stdio.h>	 // printf()
#include <unistd.h> // getpid()

int main(void)
{
	int result;
	printf("before fork ...\n");
	result = fork();
	printf("result = %d\n", result);

	if (result == 0)
	{
		printf("I'm the child process.\n");
		printf("My PID is %d\n", getpid());
	}
	else
	{
		printf("I'm the parent process.\n");
		printf("My PID is %d\n", getpid());
	}

	printf("Program terminated.\n");
}
```
- For parent, the return value of `fork()` is the PID of the created child.
- For child, the return value of `fork()` is 0.
```plaintext
$ ./fork_example_2
before fork ...
result = 674             # parent running, child waiting
I'm the parent process.
My PID is 673            # parent PID
Program terminated.      # parent dead
result = 0               # child running, parent dead
I'm the child process.
My PID is 674            # child PID
Program terminated.      # child dead
```

#### `fork()` behavior
- `fork()` behaves like *cell division*
	- It creates the child process by **cloning** from the parent process, including
		
|            Cloned items             |                                            Description                                            |
|:-----------------------------------:|:------------------------------------------------------------------------------------------------- |
|   Program code \[File & Memory\]    |                             They are sharing the same piece of code.                              |
|               Memory                |          Including local variables, global variables, and dynamically allocated memory.           |
| Opened files  \[Kernel's internal\] | If the parent has opened a file *A*, then the child will also have file *A* opened automatically. |
|  Program counter \[CPU registers\]  |          That’s why they both execute from the same line of code after `fork()` returns.          |
	
- `fork()` does not clone the following...
	
|      Distinct items      |      Parent      |                     Child                     |
|:------------------------:|:----------------:|:---------------------------------------------:|
| Return value of `fork()` | PID of the child |                       0                       |
|           PID            |    Unchanged     | Different, not necessarily "*parent PID + 1*" |
|      Parent process      |    Unchanged     |     The parent process (calling `fork()`)     |
|       Running time       |    Cumulated     |                0, just created                | 
		
- Note: they are all data inside the memory of kernel.
## Program Execution
### `exec()` system call
- `fork()` can only duplicate
	- If a process can only duplicate itself and always runs the same program, how can we execute other programs?
- `exec()` system call family.
  - `int execl(const char* path, const char* arg, …)`
  - `int execlp(const char* file, const char* arg, …)`
  - `int execle(const char* path, const char* arg, …, char* const envp[])`
  - `int execv(const char* path, const char* argv[])`
  - `int execvp(const char* file, const char* argv[])`
  - `int execvpe(const char* file, const char* argv[], char *const envp[])`
- `execl()` a member of the `exec` system call family.
	- Arguments
		1. The path of file to be executed.
		2. First argument to the program. (The name of the program)
		3. More arguments to the program. (Optional)
		4. `NULL`, indicating the end of the list of arguments.
	- The suffix `l` stands for list.
	- Example
		1. `execl("/bin/ls", "ls", NULL);`
		2. `execl("/bin/ls", "ls", "-l", NULL);`

#### Example
```c
#include <stdio.h>
#include <unistd.h>

int main()
{
	printf("before execl ...\n");
	execl("/bin/ls", "ls", NULL);
	// The code below won't be reached
	printf("after execl ...\n");
	return 0;
}
```

```shell
> ./execl_example 
before execl ...
execl_example  execl_example.c
```

1. The code after `execl` **is not reached**
2. The process is **terminated** after `execl`

#### behavior
- The `exec` system call family is not simply a function that "invokes" a command.
	- In the example above, the procedure is
		1. The `execl()` call changes the execution from `exec_example` to `/bin/ls`
		2. The execution of `/bin/ls` begins.
		3. The `exit()` or `return` statement in `/bin/ls` will terminate the process.
		- Therefore, it is certain that the process cannot go back to the old program!

#### Observation
- The process is changing the code that is executing and **never returns to the original code**.
	- The last two lines of codes are therefore not executed.
- The process that calls any one of the member of the `exec` system call family will **throw away** many things, e.g.,
	- Memory ^4e95e1
		- local variables
		- global variables
		- dynamically allocated memory
	- register value
		- the program counter
- The process will **preserve** something, including:
	- PID
	- Process relationship
	- Running time
	- etc.

### `fork()` + `exec*()`
- The mix can become
	- A shell
	- The `system()` library call
	- etc.

![[temp 9.svg|400]]

#### Example
```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int system_test(const char *cmd_str)
{
	if(cmd_str == NULL) 
		return -1;
	if(fork() == 0)
	{
		execl(cmd_str, cmd_str, NULL);
		// The code below will be executed when meet an error
		fprintf(stderr, "%d: command not found\n", cmd_str);
		exit(-1);
	}
	return 0;
}

int main()
{
	printf("before ...\n\n");
	system_test("/bin/ls");
	printf("\nafter ...\n");
	return 0;
}
```

#### Results
If we execute the example multiple times, there will be more than one output.
1. Expected
	```plaintext
	before ...

	system_implement_1  system_implement_1.c 

	after ...
	```
2. Another situation
	```plaintext
	before ...
	
	
	after ...
	system_implement_1  system_implement_1.c     
	```

![[temp 10.svg|500]]

`fork() + exec*() != system()`

### `wait()`
#### `fork()` + `exec*()` + `wait()` = `system()`
- **Suspend** the execution of the parent process
- **Wake** the parent up after the child is terminated

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int system_test(const char *cmd_str)
{
	if(cmd_str == NULL) 
		return -1;
	if(fork() == 0)
	{
		execl(cmd_str, cmd_str, NULL);
		// The code below will be executed when meet an error
		fprintf(stderr, "%d: command not found\n", cmd_str);
		exit(-1);
	}
	wait(NULL); // The parent is suspended until the child terminates
	return 0;
}

int main()
{
	printf("before ...\n\n");
	system_test("/bin/ls");
	printf("\nafter ...\n");
	return 0;
}
```

```plaintext
before ...
											# fork()
system_implement_1  system_implement_1.c
											# suspend & wake up
after ...
```

#### `wait()`
- Two cases 
	- The `wait()` system call **suspend** the calling parent process (Case 1). 
	
		`wait()` returns and wakes up the calling process when the one of its child processes changes from **running to terminated**.
	- `wait()` does not suspend the calling process (Case 2) if  ^5ed1ce
		- There were no running children 
		- There were no children
- Limitation
	- Waits for any one of the children
	- Detect child termination only

#### `waitpid()`
|               `wait()`               |                                                                 `waitpid()`                                                                 |
|:------------------------------------:|:-------------------------------------------------------------------------------------------------------------------------------------------:|
| Wait for **any one** of the children |                               Depending on the parameters, `waitpid()` will wait for a particular child only.                               |
|  Detect **child termination** only.  | Depending on the parameters, `waitpid()` can detect child’s status changing, either from running to suspended or from suspended to running. | 

> For more details, you <u>must read</u> the man-pages of [`wait()`](https://man7.org/linux/man-pages/man3/wait.3p.html) and [`waitpid()`](https://man7.org/linux/man-pages/man3/waitpid.3p.html)

## Process Termination
`exit()` system call.


# Process operation (Kernel's Perspective)
## Kernel & User
### Kernel-space VS User-space
|                   | Kernel-space memory                                          | User-space memory                               |
|:-----------------:|:------------------------------------------------------------:|:-----------------------------------------------:|
| Storing what?     | Kernel data structure </br> Kernel code </br> Device drivers | Process' memory</br>Program code of the process |
| Assigned by whom? | Kernel code                                                  | User program code + kernel code                 | 

![[temp 11.svg|inlineR|120]]

- A process will switch its execution from user space to kernel space through **invoking system call**.
- Example
	- The CPU is running a program code of a process.
	- The code is in the **user-space memory**, be like [[#^60cf32]]
	- When the process is calling a system call, say, `getpid()`
		- We need to get the PID from **PCB (in kernel-space memory)**
		- The CPU switches **from the user-space to the kernel-space**, and reads the PID
	- After having finished executing the system call, i.e., `getpid()`, the CPU **switches back to the user-space memory**, and continues running that program code

### User Mode & Kernel Mode
Remember This?
[[chap-1-overview-of-an-operating-system#Dual-mode Operation|Dual-mode]].

![[Pasted image 20220302221734.png]]

How much time was spent in each part?

### User time VS System time
The **execution of a process** is also divided into two parts
- User time
- System time

![[temp 12.svg|600]]

You can use the `time` command to count the time elapsed (user time, system time, total time)
```bash
# called in bash, there is another implementation in zsh and other shells
> time program 

real    xmx.xxxs
user    xmx.xxxs
sys     xmx.xxxs
```

#### Example 1
```c
// time_example_1
#include <stdio.h>

int main(void)
{
    int x = 0;
    for (int i = 1; i <= 10000000; i++)
    {
        x = x + i;
        // printf("x = %d\n", x);
    }
    return 0;
}
// -------------------------------------------
// time_example_2
#include <stdio.h>

int main(void)
{
    int x = 0;
    for (int i = 1; i <= 10000000; i++)
    {
        x = x + i;
        printf("x = %d\n", x);
    }
    return 0;
}
```

Both were compiled through 
```shell
> gcc -O0 -o time_example_1 time_example_1.c
> gcc -O0 -o time_example_2 time_example_2.c
```

Use the `time` command (implemented in zsh using a bash-like format, might varies in different shell). Because print the characters into the shell is very time taking, we should print them into somewhere else.

```shell
> time ./time_example_1

real    0.03s
user    0.03s
sys     0.00s
> time (./time_example_2 > ./output) # echo character is time-taking !

real    2.17s
user    0.99s
sys     1.18s
```

> Accessing hardware costs the process more time.

#### Example 2
```c
// time_example_3
#include <stdio.h>

#define MAX 10000000

int main(void)
{
	int i;
	for (i = 0; i < MAX; i++)
		printf("x\n");
	return 0;
}
// -------------------------------------------
// time_example_4
#include <stdio.h>

#define MAX 10000000

int main(void)
{
	int i;
	for (i = 0; i < MAX / 5; i++)
		printf("x\nx\nx\nx\nx\n");
	return 0;
}
```

Testing ...

```shell
> time (./time_example_3 > ./output)

real    0.27s
user    0.12s
sys     0.15s
> time (./time_example_4 > ./output)

real    0.05s
user    0.02s
sys     0.02s
```

> When writing a program, you must consider both the user time and the system time

#### Short Summary
- The user time and the system time together define the **performance** of an application
	- System call plays a major role in **performance**.
	- **Blocking system call**: some system calls even <u>stop your process</u> until the data is available.
- Programmers should pay attention to system performance, for example, read a file block-by-block rather than read it byte-by-byte.

## System Calls from the Kernel's View
Inside kernel, processes are arranged as a **double linked list**, called the task list. Each node is a entity of [[#^60cf32|task_struct]], containing (they are all in kernel-space)
- PID
- Running time
- Array of opened files

| Array Index |                    Description                    |
|:-----------:|:-------------------------------------------------:|
|      0      |     Standard Input Stream</br>`FILE *stdin;`      |
|      1      |     Standard Output Stream</br>`FILE *stdout`     |
| 3 or beyond | Files you opened, e.g., `fopen()`, `open()`, etc. |

- List of children
- Pointer to parent
- Signal handler
- etc.

In the user-space, there're
- Local variable
- Dynamically-allocated memory
- Global variable
- Code + constants
- Registers and program counter
- etc.

### `fork()` in action
After the parent process invokes `fork()`, the operating system copy the `task_struct` (node in task list)  of the parent, creating a child node in the task list. Then the nodes get to be updated.
#### Kernel-space update
- Child node
	- **PID** is updated
	- **Running time** is reset to 0
	- **Pointer to parent** is set to the parent node
	- Array of opened file is **preserved**! 
		> The child process share a set of opened files. That's why a parent process **shares the same terminal output stream** as the child process!
- Parent node
	- **List of children** is added the new child.
		- The child process was add to the task list.
#### User-space update
The parent process data in the user-space is copied to the child process.

#### Finish
The parent process and child process are ready to return from `fork()`, but their return values are different.
- Parent: The PID of the new child
- Child: 0

### `exec*()` in action
![[temp 13.svg|500]]
#### Inside the kernel
The kernel searches the target program file.
- If it is not found, the process returns from the system call.
- If it can be found, the kernel code updates the content on the user-space memory.

#### User-space
What happens to the user-space memory?
- **Local variables** are cleared
- **Dynamically-allocated memory** is cleared
- **Global variables** are reset based on the new code
- **Code and constants** are changed to the new program code
- **Registers' values** are reset

### `wait()` + `exit()`
#### `wait()`
- `wait()` system call
	- suspend the parent process
	- wake up when one child process terminates
		- through `exit()` system call
- `wait()` and `exit()` come together

#### Time analysis
![[temp 14.svg|600]]

- Child side: `exit()`
- Parent side: `wait()`

#### Child side
The child invokes `exit()`, then the kernel takes the operation below (step-by-step)
1. The kernel frees all the allocated memory. E.g., the list of opened files are all closed.
2. The kernel **removes everything on the user-space memory** about the process, including program code and allocated memory.
3. The kernel **remains** the entry of the child in the process table (terminated state).
	- The child is now called **zombie**
	- Its storage in the kernel-space memory is kept to a minimum.
	- The **PID** and **process structure** are owned by the child
4. The kernel notifies the <u>parent of the child process</u> about the termination of its child by a **signal** called `SIGCHLD`

##### Signal
- What is signal?
	- A software interrupt
	- It takes steps as in the hardware interrupt
- Two kinds of signals
	- Generated from user-space
		- `Ctrl+C`
		- `kill()` system call
		- etc.
	- Generated from kernel and CPU
		- Segmentation fault `SIGSEGV`
		- Floating point exception `SIGFPE`
		- Child process termination `SIGCHLD`
		- etc.
- Signal is very hard to master, will be skipped in this course. Reference:
	- [Advanced Programming Environment in UNIX](http://www.apuebook.com/)
	- [Linux man-page](https://www.kernel.org/doc/man-pages/)

##### Summary
![[temp 15.svg|600]]
- Steps
	1. Clean up most of the allocated kernel-space memory.
	2. Clean up all user-space memory.
	3. Notify the parent with SIGCHLD.
- Although the child is still in the system, it is no longer running. There is no program code. It turns into a *mindless zombie*.

> You cannot kill a zombie process, as it is already dead. Then how to eliminate it?

#### Parent side
- The kernel sets a **signal handling routine** (and it is a **function pointer**) to the process.
	- The signal handling routine will be executed when `SIGCHLD` comes
	- By default, every process does not respond to the `SIGCHLD` signal (the signal handlers are set only when `wait()` is called).

1. **The wait() system call blocks the process** until …
2. When `SIGCHLD` comes, the signal handling routine is invoked.
	- since the parent is still inside the system call, instead of <u>the original program code</u>, the parent process is still blocked in some sense…
3. **Default Handling of SIGCHLD**
	1. Accept and remove the `SIGCHLD`
	2. Destroy the child process that sends her the signal.
		- So, the child will be given a clean death by the `wait()` system call.
4. The signal handler is then removed, i.e., the process is ignoring `SIGCHLD` again. It returns to the previously-executing code, going back to the user space.
	- it looks like **`wait()` is returned from its invocation**.
	
	> This is the reason why `wait()` system call waits for any one of the child processes.
5. The return value of `wait()` system call is the PID of the terminated child.

##### [[#^5ed1ce|case 2]]
![[temp 16.svg|600]]

1. Child was already terminated (became a zombie), `SIGCHLD` is also sent to parent before
2. The kernel sets the signal handling routine. The `wait()` system call finds that the `SIGCHLD` signal is already there. Default actions are then taken immediately.
3. Return from `wait()`

#### Orphans (zombies)
- What would happen if a parent did not invoke `wait()` and terminated?
	- [[#^e64097|Re-parent operation]]
	- `init` is the new parent, and it **periodically** invokes `wait()`

#### Short summary
- `wait()` is to reap zombie child processes
	- You should never leave any zombies in the system.
- Linux will label zombie processes as `<defunct>`.
	- Try `ps aux | grep defunct`

#### Example
- This program requires you to type `enter` twice before the process terminates.
- You are expected to see **the status of the child process** changes between the 1st and the 2nd `enter`.
```c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
	int pid;
	if ((pid = fork()))
	{
		printf("Look at the status of the process %d\n", pid);
		while (getchar() != '\n');
		wait(NULL);
		printf("Look again!\n");
		while (getchar() != '\n');
	}
	return 0;
}
```

First check
```plaintext
UID        PID  PPID  C STIME TTY          TIME CMD
xxxxxx     320     9  0 18:48 pts/0    00:00:00 ./test
xxxxxx     321   320  0 18:48 pts/0    00:00:00 [test] <defunct>
```

Second check
```plaintext
UID        PID  PPID  C STIME TTY          TIME CMD
xxxxxx     320     9  0 18:48 pts/0    00:00:00 ./test
```

### The role of `wait()`
- Why calling `wait()` is important
	- It is not about process execution/suspension…
	- It is about **system resource management**
- Think about it
	- A zombie takes up a PID
	- The total number of PIDs are limited
		- Try `cat /proc/sys/kernel/pid_max`
		```shell
		> cat /proc/sys/kernel/pid_max
		32768
		```
	- **What will happen if we don’t clean up the zombies?**
- A infinite zombie factory
	- Don't try to run it!
		 ```c
		int main(void) {
			while (fork());
			return 0;
		}
		```
	- Parent: never reach `return 0`.
	- Child: `return 0` reached immediately, but no corresponding `wait()` for the parent (ZOMBIE)