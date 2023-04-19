## Logistics
Lab 1 is due 4/28

## Lab 1
- we are writing low level code performed by the pipe `|` operator in shells

For example
```bash
./pipe ls cat wc    # this is your program
ls | cat | wc       # this is system implementation
                    # both should print the same thing
```

```bash
ls      # lists all file and directories in teh present working directory
ls -a   # lists hidden files too
ls -l   # lists files and directories with detailed info 
```

```bash
wc
lines words characters
```
```bash
> # redirect to a file (take output from stdout and place into a file)
< # read from a file (copies data from file into standard input)
| # take everything from stdout from prog1 and pass into stdin of prog2
```

so the task is 
```bash
./pipe a b c d e f g
a | b | c | d | e | f | g
```
Both results should be the same

## 3 sub-components of lab 1
1. read an execute programs by args (a b c d e f g)
2. execution ordering align with the arguments
3. create a pip between two adjacent args
    - `argv[i]` pipes into `argv[i+1]`


example1
```c
int main(int argc, char *argv[]) {
    execlp("ls", "ls", "-a", "-l", NULL);
    /*
    replaces the current process with the new process

    we say ls twice as as ls is both the program name and first argument

    arg list ends with null
    */
}
```

### Fork API
`pid_t fork(void)`
- creates a new process (child process) by duplicating the calling process (parent process)
- child process has almost the same state as teh parent 
- on success
    - return <0 means error
    - return 0 means child process
    - return pid of the child process to the parent process (positive)

```c
int i = 1, j = 2;
int pid = fork();
    // pid = 0 for child
    // pid > 0 for parent
    // i and j are duplicated, but not objects & global variables
```

example 2:
```c
int main(argc, char *argv[]) {
    int return_code = fork();
    if(return_code == 0) {
        printf("this is the child process\n");
        execlp("ls","ls", NULL);
    } else if (return_code > 0 ) {
        printf("I am a lazy parent, letting my child ls to the directory\n");
        printf("I will just wait for their report\n");
    } else {
        printf("error");
    }
    printf("they finished! done\n");
}
```

### Wait API
A call to `wait(int *wstatus)` blocks the calling process until any one of its child processes exits or a signal is received

```c
int main(int argc, char *argv[]) {
    int return_code = fork();
    if (return_code == 0) {
        printf("this is the child process\n");
        execlp("ls", "ls","-a","-l",NULL);
    } else if (return_code > 0) {
        printf("i am a lazy parent, letting my child ls into the directory\n");
        printf("i will just wait for their report\n");
        int pid = return_code;
        int status = 0;
        waitpid(pid, &status, 0);
        print("child process exits with code: %d\n", WEXITSTTUS (status));
    } else {
        printf("error");
    }
}
```

### File descriptors
> an unsigned integer used by a process to identify an open file 

### Redirection
oftwne we don't want the stdin/stdout fd to point to their default placess
- for example `>` and `<`

`argv[i]'s stdout fd = argv[i+1]'s stdin fd`
- this don't work because of permission error of fds
- programs across the pipe are not parallel
    - there is no real time async communication
    - while waiting the output has to be temporarily stored 

### Pipe syscall
`int pipe(int fds[2])`
- `fds[0]` is file descriptor for rear end of pipe
- `fds[1]` is file descriptor for front end of pipe

Make sure to call pipe before fork, otherwise we will create two forks

### dup() and dup2()
- `int dup2(int target_fd, int prev_fd);   `
- dup - BAD
    - USES LOWEST NUMBERED UNUSED FILE DESCRIPTOR, UNCONTROLLABLE
- dup2
    - performs the same task, but redirects `fd`
    