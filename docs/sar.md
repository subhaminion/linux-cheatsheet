## sar

**SAR** stands for **System Activity Report** and as its name suggests the **sar** command is used to collect, report, or save system activity information. It can be used to monitor Linux system’s resources like CPU usage, memory utilization, I/O devices consumption, network devices, disk usage, etc.

**sar** utility comes as part of the `sysstat` package. Sysstat is a collection of Unix tools used for performance monitoring. Besides `sar`, sysstat package includes other tools such as `iostat`, `mpstat`, `pidstat`, `sadf`.

Install the `sysstat` package to start using `sar`:

```bash
$ sudo apt-get install sysstat # example for debian/ubuntu
```

After installing sysstat, we need to make sure that data collection is enabled. (For example, in Ubuntu, we can enable data collection by marking `ENABLED=”true”` in `/etc/default/sysstat`)

### Checking real-time system statistics

The syntax for viewing real-time system metrics with `sar` is the following:

```bash
$ sar <options> <interval> <count>
```

where the `interval` defines the time period in seconds between two iterations of output samples. And `count` is the number of samples to be taken. For example, the command `sar -u 2 4` will take and dispaly 4 samples of CPU usage with the time interval of 2 seconds:

```bash
$ sar -u 2 4
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

06:00:59 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
06:01:01 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
06:01:03 AM     all      0.00      0.00      0.50      0.00      0.00     99.50
06:01:05 AM     all      0.00      0.00      0.50      0.00      0.00     99.50
06:01:07 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
Average:        all      0.00      0.00      0.30      0.00      0.00     99.70
```

If the `count` is not specified, the sar command will display the results continuously until it is stopped by `ctrl + c`:

```bash
sar -u 1
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

06:05:07 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
06:05:08 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
06:05:09 AM     all      0.00      0.00      1.00      0.00      0.00     99.00
06:05:10 AM     all      0.99      0.00      0.00      0.00      0.00     99.01
...
```

#### CPU usage

To see the CPU usage report, use the `-u` option:

```bash
$ sar -u 2 3
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

06:09:04 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
06:09:06 AM     all      0.00      0.00      0.00      0.50      0.00     99.50
06:09:08 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
06:09:10 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
Average:        all      0.00      0.00      0.00      0.17      0.00     99.83
```

