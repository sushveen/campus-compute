
# Campus Compute: Project Evidence

## 1. Overview

This directory contains screenshot evidence of the development, implementation, compilation and testing of the Campus Compute Operating Systems project.

The evidence covers the major operating system concepts implemented in the application, including CPU scheduling, multithreading, synchronization, inter-process communication, deadlock avoidance, process management and resource management.

Each section describes the functionality demonstrated and includes the corresponding command used for execution.

## 2. Development Environment and Project Setup (S01–S04)

The initial evidence establishes the development environment used for the project, including the Ubuntu operating system, Linux kernel, university SRN, GCC compiler and GNU Make.

The project directory structure demonstrates the organization of source files, header files, test fixtures and documentation. A clean build verifies that all modules compile and link successfully into the final executable.

**Commands:**

```bash
printf 'SRN: %s\n' "$SRN"; uname -sr; cat /etc/os-release
gcc --version | head -n 1
make --version | head -n 1
find include src tests evidence -maxdepth 1 -type f ! -name '*.o' ! -name '*.d' | sort
ls -l Makefile README.md
make clean && make
```

## 3. Command-Line Interface (S05)

The application provides a command-line interface through which users can access the different functionalities independently. The help command displays the supported commands and their syntax.

**Command:**

```bash
./campuscompute help
```

## 4. CPU Scheduling and Job Management (S06–S09)

The job management module loads process information from CSV files. The scheduling module implements three CPU scheduling algorithms:

- **First Come First Serve (FCFS):** Processes are executed according to their arrival order.
- **Round Robin (RR):** CPU time is distributed among processes using a configurable time quantum, allowing preemption.
- **Priority Scheduling:** Processes are selected according to their assigned priority.

The scheduling outputs display execution timelines, idle intervals and performance metrics such as waiting time and turnaround time. The comparison module evaluates the three algorithms using the same workload.

**Commands:**

```bash
./campuscompute schedule fcfs tests/jobs.csv
./campuscompute schedule rr tests/jobs.csv 2
./campuscompute schedule priority tests/jobs.csv
./campuscompute compare tests/jobs.csv
```

## 5. Producer-Consumer Problem and Thread Synchronization (S10–S11)

The logger module demonstrates the Producer-Consumer problem using multiple producer threads and a consumer thread.

Producers generate activity records and place them into a shared queue, while the consumer retrieves and processes the records. Synchronization ensures coordinated access to the queue and orderly thread termination.

**Commands:**

```bash
./campuscompute logger
./campuscompute logger | tail -n 8
```

## 6. Inter-Process Communication (S12–S13)

The IPC module demonstrates communication between parent and child processes using shared memory.

The processes exchange job information through a shared memory mapping. Semaphores coordinate request and response operations, ensuring synchronized access to shared data. The implementation also demonstrates process completion and IPC resource cleanup.

**Commands:**

```bash
./campuscompute ipc
./campuscompute ipc && ./campuscompute cleanup
```

## 7. Deadlock Avoidance Using Banker's Algorithm (S14–S15)

The Banker's Algorithm module evaluates resource-allocation states to determine whether a safe execution sequence exists.

The safe-state test demonstrates the allocation matrix, remaining resource requirements, available resources and resulting safe sequence. The unsafe-state test demonstrates a resource allocation for which no safe sequence can be established.

**Commands:**

```bash
./campuscompute banker tests/banker_safe.txt
./campuscompute banker tests/banker_unsafe.txt
```

## 8. Process Creation and Failure Handling (S16–S17)

The worker module demonstrates parent-child process creation and process termination handling.

The normal execution scenario demonstrates successful child process completion. The failure scenario simulates an unsuccessful child process termination, allowing the parent to detect and report the nonzero exit status.

**Commands:**

```bash
./campuscompute worker normal
./campuscompute worker fail
```

## 9. Resource Management (S18)

The resource-limit module demonstrates operating system resource restrictions.

The implementation displays the original resource limits, applies a modified soft limit and demonstrates how the operating system enforces restrictions on file operations. Cleanup operations are also performed.

**Command:**

```bash
./campuscompute limit
```

## 10. Input Validation and Error Handling (S19)

The application is tested with invalid commands and scheduling parameters to demonstrate input validation and error handling.

An unsupported command and a zero Round Robin time quantum are supplied to verify that invalid inputs are rejected and appropriate nonzero exit codes are returned.

**Command:**

```bash
./campuscompute unknown; printf 'exit=%s\n' "$?"; ./campuscompute schedule rr tests/jobs.csv 0; printf 'exit=%s\n' "$?"
```

## 11. Cleanup and Log Management (S20, S22)

The cleanup functionality manages temporary application resources while retaining the activity log.

The generated log file contains timestamped event records, including activity descriptions and producer information. This demonstrates persistent logging of application events.

**Commands:**

```bash
./campuscompute cleanup && ls -l logs/campuscompute.log
tail -n 10 logs/campuscompute.log
```

## 12. Integrated Project Demonstration (S21)

The integrated demonstration executes multiple project functionalities through a single command. Its output provides a summary of the individual module executions and their results.

This demonstrates how the different operating system components are brought together within the Campus Compute application.

**Command:**

```bash
./campuscompute demo > demo-output.txt 2>&1; demo_status=$?; tail -n 14 demo-output.txt; printf 'demo exit=%s\n' "$demo_status"
```

## 13. Final Project Files and Submission Archive (S23–S24)

The final evidence documents the completed project structure, including the Makefile, README, source files, header files, test fixtures and compiled executable.

The submission archive packages the source code, documentation and test fixtures required to rebuild and test the application. Archive integrity is checked to verify that the generated ZIP file is valid.

**Commands:**

```bash
ls -l Makefile README.md campuscompute
find include src tests -maxdepth 1 -type f | sort
```

Create and verify the submission archive:

```bash
cd ..
zip -r "${SRN}_CampusCompute.zip" campuscompute/Makefile campuscompute/README.md campuscompute/include campuscompute/src campuscompute/tests -x '*.o' '*.d'
unzip -t "${SRN}_CampusCompute.zip"
cd campuscompute
```

## 14. Conclusion

The evidence demonstrates the implementation and execution of the major operating system concepts incorporated into Campus Compute.

The application brings together CPU scheduling, job management, multithreading, synchronization, inter-process communication, deadlock avoidance, process management, resource management and logging in a single command-line application.

The screenshots provide execution-level evidence of the individual modules, their expected behaviour, error handling and integration into the completed project.
