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

<hr>

# Other Discussion
Encouraged to submit your discussion notes 

## VirtualBox setup
1. download virtualbox
2. import vm.ova
3. set up the shared folder 
- in a past quarter a ta set up a ssh connection up with the virtual machine

## Kernel
- lowest elvel of the OS
- connects the applications to the cpu, memory, and devices
- syscalls are the only way to communicate with the kernel
- so, our labs are all related to syscall APIs

### What is a Kernel Module
**Kernel Modules**
- are pieces of code that can be loaded and unloaded into the kernel upon demand
- they extend the functionality of the kernel without the need to reboot the system
- why not just add all the new functionalities into the kernel image 
modules are stored in `/usr/lib/modules/kernel_release`

To see what kernel modules are currently loaded use the commands
- `lsmod` or `cat /proc/modules`
- do exactly the same thing 
- this is not a regular file system. called a virtual file system
- to get info about a module: `modinfo <module name>`

## Kernel Module Structure
```c
#include <linux/module.h> // neede by all modules
#include <linux/kernel.h> // needed for KERN_INFO 
int init_module(void) {
        // special print that prints to the console 
    printk(KERN_INFO, "hello world 1.\n");
    return 0;
}
void cleanup_
```

basic mechanics of installing and removing kernel modules
```bash
insmod proc_count.ko //doesn't work
sudo insmod proc_count.ko
lsmod | grep proc_count 
sudo dmesg | grep proc 
sudo rmmod proc_count
```

## Other comments
- priveliged: user mode 
- super user still doesn't have the power that the kernel has 

## lab0 extra info 
- read the task struct, which is a linked list
- print the number of active processes with a for loop
- write into the proc file 
- when the user runs cat on the proc file, it will show the number of 
  active processes

## Writing to a file - Sequence Files
- for writing, use `seq_file` api
    - based on a sequence, whihc is composed of the four functions
    - `show()`,`stop()`,`start()`, and `stop()`
    - the seq_file API starts a sequence when a user reads the `/proc` fi.e This 
      in handy when you want to output something really big
- we are printing ~ of processes only, so we only need th show() function
- use `seq_printf` for writing to the virtual file 

### Create a virtual file - `proc_create_single()` (macro)
- `proc_create_single(name, mode, parent, show)`
- name: name of hte process file we are creating: count
- mode: permission mode
- parent: parent of this process
- show: show function we want to use 

### how to count the number of processes?
- `for_each_process(t)` iterates through all the active proccesses, basically
  a for loop
- it takes `struct task_struct*t` as an argument 

## Creating our Functions
elixir.bootlin.com

`proc_create_single`

`for_each_process`
