## top

The **top** command provides a dynamic real-time view of a running system. It displays information about existing processes and system resources utilization. It is one of the most useful tools in a sysadmin’s toolbox, and it comes pre-installed on every distribution.

Unlike other commands such as `ps`, it is _interactive_, and you can browse through the list of processes, kill a process, change its priority and so on.

Simply run `top` in your terminal to see a real-time view of your system.

There are 2 main sections in the top view: the **summary area** and the **task area**.

NOTE: please, note that there is also a more powerful alternative to `top`, that is `htop`.

## Summary area

#### Uptime, User Sessions & Load Averages

The first line in the summary area shows the same information as the [uptime](uptime.md) command, i.e. how long system has been running up, the number of user sessions, and the [load averages](load-averages.md). To hide/show this information in your top view, press **l**.

#### Tasks statistics

The second line shows statistics regarding the existing processes on your system. It displays the `total` number of existing processes and also [classify them by their state](process-states.md).

#### CPU usage

The third line shows the percentage of CPU time spent on various tasks.

The `us` value is the time the CPU spends executing user processes. Similarly, the `sy` value is the time spent on running kernel processes.

The `ni` value reflects the time spent on executing processes with a manually set [nice](nice.md) values.

This is followed by `id`, which is the time the CPU remains **idle**. Next comes the `wa` value, which is the time the CPU spends waiting for I/O to complete.

[Interrupts](https://en.wikipedia.org/wiki/Interrupt) are signals to the processor about an event that requires immediate attention. _Hardware interrupts_ are typically used by peripherals to tell the system about events, such as a keypress on a keyboard. On the other hand, software interrupts are generated due to specific instructions executed on the processor. In either case, the OS handles them, and the time spent on handling hardware and software interrupts are given by `hi` and `si` respectively.

In a virtualized environment, a part of the CPU resources are given to each virtual machine (VM). The OS detects when it has work to do, but it cannot perform them because the CPU is busy on some other VM or the host system itself. The amount of time the real CPU was not available to the current virtual machine is the `steal time`, shown as `st`.

**Useful keys:**

* This section shows CPU usage averaged across all system CPUs. To get the same information for each CPU core, press `1`.
* To hide CPU & tasks information from the view, press `t`

#### Memory

The lines marked `Mem` and `Swap` show information about RAM and swap space respectively. Simply put, a swap space is a part of the hard disk that is used like RAM. When the RAM usage gets nearly full, infrequently used regions of the RAM are written into the swap space, ready to be retrieved later when needed. However, because accessing disks are slow, relying too much on swapping can harm system performance.

As you would naturally expect, the `total`, `free` and `used` values have their usual meanings.

The `avail mem` value is the amount of memory that can be allocated to processes without causing more swapping.

The Linux kernel also tries to reduce disk access times in various ways. It maintains a `disk cache` in RAM, where frequently used regions of the disk are stored. In addition, disk writes are stored to a `disk buffer`, and the kernel eventually writes them out to the disk. The total memory consumed by them is the `buff/cache` value. It might sound like a bad thing, but it really isn’t — memory used by the cache will be allocated to processes if needed.

**Useful keys:**

* Display memory information in MiB/GiB, press `E`
* To hide memory information from the view, press `m`

## Task area

The task area shows displays detailed information about existing processes.

The information reminds the output of a `ps` command which is updated regularly. But default, the `top's` view is updated every 3 seconds, see below on how to change this interval.

Here is a short explanation of the columns that you will see here:

* `PID` is the process ID, an unique positive integer that identifies a process.
* `USER` is the [effective](real-effective-user.md) username of the user who started the process. 
* `PR` shows the scheduling priority of the process from the perspective of the kernel
* `NI` shows the "nice" value of a process. The [nice value affects the priority of a process].
* `VIRT` is the total amount of memory consumed by a process. This includes the program’s code, the data stored by the process in memory, as well as any regions of memory that have been swapped to the disk.
* `%CPU` is the process's share of the elapsed CPU time since the last screen update, expressed as a percentage of total CPU time
* `RES` & `%MEM` is the memory consumed by the process in RAM, and `%MEM` expresses this value as a percentage of the total RAM available.
* `SHR` is the amount of memory shared with other processes.
* `S` shows the process state in the single-letter form.
* `TIME+` is the total CPU time used by the process since it started.
* `COMMAND` shows the command that was used to start the process.

### Useful keys

#### Display help screen without quiting top

To view help screen interactively without exiting from top command press `h` or `?`. Press any key to exit from the help screen.

#### Show/Hide collumns

By default top displays only few columns out of many more that it can display. If you want to add or remove a particular column or change the order of columns, then press `f`.

The fields marked `*` or bold are the fields that are displayed, in the order in which they appear in this list.

Navigate the list using up/down arrow keys and press `d` to toggle the display of that field. Once done, press `q` to go back to the process list

#### Sorting processes

* `M` - sort processes by memory usage
* `P` - sort processes by CPU usage
* `T` - sort by the running time

Press `x` to highlight which field is being sorted as a quick visual reminder. You can further highlight the sorted column by pressing `b` and changing its background.

By default, top displays all results in descending order. However, you can switch to ascending order by pressing `R`.

#### Showing full command

By default, top does not show the full command that was used to start the process. It also doen't make a distinction between kernelspace processes and userspace processes. If you need this information, press `c` while top is running. Kernelspace processes are marked with square brackets around them.

#### Showing child-parent relationships of processes

You can see child-parent relationships of processes with the forest view, by pressing `V` while top is running.

#### Highlighting running processes

The default top view can seem cluttered and if you want to see only active processes (i.e those that are not idle) then you can ran the top command using pressing `i` key.

Also to change put in bold the _running_ processes, you can press `y`. You can further highlight the running processes column by pressing `b` and changing rows background.

Another option to highlight running processes is the `z` key.

#### List processes for a specific user

To list processes for a certain user, press `u` when top is running and enter the name of the user. Or launch top by using `top -u <username>`.

#### Filter through processes

If you have a lot of processes to work with, a simple sort won’t work well enough. In such a situation, you can use top’s filtering to focus on a few processes. To activate this mode, press `o/O`.

A prompt appears inside top, and you can type a filter expression here.

A filter expression is a statement that specifies a relation between an attribute and a value. Some examples of filters are:

* `COMMAND=getty`: Filter processes which contain `getty` in the COMMAND attribute.
* `!COMMAND=getty`: Filter processes which do not have “getty” in the COMMAND attribute.
* `%CPU>3.0`: Filter processes which have a CPU utilization of more than `3%`.

Once you’ve added a filter, you can further prune down things by adding more filters. To clear any filters you have added, press `=`.

#### Change the number of processes to display

Lets say you want to monitor only few processes based on a certain filter criteria. Press `n` and enter the number of processes you wish to display.

#### Change refresh interval of top command

By default, the top command updates the output every 3 seconds. You can change the interval by pressing `s` or `d` key. It will ask you to enter the new time interval.

### Resources used to create this document:

* https://superuser.com/questions/575202/understanding-top-command-in-unix
* https://www.booleanworld.com/guide-linux-top-command/
* https://www.golinuxhub.com/2014/03/8-examples-to-help-you-understand-top.html
* https://www.unixmen.com/ten-top-command-examples/
* https://www.binarytides.com/linux-top-command/
