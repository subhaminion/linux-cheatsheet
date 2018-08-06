## Init system: System V & Upstart

In Unix-based computer operating systems, **init** (short for initialization) is the first process started during [booting of the computer system](boot-process.md). When the kernel has started itself (has been loaded into memory, has started running, and has initialized all device drivers and data structures and such), it finishes its own part of the boot process by starting a user space program, init. Kernel typically looks for `init` script in the `/sbin` directory.

Since it's the first process, init is assigend the process identifier (PID) **1**.

```bash
$ ps -f 1
UID        PID  PPID  C STIME TTY      STAT   TIME CMD
root         1     0  2 04:31 ?        Ss     0:00 /sbin/init
```

When init starts, it finishes the boot process by doing a number of administrative tasks required for system initialization, such as checking and mounting filesystems, cleaning up /tmp, setting hostname, enabling swap. 

Most commonly known function of init process is to start essential operating system and application services when system boots.

Your running Linux system will have a number of background processes executing at any time. These processes - also known as _services_ or [_daemons_](daemon.md) - may be native to the operating system, or run as part of an application.

Examples of operating system services:

* `sshd` daemon that allows remote connections
* `crond` daemon which is a time-based job scheduler
* `ntpd` daemon that manages clock synchronization across the network
* `syslogd` daemon which collects various system log messages

Examples of application daemons:

* `httpd/apache2` is a web server service
* `mongod` is a database daemon

We are likely to want these processes to be running continuously, and in cases when a system boots/reboots, we want to make sure these services are started/restarted automatically, i.e without our intervention. That is why we delegate this job of starting these key services to init process.

