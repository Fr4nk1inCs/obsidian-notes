#计算机 #软件 #底层

# Inter-process Communication
## Cooperating processes
- The process within a system maybe
    - **independent**
        - Independent process cannot affect or be affected by other processes
    - **Cooperating**
        - Cooperating process can affect or be affected by other processes
    - Note: Any process that shares data with others is a cooperating process
- Why we need cooperating processes?
    - Information sharing
    - Computation speedup
        - Executing subtasks in parallel
    - Modularity
        - Diving system functions into separate processes
    - Convenience

## Interprocess Communication
- **Interprocess communication**/**IPC** used for exchanging data between processes
    - Cooperating processes need IPC for exchanging data
- Paradigm for cooperating processes
    - **Producer-consumer problem**, useful metaphor for many applications (abstracted problem model) ^1bbeee
        - **producer** process produces information that is consumed by a consumer process
        - At least one producer and one consumer

### Two models
- Two (abstracted) models of IPC
    - ![[Pasted image 20220323100032.png|inlineR|125]]**Shared memory**
        - Establish a shared memory region, read/write to shared region
        - Accesses are treated as routine memory accesses
        - faster
    - ![[Pasted image 20220323095807.png|inlineR|125]]**Message passing**
        - Exchange message
        - Require kernel intervention
        - Easier to implement in distributed system

### Shared Memory Solution
![[temp 24.svg|500]]
- A buffer is needed to allow processes to run concurrently

| A buffer                                                            | A producer process                                                   | A consumer process                                                 |
| ------------------------------------------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------ |
| It is a shared object                                               | It produces a unit of data                                           |                                                                    |
| It is a queue (imagine that it is an array implementation of queue) | It writes that a piece of data to the tail of the buffer at one time | It removes a unit of data from the head of the buffer at one time. |

 - Focus on bounded buffer: requirements ^faecee
     - When the producer wants to put a new item in the buffer **which is already full**
         - **The producer should be suspended**
         - **The consumer should wake the producer up** after having dequeued an item
     - When the consumer wants to consume an item from the buffer **which is empty**
         - **The consumer should be suspended**
         - **The producer should wake the consumer up** after having enqueued an item

- Implementation![[temp 25.svg|inlineR|200]]
    - Shared memory/queue
        ```c
        #define BUFFER_SIZE 10
        typedef struct {
            ...
        } item;
        item buffer[BUFFER_SIZE];
        int in = 0;
        int out = 0;
        ```
    - Producer side
        ```c
        item next_produced;
        while (true) {
            /* produce an item in next produced */
            while (((in + 1) % BUFFER_SIZE) == out)
                ; /* do nothing */
            buffer[in] = next_produced ;
            in = (in + 1) % BUFFER_SIZE;
        }
        ```
    - Consumer side
        ```c
        item next_consumed;
        while (true) {
            while (in == out)
                ; /* do nothing */
            next_consumed = buffer[out];
            out = (out + 1) % BUFFER_SIZE;
            
            /* consume the item */
        }
        ```

### Message Passing
![[temp 26.svg|400]]
- Communicating processes may reside on different computers connected by a network
- IPC facility provides two operations
    - `send(message)`
    - `receive(message)`
- If processes *P* and *Q* wish to communicate
    - Establish a **communication link** between them
    - Exchange messages via `send`/`receive`
- Implementation issues
    - Naming: direct/indirect communication
    - Synchronization: synchronous/asynchronous
    - Buffering

#### Naming
- How to refer to each other?
##### Direct communication
- explicitly name each other
- Operations (symmetry)
    - `send(Q, message)`: send a message to process *Q*
    - `receive(P, message)`: receive a message from process *P*
- Properties of communication link
    - Links are established automatically (every pair can establish)
    - A link is associated with exactly one pair of processes
    - Between each pair, there exists exactly one link
- Disadvantage: limited modularity (hard-coding)
##### Indirect communication
- sent to and received from mailboxes (ports)
- Operations
    - `send(A, message)`: send a message to mailbox *A*
    - `receive(A, message)`: receive a message from mailbox *A*
- Properties of communication link
    - A link is established between a pair of processes only if both members have a shared mailbox
    - A link may be associated with more than two processes
    - Between each pair, a number of different links may exist
- **ISSUES**
    1.  Who receives the message when multiple processes are associated with one link?
        - Policies
            - Allow a link to be associated with at most two processes
            - Allow only one process at a time to execute a receive operation
            - Allow the system to select arbitrarily the receiver (based on an algorithm). Sender is notified who the receiver was.
    2.  Who owns the mailbox?
        - The process (ownership may be passed)
        - The OS (need a method to create, send/receive, delete)

#### Synchronization
- How to implement `send`/`receive`?
    - **Blocking** is considered **synchronous**
        - **Blocking send**: the sender is blocked until the message is received
        - **Blocking receive**: the receiver is blocked until a msg is available
    - **Non-blocking** is considered **asynchronous**
        - **Non-blocking send**: the sender sends the message and resumes
        - **Non-blocking receive** the receiver receives a valid message or null
- Different combinations are possible
    - When both send and receive are blocking, we have a *rendezvous* between the processes.
    - Other combinations need **buffering**.

#### Buffering
- Messages reside in a temporary queue, which can be implemented in three ways
    - **Zero capacity**: no messages are queued on a link, sender must wait for receiver (no buffering)
    - **Bounded capacity**: finite length of *n* messages, sender must wait if link is full
    - **Unbounded capacity**: infinite length, sender never waits

### POSIX Shared Memory
- POSIX shared memory is organized using memory-mapped file
    - Associate the region of shared memory with a file
- Illustrate with the producer consumer problem
    - Producer
    - Consumer

#### Producer
- Create a shared-memory object
    - `shm_fd = shm_fd shm_open(name, O_CREAT | O_RDWD, 0666);`
    - `name`: name of shared-memory object
    - `O_CREAT`: Create the object if it does not exist
    - `O_RDWD`: Open for reading & writing
    - `0666`: Directory permissions
- Configure object size
    - `ftruncate(shm_fd, SIZE);`
    - `shm_fd`: File descriptor for the shared-memory Object
    - `SIZE`: size of the shared-memory object
- Establish a memory-mapped file containing the object (*share*)
    - `ptr = mmap(0,SIZE, PROT_WRITE, MAP_SHARED, shm_fd, 0);`
    - `PORT_WRITE`: Allows writing to the object (only writing is necessary for producer)
    - `MAP_SHARED`: Changes to the shared memory object will be visible to all processes sharing the object

#### Consumer
- Open the shared-memory object
    - `shm_fd = shm_open (name, O_RDONLY, 0666);`
    - `O_RDONLY`: Open for read only
- Memory map the object
    - `ptr = mmap (0, SIZE, PROT_READ, MAP_SHARED, shm_fd, 0);`
    - `PROT_READ`: Allows reading to the object (only reading is necessary for consumer)
- Remove the shared memory object
    - `shm_unlink(name);`

#### Complete Solution
> Compile with `-lrt` flag
- Producer

```c
// producer.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/shm.h>
#include <sys/stat.h>
#include <sys/mman.h>

