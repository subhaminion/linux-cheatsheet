## States of a process in Linux

### Running & Runnable states

The most precious resource in the system is the CPU. The process that is executing and using the CPU at a particular moment is called a **running process**. You can run the [ps](ps.md) and [top](top.md) commands to see the state of each process. If a process is running, the **Running state** is shown as **R** in the state field.

When a process is in a **Runnable state**, it means it has all the resources it needs to run, except that the CPU is not available. The Runnable state of this process is shown as **R** in [ps](ps.md) output.

Consider a example. A process is dealing with I/O, so it does not immediately need the CPU. When the
process finishes the I/O operation, a signal is generated to the CPU and the scheduler keeps that process in the **run queue** (the list of ready-to-run processes maintained by the kernel). When the CPU is available, this process will enter into Running state.

### Sleeping state

A process enters a **Sleeping state** when it needs resources/events that are not currently available. At that point, it neither goes voluntarily into Sleep state or the kernel puts it into Sleep state. Going into Sleep state means the process immediately gives up its access to the CPU.

When the resource/event the process is waiting on becomes available, a signal is sent to the CPU. The next time the scheduler gets a chance to schedule this sleeping process, the scheduler will put the process either in Running or Runnable state.

Sometimes processes go into Sleep state for a particular amount of time. The Linux kernel uses the
**sleep()** function, which takes a time value as a parameter that specifies the minimum amount of time (in seconds that the process is set to sleep before resuming execution). This causes the CPU to suspend the process and continue executing other processes until the sleep cycle has finished. When the sleep cycle ends, the scheduler pushes the process to a run queue. When the CPU gets free time, the process gets run (goes into Running state).

There are two types of sleep states: **Interruptible** and **Uninterruptable sleep states**.

#### Interruptible sleep state

An Interruptible sleep state means the process is waiting either for a particular time slot or for a particular event to occur. Once one of those things occurs, the process will come out of Interruptible sleep. Output from the ps command will show **S** in the state field of such process.

#### Uninterruptible sleep state

A process in Uninterruptible sleep state cannot be killed by a signal (this is where the name comes from). It will wake only as a result of a waited-upon resource becoming available or after a time-out occurs during that wait (if the time-out is specified when the process is put to sleep). Output from the ps command will show **D** in the state field of such process.

The Uninterruptible state is mostly used by device drivers waiting for disk or network I/O. When the process is sleeping uninterruptibly, signals accumulated during the sleep are noticed when the process returns from the system call or trap.

### Terminated/Stopped & Zombie states

Processes can end when they call the exit system themselves or receive signals to end. When a process
runs the exit system call, it gives up all the occupied resources but does not release entry in process table. Instead, it sends a SIGCHLD signal to the parent. It is up to the parent process to release the child process entry so that the parent can determine if the process exited successfully.

Between the time when the process terminates and the parent clears out process entry in process table, the child enters into what is referred to as a **Zombie state**. A process can remain in a Zombie state if the parent process dies before it has a chance to release the child process's slot in process table. The reason you cannot kill a Zombie process is that you cannot send a signal to the process to kill it as the process no longer exists. A Zombie process has a **Z** in the state field from the output of the ps command.

## Resources used to create this document:

* https://access.redhat.com/sites/default/files/attachments/processstates_20120831.pdf
* https://kerneltalks.com/linux/process-states-in-linux/
