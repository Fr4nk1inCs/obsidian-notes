#计算机 #底层 #软件 

# File System Introduction
![[Pasted image 20220519153402.png|500]]

## Introduction
![[Pasted image 20220519153546.png|500]]
- To understand what a file system (FS) is, we follow two different, but related directions:
	- **Layout**
	- **Operations**

```ad-note
title: Layout
Every FS has an unique layout on the storage device. The layout defines:
- **What** are the things stored in the device.
- **Where** the stored things are.
```
```ad-note
title: Operation
- The set of FS operations defines how the OS should work with the FS layout.
- In other words, **OS knows the FS layout and works with that layout**
- The process uses system calls, which then invoke the FS operations, to access the storage device.
```

## FS is not OS 
```ad-note
title: OS ≠ FS
- **An OS supports a FS**.
- An OS can support more than one FS.
- A FS can be read by more than one OS
```
## Storage Device
```ad-note
title: Storage Device ≠ FS
- A FS must be stored on a device.
	- But, a device may or may not contain any FS.
	- Some storage devices can host more than one FS.
- A storage device is only a dummy container.
	- It doesn’t know and doesn’t need to know what FS-es are stored inside it
	- The OS instructs the storage device how the data should be stored.
```
- There are **two basic things** that are stored inside a storage device, and are common to all existing file systems
	- They are **Files** and **Directories**.
	- We will learn what they are and some basic operations of them.

```ad-question
title: How does a FS store data into the disk?
- That is, the **layout** of file systems.
- The layout affects many things
	- The speed in operating on the file systems
	- The reliability in using the file systems
	- The allocation and de-allocation of disk spaces
```

# Programmer's Perspective
## File
```ad-question
title: Why do we need files?
Storing information in memory is good because memory is fast. However, memory vanishes after process termination.
- File provides a long-term information storage.
	- It is **persistent** and survives after process termination.
- File is also a **shared object** for processes to access concurrently.
```
```ad-question
title: What is a file?
- A uniform logical view of stored information provided by OS.
- **OS perspective**: A file is a logical storage unit (a sequence of logical records), it is an abstract data type
- **User perspective**: the smallest allotment of logical secondary storage

> - File type (executable, object, source code, text, multimedia, archive …)
> - File attributes
> - File operations
```
````ad-question
title: What are going to be stored?
```ad-example
title: Example: a text file
`test.txt`
~~~plaintext
hello world\n
~~~
```
~~~tx
| What can we find out in this example?                                  ||
|:-------------------------------------:|:-------------------------------:|
|               Content?                |       Content of the file       |
|               Filename?               | Content of its parent directory |
|              File size?               |      Attribute of the file      | 
~~~
**When a file is named, it becomes independent of the process, the user, and even the system**
````

### File Attributes
```ad-note
title: Typical File Attributes

| Attribute  |                              Detail                               |
|:----------:|:-----------------------------------------------------------------:|
|    Name    |                        Human-readable form                        |
| Identifier |   Unique tag, a number which identifies the file within the FS    |
|    Type    |             Text file, source file, executable file …             |
|  Location  | Pointer to a device and to the location of the file on the device |
|    Size    |                 Number of bytes, words, or blocks                 |
| Time, date |               Creation, last modification, last use               |
| Protection |          Access control information (read/write/execute)          | 

> You can try the command `ls -l`.

> Some new systems also support extended file attributes (e.g., checksum)
```

```ad-note
title: FS attributes are FS dependent
FS attributes are FS dependent
- not OS dependent

|         Common Attributes          | FAT32 | NTFS | Ext2/3/4 |
|:----------------------------------:|:-----:|:----:|:--------:|
|                Name                |   ✓   |  ✓   |    ✓     |
|                Size                |   ✓   |  ✓   |    ✓     |
|             Permission             |       |  ✓   |    ✓     |
|               Owner                |       |  ✓   |    ✓     |
| Access, creation/modification time |   ✓   |  ✓   |    ✓     | 

> The design of FAT32 does not include any security ingredients.
```

````ad-note
title: File Permissions
```ad-example
title: Example: run `ls -l` in linux
~~~shell
$ ls -ll
总用量 8688
-rw-rw-r-- 1 fushen fushen 2083664  4月  6 08:41 ch6.pdf
-rw-rw-r-- 1 fushen fushen 1149854  4月 18 12:39 ch7_part1.pdf
-rw-rw-r-- 1 fushen fushen 3266656  5月  4 08:30 ch8.pdf
-rw-rw-r-- 1 fushen fushen 1000078  5月  9 08:24 ch9_part1.pdf
-rw-rw-r-- 1 fushen fushen 1364858  5月 16 08:34 ch9_part2.pdf
drwxrwxr-x 7 fushen fushen    4096  5月 17 19:42 Fildem
drwxrwxr-x 2 fushen fushen    4096  5月 17 14:51 tmp
drwxrwxr-x 8 fushen fushen    4096  4月 11 13:38 USTC_hw
drwxrwxr-x 4 fushen fushen    4096  4月 11 13:38 USTC_note
drwxrwxr-x 6 fushen fushen    4096  5月 19 14:03 WhiteSur-gtk-theme
~~~
The fist column indicates file permissions.

This 10-bit permission can be divided into 4 field. Take `-rw-rw-r--` as an example
~~~tx
|       \-       | r   | w   | \-  | r   | w   | \-  | r   | \-  | \-  |
|    1st Field   | 2nd Field ||| 3rd Field ||| 4th Field |||
|:--------------:|:---------------:| - | - |:-------:| - | - |:-------:| - | - |
| File/Directory |Permission for the **file owner**|||Permission for the **file's group**|||Permission for the **others**|||
~~~
So `-rw-rw-r--` indicates that the owner and group can both read and write this file, but the others can only read the file.
```
````

````ad-note
title: Change Attributes
```tx
|         Common Attributes          |                          Ways to change them                            ||
|                 ^^                 |                 Command                  |            Syscall            |
|:----------------------------------:|:----------------------------------------:|:-----------------------------:|
|                Name                |                   `mv`                   |          `rename()`           |
|                Size                | Too many tools to update files' contents | `write()`, `truncate()`, etc. |
|             Permission             |                 `chmod`                  |           `chmod()`           |
|               Owner                |                 `chown`                  |            `chown`            |
| Access, creation/modification time |                 `touch`                  |            `utime`            | 
```
````

````ad-note
title: Pathname v.s. Filename
```ad-question
title: A file can be referred to by its name, then how to achieve this?
Take `/home/os/test.txt` as an example
- The **pathname** is `/home/os/test.txt`
	- `/home/os/` is the directory that `test.txt` resides in
	- `test.txt` is the **filename**.
```
- The pathname is **unique** within the entire file system.
- The filename is **not unique** within the entire file system.
- The filename is **only unique within the directory** that it resides.

```ad-question
title: Why do we need to consider **uniqueness**?
- The OS kernel **translates the pathname into a set of data addresses** on the device.
	- That means the pathname is the **key**!
- If the pathname is not unique, *how come the OS can successfully find the data needed*?
```
````

## Operations
![[Pasted image 20220519153402.png|500]]
```ad-example
title: Example: `fopen()`
![[Pasted image 20220519165830.png|inlineR|100]]
- `fopen()`
	- `FILE *fopen(const char *filename, const char *mode)`
		- `FILE` is a structure defined in `stdio.h`
		- There is a lot of helpful data in `FILE`
			- Two important things: the **file descriptor** and **a buffer**.
	- `fopen()` calls `open()`
	- `fopen()` **creates memory** for the `FILE` structure.
		- occupying space in the area of dynamically allocated memory, i.e., `malloc()`
```

````ad-example
title: Example: `fileno()`
- `fileno()` returns the file descriptor of the FILE structure.
- The type of `stdin`, `stdout`, `stderr` is `FILE *`

```c
#include <stdio.h>

int main()
{
    printf("fd of stdin  = %d\n", fileno(stdin));
    printf("fd of stdout = %d\n", fileno(stdout));
    printf("fd of stderr = %d\n", fileno(stderr));
}
```
```shell
$ ./fileno 
fd of stdin  = 0
fd of stdout = 1
fd of stderr = 2
```
````