int main()
{
    /* the size (in bytes) of the shared memory object */
    const int SIZE = 4096;
    /* name of the shared memory object */
    const char *name = "OS";
    /* string written to share memory */
    const char *message_0 = "Hello";
    const char *message_1 = "World!";

    /* shared memory file descriptor */
    int shm_fd;
    /* pointer to shared memory object */
    void *ptr;

    /* create the shared memory object */
    shm_fd = shm_open(name, O_CREAT | O_RDWR, 0666);

    /* configure the size of the shared memory object */
    ftruncate(shm_fd, SIZE);

    /* memory map the shared memory object */
    ptr = mmap(0, SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);

    /* write to the shared memory object */
    sprintf(ptr, "%s", message_0);
    ptr += strlen(message_0);
    sprintf(ptr, "%s", message_1);
    ptr += strlen(message_1);

    return 0;
}
```
- Consumer

```c
// consumer.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/shm.h>
#include <sys/stat.h>
#include <sys/mman.h>

int main()
{
    /* the size (in bytes) of the shared memory object */
    const int SIZE = 4096;
    /* name of the shared memory object */
    const char *name = "OS";
    /* shared memory file descriptor */
    int shm_fd = -1;
    /* pointer to shared memory object */
    void *ptr = NULL;

    /* open the shared memory object */
    while (shm_fd < 0) 
        shm_fd = shm_open(name, O_RDONLY, 0666);

    /* memory map the shared memory object */
    ptr = mmap(0, SIZE, PROT_READ, MAP_SHARED, shm_fd, 0);

    /* read from the shared memory object */
    printf("%s", (char *)ptr);

    /* remove the shared memory object */
    shm_unlink(name);

    return 0;
}
```

Open a shell and run `./consumer`, then open another shell and run `./producer`, you can see the consumer waiting for producer to send a message. If you run `./producer` first, the consumer can get the message immediately.

```shell
> gcc -O2 producer.c -o producer -lrt
> gcc -O2 consumer.c -o consumer -lrt
> ./consumer & (sleep 5s; ./producer)
[1] 6834
HelloWorld![1]  + 6834 done       ./consumer
```

### Sockets
- A **socket** is defined as an endpoint for communication (over a network)
	- A pair of processes employ a pair of sockets
	- A socket is identified by an **IP address** and a **port** number
	- All ports below 1024 are used for standard services
		- telnet server listen to port 23
		- FTP server listen to port 21
		- HTTP server listen to port 80
- ![[Pasted image 20220323140611.png|inlineR|225]]Socket uses a client-server architecture
	- Server waits for incoming client requests by listening to a specific port
	- Accepts a connection from the client socket to complete the connection
- All connections must be unique
	- Establishing a new connection on the same host needs another port (>1024)
- Special IP address 127.0.0.1 (**loopback**) refers to itself
	- Allow a client and server on the same host to communicate using the TCP/IP protocol
- Three types of sockets
	- **Connection-oriented** (**TCP**)
	- **Connectionless**(**UDP**)
	- **Multicast** - data can be sent to multiple recipients
#### Example in Java
- Server

```java
// DateServer.java
import java.net.*;
import java.io.*;

public class DateServer {
    public static void main(String[] args) {
        try (ServerSocket sock = new ServerSocket(6013)) {
            /* Now listen for connections */
            while (true) {
                Socket client = sock.accept();

                PrintWriter pout = new PrintWriter(client.getOutputStream(), true);

                /* Write the date to the socket */
                pout.println(new java.util.Date().toString());

                /* Close the socket and resume listening for connections */
                client.close();
            }
        } catch (IOException ioe) {
            System.err.println(ioe);
        }
    }
}
```

- Client

```java
// DateClient.java
import java.net.*;
import java.io.*;

public class DateClient {
    public static void main(String[] args) {
        try { 
            /* make connection to server socket */
            Socket sock = new Socket("127.0.0.1", 6013);
            InputStream in = sock.getInputStream();
            BufferedReader bin = new BufferedReader(new InputStreamReader(in));

            /* read the date from the socket */
            String line;
            while ((line = bin.readLine()) != null) {
                System.out.println(line);    
            }

            /* close the socket connection */
            sock.close();
        } catch (IOException ioe) {
            System.err.println(ioe);
        }
    }
}

