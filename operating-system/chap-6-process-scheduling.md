#计算机 #底层 #软件 

# Outline & Reason
## Outline
![[2A219CB6-7F15-4E19-ABA6-59B0F2E006DD.jpeg]]
## Reasons
- Process Execution
	- Consists of a cycle of CPU execution and I/O wait
	- CPU burst + I/O burst

```ad-question
- How to improve CPU utilization (CPU is much faster than I/O)?
- How to improve System responsiveness (interactive applications)?

Using **[[Chap 1. Overview of an Operating System#Multiprogramming|Multiprogramming]]** and **[[Chap 1. Overview of an Operating System#Multitasking|Multitasking]]**
```
- A system may contain many processes which are at different states (ready for running/waiting for I/O)
- Scheduling is required because the number of computing resource - the CPU - is **limited**.
# Process Lifecycle
```ad-note
title: Programmer’s POV
This is how a fresh programmer looks at a process’ life cycle.

~~~c
int main()
{
	int x = 1;
	getchar();
	return x;
}
~~~
![[temp 38.svg|300]]

```

```ad-note
title: Kernel’s POV
![[temp 39.svg|400]]
- **New: The birth of a proce**
	- Except the first process `init`, every process is created using `fork()`
- **Ready**
	- It means the process is *ready to run* but is *not running*
	- A process become "**ready**"
		- just created by `fork()`
		- has been running on the CPU for some and the OS chooses another process to run
		- returning from blocked states
	- All ready processes are kept on a list called `ready queue`
- **Running**
	- The OS chooses this process to be running on the CPU and changes
its state to **Running**
- Waiting
	- The process is **blocked**
	- While the process is running, it may be waiting for **something** and becomes blocked voluntarily.
		- Reading a file
	- Sometimes, the process has to wait for the response from the device and, therefore, it is **blocked**. Nevertheless, this blocking state is **interruptible**.
		- `Ctrl + C`
	- Sometimes, a process needs to wait for a resource but it doesn’t want to be disturbed while it is waiting. In other words, **the process wants that resource very much**. Then, the process status is set to the **uninterruptible** status.
- Terminated
	- The process is going to die
		- terminate itself
		- force to be terminated
```
# Process Scheduling
```ad-question
title: What is scheduling?
- Mainly about how to make **all the ready processes** become **Running**
- This is called **short-term scheduling or CPU scheduling**.
```
**Scheduling** is the procedure that decides which process to run next.
```ad-note
title: Triggering Events
When process scheduling happens:
~~~tx
| ---------------------------------- | --------------------------------------------------------------------------------------------------- |
| A new process is created           | When `fork()` is invoked and returns successfully.                                                  |
| ^^                                | Then, whether <u>the parent</u> or <u>the child</u> is scheduled is up to the scheduler's decision. |
| An existing process is terminated  | **The CPU is freed**. The scheduler should choose another process to run                            |
| A process waits for I/O            | **The CPU is freed**. The scheduler should choose another process to run                            |
| A process finishes waiting for I/O | The interrupt handling routine **makes a scheduling request**, if necessary.                        | 
~~~
```
```ad-question
title: Key Issues
1. How to make a ready process become running? (Note that the running process may not terminate at that time)
	- **Context Switching**
2. How to decide which process should be running?
	- **Scheduling Criteria** & **Scheduling Algorithms**
3. How to design scheduling in a real/specific system?
	- **Multiprocesser system**
	- **Real-time system**
	- **Algorithm evaluation**
```
## Context Switching
**Context switching** is the actual switching procedure, from one process to another.
```ad-note
title: Switching from one process to another (Context Switching)
1. Suppose this process gives up running on the CPU, e.g., calling `sleep()`. Now it is the time for the scheduler to choose the next process to run.
2. Before the scheduler can seize the control of the CPU, a very important step has to be taken: **Backup all registers' values**. The backup will be stored in the process structure.
	- The context of a process: Union of the user-space memory and the registers' values of the process
	- The backup is stored in **kernel-space**
3. The scheduler decides to schedule another process. Then the scheduler has to **load the context of the new process into the main memory** and into the CPU.

We call the entire operation: **context switching**
```

- Context switching may be expensive.
	- Even worse, the target process may be currently *stored in the hard disk*.
- **Minimizing the number of context switching** may help boosting system performance.

## Scheduling Criteria
```ad-question
How to choose which algorithm to use in a particular situation?
```
- Types
	- Preemptive
	- Non-preemptive
