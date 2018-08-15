## Systemd

**Prerequisites:**
* [Linux boot process](boot-process.md)
* [Init system: System V & Upstart](init.md)

**Systemd** came as another replacement to the traditional [System V](init.md) init used by UNIX and then by Linux.

### Systemd main features

* _Reducing the system startup time._ Shell scripts (init scripts) execute commands in serial order, which is not efficient under the modern multicore processors. If various tasks in the script can be split and executed in parallel, it will reduce the system startup time.

* _Service dependencies._ With systemd, an explicit set of dependencies can be defined for each service, instead of being implied by boot order. This allows a service to start at any point that its dependencies are met. In this way, many services can start at the same time, making the boot process faster. Likewise, complex sets of dependencies can be set up, so the exact requirements of a service (such as storage availability or file system checking) can be met before a service starts.

* _Dynamic service activation._ Services don't just have to be always running or not running based on runlevel, as they were previous to systemd. Services can now be activated dynamically based on path, socket, bus, timer, or hardware activation. Likewise, because systemd can set up sockets, if a process handling communications terminates, the process that starts up in its place can pick up the next message from the socket. To the clients using the service, it can look as though the service continued without interruption.

* _Standard method for process termination._ Services are identified by [Cgroups](cgroups.md), which allow every component of a service to be managed. For example, the older System V init scripts would start a service by launching a process which itself might start other child processes. When the service was killed, it was hoped that the parent process would do the right thing and kill its children. By using Cgroups, all components of a service have a tag that can be used to make sure that all of those components are properly started or stopped.

* _Resource management._

    * The fact that each systemd unit is always associated with its own [cgroup](cgroup.md) lets you control the amount of resources each service can use. For example, you can set a percent of CPU usage by service which can put a cap on the total amount of CPU that service can use -- in other words, spinning off more processes won't allow more resources to be consumed by the service. Prior to systemd, nice levels were often used to prevent processes from hogging precious CPU time. With systemd's use of cgroups, precise limits can be set on CPU and memory usage, as well as other resources.
    * Also, a feature called _slices_ lets you slice up many different types of system resources and assign them to users,services, virtual machines, and other units. Accounting is also done on these resources, which can allow you to charge customers for their resource usage.

* _Backwards compatibility with SysV init_. Systemd supports SysV init scripts.

### Systemd Configuration Files: Unit Files

At the heart of systemd are **unit files** (or **units** for short). Units are the objects that systemd knows how to manage. They are basically a standardized representation of system resources.

In many ways units are similar to services or jobs in init systems. However, a unit has a much broader definition, as these can be used to describe not only services, but other system resources such as hardware devices, filesystem mounts, isolated resource pools (see slices), etc.

Systemd categories units according to the type of resource they describe. The following list describes the types of units available to systemd:

* `.service`: A service unit describes how to manage a service or application on the server. This will include how to start or stop the service, and when it should be automatically started.
* `.socket`: A socket unit file describes a [socket](sockets.md). These always have an associated .service file that will be started when activity is seen on the socket that this unit defines.
* `.device`: A unit that describes a device that has been designated as needing systemd management by udev or the sysfs filesystem. Not all devices will have `.device` files. Some scenarios where .device units may be necessary are for ordering, mounting, and accessing the devices.
* `.mount`: This unit defines a mountpoint on the system to be managed by systemd. These are named after the mount path, with slashes changed to dashes. Entries within /etc/fstab can have units created automatically.
* `.automount`: An .automount unit configures a mountpoint that will be automatically mounted. These must be named after the mount point they refer to and must have a matching .mount unit to define the specifics of the mount.
* `.swap`: This unit describes swap space on the system. The name of these units must reflect the device or file path of the space.
* `.target`: Target units are different from other unit files because they don't represent one particular resource. Rather, they represent the state of the system at any one time and act similar to runlevels in SysV init. Other units specify their relation to targets to become tied to the target’s operations.
* `.path`: This unit defines a path that can be used for path-based activation. By default, a `.service` unit of the same base name will be started when the path reaches the specified state. This uses inotify to monitor the path for changes.
* `.timer`: A `.timer` unit defines a timer that will be managed by systemd, similar to a cron job for delayed or scheduled activation. A matching unit will be started when the timer is reached.
* `.slice`: A `.slice` unit is associated with Linux [cgroups](cgroups.md), allowing resources to be restricted or assigned to any processes associated with the slice. The name reflects its hierarchical position within the cgroup tree. Units are placed in certain slices by default depending on their type.
* `.scope`: Scope units are created automatically by systemd from information received from its bus interfaces. These are used to manage sets of system processes that are created externally.