```ad-note
title: File Operations
| Operation  | Detail                                                               |
|:----------:| -------------------------------------------------------------------- |
|   Create   | Allocate space, add an entry in the directory                        |
|   Write    | Filename/File content (write pointer)                                |
|    Read    | Filename/Memory location (read pointer)                              |
| Reposition | File seek (not involve actual I/O), **required for random accesses** |
|   Delete   | Release space, and erase directory entry                             |
|  Truncate  | Keeps attributes only                                                | 
```

Many operations involve searching the directory for locating the file (read/write/reposition…), can we avoid this content searching?
```ad-note
title: Open-file Table
- An `open()` system call is provided, and it is called before a file is first used
- OS keeps a table containing information about all open files (perprocess and system-wide table)
- The file will be closed when it is no longer being actively used, using `close()` system call
```

```ad-note
title: Opening a File
![[Pasted image 20220519172413.png|500]]

> these steps are OS-independent as well as FS-independent.

1. The process supplies a pathname to the OS.
2. The OS looks for the **file attributes** of the target file in the disk.
3. The disk returns the file attributes.
4. The OS then associates the attributes to a number and the number is called the **file descriptor**.
5. The OS returns the file descriptor to the process.

- Opening a file only involves the **pathname** and the **attributes of the file**, instead of the file content!
```

```ad-note
title: Read from Open Files
![[Pasted image 20220519172958.png|500]]

1. The process supplies a file descriptor to the OS.
2. The OS reads the file attributes and uses the stored attributes to **locate the required data**.
3. The disk returns the required data.
	- File data is stored in a fixed size **cache in the kernel**.
4. The OS fills the **buffer provided by the process** with the data. **Write data to the userspace buffer**.
```

```ad-note
title: File Descriptor
![[Pasted image 20220519173317.png|500]]
```

````ad-note
title: `read()` & `write()`

I/O-related calls will invoke system calls.

| Library calls that eventually invoke the `read()` system call | Library calls that eventually invoke the `write()` system call |
|:-------------------------------------------------------------:|:--------------------------------------------------------------:|
|                     `scanf()`, `fscanf()`                     |                    `printf()`, `fprintf()`                     |
|                    `getchar()`, `fgetc()`                     |                     `putchar()`, `fputc()`                     |
|                      `gets()`, `fgets()`                      |                      `puts()`, `fputs()`                       |
|                           `fread()`                           |                           `fwrite()`                           | 

- `int read(int fd, void *buffer, int bytes_to_read)`: from file to buffer
- `int write(int fd, void *buffer, int bytes_to_write)`: from buffer to file

```ad-note
title: `read()` Syscall
![[Pasted image 20220519175351.png|500]]
1. Check whether the end of the file is reached or not. (Comparing **size** and **file seek**)
2. **Reading data**
3. File data is stored in a fixed size cache in the kernel.
4. Write data to the userspace buffer.

```
```ad-note
title: `write()` Syscall
![[Pasted image 20220519175331.png|500]]
1. Write data to the kernel buffer.
2. According to the data length,
	1. change in file size, if any, and
	2. change in the file seek.
3. The call returns
4. The buffered data will be flushed to the disk from time to time.
```

```ad-note
title: Kernel Buffer Cache
- Performance
	- Increase reading performance?
	- Increase writing performance?
- Problem
	- Can you answer me why **you cannot press the reset button**?
	- Can you answer me why **you need to press the “eject” button before removing USB drives**?
```
````
```ad-summary
![[Pasted image 20220519180523.png|inlineR|300]]
- Every file has its unique pathname.
	- Its pathname leads you to its attributes and the file content.
- A file has **two** important components! Plus, there are usually stored **separately**.
- We only introduce the read/write flow:
	- File writing involves **disk space allocation**; but…
	- The allocation of disk space is highly related to the design of the **layout of the FS**.
	- Also, the same case for the de-allocation of the disk space…
```

## Directory
```ad-note
title: Directory
- A directory is **a file**.
	- It has **file attributes** (FS dependent) and **file content**.
- A directory file looks is **an array of directory entries**.
	![[Pasted image 20220603163554.png]]
```

```ad-note
title: Directory Traversal Process
- How to locate a file using pathname?

1. Suppose that the process wants to open the file `/bin/ls`.
	- The process then supplies the OS the unique pathname `/bin/ls`
2. The OS retrieves the directory file of the root directory `/`
3. The disk returns the directory file.
4. The OS looks for the name `bin` in the directory file.
5. If found, then the OS retrieves the directory file of `/bin` **using the information of the file attributes of `bin`**.
6. The OS looks for the name `ls` in the directory file `bin`.
7. If found, then the OS knows that the file `/bin/ls` is found, and it starts the previously-discussed procedures to open the file `/bin/ls`

![[Pasted image 20220603164251.png|+grid]]![[Pasted image 20220603164310.png|+grid]]![[Pasted image 20220603164329.png|+grid]]
```

```ad-summary
- A directory file records all the files including directories that are belonging to it.
	- *Locate the directory file of the target directory* and to print contents out.
- Locating a file requires the **directory traversal process** 
	- open a file
	- listing the content of a directory
```

```ad-question
title: What is the file creation?
- E.g., creating a file named `test.txt`?
	- `touch test.txt`?
	- `vim test.txt` and then type `:wq`?
	- `cp [some filename] test.txt`?
- The truth is **File creation == Update of the directory file**
```

```ad-note
title: File Creation and Directory
- If I type `touch text.txt` and `text.txt` does not exist, what will happen to the Directory file?
	- A new directory entry is created.

> `touch text.txt` will only create the directory entry, and there is **no allocation for the file content**.

![[Pasted image 20220603165035.png]]
```

```ad-note
title: File Deletion and Directory
- **Removing** a file is the reverse of the creation process.
	- Note that we are not ready to talk about de-allocation of the file content yet.

![[Pasted image 20220603165103.png]]
```

````ad-note
title: Updating Directory File
- When/how to update a directory file?

```tx
|:-------------------------------------:|:--------------------------------- |
|       Creating a directory file       | Syscall - `mkdir()`               |
|                  ^^                   | Example Program - `mkdir`         |
|  Add an entry to the directory file   | Syscall - `open()`, `creat()`     |
|                  ^^                   | Example Program - `cp`, `mv`, etc |
| Remove an entry to the directory file | Syscall - `unlink()`              |
|                  ^^                   | Example Program - `rm`            |
|        Remove a directory file        | Syscall - `rmdir()`               |
|                  ^^                   | Example Program - `rmdir`         | 
```
````

---

```ad-summary
title: Summary of Programmer's POV
- In this part, we have an introduction to FS
	- File and directory
	- The truth about the calls that we usually use
	- We learned: The **content** of a file is not the only entity, but also the file **attributes**.
- In the next part, we will go into the disk:
	- How and where to store the file attributes? 
	- How and where to store the data? 
	- How to manage a disk?
```

# File System Layout
```ad-note
title: Outline

![[Pasted image 20220603170224.png]]

- We briefly introduce the evolution of the file system layout:
	- From a dummy way to advanced ways.
	- The pros and cons are covered.
- We begin to look at some details of the FAT file system and EXT file system
```
````ad-question
title: How to store date?
![[Pasted image 20220603170430.png|400]]
- Consider the following case
	- You are going to design the layout of a FS
	- You are given the freedom to choose the locations to store files, including directory files. 
	- How will you organize the data?
- Some (basic) rules are required:
	- Every data written to the device must be able to be retrieved.
		- Would you use the FS that will lose data randomly?
	- Every FS operation should be done as efficient as possible.
		- Would you use the FS if it takes a minute to retrieve several bytes of data?
	- When a file is removed, the FS should free the corresponding space.
		- Would you use the FS if it cannot free any occupied space?
````

## Trial 1.0 - The Contiguous Allocation
````ad-note
title: Trial 1.0 - The Basics
![[Pasted image 20220603170832.png|inlineR|200]]
- Just like a book
	- Like a book, we need to some space to store the **table of content**, which records the filename and the (starting and ending) addresses of the file content.
	- Contiguous allocation is very similar to the way we write a book. It starts with the **table of content**, which we call the **root directory**.


