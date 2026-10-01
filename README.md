# Operating System - Day 1

## Overview

This document covers the fundamentals of operating systems with a focus on high-level interview preparation. The goal is to understand how an application interacts with the OS, kernel, hardware, and CPU privilege levels.

---

## Today's Goal

By the end of today, you should be able to explain:

```text
Application → System Call → Kernel → Hardware → Kernel → Application
```

And answer questions on:

- Operating System
- Kernel
- User Space
- Kernel Space
- User Mode
- Kernel Mode
- System Calls
- Interrupts
- Mode Switching
- System Call vs Function Call
- Interrupt vs System Call

---

## 90-Minute Plan

| Time | Activity |
|------|----------|
| 0-10 min | OS architecture and big picture |
| 10-25 min | Kernel, User Space, and Kernel Space |
| 25-45 min | User Mode and Kernel Mode |
| 45-60 min | System Calls |
| 60-70 min | Interrupts |
| 70-80 min | Practical experiment |
| 80-90 min | Interview questions and mini test |

---

## Part 1: What is an Operating System?

Do not memorize this vague definition:

> "An OS is system software that manages hardware and software resources."

Instead, understand the real meaning.

Imagine a laptop with these layers:

```text
        USER
         ↓
┌────────────────┐
│ Applications   │
│ VS Code        │
│ Chrome         │
│ Python         │
└────────┬───────┘
         ↓
┌────────────────┐
│ OS / Kernel    │
└────────┬───────┘
         ↓
┌────────────────┐
│ Hardware       │
│ CPU RAM Disk   │
└────────────────┘
```

Applications should not directly control hardware. The OS acts as the controlled intermediary between applications and hardware.

### Example

```python
with open("data.txt") as f:
    data = f.read()
```

Your Python program does not directly control the disk. Conceptually:

```text
Python program
      ↓
Python runtime / libraries
      ↓
OS system call
      ↓
Kernel
      ↓
File system / storage driver
      ↓
Disk
```

The OS manages access to resources.

---

## Part 2: Kernel

### What is the Kernel?

The kernel is the core part of the operating system that runs with high privileges and manages system resources.

It handles:

- CPU and process management
- Memory management
- File systems
- Devices
- Networking
- Security and permissions

Think of the kernel as the part of the OS that has direct responsibility for managing resources.

### Important Distinction

Do not say:

> Kernel = Operating System

A more accurate statement is:

> The kernel is the core component of an operating system.

An OS includes the kernel plus other components such as:

- system utilities
- libraries
- services
- user-facing interfaces

---

## Part 3: User Space and Kernel Space

This is important for interviews.

Modern operating systems separate memory and address spaces so that normal applications cannot freely access critical kernel resources.

```text
┌──────────────────────────────┐
│         USER SPACE           │
│                              │
│ Chrome                       │
│ VS Code                      │
│ Python                       │
│ Node.js                      │
└──────────────┬───────────────┘
               │
           System Calls
               │
┌──────────────▼───────────────┐
│        KERNEL SPACE          │
│                              │
│ Process Management           │
│ Memory Management            │
│ File Systems                 │
│ Device Drivers               │
│ Networking                   │
└──────────────┬───────────────┘
               ↓
            Hardware
```

### User Space

This is where normal applications execute.

Examples:

- Python program
- Node.js application
- Chrome
- VS Code

### Kernel Space

This is where the kernel and privileged OS components execute.

It has access to protected system resources.

---

## Part 4: User Mode vs Kernel Mode

This is one of the most common OS interview topics.

Modern CPUs provide privilege levels or modes so ordinary application code cannot freely execute privileged operations.

### User Mode

Application code normally runs here.

It has restricted access.

For example, an application should not be able to:

- modify kernel memory
- control hardware directly
- disable system protection mechanisms

### Kernel Mode

The kernel runs here.

It has much greater privileges and can perform protected operations.

### Why Do We Need Two Modes?

Imagine this Python program:

```python
while True:
    pass
```

What if every program could directly:

- overwrite another program's memory
- access any disk sector
- modify kernel memory
- control hardware

One buggy or malicious program could crash or compromise the entire system.

So the flow is:

```text
Application
    ↓
Restricted User Mode
    ↓
Request OS service
    ↓
Kernel Mode
    ↓
Perform privileged operation
    ↓
Return to User Mode
```

This separation provides protection and isolation.

---

## Part 5: Mode Switching

When an application needs the OS to perform a protected operation, the CPU switches privilege levels.

