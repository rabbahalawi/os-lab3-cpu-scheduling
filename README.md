# OS Laboratory Activity 3
## Multilevel Queue and Real-Time CPU Scheduling

### Student Information
- Name: Rabbah Alawi
- Course and Section: BSCA - B186
- Instructor: Maria Fe Bahinting
- Date Submitted: November 14, 2026

### I. Introduction
CPU scheduling is the part of an operating system that decides which process or task gets to use the processor at a given time. A good scheduling policy keeps the CPU busy, shares it fairly, and, in real-time systems, makes sure that tasks finish before their deadlines.

This laboratory activity implements three scheduling algorithms in Python: **Multilevel Queue (MLQ) Scheduling**, **Rate Monotonic Scheduling (RMS)**, and **Earliest Deadline First (EDF)**. MLQ is a general-purpose approach that separates processes into groups, each with its own scheduling policy. RMS and EDF are real-time algorithms for periodic tasks, where each task repeats at a fixed interval and must complete before its deadline. By simulating each algorithm, producing Gantt charts, and computing performance metrics, this activity shows how different scheduling rules affect the order of execution, waiting time, CPU utilization, and deadline compliance.

### II. Objectives
The objectives of this laboratory activity are to:

1. Explain how Multilevel Queue scheduling separates processes into queues with different priorities and scheduling policies.
2. Implement MLQ using Round Robin (time quantum of 2 ms) for the interactive queue and First-Come, First-Served (FCFS) for the batch queue.
3. Compute completion time, turnaround time, and waiting time for each process, along with the averages.
4. Explain and implement Rate Monotonic Scheduling as a fixed-priority, preemptive algorithm.
5. Explain and implement Earliest Deadline First scheduling as a dynamic-priority, preemptive algorithm.
6. Compute CPU utilization and apply the schedulability tests for RMS and EDF.
7. Generate Gantt charts and check for deadline misses.
8. Compare the three algorithms and use Git and GitHub to document and submit the work.

### III. Scheduling Algorithms
1. **Multilevel Queue Scheduling**
   - The ready queue is divided into separate queues, and each process is permanently assigned to one queue.
   - Each queue has its own scheduling algorithm, and the queues themselves are scheduled by fixed priority.
   - In this lab, Queue 1 (interactive) uses Round Robin and has higher priority. Queue 2 (batch) uses FCFS and runs only when Queue 1 is empty.
2. **Rate Monotonic Scheduling**
   - A preemptive, fixed-priority algorithm for periodic real-time tasks.
   - A task with a shorter period gets a higher priority, and this priority never changes.
   - A set of n tasks is guaranteed schedulable if U <= n(2^(1/n) - 1).
3. **Earliest Deadline First**
   - A preemptive, dynamic-priority algorithm for periodic real-time tasks.
   - At every moment, the ready job with the nearest absolute deadline runs first, so priorities change as new jobs are released.
   - A set of tasks (with deadline equal to period) is schedulable if U <= 1.

### IV. Program Implementation
All programs were written in Python 3 and simulate the processor in steps of 1 ms.

**`multilevel_queue/mlq.py`**
- Each process is stored as a dictionary with its PID, arrival time, burst time, and queue number. A `remaining` field tracks how much burst time is left.
- At every millisecond, the program first moves newly arrived processes into Queue 1 or Queue 2 (using `deque` as the queue structure).
- If Queue 1 has a process, it runs for 1 ms and consumes part of its 2 ms quantum. If the quantum runs out and work remains, the process goes to the back of Queue 1. Queue 1 therefore always has priority over Queue 2.
- If Queue 1 is empty, Queue 2 runs in FCFS order. A Queue 2 process keeps its remaining time when Queue 1 interrupts it and resumes later.
- If both queues are empty, the CPU is marked IDLE.
- After the simulation, the program prints the Gantt chart and computes Turnaround Time (CT - Arrival) and Waiting Time (TAT - Burst) for each process, plus the averages.