| Book          | Trial \#1        |
| ------------- | ---------------- |
| Chapter       | Filename         |
| Starting Page | Starting Address |
| NIL           | Ending Address   |

```ad-example
Suppose we have 3 files to store: `rock.mp3`, `sweet.jpg`, `same.exe`. We do not consider the directory structure at this moment.
![[Pasted image 20220603171514.png|500]]
```
- You can locate files easily (with a directory sturcture).
- You can't locate the **allocated space** and the **free space** in a short period of time?
	- It needs an $O(n)$ search, where $n$ is the total number of files.
- File deletion is easy.
	- Space de-allocation and updating the root directory.
````

````ad-note
title: Trial 1.0 - The Bad
- Suppose we need to write a new but large file.
	- We have enough space
	- But there is no holes that I can satisfy the request
	- The name of the problem is called **External Fragmentation**

![[Pasted image 20220603171844.png|500]]

```ad-done
title: Solution
- The **defragmentation process** may help
- But it's **very expensive**
	- think about the disk structure

![[Pasted image 20220603172131.png|500]]
```
- The **growth** problem: There is no space for files to grow

![[Pasted image 20220603172343.png|500]]
````

```ad-summary
title: Trial 1.0 - The Reality
- This kind of file systems has a name called the **contiguous allocation**.
- This kind of file system is not totally useless.
	- The suitable storage device is something that is **read-only**. (just like a book)
- ![[Pasted image 20220603172647.png|inlineR|75]]Real Life Example: ISO9660
	- Better not grow any files.
	- OK to delete files.
	- Better not add any files; or just add to the tail.
```

## Trial 2.0 - The Linked List Allocation
- Lessons learned from Trial 1.0:
	- **File Size Growth**
		- Can we let every file to grow without paying an experience overhead?
	- **External fragmentation**
		- Can we reduce its damage?
- One goal
	- **To avoid allocating apace in a contiguous manner**!

### Trial 2.0

````ad-note
title: Trial 2.0 - The Basics
```ad-question
title: How?
- The first undesirable case in trial 1.0 is to **write a large file** (as it may fail or need defragmentation)
	- So, can we write small files/units only?
		- For large files, let us break them into small pieces…
- The second undesirable case in trial 1.0 is when **file grows** (as it needs reallocation)
	- So, how can we support dynamic growth?
		- Let’s borrow the idea from the linked list…
```
- Linked list allocation
	1. Chop the storage device into **equal-sized blocks**
	2. Fill the new file into the empty space in a **block-by-block** manner.
	3. The root directory becomes strange/complicated.
		- Since a directory file is an array, it is difficult to pretend to be a linked list….
		- Can we have a better solution to optimize the directory?

![[Pasted image 20220603202220.png|500]]
````
### Trial 2.1
```ad-note
title: Trial 2.1 - The Linked List
- Let’s borrow 4 bytes from each block.
	- To write **the block number of the next block** into the first 4 bytes of each block.
	- Real linked list
	- The root directory stores **the first block number** of the file.

![[Pasted image 20220603202707.png|500]]
```

```ad-note
title: Trial 2.1 - The File Size
- Note that we need the **file size stored in the root directory**.
	- The last block of a file **may not be fully filled**.

![[Pasted image 20220603202855.png|500]]
```

```ad-note
title: Trail 2.1 - The Free Space
- One more thing: **free space management**.
	- Extra data is needed to maintain a free list.
	- We can also maintain the free blocks as a linked list, too.

![[Pasted image 20220603203020.png|500]]
```

```ad-note
title: Trail 2.1 - The Good
- External fragmentation problem is solved.
- Files can grow and shrink freely.
- Free block management is easy to implement.
```

```ad-note
title: Trial 2.1 - The Bad

1. **Random access performance problem**
	- The random access mode is to access a file at random locations.
	- The OS needs to access a series of blocks before it can access an arbitrary block.
		- Worst case: **$O(n)$ number of I/O accesses**, where $n$ is the number of blocks of the file.
	- ![[Pasted image 20220603203505.png|500]]
2. **Internal Fragmentation**.
	- A file is not always a multiple of the block size
	- The last block of a file may not be fill completely.
	- This empty space will be wasted since no other files can be allowed to fill such space.
	- ![[Pasted image 20220603203722.png|500]]
```

### Trial 2.2 - The FAT
```ad-question
title: Can we further improve?
- We know that **the internal fragmentation problem is here to stay**.
- How about the random access problem?
	- We are very wrong at the very beginning, decentralized next block location
	- **The information about the next block should be centralized**
```

````ad-note
title: Trial 2.2 - The FAT
- The only difference between 2.1 and 2.2
	- All the information about the next block numbers are centralized
	- It is called **File Allocation Table** (**FAT**)