- Application
	- Multiprocessor system
	- Real-time system
- Algorithm Properties
	- CPU utilization
	- Throughput
	- Turnaround time
	- Waiting time
	- Response time

```ad-attention
Application requirements and algorithm properties may vary significantly
```

### Classes of Process Scheduling
```ad-note
title: None-preemptive Scheduling
- What is it?
	- When a process is chosen by the scheduler, the process would never leave the scheduler until
		- The process voluntarily waits for I/O
		- The process voluntarily release the CPU, e.g., `exit()`
- What is the catch?
	- If the process is *purely CPU-bounded*, it will seize the CPU from the time it is chosen until it terminates
- Pros
	- Good for systems that emphasize **the time in finishing tasks**, because the task is running without others' interruption.
- Cons
	- Bad for nowadays systems in which **user experience** and multi-tasking are primary goals.
- It could be found back in the mainframe computers in 1960s
```

```ad-note
title: Preemptive Scheduling
- What is it?
	- When a process is chosen by the scheduler, the process would never leave the scheduler until
		- The process voluntarily waits for I/O
		- The process voluntarily release the CPU, e.g., `exit()`
		- **particular kinds of interrupts and events are detected**
- What is the catch?
	- If that particular event is the *periodic clock interrupt*, then you can have a **time-sharing system**.
- Pros
	- Good for systems that emphasize **interactiveness**, because every will recieve attentions from the CPU.
- Cons
	- Bad for systems that emphasize the time in finishing tasks
- This is the design of nowadays systems.
```

### Performance Measures
```ad-question
title: In algorithm design
What factors/performance measures should be carefully considered?
```
- **CPU utilization**
	- We want to keep CPU as busy as possible.
	- Theoretically, it can range from 0 to 100%, but in real system, it ranges from 40% to 90%.
	- The **higher** the better.
- **Throughput**
	- Number of processes that are completed per time unit.
	- The **higher** the better.
- **Turnaround time**
	- Time to execute the process: interval from the time of submission to the time of completion.
		- total running time + waiting time + doing I/O
	- The **lower** the better.
- **Waiting time**
	- The time spent in the ready queue.
	- The **lower** the better.
- **Response time**
	- The time from the submission of a request until the first response is produced.
	- Useful measure for interactive systems.
	- The **lower** the better.

### Challenge
```ad-question
Can we optimize all the above measures simultaneously?

Usually we cannot!
```
- Design Trade-off
- Common Goal

## Scheduling Algorithm
- **Inputs** to the algorithms
	- A set of processes. $\{P_1, P_2, \cdots\}$
	- For each process
		- Arrival Time
		- CPU Requirements
			- This is a non-sense.
			- How can we know the requirement of each task?
- Offline v.s. Online
	- An **offline scheduling algorithm** assumes that you know all the processes submitted to the system before hand.
	- An **online scheduling algorithm** does not have such an assumption.
	- Yet, every real scheduler has to work in an "**online scenario**".
- **Outputs** of the algorithms
	- Scheduling order
	- Individual & Average **turnaround time**
	- Individual & Average **waiting time**
	- Number of context switching

### Different Algorithms
```tx
|                Algorithms                | Preemptive? | Target System |
|:----------------------------------------:|:-----------:|:-------------:|
|     First-come, first-served (FIFO)      |     No      |  Out-of-date  |
|         Shortest-job-first (SJF)         | Con be both |  Out-of-date  |
|             Round-robin (RR)             |     Yes     |    Modern     |
|           Priority scheduling            |     Yes     |    Modern     |
| Priority scheduling with multiple queues |  The real implementation!  ||
```

### First-come, First-served
```ad-example
title: Example 1.
Input:

| Task | Arrival Time | CPU Requirements |
|:----:|:------------:|:----------------:|
|  P1  |      0       |        24        |
|  P2  |      1       |        3         |
|  P3  |      2       |        3         | 

Output:
![[temp 41.svg]]


|  Task   | Waiting Time | Turnaround Time |
|:-------:|:------------:|:---------------:|
|   P1    |      0       |       24        |
|   P2    |      23      |       26        |
|   P3    |      25      |       28        |
| Average |      16      |       26        | 
```

