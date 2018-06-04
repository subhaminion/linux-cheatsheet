## vmstat

**vmstat** stands for [Virtual Memory](docs/virtual-memory.md) Statistics and as the name suggests provides information about the system memory utilization. But in addition to the memory statistics, it also showa  information about CPU usage, I/O activity in real time and thus becomes one of the most useful system monitoring tools.

The command has the following syntax:

```bash
$ vmstat [options] [delay [count]]
```

By specifying the `delay`, you tell the `vmstat` how often the new report with system statistics should be generated. And the `count` (if specified) regulates how many reports to produce.

For example, the following command will produce 3 system reports with 1 second delay:

```bash
$ vmstat 1 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 293568  13340  72140    0    0     7     1   10   16  0  0 100  0  0
 0  0      0 293568  13340  72140    0    0     0     0   24   31  0  0 100  0  0
 0  0      0 293560  13340  72140    0    0     0     0   13   17  0  0 100  0  0
```

**NOTE:** the first report of the `vmstat` command is special. It doesn't show you the current (real time) system statistics. Instead, it shows your the average values since the last reboot.

So the real time reports start with the second one.

Let't see what all this output means.

#### Procs

* `r`: The number of runnable processes (running or waiting for CPU to become available).
* `b`: The number of processes in [uninterruptible sleep](process-states.md), that is process which are waiting for I/O (disk, network, user input).

Remember that run-queue number (`r`) shouldn't exceed the number of CPUs on the server, otherwise it could indicate a CPU bottleneck (see [sar](sar.md) or [top](top.md) for more info).

#### Memory

* `swpd`: the amount of [virtual memory](virtual-memory.md) used. This reports how much memory has been swapped out to a swap file or disk.
* `free`: the amount of idle (unallocated) memory.
* `buff`: the amount of memory used as buffers  (see [free](free.md) for more info on this).
* `cache`: the amount of memory used as cache (see [free](free.md) for more info on this).

#### Swap

* `si:` how much memory is moved from swap space to [real memory](virtual-memory.md) (swapped in) per second.
* `so:` how much memory is moved from [real memory](virtual-memory.md) to swap space (swapped out) per second.

If these numbers are high, it means the system is starving for memory and extensively uses disk to save memory data. In other words, we have a memory bottleneck.

To find a process that eats most of your memory, you can use the [top](top.md) command which allows to sort processes by memory.

#### IO

This section reports the input and output activity statistics in terms of blocks read and written per second.

* `bi`: blocks per second received from block device.
* `bo`: blocks per second sent to a block device.

In modern Linux kernels, block size is 1024 bytes.

#### System

* `in`: the number of [interrupts](https://en.wikipedia.org/wiki/Interrupt) per second, including the clock.
* `cs`: the number of [context switches](http://www.linfo.org/context_switch.html) per second.

#### CPU

These are percentages of total CPU time.

* `us`: time spent running non-kernel code.  (user time, including nice time)
* `sy`: time spent running kernel code.  (system time)
* `id`: time spent idle.
* `wa`: time spent waiting for IO.
* `st`: time stolen from a virtual machine by the hypervisor (see [top](top.md) for more info).

### Useful options

#### Choose measurement units for memory

By default, vmstat shows memory information (`free`, `buff`, `cache`) in kilobytes. To switch the output units between 1000 (`k`), 1024 (`K`), 1000000 (`m`), or 1048576 (`M`) bytes, use the `-S` option followed by the unit letter.

For example, the following command will display memory info in megabytes:

```bash
$ vmstat -S M 1 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0    285     13     70    0    0     4     1   10   15  0  0 100  0  0
 0  0      0    285     13     70    0    0     0     0   15   17  0  0 100  0  0
 0  0      0    285     13     70    0    0     0     0   14   21  0  0 100  0  0
```

### Useful examples

[On this page](https://access.redhat.com/solutions/1160343) you can find some practical examples of how you can recognize resource bottlenecks using `vmstat`.

### Resources used to create this document:

* https://www.lazysystemadmin.com/2011/04/understanding-vmstat-output-explained.html
* https://www.linode.com/docs/uptime/monitoring/use-vmstat-to-monitor-system-performance/
* https://linoxide.com/linux-command/linux-vmstat-command-tool-report-virtual-memory-statistics/
* https://access.redhat.com/solutions/1160343