_Therefore, when the init proccess starts, it becomes the parent or grandparent of all of the processes that start up automatically on the system._  Init also automatically adopts all [orphaned processes](https://en.wikipedia.org/wiki/Orphan_process).

Note, that init is also capable of automatically restarting services when thay crash without requiring a reboot.

Init is a [daemon](daemon.md) process that continues running until the system is shut down (because it's responsible for not only starting the processes at boot, but also terminating them when system is shut down).

Init is absolutely critical to the OS. If for some reason it stops, kernel will panic and the system will halt. That is why init process is resistent to `kill -9` command which normally kills processes.

### History of Init

As Linux has evolved, so has the behavior of the init daemon. Originally, Linux started out with [System V](#system-v) init, the same that was used in UNIX. Since then, Linux has implemented many different init daemons. The most known which used by major Linux distributions include the [Upstart](#upstart) init daemon (created by Ubuntu) and the systemd init daemon (first implemented by Fedora).

Here is the history of adoption of different init systems by major Linux distrubutions:

* `System V` is the older init system:
    * Debian 6 and earlier
    * Ubuntu 9.04 and earlier
    * CentOS 5 and earlier
* `Upstart`:
    * Ubuntu 9.10 to Ubuntu 14.10, including Ubuntu 14.04
    * CentOS 6
* `systemd` is the init system for the most recent distributions featured here:
    * Debian 7 and newer
    * Ubuntu 15.04 and newer
    * CentOS 7

Each version of the init daemon has different ways of managing services. The reason behind these changes was the need for a robust service management tool that would handle not only services, but devices, ports, and other resources; that would start services more efficiently decreasing the boot time, and that would recover services after they crash.

As you can see, the major Linux distros are now using [systemd](systemd.md) as their init system. I'm going to briefly cover the [System V](#system-v) and [Upstart](#upstart) in this post and dedicate a [separate post](systemd.md) to systemd.

### Runlevel

A **runlevel** represents a state or working mode of the Linux OS. Each runlevel dictates what system services should be running and which should be stopped.

Runlevels are denoted by single digits and they can have a value between `0` and `6`. The following list shows what each of these levels mean:

* `Runlevel 0`: System shutdown
* `Runlevel 1`: Single-user (forbids non-root logins), rescue mode
* `Runlevels 2, 3, 4`: Multi-user, text mode (i.e. no GUI) with networking enabled
* `Runlevel 5`: Multi-user, network enabled, graphical mode
* `Runlevel 6`: System reboot

Runlevels `2`, `3`, and `4` vary by distribution. For example, some Linux distributions don't implement runlevel 4, while others do. Some distributions have a clear distinction between these three levels. In general, runlevel `2`, `3` or `4` means a state where Linux has booted in multi-user, network enabled, text mode.

The runlevel concept comes from [System V](#system-v) init, where the Linux system boots, loads the kernel, and then enters one (and only one) runlevel.

When we **enable** a service to **auto-start**, we are actually adding it to a runlevel. In System V, the OS will start with a particular runlevel; and, when it starts, it will try to start all the services that are associated with that runlevel.

Runlevels are replaced by `targets` in [systemd](systemd.md).

#### The reasoning behind runlevels

There are times in which you may want to operate the system in a lower mode. Examples are fixing disk corruption problems in run level 1 so no other users can possibly be on the system, or leaving a server in run level 3 without an X session running. In these cases, running services that depend upon a higher system mode to function does not make sense because they will not work correctly anyway. By already having each service assigned to start when its particular run level is reached, you ensure an orderly start up process, and you can quickly change the mode of the machine without worrying about which services to manually start or stop.

Or when a you decide to shut down the system, you probably don't want to stop each system service individually, but want your OS to take care of it.

### System V

**System V** or **Sys V** (pronounced as 'System Five') init daemon is started by the kernel using the **/sbin/init** script.

The first file **System V** init deamon reads after start is **/etc/inittab**.

The `/etc/inittab` file may look a bit complicated and we should say a few words about its syntax.

Line entries in `/etc/inittab` consist of four colon-delimited fields:

```
id:runlevels:action:process
```

* `id` is the unique sequence of characters used to identify an entry in this file
* `runlevels` specify runlevels for which this entry applies. The run levels are given as single digits, without delimiters.
* `action` is the action that should be taken. For example, `respawn` action is used to run the command in the next field again, when it exits, or `once` is used to run it just once.
* `process` specifies the command to run

Lines in `/etc/inittab` that start with `#` sign are treated as comments.

One of the entries in this file defines the **default runlevel** that the machine should boot into:

```bash
cat /etc/inittab | grep initdefault
#   0 - halt (Do NOT set initdefault to this)
#   6 - reboot (Do NOT set initdefault to this)
id:3:initdefault:
```

According to this file, the system will boot into runlevel `3` by default. The `/etc/inittab` also holds a description of what each runlevel means under this OS:

```bash
$ cat /etc/inittab | grep -A7 'The runlevels used by RHS'
# Default runlevel. The runlevels used by RHS are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode
#   4 - unused
#   5 - X11
#   6 - reboot (Do NOT set initdefault to this)
```

When System V init deamon starts, it first looks in `/etc/inittab` to determine a default runlevel it should boot the OS into. After it finds the default runlevel (which is typically the first entry in this `/etc/inittab`), it looks further into the `/etc/inittab` file and executes entries applicable for this runlevel. Let's look at the most important entries after default runlevel:

```bash
$ cat /etc/inittab | grep -A12 3:initdefault
id:3:initdefault:

# System initialization.
si::sysinit:/etc/rc.d/rc.sysinit

l0:0:wait:/etc/rc.d/rc 0
l1:1:wait:/etc/rc.d/rc 1
l2:2:wait:/etc/rc.d/rc 2
l3:3:wait:/etc/rc.d/rc 3
l4:4:wait:/etc/rc.d/rc 4
l5:5:wait:/etc/rc.d/rc 5
l6:6:wait:/etc/rc.d/rc 6
```

As you can see, the first script that System V init daemon calls is **/etc/rc.d/rc.sysinit** which applies to all runlevels since runlevels field is not explicitly specified for this entry.

This `/etc/rc.d/rc.sysinit` script performs a lot of functions for the system initialization that include: setting system hostname, checking and mountibg filesystems (/proc, /sys, and others in /etc/fstab), enabling swap, etc.

After this script has been executed, the runlevel specific scripts are run, according to `/etc/inittab`. And the most notable script is **/etc/rc.d/rc** which is used to start/stop system and application services corresponding to each runlevel. It takes the runlevel value as an argument, that's why there are seven different entries in `/etc/inittab` for each runlevel:

```
l0:0:wait:/etc/rc.d/rc 0
l1:1:wait:/etc/rc.d/rc 1
l2:2:wait:/etc/rc.d/rc 2
l3:3:wait:/etc/rc.d/rc 3
l4:4:wait:/etc/rc.d/rc 4
l5:5:wait:/etc/rc.d/rc 5
l6:6:wait:/etc/rc.d/rc 6
```

As we said, the `/etc/rc.d/rc` script is used to start/stop services according to the runlevel. But how does it know which services needs to be started/stopped at each runlevel?

Within the `/etc` directory, we have a number of **rc directories**, each with a number in its name. The numbers represent different runlevels. So we have `/etc/rc0.d`, `/etc/rc1.d`, `/etc/rc2.d` and so on. 

Each `/etc/rcN.d` directory contains scripts for starting/stopping different services:

```bash
$ ls -1 /etc/rc3.d/
K20nfs
K36mysqld
K50netconsole
K69rpcsvcgssd
K87multipathd
K89netplugd
K89rdisc
S07iscsid
S08ip6tables
S08iptables
S08mcstrans
S10network
S10vboxadd
S12restorecond
S12syslog
S13iscsi
S13portmap
S14nfslock
S18rpcidmapd
S19rpcgssd
S25netfs
S26lvm2-monitor
S30vboxadd-x11
S35vboxadd-service
S55sshd
S56rawdevices
S99local
```

The filenames start with either `K` or `S` letter in their, followed by two digits and the name of the service. `K` means Kill (i.e. stop) and `S` stands for Start.

The two digits represents the order of execution of the script. So if we have a file named `K25some_script`, it will be run before `K99another_script`.

When entering a particular runlevel, first, all `K` scripts within `/etc/rcN.d` directory are run in numerical order with an argument of `stop`, and then all `S` scripts are run in similar fashion with an argument of `start`.

So what are those of these files in `/etc/rcN.d` directories? These are scripts that allow to start and stop services.
Because a service can be started or stopped in multiple runlevels, these are files are actually symlinks to the files that reside in **/etc/init.d** directory (which could also be a symlink to `/etc/rc.d/init.d`):

```bash
$ ls -l /etc/rc3.d/
total 108
lrwxrwxrwx 1 root root 13 Jun 26  2017 K20nfs -> ../init.d/nfs
lrwxrwxrwx 1 root root 16 Aug  5 01:48 K36mysqld -> ../init.d/mysqld
lrwxrwxrwx 1 root root 20 Jun 26  2017 K50netconsole -> ../init.d/netconsole
lrwxrwxrwx 1 root root 20 Jun 26  2017 K69rpcsvcgssd -> ../init.d/rpcsvcgssd
lrwxrwxrwx 1 root root 20 Jun 26  2017 K87multipathd -> ../init.d/multipathd
lrwxrwxrwx 1 root root 18 Jun 26  2017 K89netplugd -> ../init.d/netplugd
...
```

The files in `/etc/init.d` directory are called **init scripts**.

The **init script** is what controls a specific service, like MySQL Server or SSH daemon, in System V. It contains the service management instructions, i.e. how to start, stop, restart, reload a service, etc.

In System V, an init script is a shell script. It typically takes arguments like `start`, `stop`, `status`, `restart` for service management operations. We can call this script directly:

```bash
$ /etc/init.d/mysqld status
mysqld (pid 3353) is running...
$ /etc/init.d/mysqld stop
Stopping mysqld:                                           [  OK  ]
$ /etc/init.d/mysqld status
mysqld is stopped
$ /etc/init.d/mysqld start
Starting mysqld:                                           [  OK  ]
```

And this is how System V init daemon starts and stop services in different runlevels, i.e. by calling init scripts in `/etc/rcN.d` directory with `start` or `stop` argument depending on the filename, i.e. whether it starts with `K` (e.g. `K36mysqld`) or `S` (e.g. `S55sshd`).

System users typically call init script via the **service command** line utility:

```bash
$ service mysqld status
mysqld (pid 3597) is running...
$ service mysqld stop
Stopping mysqld:                                           [  OK  ]
$ service mysqld status
mysqld is stopped
$ service mysqld start
Starting mysqld:                                           [  OK  ]
```

It's called the init script because it initializes the service. **Initialization** is responsible for ensuring the service can start and shut down cleanly. It kind of works similar to the init script used by the [kernel](kernel.md) to initialize the whole user space when the system boots up, but a service init script is limited in scope to initiliazing a single service.

Init scripts for services are either provided by the application's vendor or come with the Linux distribution (for native services). For example, when you install a mysql-server package, the init script for it will also be automatically installed. We can also create our own init scripts for custom created services like our applications.

#### System V Auto-Start

If we want some our services to automatically start at boot time, we need to add a symlink to the service's init script in the directory of our default runlevel.

In Red Hat based systems (like CentOS), the **chkconfig** tool was used ([systemd](systemd.md) has a different own mechanism) to control what services are started at which runlevels. 

Running the command `chkconfig –list` will display a list of services and whether they are enabled or disabled for each runlevel. For example, to check the status of a `mysql` server service for all runlevels on a Centos, we could use the following command:

```bash
$ chkconfig --list | grep mysqld
mysqld         	0:off	1:off	2:off	3:off	4:off	5:off	6:off
```

As you can see, `mysql` daemon is not started by init daemon in any of the runlevels. Let's also check scripts in `/etc/rcN.d` directory that corresponds to our default runlevel and make sure there is no filename for our service that starts with `S` letter:

```bash
$ ls -l /etc/rc3.d/*mysqld
lrwxrwxrwx 1 root root 16 Aug  5 06:02 /etc/rc3.d/K36mysqld -> ../init.d/mysqld
```

Yep, the service is stopped in the default runlevel. It means when I boot my system again, `mysql` server daemon wont' be running.

Let's enable the `mysql` service for runlevel 3 to ensure it's started the next time the system boots:

```bash
$ chkconfig --level 3 mysqld on
$ chkconfig --list mysqld
mysqld         	0:off	1:off	2:off	3:on	4:off	5:off	6:off
```

And we'll check the `/etc/rc3.d` again:

```bash
$ ls -l /etc/rc3.d/*mysqld
lrwxrwxrwx 1 root root 16 Aug  5 06:02 /etc/rc3.d/S64mysqld -> ../init.d/mysqld
```

As we can see, `chkconfig` command remove the old symlink file for mysql server daemon that started with `K` letter and created a new one that starts with `S`.

### Upstart

Classic Sys V init had been part of mainstream Linux distributions for a long time before **Upstart** came along.

#### What's wrong with the System V init?

System V initialization follows a linear process: individual tasks load in a predefined sequence as the system boots. The pros of using this implementation of init, is that it's relatively easy to solve dependencies, since you know service A starts before service B because, however the performance or linear booting process isn't great because usually one thing is starting or stopping at a time.

At the same time, as more and more modern devices like hot-pluggable storage media proliferated the market, System V init was found to be incapable of handling them quickly. For example, your system would have to be re-initialized before it could recognize a new storage drive connected. On-the-fly detection is what was needed, although it was not a capability of classic initialization procedure.

The developers at Ubuntu came up with another means of initialization, the Upstart daemon.

Upstart init is better than System V init in a few ways:

* Upstart does not deal with arcane shell scripts to load and manage services. Instead, it uses _configuration files_ which are easier to understand and modify
* Upstart does not load services in a linear sequence like System V and loads them in parallel instead. This cuts down on system boot time.
* Upstart has better ways of handling how a crashed service should respawn
* There is no need to keep a number of redundant symbolic links in `/etc/rcN.d` directories, all pointing to the same script
* Upstart is backwards-compatible with System V. It still runs the `/etc/init.d/rc` script at startup which executes any System V init scripts normally.

#### Upstart configuration files

Upstart does not use Bash scripts the way System V does to control services. Instead, Upstart uses service **configuration files** with a naming standard of `service_name.conf`.

Upstart looks for configuration files in **/etc/init** directory (don't confuse with **/etc/init.d** where System V stores keeps its init scripts!)

```bash
$ ls -l /etc/init
...
-rw-r--r-- 1 root root  297 Feb  9  2013 cron.conf
...
-rw-r--r-- 1 root root 1770 Feb 19  2014 mysql.conf
-rw-r--r-- 1 root root 2493 Mar 20  2014 networking.conf
...
-rw-r--r-- 1 root root 1543 Apr 11  2014 rc-sysinit.conf
-rw-r--r-- 1 root root  426 Apr 18  2013 rsyslog.conf
...
-rw-r--r-- 1 root root  641 May  2  2014 ssh.conf
```

Let's look at an example of a service configuration file.

```bash
$ cat /etc/init/cron.conf
# cron - regular background program processing daemon
#
# cron is a standard UNIX program that runs user-specified programs at
# periodic scheduled times

description	"regular background program processing daemon"

start on runlevel [2345]
stop on runlevel [!2345]

expect fork
respawn

exec cron
```

The important fields to be mindful of here are `start on`, `stop on` and `respawn`.

The `start on` directive tells Upstart to start the `crond` daemon when the system enters runlevels `2`, `3`, `4` or `5`. The service is not supposed to run on any other runlevel than these four  (exclamation mark (`!`) means exclusion here).

The `respawn` directive tells the Upstart daemon that cron service should start automatically if it crashes for any reason.

This line starts with `exec` indicated that the following command should run through the bash shell. So `exec cron` here indicates a bash command (i.e. `cron`) which should be used to start the service.

#### Commands used to manage services with Upstart

The same **service** command that was used to manage services under System V init can be used to control services under Upstart:

```bash
$ service mysql status
mysql start/running, process 3936
$ service mysql stop
mysql stop/waiting
$ service mysql status
mysql stop/waiting
$ service mysql start
mysql start/running, process 4350
```

But Upstart also provides its own command line utility called **initctl** (stands for "init control") for managing **jobs**. Job is a managed service in Upstart terminology:

```bash
$ initctl status mysql
mysql start/running, process 4350
$ initctl stop mysql
mysql stop/waiting
$ initctl status mysql
mysql stop/waiting
```

`initctl` provides more funcitonality than the `service` command. For example, it also allows to [emit events](https://www.digitalocean.com/community/tutorials/the-upstart-event-system-what-it-is-and-how-to-use-it#an-overview-of-upstart).

You may find unusual the reported status of a service `start/running` or `stop/waiting`. Why is there a slash and what does 'running' and 'waiting' mean?

Each of the Upstart jobs has a goal of either to `start` or `stop`. This are the two basic functions of service initialization. Between these two goals are a set of different states (a word after the slash `/`), which define the current actions of the job in regards to the goal. The important states are as follows:

* `waiting`: the initial state of processing
* `starting`: where a job is about to start
* `pre-start`: where the pre-start section is loaded
* `spawned`: where a script section is about to run
* `post-start`: where post-start operations take place
* `running`: where the job is fully operational
* `pre-stop`: where pre-stop operations take place
* `stopping`: where the job is being stopped
* `killed`: where the job is stopped
* `post-stop`: where post-stop operations take place - to clean up

#### Futher reading

[Here](https://www.digitalocean.com/community/tutorials/the-upstart-event-system-what-it-is-and-how-to-use-it) is a good tutorial from Digital Ocean for those who want to learn more about Upstart.

[Great presentation on systemd](https://fr.slideshare.net/enakai/systemd-study-v14e) which also covers SysV and Upstart.

### Resources used to create this document:

* https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-2-reference
* https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-1-practical-examples
* https://www.digitalocean.com/community/tutorials/the-upstart-event-system-what-it-is-and-how-to-use-it
* https://www.liquidweb.com/kb/linux-runlevels-explained/
* https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/installation_guide/s2-boot-init-shutdown-init
* https://en.wikipedia.org/wiki/Init
* https://www.tldp.org/LDP/sag/html/init-process.html
* https://www.tldp.org/LDP/sag/html/config-init.html
* https://www.networkworld.com/article/2693438/operating-systems/unix-how-to-the-linux-etc-inittab-file.html
* https://linuxjourney.com/lesson/sysv-overview
* https://www.tldp.org/LDP/intro-linux/html/sect_04_02.html
* https://linoxide.com/booting/boot-process-of-linux-in-detail/
