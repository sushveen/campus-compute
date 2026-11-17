
# CampusCompute

**Intelligent Job Scheduling and Resource Management**  
UE25CS242A: Operating Systems

## 1. Project Overview

CampusCompute is an Operating Systems project that simulates jobs submitted to a university computing lab. It brings together CPU scheduling, job management, thread synchronization, inter-process communication, deadlock avoidance and resource management in a single command-line application.

CPU scheduling is implemented as a deterministic simulation using a logical clock. The logger demonstrates real POSIX threads, while the IPC, worker and file-size experiments use actual Linux child processes.

The project uses GCC and Make for compilation and relies on standard Ubuntu command-line tools.

## 2. Build Environment

The project is designed to run on Ubuntu and WSL Ubuntu. It uses GCC, Make and Bash to compile and execute the application.

The Makefile manages compilation and provides targets for building, testing, rebuilding and cleaning the project. A clean build removes previously generated executable, object and dependency files before compiling the source files again.

The project also includes ZIP utilities for packaging the source code and documentation into a submission archive.

## 3. Command-Line Interface

CampusCompute provides a command-line interface that allows each module to be executed independently. The supported commands cover job management, scheduling algorithms, logging, IPC, Banker's Algorithm, worker processes, resource limits, integrated demonstrations and cleanup.

The application supports built-in sample jobs as well as CSV input files. When no CSV file is provided, or when `-` is specified, the scheduler uses a repeatable built-in sample.

Round Robin accepts a configurable time quantum, with a default value of 2. The application also uses exit codes to distinguish successful execution, operational failures and incorrect command usage.

A successful exit status is returned for completed operations, including the demonstration of an unsafe Banker state and a worker that intentionally exits with status 42.

## 4. Job Management and CPU Scheduling

The job management module reads process information from CSV files. Each job contains an ID, arrival time, burst time and priority.

The CSV loader validates the input before scheduling. It supports 1 to 128 jobs, requires unique positive integer IDs, nonnegative arrival times, burst times between 1 and 10,000, and priorities between 0 and 99.

Three CPU scheduling algorithms are implemented:

### First Come First Serve (FCFS)

FCFS selects the job that arrives earliest. If multiple jobs have the same arrival time, their input order determines which one runs first.

It is a non-preemptive scheduling algorithm, meaning that once a job starts executing, it continues until its burst is complete.

### Round Robin (RR)

Round Robin uses a FIFO ready queue and assigns each job a fixed time quantum. The default quantum is 2, but it can be changed through the command line.

A job executes for one time slice at a time. If it does not finish within that interval, it is placed back into the ready queue. Newly arrived jobs are added to the queue before the preempted job, including jobs arriving exactly at the end of a time slice.

This allows CPU time to be distributed among ready jobs.

### Priority Scheduling

Priority Scheduling is non-preemptive. The scheduler selects the ready job with the lowest priority number.

If multiple jobs have the same priority, arrival time is considered first, followed by input order. Once a job starts executing, it continues until completion.

### Scheduling Metrics

The scheduler displays execution timelines, including idle intervals. Each timeline interval follows the format `[a,b)`, where the starting time is included and the ending time is excluded.

The following performance metrics are calculated:

- **Completion Time (CT):** The time at which a job finishes execution.
- **Turnaround Time (TAT):** The total time between a job's arrival and completion.
- **Waiting Time (WT):** The time a job spends waiting in the ready queue, calculated as turnaround time minus burst time.
- **CPU Utilization:** The percentage of elapsed time during which the CPU is busy.
- **Throughput:** The number of completed jobs per unit of elapsed time.

The comparison module executes all three scheduling algorithms on separate copies of the same workload. Round Robin uses a quantum of 2 for this comparison.

## 5. Logger and Thread Synchronization

The logger module demonstrates the Producer-Consumer problem using POSIX threads.

Two producer threads generate five lab-job lifecycle events each. These events are placed into a shared circular queue containing three slots. A consumer thread retrieves the events and appends them to `logs/campuscompute.log`.

A mutex protects the shared queue and its counters, ensuring that multiple threads do not modify the same data simultaneously.

Two condition variables coordinate the producers and consumer:

- The not-full condition allows producers to wait when the queue has no available slots.
- The not-empty condition allows the consumer to wait when the queue contains no events.

Both conditions are checked again after every wakeup to ensure that the required condition is actually satisfied.

The consumer releases the mutex before writing an event to the log file. Once both producers finish, the main thread closes the queue and wakes the consumer. The consumer processes any remaining events before terminating.

This module demonstrates thread creation, synchronization, mutual exclusion and safe communication through a bounded buffer.

## 6. Inter-Process Communication (IPC)

The IPC module demonstrates communication between a parent process and a child process using shared memory and named semaphores.

The application creates exclusive shared-memory and semaphore objects before creating the child process using `fork()`.

The shared memory mapping allows both processes to access the same data. Semaphores coordinate the order in which the parent and child perform their operations.

The communication process works as follows:

1. The parent publishes job information in shared memory.
2. The parent signals the child using the request semaphore.
3. The child waits for the request.
4. The child calculates the number of two-unit CPU slices required for the job.
5. The child publishes its result and signals the reply semaphore.
6. The parent collects the child's status and reads the result.

The IPC object names are removed after their handles have been opened. The existing mappings and semaphore handles keep the objects available until both processes release them.

This demonstrates process creation, shared memory, semaphore synchronization and parent-child communication.

## 7. Deadlock Avoidance Using Banker's Algorithm

The Banker's Algorithm module performs a safety check on resource allocation to determine whether the system is in a safe state.

The input files contain whitespace-separated integers representing the number of processes, resource types, available resources, allocation matrix and maximum requirement matrix.

The module calculates the remaining resource requirement using:

