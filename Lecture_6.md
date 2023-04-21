```c
getpid();   // gives you process id
getppid();  // gives you parent process id
```

```c
dup2(a, b); // lets you move file descriptors around in the table
```

This is all showing us how to build `a|b`<br>
We want `a`'s fd1 to talk to `b`'s fd2

`a>tmpfile & b<tmpfile`
- will lead to race conditions
- i/o to secondary storage 
- what if `a` has a huge amount of output? wastes storage 

Pipes fix all of these things
- bounded buffer in the kernel (like 4 kilobytes)

Issues:
- sync complexity: 
    - reads can block, waiting for data to become available
    - writes can block
- maybe we could just fail instead of blocking? but of course there are
  other issues here

This is modeled by file descriptors
```c
// creates a pipe with two file descriptors
    // one for the read end, one for the write end 
int pipe(int fd[2]); // int pipe(int *fd)
// only use pipe when you're about to use a fork
```


```shell
$ sleep 3 | cat     # shell waits for cat to finish
$ cat | sleep 3     # shell waits for sleep 3 to finish
$ sleep 3 >&- | cat # >&- closes stdout, so shell immediately comes to life
```

```c
// DANCING PIPES

pid_t p = fork();
if(p == 0) {
    int fd[2];
    if (pipe(fd) < 0) ouch();
    pid_t q = fork();

    // grandchild code
    if(q == 0) {
        if(dup2(fd_1, 1) < 0) ouch();
        close(fd_1)
        close(fd_0);
        execlp("a",...);
    } else {
    // child code
    }
}
```

What can go wrong?
- read but no writer
    - when this happens, treated as EOF, read returns 0
- write but no reader
    1. (default) PIPE signal
        - if the pipe gets this, it just dies
        - `cat bigfile | true` - cat gets killed off
    2. `write()` returns -1 
    3. read, but writer never writes anything 
        - read will hang, and will hang forever
        ```c
        int fd[2];
        if(pipe(fd) >= 0) {
            read(fd[0], ...)
        }
        ```
    4. write, but reader never reads
        ```c
        int fd[2];
        if(pipe(fd) >= 0) {
            read(fd[1], ...)
        }
        ```

Abstraction
`parent.c` - class approach
```c
pid_t p = fork();
if ( p == 0 ) {
    // fiddle with fd's, signal handling
    // parent setting up the child environment

    // ^^ people don't like this often
    // we are doing a lot of changes just to set up a program to run
    eleclp("x", ... );
}
```

`parent.c` - other way
```c
// process spawning, single syscall that does all of the above
int posix_spawnp(
    pid * restrict result, // pointer restricted to not point to anything other args point to 
    char const * restrict file, // prog we want to run
    posix_spawn_file_actions_t const * file_acts
    // pointer to a piece of data that spawns all of the actions 
    posix_spawn_attr_t const * restrict attrp,
    // attributes of all the processes
    char * const * restrict argv, 
    char * const * restrict envp
    // environment variables that you want to run
)
```

## Race Conditions
`(cat a & cat b) > file`
- a and b are running at the same time, and their output is going  to the
  same file 
- order that you see in the file just depends on the speed of the programs
- you can't predict 

`(cat >a & cat >b) <file` also a race condition
<br>
`cat c | (cat >a & cat >b)` also a race condition

Another race condition
<br>Create a temp file: `open("/tmp/sort.1", O_TRUNC | O_ CREATE | O_RDWR, 0666)`
<br>An issue arises if two programs try to do this at the same time
One solution is to tack the `pid()` onto the end of each filename

We could also just tacking a random number on the end 
```c
// there's still a race condition here. process aren't atomic
char filename[1000];
sprintf(filename, "/tmp/sort.%d", getrandom());
while(fd = open(filename, O_RDONLY)  >= 0 ) close(fd);
open("/tmp/sort.1", O_TRUNC | O_ CREATE | O_RDWR, 0666);
```

To solve this, we need a lock on `/tmp`
- `fentl(fd, F_SETLK, F_SETLKW, F_GETLK, p)`
    - lets you get locks, set locks and wait, and pointer to 
    - but what if program sets lock and forgets to clear it?
    - and we can create bottlenecks
    - possibility of deadlock

Solution
we can have the flags `O_CREATE | O_EXCL`
- we have exclusive access to creating this file
- if two files try to access it at the same time, one will fail, other succeed

## Interrupts and Signals
For example, we want our c program to deal with power failure
<br>
Solutions
- `/dev/power` - you can read this file to see if the power is okay
    - and check every second (called polling)
    - this obviously isn't a very good idea

Instead, we use signals
we can have a `power.c` program, with two parts
1. setup/configure function. if power goes out, ouch
2. ouch

In our main function, we still need to call setup 

```c
#include <signal.h>
typedef void(* handler_t)(int); 
// pointer to a function that takes an int and returns void
// SIGINT, SIGHIP, SIGPIPE, ... SIGPOWER
// SIGCHLD - signal when child dies
// SIGKILL - 9 - cannot be caught or ignored (ALWAYS KILLS OFF)
// $ kill -9 27 # kill with no appeal 

void ouch(int sig) {
    print("power low");
    save_file();
    exit(3);
}

void configure() {
    signal(SIGPOWER, ouch);
}
```

```c
handler_t signal(int, handler_t); // sig_ign : ignore signal
                                  // default for signal
                                  // SIG_STOP: cntrl z

```

What happens when we call `execlp`
- Since `execlp` replaces your program, all the signal handlers go away

### Sending Signals
signal to a process <br>
`kill(SIGNINT, 27)`
- only can send signals to your own processes 
