## dd

**dd** (**Data Duplicator**) is a command-line utility whose purpose is to _copy_ files and _convert_ the format of the data.

The general syntax for the command is the following:

```bash
dd if=<input file name> of=<output file name> [Options]
```

For instance, you can copy a file with the following command:

```bash
$ cat file1.txt
Hello world
$ dd if=file1.txt of=file2.txt
0+1 records in
0+1 records out
12 bytes (12 B) copied, 0.00064786 s, 18.5 kB/s
$ cat file2.txt
Hello world
```

_But how is it different from the [cp](cp.md) command you will ask?_

Unlike the **cp** command **dd** allows to specify low level details about how the copying process should go. For example, it allows you to choose the **block size** (`bs`) to be used when copying the data [to improve perfomance](https://serverfault.com/questions/650086/does-the-bs-option-in-dd-really-improve-the-speed) (see the [catch](#catches) of choosing too large block sizes).

As the result, `dd` command is often used when copying large files. For instance, when you need to backup/restore a disk or a partition, the `dd` may come in handy:

```back
$ dd if=/dev/sda of=/dev/sdb bs=4096 conv=noerror,sync
97281+0 records in
97280+0 records out
99614720 bytes (100 MB) copied, 2.75838 s, 36.1 MB/s
```

Note that this works only if the second device is as large as or larger than the first. Otherwise, you get truncated and worthless partitions on the second one.

In this command we also specify the `block size` of 4KB which is equal to 4096 bytes. If you don't specify block size, `dd` use a default block size of 512 bytes. The number you specify in the `bs` option may also be followed by the different multiplicative suffixes: c=1, w=2, b =512, kB =1000, K =1024, MB =1000*1000, M =1024*1024, GB =1000*1000*1000, G =1024*1024*1024 and so on (see `man dd`).

The **conv** value parameter `noerror` will prevent problems from read errors. You don't need it, but it sucks to see your backup halt 12 hours in because of a small error on your disk. The `sync` option allows to use [synchronized I/O](https://www3.physnet.uni-hamburg.de/physnet/Tru64-Unix/HTML/APS33DTE/DOCU_009.HTM) which ensures that all the data is flashed to disk from the disk cache in RAM before the program exits.

Restoring from the disk backup created this way would be as simple as swapping places of `if` and `of` in the previous command:

```bash
$ dd if=/dev/sdb of=/dev/sda bs=4096 conv=noerror,sync
```

Here are some other useful use cases of `dd` command.

### Use cases

#### Backup disk to an image

Backing up a disk to an image will be faster than copying the exact data. Also, disk image makes the restoration much easier:

```bash
$ dd if=/dev/sda of=/tmp/disk-backup.img
```

You can store the output file where you want but you have to give a filename ending with `.img` extension as above.

**NOTE:**: `dd` command can take a long time to complete, but it unfortunately does not show the progress status.
But if can use this simple trick by sending the process the `USR1` signal to see the progress:

```bash
$ killall -USR1 dd
```

In the terminal where the `dd` command is running, you will see:

```bash
$ dd if=/dev/sda of=/tmp/disk-backup.img
1524882+0 records in
1524882+0 records out
780739584 bytes (781 MB) copied, 5.3687 s, 145 MB/s
```

#### Create a compressed disk image

To create a compressed disk image:

```bash
$ dd if=/dev/sda | gzip >/tmp/disk-backup.img.gz
```

To restore:

```bash
$ gzip –dc /tmp/disk-backup.img.gz | dd of=/dev/sda
```

#### Create a bootable USB drive

You can read an example [here](https://www.ostechnix.com/how-to-create-bootable-usb-drive-using-dd-command/).

I personally find [Etcher](https://etcher.io/) tool easier to use for this purposes. And it's cross platform.

#### Create a file of a fixed size

One of the most useful arguments of the `dd` command is `count` which allows to specify how many blocks (`bs`) to read from the input file. This also clearly shows the difference between the `cp` and `dd`, because with `dd` you can specify how much file data (in blocks) you want to copy if you don't want to copy the whole file.

Thanks to this funtionality, the `dd` command is often used for creating files of the required size. For example, if you need to create a file of 1GB in size, you could use the following command:

```bash
$ dd if=/dev/zero of=1gb-file bs=1M count=1024
```

Here we use [/dev/zero](dev-directory.md) special device file to create a data file with all zeros as a content.

### Catches

When you choose a big block size, you have to be aware that you will need at least the same amount of free memory as your block size for the operation to complete. Remember that disk writes are going to disk cache (RAM) first before they end up on the actual disk.

Consider an example. I have 373M of free memory on my system:

```bash
$ free -h
             total       used       free     shared    buffers     cached
Mem:          489M       124M       365M       368K       180K       7.5M
-/+ buffers/cache:       116M       373M
Swap:           0B         0B         0B
```

Check [free command](free.md) if you're confused by the output.

If I specify a block size bigger then the amount of my free memory, I'll get an error:

```bash
$ dd if=/dev/zero of=1gb-file bs=1G count=1
dd: memory exhausted by input buffer of size 1073741824 bytes (1.0 GiB)
```

No wonder my memory was exhausted by the 1G input since I only have 373 M!

Now if I choose the block size of less than the amount of my free memory, the operation will succeed:

```bash
$ dd if=/dev/zero of=1gb-file bs=350M count=1
1+0 records in
1+0 records out
367001600 bytes (367 MB) copied, 0.405582 s, 905 MB/s
```

### Resources used to create this document:

* http://nicolasbouliane.com/blog/cloning-your-drive-with-the-dd-command-step-by-step-guide
* https://linoxide.com/linux-command/linux-dd-command-create-1gb-file/
* https://www.tutorialspoint.com/unix_commands/dd.html
* https://www.cyberciti.biz/faq/howto-linux-unix-test-disk-performance-with-dd-command/
* https://www.ostechnix.com/how-to-create-bootable-usb-drive-using-dd-command/