```ad-example
title: Example 1.
Input:

| Task | Arrival Time | CPU Requirements |
|:----:|:------------:|:----------------:|
|  P3  |      0       |        3         |
|  P2  |      1       |        3         |
|  P1  |      2       |        24        | 

Output:
![[temp 42.svg]]

|  Task   | Waiting Time | Turnaround Time |
|:-------:|:------------:|:---------------:|
|   P1    |      4       |       28        |
|   P2    |      2       |       5         |
|   P3    |      0       |       3         |
| Average |      2       |       12        | 
```

- FIFO scheduling is sensitive to the input
- The average waiting time is often long. Think about the scenario (convoy effect)
	- Someone is standing before you in the queue in KFC, and 
	- you find that he/she is ordering the bucket chicken meal (P1 in example 1)!!!!
	- So, two people (P2 and P3) are unhappy while only P1 is happy.

### Shortest-job-first
```ad-example
title: Non-Preemptive SJF
Input:

| Task | Arrival Time | CPU Requirements |
|:----:|:------------:|:----------------:|
|  P1  |      0       |        7         |
|  P2  |      2       |        4         |
|  P3  |      4       |        1         |
|  P4  |      5       |        4         | 

Output:

In this example, we use **FIFO** to break the tie.
![[temp 44.svg]]

|  Task   | Waiting Time | Turnaround Time |
|:-------:|:------------:|:---------------:|
|   P1    |      0       |        7        |
|   P2    |      6       |       10        |
|   P3    |      3       |        4        |
|   P4    |      7       |       11        |
| Average |      4       |        8        | 
```

```ad-example
title: Preemptive SJF
**Rules for preemptive scheduling** (for this example only)
- Preemption happens when a new process arrives at the system
- Then, the scheduler steps in and selects the nexttask based on their **remaining CPU requirements** (Shortest-remaining-job-first)

Input:

| Task | Arrival Time | CPU Requirements |
|:----:|:------------:|:----------------:|
|  P1  |      0       |        7         |
|  P2  |      2       |        4         |
|  P3  |      4       |        1         |
|  P4  |      5       |        4         | 

Output:
![[temp 45.svg]]

|  Task   | Waiting Time | Turnaround Time |
|:-------:|:------------:|:---------------:|
|   P1    |      9       |       16        |
|   P2    |      1       |        5        |
|   P3    |      0       |        1        |
|   P4    |      2       |        6        |
| Average |      3       |        7        | 
```

```ad-note
title: Short Summary
|                             | Non-Preemptive SJF | Preemptive SJF |
|:---------------------------:|:------------------:|:--------------:|
|    Average waiting time     |         4          |     **3**      |
|   Average turnaround time   |         8          |     **7**      |
| Number of context switching |       **3**        |       5        | 

- The waiting time and the turnaround time decrease at the expense of the **increased number of context switching**.
- SJF is probably optimal in that it gives the **minimum average waiting time**.

~~~ad-question
title: Challenge
How to know the length of the next CPU request?

- **Solution**: Prediction by expecting that the next CPU burst will be similar in length to previous ones.
- **General Approach**: exponential average $$\tau_{n+1}=\alpha t_n + (1-\alpha)\tau_{n-1}$$
	- $\tau$: predicted value
	- $t$: Most recent information
	- ![[temp.png|450]]
~~~
```

### Round-Robin
- Round-robin (RR) scheduling is preemptive.
	- Every process is given a **quantum**, or the amount of time allowed to execute.
	- When the quantum of a process is **used up** (i.e., 0), the process releases the CPU and **this is the preemption**
	- Then, the scheduler steps in and it chooses **the next process which has a non-zero quantum** to run.
- Processes are running one-by-one, like a circular queue
	- Designed specially for time-sharing systems

```ad-example
title: RR Example
**Rules** (for this example only)
- The quantum of every process is fixed and is <u>2 units</u>
- The process queue is sorted according the processes' arrival time, in an ascending order.
	- Allowing us to break tie.

Input:

| Task | Arrival Time | CPU Requirements |
|:----:|:------------:|:----------------:|
|  P1  |      0       |        7         |
|  P2  |      2       |        4         |
|  P3  |      4       |        1         |
|  P4  |      5       |        4         | 

Output:
![[temp 46.svg]]

|  Task   | Waiting Time | Turnaround Time |
|:-------:|:------------:|:---------------:|
|   P1    |      9       |       16        |
|   P2    |      5       |        9        |
|   P3    |      0       |        1        |
|   P4    |      4       |        8        |
| Average |     4.5      |       8.5       | 
```

