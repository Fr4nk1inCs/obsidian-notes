#计算机 #软件 #底层 

# Operating System Services
## User Operating System Interface
### Services Overview
![[Pasted image 20220308171714.png]]
Operating systems provide
- an environment for **execution** of programs and
- **services** to programs and users

Services may differ from one OS to another. But the common classes are
- **convenience** of the user (**Helping Users**)
- **efficiency** of the system (**Ensure Efficiency**)

#### OS Services for Helping Users
- **Program Execution**
	- Services
		- load a program into memory
		- run the program
		- end execution, either normally or abnormally (indicating error)
- **I/O Operations**
	- A running program may require I/O, which may involve a file or an I/O device
	- Users usually cannot control I/O devices directly, so OS provides a mean to do I/O, <u>mainly for efficiency and protection</u>
	- Services
		- common I/Os: read, write, etc.
		- special functions: recording CD/DVD
- **File-system manipulation**
	- The file-system is of particular interest. OS provides a variety of file systems.
	- Main Services
		- read and write files and directories
		- create and delete files and directories
		- search for a given file
		- list file Information
		- permission management: allow/deny access
- **Communications**
	- Information exchange between processes
		- on the same computer 
		- between computers over a network
	- Implementations
		- **Shared Memory**: Two or more processes read/write to a shared section of memory.
		- **Message Passing**: Packets of information are moved between processes by OS.
- **Error detection**
	- OS needs to be constantly aware of possible errors
	- Error Type
		
| Source          | Example                                    |
|:---------------:|:------------------------------------------:|
| CPU             |                                            |
| Memory/Hardware | memory error, power failure, etc.          |
| I/O Devices     | parity error, connection error, etc.       |
| User Program    | arithmetic overflow, access illegal memory | 
	
- Error handling
	- Ensure correct and consistent computing
	- Halt the system, terminate an error causing process, etc.
