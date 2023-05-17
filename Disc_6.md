## Colision Handling
- Colilisions happen when it generates the same index number for different keys

## Collision Resolution with Chaining
To search this hash table
- calculate the indexes before to locate the correct element
- use a standard linked list traversal to find what you're 
  looking for

## Coding
- includ elocks in `hash_table_v1_add_entry` and `hash_table_v2_add_entry`
- add some declarations and initializations 

## v1 vs v2
v1:  
- siknce mutex only

## FAQ
- why initial code has 0 missions on v1 and v2
    - no, only assigned a single thread to the VM
    - using m1, in some edge cases, assigning multiple cores 
      still acts as 1 core

Solution
- use seasnet (not stable on performance measurement)
- compile natively on macos, need to create xcode and modify
  makefile

Can we destroy mutexes upon receiving error codes
- no you odn't need to do so, these 2 sentences (destroy 
mutexes, and add exit upon error code) are independent

## lab4 overview
Implement a program
- will initialize a file system image

Image: archive of something. Records everything inside a single file. 

## Apartment manager is a file system (ext2)
- there are many floors
- each floor has many rooms
- need a quick glance of each floor's situation
- need to record which apartments are occupied
- also need to record who owns/uses which rooms

## Formal Description of file system
- organizes, stores, retrieves data in a storage device
- without a file system, the data on a device woud become 
  meaningless chunk of zeros and ones

## Main tasks of a file system
- allocate space in a granular manner
- organize files and direcories, and keep track of whihc areas
  of the media belong to which file and which are not being
  used

directories
- a file system typically makes it possible for a user to group 
  a bunch of files together to support directories

metadata
- we need to know the length of a file
- a file's creation time, mod time, access time
- owners, access permissions
- other stuff

|analogy|actual|
|-|-|
many floors | many group blocks
many apartments per floor | many blocks per group
a quick glance of each floor | group descriptor
record occupancy | block bitmap
record ownership/meta | inode bitmap/tables

## Files
a file is a collection of data
- a file is a collection of data that is stored on disk and that can be manipulated as a signle unit by its name
- a file is simply a linear array of bytes, each of which you can read or write
- each file has some kind of low level name, called an inode number

## directories
- also a file
- store file names and meta info about each of the files
- a directory file is basically a list of inodes with their assigned names, and list includes an entry for itself, its parent, and each of its children

## Block group descriptor
- located in the next block after super block
-rprovides an overview of how the volume is split into block groups

## Special file type:
- directory
- ext2 implements directories as a special kind of file, which containf ile names together with the corresponding inode numbers
- a directoy file has a list of ext2_dir_entry
    - each entry refers to a file in this directory
    - has a variable size depending on the length of the file name
    - the maximum length of a file name is ext2_name_len (usually 255)
- each entry contains the following information

## What is the lab about?
you'll be making a 1MiB ext2 file system with 2 directories, 1 regular file, and 1 symbolic link

