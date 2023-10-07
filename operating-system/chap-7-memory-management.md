#计算机 #底层 #软件 


# Why we need memory management?
- The running program code requires memory
	- The CPU needs to fetch the instructions from the memory for execution
- We must keep several processes in memory
	- Improve both CPU utilization and responsiveness
	- Multi-programming

```ad-important
It is required to efficiently manage the memory.
```

# User-Space Memory Management
## Address Space
How does a programmer look at the memory space?
- An array of bytes?
- Memory of a process is divided into segments
- This way of arranging memory is called **segmentation**
  
```ad-question
title: What is the process address space?
~~~c
#include <stdlib.h>
#include <stdio.h>

int global_int;

int main(void) {
	int *malloc_ptr = malloc(4);
	char *constant_ptr = "hello";
	
	printf("local variable  = %15p\n", &malloc_ptr);
	printf("malloc() space  = %15p\n", malloc_ptr);
	printf("global variable = %15p\n", &global_int);
	printf("code & constant = %15p\n", constant_ptr);

	return 0;
}
~~~
~~~shell
$ ./addr
local variable  =  0x7ffe3c404418
malloc() space  =  0x557ab98242a0
global variable =  0x557ab93fb014
code & constant =  0x557ab93f9004
$
~~~
~~~ad-info
title: Note
The addresses are not necessarily the same in different processes
~~~
![[Pasted image 20220418142333.png]]
```
```ad-question
title: How large is the address space?
In a 32-bit system,
- One address maps to one byte.
- The maximum amount of memory in a process is **4GB**.

~~~ad-note
- This is the so called logical address space 
- **Each process has its own address space**, and it can reside in any part of the physical memory
~~~
```

## Program Code & Constants
- A program is an executable file
- A process is **not bounded to one program code**
	- The `exec*()` family
- The program code requires memory space because
	- The CPU needs to fetch the instructions from the memory for execution.

```ad-example
~~~c
#include <stdio.h>

int main(void) {
	char *string = "hello";
	printf("\"hello\"        = %p\n", "hello");
	printf("string pointer = %p\n", string);
	string[4] = '\0';
	printf("Go to %s\n", string);
	return 0;
}
~~~
~~~~ad-question
title: Question #1: What are the printouts from Line 3 & 4?
~~~plaintext
"hello"        = 0x5621f47c0004
string pointer = 0x5621f47c0004
~~~
~~~~
~~~~ad-question
title: What is the printout from Line 6?
~~~plaintext
[1]    xxxx segmentation fault (core dumped) 
~~~
~~~~
```
- Constants are stored in code segment.
	- The memory for constants is decided by the program code
	- Accessing of constants are done using addresses (or pointers).
- Code and constants are both **read-only**.

## Data Segment & BSS
### Properties
````ad-example
```c
int global_int = 10;
int main(void) {
	int local_int = 10;
	static int static_int = 10;
	printf("local_int  addr = %p\n", &local_int);
	printf("static_int addr = %p\n", &static_int);
	printf("global_int addr = %p\n", &global_int);
	return 0;
}
```
```shell
$ ./global_va_static 
local_int  addr = 0x7ffd4a3b9964
static_int addr = 0x55f8d2a12014
global_int addr = 0x55f8d2a12010
$
```
- `static_int` and `global_int` are stored next to each other.
- This implies that they are **in the same segment**!
```ad-info
title: Note
A static variable is treated as the same as a global variable!
```
````

- Data
	- Containing **initialized** global and static variables.
- BSS (Block Started by Symbol)
	- Containing **uninitialized** global and static variables.

### Locations
````ad-example
```c
int global_bss;
int global_data = 10;
int main(void) {
	static int static_bss;
	static int static_data = 10;
	printf("BSS:\n");
	printf("\tglobal bss = %p\n", &global_bss );
	printf("\tstatic bss = %p\n", &static_bss );
	printf("Data:\n");
	printf("\tglobal data = %p\n", &global_data );
	printf("\tstatic data = %p\n", &static_data );
}
```
```shell
$ ./data_vs_bss     
BSS:
	global bss = 0x56417897101c
	static bss = 0x564178971020
Data:
	global data = 0x564178971010
	static data = 0x564178971014
$
```
BSS has larger address.
````