```ad-note
title: RR v.s. SJF

|                             | Non-Preemptive SJF | Preemptive SJF |      RR       |
|:---------------------------:|:------------------:|:--------------:|:-------------:|
|    Average waiting time     |         4          |       3        | 4.5 (largest) |
|   Average turnaround time   |         8          |       7        | 8.5 (largest) |
| Number of context switching |         3          |       5        | 8  (largest)  |

~~~ad-question
The RR algorithm gets all the bad! Why do we still need it?

**The responsiceness of the processes** is great under the RR algorithm. E.g., you won't feel a job is "frozen" because every job is on the CPU from time to time!
~~~
```
```ad-question
title: Issues for Round-Robin
How to set the size of the time quantum?
- **Too large**: FIFO
- **Too small**: frequent context switching
- **In proctice**: 10-100 ms
- **A rule of thumb**: 80% CPU brust should be shorter than the time quantum
```
```ad-note
title: Observation on RR
- Modified versions of round-robin are implemented in (nearly) every modern OS.
	- User run a lot of **interactive jobs** on modern OS-es
	- Users' priority list:
		1. Responsiveness
		2. Efficiency
		- In other words, *ordinary* users expect fast GUI response than an efficient scheduler running behind.
- With the round-robin deployed, the scheduling **looks like random**.
	- It also looks like ***fair to all processes***
```

### Priority Scheduling
- Some basics
	- A task is given a priority (and is usually an integer)
	- A scheduler selects the next process based on the priority.
		- **A typical practice**: the highest priority is always chosen
	- Special Case: SJF, FIFO (equal priority)
- How to define priority
	- Internally
		- Time limits
		- Memory requirements
		- Number of open files
		- CPU burst and I/O burst
	- Externally
		- Process importance
		- Paid funds

```ad-example
title: Priority Scheduling
Assumption
- All arrive at time 0
- Low numbers represents high priority

Input:

| Task | CPU Burst | Priority |
|:----:|:---------:|:--------:|
|  P1  |     7     |    3     |
|  P2  |     1     |    1     |
|  P3  |     2     |    4     |
|  P4  |     1     |    5     |
|  P5  |     5     |    2     | 

Output:
![[temp 48.svg]]

```

- **Problem**: Indefinite blocking or starvation
- **Solution**: Aging
	- Gradually increase the priority of waiting processes

### Priority Scheduling with Multiple Queues (Multilevel Queue Scheduling)
- **Definitions**
	- It is still a priority scheduler.
	- At each priority class, **different schedulers** may be deployed.
	- E.g.: Foreground processes and background processes.

```ad-example
title: Just an example
- The processes are **permanently assigned** to one queue.
- Fixed-priority preemptive scheduling among queues.

|     Priority     |         Scheduler          |
|:----------------:|:--------------------------:|
| Priority class 5 |    Non-preemptive, FIFO    |
| Priority class 4 |    Non-preemptive, SJF     |
| Priority class 3 | RR with quantum = 10 units |
| Priority class 2 | RR with quantum = 20 units |
| Priority class 1 | RR with quantum = 40 units |

```
#### An Example
![[Pasted image 20220422144413.png]]
- **Properties**
	- Process is assigned a fix priority when they are submitted to the system
- The highest priority class will be selected.
	- To prevent high-priority tasks from running indefinitely, the tasks with a higher priority should be *short-lived*, but *important*.
- Lower priority classes will be scheduled only when the upper priority classes has no tasks.
- Of course, it is a good design to have **a high-priority task preempting a low-priority task**.
	- Conditioned that the high-priority task is short-lived.
- Any problem?
	- Fixed priority
	- Indefinite blocking or starvation

#### Multilevel Feedback Queue Scheduling
```ad-question
title: How to improve the previous scheme?
- Allows a process to move between queues (dynamic priority).
- Why needed?
	- E.g.: Separate processes according to their CPU bursts.
```
```ad-example
title: Just an example
A process drops to a lower priority class after it has used up its quantum and has the quantum recharged.

|     Priority     |         Scheduler          |
|:----------------:|:--------------------------:|
| Priority class 5 |    Non-preemptive, FIFO    |
| Priority class 4 |    Non-preemptive, SJF     |
| Priority class 3 | RR with quantum = 10 units |
| Priority class 2 | RR with quantum = 20 units |
| Priority class 1 | RR with quantum = 40 units |
```
- How to design (factors)?
	- Number of queues
	- Scheduling algorithm for each queue
	- Method for determining when to upgrade/downgrade a process
	- Method for determining which queue a  process will enter
