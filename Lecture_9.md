# Lecture 9
```c
enum { N = 1 << 13; }

//implementing a pipe as a bounded buffer
struct pipe {
    char buf[N];
    unsigned r, w;

    // r: total num of bytes read from a pipe
    // w: total num of bytes written to a pipe
};

// read one byte at a time
void writec(struct pipe *p, char c) {
    p->buf[p->w++]%N=c;
}

char readc(struct pipe *p) {
    return p->buf[p->r++%N];
}
/*
These implementations are bogus:
    - can't make sure that there isn't data in the pipe
    - can't tell if pipe is full
*/
```

```c
enum { N = 1 << 13; }

struct pipe {
    char buf[N];
    unsigned r, w;
};

void writec(struct pipe *p, char c) {
    lock(&l);
    // check if pipe is non full
    while(p->w - p->r < N);
    p->buf[p->w++]%N=c;
    unlock(&l);
}

char readc(struct pipe *p) {
    // check if pipe is non empty
    lock(&l);
    while(p->w - p->r >);
    char c = p->buf[p->r++%N];
    unlock(&l);
    return c;
}

/*
new issue: can't just have one lock per system
- can't have a course grained lock
- want a fine grain lock
*/
```

```c
/*
finer grain lock, within structure
*/
enum { N = 1 << 13; }

struct pipe {
    char buf[N];
    unsigned r, w;
    lock_t l;
};

void writec(struct pipe *p, char c) {
    lock(&->l);
    // check if pipe is non full
    while(p->w - p->r < N);
    p->buf[p->w++]%N=c;
    unlock(&->l);
}

char readc(struct pipe *p) {
    // check if pipe is non empty
    lock(&p->l);
    while(p->w - p->r >);
    char c = p->buf[p->r++%N];
    unlock(&->l);
    return c;
}

/* Assumption:
- We are running on a multiprocesser.
- that another thread will rescue us if we get into a deadlock
*/
```

```c
//issues with lock
bool ok;
do {
    lock(&p->l);
    ok = p->w - p->r < N;
    if(ok) {
        p->buf[p->w++%N]=c;
    }
    unlock(&p->l);
}
while(!ok);

/*
Doesn't work on a uniprocessor system, gets stuck forever
*/
```

```c
// simpler solution
do {
    disable_interrupts();
    ok = p->w - p->r < N;
    if(ok) {
        p->buf[p->w++%N]=c;
    }
    enable_interrupts();
} while(!ok);

// nicer because we don't have to touch memory

// you never want to use a while loop in a critical section
```

### Assumptions
- all of the stuff we are doing between lock and unlock are fetched out of memory
    - treat lock and unlock as memory barriers
    - compilers can't reassemble instructions outside of the locks
    - underlying machine can't do memory fetches out of order

### Is there an alternative to spin locks that doesn't make this assumption?
- spin locks based on `xchgl(test_and_set)`
- `lock incl x`
    - given any mem instruction `x`
    - adds one to x in an atomic way
- `cmpxchgl`
    - more powerful
    - compare_and_swap
    - also atomic 
    - you give it an address, and the old and new values you want to have for a 
      particular location
    - 
        ```c
        bool c_a_s(int *addr, int old, int new) {
            if(*addr == old) {
                *addr = new;
                return true;
            }
            return false;
        }
        ```
    - so you can now add one ot a global variable by:
    -
        ```c
        // lets you say *p=f(*p), atomically
        void add1(int *p) {
            while(true) {
                int old = *p;
                if(c_a_s(p, old, old+1)) 
            }
        }
        ```
    - we can just use loads and stores
        - (bakery algorithm)
        - `x86-64` - loads and stores are atomic if they are loading and storing
          aligned full words (32 or 64 bit words)
        - therefore, you can't do 
        - 
            ```c
            struct {int a, b, c; }
            x = y;

            int d:3; // three bit int
            ```
        - atomically

## Making a lock smaller
- read/write locks
    - want to let lots of people read an object

How to implement - spin locks

```c
struct rwlock {
    lock_t l;
    int readers;
}

void lock_for_write(struct wrlock *p) {
    while (true) {
        lock(&p->l);
        if(!p->readers)
            return; // rule to unlock after locking not followed here
        unlock(&p->l);
    }
}

void unlock_for_right()
{
    unlock(&p->l);
}

void lock_for_read(...) {
    lock(&p->l);
    p->readers++;
    unlock(&p->l);
}

void unlock_for_read() {
    lock(&p->);
    p->readers--;
    unlock(&p->l);
}
```