### Sizes
````ad-example
title: data
`data_large.c`
```c
char a[1000000] = {10};
int main(void) {
	return 0;
}
```
`data_small.c`
```c
char a[100] = {10};
int main(void) {
	return 0;
}
```
Compile with no optimization
```shell
$ gcc -O0 data_large.c -o data_large
$ gcc -O0 data_small.c -o data_small
$ ls -l data_small data_large
-rwxrwxr-x 1 xxx xxx 1015816  xxx data_large
-rwxrwxr-x 1 xxx xxx   15920  xxx data_small
$
```
```ad-note
The data segment has the required space already allocated.
```
````
````ad-example
title: BSS
`bss_large.c`
```c
char a[1000000];
int main(void) {
	return 0;
}
```
`bss_small.c`
```c
char a[100];
int main(void) {
	return 0;
}
```
Compile with no optimization
```shell
$ gcc -O0 bss_large.c -o bss_large
$ gcc -O0 bss_small.c -o bss_small
$ ls -l bss_small bss_large  
-rwxrwxr-x 1 xxx xxx 15800  xxx bss_large
-rwxrwxr-x 1 xxx xxx 15800  xxx bss_small
$
```
```ad-note
- To the program, BSS is just a bunch of symbols. **The space is not yet allocated**.
- The space will be allocated to the process once it starts executing.
- This is why BSS is called *Block Started by Symbol*.
```
````
### Limits
````ad-question
title: How large is the data segment?
In Linux, `ulimit` is a bulit-in command in `/bin/bash`. It sets or gets the system limitations in the current shell.
```shell
$ ulimit -a
...
core file size              (blocks, -c) 0
data seg size               (kbytes, -d) unlimited
...
$
```
```ad-question
Does the "unlimited” mean that you can define a global array with large enough size?
```
````
````ad-example
`global_1gb.c`
```c
#include <stdio.h>
#include <string.h>
#define ONE_MEG (1024 * 1024)

char a[1024 * ONE_MEG];

int main(void) {
	memset(a, 0, sizeof(a));
	printf("1GB OK\n");
}
```
`global_2gb.c`
```c
#include <stdio.h>
#include <string.h>
#define ONE_MEG (1024 * 1024)

char a[2048 * ONE_MEG];

int main(void) {
	memset(a, 0, sizeof(a));
	printf("2GB OK\n");
}
```
```shell
$ gcc -Wall -O0 global_1gb.c -o global_1gb
$ gcc -Wall -O0 global_2gb.c -o global_2gb
global_2gb.c:5:13: warning: integer overflow in expression of type ‘int’ results in ‘-2147483648’ [-Woverflow]
    5 | char a[2048 * ONE_MEG];
      |             ^
global_2gb.c:5:6: error: size of array ‘a’ is negative
    5 | char a[2048 * ONE_MEG];
      | 
$ 
```
- The size of an array is a 32-bit **signed integer**, no matter 32-bit or 64-bit systems.
````
````ad-example
`global_4gb.c`
```c
#include <stdio.h>
#include <string.h>
#define ONE_MEG (1024 * 1024)

char a[1024 * ONE_MEG];
char b[1024 * ONE_MEG];
char c[1024 * ONE_MEG];
char d[1024 * ONE_MEG];

int main(void) {
	memset(a, 0, sizeof(a));
	printf("1GB OK\n");
	memset(b, 0, sizeof(b));
	printf("2GB OK\n");
	memset(c, 0, sizeof(c));
	printf("3GB OK\n");
	memset(d, 0, sizeof(d));
	printf("4GB OK\n");
}
```
Compiling
```shell
$ gcc -m32 -O0 global_4gb.c -o global_4gb               
$ ./global_4gb
[1]    xxx segmentation fault (core dumped)  ./global_4gb
```
- On a **32-bit** Linux system, the user-space **addressing space is around 3GB**. 
- The kernel reserves 1GB addressing space.
````
### Summary
- Remember, `global variable == static variable`
	- Only the **compiler cares** about the difference!
- Everything in a computer has a limit!
	- Different systems have different limits: 32-bit VS 64-bit.
	- Your job is to adapt to such limits.
	- On a **32-bit** Linux system, the user-space **addressing space is around 3GB**. 

## Stack
### Properties
- The stack contains
	- all the local variables
	- all function parameters
	- program arguments
	- environment variables.
```ad-question
How are the data stored and what is the size limit?
```
- Stack: FILO
- When a function is called, the local variables are **allocated** in the stack.
- When a function returns, the local variables are **de-allocated** from the stack.

### Function Call, Push & Pop Mechanisms

### Limits
```shell
$ ulimit -a
...
stack size                  (kbytes, -s) 8192
...
$
```
The limit is 8MB. You can't define a local array larger than the limit.
```c
#include <stdio.h>
#include <string.h>
#define ONE_MEG (1024 * 1024)
int main(void) {
    char a[8 * ONE_MEG];
    memset(a, 0, sizeof(a));
    printf("8MB OK\n");
}
```
```shell
$ gcc -O0 local_8mb.c -o local_8mb   
$ ./local_8mb 
[1]    xxx segmentation fault (core dumped)  ./local_8mb
```

### Summary
```ad-question
title: What if it is a chain of endless recursive function calls?
- **Exception caught by the CPU**
	- **Stack overflow exception**!
- **Program Terminated**!
- Workaround
	- Minimize the number of arguments
	- Minimize the number of local variables
	- Minimize the number of calls
	- Use global variables
```
```ad-attention
A function can ask the CPU **to read and to write anywhere in the stack**, **not just the *zone* belonging to the running function**!
```

## Heap
### Properties
- The dynamically allocated memory is called the **heap**.
	- Don’t mix it up with the binary heap
	- It has nothing to do with the binary heap
- **Dynamic**: not defined at compile time
- **Allocation**: only when you ask for memory, you would be allocated the memory.
- Lecturers of a programming course would tell you the following:
	- `malloc()` is a function that allocates memory for you
	- `free()` is a function that gives up a piece of memory that is produced by previous `malloc()` call.
- The lecturer of the OS course is **to define** and **to defy** what you know about the `malloc()` and `free()` library functions.
### `malloc()`
1. When a program just starts running, the entire heap space is unallocated, or empty.
2. When `malloc()` is called, the `brk()` system call is invoked accordingly. 
	- `brk()` allocates the space required by `malloc()`. 
	- But, it doesn’t care how `malloc()` uses the space.
3. The allocated space growing or shrinking depends on the further actions of the process. 
	- That means the `brk()` system call can **grow or shrink** the allocated area.
	- In `malloc()`, the library call just invoke `brk()` for growing the heap space.
	- The `free()` call may shrink the heap space.

````ad-example
title: `malloc()`
- The return value of `malloc()` is of type `void *`, which means it is just a memory address only, and can be of any data types.
- Such a memory address is the starting address of a piece of requested memory
```c
int main(void) {
	char *ptr1, *ptr2;
	ptr1 = (char *)malloc(16);
	ptr2 = (char *)malloc(16);

	printf("Distance between ptr1 and ptr2: %d bytes\n", ptr2 - ptr1);
	return 0;
}
```
This figure below shows the memory structure in the heap of the program:
![[Pasted image 20220424191127.png]]

So the result should be >= 16. The result:
```shell
$ gcc malloc.c -o malloc
$ ./malloc
Distance between ptr1 and ptr2: 32 bytes
```
````
```ad-note
title: The data structure created by `malloc()`
![[Pasted image 20220424223751.png]]
So the result of the example above is `32` (16+16)
```
### `free()`
- `free()` *seems to* be the opposite to `malloc()`
	- It de-allocates any allocated memory.
	- When a program calls `free(ptr)`, then the address `ptr` <u>must be the start of a piece of memory obtained by a previous `malloc()` call</u>.