**`real_time_scheduling/rms.py`**
- Tasks are sorted by period so that the shortest period has the highest priority.
- For each task, the program generates a job (instance) at every release time within the 20 ms simulation window, each with its own absolute deadline (release + deadline).
- At every millisecond, the program selects the ready job whose task has the shortest period and runs it for 1 ms. A higher-priority job that arrives preempts a lower-priority one.
- It computes CPU utilization, compares it with the RMS bound n(2^(1/n) - 1), prints the Gantt chart, and labels each job as MET, MISSED, or PENDING.

**`real_time_scheduling/edf.py`**
- It uses the same task set and job generation as RMS.
- The only difference is the selection rule: at every millisecond, the ready job with the earliest absolute deadline runs first (ties are broken by release time, then task ID).
- It computes CPU utilization, applies the EDF test (U <= 1), prints the Gantt chart, and reports deadline results.

### V. Test Data

**Multilevel Queue (Queue 1: Round Robin, quantum = 2 ms; Queue 2: FCFS)**

| Process | Arrival Time (ms) | Burst Time (ms) | Queue |
|---|---|---|---|
| P1 | 0 | 5 | 1 |
| P2 | 1 | 4 | 1 |
| P3 | 0 | 6 | 2 |
| P4 | 2 | 3 | 2 |
| P5 | 3 | 2 | 1 |

**Real-Time Tasks for RMS and EDF (simulation time = 20 ms)**

| Task | Computation Time, C (ms) | Period, T (ms) | Relative Deadline, D (ms) |
|---|---|---|---|
| T1 | 1 | 4 | 4 |
| T2 | 1 | 5 | 5 |
| T3 | 2 | 10 | 10 |

### VI. Results and Discussion

#### A. Multilevel Queue Scheduling

**Gantt Chart**
```
| P1 [0-2] | P2 [2-4] | P1 [4-6] | P5 [6-8] | P2 [8-10] | P1 [10-11] | P3 [11-17] | P4 [17-20] |
```

**Process Performance**

| PID | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|
| P1 | 0 | 5 | 11 | 11 | 6 |
| P2 | 1 | 4 | 10 | 9 | 5 |
| P3 | 0 | 6 | 17 | 17 | 11 |
| P4 | 2 | 3 | 20 | 18 | 15 |
| P5 | 3 | 2 | 8 | 5 | 3 |

- Average Turnaround Time: **12.0 ms**
- Average Waiting Time: **8.0 ms**

**Discussion:** The Gantt chart shows that all Queue 1 processes (P1, P2, P5) run before any Queue 2 process, even though P3 arrived at time 0. P3 and P4 only start at 11 ms, after Queue 1 is empty, which is why their waiting times (11 ms and 15 ms) are the highest. Within Queue 1, Round Robin shares the CPU in 2 ms slices, so P1 and P2 alternate at the start, and P5 runs once it arrives at 3 ms. Within Queue 2, FCFS runs P3 to completion before P4. This shows the main strength of MLQ (interactive processes get quick service) and its main weakness (lower-priority processes can wait a long time, a risk known as starvation).

#### B. Rate Monotonic Scheduling (RMS)

- Priority Order: **T1 > T2 > T3** (shorter period means higher priority)
- CPU Utilization: 1/4 + 1/5 + 2/10 = **65.0%**
- RMS Sufficient Utilization Bound for 3 tasks: 3(2^(1/3) - 1) = **77.98%**
- Sufficient-Bound Test: **PASS** (65.0% <= 77.98%)

**Gantt Chart**
```
| T1 [0-1] | T2 [1-2] | T3 [2-4] | T1 [4-5] | T2 [5-6] | IDLE [6-8] | T1 [8-9] | IDLE [9-10] |
| T2 [10-11] | T3 [11-12] | T1 [12-13] | T3 [13-14] | IDLE [14-15] | T2 [15-16] | T1 [16-17] | IDLE [17-20] |
```

**Task Instance Results**

| Task | Release | Deadline | Completion | Status |
|---|---|---|---|---|
| T1 | 0 | 4 | 1 | MET |
| T2 | 0 | 5 | 2 | MET |
| T3 | 0 | 10 | 4 | MET |
| T1 | 4 | 8 | 5 | MET |
| T2 | 5 | 10 | 6 | MET |
| T1 | 8 | 12 | 9 | MET |
| T2 | 10 | 15 | 11 | MET |
| T3 | 10 | 20 | 14 | MET |
| T1 | 12 | 16 | 13 | MET |
| T2 | 15 | 20 | 16 | MET |
| T1 | 16 | 20 | 17 | MET |