```

- Deploy the server

```shell
> java ./DateServer.java
```

- Run the client

```shell
> java ./DateClient.java
Wed Mar 23 14:34:21 CST 2022
```

### Pipes
![[temp 27.svg#center|A shell pipe example|300]]

- Pipe is a **shared object**
    - **Using pipe** is a way to realize IPC.
    - Acts as a conduit allowing two processes to communicate
- Four issues
    1.  Is the communication unidirectional or bidirectional?
    2.  In the case of two way communication, is it half or full duplex?
    3.  Must there exist a relationship (i.e., *parent-child*) between the communicating processes?
    4.  Can the pipes be used over a network?
- Two common pipes
    - Ordinary pipes
    - Named pipes

#### Ordinary Pipes
- Ordinary pipes (no name in file system)
    - Ordinary pipes are used only for related processes (*parent-child relationship*)
        - Processes must reside on the same machine
    - Ordinary pipes are **unidirectional** (one-way communication)
    - Ceases to exist after communication has finished
- Ordinary pipes allow communication in standard producer-consumer style
    - Producer writes to one end (**write-end**)
    - Consumer reads from the other end (**read-end**)

##### UNIX Pipes
![[temp 30.svg|400]]
- UNIX treats a pipe as a special file (child inherits it from parent)
	- Create: `pipe(int fd[]);`
		- `fd[0]`: read-end
		- `fd[1]`: write-end
	- Access: Ordinary `read()` and `write()` system calls
- Pipes are anonymous (no name in file system), then how to share?
	- `fork()` duplicates parent’s file descriptors
	- Parent and child use each end of the pipe

![[Pasted image 20220323144010.png|+grid]]![[Pasted image 20220323144355.png|+grid]]

```c
#include <sys/types.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#define BUFFER_SIZE 25
#define READ_END 0
#define WRITE_END 1
int main(void)
{
    char write_msg[BUFFER_SIZE] = "Greetings";
    char read_msg[BUFFER_SIZE];
    int fd[2];
    pid_t pid;
    /* create the pipe */
    if (pipe(fd) == -1)
    {
        fprintf(stderr, "Pipe failed");
        return 1;
    }
    /* fork a child process */
    pid = fork();
    if (pid < 0)
    { /* error occurred */
        fprintf(stderr, "Fork Failed");
        return 1;
    }
    if (pid > 0)
    { /* parent process */
        /* close the unused end of the pipe */
        close(fd[READ_END]);
        /* write to the pipe */
        write(fd[WRITE_END], write_msg, strlen(write_msg) + 1);
        /* close the write end of the pipe */
        close(fd[WRITE_END]);
    }
    else
    { /* child process */
        /* close the unused end of the pipe */
        close(fd[WRITE_END]);
        /* read from the pipe */ 
        while (read(fd[READ_END], read_msg, BUFFER_SIZE) < 1);
        printf("read %s", read_msg);
        /* close the read end of the pipe */
        close(fd[READ_END]);
    }
    return 0;
}
```
```shell
> gcc -O2 pipes.c -o pipes 
> ./pipes
read Greetings
```

##### Shell Example
- Programmer's POV![[temp 31.svg|400]]
- Kernel's POV![[temp 32.svg|400]]
	- The `pipe()` system call creates a piece of **shared storage in the kernel space**
	- The pipe is a **FIFO queue with finite space**
	- More, this kind of application demonstrates the **producer-consumer communication model**
		- The first process is the producer
		- The second is the consumer
#### Named Pipes
- Named pipes (pipe with name in file system)
    - No parent child relationship is necessary (processes must reside on the same machine)
    - Several processes can use the named pipe for communication (may have several writers)
    - Continue to exist until it is explicitly deleted
    - Communication is bidirectional (still half-duplex)
- Named pipes are referred to as **FIFOs** in UNIX
    - Treated as typical files
    - `mkfifo()`, `open()`, `read()`, `write()`, `close()`

## Another POV of IPC
|                                       Shared object                                        |                                                   Message passing                                                    |
|:------------------------------------------------------------------------------------------:|:--------------------------------------------------------------------------------------------------------------------:|
|                                   ![[temp 28.svg\|250]]                                    |                                                ![[temp 29.svg\|250]]                                                 |
| **Challenge**. Coordination can only be done by detecting the status of the shared object. | **Challenge**. Coordination relies on the reliability and the efficiency of the communication medium (and protocol). |
|                          pipes, shared memory, and regular files                           |                          E.g., socket programming, message passing interface (MPI) library.                          |

# IPC Problem: Race condition
## Evil Source: the shared objects
- Pipe is implemented with the thought that **there may be more than one process accessing it <u>at the same time</u>**
- For shared memory and files, **concurrent access may yield <u>unpredictable outcomes</u>**

## One example of race condition
![[Pasted image 20220328141236.png|300]]
- Process A:
	1. attach to the shared memory X
	2. add 10 to X
		1. load memory X to register RA
		2. add 10 to RA
		3. write RA to memory X
	3. exit
- Process B
	1. attach to the shared memory X
	2. minus 10 to X
		1. load memory X to register RB
		2. add 10 to RB
		3. write RB to memory X
	3. exit

Say, the origin value of X is 10.
### Workflow \#1
![[temp 34.svg]]
The final value of X is still 10

### Workflow \#2: problem raised
![[temp 35.svg]]
No matter which process runs next, the final value of X is **either 0 or 20**, not 10.

## The curse
- The above scenario is called **race condition**
- A **race condition** means <u>the outcome of an execution depends on a particular order in which the shared resource is accessed.</u>
- Remember: race condition is always a bad thing and debugging race condition has no fun at all!
	- It may end up
		- 99% of the executions are fine
		- 1% of the executions are problematic
- Solution: **Process synchronization**

# Guarantee Mutual Exclusion
## Mutual Exclusion
![[Pasted image 20220330100040.png|300]]
- A simple solution to race condition
	- *When I’m playing with the shared memory, no one could touch it.*
	- A set of processes would not have the problem of race condition **if mutual exclusion is guaranteed**

## Realization
- Kernel
	- Preemptive kernels and non-preemptive kernels
		- Allows (not allow) a process to be preempted while it is running in kernel mode
	- A non-preemptive kernel is essentially free from race conditions on kernel data structures, and also easy to design (especially for SMP architecture)
	- A preemptive kernel is more responsive and more suitable for real-time programming
- More generally: To guarantee that when one process is executing in its critical section, no other process is allowed execute in its critical section.

### Critical Section
- Critical section program code
	- **Section entry**: declaring the start of critical section
		- Telling other processes: *I start accessing the shared object.*
	- **Critical section**: Critical sections is the c ode segment that is accessing the shared object
	- **Section exit**: declaring the end of the critical section
		- Telling other processes: *I finish accessing the shared object.*
	- Reminder section

## Short Break: Summary for above
- **Race condition** is a problem.
	- It makes a concurrent program producing unpredictable results if you are using shared objects as the communication medium.
	- The outcome of the computation **totally depends on the execution sequences** of the processes involved.
- **Mutual exclusion** is a requirement.
	- If it could be achieved, then the problem of the race condition would be gone.
	- Mutual exclusion hinders the performance of parallel computations.
- **Defining critical sections** is a solution.
	- They are code segments that access shared objects.
	- Critical section must be **as tight as possible**.
		- If you <u>declare the entire code of a program to be a big critical section</u>, the program will be a very high chance to <u> block other processes</u> or to <u>be blocked by other processes</u>.
	- Note that <u>one critical section</u> can be designed for **accessing more than one shared objects**.
- **Implementing section entry and exit** is a challenge.
	- The entry and the exit are **the core parts that guarantee mutual exclusion**, but not the critical section.
	-  Unless they are correctly implemented, race condition would appear.

## Section Entry & Exit Implementation
### Typical Scenario
![[temp 33.svg]]
### Requirements & Challenges
- Requirements ^2d3680
	1. **Mutual Exclusion**: No two processes could be simultaneously inside their critical sections. ^1e9dcc
		- **Implication**: when one process is inside its critical section, any attempts to go inside the critical sections by other processes are not allowed.
	2. Each process is executing at a nonzero speed, but no assumptions should be made about the relative speed of the processes and the number of CPUs.
		- **Implication**: the solution **cannot depend on the time spent inside the critical section**, and the solution cannot assume the number of CPUs in the
	3. **Progress**: No process running outside its critical section should block other processes ^d2ded5
		- **Implication**: Only processes that are **not executing in their reminder sections** can participate in deciding which will enter its critical section.
	4. **Bounded waiting**: No process would have to wait forever in order to enter its critical section. ^8e1bec
		- **Implication**: There exists a bound or limit on the number of time that other processes are allowed to enter their critical sections after a process has made a request to enter its critical section (no processes should be **staved to death**).

- Implementation challenges
	- All operation must be atomic, i.e., it can't be interrupted
	- Need to satisfy the above requirements
	- Performance consideration

#### Hardware solution/support
- Rely on atomic instructions
- `test_and_set()` instruction
	- Definition (C-like Pseudocode)
		```c
		boolean test_and_set(boolean *target) {
			boolean rv = *target;
			*target = true;
			return rv;
		}
		```
	- Mutual exclusion implementation
		```c
		do 
		{
			while (test_and_set(&lock)) ; /* do nothing */
				/* critical section */
			lock = false;
				/* reminder section */
		} while (true);
		```
- `compare_and_swap()` instruction
	- Definition (C-like Pseudocode)
		```c
		int compare_and_swap(int *value, int expected, int new_value)
		{
			int temp = *value;
			if (*value == expected)
				*value = new_value;
			return temp;
		}
		```
	- Mutual exclusion implementation
		```c
		do
		{
			while (compare_and_swap(&lock, 0, 1) != 0) ; /* do nothing */
				/* critical section */
			lock = 0;
				/* reminder section */
		} while (true);
		```
		```ad-question
		How to satisfy bounded waiting?
		```
	- Enhanced version
		```c
		do
		{
			waiting[i] = true;
			key = true;
			while (waiting[i] && key)
				key = test_and_set(&lock); // lock is initialized as false
			waiting[i] = false;
				/* critical section */
			j = (i + 1) % n;
			while ((j != i) && !waiting[j])
				j = (j + 1) % n;
			if (j == i)
				lock = false;
			else
				waiting[j] = false;
				/* reminder section */
		} while (true);
		```

### Proposals
```ad-note
title: Disabling interrupt
- **Method**
	- Similar idea as non-preemptive kernels
	- To **disable context switching** when process is inside the critical section.