Each unit file represents a specific system resource and has a name that comply the following format:

```
resource_name.unit_type
```

So, we will have unit files like `crond.service`, `sshd.socket`, or `home.mount`.

### Where unit files are found?

The unit files can be found in many different locations. _Units in different locations have different priorities._

#### /lib/systemd/system

Note, that on some distributions `/lib` could be a symlink to `/usr/lib`.

The system’s copy of unit files are generally kept in the **/lib/systemd/system** directory. When software installs unit files on the system, this is the location where they are placed by default. For example, when I install an Apache package, the apache service unit file is places into `/lib/systemd/system`:

```bash
$ ls /lib/systemd/system | grep httpd
$ yum install -y httpd
....
$ ls /lib/systemd/system | grep httpd
httpd.service
```

Unit files stored `/lib/systemd/system` are generic, vanilla unit files, often written by the upstream project’s maintainers that should work on any system that deploys systemd in its standard implementation.

_You should not edit files in this directory._

#### /etc/systemd/system

If you wish to modify the way that a unit functions, the best location to do so is within the **/etc/systemd/system** directory.

_Unit files found in this directory location take precedence over any of the other locations on the filesystem._

If you need to modify the system’s copy of a unit file, putting a replacement in this directory is the safest and most flexible way to do this.

If you wish to override only specific directives from the system’s unit file, you can actually provide unit file snippets within a subdirectory. These will append or modify the directives of the system’s copy, allowing you to specify only the options you want to change. The correct way to do this is to create a directory named after the unit file with `.d` appended on the end. So if you wanted to tweak a packaged unit `/lib/systemd/system/example.service` without having to replace it entirely, you would creat a subdirectory called `/etc/systemd/system/example.service.d` and within this directory you would create a file ending in `.conf` (e.g. `/etc/systemd/system/example.service.d/foo.conf`) to override or extend the attributes of the system’s unit file. Another example is described [here](https://www.freedesktop.org/software/systemd/man/systemd.unit.html).

#### /run/systemd/system

There is also a location for run-time unit definitions at **/run/systemd/system**. Unit files found in this directory have a priority lower than units in `/etc/systemd/system`, but higher than units in `/lib/systemd/system`. 

The systemd process itself uses this location for dynamically created unit files created at runtime. This directory can be used to change the system’s unit behavior for the duration of the session. All changes made in this directory will be lost when the server is rebooted.

### Anatomy of a unit file

Unit files are simple text files (like Upstart `.conf` files) with a declarative syntax. Each unit is described by a configuration in [ini-style format](https://en.wikipedia.org/wiki/INI_file).

The structure of unit files are organized with **sections**. Sections are denoted by a pair of square brackets `[]` with the section name enclosed within. Each section extends until the beginning of the subsequent section or until the end of the file. Section names are case-sensitive. So, the section `[Unit]` will not be interpreted correctly if it is spelled like `[UNIT]`.

Within the sections, unit configuration is defined through the use of directives in _key-value_ format:

```
[Section]
Directive1=value
Directive2=value
```

In the event of an override file (such as those contained in a `unit.type.d` directory), directives can be reset by assigning them to an empty string. For example, the system’s copy of a unit file may contain a directive set to a value like this:

```
Directive1=default_value
```

The "default_value" can be eliminated in an override file by referencing Directive1 without a value, like this:

```
Directive1=
```

Next, we'll look at some commonly used sections and their directives.

#### Generic sections

`[Unit]` and `[Install]` sections are generic sections which are common among all unit types. Full documentation on them can be found [here](https://www.freedesktop.org/software/systemd/man/systemd.unit.html).

##### [Unit] Section

Although _section order does not matter to systemd when parsing the file_, this section is often placed at the top because it provides an overview of the unit.

The **[Unit] section** is used for defining unit's description and its relationship with regards to other units. Some common directives that you will find in the [Unit] section are:

* `Description=`: This directive can be used to describe the name and basic functionality of the unit. It is returned by various systemd tools, so it is good to set this to something short, specific, and informative. This text is displayed for example in the output of the [systemctl status](systemctl.md) command.
* `Documentation=`: This directive provides a location for a list of URIs for documentation. These can be either internally available man pages or web accessible URLs. The systemctl status command will expose this information, allowing for easy discoverability. This is also displayed by [systemctl status](systemctl.md) command.
* `Requires=`: Configures dependencies on other units. The units listed in `Requires` are activated together (i.e. in parallel) with this unit. If any of the required units fail to start, the unit is not activated.
* `Wants=`: This directive is similar to `Requires=`, but less strict. Systemd will attempt to start any units listed here when this unit is activated. If these units are not found or fail to start, the current unit will continue to function. This is the recommended way to configure most dependency relationships. Again, this implies a parallel activation unless modified by other directives.
* `BindsTo=`: This directive is similar to `Requires=`, but also causes the current unit to stop when the associated unit terminates.
* `Before=`: The units listed in this directive will not be started until the current unit is marked as started if they are activated at the same time. This does not imply a dependency relationship and must be used in conjunction with one of the above directives if this is desired.
* `After=`: The units listed in this directive will be started before starting the current unit. This does not imply a dependency relationship and one must be established through the above directives if this is required.
* `Conflicts=`: This can be used to list units that cannot be run at the same time as the current unit. Starting a unit with this relationship will cause the other units to be stopped.
* `Condition...=`: There are a number of directives that start with `Condition` which allow the administrator to test certain conditions prior to starting the unit. This can be used to provide a generic unit file that will only be run when on appropriate systems. If the condition is not met, the unit is gracefully skipped.
* `Assert...=`: Similar to the directives that start with `Condition`, these directives check for different aspects of the running environment to decide whether the unit should activate. However, unlike the `Condition` directives, a negative result causes a failure with this directive.

Here is an example of SSH daemon service unit [Unit] section:

```bash
$ systemctl cat sshd
# /usr/lib/systemd/system/sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service
...
```

##### [Install] section

On the opposite side of unit file, the last section is often the **[Install] section**. It encodes information about how the suggested installation should look like, i.e. under which circumstances and by which triggers the service shall be started.

This section is not interpreted by systemd during runtime; it is used by the `enable` and `disable` commands of the [systemctl](systemctl.md) tool during installation of a unit.

Some common directives that you will find in the [Install] section are:

* `WantedBy=`: This directive is the most common way to specify how a unit should be **enabled**. Enabling a unit means configuring it to autostart under a specified [runlevel or target](#targets-runlevel-replacement). This directive allows you to specify a dependency relationship in a similar way to the `Wants=` directive does in the [Unit] section. The primary result is that the current unit will be started when the listed unit is started. When a unit with this directive is enabled, a directory will be created within `/etc/systemd/system` named after the specified unit with `.wants` appended to the end. Within this directory, a symbolic link to the current unit will be created, creating the dependency. For instance, if the current unit has `WantedBy=multi-user.target`, a directory called `multi-user.target.wants` will be created within `/etc/systemd/system` (if not already available) and a symbolic link to the current unit will be placed within. Disabling this unit removes the link and removes the dependency relationship.
* `RequiredBy=`: This directive is very similar to the `WantedBy=` directive, but instead specifies a required dependency that will cause the activation to fail if not met. When enabled, a unit with this directive will create a directory ending with `.requires`.
* `Alias=`: This directive allows the unit to be enabled under another name as well. A space-separated list of additional names this unit shall be installed under. The names listed here must have the same suffix (i.e. type) as the unit filename. At installation time, [systemctl enable](systemctl.md) will create symlinks from these names to the unit filename. Note that not all unit types support such alias names, and this setting is not supported for them. Specifically, mount, slice, swap, and automount units do not support aliasing.
* `Also=`: This directive allows units to be enabled or disabled as a set. If the user requests installation/deinstallation of a unit with this option configured, [systemctl enable](systemctl.md) and [systemctl disable](systemctl.md) will automatically install/uninstall units listed in this option as well.

Here is an example of SSH daemon service unit [Install] section:

```bash
$ systemctl cat sshd
...
[Install]
WantedBy=multi-user.target
```

This install section simply says that this service shall be started when the `multi-user.target` unit is activated. `multi-user.target` is a special unit that basically takes the role of the classic [SysV runlevel 3](init.md).

And because this ssh daemon is enabled, a symlink exists in the `/etc/systemd/system/multi-user.target.wants` directory to this unit file:

```bash
$ systemctl is-enabled sshd
enabled
$ ls -l /etc/systemd/system/multi-user.target.wants/sshd.service
lrwxrwxrwx. 1 root root 36 May 12 18:52 /etc/systemd/system/multi-user.target.wants/sshd.service -> /usr/lib/systemd/system/sshd.service
$ systemctl disable sshd
Removed symlink /etc/systemd/system/multi-user.target.wants/sshd.service.
$ ls -l /etc/systemd/system/multi-user.target.wants/sshd.service
ls: cannot access /etc/systemd/system/multi-user.target.wants/sshd.service: No such file or directory
```

#### Unit-Specific sections

Apart from the generic sections common across all units, there are sections specific to each unit type. In this post, I'm going to cover only the [Service] section which is specific to the service units. You read about other unit specific sections [here](https://n0where.net/understanding-systemd).

##### [Service] section

The **[Service] section** is used to provide configuration that is only applicable for service units. A service unit, i.e. configuration file whose name ends in `.service`, encodes information about a process controlled and supervised by systemd. The full documentation on the section can be found [here](https://www.freedesktop.org/software/systemd/man/systemd.service.html).

One of the basic things that should be specified within the `[Service]` section is the `Type=` of the service. This categorizes services by their process and daemonizing behavior. This is important because it tells systemd how to correctly manage the servie and find out its state.

The `Type=` directive can be one of the following:

* `simple:` This is the default type if neither `Type=` nor `BusName=`, but `ExecStart=` are specified. It is expected that the process configured with `ExecStart=` is the main process of the service. In this mode, if the process offers functionality to other processes on the system, its communication channels should be installed before the daemon is started up (e.g. sockets set up by systemd, via socket activation), as systemd will immediately proceed starting follow-up units.
* `forking`: If set to `forking`, it is expected that the process configured with `ExecStart=` will call [fork()](fork.md) as part of its start-up. The parent process is expected to exit when start-up is complete and all communication channels are set up. The child continues to run as the main daemon process. This is the behavior of traditional UNIX daemons. If this setting is used, it is recommended to also use the `PIDFile=` option, so that systemd can identify the main process of the daemon. systemd will proceed with starting follow-up units as soon as the parent process exits.
* `oneshot`: Behavior of `oneshot` is similar to `simple`; however, it is expected that the process has to exit before systemd starts follow-up units. `RemainAfterExit=` is particularly useful for this type of service. This is the implied default if neither `Type=` nor `ExecStart=` are specified.
* `dbus`: Behavior of `dbus` is similar to `simple`; however, it is expected that the daemon acquires a name on the [D-Bus](dbus.md) bus, as configured by BusName=. systemd will proceed with starting follow-up units after the D-Bus bus name has been acquired. Service units with this option configured implicitly gain dependencies on the dbus.socket unit. This type is the default if BusName= is specified.
* `notify`: Behavior of `notify` is similar to `simple`; however, it is expected that the daemon sends a notification message via `sd_notify` or an equivalent call when it has finished starting up. systemd will proceed with starting follow-up units after this notification message has been sent. If this option is used, NotifyAccess= (see below) should be set to open access to the notification socket provided by systemd. If NotifyAccess= is missing or set to none, it will be forcibly set to main.
* `idle`: Behavior of `idle` is very similar to `simple`; however, actual execution of the service program is delayed until all active jobs are dispatched. This may be used to avoid interleaving of output of shell services with the status output on the console. Note that this type is useful only to improve console output, it is not useful as a general unit ordering tool, and the effect of this service type is subject to a 5s time-out, after which the service program is invoked anyway.

Some additional directives may be needed when using certain service types. For instance:

* `RemainAfterExit=`: Takes a boolean value that specifies whether the service shall be considered active even when all its processes exited. Defaults to no. This directive is commonly used with the oneshot type.
* `PIDFile=`: Usage of this option is recommended for services where `Type=` is set to `forking`. This directive is used to set the absolute path to the file that should contain the PID number of the main child process that should be monitored by systemd. The systemd will not write to the file configured here, although it will remove the file after the service has shut down if it still exists. The PID file does not need to be owned by a privileged user, but if it is owned by an unprivileged user additional safety restrictions are enforced: the file may not be a symlink to a file owned by a different user (neither directly nor indirectly), and the PID file must refer to a process already belonging to the service.
* `BusName=`: This directive should be set to the D-Bus bus name that the service will attempt to acquire when using the `dbus` service type.
* [NotifyAccess=](https://www.freedesktop.org/software/systemd/man/systemd.service.html): This specifies access to the socket that should be used to listen for notifications when the `notify` service type is selected.

So far, we have discussed some pre-requisites for running a service, but we haven’t actually defined how to start our services. The following directived are used to manage the service:

* `ExecStart=`: This specifies the _full path_ and the arguments of the command to be executed to start the process. This may only be specified once (except for `oneshot` services). Unless `Type=forking` is set, the process started via this command will be considered the main process of the daemon. If the executable path is prefixed with `-`, an exit code of the command normally considered a failure (i.e. non-zero exit status or abnormal exit due to signal) is ignored and considered success.
* `ExecStartPre=`, `ExecStartPost=`: Additional commands that are executed before or after the command in `ExecStart=`, respectively. Syntax is the same as for ExecStart=, except that multiple command lines are allowed and the commands are executed one after the other, serially. If any of those commands (not prefixed with "-") fail, the rest are not executed and the unit is considered failed. Note that `ExecStartPre=` may not be used to start long-running processes. All processes forked off by processes invoked via `ExecStartPre=` will be killed before the next service process is run.
Note that if any of the commands specified in `ExecStartPre=`, `ExecStart=`, or `ExecStartPost=` fail (and are not prefixed with "-", see above) or time out before the service is fully up, execution continues with commands specified in `ExecStopPost=`, the commands in `ExecStop=` are skipped.
* [ExecReload=](https://www.freedesktop.org/software/systemd/man/systemd.service.html): This optional directive indicates the command necessary to reload the configuration of the service if available.
* [ExecStop=](https://www.freedesktop.org/software/systemd/man/systemd.service.html): This indicates the command needed to stop the service started via `ExecStart=`. Note that it is usually not sufficient to specify a command for this setting that only asks the service to terminate (for example, by queuing some form of termination signal for it), but does not wait for it to do so. Since the remaining processes of the services are killed according to KillMode= and KillSignal= as described above immediately after the command exited, this may not result in a clean stop. The specified command should hence be a synchronous operation, not an asynchronous one.
* [ExecStopPost=](https://www.freedesktop.org/software/systemd/man/systemd.service.html): Additional commands that are executed after the service is stopped. This includes cases where the commands configured in `ExecStop=` were used, where the service does not have any `ExecStop=` defined, or where the service exited unexpectedly. It is recommended to use this setting for clean-up operations that shall be executed even when the service failed to start up correctly. Commands configured with this setting need to be able to operate even if the service failed starting up half-way and left incompletely initialized data around. As the service's processes have been terminated already when the commands specified with this setting are executed they should not attempt to communicate with them.
* [TimeoutSec=](https://www.freedesktop.org/software/systemd/man/systemd.service.html): This configures the amount of time that systemd will wait when stopping or stopping the service before marking it as failed or forcefully killing it. You can set separate timeouts with TimeoutStartSec= and TimeoutStopSec= as well.
* [Restart=](https://www.freedesktop.org/software/systemd/man/systemd.service.html): Configures whether the service shall be restarted when the service process exits, is killed, or a timeout is reached. The service process may be the main service process, but it may also be one of the processes specified with `ExecStartPre=`, `ExecStartPost=`, `ExecStop=`, `ExecStopPost=`, or `ExecReload=`. When the death of the process is a result of systemd operation (e.g. service stop or restart), the service will not be restarted. Takes one of "no", "on-success", "on-failure", "on-abnormal", "on-watchdog", "on-abort", or "always". If set to "no" (the default), the service will not be restarted. If set to "on-success", it will be restarted only when the service process exits cleanly. 
* `RestartSec=`: If automatically restarting the service is enabled, this specifies the amount of time to wait before attempting to restart the service.

**NOTE:** Systemd managed services do not inherit any context (such as the `HOME` and `PATH` environment variables) from the invoking user and their session. Each service runs in a clean execution context. That is why in the `ExecStart` directive it's required to specify the absolute path to the executable. Take a look at the [execution environment configuration](https://www.freedesktop.org/software/systemd/man/systemd.exec.html) for environment configuration.

Also worth taking a look are [killing options](https://www.freedesktop.org/software/systemd/man/systemd.kill.html) for services.

### Targets - runlevel replacement

A special type of unit file is a **target unit**.

A target unit filename has `.target` suffix. Target units are different from other unit files because they don't represent one particular resource. Rather, they represent the state of the system at any one time.

Target units do this by grouping and launching multiple unit files that should be part of that state. systemd targets can therefore be loosely compared to System V [runlevels](init.md), although **they are not the same**.

#### Comparison with SysV runlevels

Unlike runlevels which are identified by number, dach target is identified by name. For example, we have `multi-user.target` instead of `runlevel 3` or `reboot.target` instead of `runlevel 6`.

When a Linux server boots with say, `multi-user.target`, it's essentially bringing the server to [runlevel 2, 3, or 4](init.md), which is the multi-user text mode with networking enabled.

The table below shows targets which could be compared to SysV runlevels:

Runlevel (System V init) |  Target Units (Systemd)
------------------------ | -----------------------
runlevel 0               |  poweroff.target
runlevel 1               |  resuce.target
runlevel 2, 3, 4         |  multi-user.target
runlevel 5               |  graphical.target
runlevel 6               |  reboot.target

Another difference between target units and runlevels is that in System V, a Linux system could exist in only one runlevel. You could change the runlevel, but the system would exist in that new runlevel only. With systemd, target units can be inclusive, which means when a target unit activates, it can ensure other target units are loaded as part of it.

For example, a Linux system that boots with a graphical user interface will have the `graphical.target` activated, which in turn will automatically ensure `multi-user.target` is loaded and activated as well (note the `Requires` directive):

```bash
$ cat /lib/systemd/system/graphical.target
[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
```

If in traditional init system there were 7 runlevels, you will find much more targets in the systemd:

```bash
$ systemctl list-unit-files --type=target
UNIT FILE                 STATE
basic.target              static
bluetooth.target          static
cryptsetup-pre.target     static
cryptsetup.target         static
...

$ systemctl list-unit-files --type=target | wc -l
64
```

So we can say that systemd has only limited support for runlevels. It provides a number of target units that can be directly mapped to these runlevels and for compatibility reasons, but they are not the same.

Systemd targets are more granular than runlevels and allow to activate smaller parts of the operating system. For instance, there is the `network.target` unit for activating the network on the system. This target acts as a syncronization point for system network services, i.e. when this network target gets activated all of the services that are required by this target unit are started:

```bash
$ fgrep -r 'Before=network.target' /lib/systemd/system/*.service
/lib/systemd/system/arp-ethers.service:Before=network.target
/lib/systemd/system/NetworkManager.service:Before=network.target network.service
/lib/systemd/system/wpa_supplicant.service:Before=network.target
```

#### Default target

In System V, we had the default runlevel defined in a file called [inittab](init.md). In systemd, that file is replaced by **default.target**. The default target unit file lives under `/etc/systemd/system` directory. It's a symbolic link to one of the target unit files under `/lib/systemd/system`:

```bash
$ ls -l /etc/systemd/system/default.target
lrwxrwxrwx. 1 root root 37 May 12 18:54 /etc/systemd/system/default.target -> /lib/systemd/system/multi-user.target
```

You can also see the default target with [systemctl](systemctl.md) command:

```bash
$ systemctl get-default
multi-user.target
```

When we change the default target, we are essentially recreating that symbolic link and changing the system's runlevel.

```bash
$ systemctl set-default multi-user.target
Removed symlink /etc/systemd/system/default.target.
Created symlink from /etc/systemd/system/default.target to /usr/lib/systemd/system/multi-user.target.
```

### Service Auto-Start

In systemd, a unit that _requires_ or _wants_ another unit will not start until the required unit is activated first.
This is how dependecies are configured between units.

Most of the times a unit is [WantedBy](#[Install]-section) or [RequiredBy](#[Install]-section) some target unit:

```bash
$ systemctl cat sshd | grep -A2 Install
[Install]
WantedBy=multi-user.target
```

When a unit with this directive is **enabled**, a directory will be created within `/etc/systemd/system` named after the specified unit with `.wants` appended to the end. Within this directory, a symbolic link to the current unit will be created, creating the dependency:

```bash
$ systemctl enable sshd
Created symlink from /etc/systemd/system/multi-user.target.wants/sshd.service to /usr/lib/systemd/system/sshd.service.
$ ls -l /etc/systemd/system/multi-user.target.wants | grep sshd
lrwxrwxrwx. 1 root root 36 Aug 15 03:40 sshd.service -> /usr/lib/systemd/system/sshd.service
```

As the result this ssh service unit will be started when the `multi-user.target` unit is started. If the unit specified in `WantedBy` (or `RequiredBy`) directive is the [default target unit](#default-target), then this unit will be activated on system boot, i.e. it will be automatically started.

Disabling this unit removes the link and removes the dependency relationship:

```bash
$ systemctl disable sshd
Removed symlink /etc/systemd/system/multi-user.target.wants/sshd.service.
```

### Systemd is not just an init system

Systemd is more than just init system. Systemd refers to the whole software bundle around it, which, in addition to the `systemd init daemon`, includes the daemons [journald](journald.md), `logind` and `networkd`, and many other low-level components. That's why systemd is also often called **system and service manager**, as it allows to manage not only services but many other important part of the operating system.

### Systemd is controversial

Systemd is controversial for various reasons: it eschews some established Unix conventions, such as plain text log files. It’s seen as a "monolithic" project trying to take over everything else. And it’s a major change to the underpinnings of Linux OS. Yet almost every major distribution has adopted it (or is about to), so it’s here to stay. And there are benefits: faster booting, easier management of services that depend on one another, and powerful and secure logging facilities too.

### Further reading

* [Unit and system management with systemctl command](systemctl.md)
* [Log management with journald](journald.md)
* [Your first dive into systemd!](https://fr.slideshare.net/enakai/systemd-study-v14e)
* [Systemd course](http://www.enricozini.org/blog/2017/debian/systemd-01-intro/)

### Resources used to create this document

* https://www.certdepot.net/rhel7-get-started-systemd/
* https://fr.slideshare.net/enakai/systemd-study-v14e
* http://pminkov.github.io/blog/the-linux-init-system.html
* https://n0where.net/understanding-systemd
* https://access.redhat.com/articles/754933
* https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd#sect-Managing_Services_with_systemd-Introduction
* http://www.enricozini.org/blog/2017/debian/systemd-01-intro/
* https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-2-reference
* https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-1-practical-examples
* https://www.digitalocean.com/community/tutorials/systemd-essentials-working-with-services-units-and-the-journal
* https://www.freedesktop.org/software/systemd/man/systemd.unit.html
* http://0pointer.de/blog/projects/systemd-for-admins-3.html
