## systemctl

**systemctl** is the command line utility used to manage [systemd units](systemd.md).

Unlike [service](init.md) command which is used to manage services init scripts under [System V](init.md), `systemctl` utility does not support custom commands. In addition to standard commands such as `start`, `stop`, and `status`, authors of SysV init scripts could implement support for any number of arbitrary commands in order to provide additional functionality. For example, the init script for iptables in Red Hat Enterprise Linux 6 could be executed with the `panic` command, which immediately enabled panic mode and reconfigured the system to start dropping all incoming and outgoing packets. This is not supported in systemd and the `systemctl` only accepts documented commands.

### Unit management with systemd

As systemctl is used to manage systemd units, you need to pass unit names to systemctl commands as arguments. Systemd unit file names have the following format:

```
resource_name.unit_type
```

So, you have unit files like `crond.service`, `sshd.socket`, or `home.mount`, etc.

If you omit the unit type suffix when passing the unit name to `systemctl`, it will append a suitable suffix for you. By default, it will append `.service` suffix as service units are the most numerous. So these two command will work the same:

```bash
$ systemctl is-active crond.service
active
$ systemctl is-active crond
active
```

`systemctl` will also append a type-specific suffix in case of commands which operate only on specific unit types. For example, **set-default** command used to set default target unit (or default runlevel) only works on target units, thus we can omit specifying the `.target` suffix in the unit name and systemctl will add it for us. So these two commands are equivalent:

```bash
$ systemctl set-default multi-user
$ systemctl set-default multi-user.target
```

**NOTE 1:** `systemctl` also supports [globbing](globbing.md) which mean you can use [wildcards](globbing.md) in the unit names. For example, this may come in handy when you don't know the exact unit name:

```bash
$ systemctl is-active ssh
unknown
$ systemctl is-active ssh*
active
```

**NOTE 2:** the `status` command can take PIDs instead of unit names. This also may be useful when you're able to find a process ID, but not sure which unit represents this process:

```bash
$ systemctl status 2776
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2018-08-17 05:57:51 UTC; 34min ago
 Main PID: 2776 (nginx)
...
```

### Useful commands

#### Start, Stop, Reload and Restart a service

```bash
$ systemctl start nginx
$ systemctl restart nginx
$ systemctl stop nginx
```

If the application in question is able to reload its configuration files (without restarting), you can issue the reload command to initiate that process:

```bash
$ systemctl reload nginx
```

If you are unsure whether the service has the functionality to reload its configuration, you can issue the reload-or-restart command. This will reload the configuration in-place if available. Otherwise, it will restart the service so the new configuration is picked up:

```bash
$ systemctl reload-or-restart nginx
```

A few more restart/reload commands:
* `try-restart`: restarts a unit only if it is already active, otherwise do nothing, to prevent accidentally starting a service
* `try-reload-or-restart`: tell a unit to reload its configuration if supported, otherwise restart it. If the unit is not already active, do nothing to prevent accidentally starting a service.

#### Check the service status

```bash
$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2018-08-17 05:57:51 UTC; 1min 30s ago
  Process: 2785 ExecReload=/bin/kill -s HUP $MAINPID (code=exited, status=0/SUCCESS)
  Process: 2773 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 2770 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 2769 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 2776 (nginx)
   CGroup: /system.slice/nginx.service
           ├─2776 nginx: master process /usr/sbin/nginx
           └─2786 nginx: worker process

Aug 17 05:57:51 localhost.localdomain systemd[1]: Starting The nginx HTTP and reverse proxy server...
Aug 17 05:57:51 localhost.localdomain nginx[2770]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Aug 17 05:57:51 localhost.localdomain nginx[2770]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Aug 17 05:57:51 localhost.localdomain systemd[1]: Failed to read PID from file /run/nginx.pid: Invalid argument
Aug 17 05:57:51 localhost.localdomain systemd[1]: Started The nginx HTTP and reverse proxy server.
Aug 17 05:57:59 localhost.localdomain systemd[1]: Reloaded The nginx HTTP and reverse proxy server.
```

