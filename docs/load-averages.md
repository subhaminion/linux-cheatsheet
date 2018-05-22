## System Load Averages

You might've already seen how tools commands like `top`, `uptime`, `w` display the `load average` information:

```bash
$ uptime
 00:23:53 up  3:22,  1 user,  load average: 0.00, 0.01, 0.05
```

But what is system load and those numbers `0.00, 0.01, 0.05` mean?

### System load & average system load

**System load** is the number of process in `runnable` or `uninterruptable` state at a given moment. A process in a [runnable state](process-states.md) is either using the CPU or waiting to use the CPU. A process in [uninterruptable state](process-states.md) is waiting for some I/O access, e.g. waiting for disk.

So in simple words, **system load** shows the number of processes running or waiting for a resource (e.g. CPU or disk) to become available inside the Linux operating system before they can be run. For example, when one process is actively using the CPU (running), and two are waiting their turn, the system load is 3. Thus, system load is essentially the length of the run-queue that the system has at a given moment.

But system load fluctuates quickly because of short-lived processes and can jump from zero to 5 in milliseconds and back again the next instant. Because of this volatility, it’s more useful to look at the **average system load** over time, which gives a better overview of the load the system has been under.

This is why the numbers `0.00, 0.01, 0.05` show the system load averaged over the last minute (`0.00`), the last 5 minutes (`0.01`) and the last 15 minutes (`0.05`) accordingly.

If this definition of system load averages still seems confusing to you, you might want to read [this nice post](http://blog.scoutapp.com/articles/2009/07/31/understanding-load-averages) where the author uses machine traffic analogy to explain the load average in a more digestable way.

### Interpreting the numbers

Now that we've figured out what system load averages mean, how do we interpret those numbers?

First of all, in order to interpret those numbers we need to know the number of CPUs in system. Why? Because the system load averages are not normalized for the number of CPUs in a system.

This means that a load average of 1 can mean different things depending on how many CPUs we have. For example, a system with one CPU can handle one process at a time and in this case it would be fully loaded all the time. But for a 4 CPU system which can run concurrent processes the load average of 1 would mean that the system was idle 75% of the time.

Can a load average be higher than the number of CPUs in the system? Yes!

If the load average rises to 1.5, on a single CPU system this means that the CPU was busy all the time while there was on average one other process waiting in the run-queue for 50% of the time. Our system is **overloaded** in this case, meaning the CPU has more work than it can handle.

The general rule of thumb is that the load average shouldn’t exceed the number of CPU in the machine. If the number of CPUs is four, the load should generally stay under 4.0.

Usually, it’s fine if the load average is above `1.0` per CPU in the last minute mark, but elevated load in the five or fifteen-minute averages could indicate that the system is overloaded and might have a perfomance problem.

Some other useful interpretations of load averages:

* If the averages are 0.0, then your system is idle.
* If the 1 minute average is higher than the 5 or 15 minute averages, then load is increasing.
* If the 1 minute average is lower than the 5 or 15 minute averages, then load is decreasing.

### Counting CPUs

We've seen how we can interpret the load averages, but we can't do it without first finding out the number of CPUs in the system.

So how do we count the CPUs?

First, we have to know what CPU types exist.

A single computer system can have several **physical CPUs**. In which case, it's a multi-processor system.

But a single physical CPU can have at least two or more separate **cores** (or what we can also refer to as **processing unit**) that work in parallel. Meaning a dual-core has 2 two processing units, a quad-core has 4 processing units and so on.

Furthermore, there is also a processor technology which was first introduced by Intel to improve parallel computing, referred to as **hyper-threading**. Under hyper-threading, a single physical CPU core appears as two **logical cores** to an operating system (but in reality, there is one physical hardware component).

The reason the technologies such as multiple CPUs/processors, multi-core CPUs and hyper-threading were brought to life, was because a single CPU core system can only carry out one task at a time. With more than one CPU, several programs can be executed simultaneously.

To find out the number of processing units available in the system, you can use any of the following commands:

```bash
$ nproc
1

$ lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                1
...

$ grep 'model name' /proc/cpuinfo | wc -l
1
```

### A few catches

You should remember that the load averages count not only processes in the [runnable state](process-states.md) which either run or wait for CPU to become available, but also processes in the [uninterruptable state](process-states.md) which can be waiting some I/O access. It basically, measures the number of processes that aren't completely idle.

This means that the load averages technically don't reflect only the CPU demand as you might expect.
[Disk I/O and network I/O also affect the load average numbers.](https://prutser.wordpress.com/2012/05/05/understanding-linux-load-average-part-2/)

I also recomend reading [a great post by Brendan Gregg](http://www.brendangregg.com/blog/2017-08-08/linux-load-averages.html) in which he investigates why the load averages work this way.

### Resources used to create this document:

* https://blog.appsignal.com/2018/03/28/understanding-system-load-and-load-averages.html
* man uptime
* https://prutser.wordpress.com/2012/04/23/understanding-linux-load-average-part-1/
* https://www.tecmint.com/understand-linux-load-averages-and-monitor-performance/
* http://www.brendangregg.com/USEmethod/use-linux.html
