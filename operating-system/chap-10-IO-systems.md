#计算机 #底层 #软件 

# Overview
- I/O management is a major component of operating system design
	- Important aspect of computer operation 
	- I/O devices vary greatly 
	- Various methods to control them 
	- Performance management
- Ports, buses, device controllers connect to various devices
- **Device drivers** encapsulate device details
	- Present uniform device-access interface to I/O subsystem

# I/O Hardware
- Incredible variety of I/O devices
	- Storage 
	- Transmission 
	- Human-interface
- Common concepts
	- **Ports**: connection point for device
	- **Bus**: **daisy chain** or shared direct access
		- **PCI** bus common in PCs and servers, PCI Express (PCIe)
		- **expansion bus** connects relatively slow devices
	- **Controller** (**host adapter**): electronics that operate port, bus, device 
		- Sometimes integrated 
		- Sometimes separate circuit board (host adapter)

![[Pasted image 20220604050611.png#center|A Typical PC BUS Structure|500]]

```ad-question
title: How to control devices?
Devices usually have registers where device driver places commands, addresses, and data to write, or read data from registers after command execution
- Data-in register
- Data-out register
- Status register
- Control register
```

```ad-question
title: How to communicate with controller?
Devices have addresses, used by
- Direct I/O instructions
- **Memory-mapped I/O**
	- Device data and command registers mapped to processor address space
```

![[Pasted image 20220604050900.png#center|Device I/O Port Locations on PCs (partial)|500]]

## Polling
For **each byte** of I/O
1. Read busy bit from status register until 0  
2. Host sets read or write bit and if write copies data into data-out register 
3. Host sets command-ready bit 
4. Controller sets busy bit, executes transfer 
5. Controller clears busy bit, error bit, command-ready bit when transfer done

Step 1 is busy-wait cycle to wait for I/O from device 
- Reasonable if device is fast 
- But inefficient if device is slow
## Interrupts
- CPU **Interrupt-request line** triggered by I/O device
	- Two lines: **Maskable** and **nonmaskable** interrupt
	- Checked by processor after each instruction
- **Interrupt handler** receives interrupts
- **Interrupt vector** to dispatch interrupt to correct handler 
	- Context switch at start and end 
	- Based on priority, some are **nonmaskable** 
	- Interrupt chaining if more than one device at same interrupt number
- Interrupt mechanism also used for **exceptions**
	- Terminate process, crash system due to hardware error
- Page Fault
	- executes when memory access error
- System call
	- executes via software interrupt or trap to trigger kernel to execute request

![[Pasted image 20220604051514.png#center|Intel Pentium Processor Event-Vector Table|500]]

## Direct Memory Access
- Used to avoid **programmed I/O** (one byte at a time) for large data movement
	- Require **DMA** controller
	- Bypasses CPU to transfer data directly between device & memory

```ad-question
title: How to work?
- OS writes DMA command block into memory
	- Source and destination addresses
	- Read or write mode
	- Count of bytes
- Writes location of command block to DMA controller, then CPU can continue to execute other tasks
- DMA controller masters bus and does the transmission without CPU
	- DMA-request and DMA acknowledge between DMA controller and device controller
```


# Application I/O interface
![[Pasted image 20220604052125.png|400]]
- Devices vary in many dimensions
	- **Character-stream** or **block** 
	- **Sequential** or **random-access**
	- **Synchronous** or **asynchronous** 
	- **Sharable** or **dedicated** 
	- **Speed of operation** 
	- **read-write**, **read only**, **write only**
- How to provide a standard and uniform I/O interface?
	- **Abstraction**, **encapsulation**, **layering**

![[Pasted image 20220604052225.png#center|A Kernel I/O Structure|400]]

## I/O Devices
- **Block devices** include disk drives
	- Commands include `read`, `write`, `seek`
	- **Raw I/O**, **direct I/O**, or file-system access
	- Memory-mapped file access possible
		- File mapped to virtual memory and clusters brought via demand paging
	- DMA
- **Character devices** include keyboards, mice, serial ports
	- Commands include `get()`, `put()`
- **Network devices**
	- **socket** interface

## Clocks and Timers
- Functionalities of hardware clock and timer
	- Get current time 
	- Get elapsed time 
	- Timer
- **Programmable interval timer** used for timings, periodic interrupts
	- Process scheduler: interrupt when time quantum is zero
	- I/O subsystem: periodic flush

## Two I/O Methods
![[Pasted image 20220604052820.png|500]]

# Kernel I/O Subsystem
- **Kernel I/O subsystem provides many services**
- **I/O scheduling**
	- Maintain a per-device queue 
	- Re-ordering the requests 
	- Average waiting time, fairness, etc.
- **Buffering** - store data in memory while transferring between devices 
	- To cope with device speed mismatch 
	- To cope with device transfer size mismatch 
	- To maintain "copy semantics" (e.g., copy from application’s buffer to kernel buffer)
- **Caching** - faster device holding copy of data 
	- Always just a copy 
	- Key to performance 
	- Sometimes combined with buffering
- **Spooling** - hold output for a device 
	- If device can serve only one request at a time, e.g., Printing 
- **Error handling** and **I/O protection** 
	- OS can recover from disk read error, device unavailable, transient write failures
	- All I/O instructions defined to be privileged 
- **Power management**, etc.