Let's break down the output and explain it a little.

The first line shows the description of the service unit contained in the unit file:

```
● nginx.service - The nginx HTTP and reverse proxy server
```
The dot ("●") uses color on supported terminals to summarize the unit state at a glance. White indicates an "inactive" or "deactivating" state. Red indicates a "failed" or "error" state and green indicates an "active", "reloading" or "activating" state.

The `Loaded:` line in the output will show `loaded` if the unit has been loaded into memory. Other possible values for `Loaded:` include: `error` if there was a problem loading it, `not-found` if not unit file was found for this unit, `bad-setting` if an essential unit file setting could not be parsed and `masked` if the unit file has been masked. It will also show where the unit file is located in the filesystem and whether the service is `enabled/disabled`, i.e. if it's configured to auto start when system boots:

```
Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
```

The `vendor preset` here indicates whether this service will be enabled/disabled by default when its package is initially installed on the system.

The third line shows the current status of the service, i.e. whether it's `active (running)` or `inactive (dead)`. If the service is active, it will also show the date when the service has been started and how long it has been running:

```
Active: active (running) since Fri 2018-08-17 05:57:51 UTC; 1min 30s ago
```

The output will include the Main PID and [cgroup](cgroup.md) information. Lastly, it will show a few last log lines of this service daemon.

#### Enable services to auto-start

**Enabling** a unit marks it to be automatically started at boot. In essence, this is accomplished by latching the unit in question onto another unit that is somewhere in the line of units to be started at boot.

Enabling behavior of a unit is determined by the [[Install] section](systemd.md) in the unit file.