- **[[#User Interface]]**
	- Almost all operating systems have a user interface (**UI**)
	- Three Forms
		- **Command-Line (CLI)**
			- Shell command
		- **Batch**
			- Shell script
		- **Graphics User Interface**
			- Windows System

#### OS Services for Ensure Efficiency
Systems with multiple users can gain efficiency by sharing the computer resources

- **Resource allocation**
	- Resources must be allocated to each user/job
	- **Resource types**
		- CPU Cycles
		- Main Memory
		- File Storage
		- I/O Devices
	- Special allocation code may be required.
		- E.g., CPU scheduling routines depend on
			- Speed of CPU
			- Jobs
			- Number of Registers
			- etc.
- **Accounting**
	- To keep track of <u>which users use how much and what kinds of resources</u>
	- Usage
		- Accounting for **billing** users
		- Accumulating **usage statistics**, can be used for
			- Reconfiguration of the system
			- Improvement of the efficiency
- **Protection and Security**
	- Concurrent processes should not interfere with each other
	- Control the use of computer
	- **Protection**
		- Ensure that all access to system resources is **controlled**
	- **Security**
		- **User authentication** by password to gain access
		- Extends to **defending external I/O devices** from invalid access attempts
### User Operating System Interface
#### CLI
Command line interface or command interpreter
- Allows direct command entry
- Included in the kernel or treated as a special program
- Sometimes multiple flavors implemented - **shells**
	- Linux: multiple shells (C shell, Korn Shell, etc.)
	- Third party shell or free-user written shell
	- Most shells provide similar functionality (personal preference)
	- Example
		- Windows PowerShell in Windows Terminal![[Pasted image 20220308180800.png]]
		- Linux Z Shell with Oh-My-Zsh module in Ubuntu Xterm!![[2022-03-09 07-26-36 的屏幕截图.png]]
- **Main Function**
	- Get and execute the next user specified command
	- Many commands manipulate files
- Two Ways of **Implementing Commands**
	- CLI does not understand the command
	- Use the command to identify a file to be loaded into memory and executed
	- Add new commands easily
	- Example : `rm file.txt`
		- search for file rm
		- load into memory
		- execute `rm file.txt`

#### GUI
- User-friendly graphical user interface
	- **Mouse-based window-and-menu system** (**desktop** metaphor)
	- *Icons* represents files, programs, actions, etc.
	- Various mouse buttons over objects in the interface cause various actions - provide information, options, execute function, open directory (known as *folder*)
	- Invented at Xerox PARC in early 1970s
- Many systems now include both CLI and GUI interfaces
	- Microsoft Windows is GUI with CLI "command" shell
	- Unix and Linux have CLI with optional GUI interfaces (CDE, KDE, GNOME)

#### Touchscreen Interface
Touchscreen devices require new interfaces
- Mouse not possible or not desired
- Actions and selection based on *gestures*
- Virtual keyboard for text entry
- Voice commands

#### Choices of Interface
- Personal preference
- CLI
	- **More efficient**
	- **Easier for repetitive tasks**
	- System administrator
	- Power users who have deep knowledge of a system
	- Shell scripts
- GUI
	- User-friendly
- The design and implementation of user interface is not a direct function of the OS

## System Calls
### System Calls
- **Programming interface** to the services provided by the OS
- Simple programs may make heavy use of the OS
	- A system executes thousands of system calls per second
	- Not user friendly
- Implementation language
	- Typically written in a high level language (C or C++)
	- Certain low level tasks (direct hardware access) are written using assembly language
- **Example** of using system call
	- Read data from a file and copy to another file
	- `open()` + `read()` + `write()`?
	- ![[Pasted image 20220311223711.png|500]]
- Each OS has its own name for each system call
- How to use?
	- Mostly accessed by programs via a high level **API** rather than direct system call use
- Why prefer API rather than invoking system call?
	- **Easy of use**
		- Simple programs may make heavy use of the OS
	- Program portability
		- Compile and run on any system that supports the same API

### API
**Application Programming Interface (API)**
- A set of functions that are available to application programmers
- Three most common APIs
	- Win32 API for Windows
	- POSIX API for POSIX based systems
		- including virtually all versions of UNIX, Linux and Mac OS X
	- Java API for the Java virtual machine (JVM)
- How to use API?
	- Via a library of code provided by OS
	- Libc: UNIX/Linux with C language

### Implementation
- **System call interface** invokes system call
	-  Provided by the **run-time support system** , which is a set of functions built into libraries within a compiler
- How?
	- intercepts function calls in the API
	- invokes necessary system calls
- Implementation
	- Typically, a number associated with each system call
	- System call interface maintains a table indexed according to the numbers
	- ![[Pasted image 20220311224601.png|500]]
- Standard C Library Example
	- C program invoking `printf()` library call, which calls `write()` system call![[Pasted image 20220311224644.png]]
- **Benefits**
	- The caller needs to know nothing about
		- how the system call is implemented
		- what it does during execution
		
		Just needs to obey API and understand what OS will do as a result call
	- **Most details of OS interface are hidden from programmer by API**
		- Managed by run time support library

### Parameter Passing
- More information is required rather than simply the identity of desired system call
- Parameters
	- File
	- Address
	- Length of buffer
- Three methods to **pass parameters** to the OS
	- Simplest: pass the parameters in **registers**
		- In some cases, may be more parameters than registers
	- Table-based
		- Parameters stored in a block , or table, in memory, and address of block passed as a parameter in a register
		- This approach taken by Linux and Solaris
		
		![[Pasted image 20220311225150.png]]
	- Stack-based
		- Parameters are placed, or *pushed*, onto the *stack* by the program and *popped* off the stack by the operating system

### Types
**Six Major Categories**:
- Process control
	- `end()`, `abort()`
		- Halt a running program normally or abnormally
		- Transfer control to invoking command interpreter
		- Memory dump & error message
			- Written to disk and examined by debugger
			- Respond to error: alert window (GUI system) or terminate the entire job (batch system)
		- Error level: normal termination (level 0)
	- `load()`, `execute()`
		- Return to existing program: save memory image
		- Both programs continue concurrently: multiprogram
	- `create_process()`, `terminate_process()`
	- `get_process_attributes()`, `set_process_attributes()`
		- Job’s priority
		- Maximum allowable execution time
		- Etc.
	- `wait_time()`
	- `wait_event()`, `signal_event()`
	- `aquire_lock()`, `release_lock()`
- File management
	- create file, delete file
	- open, close file
	- read, write, reposition
	- get and set file attributes
- Device management: physical/virtual devices
	- request device, release device
	- read, write, reposition
	- get device attributes, set device attributes
	- logically attach or detach devices
- Information maintenances 
	- get time or date, set time or date
	- get system data, set system data
		- Number of current users
		- OS version
		- Amount of free memory and disk
	- debugging
		- Dumping memory
		- Single-step execution
		- Time profile: timer interrupt
			- The amount of time that the program executes at a particular location
- Communications
	- **Message passing model**
		- Host name, IP, process name
		- `get_hostid()`, `get_processid()`, `open_connection()`, `close_connection()`, `accept_connection()`, `read_message()`, `write_message()`
		- <u>Useful for exchanging smaller amounts of data</u>
	- **Shared-memory model**
		- Remove the normal restriction of preventing one process from accessing another process’s memory
		- Create and gain access to shared memory region
			- `shared_memory_create()`, `shared_memory_attach()`
		- **Threads: memory is shared by default**
		- <u>Efficient and convenient, having protection and synchronization issues</u>
- Protection
	- Control access to resources
	- All computer systems must be concerned
	- Permission setting
		- `get_permission()`, `set_permission()`
	- Allow / deny access to certain resources
		- `allow_user()`, `deny_user()`
#### Examples of Process Control
##### MS-DOS
- Single tasking
- Shell invoked when system booted
- Simple method to run program
	- **No process created**
- Single memory space
- Loads program into memory, overwriting all but the kernel
- Program exit -> shell reloaded

![[Pasted image 20220311225950.png#center|System startup (Left) and Running a program (Right)|400]]

##### FreeBSD
![[Pasted image 20220311230208.png|inlineR|140]]
- Unix variant
- Multitasking
- User login -> invoke user s choice of shell
- **Shell executes `fork()` system call to create process**
	- Executes `exec()` to load program into process
	- Shell waits for process to terminate or continues with user commands
- Process exits with:
	- `code = 0` -> no error
	- `code > 0` -> error code

### Examples of Windows and Unix
![[Pasted image 20220311232335.png]]

> [https://www.kernel.org/doc/man-pages/](https://www.kernel.org/doc/man-pages/)
> 
> [http://man7.org/linux/man-pages/](http://man7.org/linux/man-pages/)

# Operating System Structures
- General-purpose OS is a very large program
- Various ways to structure ones
	- Simple structure
		-  MS-DOS
	- Monolithic
		- UNIX
	- Layered - an abstraction
	- Microkernel
		- Mach
	- Modules
	- Hybrid system
		- most OSes

## Simple Structure (MS-DOS)
![[Pasted image 20220311233024.png|inlineR|300]]
MS-DOS was written to provide the most functionality in the **least space**
- Do not have well defined structures
- Not divided into modules
- Its interfaces and levels of functionality are not well separated
	- Application programs can access basic I/O routines
	- Vulnerable to errant programs
	- Limited by hardware

## Monolithic Structure (UNIX)
- The original UNIX operating system had **limited structuring**, it consists of **two separable parts**
	- Systems programs
	- The kernel
		- Consists of **everything** below the system call interface and above the physical hardware
		- A series of interfaces and device drivers
- **Monolithic structure**: combine all functionality in one level
	- File system, CPU scheduling, memory management, and other operating system functions
	- Difficult to implement and maintain
	- Performance advantage
- Beyond simple but not fully layered![[Pasted image 20220311233627.png|500]]

## Layered Approach
![[Pasted image 20220311233707.png|inlineR|300]]
- The operating system is divided into a number of layers (levels), each built on top of lower layers
	- The bottom layer (0) is the hardware
	- The highest layer (N) is the user interface
- Implementation
	- Each layer is an implementation of an abstract object made up of data and operations
- Advantages
	- Simple to construct and debug
	- Hides the existence of DS, Ops, hardware from upper layers
- Challenges
	- How to define various layers?
	- Efficiency problem
		- I/O -> memory manage -> CPU scheduling -> hardware

## Microkernel
![[Pasted image 20220311234315.png]]
- Moves as much from the kernel into user space as possible
	- Provides minimal process, memory management and communication
	- **Mach**: example of microkernel (developed by CMU in mid 1980s)
		- Mac OS X kernel (**Darwin**) partly based on Mach
- Main function
	- Communication between client program and services (also in user space)
	- Provided through **message passing**
- Benefits
	- Easier to extend a microkernel: add services to user space, no changes to kernel
	- Easier to port the operating system to new architectures
	- More reliable & more secure (less code is running in kernel)
- Detriments
	- Performance overhead of user space to kernel space communication

## Modules
![[Pasted image 20220312164406.png|450]]
- Many modern operating systems implement **loadable kernel modules**
	- The Kernel has a set of core components
	- Links in additional services via modules (boot time or run time)
	- Common in most modern OSes
- Similar to layered system
	- Any module can call any other model
	- **More flexible**
- Similar to the microkernel
	- Primary module has only core functions
	- No need to invoke message passing
	- **More efficient**

## Hybrid Systems
- Most modern operating systems combine different structures, resulting in **hybrid systems**
	- For address performance, security, usability needs
- Examples
	- Linux kernel
		- Monolithic: single address space (**for efficient performance**)
		- Modular: dynamic loading of functionality
	- Windows
		- Mostly monolithic, plus microkernel for different subsystem personalities (running in user mode), also support loadable kernel module
	- Apple Mac OS X
		- Mach microkernel, BSD Unix parts, plus I/O kit and dynamically loadable modules (called **kernel extensions**)

## Example
### Mac OS X
![[temp.svg|inlineR|400]]
- Layered system
	- user interface
	- application environment & services
	- kernel (Mach + BSD UNIX)
- Mach Microkernel
	- Memory management
	- Inter-process communication
	- Thread scheduling
- BSD UNIX
	- CLI
	- POSIX API
	- Networking
	- File system

### iOS
- Apple mobile OS for *iPhone*, *iPad*
	- Structured on Mac OS X
	- Added functionality
	- Does not run OS X applications natively
		- Also runs on different CPU architecture (ARM vs. Intel)
- Structure
	- **Cocoa Touch** 
		- Objective-C API for developing apps
	- **Media Services**
		- layer for graphics, audio, video
	- **Core Services**
		- provides cloud computing, databases
	- **Core Operating System**
		- based on Mac OS X kernel

### Android
![[Pasted image 20220312170747.png|inlineR|300]]
- Developed by Open Handset Alliance (mostly Google)
	- Similar stack to iOS
	- Open Source
- Based on Linux kernel
	- Provides process, memory, device-driver management
- Optimization
	- **Adds power management**

# Operating System Design and Implementation
Design and Implementation of OS not "solvable", but some approaches have proven successful
## Design
- First problem: **Design goals and specifications**
	- Affected by choice of hardware, type of system
		- batch
		- time-sharing
		- single/multiple users
		- distributed
		- real-time
		- etc.
	- User goals
		- convenient to use
		- easy to learn
		- reliable
		- safe
		- fast
	- System goals
		- easy to design
		- implement
		- maintain
		- flexible
		- reliable
		- error-free
		- efficient
	- **No unique solution to the problem of defining the requirements**
- Important principle to separate
	- **Mechanism**: *How* to do it?
	- **Policy**: *What* will be done?
	- Examples
		- Timer mechanism (for CPU protection)
			- Policy decision: How long the timer is to be set?
		- Priority mechanism (in job scheduling)
			- Policy: I/O intensive programs have higher priority than CPU intensive ones or vice versa
	- Benefits: maximum flexibility
		- Change policy without changing mechanism

## Implementation
- Much variation
	- Early OSes in assembly language
	- Now C, C++
- Actually usually a mix of languages
	- Main body in C
	- Lowest levels in assembly
	- Systems programs in C, C++, scripting languages
- Pros and cons
	- Code can be written faster, easier to understand/debug
	- More high level language, easier to **port** to other hardware
	- Slower & increased storage requirement
- Performance
	- Major performance improvements
		- Better data structure
		- Better algorithm
	- How about developing excellent assembly language code in OS implementation?
		- Modern compiler is well optimized
		- A small amount of the code is critical to performance, easy to do specialized optimization
			- Interrupt handler
			- I/O manager
			- Memory manager
			- CPU scheduler

# MISC
## Operating System Debugging
- **Failure analysis**
	- *log files*: written with error information when process fails
	- *core dump*: a capture of the memory of the processes
	- *crash dump*: memory state when OS crashes
- **Performance tuning**
	- *Trace listing* of system behavior
	- *Interactive tools*: top displays resource usage of process
- Kernighan's Law
	- **Debugging is twice as hard as writing the code in the first place**. Therefore, if you write the code as cleverly as possible, you are, by definition, not smart enough to debug it.

## Operating System Generation
- Operating systems are designed to run on any of **a class of machines**
	- The system must be **configured or generated** for each specific computer site
- **SYSGEN** program obtains information concerning the specific configuration of the hardware system
	- Read from file, ask the operator or probe
	- Generation methods
		- Modify source code and completely recompile
		- Select modules from precompiled library and link together

## System Boot
- System booting on most computer systems
	- Bootstrap program (residing in ROM) locates the kernel, loads it into memory, and starts it
		- ROM needs no initialization, cannot be easily infected by virus
		- Diagnostics to determine machine state
		- Initialization: CPU registers, device controllers, memory
	- Some use two step process: a simple bootstrap loader fetches a more complex bootstrap program, which loads kernel (large OSes)
	- Some store the entire OS in ROM (Mobile OS)
- Common bootstrap loader allows selection of kernel from multiple disks, versions, kernel options (**GRUB**)