- **Effect**
	- When a process is in its critical section , no other processes could be able to run.
- **Implementation**
	- A new system call should be provided

	~~~plaintext
	Interrupt disabled -> Critical Section -> Interrupt enabled
	~~~

- **Correctness?**
	- *Correct*, but it is not an *attractive* solution.
	- Not as feasible in a multiprocessor environment
	- Performance issue (may sacrifice concurrency)
``````
```ad-note
title: Mutex Locks
- **Idea**
	- A process must acquire the lock before entering a critical section, and release the lock when it exits the critical section
	- Using a new shared object to detect the status of other processes, and **lock** the shared object

	~~~cpp
	// Shared object: available (lock)
	acquire()
	{
		while (!available) ; // busy waiting
		available = false;
	}
	release()
	{
		available = true;
	}
	~~~
- **Implementation**
	- Calls to acquire and release locks must be performed **atomically**
	- Often use hardware instructions
	~~~plaintext
	acquire() -> Critical Section -> release()
	~~~
- **Issue**
	- Busy waiting: waste CPU resourse
		- **Spinlock**
- **Application**
	- Multiprocessor system
		- When locks are expected to be held for short times
```
Other software-based solutions
- Aim: To decide which process could go into its critical section
- Key Issues
	- Detect the status of processes (section entry)
		- Need other shared variables
	- Atomicity of section entry and exit

```ad-note
title: Strict alternation
- **Method**
	- Using a new shared object to detect the status of other processes
~~~c
// Process 0
// allow to enter when turn == 0
while (TRUE)
{
	while (turn != 0) // Entry
		;             // busy waiting
	critical_section();
	turn = 1;         // Exit
	non_critical_section();
}
~~~
~~~c
// Process 1
// allow to enter when turn == 1
while (TRUE)
{
	while (turn != 1) // Entry
		;             // busy waiting
	critical_section();
	turn = 0;         // Exit
	non_critical_section();
}
~~~
![[Pasted image 20220404223958.png]]
- Drawbacks/Cons
	- Strict alternation seems good, yet, it is **inefficient**
		- Busy waiting wastes CPU resources
	- In addition, the alternation is **too strict**
		- What if Process 0 wants to enter the critical section **twice in a row**?
		- Violate [[#^d2ded5|requirement 3]]
```

