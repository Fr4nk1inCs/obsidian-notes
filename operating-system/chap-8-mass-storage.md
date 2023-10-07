#计算机 #底层 #软件 

# Disk Structure
## Hard Disk Structure
```ad-note
title: Physical View
![[Pasted image 20220504094323.png|400]]
- Physical Address
	- **Track**: The surface of a platter is divided into tracks
	- **Sector**: Track is divided into sectors (512B data + ECC)
	- **Cylinder**: Set of tracks that are at one arm position
- **Access**: Seek + Rotation
	- **Seek**: Move disk arm to desired cylinder.
	- **Rotation**: Spin at 5400/7200/10K/15K RPM
- **Constant Liner Velocity** (**CLV**)
	- Uniform density of bits per track, outer track hold more sectors
	- Variable rotation speed to keep the same rate of data moving
	- **CD-ROM**/**DVD-ROM**
- **Constant Angular Velocity** (**CAV**)
	- Constant rotation speed
	- Higher density of bits in inner tracks
	- **Hard Disks**
```
```ad-note
title: Logical View
- How to use?
	- Large 1-Dimension arrays of logical blocks (usually 512 bytes)
- **Address Mapping**
	- Logical Block Number → Cylinder Number, Track Number, Sector Number
- **Disk Management** is required
	- Disks are prone to failures: defective sectors are common (bad blocks)
		- Need to handle defective sectors: **bad block management**
	- **Disk Formatting**
```

## Disk Management
```ad-note
title: Bad Block Management
- Maintain a list of bad blocks (initialized during low-level formatting) and preserve an amount of spare sectors
- **Sector Sparing/Forwarding**: Replace a bad sector logically with one spare sector
	- Problem: Invalidate disk scheduling algorithm
	- Solution: Spare Sectors in each cylinder + spare cylinder
- **Sector Slipping**: remap to the next sector (data movement is needed)
```
```ad-note
title: Disk Formatting
2 Steps
1. **Low-level Formatting/Physical Formatting**
	- Divide into sectors so disk controller can read/write
	- Fills the disk with a special data structure for each sector
		- data area (512B)
		- header & trailer (sector number & ECC)
		- The controller automatically does the ECC processing whenever a sector is read/written
	- Done at factory, used for testing and initializing (e.g., the mapping). It is also possible to set the sector size (256B, 512B, 1K, 4K)
2. **How to use disks to hold files after shipment?**
	- **Choice 1: File System**
		- Partition into one or more groups of cylinders (each as a separate disk)
		- Logical formatting: creating a FS by storing the initial FS data structures
		- I/O optimization: Disk I/O (via blocks) & file system I/O (via clusters), why?
			- More sequential access, fewer random access
	- **Choice 2: Raw Disk**
		- Use disk partition as a large sequential array of logical blocks, without FS
		- Raw I/O: bypass all FS services (buffer cache, prefetching …), be able to control exact disk location
```

# Disk Scheduling
```ad-question
title: Why needed?
- Requests are placed in the queue of pending requests for that drive if the drive/controller is busy
- Request ordering significantly affects the access performance (seek + rotate), so scheduling is needed
```
```ad-note
title: Disk Scheduling
- I/O access procedure
	- Seek: move the head to the desired cylinder
	- Rotate: spin to the target sector on the track
- Disk scheduling
	- Choose the next request in the pending queue to service so as to minimize the **seek time**
- Scheduling Algorithms
```
## Algorithms
We will take the request queue below to illustrate each algorithm.
```plaintext
head -> {98, 183, 37, 122, 14, 124, 65, 67}
The head starts at 53.
```
````ad-note
title: FCFS Scheduling
- First-come, first-served (FCFS)
	- Intrinsically fair, but does not provide the fastest service
```ad-example
![[Pasted image 20220504101401.png|400]]
![[Pasted image 20220504101432.png#center|Scheduling Diagram|400]]

- Total heal movement: 640 cylinders
```
- Wild swing is very common
	- E.g.: 122 → 14 → 124
```ad-question
title: How to reduce the head movement?
Handle nearby requests first.
```
````
````ad-note
title: SSTF Scheduling
- Shortest seek time first (SSTF)
	- Choose the request with the least seek time
	- Choose the request closest to current head position