```ad-example
title: Case #1: de-allocating the last block.
This is accomplished by calling `brk()` system call. This heap has become smaller.
![[Pasted image 20220424192454.png]]
```

```ad-example
title: Case #2: de-allocating an intermediate block.
- We have to keep the de-allocated blocks because they cannot be returned to the system.
- As the number of de-allocated blocks cannot be known in prior, we need a **linked list**.
	- The `Head` variable is a pointer acting as the **start of the list of the free blocks**.
	- `NULL` defines the end of the free list.

![[Pasted image 20220424192318.png]]
```

```ad-example
title: Case #2: Another Example
![[Pasted image 20220424224234.png]]
```

```ad-caution
title: Cautions
- The calling program is **assumed** to be carefully written.
	- After `malloc()` has been invoked, the program should read and write inside the requested area only.
		- Now, you know why you’d **have troubles** when you write data outside the allocated space.
		- ![[Pasted image 20220424224559.png]]
	- When `free()` is called, the program should provide `free()` with the correct address…
		- i.e., the address previously returned by a malloc() call.
		- ![[Pasted image 20220424224628.png]]
```

### When `malloc()` meets free blocks …
```ad-question
title: Whether to use the free blocks or not?
Is there any free block that is **large enough** to satisfy the need of the `malloc()` call?
```

```ad-example
title: Case #1: If there is <u>no suitable free block</u>
The `malloc()` function should call `brk()` system call in order to claim more heap space.
![[Pasted image 20220424225038.png]]
```
```ad-example
title: Case #2: If there is a suitable free block
The `malloc()` function should reuse that free block.
![[Pasted image 20220424225415.png]]
```
- Other cases
	- A `malloc()` request that takes a partial block
	- A `malloc()` request that takes a partial block, but leaving no space in the previously free block
- We will skip those subtle cases…
	- It boils to implementation only...
	- You already have the **big picture** about `malloc()` and `free()`.

### Some Implementations
#### Implicit Free List
- Needs two information for each block
	- size
	- is_allocated