```ad-note
title: Peterson's solution
The improve of strict alternation proposal.
- Highlights
	- Share two data items
		- `int turn;` // whose turn to enter its critical section
		- `Boolean interested[];` // if a process wants to enter
	- Processes would act *as a gentleman*: if you wants to enter, I'll let you first
	- No alternation here

~~~c
// Suppose there are 2 processes
int turn;                               // who can enter critical section 
int interested[2] = {FALSE, FALSE};     // if processes want to enter critical section
// Entry
void enter_region(int process)          // process is 0 or 1
{
	int other;                          // the number of the other process
	other = 1 - process;                // other is 1 or 0
	interested[process] = TRUE;
	turn = other;
	while (turn == other &&
	        interested[other] == TRUE)  // gentleman
		;                               // busy waiting 
}
// Exit
void leave_region(int process)         // process: who is leaving
{
	interested[process] = FALSE;
}
~~~
- Satisfied requirements
	- [[#^1e9dcc|Mutual Exclusion]]
	- [[#^d2ded5|Progress]]
	- [[#^8e1bec|Bounded waiting]]
- Issues
	- Busy waiting has its own problem
		- **An apparent problem**: wasting CPU time
		- **A hidden, serious problem**: **priority inversion problem**![[Pasted image 20220404230933.png]]
			1. If a low priority process is inside the critical region, but a high priority process wants to enter the critical region.
			2. Then, the high priority process will perform busy waiting for a long time or even forever.
```
```tx
| Critical Section Problem                                                      ||||
| Disabling interrupts     | Strict alternation | Peterson's solution | Mutex lock |
|:------------------------:|:------------------:|:-------------------:|:----------:|
| Efficiency Concurrency   | Busy waiting                                        |||
| ^^                       | Use other shared variables to detect process status |||
```
### Final proposal: Semaphore
```ad-note
title: Semaphore
- Meaning
	- In real life
		- Semaphore is a flag signaling system
		- It tells a train dirver (or a plane pilot) when to stop and when to proceed.
	- In programming
		- A semaphore is a data type
		- You can imagine it is **an integer**
			- But it is certainly not an integer when it comes to real implementation
- Semaphore is a data type (**additional shared object**)
	- Accessed only through two standard **atomic** operations
	- `down()`: originally termed P (from Dutch *proberen*, "to test")
		- `wait()` in textbook
		- Decrementing the count
	- `up()`: originally termed V (from *verhogen*, "to increment")
		- `signal()` in textbook
		- Incrementing the count
- Two types
	- **Binary semaphore**: 0 or 1 (similar to mutex lock)
	- **Counting semaphore**: control finite number of resources
- Idea
	- Initialize the semaphore to the number of resource instances![[Pasted image 20220404232411.png|400]]
	- Wish to use a resource, perform `down()` to decrement the count![[Pasted image 20220404232458.png|400]]
	- Release a resource, perform `up()` to increment the count![[Pasted image 20220404232535.png|400]]
	- When the count goes to 0, block the processes that wish to use![[Pasted image 20220404232606.png|400]]
- Simple implementation
~~~c
// Data Type Definition
typedef int semaphore;
// Section Entry
void down(semaphore *s)
{
	while (*s == 0) ; // busy waiting
	*s = *s - 1;
}
// Section Exit
void up(semaphore *s)
{
	*s = *s + 1;
}
~~~
- Issues
	- **Busy waiting**
		- **Solution**: block the process instead of busy waiting (place the process into a waiting queue)
			~~~c
			// Section Entry
			void down(semaphore *s)
			{
				while (*s == 0) 
				{
					special_sleep();
				}
				*s = *s - 1;
			}
			// Section Exit
			void up(semaphore *s)
			{
				if (*s == 0)
				{
					special_wakeup();
				}
				*s = *s + 1;
			}
			~~~
		- **Implementation**: : The waiting queue may be associated with the semaphore, so a semaphore is not just an integer
			~~~c
			typedef struct
			{
				int value;
				struct process *list;
			} semaphore;
			~~~
	- **Atomicity**: both operations must be atomic
		- **Solution**: Disabling interrupts
			- Only one process can invoke `disable_interrupt()`. Later processes would be blocked until `enable_interrupt()` is called.

- Implementation
~~~c
// Data type definition
typedef struct
{
	int value;
	struct process *list;  // wait list
} semaphore;
// Section Entry
/*
 * Note that it is impossible for two blocked processes 
 * to get out of the down() simultaneously.
 * Because only one process can invoke disable_interrupt()
 * after sleep and manipulate semaphore
 */
void down(semaphore *s)
{
	disable_interrput();
	while (s->value == 0) 
	{
		/*
		 * Disabling interrupts may sacrifice concurrency, So it is
		 * essential to keep the critical section as short as possible.
		 */
		enable_interrupt();  // 
		special_sleep();
		disable_interrput(); // Only one process can invoke this function
	}
	s->value = s->value - 1;
	enable_interrupt();
}
// Section Exit
void up(semaphore *s)
{
	disable_interrput();
	if (s->value == 0)
	{
		special_wakeup();
	}
	s->value = s->value + 1;
	enable_interrupt();
}
~~~
- In action ![[Pasted image 20220405000208.png]]
- More
	- **Atomic operation**
		- Either none of the instructions of an atomic operation were completed
		- Or all instructions of an atomic operation are completed
	- In other words, the entire `up()` and `down()` are indivisible.
		- If it returns, the change must have been made
		- If it is aborted, no change would be made
```
### Summary
What happened is just the implementation of mutual exclusion (section entry and section exit).

|                      |                                  Comments                                  |
|:--------------------:|:--------------------------------------------------------------------------:|
| Disabling interrupts |     Time consuming for multiprocessor systems, sacrifices concurrency.     |
|  Strict alternation  | Not a good one, busy waiting & violating one mutual exclusion requirement. |
| Peterson’s solution  |         Busy waiting & has potential *priority inversion problem*          |
|      Semaphore       |                              **BEST CHOICE**                               | 

