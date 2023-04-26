

https://www.youtube.com/watch?v=fOaK6oRqhEo
## Pipe
> Cretes a uni-directional communication channel between two processes

```c
int pipefd[2];
pipe(pipefd);
/*
 0 | read end
 1 | write end

 write     read
 -> ======= ->
*/
```

## Dup2
```c
dup2(old_fd, new_fd);

/*

closes newfd (if open)
copies fdtab[old_fd] -> fdtab[new_fd]
*/
```
So what's the purpose?
