## uname

The **uname** command reports basic information about a computer's operating system and hardware.

When used without any options, `uname` reports the name, but not the version number, of the [kernel](kernel.md) (i.e., the core of the operating system). Thus, on a system running some distribution (i.e., version) of Linux, it will display the word [Linux](kernel.md#linux-is-the-kernel) on the monitor screen:

```bash
$ uname
Linux
```

### Useful options

#### Display all system information

The **-a** option tells `uname` to provide the following information:

```bash
$ uname -a
Linux localhost.localdomain 3.10.0-862.6.3.el7.x86_64 #1 SMP Tue Jun 26 16:32:21 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

* name of the kernel (`Linux`)
* network node host name (`localhost.localdomain`)
* kernel release information  (`3.10.0-862.6.3.el7.x86_64`); from this info, you can get the kernel version number and release level (`3.10.0-862`)
* kernel release date (`Tue Jun 26 16:32:21 UTC 2018`)
* machine hardware name (`x86_64`)
* CPU architecture (`x86_64`)
* hardware platform (`x86_64`)
* [operating system name](kernel.md#linux-is-the-kernel) (`GNU/Linux`)

Below are described options which allow each of these pieces of information to be reported individually

#### Get kernel release information

Use **-s** option to print the kernel name and the **-r** option to display release info:

```bash
$ uname -sr
3.10.0-862.6.3.el7.x86_64
```

#### Get kernel release date

```bash
$ uname -v
#1 SMP Tue Jun 26 16:32:21 UTC 2018
```

#### Get the CPU architecture

```bash
$ uname -p
x86_64
```

#### Check if your Linux OS is 32-bit or 64-bit

The hardware name shows if your OS is 32-bit (`i686` or `i386`) or 64-bit (`x86_64`).

```bash
$ uname -m
x86_64
```

#### Check network hostname

```bash
$ uname -n
localhost.localdomain
```

note, that you will get the same output with the [hostname](hostname.md) command:

```bash
$ hostname
localhost.localdomain
```

### Resources used to create this document:

* https://linoxide.com/linux-command/uname-command/
* http://www.linfo.org/uname.html
* man uname
