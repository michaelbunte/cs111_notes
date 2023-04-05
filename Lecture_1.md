# Lecture 1 - Syllabus & Introduction
"We didn't off a ready-made programme, but an entire operating system."\
-Maria Wiesband, a leader of the Pirate Party, Germany
## Introduction
An operating system is too big and complex to treat as "just a program"

This course both looks at the practical and the theory 

| category | hours |
|----------|-------|
| lecture  |  3.5  |
| lab      |  1.7  |
| outside study | 15 |

Website: www.cs.ucla.edu/classes/spring23/cs111
<br>
<br>
### Grading:
|category | % |
|-|-|
| midterm | 25% |
| final | 35% |
| warmup lab | 3% | 
| other four labs | 32% |
| report on operating systems | 5% | 

<br>

- You're encouraged to work with other people on labs, but you should write all 
the code and all the documentation yourself. 
- If there is code off the net that you want to use, ask in advance
- Assignments are due on bruinlearn at 23:55 on the due date. 
- Lateness penalty is the same as 35L:
- 1%, 2%, 4%, 8%, 16%, 32%, 64%
- However, all asignments are due by Friday week 10
- exams are closed book
- Cite other's work.
    - Use DOI (Dot Object Interface)

<br>
<br>
### example of a question
Here's a dumb idea, explain why it's dumb


## Prerequisites 
- CS 32 - Data structures, algorithms, C++
- CS 33 - Machine organization, machine instructions and architecture
- CS 35L - Software construction (scripting, OS basics from the user's viewpoint)

### Textbooks
- *Operating Systems, Three Easy Pieces* by Arpaci Dusseau
    - pages.cs.wisc.edu/~remzi/OSTEP/#book-chapters
    - operating systems
    - 2018
- *Principles of Computer System Design: An Introduction* by Salter and Kaashack
  (SK)
    - www.sciencedirect.com/book/9780123749574/principles-of-computer-system-design
    - more about system principals
    - 2009
- Mark Kampe
    - specific readings provided at www.cs.ucla.edu/classes/spring23/cs111/syllabus.html
    - 2016

### Readings for next lecture
- AD §1–§2, §36 • SK §1, §2–§2.3 • MK Software interface standards
---
## What is a System?
> Oxford English Dictionary Definition: <br>
An organized and connected group of objects 

> Another Definition: <br>
A set of principles, etc, a scheme, method 

> From ancient Greek: <br>
An organized whole, government, a constitution, or a flock of birds 

> From SK <br>
A set of interconnected components that have a specified behavior observed 
at the interface with its environment

> More casually / from the view of computer scientists <br>
 People can look from the outside and see the system, and the system is built
 of components that work together

## What is an Operating System
> American Heritage Dictionary - This definition would be good in the 1960's<br>
Software designed to control the hardware of a specific data processing system
in order to allow users to  and application programs to make use of it
- there are some issues with this definition
- "specific" is incorrect. Implies that there must be different OS for different
  machines. "specific" here is obselete, we have portable OS's
> Encarta (2007) <br>
  Master control program in a computer
- implies that you are not in control
- however, this is a more accurate definition 
- your program can't do anything without the operating system's permission
    - so the word "control" is accurate
- However, it doesn't explain why we need them
> Wikipedia v1146613150, 2023-03-23 <br>
  System software manages computer hardware, software resources, and provides
  common services for computer programs.
- this is good, as operating systems are in charge of managing resources
    - such as the SEASnet servers, which have to decide who gets RAM
- "common services" is good too
    - in software development, we often need to use the same programs, so it
      is nice to migrate these programs to the operating system
    - however, it's important to remember to not go over and bloat your OS

### Aside - Services of OS's
- piping the output to one program to the input of another
- some kind of authentication system for applications
---
## OS Goals 
#### Protection 
- Applications should be protected from outside influence or other applications
  on the same system
- provides security. You don't want other applications looking at your bank
  account info. 
- Programs can also go bad, not just because they are malicious. They can also
  be buggy. (Liability)
- You don't want your whole system to go down just because one program to do a 
  bad thing

#### Robustness
- You want your system to respond well to "weird" stuff happening 
- Systems that don't fall over with unusual influence 

#### Utilization
- We want every GPU/CPU cycle to be part of an acutal computation
- use your hardware up 
- you spend so much money building it, so you should try to use all of it

#### Performance 
- "end user sees good performance"
- this can often collide with utilization

#### Flexibility
- You want your OS to be general purpose, and run anything you'd like 
- Easy to configure for many different kinds of applications

#### Simplicity
- If it's simple, it's easy to explain, implement, and document

#### Slim 
- simple, small, good performance

#### Compatibility
- Want your progams written in COBOL decades ago to still work 

#### Portability 
- For hardware 
- For the application 

#### Inspectability
- be able to peer into the OS and see what is going on 
- open source ( not the same thing, but very related )
---
## OS Tools
#### Abstraction
- solution technique for managing complexity 
- so complex that no one understands every part of them
- if we want to build Linux, we build an abstract model of how Linux works 

#### Modularity (similar to abstraction)
- Have a big operating system that we break into pieces

#### Interface vs implementation
- interface
    - part of your class that you expose to the outside world
- implementation
    - how you build the inside

#### Policy vs mechanism 
- policy 
    - ex: "interactive apps get higher priority than batch apps"
- mechanism
    - how we implement our policy 
    - users don't care about this 

#### Measurements and Monitoring
- check to see that our OS is working how we want to 
- measure and monitor performance and correctness 

## AD - Main Problems
#### Virtualization 
- how an operating system can pretend to be a machine even though it's not 
#### Concurrency
- managing multiple applications at the same time 
#### Persistence
- if your computer crashes and your RAM loses all it's information, you know 
  that you can get your data back

## SK - System Problems
1. Incommensurate scaling 
    - you have a system that works pretty well at first, but stops working
      when your grow it
    - ex: bubble sort, great for small arrays
    - we don't see 9 feet tall people because the human physiology breaks down
      at the extreme 
    - economies of scale
        - pin factories. Difference in per capita cost of making pins depends on 
          how many pins you have 
    - diseconomies of scale 
        - star network - everyone connects to a central hub 
        - works well with a small number of people but not 400 
2. Emergent properties 
    - problems that arise that you didn't anticipate when things are small
    - ex: computer virus in the 1960s, wasn't even a concern at the time 
3. Propogation of events (butterfly effect)
    - unanticipated module boundary crossings 
    - for example, you have <br>
        - module 1, which treates paths like `abcd/efg/heic/jkl`, like a path
          and file name
        - module 2, which takes shift.jis
            - 2 bytes, with the first byte having a flag saying that this isn't
              ASCII
            - so we can have names like `ビッ/グバ/ガ`
        - the issue with combining these two modules together means that the 
          modules interperet the same text differently 
        - microsoft solved this issue by migrating some of the code of mod2 to 
          mod1 
4. Problem of Tradeoffs
    - also called the waterbed effect, if you sit on one side of the waterbed,
      other spots will go up 
    - focusing on one problem makes other problems worse 
5. Complexity
    - Complexity keeps growing 
    - ex: number of transitors on a chip doubles every two years (Moores Laws)
6. Kryder's Law
    - Capacity of hard disk drives, how many bits can you put on a drive 
    - doubles every 2 months
    - cpu speed has unfortuantely plateaued, and this is something we need to
      consider