```ad-example
![[Pasted image 20220504102141.png|400]]
![[Pasted image 20220504102154.png#center|Scheduling Diagram|400]]
- Total head movement: 236 cylinders
```
- Essentially a form of SJF scheduling
- It is not optimal
	- The sequence of 53-37-14-65… could reduce the head movement to 208
- It may cause **starvation**
````
````ad-note
title: SCAN Scheduling
- Scan back and forth
	- Starts at one end, moves toward the other end
	- Service the requests as it reaches each cylinder
	- Reverse the direction
	- **Elevator Algorithm**
```ad-example
Suppose the head is moving from 53 to 0
![[Pasted image 20220504102741.png|400]]
![[Pasted image 20220504102806.png#center|Scheduling Diagram|400]]
- Total head movement: 236 cylinders
```
```ad-warning
title: Problem
Assume a uniform request distribution
- The heaviest density of requests is at the other end of the disk
- They need to wait for a long time
```
````

````ad-note
title: C-SCAN Scheduling
- Circular Scan back and forth
	- A variant of SCAN: immediately return when reaches the end
	- Aim for providing a more uniform wait time

```ad-example
![[Pasted image 20220504103541.png|400]]
![[Pasted image 20220504103827.png#center|Scheduling Diagram|400]]
There are unnecessary movement.
```
```ad-question
title: How to improve?
- No need to move across the full width of the disk, but only need to reach the final request
- Improved SCAN and C-SCAN: LOOK and C-LOOK
```
````
````ad-note
title: C-LOOK Scheduling
- Goes only as far as the final request
	- Look for a request before continuing to move in a given direction
```ad-example
![[Pasted image 20220504104311.png|400]]
![[Pasted image 20220504104334.png#center|Scheduling Diagram|400]]
Fewer head movements than SCAN/C-SCAN
```
````
```ad-summary
- SSTF outperforms FCFS, but may suffer from starvation
- SCAN and C-SCAN perform better for heavy load systems, and they are less likely to cause starvation
```
```ad-note
title: Selection of a Scheduling Algorithm
- Disk **Performance**
	- Number and types of requests
	- File allocation method
		- Large sequential I/O or small random I/O
	- Location of directories and index blocks (metadata I/O)
- Implementing scheduling in OS is necessary to satisfy other constraints
	- E.g.: Priority defined by OS
- Write disk scheduling as a separate module of the OS
	- Can be easily replaced with different algorithm (default: SSTF/LOOK).
```
# Solid-State Drives (SSDs)
```ad-note
title: Advantages of flash-based SSDs
- Non-volatility
- Shock resistance
- High speed
- Low energy consumption
```
## SSD Architecture
```ad-note
title: SSD Components
- Multiple flash packages
- Controller
- RAM

![[temp 1.png|+grid]]![[temp 2.png|+grid]]
```
```ad-note
title: Flash Package
Package > Die/Chip > Plane > Block > Page
![[temp 3.png#center|Samsung K9XXG08UXM (SLC) (2 dies, 4 planes, 2048 blocks, 64 pages)]]
```
```ad-note
title: Flash Cell
![[temp 4.png]]
- Each cell stores one bit (or multiple bits)
- Program operation can only change the value from 1 to 0 (erase operation changes the value from 0 to 1) 
	- **No overwritten** 
- The floating gate becomes thinner as the cell undergoes more program-erase cycles 
	- **Decreasing reliability**
```
```ad-note
title: Flash Type
- NAND flash and NOR flash
	- NAND flash: denser capacity, only allow access in units of pages, faster erase operation
	- Most SSD products are based on NAND flash
- NAND flash: SLC and MLC
	- SLC: each cell stores one bit
		- Longer life time, lower access latency, higher cost
	- MLC: each cell stores two (or three) bits
		- Higher capacity
```

## * SSD Operations
```ad-note
title: Read & Write
Read/Write in unit of pages (4KB)
$$
\text{page}\longleftrightarrow\text{register}\longleftrightarrow\text{controller}
$$
- Read: $\text{page}\xrightarrow{\text{data read: 25} \mu\text{s}}\text{register}\xrightarrow{\text{serial bus: 100} \mu\text{s}}\text{controller}$
- Write: $\text{page}\xleftarrow{\text{program: 200} \mu\text{s}}\text{register}\xleftarrow{\text{serial bus: 100} \mu\text{s}}\text{controller}$
```

