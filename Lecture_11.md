# Lecture 11 - Block Level IO / File Systems

## Speculation
a way of improving block level io performance

### Prefetching
even though the user just reads in block x, you also read in x + 1 & 2  
good for reads

### Dallying
system says "please write block number i"  
however, we wait a bit before doing it  
we wait the system to instead ask for an adjacent block  
and then we write out both of them
and then we save some overhead

### Batching
more efficient to write out one big block than a bunch of a smaller adjacent 
blocks  

### We might have multiple caches

- RAM cache
- L1 cache for single cpu
- L2 cache shared by cpus
- cache in disk controller

### Cache cohererence
Want caches to agree for what they are holding

### Classic example of problem
CPU1 - running lnx process 1  
CPU2 - running lnx process 2  

CPU1 - does a write, which reads into the buffer cache  
CPU2 - does a read, which readss from the disk  

How do we let CPU2 know that the disk is now up to date?

### What if above we were using `putchar` and `getchar` instead?

`putchar` and `getchar` operate inside a buffer inside of userspace

Yet another cache sitting inside of each process

<hr>

## Disk Scheduling Algorithms
number of processes talking to the same storage device  

>limited resource: access to the harddrive

>want:  
high throughput (as many io's as we can)  
fairness (no starvation)

### Hard Disk
- seek time + rotational latency + other stuff  

for now, we are just going to focus on the seek time

### First come first serve
```
innermost                   outermost
track      h           i        track
------------------------------------
0                                  1
```
$\text{cost} = |i-h|$  
$\text{cost}_{\text{average given h}}=\int_0^1 |i-h|di$  
$\text{cost}_{\text{average}}=\int_0^1\int_0^1 |i-h|didh=\frac13$  
^ for fcfs

### Shortest Seek time first
It's in the name!

High throughput, low fairness

### Elevator Algorithim
Shortest seek time first, but only in the positive direction  
Once at highest read, then start going down  
Fair (no starvation)  
Can still be made fairer, as it favors accesses in the middle

### Circular Elevator Algorithim
Only goes up  
Once it gets to the top, it seeks all the way down to the bottom  
Slight inefficiency, since has to move all the way down  
Not as bad as you'd think, as arms in real life accelerate

### SSTF + FCFS Batches
take incoming requests: `12, 97, 51, 11, 19, 29`  
break into batches `12, 97, 51` + `11, 19, 29`  
Do SSTF for each batch, but finish first batch before the second  

### Anticipatory Scheduling
Start getting requests: `12, 97, 51`  
Then wait a bit after servicing the first read (`12`), to see if anything closer
comes by  
Variant of **dallying** that helps with read performance  
Useful because often sequential reads often come in

## File System Level
Data structure that lives on secondary storage and in memory  
Important part living on the hard drive  
But then cached into ram

### Levels of the Linux-ish file system
| level | size|
|-|-|
|Symbolic Links | |
|file naming | | 
|Inode level| |
|file systeam | |
|Partitian Level| Maybe 3 Partitions, 
|block|8KiB|
|sector|512 Bytes|

Partitian Level
- partition table (keeps track of partition's locations and size)

File system
- root file system
- swap space
- user file system

Inode level
- data structure inside the file system that holds extra info about files
- each file gets a unique number 

File naming
- need some way of taking `/usr/bin/sh` and navigating to the actual file
- absolute file names, relative file names, and slashes

Symbolic Links


### Eggert 1974 Ignorant File System
break down into 512 byte sectors  
First four sectors have a table of contents
|file name | sector number <br> (where it starts) | file size in bytes | 
|-|-|-|

This strategy wastes space, called internal fragmentation 

### Rt11 File system
More sophisticated version of the Eggert File System  
Downside: file size must be known in advance  
Free space is split, so we don't have room to store a big file even though we 
actually do have room  
called external fragmentation

Advantage:  
All the files are created contiguously, so we can do sequential access of files,
which is faster, since we don't need to see much  
fast and more predictable IO

### FAT (File allocation table) file system
No more contiguous file system  
Splits up file system into 4 parts

boot sector | superblock |File allocation table| data (4 KiB blocks)
|-|-|-|

boot sector:  
unused, needs to be read into ram

superblock:  
meta information about the entire file system  
- version  
- size  
- \# of used blocks (lets you know how full file system is)
- root location
- block \# of the root file system
- etc

file allocation table:  
early model:
- array of 16 bit numbers put into sectors
- size of file allocation table 
- \# of data blocks / 2048
- see it more as an array of 16 bit integers

In each block, gives index of next index of the next block

```
1   2   3   4   5   ...
5       0   3
```

> 0 means end of file

> -1 or $2^{16}-1$ (block is unused)

No more external fragmentation

Since we treat blocks like a linked list, `lseek` is a $\text O(n)$ function

### Second Fat Idea  
Very handy to have a tree structured file system  
don't have a single table of contents for the files  
each directory is a table of contents that maps file names to file contents  

|name, 8|extension 3|location, 2|type, 1|size in bytes, 4?|
|-|-|-|-|-|
|`foo\0\0\0\0\0`|`.txt`|5|`dir/file`|45|

### Disk Defragmenter
Reorganizes file systems to make all the files nice and sequential


## Inode Idea - Unix Approach
>issue with fat  
a file doesn't exist unless there is a directory pointing to it 

Instead, we want to have files as "first class objects" independent of 
directories so that we can do some kinds of operations that make sense 
indepent of what the directory table looks like

Still some superblock that holds meta information about the table, but we 
also have an inode table

Holds all the information about the files in the systems
- time stamps
- owner, groups
- permissions
- `ls -l` gives you some of the info that is in the inode
- `ls -li` also gives you the inode number

Directory:  
Maps names to inode numbers

### We want lseek to be faster than order n
trailing part of inode tells us where the remaining blocks of the file system 
are 

For example:  
first slots tell you the first 10 locations of the first 10 blocks in the file  
if no blocks allocated, put a `0`  
easy for short files

**Indirect Blocks** 
11th block will point to a block that doesn't contain user data, called an 
indirect block. Array of block pointers. Each one contains to more blocks

If that's not enough:  
can create a doubly indirect block  
this is slower

### Berkeley File System  
> Issue with previous file system:  
doesn't tell us where availaible blocks are

added another table, that has a bitmap of free blocks (array of bits)