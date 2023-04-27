# Scheduling
Algorithims
- FCFS
- SJF

## Shortest Job First
- has an issue with fairness
> metric of fairness: if you add a job to a system, it will eventually 
  finish

**preemption** is a solution to the fairness problem

**Round Robin Scheduling** is a common technique for establishing 
fairness

| process | arrival | run | wait time |
|-|-|-|-
A|0|5|0
B|1|2|$1\delta$
C|2|9|$2\delta$
D|3|4|$3\delta$

`ABCDABCDACDACDACCCCC`

- So average wait time is $1.5\delta$
- but the throughput is worse, because of all of the context switching

## More ideas
- put arrivals at the end of the queue
    - this allows you to avoid starvation
- put processes on a stack
    - can cause starvation

## Priority Scheduling
Some processes are more important than others, so they should be 
scheduled earlier than other processes 

### Highest Priority Scheduling
- A: is a garbage collection process
- B: real time interrupt

You prolly want to put B at a higher level

#### Linux: Processes have niceness values
- range between -20 to 20
- -20 processes are greedy
- 20 will let anyone else run ahead of them

### Static vs dynamic priority
#### Static: everything is assigned a priority at the the beginning
- very simple robust systems
#### Dynamic: Assigned by the OS as the process run