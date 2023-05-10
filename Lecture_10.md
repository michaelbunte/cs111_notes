# Lecture 10 - IO Performance
Want to work our way up to the design of file systems  
File system
- data structure that operates on secondary storage

Two parts of secondary storage that perform differently
- disk and flash are slower
- persistent 

## Review of Storage Hierarchy
Sorted by speed:  
| type | amount | time | other info
|-|-|-|-|
registers|1 KiB|1 Cycle
cache| 16MiB | 2-3 cycles | 3ish levels
RAM|32GiB|
flash/disk (secondary storage)|4 TiB| | flash more common on laptops<br>disk more common on servers
tapes(optical or mag)<br>optical drives|
network drives (cloud / tertiary storage) | 4 PiB


Boost performance on overall system (large storage but fast), through use of 
data structures

## Performance Metrics for I/O
> **Utilization**  
  Are we using the hardware to the extent to which it can be used  
  Important from the perspective of the provisioner  
  Fraction of the IO rate capacity that is being used  
  Want to be 100%

> **Latency**  
  Delay between issuing an IO request and completing it  
  Important from the perspective of the user

> **Throughput**  
  Rate of IO completions per second

Relatively speaking, CPU's have gotten faster than hard disks. Used to be the 
other way around

What this means is that older devices use simpler scheduling algos, more modern
ones use simpler ones

## Hard Disk
5400 - 10000 rpm  
190 Hz

I/O Heads (magnetic surfaces), like needle on record player  
Arm can move head to get info from different parts of the disk

> Overhead of doing a read:  
 \- seek time: time to move the arm, ~8 milliseconds on a cheap disk  
 \- rotational latency:  time to rotate the disk ~$\frac 1{190 \times 2}$ seconds  
 \- transfer speed: getting data from disk to head

Modern disks now have a disk controller
- presents to the cpu the illusion of having a somewhat different device
- CPU says "i want sector # 5 million"
- Disk controller's responsibility to translate to a position on the disk
- Makes disk look like an array of sectors

Sequential access is pretty effienct, random access isn't

Disk controller has it's own RAM
- can cache small amounts of data
- if your IO requests are all to the same sector, you'll find that you'll get 
  your data very quickly

If you see a 8 ms read, 10 ms write, this is because a read can be done in 
slightly the wrong spot, while the write has to be done at exactly the write 
spot 

## Flash
More expensive per byte  
NAND Cell - eggert thinks of it as a black box
- tiny chunk of a flash disk that hold 1,2,3,4,5 bits depending on what kind of 
  flash geometry you are using 
- small number of bits
- state is persistent

Think of a flash chip as a 3D array of NAND Cells
- but don't think of cells as individually read and writable
- arrange in groups:
    - page:
        - smallest grouping
        - array of cells ( + extra data including error correction)
        - ~4 KiB
    - erasure block:
        - array of pages
    - plane: 
        - array of blocks
        - think of 2d slices of 3d system
    - channel (or die):
        - array of planes

Read granularity is a page

To write to a page
- erase an erasure page
- write to the pages in an erasure block
- if you want to change a page, you have to erase the whole page and start over
- could be 6ish times slower than a read operation

Flash drives wear out (~10,000 - 1,000,000 times)

Like disks, have an abstraction system from the hardware to isolate these 
details from the hardware
- flash controller 
- cache

Block layer abstraction  
read/write to block i

FTL - flash translation layer
- translates reads and writes to read, writes, and erases

Hides:  
Geometry  
Existence of Erasure Blocks  

We hide geometry by mapping logical block numbers to physical block numbers
- has a table that maps logical block numbers to physical block numbers
- however, as time goes on, this mapping changes

**CPU**:  
write logical block 10 with AAA  
write logical block 98 with BBB  
write logical block 51 with CCC  

**In flash**  
erase block 33  
write to page, map from block to physical page  

Mapping table is stored in RAM, so remember it by:
- if we know power is going out, we write to a certain part in the machine

in practice, we actually
- put enough info to reconstruct the mapping table, by putting extra info in
  each block. 
- if given a hard shut down, we can read through each block's out of band info

> **write amplification**  
user wants to write a small amount of data, but uses more space  
\- causes you to wear out drive faster
\- worse performance

### Wear leveling  
- pick blocks in sequence so all we don't just waste a certain part of the drive

### FTL Complexity
Over time, the flash will need to move pages to other blocks to be to condense
pages into a fewer number of blocks

Called garbage colleciton

Full flash drive is slower and wears out faster

Level of **write amplification**

### ZNS abstraction for flash
- CPU can read and write to the cache, directly
- put not only data, but also commands in the cache 
- commands for data structure
    - think of like a todo list
    - up to the flash controller to execute list
- up to both writers to use locks to deal with race conditions

Partitions drives into zones (big erasure block)

Pratitions drives into zones ~1 GiB
- eacch written only sequentially
- each erased all at once
- much coarser grain of address translation

zone states:
- empty - no data
- open for writing - appending data into the zone
    - limit to how many writes you can do simultaneously ~12
- closed  - can't do IO, but has data
- full
- read only - nobody can write to it
- offline - can't access until open 

ZNS doesn't do garbage collection
- you're responsible for it (as the OS)

ZNS commands  
administrative:  
create zone  
erase zone

common:
read  
write  
flush  
- make sure the writes are done (it's been committed to the underlying storage)
- OS says, let's write this and this and this, and then do a flush to wait to 
  make sure that the data is actually there so we know it can survive a power
  outage
- maybe flush every 10ish writes
- think of it like a commit record

Simplified versiion of IO storage devices from IO's view
- RAM cache of commonly accessed parts
- also stores metadata that maintains mapping ram and devices

When we `write`, copy the data into ram and then say done. ram will be mapped
to a place in disk
- promise to upload to cache later

`fdatasync()` make sure all the cached data is sent out to secondary storage
- synchronize device contents with the cache
- fastest

`fsync()`
- `fdatasync()` + save metadata (last modified timestamps)

`sync()`
- save everything
- slowest
- syncs all devices

### How do you decide what to cache? - Secondary Storage Scheduling Problem

asummes (usually)
- spatial locality
    - if we are accessing the `ith` block, we will prolly be accessing the 
      `ith + ` $\epsilon$ at the same time 
- temporal locality
    - if you access a block at time $t$, you will probably want to access the same
      block at time $t + \epsilon$

Basic Performance Technique: SPECULATION  
guess on the above assumptions 
> **Prefetching:**  
read in more blocks than you need to, guessing that they will be needed