## free

The **free** command is used to check system memory utilization. It displays the total amount of free and used physical and swap memory in the system, as well as the buffers used by the kernel.

```bash
$ free
             total       used       free     shared    buffers     cached
Mem:        501604     241712     259892        368      12580     102912
-/+ buffers/cache:     126220     375384
Swap:            0          0          0
```

To make information more human-readable and display information in megabytes and gigabytes instead of default kilobytes, use the `-h` option:

```bash
$ free -h
             total       used       free     shared    buffers     cached
Mem:          489M       236M       253M       368K        12M       100M
-/+ buffers/cache:       123M       366M
Swap:           0B         0B         0B
```

Let's break down this output.

The first two lines are giving information about **RAM** and the third line is showing the **Swap space** usage.

The first line shows the `total` capacity of system's RAM, how much of it is `used` by processes and how much of the total is `free` and `shared` among processes.

Memory lying around unused is useless, that's why kernel tries to use it as `buffers` and `disk cache`.

The part of RAM currently used as `disk cache` (shown in the `cached` column) stores frequently used regions of the disk and helps to make the programs run faster since accessing data in RAM is thousands time faster than on disk.

In addition, disk writes are stored to a `disk buffer`. On one hand, data that is written is often soon read again (e.g., a source code file is saved to a file, then read by the compiler), so putting data that is written in the cache is a good idea. On the other hand, by only putting the data into the cache, not writing it to disk at once, the program that writes runs faster. The writes can then be done in the background, without slowing down the other programs. So the `buffers` column shows the amount of RAM currently used by kernel for these purposes.

_[The memory allocated for buffers and caches is not a bad thing and doesn't steal memory from processes.](https://www.linuxatemyram.com/)_ The memory used for cache and buffers will be allocated to processes as soon as they need it.

If you're concerned about how much memory is truly being used by processes you are running, this is where the second line of `free` output comes in handy:

* `used-in-second-line = used-in-first-line - buffers - cached`
* `free-in-second-line = free-in-first-line + buffers + cached`

These calculations make sense since if your processes ask for more memory, the kernel will happily free its buffers and cached resources and hand it over.

So the second line is what you're really should paying attention to when using the `free` command!

Finally, the last line shows the `total` amount of Swap space, how much of it is `used` and `free`.

### Useful options

If you want to display the output in some paricular units, e.g. only in megabyts or gigabytes, you can use the following options:

```bash
$ free -k # Kilobytes, this is the defaul
$ free -m # Megabytes
$ free -g # Gigabytes
```

___

To show a total number in each column, use the `-t` option:

```bash
$ free -ht
             total       used       free     shared    buffers     cached
Mem:          489M       236M       253M       368K        12M       100M
-/+ buffers/cache:       123M       366M
Swap:           0B         0B         0B
Total:        489M       236M       253M
```

This shows you how much memory you have in total (`RAM + Swap`) and how it's used.

### Recommended reading:

[Why (on Linux) am I seeing so much RAM usage?](http://www.chrisjohnston.org/ubuntu/why-on-linux-am-i-seeing-so-much-ram-usage)

### Resources used to create this document:

* https://codeyarns.com/2016/06/15/free-command-in-linux/
* https://www.tldp.org/LDP/sag/html/buffer-cache.html
* https://www.linuxatemyram.com/
* man free