### However spin locks are bad!!! They tie up the core they are running on
`blocking mutex`
- gives up the core to let some other thread run if the condition is not true

```c
struct bmutex {
    lock_t l;
    bool locked; // someone owns this locking mutex
    proc_t *blocked_list; // linked list of all the threads waiting on this mutex

}

void acquire(struct bmutex *p) { // gives access to the following mutex
    lock(&p->l);

    if(!p->locked) { // if no one owns the blocking mutex
        p->locked = true;
        unlock(&p->l)
        return;
    }

    unlock(&p->l);
    yield(); // gives up the cpu, lets someone else run
}

// just telling the OS to just let some other thread run
void release(struct bmutex *p) {
    lock(&p->l);
    p->locked = false;
    unlock(&p->l);
}

/* 
want to add a runnable bit in the process table, so the processor doesn't have to go through all this code.
*/
```

```c
void acquire(struct bmutex *p) { 
    lock(&p->l);

    if(!p->locked) { 
        p->locked = true;
        unlock(&p->l)
        return;
    }

    unlock(&p->l);

    // cp = current thread
    cp->runnable = false;
    cp->next = p->blocked_list;
    p->blocked_list = current_thread;
    yield(); 
}
void release(struct bmutex *p) {
    lock(&p->l);
    p->locked = false;
    if(p->locked_list) {
        p->blocked_list->runnable = true;
        p->blocked_list = p->blocked_list->next;
    }
    unlock(&p->l);
}
```

## Semaphore
First locking primitive widely written about
- blocking mutex, except you can have more than one thread active
- blocking mutex with capacity >1
- how many threads are currently running?
    - if that number is less than the semaphor's capacity, they let you in
- blockign mutex = semaphore c with capacity 1   


```c
// sempahores not good enough for pipes
// issue with this is that we'll have to constantly context switch into this 
// to see that we can't run it let
void writec(struct pipe*p, char c) {
    while(true) {
        acquire(&p->m);
        if(p->w-p->r <N) {
            p->buff[p->w++%N] = c;
            release_(&p->m); return;
        }
        release(&p->m);
    }
}
```

## Conditional variables
Up to us a programmer to determine what conditions are needed

We need
1. a condition (up to us) defining the condition
2. a blocking mutex protecting (1)
3. our condition variables that represents the condition

### API
```c
// condition variable and blocking mutex
void wait(condvar_t *c, bmutex_t *b){
    // assume that b has been acquired
    /*
    releases b, and then blocks until some other thread changes the truth value of that condition
    */
    // then returns
}

void notify(condar_t *c) {
    // wakes up one of the threads waiting on the condition
}

void broadcast()
    // wakes up all of the threads
```

So our code can now look like
```c
struct pipe {
    ...,
    bmutex_t m;
    condvar_t nonfull, nonempty;
}


void writec(struct pipe *p, char c) {
    while(true) {
        acquire(&p->m) // assuming m is a blocking mutex
        if(p->w - p->r < N) { // check to see if pipe is not full
            p->buf[p->w++ %N] = c;
            release(&p->m);
            notify(&p->nonempty);
            return;
        }
        wait(&p->nonfull, &p->m);
    }
}

char readc(struct pipe *p) {
    while(true) {
        acquire(&p->m);
        if(p->w-p->r > 0) {
            char c = p->buf[p->r++%N];
            release(&p->m);
            notify(&p->nonfull);
            return c;
        }
        wait(&p->nonempty, &p->m);
    }
}
```

## Deadlock
`cat` implementation
```c
// copies bytes from buff a to user space
read(0, buff, buffsize); 
// internally uses a lock to access the memory in buff

// copies bytes from user space buf to bufb
write(1, buff, buffsize);
// also uses a lock
```

```c
acquire(buffa->m)
acquire(buffb->m)
release(buffa->m)
release(buffb->m)
// this could cause deadlock!!
```

Faster way:  
`rw(0,1, bufsize)`   
- system call
- faster, since it doesn't copy into user space


### Deadlock is a race condition1
### Conditions
1. circular wait 
2. mutual exclusion
    - us grabbing a lock means that no one else can use the lock
3. no preemption of locks
    - some operating systems let you preempt a blocking mutex
4. system where you can hold and wait 

### How to prevent deadlock?
1. OS detects deadlocks dynamically
    - `acquire` returns a boolean
    - keep track of all of processes with locks, and who those locks belong to
2. Lock ordering
    - acquire locks in increasing order
    - if you notice the next lock you are trying to acquire is actually taken,
      give up all of your locks and start over