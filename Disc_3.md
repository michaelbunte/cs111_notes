## new notes submission guidelines
turn in notes right at the end of discussion

[Week-x-Disc-A-UID-Notes]

### Agenda - What is scheduling
- why do we need good scheduling?
- typical scheduling policies, pros and cons
    - lab2 background (round - robin, concrete example)
    - lab2 starter (coding)

### Can you depict a modern computer without scheduling
- for example, if photoshop is still rendering, you are stuck
- becaue cpu is an **preemptable resource**, sharing cpu time beteen processes 
  can increase efficiency and fairness, crucial for concurrency
- why do we need good scheduling 
    - scheduler obeys the strict ordering of incoming tasks
    - first come first serve sucks!
- maybe it would be smarter to do the lightest every time 
    - but this starves heavy jobs