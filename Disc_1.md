# Discussion 1
## Linux Kernel Model
### User Space Applications
browers, terminal, other user space programs
### Kernel Space
- Connect user space programs to hardware
- Expose APIs for user space programs to use
- listens to to certain events
- allows injecting/removing kernel modules

<!-- kernelmmode, user mode, system call -->

<!-- | ring | - | privilege level|
| - | - | - | -->

## Git
### git configuration
``` bash
git config --global user.name "Your full name"
git config --global user.email "your@email.com"
```
### generate SSH public key
```bash
ssh-kegen -o
cd ~/.ssh
ls
```
### Other git commands
```bash
git status
git add myFile
git add . // adds everything in the current directory and downwards
```
## Lab0
### How do we get files out of the virtualbox?
The easy way is to synchronize our files with github 
Our project is to count how many processes are running in `/proc/c  ount`

- For our submission, we need to modify `proc_count.c` and only this file
- and do `readme`
