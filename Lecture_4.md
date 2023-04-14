# Lecture 4 - OS Organization
### Layers of a computer

- aps
- kernel
- hardware

However, we can say there's an abstract layer betwen apps and the kernel,
that create an abstract machine

This abstract machine / layer which allow us to use the machine without worrying
about the kernel

The approach we are looking at today is **virtualization**
- extra instructions
- system calls 

Assuming x86-64 system call Linux

`SYSCALL` instruction
- privileged instruction

## From the point of view of the user code

- arguments go in `rax rdi rsi rdx r10 r8 r9` (64 bit args)
- if we want to pass an int into the kernel, pass it into a register
- if we want to pass a block of memory into the kernel, have one point to 
    the memory
- syscall # goes in `rax`
- notice that there is no concept of a stack here
    - we don't use the stack because it is too slo

## From inside the kernel

- kernel then issues a `SYSRET` instruction which brings you back to where
  you were 
- `rax` tells you what the kernel is returing
    - if it returns -1 through -4095, it means an error 
- `r11` is trashed

## More on System Calls

- System calls are like function calls, but they are a bit more of a hassle
    - less efficient
    - more state needs to be saved

On the other hand, using a **client server approach**
- pros
- client and server are both insulated from each other
    - both can't mess up each other
- cons
- since we have harder modularity, (client and server can't see each other's memory)
- no fast transfer of data


```c
// open function could be implemented as:
int open(char const *f, int flags) {
    asm("move f into $rdi");
    asm("move flags into $rsi");
    asm("movsqsl eax, rax");
    asm("movq   $12, %rax");
    asm("syscall")
}
```

<hr>

# OS organization (main topic today)

We want to take our big application and split it up into a bunch of modules 
such tht the combination of these modules will work together and solve the
problem 

### Aproaches

- we don't use OO to build OS's because
    - too monolithic (one bug can crash the entire thing)
    - sometimes slow (but not the main reason)

## 3 fundamental system abstractions

1. memory
    - RAM - big array of bytes
    - however, the details differ, including
        - size
        - speed
        - word size (32 bytes? 64?)
        - volatility 
            - (does the memory change spontaneously? or just to us 
          explity adding and removing things)
        - coherence 
            - what if two cpu's are loading into the cpu at the same time?
        - addressing
            - linear addressing? or addressed by contents?
            - we will focus on the former in this course
    - how do the kernel and user programs access memory? how do these interact?
2. Interpreter
    - we have a program that interprets another machine
    - for example, we have a c program that emulates 386 (32 bit machine)
    - the way we specify how an interpreter should behave 
    - interpreter state 
    1
    - repertoire:
        - what do the instructions do?
        - what does `0xcd` do?
    - `ep` (environment/stack pointer)
        - points into memory
3. Communications Link
    - used heavily by client serve systems 
        - send/receive
    - io bus
    - most advanced topic for virtualization

## Layering

2
- kernel walls off much of the instruction repertoire 
- `lsmod | less` shows all of the kernel modules 
- `lsmod | wc` shows how many modules we have 
- `/proc`
    - files inside proc don't really exist
        - provided by the kernel to tell us details about the kernel
    - way for the kernel to send our processes messages when our kernel is
      interested in something 
- `/proc/modules`
    - "has 0 bytes"
    - kernel dynamically constructs the contents of the file upon request
    - tells you what modules are currently active 
- `/proc/self`
    - info about the currently running processes 
- `ls -l /proc/self/fd`
- `ps -ef | wc`
    - counts out how many running processes are there
- `strace`
    - tells us how ps is telling us about the process status 
    - 

### Using virtualization, we are building a pretend machine on top of a real one
Apps are insulated from each other
- they don't see each other's hardware
- but apps need to use registers, chace, ram, IO


Registers *- Level 1*

When an app wants to access and change/access a regsiter, such as
- `movq rax rbx`
- full access to registers

Cache
- supposed to be invisible
- instruction repetoire never mentions it 
- hardware is just supposed to handle it 
- even though we are not supposed to care, cache makes our prorgam go faster
- caches are a security problem however

Ram *- Level 2*
- each app with have a pretend machine with enough memory for the app
- virtual memory
- some sort of MMU (Memory Management Unit)
- app's memory have virtual addresses that are mapped to physical addresses

IO devices *- Level 3*
- in virtual machines, programs are not generally allowed to do i/o
    - if you try to use `outb` your program will crash

### Registers Access
Currently running application actually does have access to the registers
- `%rip %rax %rip ...`

All of the other process think that they have access, but they don't

- We achieve this a process table
- In it's simplist model, it's a 2d array

If we want to change processes, we need to use a context switch

cr3 - tells the hardware what the mapping is from virtual page number
      to physical page number 
- who ever controls this register controls what the cpu believes to be memory
- page tables may have multiple levels if they need to access multiple 
  parts of memory

### IO 
- Completely probited without a `syscall`
- this is kinda slow, but if you're doing io anyways this shouldn't be too much
  of an issue 
- can be done in software 

### Two main types of IO devices
| stream devices | data storing devices |
|-|-|
| keyboard | flashdrive |
| mouse | disk |
| network interface | |
| | |
| spontaneous data generation | requests todo io / responses |
| (infinite) | (finite)|
| | random access |

## What can go wrong with access to regs, mem, and io?
regs <br>
okay! affects only that program

memory
- usually app crashes
- not that bad, other apps are okay

io
- More dangerous. Can affect other applications

kernel
- if there's a bug in the kernel, **"you are toast"**

app loops
- problem that needs a solution (will talk later)