# The Deadlock Problem
```ad-example
title: Example: Problems when using semaphore
Scenrio: ![[Pasted image 20220405002108.png]]
- P0 must wait until P1 executes up(Y)
- P1 must wait until P0 executes up(X)
```
## Requirements
1. Mutual Exclusion
	- Only one process at a time can use a resource
2. Hold and Wait
	- A process must be holding at least one resource and waiting to acquire additional resources held by other processes
3. No Preemption
	- A resource can be released only voluntarily by the process holding it after that process has completed its task
4. Circular wait
	- There exists a set $\{P_0, P_1, \cdots, P_n\}$ of waiting processes such that $P_0$ waits for $P_1$, $P_1$ waits for $P_2$, …, $P_{n-1}$ waits for $P_n$, $P_n$ waits for $P_0$

## Characterization
Deadlocks can be described using **resource-allocation graph**
```ad-note
title: resource-allocation graph
- Vertex Set $V$ is is partitioned into two types:
	- $P=\{P_0, P_1, \cdots, P_n\}$: processes (represented as circle)
	- $R=\{R_0, R_1, \cdots, R_m\}$: all resource types (represented as big block)
		- each type may have multiple instances (represented as dot in block)
- Edge Set $E$
	- **request edge**: directed edge $P_i\to R_j$
	- **assignment edge**: directed edge $R_j\to P_i$
```
```ad-example
![[Pasted image 20220405010439.png|+grid|200]]![[Pasted image 20220405010446.png|+grid|200]]
```
## Handle
### Detect Deadlock and Recover
#### Detect
##### Single instance of each resource type
Detect the existence of a cycle in resource-allocation graph
```ad-example
![[Pasted image 20220405011733.png#inl|Deadlock|200]]
![[Pasted image 20220405011949.png#inl|No Deadlock|200]]
```