where
* `%user` is percentage of CPU utilization that occurred while executing user processes
* `%nice` percentage of time spent on executing processes with a manually set [nice](nice.md) values
* `%system` is percentage of CPU utilization that occurred while executing system level (kernel) processes. Note that this field includes time spent servicing hardware and software [interrupts](https://en.wikipedia.org/wiki/Interrupt)
* `%iowait` is percentage of time that the CPU or CPUs were idle during which the system had an outstanding  disk I/O request.
* `%steal` is percentage of time spent in involuntary wait because the real CPU was busy servicing another virtual machine or the host system.
* `%idle` is percentage of time that the CPU or CPUs were idle and the system did not have an outstanding disk I/O request.

In the CPU column, you could notice the value `all`, meaning that the results represent CPU usage averaged across all CPU cores. If you want to see usage percentage for each CPU, use the `-P ALL` option:

```bash
$ sar -P ALL 2 3
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

09:07:06 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
09:07:08 PM     all      0.00      0.00      0.00      0.00      0.00    100.00
09:07:08 PM       0      0.00      0.00      0.00      0.00      0.00    100.00

09:07:08 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
09:07:10 PM     all      0.00      0.00      0.50      0.00      0.00     99.50
09:07:10 PM       0      0.00      0.00      0.50      0.00      0.00     99.50

09:07:10 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
09:07:12 PM     all      0.00      0.00      0.00      0.00      0.00    100.00
09:07:12 PM       0      0.00      0.00      0.00      0.00      0.00    100.00

Average:        CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all      0.00      0.00      0.17      0.00      0.00     99.83
Average:          0      0.00      0.00      0.17      0.00      0.00     99.83
```

Here I have a single CPU and it's reported in the `CPU` column as number `0` CPU.

#### Run Queue & Load Averages

To see queue length and load averages, use the `-q` option:

```bash
$ sar -q 2 3
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

09:20:05 PM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
09:20:07 PM         0        97      0.00      0.01      0.04         0
09:20:09 PM         0        97      0.00      0.01      0.04         0
09:20:11 PM         0        97      0.00      0.01      0.04         0
Average:            0        97      0.00      0.01      0.04         0
```

where:

* `runq-sz` shows the size of the system run queue, i.e. number of tasks waiting for run time
* `plist-sz` shows the number of tasks (processes) in the task list
* `ldavg-1`, `ldavg-5` & `ldavg-15` show [load averages](load-averages.md) for the last 1, 5 and 15 minute intervals
* `blocked` shows the number of tasks currently blocked, waiting for I/O to complete.

#### Memory utilization

The `-r` option allows to see system memory utilization statistics:

```bash
$ sar -r 2 3
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

09:37:27 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
09:37:29 PM    290764    210840     42.03     12548     71692    152112     30.33    128496     45464         4
09:37:31 PM    290764    210840     42.03     12548     71692    152112     30.33    128496     45464         4
09:37:33 PM    290764    210840     42.03     12548     71692    152112     30.33    128500     45464         4
Average:       290764    210840     42.03     12548     71692    152112     30.33    128497     45464         4
```

where:

* `kbmemfree` & `kbmemused` is amount of free and used memory in kilobytes
* `%memused` is percentage of used memory
* `kbbuffers` & `kbcached` is the amount of memory used as [buffers and data cache](https://www.tldp.org/LDP/sag/html/buffer-cache.html) by the kernel.
* `kbcommit` is amount of memory in kilobytes needed for current workload. This is an estimate  of  how  much  RAM/swap is needed to guarantee that there never is out of memory ([OOM](https://en.wikipedia.org/wiki/Out_of_memory)).
* `%commit` is percentage of memory required to support the current workload in relation to the total amount of  memory (RAM+swap). This number may be greater than 100% because the kernel usually overcommits memory. See example [here](https://www.lisenet.com/2014/measure-and-troubleshoot-linux-memory-resource-usage/). If the percentage is much greater than 100%, it may indicate a memory shortage.

#### Swap Space utilization

This `-S` option goes hand in hand with the memory usage option and allows to see how Swap space is utilized:

```bash
$ sar -S 2 3
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

10:05:29 PM kbswpfree kbswpused  %swpused  kbswpcad   %swpcad
10:05:31 PM         0         0      0.00         0      0.00
10:05:33 PM         0         0      0.00         0      0.00
10:05:35 PM         0         0      0.00         0      0.00
Average:            0         0      0.00         0      0.00
```

where:

* `kbswpfree` & `kbswpused` is the amount of free & used swap space in kilobytes
* `%swpused` is percentage of used swap space

#### I/O statistics

To view information related to I/O operations, use the `-b` option:

```bash
$ sar -b 1 3
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

11:07:08 PM       tps      rtps      wtps   bread/s   bwrtn/s
11:07:09 PM      1.00      0.00      1.00      0.00      8.00
11:07:10 PM      0.00      0.00      0.00      0.00      0.00
11:07:11 PM      0.00      0.00      0.00      0.00      0.00
Average:         0.33      0.00      0.33      0.00      2.68
```

where:

* `tps` is a total number of transfers per second. A transfer is an I/O request to a physical device. Multiple logical requests can be combined into a single I/O request to the device. In other words, `tps` shows the number of I/O Operations Per Second (**IOPS**). It is the sum of `rtps` and `wtps` (see below).
* `rtps` & `wtps` is total number of read & write requests per second issued to physical devices.
* `bread/s` is total amount of data read from the devices in blocks per second. Blocks are equivalent to sectors and therefore have a size of 512 bytes.
* `bwrtn/s` is total amount of data written to devices in blocks per second.

To identify the activities by individual block devices, use the `-d` options. The output will be similar to the one given by the [iostat](iostat.md):

```bash
$ sar -dp 1 3 # the `-p` option is used to pretty print the names of devices
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

11:33:57 PM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
11:33:58 PM       sda     34.34   2787.88     64.65     83.06      0.10      2.82      1.41      4.85

11:33:58 PM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
11:33:59 PM       sda     26.26   1252.53      0.00     47.69      0.11      4.31      1.38      3.64

11:33:59 PM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
11:34:00 PM       sda      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

Average:          DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:          sda     20.13   1342.28     21.48     67.73      0.07      3.47      1.40      2.82
```

where:

* `rd_sec/s`/`wr_sec/s` is number of sectors read from/written to the device. The size of a sector is 512 bytes.
* `avgrq-sz` is the average size (in sectors) of the requests that were issued to the device.
* `avgqu-sz` is the shows [how many requests were either in the queue waiting, or being serviced](https://www.xaprb.com/blog/2010/01/09/how-linux-iostat-computes-its-results/). [Single digit numbers with the occasional double digit spike are safe(ish) values. Triple digit numbers are generally not.](https://coderwall.com/p/utc42q/understanding-iostat)
* `await` is the average time from when a request is put into the queue to when the request is completed.
* for `svctm` the man page says that this metric has been deprecated and shouldn't be trusted.
* `%util`  is the total time during which I/O operations were in progress, divided by the sampling interval. This tells you how much of the time the device was busy, but it doesn’t really tell you whether it’s reaching its limit of throughput, because the device could be backed by many disks and hence capable of multiple I/O operations simultaneously and therefore [this value is useless as a saturation indicator for SSDs and RAID arrays](https://coderwall.com/p/utc42q/understanding-iostat).

#### Network device statistics

```bash
$ sar -n DEV 2 2
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/29/2018 	_x86_64_	(1 CPU)

12:16:42 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
12:16:44 AM      eth0      3.50      3.00      0.24      0.27      0.00      0.00      0.00      0.00
12:16:44 AM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

12:16:44 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
12:16:46 AM      eth0      8.50      8.50      0.54      0.76      0.00      0.00      0.00      0.00
12:16:46 AM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
Average:         eth0      6.00      5.75      0.39      0.52      0.00      0.00      0.00      0.00
Average:           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

where:

* `IFACE` shows the name of the network interface
* `rxpck/s`/`txpck/s` is total number of packets received/transmitted per second
* `rxkB/s`/`txkB/s` is total number of kilobytes received/transmitted per second
* `rxmcst/s`  is number of multicast packets received per second
* `%ifutil` utilization  percentage  of the network interface. For half-duplex  interfaces,  utilization  is   calculated using the sum of `rxkB/s` and `txkB/s` as a percentage of the interface speed.  For  full-duplex,  this is the greater of `rxkB/s` or `txkB/s`.

### Collecting system statistics over time with SAR

Once you [install sysstat and enable metrics collection](#sar), the `sadc` (system activity data collector) utilty will be used to collect system metrics every 10 minutes and save it to a file. You can see it in the new cron job that got created in your system:

```bash
cat /etc/cron.d/sysstat
# The first element of the path is a directory where the debian-sa1
# script is located
PATH=/usr/lib/sysstat:/usr/sbin:/usr/sbin:/usr/bin:/sbin:/bin

# Activity reports every 10 minutes everyday
5-55/10 * * * * root command -v debian-sa1 > /dev/null && debian-sa1 1 1

# Additional run at 23:59 to rotate the statistics file
59 23 * * * root command -v debian-sa1 > /dev/null && debian-sa1 60 2
```

The `sa1` script calls `sadc` with arguments that specifies the time interval between sample, the number of samples, and the name of a file into which the binary results should be written.

The collected system metrics get written to files which can be found under `/var/log/sysstat` or `/var/log/sa` directory depending on your distribution. 

The files end with a number that denotes the day. For example, `sa04` is actually the file from the 4th day of the current month.

In my case, I have two files `sa28` and `sa29`. I installed sysstat on two yesterday which was May 28 (`sa28`) and now have two days of collected metrics data:

```bash
$ ls /var/log/sysstat/
sa28  sa29
```

By default, historical performance statistics will be saved only for past 7 days and then the files will be rotated. If you want to keep data longer, see [how to change the retention period](http://bencane.com/2012/07/08/sar-sysstat-linux-performance-statistics-with-ease/).

The data written in files are in binary format and you have to read it with `sar` to make any sense of it. All you need is to pass `-f` option and the path to the file with the data in addition to the options that were discussed above:

```bash
sar -r -f /var/log/sysstat/sa28
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

05:47:51 AM       LINUX RESTART

05:55:01 AM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
06:05:01 AM    302896    198708     39.61     11360     69584    141612     28.23    116084     47596         0
Average:       302896    198708     39.61     11360     69584    141612     28.23    116084     47596         0

08:45:01 PM       LINUX RESTART

08:55:01 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
09:05:01 PM    290840    210764     42.02     12380     71576    151600     30.22    126312     47504        28
09:15:01 PM    290684    210920     42.05     12412     71580    152112     30.33    126660     47320         8
```

It's also handy to pass numbers like `-1`, `-2` instead of specifying a file path to watch the data from yesterday (`-1`) or from two days ago (`-2`):

```bash
$ sar -r -1
Linux 3.13.0-149-generic (vagrant-ubuntu-trusty-64) 	05/28/2018 	_x86_64_	(1 CPU)

05:47:51 AM       LINUX RESTART

05:55:01 AM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
06:05:01 AM    302896    198708     39.61     11360     69584    141612     28.23    116084     47596         0
Average:       302896    198708     39.61     11360     69584    141612     28.23    116084     47596         0

08:45:01 PM       LINUX RESTART

08:55:01 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
09:05:01 PM    290840    210764     42.02     12380     71576    151600     30.22    126312     47504        28
09:15:01 PM    290684    210920     42.05     12412     71580    152112     30.33    126660     47320         8
```

### Resources used to create this document:

* https://www.geeksforgeeks.org/sar-command-linux-monitor-system-performance/
* http://bencane.com/2012/07/08/sar-sysstat-linux-performance-statistics-with-ease/
* https://kerneltalks.com/commands/sar-command-with-examples/
* https://coderwall.com/p/utc42q/understanding-iostat
* http://sebastien.godard.pagesperso-orange.fr/tutorial.html