```ad-note
title: Erase
- Erase
	- In unit of blocks (64/128 pages)
	- Change all bits to 1
	- Much slower than read/write: 1.5 ms
- Each block can only tolerate limited number of P/E cycles
	- SLC: 100K, MLC: 10K, TLC (several K to several hundred)
- The number of maximum P/E cycles decreases when
	- More bits are stored in one cell
	- The feature size of flash cell decreases (72nm, 34nm, 25nm)
```
```ad-note
title: Overwrite & Delete
- Delete
	- Simply mark the page as invalid
- Overwrite/update
	- Does not support in-place overwrite
	- Data can only be programmed to clean pages
```
```ad-note
title: Read-Modify-Write & Trim
- RMW may require a lot of read and write operations, so it is very slow
- TRIM avoids slow RMW operation during write, so it increases write performance
	- Improve write performance degraded by RMW
		- The OS also sends a TRIM command to SSD after delete pages
		- Requires both OS and SSD to support
```

```ad-note
title: Software Layer in Controller
- How to further improve write performance?
	- Address mapping is needed
- Page states
	- Garbage collection is also necessary
- Flash translation layer

![[temp 49.svg]]
```

## * Flash Translation Layer
- Three Functionalities
	- Address mapping
	- Garbage collection
	- Wear-leveling

![[temp 5.png]]


### Address Mapping
- Sector mapping 
- Block mapping 
- Hybrid mapping 
- Log-structured mapping

```ad-note
title: Sector Mapping
Mapping table is large: requires a large amount of RAM
![[Pasted image 20220517143755.png]]
```
```ad-note
title: Block Mapping
- The logical sector offset is the same with the physical sector offset
- Smaller mapping table
- If the FS issues writes with identical lsn, many erases

![[Pasted image 20220517143818.png]]
```
```ad-note
title: Hybrid Mapping
- First use block mapping, then use sector mapping in each block
- Small mapping table 
- Avoid a lot of erase operations 
- Longer time to identify the location of a page

![[Pasted image 20220517144016.png]]
```

```ad-note
title: Log-structured Mapping
![[Pasted image 20220517144124.png]]
```

```ad-summary
- The performance of address mapping is workload dependent
	- Block mapping is suitable for sequential workloads 
	- Sector mapping is suitable for random workloads 
	- Log-structured mapping is suitable for workloads with large sequential and small random requests
- **Tradeoff exists**
```

### Garbage Collection
- Due to the existence of invalid pages, GC must be called to reclaim storage
	- Choose a candidate block
	- Write valid pages to another free block
	- Erase the original block

![[Pasted image 20220517144426.png]]

```ad-caution
title: Design Issues
- **Tradeoff** in GC design
	- Efficiency: minimize writes 
	- Wear-leveling: erase every block as even as possible 
	- Tradeoff 
	- GC is considered together with wear-leveling
- Algorithms
	- Greedy, random, and their variants
	- Hot/cold identification
```

## Other Tech
- 3D NAND flash
- Non-volatile memory (NVRAM)
	- PCM, STTRAM, ReRAM, etc…
	- Byte-addressable and non-volatile
	- 3D XPoint

# RAID & Erasure Codes
```ad-note
title: RAID Motivation
![[temp 50.svg]]
- Performance: Disks are slow
- Reliablity: One disk failure incurs data loss
- Cost: Fast and reliable disks are expensive
```
```ad-note
title: RAID Introduction
RAID: Redundant Array of Inexpensive (independent) Disks
- In the past
	- Combine small and cheap disks as a **cost-effective** alternative to large and expensive disks
- Nowadays
	- Higher performance
	- Higher reliability via redundant data
	- Larger storage capacity
- Many different levels of RAID systems
	- Different levels of redundancy, capacity, cost …
```
```ad-note
title: RAID Evaluation
![[temp 50.svg]]
- Performance: Sequential and random read/write
- Reliability: Tolerance of disk failures
- Cost: Data capacity/all capacity
```
```ad-note
title: RAID 0
![[Pasted image 20220517145649.png|400]]
- Block-level striping, no redundancy
- Provides higher data-transfer rate
- Does not improve reliability. 
	- **Once a disk fails, data loss may happen (MTTF: mean time to failure)**
```
````ad-note
title: RAID 1
```ad-question
How to improve reliability?
```
![[Pasted image 20220517145705.png|200]]
- Data mirroring (RAID1)
	- Two copies of the data are held on two physical disks, and the data is always identical.
	- Replication