```text
USER MODE
   │
   │ System Call
   ↓
KERNEL MODE
   │
   │ Kernel performs operation
   ↓
USER MODE
```

The transition is controlled by both the CPU and the OS.

### Example

Your application wants to read a file:

```text
Application
     ↓
read request
     ↓
System Call
     ↓
CPU switches privilege level
     ↓
Kernel
     ↓
File system / driver
     ↓
Storage
     ↓
Kernel
     ↓
Return result
     ↓
Application
```

This is a useful mental model for interviews.

---

## Part 6: System Calls

A system call is a controlled interface through which a user-space program requests a service from the operating system kernel.

Examples on Unix-like systems include:

- `open()`
- `read()`
- `write()`
- `close()`
- `fork()`
- `exec()`
- `wait()`

You do not need to memorize all of them. Understand the categories.

### Categories of System Calls

#### Process management

- `fork()`
- `exec()`
- `wait()`

#### File management

- `open()`
- `read()`
- `write()`
- `close()`

#### Memory-related services

Examples include operating-system interfaces for memory mapping and allocation.

#### Communication

System calls also support mechanisms used for process and network communication.

### System Call Flow

Suppose:

```c
read(fd, buffer, size);
```

Conceptually:

```text
Your application
      ↓
read()
      ↓
System-call interface
      ↓
CPU enters privileged execution
      ↓
Kernel
      ↓
File system / device driver
      ↓
Storage / device
      ↓
Kernel prepares result
      ↓
Return to application
```

### Important Point

A system call is not simply another ordinary function call.

It involves crossing from user-space execution into the protected kernel interface.

### System Call vs Function Call

| Aspect | Function Call | System Call |
|--------|---------------|-------------|
| Scope | Calls a function in program or library | Requests a kernel service |
| Execution | Usually stays in user space | Crosses into kernel-controlled execution |
| Cost | Usually cheaper | Generally has more overhead |
| Privilege | No privilege transition required | Involves controlled entry into kernel |
| Example | `calculateTotal()` | `read()` |

### Interview Answer

> A normal function call transfers control to another function within the program or its libraries. A system call is a controlled request from user-space software to the OS kernel for a service that requires OS privileges or kernel involvement.

---

## Part 7: Interrupts

An interrupt is a mechanism that causes the CPU to temporarily stop its current execution flow and handle an event that requires attention.

```text
CPU executing program
        ↓
Hardware event occurs
        ↓
Interrupt
        ↓
CPU handles interrupt
        ↓
Interrupt handler
        ↓
Resume previous execution
```

### Example

A keyboard key is pressed.

```text
Keyboard
   ↓
Hardware interrupt
   ↓
CPU
   ↓
OS interrupt handler
   ↓
Keyboard input processed
   ↓
Program can receive input
```

### System Call vs Interrupt

Do not confuse them.

#### System Call

Usually initiated intentionally by software to request an OS service.

Example:

```text
Application → read()
```

#### Hardware Interrupt

Usually generated by hardware to notify the CPU about an event.

Example:

```text
Keyboard → CPU
```

There are also software-generated exceptions and traps, so avoid saying that all interrupts come from hardware.

---

## Part 8: Practical Experiment

Now do not only read the theory.

If you are using Windows, you can use WSL if available.

Run the following commands:

```bash
ps
ps aux
top
```

If `top` is not available, try:

```bash
htop
```

You are looking at processes currently running on the system.

Try:

```bash
sleep 100
```

Open another terminal and run:

```bash
ps
ps -ef | grep sleep
```

### What are you learning?

You are connecting the theory to real system behavior:

```text
Application
    ↓
Process
    ↓
OS manages process
    ↓
Kernel maintains process information
```

Do not worry if you do not understand every column yet. Processes will be explained in depth later.

### Practical Experiment 2: Observe CPU and Memory

Run:

```bash
top
```

Look at:

- PID
- CPU %
- Memory %
- Process name

Ask yourself:

> Who is managing these processes?

The answer is the OS kernel.

We will later learn exactly how scheduling and memory management work.

---

## Most Asked Interview Questions

### Q1. What is an Operating System?

**Answer:**

An operating system is system software that manages hardware and system resources and provides services and abstractions that allow applications to run safely and efficiently.

### Q2. What is a kernel?

The kernel is the core component of an operating system that manages resources such as CPU, memory, devices, and file systems and provides controlled services to applications.

### Q3. Why do we need kernel mode and user mode?