##### Multiple instances of each resource type
Matrix method: 
- 4 data structures ($m$ types of resources, $n$ processes) ^e7a20e
	- Existing (total) resources: $(E_1, E_2, \cdots, E_m)$
	- Available resources: $(A_1, A_2, \cdots, A_m)$
	- Allocation matrix: $\begin{bmatrix}C_{11}&\cdots&C_{1m}\\\vdots&\ddots&\vdots\\ C_{n1}&\cdots& C_{nm}\end{bmatrix}$ ($C_{ij}$: \# of type-j resources held by process i)
	- Need matrix: $\begin{bmatrix}N_{11}&\cdots&N_{1m}\\\vdots&\ddots&\vdots\\ N_{n1}&\cdots& N_{nm}\end{bmatrix}$ ($N_{ij}$: \# of type-j resources needed by process i)
		- $N_{ij} = Max_{ij} - C_{ij}$, $Max$ is the maximum resource demand
- Method:  ![[#^5463d4]]

#### Recover
##### Process and Thread Termination
Use one of two methods. In both methods, the system reclaims all resources allocated to the terminated processes.
- **Abort all deadlocked processes**
	- Clearly break the deadlock cycle
	- At a great expense
- **Abort one process at a time until the deadlock cycle is eliminated**
	- Incurs considerable overhead
	- **Policy decision**: determine which deadlock process should be terminated, factor including
		- Process priority
		- Computed time and expected remaining time
		- Number and types of used resources
		- Number of requested resources
		- Number of processes needed to be terminated

##### Resource Preemption
Successively preempt some resources from processes and give these resources to other processes until the deadlock cycle is broken.

If preemption is required to deal with deadlocks, then three issues need to be addressed:
1. **Selecting a victim**: Which resources and which processes are to be preempted?
2. **Rollback**: roll back the preempted process to some safe state and restart it from that state.
3. **Starvation**: How do we guarantee that the same resources will not always be preempted from the same process?

### Prevent/Avoid Deadlocks: Banker's algorithm
![[#^e7a20e]]
```ad-note
title:Safety Algorithm
Let $W$ (Work) be vector of length $m$, i.e., $(W_1, W_2,\cdots, W_m)$. Let $F$ (Finish) be vector of length $n$ .
1. Initialize $W:=A$, $F_i:=\texttt{false}$, for $i=1, 2,\cdots, n$
2. Find an index $i$ such that both, if no such $i$ exists, go to step 4.
	1. $F_i == \texttt{false}$
	2. $N_i\leq W$
3. $W:= W+C_i$, $F_i=\texttt{true}$, go to step 2.
4. If $F_i==\texttt{true}$ for all $i$, then the system is in a safe state, otherwise it's not (deadlocks detected).

Time Complexity: $\mathcal{O}(m\times n^2)$
```
^5463d4

```ad-note
title: Resource-Request Algorithm
Let $R_i$ (Request) be the request vector for process i. When a request for resources is made by process i, the following steps are taken
1. If $R_i\leq N_i$, go to step 2. Otherwise, raise an error condition, since the process has exceeded its maximum claim.
2. If $R_i\leq A$, go to step 3. Otherwise, the process must wait, since the resource are not available.
3. Have the system pretended to have allocated the requested resources to process i by modifying the state as follows $$\begin{gather}A:=A-R_i\\C_i:=C_i+R_i\\N_i:=N_i-R_i\end{gather}$$ If the resulting state is safe, the transaction is completed, and process i allocated its resources. **However**, if the new state is unsafe, the process must wait for $R_i$, and the old state is restored.
```

```ad-example
title: An Illustrative Example
![[Pasted image 20220405163808.png]]
```

### Ignore (Ostrich algorithm)
Ignore the problem and pretend that deadlocks never occur
- stop functioning and restart manually
- Used by most operating systems, including UNIX and Windows
- Deadlocks occur infrequently, avoiding/detecting it is expensive

---
```ad-caution
A deadlock free solution does not eliminate **starvation**
```

# Classic IPC Problems
All the IPC classical problems use **semaphores** to fulfill the synchronization requirements.

|                            |                                                 Properties                                                  |         Examples          |
|:--------------------------:|:-----------------------------------------------------------------------------------------------------------:|:-------------------------:|
| Producer-Consumer Problem  |      Two classes of processes: **producer** and **consumer**. At least one producer and one consumer.       | FIFO buffer, such as pipe | 
| Dining Philosopher Problem |                       They are all running the same program. At least two processes.                        | Crossroad traffic control |
|   Reader-Writer Problem    | Two classes of processes: **reader** and **writer**. No limit on the number of the processes of each class. |         Database          |

## Producer-Consumer Problem
### Recall
![[#^1bbeee]]


```tx
|:------------------:| ---------------------------------------------------------- |
| A bounded buffer   | - It is a shared object                                    |\
|                    | - Its size is bounded, say N slots                         |\
|                    | - It is a queue (FIFO array)                               |
| A producer process | - It produces a unit of data                               |\
|                    | - Write the data to the tail of the buffer at one time     |
| A consumer process | - Remove a unit of data from the head of the bounded buffer at one time | 
```

```ad-note
title: Requirements
![[#^faecee]]
```

```ad-question
title: Is pipe enough?
- What if we cannot use pipes?
	- Say, there are 2 producers and 2 consumers without any parent child relationships?
- Then, **the kernel can’t protect you with a pipe**.
```

In the following, we revisit the producer consumer problem with **the use of shared objects and
semaphores**, instead of pipe.

### Design - Semaphore
- **ISSUE \#1: Mutual Exclusion**
	- **Solution**: one binary semaphore (`mutex`)
- **ISSUE \#2: Synchronization (coordination)**
	- Remember the two requirements:
		- Insert an item when it is not FULL
		- Consume an item when it is not EMPTY
	- **Solution**: two counting semaphores (`full` & `empty`)

### Solution
```ad-attention
title: Note
- The function `insert_item()` and `remove_item()` are accessing the bounded buffer (critical section)
- The size of bounded buffer is `N`
- Requirements
	- Mutual Exclusion
	- Synchronization
- The order of `down/up(full/empty)` & `down/up(mutex)`
	- inproper order cause deadlocks
```

```c
// ---------- Shared Object ----------
#define N 100
typedef int semaphore;
semaphore mutex = 1;
semaphore empty = N; // The number of empty slots
semaphore full  = 0; // The number of occupied slots
// -------- Producer Function --------
void producer(void)
{
	int item;

	while (TRUE)
	{
		item = produce_item();
		// Critical section entry
		down(&empty);
		down(&mutex);
		// Critical section
		insert_item(item);
		// Critical section exit
		up(&mutex);
		up(&full);
	}
}
// -------- Consumer Function --------
void consumer(void)
{
	int item;

	while (TRUE)
	{
		// Critical section entry
		down(&full);
		down(&mutex);
		// Critical section
		item = remove_item();
		// Critical section exit
		up(&mutex);
		up(&empty);
		// Reminder section
		consume_item(item);
	}
}
```
### Summary
![[Pasted image 20220405173617.png|400]]
- The problem can be divided into two sub problems.
	- Mutual exclusion.
		- The buffer is a shared object.
	- Synchronization.
		- The buffer is bounded.
- Guarantee mutual exclusion
	- A binary semaphore `mutex` is used as the entry and the exit of the critical sections.
- Achieve synchronization
	- Two semaphores are used as **counters** to monitor the status of the buffer.
	- Two semaphores are needed because the two suspension conditions are different.

## Dining Philosopher Problem
### Introduction
```ad-info
title: Dining Philosopher Problem

![[Pasted image 20220406132341.png|inlineR|200]]
- 5 philosophers, 5 plates of spaghetti, and 5 chopsticks.
- The jobs of each philosopher are
	- <u>to think</u>
	- <u>to eat</u>
		- They need **exactly two chopsticks** in order to eat the spaghetti.

~~~ad-question
How to construct a <u>synchronization protocol</u> such that
- they will not result in any **deadlocking scenarios**
- they will not be **starved to death**
~~~
```

|   Problem    |      OS       |
|:------------:|:-------------:|
| Philosophers |   Processes   |
|  Chopsticks  | Shared Object | 

![[Pasted image 20220406132331.png]]

### Requirements
#### Mutual exclusion
- If there is no mutual exclusion, then while you’re eating, the two men besides you must **steal all your chopsticks**!
- **Solution**
	1. Check if anyone is using the chopstick that you need
		1. If yes, you have to wait
		2. If no, **seize both chopsticks**
	2. After eating, put down all your chopsticks

```c
/********** Shared Object **********/
#define N 5
semaphore chop[N]; // init value is 1

/********* Helper Function *********/
void take(int i)
{
	down(&chop[i]);
}
void put(int i)
{
	up(&chop[i]);
}

/********** Main Function **********/
void philosopher(int i)
{
	while (TRUE)
	{
		think();
		// Section Entry
		take(i);
		take((i+1) % N);
		// Critical Section
		eat();
		// Section Exit
		put(i);
		put((i+1) % N);
	}
}
```
```ad-danger
title: Problem
The final destination of the above solution would be **deadlocks**, every tried to take the chopstick on the left, causing deadlock cycle!

![[69FC2EF6-5746-4619-B656-9368632E50AC.jpeg]]
```
#### Synchronization
- Should avoid any **potential deadlocking execution order**
- **Solution**
	1. A philosopher **takes a chopstick**
	2. If a philosopher finds that he cannot take the second one, then he should **put down the first chopstick** and **goes to sleep** for a while.
	3. The philosopher tries to get both chopsticks until both ones are seized.

```c
void take(int i)
{
	while (TRUE)
	{
		down(&chop[i]);
		if (isUsed((i+1) % N))
		{
			up(&chop[i]);
			sleep(1);
		}
		else
		{
			down(&chop[(i+1) % N]);
			break;
		}
	}
}

void philosopher(int i)
{
	while (TRUE)
	{
		think();
		// Section Entry
		take(i);
		// Critical Section
		eat();
		// Section Exit
		up(i);
		up((i+1) % N);
	}
}
```
```ad-danger
title: Potential Problem
Philosophers are all busy but no progress were made!
![[Pasted image 20220406133105.png]]
```

### Final Solution
```ad-note
title: Shared Object
~~~c
#define N     5
#define LEFT  ((i + N - 1) % N)
#define RIGHT ((i + 1) % N)

enum states {
	EATING,
	THICKING,
	HUNGRY
};
states state[N];

semaphore mutex = 1;
semaphore s[N];
~~~
- `LEFT`/`RIGHT`: Going "left" and "right" in a circular manner.
- `state[]`: A shared array, represents the states of the philosophers.
- `mutex`: To guarantee mutual exclusive access to the `state[]` array
- `s[]`: To fulfill the synchronization requirement, initial values are 0
```
```ad-note
title: Extremely Important Helper Function
~~~c
void test(int i)
{
	if (state[i] == HUNGRY &&
		state[LEFT] != EATING &&
		state[RIGHT] != EATING)
	{
		state[i] == EATING;
		up(&s[i]);
	}
}
~~~
```
```ad-note
title: Section Entry
~~~c
void take(int i)
{
	down(&mutex);
	state[i] = HUNGRY;
	test(i);
	up(&mutex);
	down(&s[i]);
}
~~~
- Modifying `state[i]` is also a critical section, so we need semaphore `mutex`.
- `s[i]` are used for blocking philosopher
	- If philosopher is `HUNGRY` and `LEFT` or `RIGHT` is `EATING`, the `up(&s[i])` in `test()` wont't be invoked.
	- Thus, the `down(&s[i])` will block the philosopher until it was waked up.
```
```ad-note
title: Section Exit
~~~c
void put(int i)
{
	down(&mutex);
	state[i] = THINKING;
	test(LEFT);
	test(RIGHT);
	up(&mutex);
}
~~~
After modifying `state[i]` to `THINKING`, the philosopher try to let `LEFT` and then `RIGHT` to eat by invoking `test(LEFT)` and `test(RIGHT)`, waking the philosopher (if blocked/sleeping).
```
```ad-note
title: Main Function
~~~c
void philosopher(int i)
{
	think();
	take(i);
	eat();
	put(i);
}
~~~
```
### Summary
- Solution to IPC problem can be difficult to comprehend.
	- Usually, intuitive methods failed.
	- Depending on time, e.g., `sleep(1)`, does not guarantee a useful solution.
- As a matter of fact, dining philosopher **is not restricted to 5 philosophers**.
## Reader-Writer Problem
### Introduction
```ad-info
title: Reader-Writer Problem
It is a concurrent database problem.
- **Readers** are allowed to read the content of the database concurrently
- A **Writer** need to **lock the database exclusively** so that the readers would not retrieve in consistent data
	- A writer is fobbiden to write any data **before the readers have finished reading**.
	- A writer will also **block the access from other writers**
```
### Subproblems
- A **mutual exclusion** problem.
	- The database is a shared object.
- A **Synchronization** problem.
	1. While a reader is reading, other readers is allowed to read the database.
	2. While a reader is reading, no writer is allowed to write to the database.
	3. While a writer is writing, no writers and readers are allowed to access the database.
- A **concurrency** problem
	- **Simultaneous access for multiple readers** is allowed and must be guaranteed.

### Solution Outline
- **Mutual Exclusion**: relate the readers and the writers to one semaphore `database`.
	- Guarantees **no readers and writers** could proceed to their critical sections at the same time
	- Guarantees **no two writers** could proceed to their critical sections at the same time.
- **Readers’ concurrency**
	- The **first reader coming** to the system `down()` the `database` semaphore
	- The **last reader leaving** the system `up()` the `database` semaphore
	- Using a shared object reader counter `read_count`;

### Final Solution
```ad-note
title: Shared Object
~~~c
semaphore db         = 1;
semaphore mutex      = 1;
int       read_count = 0;
~~~
- `db`: Guarantee the mutual exclusion between the readers and the writers.
- `mutex`: Protect the `read_count` variable.
- `read_count`: Keep track of the number of readers in the system.
```
```ad-note
title: Writer Function
~~~c
void writer(void)
{
	while (TRUE)
	{
		// Section Entry
		prepare_write();
		down(&db);
		// Critical Section
		write_database();
		// Section Exit
		up(&db);
	}
}
~~~
The writer is allowed to enter its critical section when no other process is in its critical section (protected by the `db` semaphore)
```
```ad-note
title: Reader Function
~~~c
void reader(void)
{
	while (TRUE)
	{
		// Section Entry
		down(&mutex);
		read_count++;
		if (read_count == 1)
			down(&db);
		up(&mutex);
		// Critical Section
		read_database();
		// Section Exit
		down(&mutex);
		read_count--;
		if (read_count == 0)
			up(&db);
		up(&mutex);
	}
}
~~~
- Section Entry: The first reader `down()` the `db` semaphore so that no writers would be allowed to enter their critical sections.
- Section Exit: The last reader `up()` the `db` semaphore so as to let the writers to enter their critical section.
```
### Summary
- This solution does not limit the number of readers and the writers admitted to the system.
	- A realistic database needs this property.
- This solution gives readers a higher priority over the writers.
	- Whenever there are readers, writers must be blocked, not the other way round.

```ad-question
What if a writer should be given a higher priority?
```
## Summary on IPC Problems
- Properties in common
	- Multiple processes
	- Shared and limited resources
	- Processes have to be synchronized in order to generate useful output
- Requirements in common
	- Guarantee mutual exclusion
	- Uphold the correct synchronization among processes
	- Deadlock-free
# Summary on Chap 5
- Cooperating Processes: **Process Communication**
	- IPC methods
		- [[chap-5-process-communication-&-sychronization#Shared Memory Solution|Shared Memory]]
		- [[chap-5-process-communication-&-sychronization#Pipes|Pipes]]
		- [[chap-5-process-communication-&-sychronization#Sockets|Sockets]]
- Race Condition: **Mutual Exclusion**
	- Realization
		- Define **[[#Critical Section|critical section]]**
	- Implementation
		- [[chap-5-process-communication-&-sychronization#^2d3680|4 requirements]] & [[#Proposals| 5 schemes]]
		- **[[#Final proposal Semaphore|Semaphore]]**
- Process Synchronization: **Deadlock**
	- Classic Problems
		- Producer-Consumer Problem
		- Dining Philosopher Problem
		- Reader-Writer Problem