- **High storage cost**
	- Twice as many disks are required to store the same data when compared to RAID 0.
	- Even worse storage efficiency with more copies
````

````ad-note
title: Combinations of RAID 0 & RAID 1
- RAID 0 provides performance and RAID 1 provides reliability
- RAID 0+1 (RAID 01)
	- First data striping
	- Then data mirroring
- RAID 1+0 (RAID 10)
	- First data mirroring
	- Then data striping

![[Pasted image 20220517150038.png|+grid]]![[Pasted image 20220517150049.png|+grid]]

```ad-warning
title: Issue
Both suffer from high storage cost!
```
````

````ad-note
title: RAID 4
```ad-question
title: How to balance the tradeoff between reliability and storage cost?
Redundancy with parities
```
![[Pasted image 20220517150536.png|250]]
- **Parity generation**
	- Each parity block is the XOR value of the corresponding data disks
	- $A_p = A_1\oplus A_2\oplus A_3$
- **Block-level data striping**
	- Data and parity blocks are distributed across disks
	- Dedicated parity disk

```ad-question
title: How to update data?
- Suppose A1 will be updated to A1'
	- Both A1 and Ap need to be updated  $$A_p' = A_p\oplus A_1\oplus A_1' = A_2\oplus A_3\oplus A_1'$$
	- Read-modify-write (RMW)
- How about updating both A1 and A2 simultaneously?
	- RMW
	- Read-reconstruct-write (RRW)
- Selection of RMW/RRW
	- **Both RMW and RRW incur extra reads and writes**
```
```ad-caution
title: Problems of RAID 4
- Disk bandwidth are not fully utilized
	- Parity disk will not be accessed under normal mode
- Parity disk may become the bottleneck
```
````
```ad-note
title: RAID 5
![[Pasted image 20220517151311.png|inlineR|300]]
- Similar to RAID 4
	- One parity per stripe
	$$
		\begin{gather}
			A_p = A_1\oplus A_2\oplus A_3\oplus A_4\\
			\vdots\\
			E_p = E_1\oplus E_2\oplus E_3\oplus E_4
		\end{gather}	
	$$
- Key difference
	- Uniform parity distribution
- RAID 5 is an ideal combination of
	- good performance 
	- good fault tolerance 
	- high capacity 
	- storage efficiency
- **Parity update overhead still exist**
```
````ad-note
title: RAID 6
```ad-question
How to tolerate more disk failures?
```
![[Pasted image 20220517151927.png|300]]
- RAID-6 protects against two disk failures by maintaining two parities
- Encoding/decoding operations
	- Based on **Galois field**
- **Parity update overhead becomes larger**

$$
\begin{align}
A_p &= A_1\oplus A_2\oplus A_3 \\
A_q &= c^0 A_1\oplus c^1A_2\oplus c^2 A_3 
\end{align}
$$
Where $A = a_n\cdots a_1a_0$, $c^1A = a_0a_n\cdots a_2a_1$, $c^2A = a_1a_0a_n\cdots a_3a_2$ and etc.
````
```ad-note
title: Erasure Codes
- Erasure Codes
	- Different redundancy levels
		- **2-fault tolerant**: RDP, EVENODD, X-Code
		- **3-fault tolerant**: STAR
		- **General-fault tolerant**: Cauchy Reed-Solomon (CRS)
- Generate **m** code blocks from **k** data blocks, so as to tolerate any **m** disk failures

![[Pasted image 20220517153134.png]]
```
```ad-summary
- The motivation to introduce erasure codes in large-scale storage systems
	- **The need to reduce the tremendous cost of storage**
- In practice, erasure codes have seen widely deployment
	- Google File System \[Ford, OSDI’10\]
	- Windows Azure Storage \[Huang, ATC’12\] 
	- Facebook \[Borthakur, Hadoop User Group Meeting 2010\]
	- …
```