- Deadline Misses: **0**

**Discussion:** Utilization is 65%, below the 77.98% bound, so the task set is guaranteed to be schedulable under RMS, and the simulation confirms it with no deadline misses. At time 0, T1 runs first because it has the shortest period, then T2, then T3. Preemption is visible at 11 to 12 ms: T3 had started running, but T1 was released at 12 ms and took over, so T3 resumed and finished at 14 ms. The CPU is idle for 7 of the 20 ms, which matches the 35% unused capacity.

#### C. Earliest Deadline First (EDF)

- CPU Utilization: **65.0%**
- Classical EDF Utilization Test (U <= 1): **PASS**

**Gantt Chart**
```
| T1 [0-1] | T2 [1-2] | T3 [2-4] | T1 [4-5] | T2 [5-6] | IDLE [6-8] | T1 [8-9] | IDLE [9-10] |
| T2 [10-11] | T3 [11-12] | T1 [12-13] | T3 [13-14] | IDLE [14-15] | T2 [15-16] | T1 [16-17] | IDLE [17-20] |
```

**Task Instance Results**

| Task | Release | Deadline | Completion | Status |
|---|---|---|---|---|
| T1 | 0 | 4 | 1 | MET |
| T2 | 0 | 5 | 2 | MET |
| T3 | 0 | 10 | 4 | MET |
| T1 | 4 | 8 | 5 | MET |
| T2 | 5 | 10 | 6 | MET |
| T1 | 8 | 12 | 9 | MET |
| T2 | 10 | 15 | 11 | MET |
| T3 | 10 | 20 | 14 | MET |
| T1 | 12 | 16 | 13 | MET |
| T2 | 15 | 20 | 16 | MET |
| T1 | 16 | 20 | 17 | MET |

- Deadline Misses: **0**

**Discussion:** EDF also meets every deadline, as its utilization (65%) is well under the 100% limit. For this task set, the EDF Gantt chart is identical to the RMS chart. The reason is that the earliest absolute deadline happens to belong to the same task that RMS would choose at every decision point. For example, at 10 ms, T2 (deadline 15) beats T3 (deadline 20), and at 12 ms, T1 (deadline 16) beats T3 (deadline 20). The two algorithms are not always equivalent, though. EDF can schedule task sets with utilization up to 100%, while RMS may fail above its bound, and EDF picks jobs by deadline rather than by a fixed priority.

### VII. Algorithm Comparison

| Criteria | MLQ | RMS | EDF |
|---|---|---|---|
| Scheduling basis | Queue priority | Task period | Absolute deadline |
| Priority assignment | Fixed queue priority | Fixed task priority | Dynamic task priority |
| Scheduling type | Depends on queue policy | Preemptive | Preemptive |
| Main application | Process classification | Periodic real-time tasks | Deadline-based real-time tasks |
| Deadline monitoring | Not inherent | Yes | Yes |

### VIII. Conclusion
This laboratory activity showed how three different scheduling algorithms decide who gets the CPU. In Multilevel Queue scheduling, the interactive Queue 1 always ran before the batch Queue 2, which gave fast service to interactive processes but made batch processes wait much longer (an average waiting time of 8.0 ms and average turnaround time of 12.0 ms). This showed both the usefulness of separating process types and the danger of starvation for the lower-priority queue.

For real-time scheduling, both RMS and EDF handled the periodic task set with 65% CPU utilization and no deadline misses. RMS uses a simple fixed priority based on the period, which is easy to implement and predict, but it can only guarantee schedulability up to a utilization bound (77.98% for three tasks). EDF uses dynamic priorities based on the nearest deadline and can use the CPU up to 100%, at the cost of recomputing priorities as jobs arrive. For this particular task set, both produced the same schedule.

Overall, the lab showed that the choice of scheduling algorithm depends on the system's goals: MLQ for general-purpose systems with different kinds of processes, RMS for simple and predictable real-time systems, and EDF for real-time systems that need higher CPU utilization. The activity also gave practice in writing simulations in Python and using Git and GitHub to document and submit the work.