**Need = Maximum − Allocation**

It then checks whether a process can complete with the resources currently available in Work.

If a process can finish, its allocated resources are returned to Work. The algorithm continues checking the remaining processes until either every process has completed or no further progress can be made.

If all processes can finish, the system is considered safe and the resulting order is displayed as a safe sequence.

If no further process can complete, the state is considered unsafe.

The project includes separate test cases for safe and unsafe resource-allocation states.

## 8. Worker Processes and Process Management

The worker module demonstrates process creation, execution and termination using Linux child processes.

A child process is created using `fork()`. The parent waits for the child using `waitpid()` and checks its termination status.

Two execution modes are implemented:

- **Normal mode:** The child calculates the sum of the first ten squares and exits with status 0.
- **Failure mode:** The child deliberately exits with status 42.

The parent process identifies whether the child exited normally or was terminated by a signal.

This module demonstrates process creation, parent-child relationships, exit statuses and the importance of reaping terminated child processes.

## 9. File-Size Resource Limits

The resource-limit module demonstrates how Linux enforces process resource restrictions using `RLIMIT_FSIZE`.

The soft limit represents the file-size threshold currently enforced by the operating system, while the hard limit represents the maximum value to which the soft limit can be raised.

The child process lowers its file-size soft limit to at most 1024 bytes while keeping the existing hard limit unchanged.

The child locally ignores `SIGXFSZ` so that it can inspect the results of write operations and the associated error codes.

When a process attempts to write beyond the permitted file size, a write operation may transfer some bytes before a subsequent write fails with `EFBIG`.

The initial resource limit determines how many bytes can be written during the experiment.

The temporary file is unlinked after it is opened. Its open file descriptor keeps it accessible until the experiment finishes.

## 10. Cleanup and Resource Management

Each execution manages its own IPC objects and temporary files.

The cleanup functionality checks for pending resources and retries cleanup operations that the current process is tracking.

A new invocation does not own IPC names created by earlier invocations and does not scan the system globally.

The application log is retained as evidence of execution. This allows the project to demonstrate resource cleanup without deleting the recorded activity history.

## 11. Testing and Limitations

The project includes a test suite that builds the executable and runs timed command-line assertions.

The tests cover scheduling timelines, performance metrics, error codes, log records, IPC communication, Banker outcomes, child process statuses, file-size limits, cleanup operations and successful and failing integrated demonstrations.

Logger and integrated demonstration tests use a temporary directory so that previously generated evidence logs remain untouched.

The project has the following implementation limitations:

- The scheduler models a single CPU and one burst per job.
- Context-switching cost and I/O blocking are not included.
- Banker's Algorithm performs a static safety check.
- Each operating system demonstration uses a small, fixed workload.
- Thread interleaving, timestamps, process IDs and actual write results may vary between executions.
- Abrupt termination between exclusive IPC creation and early unlinking may leave an owned name behind.
- Cleanup ownership is tracked only within the running invocation.

## 12. Technical Concepts Demonstrated

### Process vs Thread

A process has its own address space, whereas threads within a process share the same address space. Threads have their own stacks and execution states.

### fork() and waitpid()

`fork()` creates a child process. It returns zero in the child and the child's process ID in the parent. `waitpid()` collects the child's termination status and releases its exit record.

### FCFS, Round Robin and Priority Scheduling

FCFS follows job arrival order. Round Robin gives ready jobs repeated CPU turns using a fixed time quantum. Priority Scheduling selects the most important ready job at dispatch time.

### Preemption

Preemption occurs when a running process is interrupted so that another process can use the CPU. Round Robin supports preemption when its time quantum expires. FCFS and non-preemptive Priority Scheduling allow a dispatched job to complete its entire burst.

### Turnaround Time and Waiting Time

Turnaround time represents the total time between arrival and completion. Since the project uses one CPU burst per job and does not include blocking I/O, waiting time is calculated by subtracting burst time from turnaround time.

### Mutex vs Semaphore

A mutex protects a critical section and has an owner. A semaphore maintains permits that can be released by one thread or process and acquired by another.

### Condition Variables

A condition-variable wait releases the associated mutex while the thread is blocked and reacquires it before returning. The thread checks its condition in a loop because a wakeup does not necessarily mean the condition has become true.

### Producer-Consumer Problem

Producers wait when a bounded buffer is full, while the consumer waits when it is empty. A mutex and two condition variables coordinate access to the shared queue.

### Shared Memory

Shared memory allows multiple processes to access the same memory region. In this project, semaphores control when each process can read or update shared information.

### Race Conditions

A race condition occurs when the result of concurrent operations depends on their execution order. The logger uses a mutex to keep shared queue slots and counters consistent.

### Deadlock vs Unsafe State

Deadlock occurs when processes are blocked waiting for progress that cannot happen. An unsafe state means the system cannot guarantee that all processes will complete under their declared maximum resource requirements, even if no deadlock currently exists.

### Banker's Safety Algorithm

The algorithm checks whether processes can finish with the currently available resources. When a process can complete, its allocated resources are returned to Work. If every process can finish, the resulting order is a safe sequence.

### Resource Limits

Linux enforces resource limits on processes. The project demonstrates how a child process changes its own file-size soft limit and how the kernel handles write operations that exceed it.

### Zombie Processes

A terminated child process retains an exit record until its parent collects its status. The project uses `waitpid()` to reap each child process created during its demonstrations.

## 13. Conclusion

CampusCompute brings together several fundamental Operating Systems concepts in one application. It demonstrates CPU scheduling, process management, thread synchronization, inter-process communication, deadlock avoidance, resource management and file handling.

The project combines deterministic scheduling simulations with real Linux process and thread operations, providing practical demonstrations of how operating system mechanisms work and interact.