- Most general, but also most complex

#### Summary
![[Pasted image 20220422151826.png]]
## Applications & Scenarios
### Scheduling Issues with SMP
![[Pasted image 20220422152436.png]]
- SMP: Each processor may have its private queue of ready processes
- **Scheduling between processors**
	- Process migration
		- Invalidating the cache of the first processor and repopulating the cache of the second processor
	- **Process migration is costly**
- **Processor Affinity**
	- Attempt to keep a process running on the same processor
	- Soft/hard affinity
- **NUMA**
	- CPU scheduler and memory-placement algorithms work together
- **Load balancing**
	- **Push migration**
		- a specific task periodically check the status & re-balance
	- **Pull migration**
		- an idle processor pulls a waiting task from busy processor
- **No absolute rule concerning what policy is best**

### Real-time CPU Scheduling
- Example: **Anti-lock brake system** (ABS)
	- Latency requirement: 3-5 ms
- **Hard real-time systems**
	- A task must be served by its deadline (otherwise, expired as no service at all)
- **Soft real-time systems**
	- Critical processes will be given preference over noncritical processes (no guarantee)
- **Responsiveness**
	- Respond immediately to a real-time process as soon as it requires the CPU
	- **Support priority-based algorithm with preemption**
- **Interrupt latency** (minimized/bounded)
	- Determining interrupt type and save the state of the current process
	- Minimize the time interrupts may be disabled
- **Dispatch latency**
	- Time required by dispatcher (preemption running process and release resources of low-priority process).
	- Most effective way is to use preemptive kernel
#### Algorithms
````ad-note
title: Rate Monotonic Scheduling
- Assumption
	- Processes require CPU at constant periods
	- Processing time $t$
	- Period $p$ (rate $1/p$)
- Each process is assigned a priority proportional to its rate, and schedule processes with a static priority policy with preemption (**fixed priority**)

```ad-example
title: Example #1
![[Pasted image 20220423231722.png]]
```
```ad-example
title: Example #2
![[Pasted image 20220423231805.png]]
```
```ad-caution
Can not guarantee that a set of processes can be scheduled.
```
````
````ad-note
title: Earliest-Deadline-First Scheduling (EDF)
**Dynamically** assigns priorities according to deadline (the earlier the deadline, the higher the priority)
- Does not require the processes to be periodic, nor require a constant CPU time per burst.
- Requires the announcement of deadlines

![[Pasted image 20220424143257.png]]
````

### Example: Linux Scheduler
![[Pasted image 20220424183938.png]]
- A multiple queue, (kind of) static priority scheduler
	- Priorities 0 to 99 are privileged classes
		- The processes in those queues are called **real-time processes**.
		- Real-time processes are either following **RR** or **FIFO** scheduling algorithm.
	- Normal class follows **CFS** scheduling algorithm. (Completely Fair Scheduler)
		- Each process maintains virtual run time (`vruntime`), recording how long each has run
		- CFS selects the process that has the smallest `vruntime` value
		- Decay factor: `nice` value (-20 to +19)
			- the smaller the value is, the *higher priority* the process get
		- Use a red-black tree to maintain run-able tasks
			- The leftmost value is cached

### Algorithm Evaluation
```ad-question
title: How to select a scheduling algorithm? 
- many algorithms with different parameters and properties
- **Steps**
	1. Define a criteria or the importance of various measures (application dependent)
	2. Design/Select an algorithm to satisfy the requirements. How to guarantee?
```
```ad-note
title: Evaluate Algorithms
- **Determininistic modeling**
	- Simple and fast
	- Demonstration examples
- **Queuing modeling**
	- Queueing network analysis
	- Distribution of CPU and I/O burst (Poisson arrival)
	- Little's law: $n=\lambda\times W$
- **Simulation & Implementation**
	- Trace driven
	- High cost (coding/debugging)
	- Hard to understand the full design space
```

## Summary
- So, you may ask:
	- *What is the **best** scheduling algorithm?*
	- *What is the **standard** scheduling algorithm?*
- There is **no best or standard** algorithm because of, at least, the following reasons:
	- No one could predict how many clock ticks does a process requires.
	- On modern OS-es, processes are submitted online.
	- Conflicting criteria

# Summary
![[Pasted image 20220424184934.png]]