![[Pasted image 20220603204144.png#center|Difference between Trial 2.1 and Trial 2.2|500]]
![[Pasted image 20220603204606.png#center|The FAT Implementation|500]]

```ad-example
title: Sequential Read Example
Task: Read `ubuntu.iso` sequentially.
1. Look for the first block number of the file.
2. Read the file allocation table to determine the location of the next block.
	- Note that the next block is not necessarily the adjacent one.
3. The process stops until the block with the next block number is 0

![[Pasted image 20220603205414.png|500]]
```
![[Pasted image 20220603205451.png#center|The Entire Layout|500]]
````

```ad-note
title: Trial 2.2 - The Lookup
- A point to look into
	- Centralizing the data does not mean that the *random access problem* will be gone automatically, unless
	- the file allocation table is presented as **an array**
		- So, going to an arbitrary location is as simple as **doing a pointer addition operation**.
- The random access problem can be eased by keeping a **cached version of FAT** inside the kernel.
	- If this table is partially kept on the cache, then **extra I/O requests** will be generated in locating the next block number.

![[Pasted image 20220603210243.png#center|File Allocation Table|500]]
![[Pasted image 20220603210412.png#center|Cached FAT|500]]
```

```ad-note
title: Trial 2.2 and The Reality
- **Every file system** supported by MSDOS and the Windows family is implementing the linked list allocation.
- The file systems are
	- The FAT family: FAT12, FAT16, and FAT32
	- The New Technology File System: NTFS
```

```ad-note
title: FATs Brief Introduction
- What is the meaning of the numbers (12/16/32)?
	- A block is named a **cluster**
	- The main difference among all versions of FAT FA-es is the **cluster address size** $$\texttt{The number of clusters}=2^{\texttt{ cluster address size}}$$

|                        |  FAT12  |  FAT16  |     FAT32     |
|:----------------------:|:-------:|:-------:|:-------------:|
| Cluster address length | 12 bits | 16 bits | 32 bits (28?) |
|   Number of clusters   |   4K    |   64K   |     256M      | 

- **Cluster address sizes**
	- The larger the cluster address size is, the larger **the size of the file allocation table**.
	- The larger the cluster size is, the larger **the size of the disk partition** is.
```

````ad-summary
title: Summary of Trial 2.2
- FAT is not a perfect solution
	- Tradeoff: **trade space for performance**
		- The entire FAT has to be stored in memory so that
		- the performance of looking up of an arbitrary block is satisfactory.

```ad-question
title: Can we have a solution that stands in middle?
- Not store *the entire set* of block locations in memory
- I don’t need an *extremely high performance* in block lookups.
```
````

## Trial 3.0 - The Index-Node Allocation
```ad-note
title: Back to Trial 1.0 - 2.2
- File system layout: how to store file and directory
	- 1.0: Contiguous allocation (*just like a book*)
		- Two key problems: **External Fragmentation** + **File Growth**
	- 2.0: Linked-list allocation (**blocking**)
		- Key problem: **Complicated root Directory**
	- 2.1: Linked-list allocation (**blocking** + **linked list**)
		- Key problem: **Random Access Problem**
	- 2.2: Linked-list allocation (**centralized next-block number/FAT**)
		- Requirement: **FAT Caching**
- FAT
	- FAT provides a good performance in all aspects
		- File creation, file growth/shrink, file deletion …
		- Random access performance 
			- but requires to **cache the FAT**
	- Balance the tradeoff between Performance and memory space
		- Partial caching
		- How?
			- We are going to **break the FAT into pieces**
```

```ad-note
title: Trial 3.0 - The Beginning
> We are going to **break the FAT into pieces**

![[Pasted image 20220603212734.png|500]]

- **Problem**: The index nodes are variable-sized
	- How to manage them?
	- How to locate an index node?
	- How to support file growth?
		- Size of index nodes depends on file size
```

```ad-note
title: Trial 3.0 - The Heart: Extent
An innovative design of index node called **extent**.

![[Pasted image 20220603213305.png|500]]
```

```ad-note
title: Trial 3.0 - The 2 Kinds of Blocks
- **Data** block
	- Stores file data
- **Indirect** block
	- Stores an array of **block addresses**.
	- An address may point to either a data block or another indirect block.\
	- However, in a block, **all** the addresses are either pointing to indirect blocks or data blocks.

![[Pasted image 20220603215111.png|500]]
```

````ad-note
title: Trial 3.0 - The File Size
How large files can be supported?
$$\text{File Size} = \text{number of data blocks} \times \text{block size}$$

```ad-example
title: File size example
| Number of direct blocks | Number of indirect blocks | Number of double indirect blocks | Number of triple indirect blocks | Block size  | Address length |
|:-----------------------:|:-------------------------:|:--------------------------------:|:--------------------------------:|:-----------:|:--------------:|
|           12            |             1             |                1                 |                1                 | $2^x$ bytes |    4 bytes     |

$$
\begin{align*}
&\underbrace{12\times 2^x}_{\text{Direct Block}} &+& \underbrace{1\times 2^{x-2}\times 2^x}_{\text{Indirect Block}} &+& \underbrace{1\times 2^{x-2} \times 2^{x-2}\times 2^x}_{\text{Double Indirect Blocks}} &+& \underbrace{1\times 2^{x-2}\times 2^{x-2} \times 2^{x-2}\times 2^x}_{\text{Triple Indirect Blocks}} \\
= &12 \times 2^x &+& 2^{2x - 2} &+& 2^{3x - 4} &+& \fbox{$2^{4x-6}$}
\end{align*}
$$
The dominating factor is the file size of triple indirect blocks.

|      Block size       |      File size      |
|:---------------------:|:-------------------:|
| 1024 bytes = $2^{10}$ | $\approx$ 16 GBytes |
| 4096 bytes = $2^{12}$ | $\approx$ 4 TBytes  | 
```
````

```ad-note
title: Trial 3.0 - The Final Design
- Now, every index node is of a fixed size
- The root directory stores the **index node number**.
- The index node table is arranged as **an array**. 
	- Looking up **using the index node number**.
	- So looking up an index node will be fast. 

![[Pasted image 20220603222005.png|500]]
```

```ad-question
title: How about the tradeoff between performance and memory usage?
Partial caching is easy
```

```ad-question
title: Any overhead of Trial 3.0?
- The index-node allocation uses more storage:
	- **to trade for a larger file size (with fixed-size index nodes).**
- The indirect blocks are the **extra things**
```

```ad-example
title: Storage Overhead
> The indirect blocks are the **extra things**.

For the table below, we suppose $x = 12$

|                         File Size                         |                                      Number of Indirect Blocks                                      |
|:---------------------------------------------------------:|:---------------------------------------------------------------------------------------------------:|
|            $12\times 2^x + 2^{2x-2}\approx 4$M            |                                    $(2^{x-2})^0 \approx 1$ block                                    |
|      $12\times 2^x + 2^{2x-2} + 2^{3x-4}\approx 4$G       |                     $(2^{x-2})^0 + [(2^{x-2})^0+(2^{x-2})^1] \approx 1$K blocks                     |
| $12\times 2^x + 2^{2x-2} + 2^{3x-4} + 2^{4x-6}\approx 4$T | $(2^{x-2})^0 + [(2^{x-2})^0+(2^{x-2})^1] + [(2^{x-2})^0+(2^{x-2})^1+(2^{x-2})^2] \approx 1$M blocks |

- Maximum number of indirect blocks depends on
	- Block size
	- File size

For the table below, we suppose there is one triple indirect block/

|      Block size       | Max Number of Indirect Block | Max Extra Size Involved |
|:---------------------:|:----------------------------:|:-----------------------:|
| 1024 bytes = $2^{10}$ |       $\approx 2^{16}$       | $\approx$ 256 MBytes    |
| 1024 bytes = $2^{10}$ |       $\approx 2^{20}$       | $\approx$ 4 GBytes      | 

```

```ad-summary
title: Trial 3.0 - The Summary
- FSes in UNIX and Linux use the index-node allocation method.
	- The **Ext2/3/4** file systems
		- The index node is called **inode** in those systems.
		- Ext4 uses extent, not indirect blocks
- We will discuss the details of Ext file system later
```

```ad-summary
title: From Trial 1.0 to Trial 3.0
We studied what are the possible ways to store data in the storage device.
- The things stored are usually
	- **Root directory**
		- Hey, where are the sub-directories?
		- Still remember the directory traversal
	- **File attributes**
		- Except the **file size** and **the locations** of the data blocks, where and what are the **other attributes**?
	- **Free space management**
		- Actually, we didn’t cover that much…
	- **Data block management**
		- The FAT, the extents, the table of content.
```

## Sub-directory
- Let's take the index-node allocation as an example
	- **Directory is also a file**, so it has an inode too
	- Each directory entry keeps the **address** of the file attributes, not the attributes themselves

![[Pasted image 20220603224708.png|500]]

````ad-note
title: Traversing Directory Structure
- Work Together with the layout
	- Let’s take index-node allocation as an example

![[Pasted image 20220603224814.png#center|Directory Traverse tree|500]]

```ad-example
title: `/file`
![[Pasted image 20220603225010.png|500]]
```
```ad-example
title: `/os/file`
![[Pasted image 20220603225041.png|+grid]]![[Pasted image 20220603225122.png|+grid]]
```
````

## File System Information
```ad-question
title: What are stored on disk?
- Root directory, index nodes/FAT, data blocks, free space information …
- Others?
	- How do we know where the root directory is?
	- Where is the first inode?
- **File System Information**
```

````ad-note
title: File System Information
- It is a set of important, FS-specific data

| Examples of FS-specific Data                                  |
| ------------------------------------------------------------- |
| How large is a block?                                         |
| How many allocated blocks are there?                          |
| How many free blocks are there?                               |
| Where is the root directory?                                  |
| Where is the allocation information, e.g., FAT & inode table? |
| How large is the allocation information?                      |

```ad-question
title: Can we ***hardcode*** those information in the kernel code?
**No!!** Because different storage devices have different needs.
![[Pasted image 20220603225724.png|500]]
```

- **Solution**: The workaround is to save those information on the device.
	- Each device should has its own copy of information.

![[Pasted image 20220603225921.png|500]]
````

```ad-summary
title: Story so far
- We talked about the file system layout
	- FAT & index node
- Only one file system can be stored in a disk? **No**
- What is the problem with a very large file system? **Large FAT**
```

## Disk Partitioning
![[Pasted image 20220603230227.png|500]]
- Partitioning is needed to
	- limit the file system size
	- support multiple file systems on a single disk

```ad-question
title: What is a disk partition?
- A disk partition is a logical space
	- A file system must be stored in a partition.
	- An operating system must be hosted in a partition.
- **Boot Code**
	- the code specifies which partition to boot.
- A **partition table** stores the
	- first sector
	- the length
	- the type of a partition

![[Pasted image 20220603230546.png|500]]
```

````ad-note
title: Master Boot Record (MBR)
![[Pasted image 20220603230759.png|500]]
The range of a partition is described by the `(offset, length)` tuple.

```tx
|                             Partition Table Entry                             ||
| Byte(s) | Description                                                          |
| ------- | -------------------------------------------------------------------- |
| 0       | Bootable flag; 0x80 means bootable.                                  |
| 1-3     | Starting CHS address                                                 |
| 4       | Partition type ([details](http://www.datarecovery.com/hexcodes.asp)) |
| 5-7     | Ending CHS address                                                   |
| 8-11    | Starting LBA address (measured in number of sectors)                 |
| 12-15   | Sizes in sectors                                                     | 
```
````

```ad-summary
- Benefits of partitioning
	- **Performance**
		- A smaller file system is more efficient!
	- **Multi-booting**
		- You can have a Windows XP + Linux + Mac installed on a single hard disk, not using VMware.
	- **Data management**
		- You can have one logical drive to store movies, one logical drive to store the OS-related files, etc.
```

````ad-note
title: Final view of a disk storage space
![[Pasted image 20220603231422.png#center|Disk Layout|500]]

```ad-question
title: What is meant by ***formatting*** a disk?
- **Create and initialize** a file system
- In Windows, we have `format.exe`
- In Linux, we have `mkfs.ext2`, `mkfs.ext3`, etc.
```
````
---
```ad-summary
title: Summary of File System Layout
- We have looked into many details about different file system layouts: 
	- Contiguous allocation; 
	- Linked list allocation; 
	- Index-node allocation. 
- We also show the complete view of disk space 
	- File system specific information & disk partition 
- Linked list allocation and index-node allocation are the main streams but not the only way to implement modern file systems.
```

# Details of FAT32
## Introduction
![[Pasted image 20220603234628.png#center|FAT Layout|500]]

```ad-note
title: Trivia

- Cluster Size![[Pasted image 20220603235906.png|500]]
	- Try typing `help format` in the command prompt in Windows.
- Calculating the maximum partition size
	- with the cluster size = 32KB $$(32\times 2^{10})\times 2^{28} = 2^{43}(8\text{TB})$$
```

````ad-note
title: Typical Layout of a FAT32 Partition
|                  | Propose                                                                                                   | Size                                                            |
|:---------------- |:--------------------------------------------------------------------------------------------------------- |:--------------------------------------------------------------- |
| Boot Sector      | Store FS-specific Parameters                                                                              | 1 sector, 512 bytes                                             |
| FSINFO           | Free-space management                                                                                     | 1 sector, 512 bytes                                             |
| Reserved sectors | \-                                                                                                        | Variable, can be changed during format                          |
| FAT (2 pieces)   | A **robust design**: if `FAT 1` is corrupted or containing bad sectors, then `FAT 2` can act as a back up | Variable, depends on disk size and cluster size.                |
| Root directory   | Start of the directory tree.                                                                              | At least one cluster, depend on the number of director entries. | 

![[Pasted image 20220604000504.png|500]]

```ad-example
Try the shell script below
~~~shell
truncate fat.img --size=1G   # Create a image file of 1G
sudo mkfs.vfat -F32 fat.img  # Format the image, -F32 means FAT32
sudo dosfsck -v fat.img      # Read the information stored in the boot sector
~~~
Running `dosfsck`, *DOS file system check*, on a FAT32 FS. This program reads details from the **boot sector**. The output is shown below.
> Different image size results in defferent layout, thus, different output.
~~~plaintext
fsck.fat 4.2 (2021-01-31)
Checking we can access the last sector of the filesystem
Boot sector contents:
System ID "mkfs.fat"
Media byte 0xf8 (hard disk)
       512 bytes per logical sector
      4096 bytes per cluster
        32 reserved sectors
First FAT starts at byte 16384 (sector 32)
         2 FATs, 32 bit entries
   1048576 bytes per FAT (= 2048 sectors)
Root directory start at cluster 2 (arbitrary size)
Data area starts at byte 2113536 (sector 4128)
    261627 data clusters (1071624192 bytes)
63 sectors/track, 64 heads
         0 hidden sectors
   2097144 sectors total
Checking for unused clusters.
Checking free cluster summary.
fat.img: 0 files, 1/261627 clusters
~~~
Let's take the information apart:
1. The boot sector says: A cluster is made of 8 sector. The size of reserved sector is 32 sector.
	~~~plaintext
	Media byte 0xf8 (hard disk)
	       512 bytes per logical sector
	      4096 bytes per cluster
	        32 reserved sectors
	~~~
2. The boot sector says: 2 FATs and each of them is of sized 1,048,576 bytes.
	~~~plaintext
	First FAT starts at byte 16384 (sector 32)
	         2 FATs, 32 bit entries
	   1048576 bytes per FAT (= 2048 sectors)
	~~~
	There's so slack space between reserved sectors of the first FAT.
3. The first data cluster is **Cluster \#2** and it is usually, not always, the **root directory**. **Cluster \#0 & \#1** are reserved.
	~~~plaintext
	Root directory start at cluster 2 (arbitrary size)
	Data area starts at byte 2113536 (sector 4128) # 4128 = 32 + 2 * 2048
	    261627 data clusters (1071624192 bytes)
	~~~

> Here's another output of a image sized 67123200 bytes (65500 KB, 64.01MB)
> ~~~plaintext
> fsck.fat 4.2 (2021-01-31)
> Checking we can access the last sector of the filesystem
> Boot sector contents:
> System ID "mkfs.fat"
> Media byte 0xf8 (hard disk)
>        512 bytes per logical sector
>        512 bytes per cluster
>         32 reserved sectors
> First FAT starts at byte 16384 (sector 32)
>          2 FATs, 32 bit entries
>     516608 bytes per FAT (= 1009 sectors)
> Root directory start at cluster 2 (arbitrary size)
> Data area starts at byte 1049600 (sector 2050)
>     129022 data clusters (66059264 bytes)
> 32 sectors/track, 8 heads
>          0 hidden sectors
>     131072 sectors total
> Checking for unused clusters.
> Checking free cluster summary.
> fat.img: 0 files, 1/129022 clusters
> ~~~
```
````


## Directory and File Attributes
```ad-note
title: Directory Traversal
Example: `dir C:\Windows` in windows command prompt

~~~plaintext
C:\Windows>dir C:\Windows
...
2022/05/07  04:06           360,448 notepad.exe
...
~~~

1. Read the directory file of the root directory starting from **Cluster \#2**. Found `C:\windows` starts from **Cluster #123**.
2. Read the directory file of the `C:\windows` starting from **Cluster \#123**.

![[Pasted image 20220604004157.png|+grid]]![[Pasted image 20220604004232.png|+grid]]
```

````ad-note
title: Directory Entry
Directory entry is just a structure.

| Byte(s) | Description                                                         |
|:-------:| ------------------------------------------------------------------- |
|    0    | 1st character of the filename. (`0x00` or `0xe5` means unallocated) |
|  1-10   | 7+3 character of filename + extension                               |
|   11    | File attributes (e.r., read-only, hidden)                           |
|   12    | Reserved                                                            |
|  13-19  | Creation and access time information                                |
|  20-21  | High 2 bytes of the first cluster address (0 for FAT12/16)          |
|  22-25  | Written time information                                            |
|  26-27  | Low 2 bytes of first cluster address                                |
|  28-31  | File size                                                           | 

- File Name
	- It is the 8 + 3 naming conversion
		- 8 characters for name
		- 3 characters for file extension
- First Cluster Address
	- The FAT is defined to use **little-endian** byte ordering, as its original implementation was on the Intel x86 platform
	- So the address would be \{`entry[21]`, `entry[20]`, `entry[27]`, `entry[26]`\}
		- rather than \{`entry[20]`, `entry[21]`, `entry[26]`, `entry[27]`\}
- File Size
	- The largest size of a file is (4G - 1) bytes

```ad-note
title: Big Endian v.s. Little Endian

Endian-ness is about byte ordering. 

It means the way that a machine (we mean the entire computer architecture) orders the bytes. 

For example, there is a 4-byte integer value `0x89ABCDEF`.

![[Pasted image 20220604005751.png|500]]
```
```ad-example
|    Filename    | Attributes | Cluster Number |
|:--------------:|:----------:|:--------------:|
| `explorer.exe` |   ......   |       32       |

The directory entry looks like
![[Pasted image 20220604010339.png|inlineR|200]]
- Filename
	- Name: `explorer`
	- Extension: `exe`
- First Cluster Address
	- [ ] Big endian: `0x00002000` = 8192 (wrong)
	- [x] Little endian" `0x00000020` = 32
- File Size
	- `0x000FC400` = 1009 KByte
```
````
```ad-question
title: How to store long filename?
For example, how to store the file `I_love_the_operating_system_course.txt`? The Filename is only 11-byte long.
```

````ad-note
title: LFN Directory Entry
![[Pasted image 20220604011336.png|inlineR|200]]
- LFN: Long File Name
	- In FAT32, the 8+3 naming convention is removed by adding more entries to represent the filename
	- Each LFN entry represents 13 characters in Unicode. 
		- 2 bytes per character. 
		- The sequence is upside-down!
	- The normal directory entry is still there.

| Byte(s) |                       Description                       |
|:-------:|:-------------------------------------------------------:|
|    0    |                     Sequence Number                     |
|  1-10   |     File name characters (5 characters in Unicode)      |
|   11    | File attributes - **always** `0x0F` (attribute for LFN) |
|   12    |                        Reserved                         | 
|   13    |                        Checksum                         |
|  14-25  |     File name characters (6 characters in Unicode)      |
|  26-27  |                        Reserved                         |
|  28-31  |     File name characters (2 characters in Unicode)      |

```ad-example
Filename: `I_love_the_operating_system_course.txt`

|          Directory File          |
|:--------------------------------:|
| LFN \#3: `m_cou`, `rse.tx`, `t`  |
| LFN \#2: `erati`, `ng_sys`, `te` |
| LFN \#1: `I_lov`, `e_the_`, `op` |
|           Normal Entry           | 

~~~plaintext
LFN #3      436d 005f 0063 006f 0075 000f 0040 7200     Cm._.c.o.u...@r.
            7300 6500 2e00 7400 7800 0000 7400 0000     s.e...t.x...t...

LFN #2      0265 0072 0061 0074 0069 000f 0040 6e00     .e.r.a.t.i...@n.
            6700 5f00 7300 7900 7300 0000 7400 6500     g._.s.y.s...t.e.

LFN #1      0149 005f 006c 006f 0076 000f 0040 6500     .I._.l.o.v...@e. 
            5f00 7400 6800 6500 5f00 0000 6f00 7000     _.t.h.e._...o.p.

Normal      495f 4c4f 5645 7e31 5458 5420 0064 b99e     I_LOVE~1TXT .d.. 
            773d 773d 0000 b99e 773d 0000 0000 0000     w=w=....w=......
~~~

- The first byte is the sequence number, and they are arranged in descending order.
- The terminating directory entry has the sequence number `OR-ed with 0x40`. 
	- In LFN \#3, it is `0x03 | 0x40 = 0x43`
- Byte 11 is always `0x0F` to indicate that is a LFN
```
````

```ad-summary
A directory is an extremely important part of a FAT-like file system.
- It stores the **start** of the content, i.e., the start cluster number.
- It stores the **end of the content**, i.e., the **file size**.
	- Without the file size, how can you know when you should stop reading a cluster?
- It stores all file attributes.
```

## File Operations
````ad-note
title: Read
Task: read `C:\Windows\explorer.exe` sequentially.

Suppose we already read out the directory entry by directory traversal.

|    Filename    | Attributes | Cluster Number |
|:--------------:|:----------:|:--------------:|
| `explorer.exe` |   ......   |       32       |

![[Pasted image 20220604013410.png|inlineR|100]]
1. Read the content from Cluster \#32.
	- Note: The **file size** may also help determine if the last cluster is reached.
2. Look for the next cluster and it is Cluster #33 (from the FAT table)
3. Since the FAT has marked `EOF`, we have reached the last cluster.
	- Note: The file size help determine **how many bytes** to read from the last cluster.

- 28 bits are used to represent cluster number of FAT32
	- Damaged = `0xFFFFFFF7`
	- EOF >= `0xFFFFFFF8`
	- Unallocated = `0x00000000`
````

````ad-note
title: Write
Task: append data to `C:\windows\explorer.exe`.

|    Filename    | Attributes | Cluster Number |
|:--------------:|:----------:|:--------------:|
| `explorer.exe` |   ......   |       32       |


1. Locate the last cluster.
2. Start writing to the non-full cluster.
3. Allocate the next cluster through FSINFO.
4. Update the FATs and FSINFO.
5. When write finishes, update the file size

![[Pasted image 20220604013931.png|+grid|150]]
![[Pasted image 20220604014006.png|+grid|150]]

```ad-question
title: How to obtain the next free cluster?


![[Pasted image 20220604014343.png|150]]
- The search for the next free cluster is a **circular**, **next-available** search.
- Why implementing next-available? 
	- **Principle of locality**
- Why circular?
	- **To find out every free block**
```
````

````ad-note
title: Delete
1. De-allocate all the blocks involved. Update FSINFO and FATs.![[Pasted image 20220604014759.png|300]]
2. Delete Entry: Change the first byte of the directory entry to `0xE5`.
	- LFN entries also receive the same treatment.

**That’s the end of deletion!**

```ad-question
title: Really delete a file?
- **The file is not really removed from the FS layout**.
	- Perform a search in all the free space. Then, you will find all deleted file contents.
- *Delete data* persists until the de-allocated clusters are reused.
	- This is an issue between performance (during deletion) and security.
```

```ad-question
title: How to delete a file ***securely***?
![[Pasted image 20220604015259.png|+grid]]![[Pasted image 20220604015313.png|+grid]]
- Brute Force?
	- http://www.ohgizmo.com/2009/06/01/manual-hard-drive-destroyer-looks-like-fun/
- What will the research community tell you?
	- http://cdn.computerscience1.net/2006/fall/lectures/8/articles8.pdf
```
````

````ad-note
title: Recover Deleted Files
```ad-question
title: How to ***rescue*** a deleted file?
- If you’re really care about the deleted file, then
	- **PULL THE POWER PLUG AT ONCE!**
	- Pulling the power plug stops the target clusters from being over-written.
```
- Principle of *rescue* deleted file
	- Data persists unless the sectors are reallocated and overwritten.
- The directory entry is still there
	- The first character of filename becomes `0xE5`
	- The first cluster is still there
- If file size <= 1 cluster
	- Because the first cluster address is still readable, the recovery is having a very high successful rate.
	- Note that filenames with **the same postfix** may also be found.
- If file size > 1 cluster
	- It is still possible as the clusters of a file are likely to be contiguously allocated.
	- The next-available search provides a hint in looking for deleted blocks
	- If not, you’d better have the **checksum** and **the exact file size** beforehand, so that you can use a brute-force method to recover the file.
````
## Conclusion
- It is a *nice* file system
	- Space efficient: 4 bytes overhead (FAT entry) per data cluster
- Deletion problem
	- This is a **lazy yet fast** implementation.
	- Need extra protection for deleted data.
- Deployment
	- It is everywhere: SD cards, USB drives, disks …

# Details of Ext2/3
```ad-note
title: Trivia 
- Extended File System (Ext2/3/4) 
	- Follow index-node allocation 
	- Primary FS for Linux distribution 
		- Ext4 was merged in the Linux 2.6.28 and released in 2008 
	- Backward-compatible – For simplicity, we focus on Ext2/3 
		- Features of Ext2/3/4 
		- https://ext4.wiki.kernel.org/index.php/Main_Page 
		- http://e2fsprogs.sourceforge.net/ext2.html
```

## Layout
> You can show the status of a file using `stat` command in Linux
> 
- Ext2/3 file systems follow the [[#Trial 3 0 - The Index-Node Allocation|index-node allocation]] .
- The file system is not that simple
	- It is divided into groups, and
	- every group has the same structure.

![[Pasted image 20220604020835.png|500]]
![[Pasted image 20220604021117.png|500]]

- Why doing so?
	- This is for **reliability and performance**.
- **Reliability**
	- There are many copies of the *superblock*. So, this increases the **reliability of the FS**.
		- The superblock in Group 0 is called the *primary superblock*. 
		- Other superblocks are called the *backup superblock*.
- **Performance**
	- Example
		- Inode table in Group 0 stores inodes from \#1 to \#100
		- Inode table in Group 1 stores inodes from \#101 to \#200
		- etc.
	- The good about this is to **keep the inodes and the file contents close together**!
		- The inodes in a particular group will *usually* refer to the data blocks in the same group.
		- So, this keeps them close together in a physical sense.
		- The storage device may be able to locate the data in a faster manner. (the **principle of locality**)

````ad-note
title: Layout in Each Group
![[Pasted image 20220604021738.png|500]]
```tx
|             Part             |                                          Description                                         |
|:----------------------------:| -------------------------------------------------------------------------------------------- |
|          Superblock          | Stores FS specific data.                                                                     |
| Group Descriptor Table (GDT) | It stores:                                                                                   |\
|                              | - The **starting block numbers** of the block bitmap, the inode bitmap, and the inode table. |\
|                              | - Free block count, free inode count, etc.                                                   |
|         Inode Table          | An array of inodes ordered by the inode number.                                              |
|         Data Blocks          | An array of blocks that stored files                                                         |
|         Block Bitmap         | A **bit string** that represents if **a block** is allocated or not.                         |
|         Inode Bitmap         | A bit string that represents if an inode is allocated or not.                                |
```
- Superblock: FS specific data, including
	- Total number of inodes in the system
	- Total number of blocks in the system
	- Number of reserved blocks
	- Total number of free blocks
	- Total number of free inodes
	- Location of the first block
	- The size of a block
- **Block bitmap**: A sequence of bits indicates the allocation of the blocks. ![[Pasted image 20220604022834.png|400]]
- **Inode bitmap**: A sequence of bits indicates the allocation of the inodes. ![[Pasted image 20220604023006.png|400]]
	- This implies that the **number of files** in the file system is fixed!
````  

## Inode Structure
- We know that the locations of the data blocks of a file are stored in the inode.

![[Pasted image 20220604023207.png|500]]

- An inode is the structure that stores every information about a file.

|    Bytes    | Value                                |
|:-----------:| ------------------------------------ |
|     0-1     | File type and permission             |
|     2-3     | User ID                              |
|     4-7     | Lower 32 bits of file sizes in bytes |
|    8-23     | Time information                     |
|    24-25    | Group ID                             |
|    26-27    | Link count                           |
|      …      | …                                    |
| **40 - 87** | **12 direct data block pointers**    |
| **88 - 91** | **Single indirect block pointer**    |
| **92 - 95** | **Double indirect block pointer**    |
| **96 - 99** | **Triple indirect block pointer**    |
|      …      | …                                    |
|  108 - 111  | Upper 32 bits of file sizes in bytes | 

> More details: https://ext4.wiki.kernel.org/index.php/Ext4_Disk_Layout#Inode_Table

```ad-question
title: What is the maximum file size supported?
$$2^{64} - 1 = 16 \times 2^{30}\text{GBytes} -1\text{byte}$$

Is this really the case?
- Remember the dominating factor: $2^{4x-6}$

|      Block size       |      File size      |
|:---------------------:|:-------------------:|
| 1024 bytes = $2^{10}$ | $\approx$ 16 GBytes |
| 4096 bytes = $2^{12}$ | $\approx$ 4 TBytes  | 
```

```ad-question
title: What is link count?
We will talk about it later.
```

```ad-question
title: Where is the file name?
Let us take a look at the directory structure.
```

## Directory Structure
- The directory entry stores the **file name** and the **inode number**.

```c
struct dirent {
    ino_t          d_ino;     // inode number
    off_t          d_off;     // offset to next dirent
    unsigned short d_reclen;  // record length
    unsigned char  d_type;    // file type
    char *         d_name;    // file name
}
```

![[Pasted image 20220604025019.png|500]]

````ad-note
title: Accessing Directory File

> Note: `opendir()`, `readdir()`, and `closedir()` are library function calls.

```c
#include <stdio.h>
#include <dirent.h>

int main(void) {
    DIR * dir;
    struct dirent *entry;
    // Open the directory file.
    dir = opendir("/");
    // Read the directory entries one by one until there is not further entries.
    while ((entry = readdir(dir)) != NULL) {
        // print the directory name
        printf("%s\n", entry->d_name);
	}
    // Close the directory file
    closedir(dir);
    return 0;
}
```
Output
```plaintext
dev
snap
opt
root
lib32
run
media
libx32
home
swapfile
etc
srv
cdrom
proc
tmp
bin
lib64
usr
mnt
lib
lost+found
sys
..
var
sbin
boot
.
```
````

## Link File
```ad-question
- Can we allow a file to have multiple names and be accessed by several paths?
- How to create shortcuts?
```
````ad-example
Using `ln` command in Linux
```shell
$ echo link > test.txt
$ ln test.txt hard_link
$ cat hard_link    
link
$ ln -s symbolic_link 
$ cat symbolic_link 
link
$ ll
-rw-rw-r-- 2 xxx xxx 5 xx 4 xx:xx hard_link
lrwxrwxrwx 1 xxx xxx 8 xx 4 xx:xx symbolic_link -> test.txt
-rw-rw-r-- 2 xxx xxx 5 xx 4 xx:xx test.txt
```
These are called **hard link** and **symbolic link**
````

````ad-note
title: Hard Link
- A **hard link is a directory entry** pointing to an existing file.
	- No new file content is created!

```shell
$ echo link > test.txt
$ ln test.txt hard_link
$ cat hard_link    
link
$ shred --remove test.txt 
$ cat hard_link # no output
```
![[Pasted image 20220604031935.png|500]]
- Conceptually speaking, this **creates a file with two pathnames**.

![[Pasted image 20220604032041.png|500]]

```ad-note
title: Link Count

- There is a field called **link count** in an inode.
	- It stores the number of directory entries pointing to the inode.
- The second column of `ls -l` is the link count of the file 
```

- Special hard links
	- The directory `.` is a hard link to itself.
	- The directory `..` is a hard link to the parent directory.
		- A large link count of a directory entry shows that it has a lot of sub-directories

- When a regular file is created, the link count is **always 1**
- When a directory is created, **the initial link count is always 2**
	- ![[Pasted image 20220604032642.png|500]]
- The hosting directory of the newly creating directory will have its **link count increased by 1**.
	- ![[Pasted image 20220604032701.png|500]]

```ad-note
title: Decrementing the Link Count / Removing a File
- The system call that removing a file is, therefore, called `unlink()`.
	- The `unlink()` system call is to decrement the link count by exactly one.
	- When the **link count == 0**, the **data blocks** and the **inode** will all be de-allocated by the kernel.

![[Pasted image 20220604033455.png|500]]
![[Pasted image 20220604033535.png|500]]
```
````

````ad-note
title: Symbolic Link
- A symbolic link is **a file**.
	- Unlike the hard link, **a new inode is created** for each symbolic link.
	- It stores the pathname (shortcut)
- How to store the pathname?
	- It is stored in the **12 direct block and the 3 indirect block pointers**.
	- Else, **one extra data block** is allocated
````

```ad-summary
title: Short Summary
- Hard link
	- **A directory entry** pointing to an existing file
	- They point to the same inode (no new file content)
	- A file with two pathname
	- Remove file == unlink (link count - 1)
	- Examples: `.`/`..`
- Symbolic link 
	- A file with a new inode 
	- Stores the target pathname 
	- Shortcuts
```

## Buffer Cache
- Kernel Buffer Cache
	- The kernel will keep a set of copies of the read/written data blocks.
	- The space that stores those blocks are called the **buffer cache**.
	- It is used for **reducing the time** in accessing those blocks in the near future
- Why effective?
	- **Principle of locality**
- What need to be cached?
	- Data blocks, directory file, inode?
	- All of them can benefit from caching
- Three types of buffer caches
	- **Page Cache**
		- It buffers the data blocks of an opened file.
	- **Directory entry (dcache) cache**
		- Directory entry is stored in the kernel.
	- **Inode cache**
		- The content of an inode is stored in the kernel temporary
	
	> Remember, those cached data is stored in the kernel even though **the corresponding file is closed**!
	> 
	> By the way, the cache is managed under the LRU algorithm.
	
- Read/write mode with kernel buffer cache
	- **Reading mode**: When a process reads a file, the data will be cached automatically.
		- E.g., the `readahead()` syscall
		- `ssize_t readahead(int fd, off64_t offset, size_t count);`
			- A **blocking system call** that stores requested range of data into the kernel page caches
			- Later `read()` calls aver the range **will not block**
	- **Write-through mode**: Both the on-disk and the cached copies update together.
		- E.g., The `write()` system call will not return until the on-disk copy is written.
	- **Write-back mode**: When a piece of data is going to be written to a file, the **cached copy is updated first**. The update of the on-disk copy is delayed.
		- **On-demand writing dirty blocks back.**
		- Command: `sync`
		- Syscall: `sync()`, `fsync()`

```ad-note
title: Readahead
How does it work?
- When a file reading operation is requesting for **Block x**, there is a chance that **Block x+1** will also be needed.
- Such a chance depends on:
	- The file reading mode: sequential access or random access.
	- The file reading history: whether the process **prefers** reading sequentially or not.
- If such a chance is high, then reading a series of continuous blocks will **reduce the number of disk accesses**. Why?
	- Because the disk head is not always stopped at your desired locations.
	- Because a mechanical disk is good at reading sequential data.
	- How about SSD?
```

## Journaling
```ad-note
title: File System Consistency
- Think about caching tradeoff?
	- System inconsistency exists
		- Power failure, pressing reset button accidentally; etc.
- Disk only provides
	- **atomic** write of **one sector at a time**
- A write may require modifying several sectors
	- How to atomically update file system from one consistent state to another?
- The **file system journal** is the current, stateof-the-art practice.
```

```ad-example
title: Example: Journaling File System
![[Pasted image 20220604035651.png]]
```

````ad-note
title: Journal
![[Pasted image 20220604035905.png|500]]
- A journal is the log book for the file system.
	- It is kept inside the file system, i.e., inside the disk.
- In database: **Write-ahead logging**
- In file systems: **Journaling**
	- Applications: Linux ext3 and ext4, Windows NTFS
- **Basic idea**: when updating the disk, before overwriting the structures in place, first write down **a little note** describing what you are about to do
- In order to make use of the journal:
	- A set of file system operations becomes an atomic **transaction**.
		- Either all operations are completed successfully, or
		- no operation is completed.
	- A transaction marks all the changes that **will be done** to the FS
	- Every transaction is written to the journal.
````

````ad-note
title: Journaling in Linux ext3
![[Pasted image 20220604040301.png|500]]
- How does Linux ext3 incorporate the journaling?
	- Most of on-disk structures are identical to Linux ext2
	- The new key structure is the journal itself
	- It occupies some small amount of space within the partition or on another device
````


### Data Journaling
```ad-question
title: How to do journaling?

- **Task**: Update inode `I[v2]` , bitmap `B[v2]`, and data block `Db` to disk
	- Metadata + data
- **Strategy**: **Data journaling**
	- Write all data (metadata+data) to journal
		- Before writing them to their final disk locations, we first write them to log (a.k.a. journal)
	- An available mode with the Linux ext3 file system
```

```ad-note
title: Journal Layout
![[Pasted image 20220604040917.png|500]]
- TxB: Transaction begin block
	- It contains some kind of **transaction identifier (TID)**
- TxE: Transaction end block
	- Marker of the end of this transaction
	- It also contain the TID
- **Checkpoint**
	- Overwrite the old structures in the file system after the transaction being safely on disk
```

````ad-note
title: Operation sequence
1. **Journal write**
	- Write the transaction to log and wait for these writes to complete
	- TxB, all pending data, metadata updates, TxE
2. **Checkpoint**
	- Write the pending metadata and data updates to their final locations

- **Problem**: What if crash occurs during the writes to journal
- We need to write the set of blocks (TxB, `I[v2]`, `B[v2]`, `Db`, TxE)
	- **Issue one block at a time**
		- It is slow because of waiting for each to complete
	- **Issue all blocks at once**
		- Five writes -> a single sequential write: Faster way
		- However, it is unsafe
			- The disk internally may perform scheduling and complete small pieces of the big write **in any order**
		- **Suppose**: disk internally
			1. writes TxB, `I[v2]`, `B[v2]`, TxE and later
			2. writes `Db`
			- When crash occurs during the writes to journal
				- If the disk loses power between (1) and (2)
			- **Problem**: Transaction looks like a valid transaction, but **the file system can’t look at the `Db` block and know it is wrong**.
	- Solution: **Issue transactional write in two steps**
		1. writes all blocks except **the TxE block** to journal
		2. file system issues the write of the TxE
		> Make sure the write of TxE is atomic
		
---
1. **Journal write**
	- Write the contents of the transaction (including TxB, metadata, and data)
2. **Journal commit**
   - metadata, and data (including TxE)
3. **Checkpoint**
	- Write the contents of the update to their on-disk locations

> The write order must be guaranteed
````

````ad-note
title: Recovery
1. Crash happens **before journal commit**
	- Easy! Skip the pending update
2. crash happens **after journal commit, but before checkpoint**
	- Replay transactions in order. Called redo logging
````

````ad-note
title: Free Log
- The log is of finite size
	- What problems may arise if it is full?
		- Long time to replay
		- Unable to append new transactions
- Manage as a **circular log**
	- **Free** space after checkpointing

![[Pasted image 20220604043036.png|500]]
````

````ad-summary
title: Solution So Far
- Write sequence
![[Pasted image 20220604043114.png|500]]
- Data Journaling Timeline![[Pasted image 20220604043153.png|500]]
````

```ad-note
title: Metadata Journaling
- Any problem with data journaling?
	- Write every `Db` to disk **twice**
		- Commit to log (journal file)
		- Checkpoint to on-disk location
- How to avoid writing twice?
	- **Metadate Journaling**: Logging metadate only

![[Pasted image 20220604043538.png|500]]

- **Write-back mode**: no order restriction (data/journal)
	- How about data is written to disk after journal commit?
		- File system is consistent (from the perspective of metadata)
		- Metadata points to garbage data
- **Ordered mode**
	- Data is written to file system **before** journal commit
	- Rule:
		- **Write the pointed-to object before the object that points to it**
		- Core of crash consistency
	- Widely deployed by Ext3, NTFS, etc.
```

```ad-summary
title: Final Solution
![[Pasted image 20220604043828.png|500]]![[Pasted image 20220604043153.png|500]]
```

```ad-summary
- Working principle:
	- All the changes to the FS are **written to the journal first**
		- the changes in the metadata, i.e., information other than the file content. E.g., the inodes, the directory entries, etc.
		- **the file data (depends on data journaling/metadata journaling)**
	- Then, the system call returns to the user process.
	- Meanwhile, **the entries in the journal are replayed** and the changes are reflected to the actual file system.
```

# Virtual File System (VFS)
![[Pasted image 20220604044131.png|400]]
- Old days: “the” file system
- Nowadays: many FS types and instances co-exist

```ad-note
title: VFS
VFS: an FS abstraction layer
- Transparently and uniformly supports multiple FSes
- A VFS specifies an interface
- A specific FS implements this interface
```
- Let’s look into the implementation of `open()`. 
	- http://lxr.linux.no/linux-old+v2.4.31/fs/open.c

```c
/*
 * This is line 710 - 714 of open.c
 * - The type of f is struct file
 * - The type of f_op is struct file_operations
 *     ┌───────────────────────────┐
 *     │ struct file_operations {  │
 *     │     loff    (*llseek)...  │
 *     │     ssize_t (*read)...    │
 *     │     ...                   │
 *     │     int (*open)...        │
 *     │     ...                   │  
 *     │ };                        │
 *     └───────────────────────────┘
 */
if (f->f_op && f->f_op->open) { 
    error = f->f_op->open(inode,f);
    if (error)
        goto cleanup_all;
}
```

- For each file system, they have their own set of file operations.
	- The VFS layer just invokes them.
	- FAT32 Methods: `fat_file_operations` http://lxr.linux.no/linux-old+v2.4.31/fs/fat/file.c#L26
	- Ext3 Methods: `ext3_file_operations` http://lxr.linux.no/linux-old+v2.4.31/fs/ext3/file.c#L113
- So, the beauty in such design is that:
	- The caller, i.e. the VFS layer, doesn’t need to care about nor hard-coding which FS you are working on.
	- The only things that require hard-coding are:
		- The definition of the file operations.
		- The assignment of file operation structures for each FS.
- A follow-up question is:
	- What if a FS does not support a particular subset of operations?
	- E.g., FAT32 does not need to implement `chmod()`!
	- Solution: using `NULL` pointers.
		- When a `NULL` pointer to a file is detected, returning an error or proceed without any changes.