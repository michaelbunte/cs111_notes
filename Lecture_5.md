# Lecture 5
#
### Software interfaces - Files & Processes

## Orthogonality
### How good are the interfaces that we have?
we want them to be modular and composable

files
- say that we open a file, remove the file, but there's some process 
  reading/writing to the file

So good programs make our processes independent from one another

## Processes

### Goal
- multitasking - lots of apps at once
- performace
- isolation - don't want all these applications interfere with each 
  other (stamp on each other's work)
    - this makes our systems reliable
    - and more secure

### how much isolation
- processes don't know anything about each other's processes 
- however, this is too far
- we need some form of **controlled communication**

### Controlled Communication
1. files 
- `cat >f` and `head <f`
2. message passing - sockets
- `cat | head`
3. shared memory 
- bypass copying operation
- both processes have access to the same piece of RAM
<br>**DANGERS**
- race conditions
4. other — signals 
5. create another process 
6. wait for another process (act of process finishing)
7. backdoors - process can evade the OS's protections in order to talk
  to each other
  - security hole! (or something that acts like one)

### How to model OS resources in your app
1. model OS resources as objects<br>
`struct file * `<br>
this is simple and fast, but no protection <br>
and race problems
2. opaque file/process handles<br>
instead of pointers, we have "something" like a bit pattern
- we can't look at bit pattern directly
- not a pointer
- all we have is some `int`
- if we want to know how many bytes are in the file, we *have* to use 
  an API that controls access to this information
  - `fstat(int fd, struct stat* st)`
  - you give a file descriptor, and they give you a structure of the object
  - so now we hav emore protection and few racer conditions
  - but it's slower and more complex

### Api to get access to a file
`int open(char const * filename, int flags, mode_t mode)
- `flags` provide info about what kind of access we want
- `mode_t mode` (if we create a file, what kind of permissions)
    - ex `0644` means `-rw-r--r--`
- `O_RDONLY`, `O_WRONLY`, `O_RDWR`, `O_PATH`
    - chose one 
    - you can | `O_CREATE` to create if nonexist
- `O_PATH` - neither want to read or write
- returns `fd` or `< -1` errno
- `int close(int fd)`
    - returns 0 if succeeded, -1 if fails
- `ssize_t read(int fd, void *buf, size_t bufsize)`
    - returns the number of bytes read
- `write(int fd, void const *buff)`
- `off_t lseek(int fd, off_t offset, int whence)`
    - whence flags
        - `SEEK_SET` - curr = offset
        - `SEEK_CUR` - curr += offset
        - `SEEK_END` - filesize + offset 
    - used to change file size offset
    - doesn't work for pipes (stream devices), only storage devices
- there's also a `pread` and `pwrite` that let's you put the positioning
  in one line so you don't need `lseek`

## Interface Stability
`int open(char const * filename, int flags, ...)`
- want ability to open file on some other machine <br>
can create an api change: `int remote_open(char const *fileserver, ... )`
- so instead, using the same API, we can open with 
    - `lnsrv11:/etc/pswd`
- this creates a compatability issue, since files with colons are gonna
  have problems
- current solution:
    - `//lnxsrv1/etc/passwd` or `/proc/self/fd/27`
### API for processes
`pid_t fork();`
```c
// for example
pid_t p = fork();
if(p < 0)
    error("Fork failed");
else if (p == 0)
    printf("executing in child process");
else if (p > 0) 
    printf("executing in a parent. pid is the childs");
```
```c
// BAD: RECURSIVE DEATH
while(fork() >= 0) continue
```
<br>

`[[noreturn]] void _exit(int status);`
- this does not return
- kills the running process
<br>

`pid_t waitpid(pid_t p, int *status, int options)`
- options: 
    - `0` wait
    - `WNOHANG` – want to know if anyone has exited
- waits for the child to access

If the parent forks and then exits
- process #1 gains a new child

If the child exits but the parent doens't call wait pid 
- OS has recorded the fact that the child is dead
- dead child = zombie 
- `init process` (#1)
    - calls `waitpid(-1, ..., WNOHANG)` (wait for any of my children, to finish all of the zombies)

`execvp (char const * filename, char *const *argv)`
- reuses the process to run a different program from scratch 
- if it succeeds, it does not return