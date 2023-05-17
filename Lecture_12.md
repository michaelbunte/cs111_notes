# Lecture 12 
## File System Design
### Inodes
On disk data structure  
at key moments is cached in ram  
| contains info about |
|-|
type
size
group
permission
stamp
disk block pointer #1
disk block pointer #2
...
disk block pointer #10
indirect block pointer
double indirect block pointer

### `truncate()` syscall
`truncate("/a/b",109238)`  
changes file to be the amount of specified 
> if this is smaller than initial size, we need to discard data

> if larger, we just allocate 

### Type Field
- directories
- files
- other stuff

## Directories
Highly structured contents  
not something that a program can easily change  
under control of the OS  

### Original Unix Model
Think of directory as a table that maps names to inode numbers

__2 default directory entries__

|directory name | inode number | 
|-|-|
|`. ` (current) | 271 |
|`.. ` (parent) | 47 |

### Issue:
how many bytes do you allow for the name of the file?  
Any amount is bad, either too small, or too large

**solution** - ext4 directory entry
- inode - 32 bits
- directory entry length - 16 bits
- name length - 8 bits
- file type -  8
- N-byte-file name -

Why do we have both a directory entry length and a name length

We can delete directory entries by change the the size of the directory entry
length before the current directory

### syscall `mkdir("d", 0755)`
directory name and permissions  

Need to keep track of working directory 
- in our process table entry, we need a slot that holds the working 
    directory, which is an inode number

**Steps**
1. get inode #510 into RAM (known block number)
2. look at the contents of the current directory to see if it already has a
   directory called `"D"`
   - get this inode's data block into RAM (could be here already)
   - walk through our main memory copy and find a directory entry named "d"
3. Make sure we have search permissions for inode 510
    - check our `uid` and `gid` against the permissions of the inode
4. find a free area in the directory block
5. assume we have enough room
6. create a new directory in memory
7. look for unused inode number in the table
    - fill it in with info suitable for the now empty directory
        - owner
        - group
        - permissions
        - time stamp
        - data block `.` and `..`
        - etc
7. allocate data block
    - find unused data block to fill in with `.` and `..`
8. write out
    - new directory's data block
    - new directory's inode entry
    - data block for the parent directory
    - parent directory's inode
    - bitmap block for newly allocated directory


### Something simpler
Suppose we have the filename `/home/eggert/cs111/d`  
and we execute the syscall `chdir("/home/eggert/cs111/d")`  
Want to change the working directory in our process table entry 

Kernel function `namei`, which maps file names to inode numbers
- you give it a string
- and it gives you the inode number of that directory

```
if first character is '/'
    start in root directory
else
    start in the current working directory
```

System call `chroot("/home/eggert/jail")`, which changes the root  
has a security flaw:
- then a file like `/etc/passwd` would be be able to be reassigned
- therefore, only the superuser can use chroot

Since `/..` is impossible `/.. == /`

```c
int i = name[0] == '/' ? curproc -> rootdir : curproc->wd
char const *p = name;
while(true) {
    lookup(i, filenamecomponent(p));
    if(!j) return 0;
    // since there are no simple ways to watch out for pointers, 
    // then we just 
    if (j is a symlink ){
        c = contents of j;
        if c starts with a slash {
            i = curproc -> rootdir;
            splice cs contents into filename
        }
    }
    i = j;
}
```

### Another issue
not all files are directories  
such as symlinks

An inode for a symlink will have som data block value, and then bytes that 
point to a data block

Difference is how it affects name i 

So for example  
`/home/eggert/bin/sh` will become `/home/eggert/user/bin/sh`

### Another Issue
cycles:  
`ln -s b b`

`ln -s a b`  
`ln -s b a` 

`ln -s a/b/c d`  
`l -s x a`

`link("a","b")`  
will make both have the same inode file  

`symlink("a","c")`  
creates another file with the text "a"  

`link("c","d")`  
creates a hard link to a symlink  

`unlink("c")`  
one of the few syscalls that don't treat symlinks like ordinary links  
removes the symlink instead of what it's pointing to 

**other examples**  
`link`  

`lstat("a", &st)`  
doesn't follow symlink

`find . -name bin`  
doesn't follow symlink because it could end up in a looping situation


### Another Issue
when you remove a file, how does the file system remove storage?  
We don't want the file system to gain leaks  
reclaiming storage: 

1. truncating a file  
for example `$ >foo`  
return the file's data blocks:  
indirect blocks to the file area

2. remove a file  
remove the files to the last (hard) link and no process has the file open  
remember, you can have more than one hard link to a file  
(1)  
mark the inode entry as being unused  

### link count
in indoe  
count the number of directory entries pointing to this inodes  
every time you call `link("a", "b")` we increment the link count  
every time we call `unlink("b")` it decrements it  

`mkdir("c")` creates a `count = 2` for the new directory  
increments the count for the parent directory  
`rmdir("c")`  

cannot create cycles with directories, apart from with symlinks

so if `d` is a directory, `link("d", "e")` is not allowed  
no hard links to directories 

### Other file types 
`ls -l`  
`-` for regular files  
`d` for directories  
`l` symlinks  
`b` block special file  
`c` character special files, read in characters, not blocks
- `/dev/tty`
- `/dev/null`
- `/dev/full`  

`p` named pipes (fifos)  
pipes that are visible to the file system  
brings the pipe into the namespace of the file system
```bash
$ mkkfifo x
$ ls -l x
prw-rw-r--
$ cat x     # hangs until another process outputs to x 
```
**other types**
sockets  
shared memory object (only in ram, so fast)  
semaphores (files that act like a semaphore)

## What if we have a computer with two drives?
### Solution 1 
Pretend that we have one huge concatenated drive  
This is a pain, since we have a single file system that is vulnerable to a 
single file system failing  

### Solution 2
More typical, split into two file system

In the first drive, put very important files (such as booting info) into the 
first part of the first directory

file system prefix in file names  (in windows)
`C:\a\b`  
`D:\a\b`

Downside to this approach  
complicate file naming, colons?

Every file system needs to be cognisent of the existence of drives

### Unix approach - mounting
programming file prefixes

Mount table  
prefixes for a file system are file names themselves
ex: `/home/eggert/bin`

Now our files also need to store file system number
- `ino_t` - inode number
- `dev_t` - filesystem number
