# Scheduling
Algorithims
- FCFS
- SJF

## Shortest Job First
- has an issue with fairness
> metric of fairness: if you add a job to a system, it will eventually 
  finish

**preemption** is a solution to the fairness problem

**Round Robin Scheduling** is a common technique for establishing 
fairness

| process | arrival | run | wait time |
|-|-|-|-
A|0|5|0
B|1|2|$1\delta$
C|2|9|$2\delta$
D|3|4|$3\delta$

`ABCDABCDACDACDACCCCC`

- So average wait time is $1.5\delta$
- but the throughput is worse, because of all of the context switching

## More ideas
- put arrivals at the end of the queue
    - this allows you to avoid starvation
- put processes on a stack
    - can cause starvation

RR = FCFS + preemption
?? = SJF + preemption

## Priority Scheduling
Some processes are more important than others, so they should be 
scheduled earlier than other processes 

### Highest Priority Scheduling
- A: is a garbage collection process
- B: real time interrupt

You prolly want to put B at a higher level

#### Linux: Processes have niceness values
- range between -20 to 20
- -20 processes are greedy
- 20 will let anyone else run ahead of them

### Static vs dynamic priority
Static: everything is assigned a priority at the the beginning
- very simple robust systems
Dynamic: Assigned by the OS as the process run

### Other ideas
preemption: niceness = arrival time + job length

### Static + Dynamic Priority
|niceness|user|
|-|-|
-2|ops
-1 | faculty
0 | staff
1 | students

> aging - involving arrival time in your nicessness formula

### Priority Inversion
Say we have three threads $T_{\text{low}}$, $T_{\text{between}}$, and $T_{\text{high}}$.

Let's say that $T_{\text{between}}$ is waiting, and $T_{\text{high}}$ is waiting too.

Since  $T_{\text{low}}$ is the only program that can run, we let it run.

Then $T_{\text{low}}$ gets a lock on some memory.

However, while $T_{\text{low}}$ is during a critical section, 
$T_{\text{high}}$ becomes runnable

We then send control to $T_{\text{high}}$, but $T_{\text{high}}$ needs
the lock that $T_{\text{low}}$ has. So it is waiting.

But while this is waiting, we get a contact switch to 
$T_{\text{between}}$, which runs off with the program

So now, $T_{\text{high}}$ can never be run

**Solution**<br>
OS detects and deals with the problem


## Real time Scheduling
Involve deadlines imposed by the system

Once we start talking about deadlines, many of the things that we talked
about above are wrong

Types: 
- hard: you absolutely cannot miss a deadline
    - predictability trumps performance
    - so we don't use caching, which is unpredicatable
    - we want to know exactly how long an instruction will take
    - interrupts are bad! can happen at any time. use polling instead
- soft realtime: you want to hit deadlines, but missing occasionally is ok
    - missing is bad, but not catastrophic
    - such as rendering frames in videos
    - simple algo
        - think of all the tasks and deadlines of your system
        - and do earliest deadline first
        - as long as each of your jobs is relatively short, this isn't
          terrible
        - drop activities if you can't do them in time
        - can't handle bursts
    - rate-monotonic scheduling
        - try to think of each of your tasks as repeated tasks
        - such as a frame rate to match
        - try to allocate CPU resources to keep those rates going
        - assign higher priorities to more frequent tasks

# Coordination
What happens when actions being done collide?

What if action $A_4$ deposits into the account $B_3$ is withdrawing from?

sequence coordination: when a thread does actions in sequences
- think semicolons!
- but it breaks down when we are dealing with actions in different threads

We want **isolation** between any pair of actions in different threads <br>
if two actions at the same time are dealing with the same storage, we're in trouble

### Atomicity
An action done in a "single instance in time" with respect to other actions.

Effects occur either all before or all after other actions

This can lead to some efficiency issues, as we aren't doing things in parallel

### Serializability and Observability
The simplest way to ensure we are correct: make all other actions atomic <br>
no parallelism!!

Instead, OS's have a weaker approach. instead:
- what you can observe about the behavior of the system can be explained
  as if the sequence of events was sequential


### Deposit Withdraw example
```c
long balance;
void deposit(long amount) {
    if(amount < 0 || amount > LONG_MAX - balance) return false;
    balance += amount;
}
bool withdraw(long amount) {
    if(amount >= balance || amount < 0) return false;
    balance -= amount;
    return true;
}
/*
but what if at another ATM, someone else withdraws right after we checked??
*/

// more race condition problems
bool transfer(amount, acc1, acc2) {}
bool bank_audit()
```

## A critical section
a section of our code that at most will have one thread executing. We want to<br>
have at most 1 thread executing.
- correctness<br>
- performance: 
    - overhead of getting locks 
    - bottlnecks of getting critical section

### Enforcement of C.S
- mutual exclusion
    - when my thread is executing, no other thread can execute
- bounded waits
    - don't want to be stuck forever
    - prevents starvation
    - classic way to get a bounded wait: whenever a thread is in a critical
      section, it finishes as quick as possible
    - to determine where a CS boundaries are, do two things:
        1. find shared writes
            - writing into storage that other threads might be
              as well
            - `balance += amount;`
        2. find dependent reads for those writes
            - find all the shared storage that you're reading that you use to
              figure out what to write
            - `amount > LONG_MAX - balance`
    - find the minimum piece of code doing the above


```c
// simple lock example - DOESN'T WORK
// two threads could grab the lock at the same time

// lock is 0 if no one owns it, 1 if someone does
typedef int lock_t;

// PRE - you don't have the lock
void lock(lock_t *l) {
    while(*l) continue;
    *l = 1;
}

// PRE - you have the lock
void unlock(lock_t *l) {
    *l = 0;
}
```

There's an algo for this, but in reality, there is hardware support for this<br>
We want a single instruction:
```c
test_and_set(int *p, int new) {
    ///vvv critical section enforced by the hardware
    int old = *p;
    *p = new;
    return old;
    ///^^^
}
```

```x86
xchgl %eax, (%rbp)
```

```c
// revised

typedef int lock_t;

// PRE - you don't have the lock
void lock(lock_t *l) {
    while(test_and_set(l, 1)) continue;
    *l = 1;
}

// PRE - you have the lock
void unlock(lock_t *l) {
    *l = 0;
}
```