Most of the times a unit is [WantedBy](#[Install]-section) or [RequiredBy](#[Install]-section) some target unit:

```bash
$ systemctl cat sshd | grep -A2 Install
[Install]
WantedBy=multi-user.target
```

When a unit with this directive is enabled, a directory will be created within `/etc/systemd/system` named after the specified unit with `.wants` appended to the end. Within this directory, a symbolic link to the current unit will be created:

```bash
$ systemctl enable sshd
Created symlink from /etc/systemd/system/multi-user.target.wants/sshd.service to /usr/lib/systemd/system/sshd.service.
$ ls -l /etc/systemd/system/multi-user.target.wants | grep sshd
lrwxrwxrwx. 1 root root 36 Aug 15 03:40 sshd.service -> /usr/lib/systemd/system/sshd.service
```

**Disabling** a unit will in turn remove that symlink:

```bash
$ systemctl disable sshd
Removed symlink /etc/systemd/system/multi-user.target.wants/sshd.service.
```

To check if unit is enable, use the [status command](#check-the-service-status) or the following command:

```bash
$ systemctl is-enabled sshd
enabled
```

The **reenable** command is a combination of disable and enable and is useful to reset the symlinks a unit is enabled with to the defaults configured in the "[Install]" section of the unit file. This could be useful when you changed the "[Install]" section:

```bash
$ systemctl reenable sshd
Removed symlink /etc/systemd/system/multi-user.target.wants/sshd.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/sshd.service to /usr/lib/systemd/system/sshd.service.
```

**NOTE:** _Enabling units should not be confused with starting (activating) units, as done by the start command._

Enabling and starting units is orthogonal: units may be enabled without being started and started without
being enabled. _Enabling_ simply hooks the unit into various suggested places (for example, so that the unit
is automatically started on boot or when a particular kind of hardware is plugged in). _Starting_ actually
spawns the daemon process (in case of service units), or binds the socket (in case of socket units), and
so on.

#### Display a unit file

To display the unit file that systemd has loaded into its system, you can use the `systemctl cat` command:

```bash
$ systemctl cat sshd
# /usr/lib/systemd/system/sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

Note that the first line of the output, i.e. `/usr/lib/systemd/system/sshd.service`, shows the location of the unit file on the filesystem.

Another useful command for displaying unit configuration is `systemctl show`. This will display a list of unit properties in a "key=value" format:

```bash
$ systemctl show sshd
Type=notify
Restart=on-failure
NotifyAccess=main
RestartUSec=42s
TimeoutStartUSec=1min 30s
TimeoutStopUSec=1min 30s
WatchdogUSec=0
WatchdogTimestamp=Sat 2018-08-18 03:57:57 UTC
...
```

It also allows you to display a single property. You just need to pass the `-p` flag with the property name:

```bash
$ systemctl show sshd -p Conflicts
Conflicts=shutdown.target
$ systemctl show sshd -p WantedBy
WantedBy=multi-user.target
```

#### Editing unit files

`systemctl` allows you to edit unit files directly from the application, without needing to know the exact location of the file on the disk.

The **edit** command, by default, will open a unit file snippet for the unit in question:

```bash
$ systemctl edit httpd
```

This will be a blank file that can be used to [override or add directives](systemd.md) to the unit definition. A directory will be created within the `/etc/systemd/system` directory which contains the name of the unit with `.d` appended. For instance, for the `httpd.service`, a directory `/etc/systemd/system/httpd.service.d` will be created.

Within this directory, a snippet will be created called `override.conf`. When the unit is loaded, systemd will, in memory, merge the override snippet with the full unit file. The snippet's directives will take precedence over those found in the original unit file.

If you wish to edit the full unit file instead of creating a snippet, you can pass the **--full** flag:

This will load the current unit file into the editor, where it can be modified. When the editor exits, the changed file will be written to `/etc/systemd/system`, which will take precedence over the system's unit definition (usually found somewhere in `/lib/systemd/system`).

To remove any additions you have made, either delete the unit's `.d` configuration directory or the modified service file from `/etc/systemd/system`.

```bash
# to remove snippet
$ sudo rm -r /etc/systemd/system/nginx.service.d
# to remove a fully modified file
$ sudo rm /etc/systemd/system/nginx.service
```

After creating/deleting the file or directory, you should reload the systemd process for systemd to start using changes. You can do this by typing:

```bash
$ sudo systemctl daemon-reload
```

#### Reload systemd

After you make changes to your unit files, you need to **reload** systemd daemon.

```bash
$ systemctl daemon-reload
```

Note that `daemon-reload` won't reload/restart the services themselves, it just makes systemd aware of the new configuration.

#### List units

To list all available unit files on the system and their enablement state (as reported by is-enabled), use `list-unit-files` command:

```bash
$ systemctl list-unit-files ssh*
UNIT FILE           STATE
sshd-keygen.service static
sshd.service        enabled
sshd@.service       static
sshd.socket         disabled

4 unit files listed.
```

To get a list of only loaded units, run the `list-units` command:

```bash
$ systemctl list-units nginx*
UNIT          LOAD   ACTIVE SUB     DESCRIPTION
nginx.service loaded active running The nginx HTTP and reverse proxy server
...
```

"Loaded" status indicated that the unit's configuration has been parsed by systemd. The configuration of loaded units is kept in memory.

In the example below, no unit whose name starts which `http` is loaded, although I do have a service unit file for httpd daemon. But because it's `disabled` it's not loaded automatically on system boot:

```bash
$ systemctl list-units http*
0 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.

$ systemctl list-unit-files http*
UNIT FILE     STATE
httpd.service disabled

1 unit files listed.
```

To see a list of units that were failed to load use the following command:

```bash
$ systemctl --failed
 UNIT                                                  LOAD   ACTIVE SUB    DESCRIPTION
● selinux-policy-migrate-local-changes@targeted.service loaded failed failed Migrate local SELinux policy changes from the old store struc
```

**NOTE** if you want to list units of only particular type, use the `--type` option:

```bash
$ systemctl list-units --type=target
UNIT                  LOAD   ACTIVE SUB    DESCRIPTION
basic.target          loaded active active Basic System
cryptsetup.target     loaded active active Local Encrypted Volumes
getty.target          loaded active active Login Prompts
local-fs-pre.target   loaded active active Local File Systems (Pre)
local-fs.target       loaded active active Local File Systems
multi-user.target     loaded active active Multi-User System
network-online.target loaded active active Network is Online
network.target        loaded active active Network
...

$ systemctl list-unit-files --type=target
UNIT FILE                 STATE
basic.target              static
bluetooth.target          static
cryptsetup-pre.target     static
cryptsetup.target         static
ctrl-alt-del.target       disabled
```

#### Display unit dependencies

To see a unit's dependency tree, you can use the `list-dependencies` command.

This will display a hierarchy mapping the dependencies that must be dealt with in order to start the unit in question. Dependencies, in this context, include those units that are either _required_ by or _wanted_ by the units above it:

```bash
$ sshd.service
● ├─sshd-keygen.service
● ├─system.slice
● └─basic.target
●   ├─rhel-dmesg.service
●   ├─selinux-policy-migrate-local-changes@targeted.service
●   ├─paths.target
●   ├─slices.target
●   │ ├─-.slice
●   │ └─system.slice
●   ├─sockets.target
●   │ ├─dbus.socket
...
```

The recursive dependencies are only displayed for `.target` units, which indicate system states. To recursively list all dependencies, include the --all flag.

To show reverse dependencies (units that depend on the specified unit), you can add the `--reverse` flag to the command:

```bash
$ systemctl list-dependencies sshd --reverse
sshd.service
● └─multi-user.target
●   └─graphical.target
```

#### Manage default target

To see the [default target](systemd.md) the system boots into, run the following command:

```bash
$ systemctl get-default
multi-user.target
```

This returns the default.target unit is symlinked to.

To set the default target, use the set command:

```bash
$ systemctl set-default multi-user
```

### Kill unit processes

Systemd also allows to [kill](kill.md) service proccesses managed by systemd. Read more about why this approach is preferred over using the [kill command](kill.md) in [this post](http://0pointer.net/blog/projects/systemd-for-admins-4.html).

Consider the following example. When I run status command on the nginx service unit, I see that it launched two processes:

```bash
$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2018-08-18 19:13:18 UTC; 1min 5s ago
  Process: 910 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 821 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 813 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 912 (nginx)
   CGroup: /system.slice/nginx.service
           ├─912 nginx: master process /usr/sbin/nginx
           └─913 nginx: worker process
...
```

To kill all those processes I would run:

```bash
$ systemctl kill nginx
```

This will ensure that `SIGTERM` signall is delivered to all processes of the nginx service, not just the main process.

You can also send a different signal if you wish. For example, if you are bad-ass you might want to go for SIGKILL right-away:

```bash
$ systemctl kill -s SIGKILL nginx
```

Sometimes all you need is to send a specific signal to the main process of a service, maybe because you want to trigger a reload via SIGHUP. Instead of going via proccess PID, here's an easier way to do this:

```bash
$ systemctl kill -s HUP --kill-who=main crond.service
```

How does killing relate to stopping a service? `kill` goes directly and sends a signal to every process in the service group, however stop goes through the official configured way to shut down a service, i.e. invokes the stop command configured with ExecStop= in the service file. Usually stop should be sufficient. kill is the tougher version, for cases where you either don't want the official shutdown command of a service to run, or when the service is hosed and hung in other ways.

### Resources used to create this document
* https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd#sect-Managing_Services_with_systemd-Introduction
* https://www.freedesktop.org/software/systemd/man/systemctl.html
* https://www.linux.org/docs/man1/systemctl.html
* https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units
* https://www.scaleway.com/docs/learn-systemd-essentials/
* http://www.enricozini.org/blog/2017/debian/systemd-02-cmdline/
* https://serverfault.com/questions/700862/do-systemd-unit-files-have-to-be-reloaded-when-modified
* http://0pointer.net/blog/projects/systemd-for-admins-4.html
