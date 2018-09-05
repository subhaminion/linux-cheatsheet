## du (disk usage)

The **du** (disk usage) command is used for checking the sizes of files and directories (in other words, how much disk space they use).

### Check disk usage in the directory

Without any arguments the `du` command will _recursively_ list the space that each subdirectory in the current working directory and the current directory itself. Use the **-h** option to make the `du` output **human-readable**, that is to display it in kilobytes (`K`), megabytes (`M)` and gigabytes (`G`) rather than  in the default kilobytes.:

```bash
$ find . -type d # list directories in the current directory
.
./test-dir
./test-dir/subdir1
./test-dir/subdir1/subdir2
./.ssh
./.cache
$ du -h # list sizes of directories in the current directory
25M	./test-dir/subdir1/subdir2
25M	./test-dir/subdir1
49M	./test-dir
8.0K	./.ssh
4.0K	./.cache
49M
```

Note that both command `find` and `du` aaslso list the current working directory itself indicated by dot `.`.

---

By default, `du` command recursively lists all subdirectories, to restrics how deep in directory tree the `du` command should go, you can use the **-d** (**--max-depth**) option:

```bash
$ du -hd 1 # display sizes of directoris only in the current directory
49M	./test-dir
8.0K	./.ssh
4.0K	./.cache
49M	.
$ du -hd 2 # go 1 level deeper than the current directory
25M	./test-dir/subdir1
49M	./test-dir
8.0K	./.ssh
4.0K	./.cache
49M	.
$ du -hd 3 # go 2 levels deeper than the current directory
du -hd 3
25M	./test-dir/subdir1/subdir2
25M	./test-dir/subdir1
49M	./test-dir
8.0K	./.ssh
4.0K	./.cache
49M	.
```

---

To list directory sizes in some specific directory, provide the path to that directory as an argument to the `du` command:

```bash
$ du -h ./test-dir/
25M	./test-dir/subdir1/subdir2
25M	./test-dir/subdir1
49M	./test-dir/
```

### Don't display content from other filesystems

By default the `du` command will search recursively either from where specified or otherwise from within the current working directory. If we have other files systems mounted, such as an NFS share mounted to `/mnt`, we may not want to go and look through these other file systems which may be from different disks or partitions. This is where the **-x** option comes in, it will ignore all other file systems that it may find.

For example, the following command will find the top disk usage by the top level directories:

```bash
$ du -h -d 1 / | sort -h
du: cannot access ‘/proc/5092/task/5092/fd/3’: No such file or directory
du: cannot access ‘/proc/5092/task/5092/fdinfo/3’: No such file or directory
du: cannot access ‘/proc/5092/fd/4’: No such file or directory
du: cannot access ‘/proc/5092/fdinfo/4’: No such file or directory
0	/dev
0	/media
0	/mnt
0	/opt
0	/proc
0	/srv
0	/sys
4.3M	/vagrant
4.5M	/run
8.6M	/home
34M	/etc
182M	/boot
300M	/var
786M	/usr
1.1G	/root
1.1G	/tmp
3.4G	/
```

As you can see, it generates errors when trying to access `/proc` directory. To exclude `/proc`, `/boot` and other directories that belong to a filesystem different than the root (`/`) directory under which we run the command, we can use the `-x` option:

```bash
$ du -xh -d 1 / | sort -h
0	/media
0	/mnt
0	/opt
0	/srv
4.3M	/vagrant
8.6M	/home
34M	/etc
300M	/var
786M	/usr
1.1G	/root
1.1G	/tmp
3.2G	/
```

### Get total size of a directory

The `du -h` command reports the total size of a directory at the end, but it also list the sizes of all subdirectories it contains. If you're only interested in the total size of a directory, use the `-s` option to only get the total size:

```bash
$ du -hs ./test-dir
49M	./test-dir
```

### Check size of files

The **-a** (**--all**) option tells `du` to report not just the total disk usage for each directory at every level in a directory tree but also to report the size for each individual file:

```bash
$ du -h ./test-dir/ # lists only directories
25M	./test-dir/subdir1/subdir2
25M	./test-dir/subdir1
49M	./test-dir/
$ du -ha ./test-dir/ # also lists files
24M	./test-dir/file2.txt
4.0K	./test-dir/file1.txt
24M	./test-dir/subdir1/subdir2/blob.txt
25M	./test-dir/subdir1/subdir2
25M	./test-dir/subdir1
49M	./test-dir/
```

---

Show only files that are larger than a specific value by setting the size filter with the **-t** option:

```
# du -hat 100M  /
131M	/var/lib
194M	/var
160M	/usr/lib/x86_64-linux-gnu
114M	/usr/lib/juju-1.25.6/bin/jujud
109M	/usr/lib/juju-1.25.6/bin/juju
380M	/usr/lib/juju-1.25.6/bin
380M	/usr/lib/juju-1.25.6
738M	/usr/lib
201M	/usr/share
113M	/usr/src
1.2G	/usr
```

---

Include the last modification time information in the output using the **--time** option:

```
$ du -ha --time ./test-dir/
24M	2018-03-28 05:20	./test-dir/file2.txt
4.0K	2018-03-28 07:16	./test-dir/.test
4.0K	2018-03-28 05:19	./test-dir/file1.txt
24M	2018-03-28 05:41	./test-dir/subdir1/subdir2/blob.txt
25M	2018-03-28 05:41	./test-dir/subdir1/subdir2
25M	2018-03-28 05:41	./test-dir/subdir1
49M	2018-03-28 07:16	./test-dir/
```

### Filtering the output

Sort output items by size by piping the output to the [sort](sort.md) command.

The following command will find the top 5 directories that take up the most of the disk space:

```bash
$ du -xh -d 1 / | sort -hr | head -5
3.2G	/
1.1G	/tmp
1.1G	/root
786M	/usr
298M	/var
```

---

As `du` will often generate more output than can fit on the monitor screen at one time, the output will fly by at high speed and be virtually unreadable, in this case it makes sense to pipe the output to the less command:

```bash
$ du -h | less
```


### Resources used to create this document:

* https://www.rootusers.com/13-du-disk-usage-command-examples-in-linux/
* https://www.tecmint.com/check-linux-disk-usage-of-files-and-directories/
* http://www.linfo.org/du.html