They provide privilege separation. Applications run with restricted privileges, while the kernel runs with higher privileges to perform protected operations. This prevents applications from directly accessing critical system resources.

### Q4. What is a system call?

A system call is a controlled interface through which a user-space program requests a service from the operating system kernel.

### Q5. Give examples of system calls.

Examples include:

- `open()`
- `read()`
- `write()`
- `close()`
- `fork()`
- `exec()`
- `wait()`

### Q6. What happens when a system call is made?

A good high-level answer:

The application requests a kernel service through the system-call interface. The CPU enters the appropriate privileged kernel execution path, the kernel performs the requested operation, and the result is returned to the application.

### Q7. What is an interrupt?

An interrupt is a mechanism that causes the CPU to handle an event that requires attention, after which normal execution can resume.

### Q8. System call vs interrupt?

A system call is normally a software-initiated request for an OS service, whereas an interrupt is a notification mechanism for an event requiring CPU attention. Hardware devices commonly generate interrupts.

### Q9. Why can't a Python program directly access hardware?

Because unrestricted hardware access would compromise system protection and isolation. The program requests OS services through controlled interfaces such as system calls.

### Q10. Does every function call switch to kernel mode?

No. A normal function call generally remains in the current execution mode. A system call is different because it enters the OS's controlled kernel interface.

### Q11. Is `printf()` itself a system call?

Not exactly. `printf()` is a library function. Depending on the implementation and buffering, it may eventually cause a system call such as `write()` to send output to a file descriptor.

```text
printf()
   ↓
C library
   ↓
possibly write()
   ↓
Kernel
```

### Q12. Why are system calls more expensive than normal function calls?

Because entering the kernel involves additional controlled execution and privilege-transition mechanisms, plus kernel work, rather than simply transferring control to another user-space function.

### Q13. What happens if a user application tries to execute a privileged operation?

The CPU's protection mechanisms prevent unauthorized execution, typically resulting in a fault or exception rather than allowing the application to perform the operation.

### Q14. Why is kernel space protected?

Because the kernel controls critical system resources. If ordinary applications could freely modify kernel memory, a single bug could potentially corrupt the entire system.

---

## Scenario-Based Questions

### Scenario 1

Your Node.js application needs to read a file. Explain the OS-level flow.

```text
Node.js application
      ↓
Runtime / library
      ↓
OS interface
      ↓
Kernel
      ↓
File system
      ↓
Storage
      ↓
Result
```

### Scenario 2

Two applications are running. One tries to modify the memory belonging to the other. What should happen? Why?

Expected concept:

- process isolation
- virtual memory
- OS and CPU protection

### Scenario 3

A keyboard key is pressed while the CPU is executing another program. How does the CPU become aware of it?

```text
Keyboard
   ↓
Interrupt
   ↓
CPU
   ↓
OS interrupt handling
```

---

## Day 1 Mini Test

### Questions

1. Which component directly manages CPU and memory resources?
   - A. Browser
   - B. Kernel
   - C. Compiler
   - D. Text editor

2. Why does user mode exist?
   - A. To make programs faster
   - B. To restrict applications from performing privileged operations
   - C. To increase RAM
   - D. To compile programs

3. Which is closest to a system call?
   - A. `calculateSum()`
   - B. `sortArray()`
   - C. `read()`
   - D. `printName()`

4. Which usually initiates a system call?
   - A. Application software
   - B. Keyboard hardware
   - C. Monitor
   - D. RAM

5. Which commonly generates hardware interrupts?
   - A. Keyboard
   - B. `if` statement
   - C. Variable assignment
   - D. Function definition

6. Does every function call enter kernel mode?
   - A. Yes
   - B. No

7. `printf()` is best described as:
   - A. Always a direct system call
   - B. A library function that may eventually cause system calls for output
   - C. A hardware interrupt
   - D. A kernel

8. What protects the kernel from unrestricted application access?
   - A. HTML
   - B. Privilege separation and hardware/OS protection mechanisms
   - C. CSS
   - D. Database

### Answers

1. B  
2. B  
3. C  
4. A  
5. A  
6. B  
7. B  
8. B  

---

## Key Takeaways

- The OS acts as an intermediary between applications and hardware.
- The kernel is the core privileged component of the OS.
- User space and kernel space are separated for protection and isolation.
- User mode limits application access; kernel mode allows privileged operations.
- System calls are the controlled bridge between user programs and OS services.
- Interrupts are hardware or software signals that require CPU attention.

This is the foundation for deeper topics such as processes, scheduling, memory management, and synchronization.
