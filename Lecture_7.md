# Lecture 7
## Processes vs Threads
## Processes
### fork()
child = parent, with the exceptions of a few things:
- pid
- ppid 
- file descriptors
- copy of file descriptor table
- file locks
- execution times
- pending signals

### execlp()
opposite!
<br>reusing process, so same
- pid
- ppid
- file descriptors
- etc

<br>

- however now, you now have a new program
- so new registers (they are reset to whatever appropriate value they should be)

## Threads
thread - lightweight processes
- the issue with fork is that it's kinda heavyweight
- thread creation is really cheap
- thread shares process memory with other threads
    - memory state
    - file descriptors
    - working directory (all threads will change if one thread `cd`'s)
    - signal handler table 
        - which controls how processes handle signals

But what isn't shared?:
- can't share instruction pointers (`rip`)
- stack pointers (`rsp`)
- and the rest of the registers 
    - threads might be operating on different cores with different registers
- don't share the stack 
- `threadID`'s
- `errno` global variable (most recent error shared by syscalls)
    - can't be shared because threads could compete 
    - `errno` lives in a separete space called TLS
- signal mask
    - set of signals that you are **temporarily** ignoring

### TLS (thread local storage)
- each thread has a copy of thread local storage
- each thread can access it quickly
- often system will have a certain register point towards the TLS

## Creating Threads
In the posix model, each thread belongs to a process

```c
#include <pthread.h>
/*
think of this as like fork but with more args

pthread_t * thread:
- pointer to a thread object, which is initialized by the function
- holds info about the thread

pthread_attr_t const *attr
- extra attributes about the thread

void(*start)(void *)
- function pointer
- points to the code that you want to run and create a thread of 
*/

int pthread_create(pthread_t * thread, pthread_attr_t const *attr, void*(*start)(void *))

void pthread_exit(void **)

// thread you want to wait on
// where you want return value to be stored
void pthread_join(pthread_t t, void ** value);

// lets you send signals to any thread that you like
// threads can ignore (i think)
int pthread_kill(pthread_t t, int sig);

// you're actually trying to get the thread to stop
// causes the thread to exit eventually (when it can)
// roughly at the next system call
// more accurately, at the next syscall that is a cancellation point 
int pthread_cancel(pthread_t t);

/*
*/

/*
- if threads get stuck in a tight loop, it might be hard to actually kill it
- since you're in charge of the code, this is your fault 
*/

/*
we don't have sigkill, as the thread might be in a critical section, 
and we don't want want the critical section to be wiped out
*/

void run(void **) {

}
```

### What if execlp runs in the thread?
Immediately kills off all the other threads

## How does multithreading work with cores? 
Often with multiple cores, high end processes will often run two threads
in the same core. (Hyperthreading)
- two copies of the registers
- but the two threads share the heavyweight components of the cores

Since two hyperthreads might compete components, OS's try to schedule threads
- not important for this course 

### when a signal is sent to a process, what happens to the threads?
POSIX - one of the threads at random gets the signal (not truely random)

## Signal Downsides
changes the way your abstract machine operates
- think of a thread as executing your call
- when a signal arrives, you can think of it calling another function
> signals cause an asyncronous function calls at any time between two instructions

<br>for example
```asm
movq abc, %rax
BAM, SYSCALL
addq %37, %rax
movl %eax, def
shr
mov %odw, def + 2
```
^ what if our signal caller messed with def?
<br>could cause "your variable turning to mush"

## Couple of pieces of advice when dealing with signal handling
1. only do very simple things inside of a signal handler
```c
int volatile x;
void handle(int sig) { x = 1; } // stores 1 in a global var
```
what not todo:
```c
// this will prolly crash your program
void handle(int sig) {
    char *p = malloc(1000);
    *p = 9; 
    ...
    free(p);
}
```
what's happening here?
```c
main() {
    signal(SIGINT, handle);
    char *q = malloc(2000);
    /*
    maybe signal happens right when *q is getting malloced

    allocating within handle will screw up everything
    */
}
```
Instead, you can only use async_signal-safe functions, which is safe

`close` is safe. there is a very small number of functions is safe.

another bad example
```c
// anything that touches global state is prolly not safe 
void handle(int sig) {
    printf("ouch");
    // use stdout's output buffer

    // instead, use
    write("ouch");
    // signal safe, doesn't rely on output buffers
}
```

 
2. 


### what if you are in a signal handler and another signal is called
Your program is "safe" from other signal handlers while another is being called

## Blocking Signals
`int pthread_sigmask`
- think of signal set as an array of bits that represent signals
- can set bits to set & clear signals
```c
int pthread_sigmask(
    int how,
    const sigset_t * restrict_set,
    sigset_t * restrict_bset
)

/*
values for how
- SIG_SETMASK - takes the current signal mask and stores it into old
              - sets signal to what it was before
- SIG_BLOCK - block more signals 
- SIG_UNBLOCK - clears the set of signals 
*/
```

```c
sigseet_t old;
pthred_signmask(SIG_BLOCK, set, &old);
// DELICATE WORK
// critical section - don't want signal handler touching this code 
pthread_sigmaks(SIG_SETMASK, &old, NULL);
```
- make sure to not block signals for too long!
- if you delay too much, it makes the signal kinda useless
- want your critical section short, bounded in time

# Scheduling
## Mechanism and Policy
Policy - Rules

Mechanism - How to implment Policy

## Implementing Mechanism
two main approaches to do scheduling
1. thread volunteers to let other threads run
    - each thread decides for itself 
    - cooperative multitasking
2. kernel decides at any momement to switch the core from one thread to another
    - preemptive multitasking

## Cooperative Multitasking
- Smaller systems
- developers all trust each other to write good code
- used in early iterations of Macs

Apps want to
1. compute
2. do IO
3. communicate with other apps 

IO ad communication requires syscalls
<br>When you do a syscall, OS might decide to give CPU control to another thread
<br>When the kernel returns, it either returns to your thread or some other thread

It can do this because in the process table, it will just save in some registers
from another thread 

For cooperative multitasking, we just need our programs to have a syscall once
and a while.

If we have a program that needs to do a lot of computation, we meed a syscall
that does nothing: `sched_yield()`

When you drop into the kernel, and it is about to return from the syscall,
the kernel's `return` is more along the lines of
```
for each thread t:
    look to see if t deserves to run
    if so, then run it
```
^ this would be our scheduler ^ 

So we want our scheduler: (Tradeoff!)
- to be fast, only a few instructions
- but also not too dumb 

Linux has a dumb scheduler! smart schedulers are too slow

## Preemptive Multitasking
Timer interrupt 

On the motherboard, there's a clock. And the hardware can send a signal
every once a while. Acts like an instruction. In linux, it is every 10 ms.

Acts like an asynchronous system call to `sched_yield()`

There are new problems, now that it's preemptive.

We need to write our programs to understand that they can be taken away at any time

Not enough to solve infinite loops

`SIGXCPU`, you can arrange for a process to decide how much is too much time
with the cpu

### Scheduling Scale
Three scales for scheduling CPU time
1. long term - hours into the future
    - which processes are you going to admit into the system?
    - this issue typically comes up in large backend systems
    - you don't want these systems to be overloaded with too much work
    - you may need to reject process than instead of trying to do too much
2. medium term
    - which processes are you putting into ram?
    - which are you putting on drive?
3. short term - topic of the lecture
    - if you have processes in ram, but not not enouch cores, which processes 
      are going into the CPU?

### Good vs Bad Policies. What's a good metric for scheduling?
- wait time - how long do we wait for our process to start going
- turnaround time - when do we get our process done?
- response time - how long to get useful information?
- utilization - % of time that the CPU is doing useful work. Want it to be high
- fairness - don't want to give all the CPU time to one processer
    - various ways to measure, including:
    - if you submit a job to the system, are you guaranteed to get a result back?


## Policies
### First come first serve (FCFS)
sample processes
|name|arrival time|run time|
|-|-|-|
|A|0|5|
|B|1|2|
|C|2|9|
|D|3|4|

So our processes look like <br>
AAAAABBCCCCCCCCCDDDD

Too simplistic!<br>
above we are going to have a context switch between process<br>
so the actual wait times will be a bit higher than the time of all the processes

### Shortest Job First (SJF)
Highest priority job is the job that's the shortest