![[Pasted image 20220424231320.png#inl|150]]![[Pasted image 20220424231357.png#inl|White: free | Grey: Allocated | Dark Grey: Allocated & unused |600]]

- Allocation
	- May need linear time search
		- **First fit**: allocate the first hole that is big enough (fast)
		- **Next fit**: similar to first fit, but start where previous search finishes
		- **Best fit**: allocate the smallest hole that is big enough (helps fragmentation, larger search time)
		- **Worst fit**: allocate the largest hole
	- Allocate the whole block or splitting
- Free: Coalescing
	- Coalescing with next block: easy![[Pasted image 20220424232954.png]]
	- How about coalescing with previous block?
		- Add a boundary tag in the footer![[Pasted image 20220424233056.png]]
		- 4 Cases![[Pasted image 20220424233114.png]]

- Summary
	- May not be used in practical `malloc()` and `free()` implementations
		- High memory allocation cost
	- Some ideas are still useful and important
		- Splitting available blocks
		- Boundary tag

#### Explicit Free List
![[Pasted image 20220424233313.png]]
- Track only free blocks (LIFO or address-ordered)
- Block splitting is useful in allocation
- Boundary tag is still useful in coalescing

#### Segregated Free List
![[Pasted image 20220424233435.png]]
- Different free lists for different size classes
- Allocation
	- Search appropriate list (larger size)
	- Found and split
	- Not found: search next
	- **Approximates best-fit**

```ad-example
title: Special Example: Buddy System (power-of-two block size)
![[Pasted image 20220424233540.png]]
```

### Issues
Issues raised by `malloc()` and `free()`
- The kernel knows how much memory should be given to the heap.
	- When you call `brk()`, the kernel **tries** to find the memory for you
- Then…one natural question…
	- Is it possible to **run out of memory** (**OOM**)?

### Out of Memory
````ad-example
title: OOM Generator
```c
#include <stdio.h>
#include <stdlib.h>
#define ONE_MEG 1024 * 1024

int main(void) {
	void *ptr;
	int counter = 0;
	while(1) {
		ptr = malloc(ONE_MEG);
		if(!ptr)
			break;
		counter++;
		printf("Allocated %d MB\n", counter);
	}
	return 0;
}
```
Compile & run the program on a 32-bit Linux machine
```shell
$ gcc -m32 -O0 oom_gen.c -o oom_gen
$ ./oom_gen
...
Allocated 3052 MB
Allocated 3053 MB
Allocated 3054 MB
Allocated 3055 MB
```
(Sometimes the output stop at `3054 MB`)

It runs very fast.
```` 

^a59584

```ad-question
title: On 32-bit Linux, why does the OOM generator stop at around 3055MB?
- Every 32-bit Linux system has an addressable memory space of 4G-1 bytes.
- The kernel reserves 1GB addressing space.
```
````ad-example
title: Another OOM Generator
```c
#include <stdio.h>
#include <stdlib.h>
#define ONE_MEG 1024 * 1024

char global[1024 * ONE_MEG];
int main(void) {
	void *ptr;
	int counter = 0;
	char local[8000 * 1024];
	while(1) {
		ptr = malloc(ONE_MEG);
		if(!ptr)
			break;
		counter++;
		printf("Allocated %d MB\n", counter);
	}
	return 0;
}
```
Compile & run the program on a 32-bit Linux machine
```shell
$ gcc -m32 -O0 oom_gen.c -o oom_gen
$ ./oom_gen
...
Allocated 3044 MB
Allocated 3045 MB
Allocated 3046 MB
Allocated 3047 MB
```
````
````ad-example
title: Real OOM
```ad-warning
1. Don’t run this program on any department’s machines.
2. Don’t run this program when you have important tasks running at the same time.
```
```c
#include <stdio.h>
#include <stdlib.h>
#define ONE_MEG 1024 * 1024

int main(void) {
	void *ptr;
	int counter = 0;
	while(1) {
		ptr = malloc(ONE_MEG);
		if(!ptr)
			break;
		memset(ptr, 0, ONE_MEG);
		counter++;
		printf("Allocated %d MB\n", counter);
	}
	return 0;
}
```
Compile & run the program on a 32-bit Linux machine
```shell
$ gcc -m32 -O0 oom_gen.c -o oom_gen
$ ./oom_gen
...
Allocated 3052 MB
Allocated 3053 MB
Allocated 3054 MB
Allocated 3055 MB
```
The program runs very slowly.
````

^be568d

```ad-important
title: Other Issues
- External fragmentation
	- The heap memory looks like a map with many holes
	- It is the source of inefficiency because of the **unavoidable search for suitable space**
	- The memory wasted because **external fragmentation is inevitable**
- Internal fragmentation![[Pasted image 20220425132942.png|inlineR|450]]
	- Payload is smaller than allocated block size
	- Padding for alignment
	- Placement policy
		- Allocate a big block for small request
```

## Segmentation Fault
```ad-question
title: What is segmentation fault?
- Someone must have told you:
	- When you are accessing a piece of memory that is not allowed to be accessed, then the OS returns you an error called - segmentation fault.
- The memory in a process is separated into **segments**.
	- So, when you visit a segment in an **illegal way**, then **segmentation fault**
```
![[Pasted image 20220425133455.png#center|Segments]]

```ad-summary
title: Summary
When you have a **so-called address** (maybe it is just a random sequence of 4 bytes), one of the following cases happens:

|         |   Read-only Segments   | Allocated Segments | Unused or Unallocated Segments |
|:-------:|:----------------------:|:------------------:|:------------------------------:|
| Reading |       No Problem       |     No Problem     |     **Segmentation Fault**     |
| Writing | **Segmentation Fault** |     No Problem     |     **Segmentation Fault**     | 

- Now, you know what is a segmentation fault, and the cause is always **carelessness**!
	- Now, you know why **free()** sometimes give you segmentation fault
		- because you **corrupt the list of free blocks**!
	- Now, you know why `malloc()`-ing a space that is smaller than required is ok
		- because **you are overwriting the neighboring blocks**!
```

## Summary
![[Pasted image 20220425134238.png|inlineR|150]]
- Memory of a process is divided into segments (segmentation)
	- codes and constants;
	- global and static variables
	- allocated memory (or heap)
	- local variables (or stack)
- When you access a memory that is not allowed, then the OS returns you **segmentation fault**
- **Every process’ segments are independent and distinct.**
- The dynamically allocated memory is not as simple as you learned before.
	- Allocating large memory blocks is not efficient; instead, **allocating small memory blocks** can make use of the **holes** in the heap memory efficiently.
	- **Keep calling `malloc()` without calling `free()` is dangerous**
		- because there is no garbage collector in C or the OS
		- **OOM error**

# Kernel’s Perspective: Virtual Memory Support
## Virtual Memory
```ad-info
title: CPU Working: Fetch-Decode-Execute Cycle
![[Pasted image 20220502052428.png]]
```
````ad-example
title: Virtual Memory Example
```c
#include <stdlib.h>
#include <stdio.h>

int main(void)
{       
    int pid;
    pid = fork();
    printf("PID %d: %p.\n", getpid(), &pid);
    if (pid)
        wait(NULL);
    return 0;
}     
```
```shell
$ ./same_addr                             
PID 15450: 0x7fff8c9978c4.
PID 15451: 0x7fff8c9978c4.
```
````
- Can you guess the result?
	- Two **different processes**, the **same variable name**, carry **different values**
	- Use the **same address**!
- Well, what is the meaning of a memory address?!
	- Logical address: **virtual memory**
	- Address translation needed (logical/virtual->physical)
	- Why we use virtual memory??

```ad-info
title: CPU Working: Discontiguous Allocation
- If allocation is contiguous …
	- Each process is contained in a single section of memory
	- **Problem #1**: Does a process always have a chance to grow to reach its need?
	- **Problem #2**: What if we have a process that is larger that the physical memory?
		- What the CPU (or OS) can do is to give up running …
- So, we need to have the CPU design that can understand processes so that:
	1. The address space is no longer required to be contiguous.
	2. It allows a process to have a size beyond the physical memory.
```
```ad-note
title: Virtual Memory Support in Modern CPU
The new design of the CPU includes a new module: the **memory management unit** (**MMU**).
- MMU is designed to perform address translation.
- The MMU is an **on-CPU** device.

![[Pasted image 20220502141550.png]]
```
```ad-note
title: Virtual Memory Workflow
1. When CPU wants to fetch an instruction, the **virtual address** is sent to MMU and is translated into a **physical address**.
2. The memory returns the instruction addressed in physical address
3. The CPU decodes the instruction.
	- An instruction **always stores virtual addresses**, but not physical addresses.
4. With the help of the MMU, the target memory is retrieved.
```
```ad-note
title: Merits
1. Different processes use the same virtual addresses, they may be **translated to different physical addresses**.
	- The address translation helps the CPU to **retrieve data in a discontiguous layout** (the process address space is contiguous).
2. **Memory sharing** can be implemented.
	- This is how threads share memory!
	- This is how different processes share codes!
3. **Memory growth** can be implemented.
	- When the memory of a process grows, the newly allocated memory is not required to be contiguous
```

## MMU Implementation
- How to implement the MMU?
	- How to efficiently translate from virtual address to physical address?
	- Translation is needed for every process

### Translation Table
```ad-question
title: Can translation be done by a ***lookup table***?
Remember, every process needs its own lookup table.

The answer is no.
```
```ad-example
title: The Size of Lookup Table
Say, we has an 4G memory.
- How many address are there?
	- $2^{32}$
- How large is an address?
	- 4 Bytes
- We only need a physical address table since the index is the virtual memory address.
- The size of lookup table is $2^{32}\times 4\text{ Bytes}=16\text{ GBytes}$
```
So a single lookup table is larger than the memory itself, and each process needs one look up table.
- This is definitely not acceptable.

```ad-question
title: Can we reduce the table size?
Note that every address in a CPU is always of 4 bytes. The only choice is to reduce the number of addresses.
```
```ad-note
title: Partial Lookup Table
![[Pasted image 20220502144702.png]]
```
## Paging
The partial lookup table technique is called **paging**.
- This partitions the memory into fixed blocks called **pages**.
- The lookup table inside the MMU is now called the **page table**.

```ad-note
title: Properties
![[Pasted image 20220502145647.png]]
- Adjacent virtual pages are not guaranteed to be mapped to adjacent physical pages.
	- **Contiguous virtual addresses** map to **non-contiguous physical address**.
	- Virtual addresses within the same page are always mapped to the same physical page.
```
### Memory Allocation
````ad-question
title: How to do memory allocation with paging?
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

char *prev_ptr = NULL;
char *ptr      = NULL;

void handler(int sig)
{
    printf("Page size = %d Bytes\n", (int)(ptr - prev_ptr));
    exit(0);
}

int main(void)
{
    char c;
    signal(SIGSEGV, handler);
    prev_ptr = ptr = sbrk(0); // find the heap's start
    sbrk(1);                  // increase heap by 1 Byte?
    while (1)
        c = *(++ptr);
}
```
```shell
$ ./allocation
Page size = 4096 Bytes
```
````
- A page is the basic unit of memory allocation.
	- The allocation is in a **page-by-page** manner.
	- The same case for the growth of the stack.

![[Pasted image 20220502151446.png|450]]

```ad-caution
title: Problem
- The minimum allocation unit is 4,096 bytes.
- But, the process cannot use that much.
- So, the rest of the page is unused.
```
### Internal Fragmentation
```ad-info
title: Internal Fragmentation
**Internal fragmentation** means space is avoidably wasted when allocation is done in a page-by-page manner.
```
```ad-question
title: How about letting another process to use the "unused space"?
The MMU has to memorize that none of the processes could occupy the whole page. **The growth of the usage has to be limited and monitored**!
```

### Putting It Together
![[Pasted image 20220502152106.png]]

### Page Table Design
```ad-question
title: The next waves of questions
- Who can tell which virtual page is allocated?
- Who can tell which page is on which device?

Those questions can be answered by the design of the page table.
```
```ad-question
title: How to design the page table?
1. First of all, which information need to be maintained?
	- Mapping from virtual pages to physical pages (called frames)
	- Permission information
	- Where is the page (in memory or not)
2. Second
	- Each process needs one page table
```
````ad-note
title: Page Table
```ad-example
title: Page Table of Process A
| Virtual Page Number | Permission | Valid-invalid bit | Frame Number |
|:-------------------:|:----------:|:-----------------:|:------------:|
|          A          |   `rwx-`   |         1         |      0       |
|          B          |   `NIL`    |         0         |    `NIL`     |
|          C          |   `r--s`   |         1         |      2       |
|          D          |   `NIL`    |         0         |    `NIL`     |
|          …          |     …      |         …         |      …       | 

- The second row means the virtual page "A" is mapped to the physical frame "0"
- The row with `NIL` means the virtual page is **not allocated**.
```
- Virtual Page Number
	- For the sake of convenience, we don’t use addresses here.
	- This column is **not stored in the page table**.
- Permission
	- `r`: readable
	- `w`: writable
	- `x`: executable
	- `s`: sharable
- Valid-invalid bit
	- This bit is to tell the CPU whether **this row is valid or not**.
	- If the row is invalid, it means that **the virtual page is not in the memory**.
		- This is not the same as an unallocated page.
- Frame Number
	- The physical memory is just **an array of frames**. The size of a frame is 4KB.
````
```ad-question
title: How does the CPU check if you can write to a memory zone?
The situation below results in segmentation fault:
- When a virtual address is translated to an **unallocated frame**…
- When you write to **read-only pages**…
- When you try to execute a nonexecutable pages…
```
```ad-question
title: Other Design Issues
- How to store the page table if it is large?
	- Page Table Structure
- How to improve memory access performance (page table look incurs large overhead)? 
	- Caching: Translation lookaside buffer (TLB)
```

### Page Table Structure
- The page table may be large…multiple MBs
	- We would not want to allocate the page table contiguously in memory, how?
	- Divide the page table into pieces

```ad-note
title: Hierarchical Paging
- Two-level page table
	- logical address![[Pasted image 20220502161716.png|300]]
	- ![[Pasted image 20220502161656.png|500]]
- Hierarchical Paging
![[Pasted image 20220502161741.png|350]]
```
```ad-info
Besides hierarchical paging, we can also use
- Hashed page tables
- Inverted page tables
```
### Performance Boost
Memory access requires to look up page table
- This overhead is even larger with multi-level page tables
- Any solution?

```ad-note
title: Large Pages
![[Pasted image 20220502164024.png|inlineR|400]]
- Reduce the page table entries
- Cons?
	- Internal fragmentation
	- Deduplication
```

```ad-note
title: Caching
![[Pasted image 20220502164652.png|inlineR|400]]
- **Translation lookaside buffer** (**TLB**)
	- The search in TLB is fast: Part of the instruction pipeline
	- The size of TLB is small: e.g., 32-1024 entries
- Effective memory-access time
	- example
		- Hit ratio: 80%
		- Memory access time: 100ns
		- One memory access for page table lookup
		- Effective mem-access time is $0.8\times 100+0.2\times(100+100)=120\text{ ns}$
```

### Summary
- Virtual memory (VM) is just a table-lookup implementation. The specials about VM are:
	- The table-lookup is implemented inside the CPU, i.e., a hardware solution.
	- **Each process should have its own page table.**
- How about the OS?
	- The OS stores and manages the page tables of all processes.

![[Pasted image 20220502164835.png]]

- Address mapping can also be done in segments
	- Also permits physical address space of a process to be non-contiguous
	- But usually incurs severe **fragmentation** in both memory and backing store
- **Paging is used in most operating systems**
	- Hybrid scheme is also possible

## Demand Paging
```ad-note
title: Memory/Page Allocation
- The stack and the heap will grow when
	- calling `brk()`, i.e., the **heap** grows
	- calling **nested function calls**, i.e., the **stack** grows
```
```ad-question
title: Will the memory be immediately allocated for you when you call `malloc()`?
The [[#^a59584|OOM generator]] runs very fast! It seems the memory was not allocated.
```
````ad-note
title: Demand Paging
- The reality is: allocation is done in a **lazy** way!
	- This lazy way is called **demand paging**
- Yet, it is not really allocated until you access it.
```ad-example
title: Stack
~~~c
#include <stdio.h>
#include <string.h>

#define BUF_SIZE 512 * 1024

void re()
{
    char buf[BUF_SIZE];
    while (getchar() != '\n');
    memset(buf, 0, sizeof(buf));
    while (getchar() != '\n');
    re();
}

int main()
{
    re();
    return 0;
}
~~~
- The `char buf[BUF_SIZE]` statement does not involve any memory access.
	- The virtual address space is allocated, but **the page is not allocated yet**.
- The `memset` statement really accesses the *allocated* memory.
	- This statement really **asks the system** to allocate memory.
```
```ad-example
title: Heap
~~~c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define ONE_MEG (1024 * 1024)
#define COUNT   1024

int main(void)
{
    int i;
    char *ptr[COUNT];

    for (i = 0; i < COUNT; i++)
        ptr[i] = (char *)malloc(ONE_MEG);

    for (i = 0; i < COUNT; i++)
    {
        while (getchar() != '\n');
        memset(ptr[i], 0, ONE_MEG);
    }
}
~~~
- As a matter of fact, `malloc()` does not involve any memory allocation.
	- It only involves **the allocation of the virtual address page**.
	- **The first loop is only for enlarging the virtual page allocation**.
- The `memset` statement really accesses the *allocated* memory.
	- This statement really **asks the system** to allocate memory.
```
````

^a2c931

````ad-note
title: Illustration
Consider the heap example.
- Suppose that a process initially has 4 page frames.
- We are now in the `memset()` for-loop

- When `memset()` runs
	- The MMU finds that a **virtual page involved is invalid**
	- The CPU the generates an interrupt called **page fault**
- The **page fault handling routine** is running
	- The kernel knows the page allocation for all processes.
	- It allocates a memory page for that request.
	- Last, the **page table entry** for Page E (i.e. the 5th page) is updated.
- The routines finishes and the `memset()` statement is **restarted**
	- Then, no page fault will be generated until the next unallocated page is encountered.
- When the routine finds that **all frames are allocated**, we need the help of the **swap area**.
	1. Select a **victim virtual page** and copy the victim to the swap area.
		- Say, the victim virtual page is Page A taking Frame 0.
		- Now, <u>Frame 0 is a free frame</u> and <u>the bit for Page A is 0</u>.
	2. Allocate the free frame to the new frame allocation request.
		- Now Page I (the 9th page) takes Frame 0.
- When **virtual page A** is accessed again, a page fault is generate, and similar steps takes place.
````

````ad-note
title: [[#^be568d|Real OOM]] Illustration
```ad-question
- How does this program *eats* your memory?
- What is the consequence after running this program?
```
Suppose the OOM program has just started with only one page allocated, and there are some other processes in the system.
- First Stage
	- The **free memory frames** are the first zone that the process has conquered.
	- All other processes could hardly allocate pages.
- Second Stage
	- Occupied memory frames are the next zone that the process conquers (no unused frames).
	- Page replacement operations will be carried out by the OS.
	- **Disk activity flies high**!
- Third Stage
	- The previously-conquered frames are swapping to the swap area.
	- Page replacement operations will be carried out by the OS.
	- **Disk activity flies high**!
- Final Stage
	- The page fault handling routine finds that:
		- **No free space left in the swap area**!
		- **Decided to kill the OOM process**!
- **Painful Aftermath**
	- **Lots of page faults**!
		- It is because other processes need to take back the frames!
		- **Disk activity flies high again**, but will go down eventually.
````
```ad-note
title: Issues
- Swap area
	- Where is it?
	- How large is it?
- Can we run a really large process (e.g., bigger than physical memory)?
	- How large is it at most?
- How about `fork()` and `exec*()`?
	- Can they be clever?
```
```ad-note
title: Swap Area
- Location
	- The swap area is usually **a space reserved** in a permanent storage device.
	- Linux needs a separate partition and it is called the swap partition.
	- Windows hides a file `pagefile.sys`, which is the swap area, in one of the drives.
- Size
	- How large should the swap space be?
		- It should be at least the same as the size of the physical memory
			- So that when a really large process wants to take all the memory all the pages on the physical memory can find a place to hide.
		- An old rule said that *swap should be twice the size of the physical memory*.
			- But, I can’t find the reasons anymore, and this rule does not hold nowadays because we now have too much RAM!
```
```ad-note
title: Running Large Programs
- When a process is larger than the physical memory, is it able to run?
	- No need to load all data in memory because of demand paging
		- Generates **page fault** to allocate physical page frames
		- Trigger **page replacement** if there is no unused frames
- Maximum Process Size = Physical Memory Size + Available Space in the Swap Partition (File) - Kernel Memory Size 
```
````ad-note
title: About `fork()` and `exec*()`
- What we have learned about the `fork()` system call is duplication!
	- The parent process and the child process are **identical** from the *userspace memory* point of view
	- What does duplication mean? Allocate new pages for the child process?
		- If yes, then consider `exec*()` system call as well, isn't it stupid?
- We have a clever design with demand paging.
	- A technique called **copy-on-write** (**COW**) is implemented.

```ad-note
title: Copy-on-Write (COW)
- Copy-on-write technique allows the parent and the child processes to share pages after the `fork()` system call is invoked.
- A new, separated page will be **copied and modified** only when one of the processes **wants to write** on a shared page.
```
```ad-example
title: Copy-on-Write Example
![[Pasted image 20220503235625.png|+grid]]![[Pasted image 20220503235724.png|+grid]]

![[Pasted image 20220503235814.png|+grid]]![[Pasted image 20220503235743.png|+grid]]
```
````

^694a6a

````ad-note
title: Performance
- Demand paging can significantly affect performance
	- Service the page fault interrupt
	- Read in the page
	- Restart the instruction/process
- How to characterize?
	- Effective access time
	- $(1-p)\times ma + p\times page\ fault\ time$
		- $ma$: Memory Access Time (10 - 200 ns)
		- $p$: Probability of a Page Fault
		- $page\ fault\ time$: measured in ms
```ad-example
- $ma$: 200 ns
- $p$: 1/1000
- If $page\ fault\ time$: 8 ms
	-  Effective access time: $(1-p) 200\text{ ns} + p × 8\text{ ms} = 8.2\ \mu\text{s}$
- To allow 10% performance degradation only
	- $(1-p) 200\text{ ns} + p × 8\text{ ms}<220\text{ ns}$
	- $p < 0.0000025$
- **Thus, page fault rate must be low**
```
````
```ad-summary
- Demand paging enables **over-commitment**
	- Large process can be supported
	- Concurrent running of multiple processes is also supported
- One key issue is
	- How to select victim pages to swap out?
	- Page-replacement algorithm
```

## Page Replacement 
### Algorithms
```ad-info
title: Introduction
- Remember the **page replacement operation**?
	- It is the job of the kernel to **find a victim page** in the physical memory, and **write the victim page** to the swap space.
- Replacing a page involves disk accesses, therefore a page fault is **slow and expensive**!
	- Key issue: which page should be swapped out?
	- Page replacement algorithms should **minimize further page faults**.
```
In the following, we introduce 4 algorithms:
- Optimal
- First-in first-out (FIFO)
- Least Recently Used (LRU)
- Second-Chance Algorithm

 Imagine that you are the kernel
 - You have a process just started to run
 - The process' memory is larger than the physical memory
 - Assume that all the pages are in the swap space

![[Pasted image 20220504001530.png]]

Initial Condition: Let all the frames be empty.

````ad-note
title: Optimal Algorithm
```ad-question
- If I **know the future**, then I know how to do better.
	- That means I can optimize the result <u>if the page reference string is given in advance</u>.
	- That’s why the algorithm is called *optimal*.
```
- The first page request will cause a page fault.
	- Because there are free frames, no replacement is needed.
- Replace Strategy
	- To replace the page that **will not be used for the longest period of time**.

```ad-example
|  7  |  0  |  1  |  2  |  0  |  3  |  0  |  4  |  2  |  3  |  0  |  3  |  2  |  1  |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| *7* |  7  |  7  | *2* |  2  |  2  |  2  |  2  |  2  |  2  |  2  |  2  |  2  | *1* |
| \-  | *0* |  0  |  0  |  0  |  0  |  0  | *4* |  4  |  4  | *0* |  0  |  0  |  0  |
| \-  | \-  | *1* |  1  |  1  | *3* |  3  |  3  |  3  |  3  |  3  |  3  |  3  |  3  | 
The number of page fault is 8.
```
- Unfortunately, you never know the future
	- It is not practical to implement such an algorithm
	- Is there any easy-to-implement algorithm?
		- You have already learnt process scheduling
````
````ad-note
title: FIFO Algorithm
- FIFO: the **first page being swapped into** the frames will be the **first page being swapped out**.
	- The victim page will always be the oldest page.
	- The age of a page is counted by the time period that it is stored in the memory.
- When there is no free frames
	- The **FIFO page replacement algorithm** will choose the oldest page to be the victim.
	- Of course, the oldest page changes.
- When a memory reference can be found in the memory, the age of that frame won't change.
```ad-example

|  7  |  0  |  1  |  2  |  0  |  3  |  0  |  4  |  2  |  3  |  0  |  3  |  2  |  1  |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| *7* |  7  |  7  | *2* |  2  |  2  |  2  | *4* |  4  |  4  | *0* |  0  |  0  |  0  |
| \-  | *0* |  0  |  0  |  0  | *3* |  3  |  3  | *2* |  2  |  2  |  2  |  2  | *1* |
| \-  | \-  | *1* |  1  |  1  |  1  | *0* |  0  |  0  | *3* |  3  |  3  |  3  |  3  | 

The number of page fault is 11.
```
- Seems that there is no intelligence in this method
- Pages which will be accessed again are swapped out
````
```ad-question
title: Can we do better?
- Still remember the locality rule?
	- Recently accessed pages may be accessed again in near future
- Why not swap out the pages which are not accessed recently
	- This is the **least-recently-used** (LRU) page replacement.
```
````ad-note
title: LRU Algorithm
- Strategy
	- Attach every frame with an age, which is an integer.
	- When a page is just accessed
		- no matter that page is originally on a frame or not, **set its age to be 0**
		- Other frames’ ages are incremented by 1.
```ad-example

|  7  |  0  |  1  |  2  |  0  |  3  |  0  |  4  |  2  |  3  |  0  |  3  |  2  |  1  |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| *7* |  7  |  7  | *2* |  2  |  2  |  2  | *4* |  4  |  4  | *0* |  0  |  0  | *1* |
| \-  | *0* |  0  |  0  |  0  |  0  |  0  |  0  |  0  | *3* |  3  |  3  |  3  |  3  |
| \-  | \-  | *1* |  1  |  1  | *3* |  3  |  3  | *2* |  2  |  2  |  2  |  2  |  2  | 

The number of page fault is 10.
```
- The performance of LRU is considered to be good, but **how to implement** the LRU algorithm efficiently
	- Counters: requires to update counter and search the table to find the page to evict
	- Stack: implement with doubly linked list (pointer update)
- Common case in many systems
	- A reference bit for each page (set by hardware)
	- LRU approximation: **Second-chance algorithm**
````
````ad-note
title: Second-Chance Algorithm
- LRU Approximation
- Basic: FIFO
- Give the page a second chance if its reference bit is on
	- If a page is heavily used, its reference bit will be very likely to be on.

![[Pasted image 20220504004158.png]]
- Clock is the efficient implementation of the Second chance algorithm (**circular queue**).

![[Pasted image 20220504004423.png|+grid]]![[Pasted image 20220504004443.png|+grid]]

- What if all reference bits are set?
	- Degenerates to FIFO
````
### Performance
- Number of page frames v.s. Performance.
	- Increasing the number of page frames implies increasing the amount of the physical memory.
- So, it is natural to think that:
	- I have more memory and more frames
	- Then, my system **must be faster** than before!
	- Therefore, the number of **page faults must be fewer** than before, given the same page reference string.
- But if you try the following:
	- All page frames are initially empty
	- Use **FIFO** page replacement algorithm
	- Use the number of frames: 3, 4, and 5
	- The page reference string is: `1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5`

![[Pasted image 20220504012327.png|+grid]]![[Pasted image 20220504012445.png|+grid]]

- This is called **Belady's Anomaly**
	- It exists for some algorithm
	- Both optimal and LRU do not suffer from it
- **Stack algorithms** never exhibit Belady's Anomaly
	- Feature: The set of pages in memory for $n$ frames is always a **subset** of the set of pages in memory for $n + 1$ frames
	- Example: LRU
		- The $n$ most recently referenced pages will still be the most recently referenced pages when the number of frames increases

## Allocation of Frames
```ad-note
title: Allocation for User Processes
- Free-frame list
	- Demand paging and page replacement
- Constrains
	- Limit on number of frames
		- Upper bound: total available frames
		- Lower bound: has a minimum number
			- Performance consideration (limit page-fault rate)
			- Defined by computer architecture (instructions)
			- Process will be suspended if the number of allocated frames falls below the minimum requirement
	- Global v.s. Local allocation (replacement)
```

^1ace24

```ad-note
title: Allocation Algorithms
- Equal Allocation
	- $m$ frames among $n$ processes
		- $\frac{m}{n}$ frames for each process
	- Memory waste
- Proportional allocation
	- Size of process $p_i$ is $s_i$, then allocate $$a_i = \frac{s_i}{\sum s_i}\times m$$
- Priority-based scheme
	- Ratio depends on both process size and priority
```
````ad-note
title: Issues - Thrashing
- If a process does not have enough frames
	- Frequent page fault
		- Replace a page that will be needed again right away
	- This is called thrashing
		- Spend more time paging than executing
```ad-example
title: Example: Multi-programming + Global Page Replacement
- Increase CPU utilization (increase degree of multiprogramming)
- Frequent page fault (queue up for paging, reduce CPU utilization, increase degree of multiprogramming)

![[Pasted image 20220504014141.png]]
```
```ad-question
title: How to Address?
- Local replacement/priority replacement
	- Will not cause other processes to thrash
	- Still not fully solve this problem
		- Increase average time for a page fault
		- Longer queue for the paging device
		- Longer effective access time even for non-thrashing processes
- Provide as many frames as needed
	- Use working-set strategy to estimate needed frames
		- Working set: the set of pages in the recent $\Delta$ page references![[Pasted image 20220504014622.png]]
			- $WSS_i$: Working Set Size for Process $p_i$
		-  If $\sum WSS_i > m$, thrashing may occur.
	- Use page-fault frequency![[Pasted image 20220504014752.png]]
```
````

^bc1978

````ad-note
title: Allocation for Kernel Memory
- Kernel memory allocation requirement
	- Features
		- Varying (small) size requirement: different data structures
		- Contiguous requirement (certain hardware devices interact with physical memory)
	- Paging: Internal fragmentation
- Buddy System + Slab Allocation

```ad-note
title: Buddy System
- Allocate memory from a fixed-size segment
	- Power-of-2 Allocator (11 orders)
	- Advantage: coalescing

![[Pasted image 20220504015011.png]]
```
```ad-note
title: Slab Allocation
- Allocate memory for small objects (limit fragmentation)
	- Slab: one/more contiguous pages
	- Cache: one/more slabs
- Reduce fragmentation
- Fast allocation (caching benefit)
- Further reading: SLOB/SLUB

![[Pasted image 20220504015129.png]]
```
````

^f68ceb

## Memory Mapped File
- Ordinary file access
	- `open()`, `read()`, `write()`
	- System Call + Disk Access
- Memory mapped file
	- Memory mapping a file: associate a part of the virtual address space with the file
	- File access
		- Initial access to file: demand paging
		- Subsequent reads/writes: routine memory accesses
		- Improves performance
	- Refer to `mmap(2)` system call
- Also allow multiple processes to map the same file

![[Pasted image 20220504015402.png]]

# Summary
- We have introduced
	- [[#User-Space Memory Management|Segmentation]]
	- [[#Paging|Paging + Page Table]]
	- [[#^a2c931|Demand Paging]] + [[#^694a6a|COW]] + [[#Algorithms|Page Replacement Algorithms]]
	- [[#Allocation of Frames]]
		- [[#^1ace24|User process]]
		- [[#^bc1978|Thrashing]]
		- [[#^f68ceb|Kernel Memory]]
	- [[#Memory Mapped File]]
- More
	- `malloc()` is not that simple: refer to *glibc `malloc()`*
	- Other page-replacement algorithms