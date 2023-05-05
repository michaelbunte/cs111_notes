# Discussion 5

## Lab 3
Implement a multople threads hash table  

### Concepts
- hash tables
- slist apis
- thread api and locks
- lab3 starter

### Problem 1
Consider a simple one-dimensional array variable  
If you want to find a particular person in the array, you need to loop through  
$\text{O}(n)$  

You could also use a binary search, which is $\text{O}(n)$

Best thing would be to use a hash table, which is $\text{O}(1)$  
A hash table is a data structure that allows for a very fast retrieval of data,
no matter how much data there is  
And we can have a linked list for each element of the array. We deal with 
collisions with chaining


## Threads
Like processes, but with shared memory  
they have their own 
- registers
- prorgam counter
- stack

They have the same address space, so changes appear in each thread
One process can have multiple threads


```c
void *run(void *arg){
    printf("in run\n");
    return NULL;
}
int main() {
    pthread_t thread;
    int rc;
    rc = pthread_create(&thread, NULL, &run, NULL);
    assert(rc == 0);
    printf()
}
```

## Data Races
Occur when two threads share the same data


## Race Prevention
lock: only enable one thread to execute the code at one time 
- everything within the lock is a critical section

## Goal
- make hash table "add entry" thread safe

Implement two locking strategies
- `hash_table_v1_add_entry`
    - only correctness
    - create single mutex
- `hash_table_v2_add_entry`
    - correctness and performance
    - create as many mutexes as you like
- compare v1, v2, and base implementations

`>./hash-table-tester -t -8 -s 500000
- -t changes the number of threads to use (default 4)
- -s changes the number of hash table entries to add per thread (default 25000)


Coding (trivial)
- include locks in `hash_table_v1_add_entry` and `hash_table_v2_add_entry`
- in total 30 lines of code 

### v1
- Single mutex only
- only care about correctness

### v2
- as many mutexes as needed
- make it thread safe
- care about both correctness and performance
- think about parts which can go wrong if not locked, while being accessed 
concurently
- meaning you need enough granularity to get a full mark in v2

DON'T FORGET THE README