## Modulularity
Breaks the big problem into smaller, more manageable pieces, whic are easier 
to change without affecting other modules

Soft Modularity:  
a form which is easier to implmenent, but relies more heavily on modules 
cooperating with each other. 

Hard Modularity:  
a more secure form of modularity which attempts to attain security through 
rigorous checks and validation, while keepign each module more "isolated" 
from one another

Process:  
PcB - Process control block, which contains all execution information
- process state
- cpu registers
- scheduling information 
- memry management info
- io status info

context switching
- swapping process is called context switching
- at min CPU has to save all teh current registers in order to restore them later

Scheduling  
Preemptive  
- interruptable
- state of a process gets changed
- process may go to the ready state from running state or from the waiting state 
to the blocked state

Types  
- first come first serve
    - long response time
    - most basic form
- shortest job first
    - assumes no preemption
- shortest remaining time first (SRTF)
    - changing sjf to run with preemption
- Round robin
    - optimizes fairness and response time, and can add priorities

Fork()  
creates a new process by duplicating the calling process (parent process)  
child process has almost the same state as the parent process, including vars  
on success
- 0 means child
- >0 means parent
- <0 means error

### IPC
transferring bytes between two or more porocess  
Signals are a form of IPC that interrupts
- you could also press cntrl c to stop your programs
- cntr c sends SIGINT 
- there is a defautl handler
- signals are only handled in kernel mode
- one line of code may combine both user mode and kernel mode

## basic bios
- covered in lec0
- when powered off mem, has nothing inside
- only hard disk has memory
- need to jump start memory

## two models
Monolithic kernel runs os services in kernel mode

A microkernel runsthe minimum amount of services in kernel mode

## Exceptions
each proesses has its own kerenl stack
- to save the user space execution sptet:
- eip, esp and other registers
- to manage the kerenl related execution for the process



Practice exam:  
1. Does Ubunto use soft or hard modularity?  
Hard, because the kernel is designed to protect itself
Soft, because user mode applications have much more soft modularity.
    - lots of system calls coordinate together, many libraries depend on
      one another

5. Scheduling
5a.  
The utiliztion is higher because we spend less time doing context switching  
they have the same average wait time because even though the response time goes 
up, we have to take longer  
The response time doubles, since 1 + 0.5 + 0.25 + 0.125 ... + 0 = 2

5b.
Starvation is not possible, as the chance of a process repeating infintely
due to luck is esentially impossible (0.5)^10 is essentially 0

Example how your solution in lab1 works  
- You call pipe
- you fork the process
- in the child, we put ls stdout in the right end of the pipe
- we execlp to hiijack the child process
- in the parent process, we take the output of the pipe and route it to stdin
- each child later closes both ends of pipe
- parent close bth ends of pipe
- wait and check status 

