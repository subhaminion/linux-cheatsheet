## rsync

**Rsync** (stands for _remote sync_) is a command line utility which synchronizes files and folders from one location to another.

`Rsync` is famous for its **delta-transfer algorithm**. When copying files, `rsync` checks if there is already data that you are copying and will only copy the differences between the source files and the existing files in the destination. This greatly reduces the amount of data sent over the network. For example, let's say that you were to back up 500G of disk space to a remote storage. The initial copying operation will take a long time, but all the subsequent runs of the same command will likely to be much faster because only the differences between the files will copied, i.e. if only 100M out of those 500G have changed, only those 100M will be sent - not the entire 500G.

`Rsync` is widely used for _directory synchronization_ remotely and locally, _backups_ and _mirroring_, and as an improved copy command for everyday use.

#### Reasons to use rsync over CP and SCP commands

* copies from source to destination only the data which is different between the two locations
* rsync’s `--delete` option deletes files located at the destination which are no longer at the source
* rsync can compress data before transferring with the `-z` option, so no need to pipe to an archiving utility

### Basic usage

The basic structure of an rsync command is similar to [cp](cp.md) and [scp](scp.md):

```
$ rsync -[options] source destination
```

If you have multiple destinations, they’re appended to the end of the string like is done with the cp command:

```
$ rsync -[options] source destination1 destination2 destination3
```

Either the source or the destination, or both, can be local or remote. If you’re synchronizing files over a network, then both the local and remote machines will need rsync installed.

_Rsync uses **SSH** when transferring over networks._

Remote locations are formatted like [scp](scp.md) command. For example, to synchronize a local folder with one on a remote server, you’d use:

```bash
$ rsync -[options] /path/to/source_folder username@<remote_host>:/path/to/destination_folder
```

### Notes about rsync command

**NOTE 1:** the trailing slash (**/**) at the end of the source directory is important.

_Case 1._ If the source directory has a trailing slash `/`, the `rsync` will copy only the contents of the source directory, but will not copy the source directory itself:

```bash
$ ls dir1
file1.txt  file2.txt  subdir1
$ rsync -r dir1/ dest1
$ ls dest1
file1.txt  file2.txt  subdir1
```

_Case 2._ If the source directory does NOT have a trailing slash `/`, the `rsync` will copy the contents of the source directory and the source directory itself:

```bash
$ rsync -r dir1 dest2
$ ls dest2
dir1
$ ls dest2/dir1/
file1.txt  file2.txt  subdir1
```

**NOTE 2:** If destination directory is not created then it will be created automatically when you run rsync

### Common commands & options

Some common options used with `rsync` commands:

* `-a` : This is a combination flag and is equivalent to `-rlptgoD` combination of options. The `-a` stands for "archive" mode and syncs _recursively_ and preserves symbolic links, special and device files, timestamps, group, owner, and permissions. It is basically a quick way of saying you want recursion and want to preserve almost everything with `-H` (used to preserve hard links) being a notable omission.
* `-r` : Copies data recursively, but unlike the `-a` flag, this doesn't preserve timestamps and permissions while transferring data.
* `-v` : Activates verbose mode. By default, rsync works silently. A single `-v` will give you information about what files are being transferred and a brief summary at the end. Two `-v` flags will give you information on what files are being skipped and slightly more information at the end.
* `-z`: With this option, `rsync` compresses the file data as it is sent to the destination machine, which reduces the  amount of data being transmitted -- something that is useful over a slow connection.
* `-h` : Outputs numbers in human-readable format.
* `--progress`: Shows progress during the transfer.

#### Copy files locally

This will copy file `1gb-file` to `/tmp` directory preserving timestamps, group and owner permissions of the file:

```bash
$ rsync -avh --progress 1gb-file /tmp
sending incremental file list
1gb-file
          1.07G 100%  139.71MB/s    0:00:07 (xfr#1, to-chk=0/1)

sent 1.07G bytes  received 35 bytes  126.35M bytes/sec
total size is 1.07G  speedup is 1.00
```

#### Sync local directories

The following command will create a local backup of a directory:

```bash
$ rsync -avh linux-cheatsheet/ /tmp/linux-cheatsheet.bak
sending incremental file list
created directory /tmp/linux-cheatsheet.bak
./
README.md
.git/
.git/HEAD
.git/config
.git/description
...
sent 4.18M bytes  received 2.26K bytes  8.37M bytes/sec
total size is 4.18M  speedup is 1.00
```

If I run it again, no file data will be transferred and the operation will finish immediately because files in two locations are already in sync:

```bash
$ rsync -avh linux-cheatsheet/ /tmp/linux-cheatsheet.bak
sending incremental file list

sent 3.06K bytes  received 32 bytes  6.18K bytes/sec
total size is 4.18M  speedup is 1,351.08
```

#### Exclude files from transferring

`rsync` provides `--exclude` option which allows to specify matching patterns for files you want to exclude from syncing. In the following example, I will exclude `.git` folder from sync:

```bash
$ rsync -avh --exclude='.git' linux-cheatsheet/ /tmp/linux-cheatsheet2.bak
sending incremental file list
created directory /tmp/linux-cheatsheet2.bak
./
README.md
docs/
docs/awesome-things.md
docs/boot-process.md
...
sent 2.42M bytes  received 1.76K bytes  4.85M bytes/sec
total size is 2.42M  speedup is 1.00
```

#### Dry Run

`rsync` has a very useful `-n` (`--dry-run`) option which allows you see what an rsync command will do without actually making any changes:

```bash
$ rsync -avh -n  linux-cheatsheet/ /tmp/linux-cheatsheet4.bak
sending incremental file list
created directory /tmp/linux-cheatsheet4.bak
./
README.md
.git/
.git/HEAD
.git/config
.git/description
```

But if we check the destination directory, it wasn't even created, so no actual changes were made:

```bash
$ ls /tmp/linux-cheatsheet4.bak
ls: cannot access /tmp/linux-cheatsheet4.bak: No such file or directory
```

#### Sync local and remote files/directories

`rsync` uses SSH to transfer files to remote locations. Here is a basic example of syncing a directory to a remote location:

```bash
$ rsync -avzh linux-cheatsheet ubuntu@X.X.X.X:/tmp
building file list ... done
linux-cheatsheet/
linux-cheatsheet/README.md
linux-cheatsheet/.git/
linux-cheatsheet/.git/HEAD
linux-cheatsheet/.git/config
linux-cheatsheet/.git/description
...
sent 3.47M bytes  received 2.60K bytes  533.75K bytes/sec
total size is 4.18M  speedup is 1.20
```

If you need to pass additional options to ssh such as a non-default port or a private key location, use the `-e` option:

```bash
$ rsync -avzh -e 'ssh -i ~/keys/test-key.pem' linux-cheatsheet ubuntu@X.X.X.X:/tmp
```

To copy directory from a remote host to a current directory on the local machine, you could use a command like this:

```bash
$ rsync -avzh ubuntu@X.X.X.X:/tmp/linux-cheatsheet .
receiving file list ... done
linux-cheatsheet/
linux-cheatsheet/README.md
linux-cheatsheet/.git/
linux-cheatsheet/.git/HEAD
linux-cheatsheet/.git/config
linux-cheatsheet/.git/description
...
sent 2.60K bytes  received 3.47M bytes  1.39M bytes/sec
total size is 4.18M  speedup is 1.20
```

#### Dealing with new files at the destination host

If a syncronized directory at the destionation host has files that are missing in the directory at the source host, you can remove them during the sync with the `--delete` option.

For example, the following command will remove files that are present on the remote host, but are missing locally:

```bash
$ rsync -avzh --delete linux-cheatsheet ubuntu@X.X.X.X:/tmp
building file list ... done
deleting linux-cheatsheet/new-file.txt
linux-cheatsheet/

sent 2.68K bytes  received 26 bytes  1.81K bytes/sec
total size is 4.18M  speedup is 1542.60
```

To remove new files that are present locally, but are missing on a remote host, simply switch the source and the destination:

```bash
$ touch linux-cheatsheet/new-file2.txt
$ rsync -avzh --delete ubuntu@X.X.X.X:/tmp/linux-cheatsheet/ linux-cheatsheet
receiving file list ... done
deleting new-file2.txt
./

sent 22 bytes  received 2.64K bytes  1.06K bytes/sec
total size is 4.18M  speedup is 1569.83
```

It's wise to use a [dry run](#dry-run) before using this command or/and using [the --backup option to backup removed files](https://www.createdbypete.com/articles/a-practical-guide-to-using-rsync/).

#### Update only existing files

`rsync` allows to only update the existing files at the destination with the `--existing` option. So in case source has new files, which are not present at the destionation, this option will help to avoid creating these new files at the destination:

```bash
$ touch linux-cheatsheet/new-file3.txt
$ rsync -avzh --existing linux-cheatsheet ubuntu@X.X.X.X:/tmp
building file list ... done
linux-cheatsheet/

sent 2.71K bytes  received 26 bytes  1.09K bytes/sec
total size is 4.18M  speedup is 1526.82
$ rsync -avzh linux-cheatsheet ubuntu@X.X.X.X:/tmp
building file list ... done
linux-cheatsheet/new-file3.txt

sent 2.75K bytes  received 42 bytes  1.11K bytes/sec
total size is 4.18M  speedup is 1499.98
```

Besides, you might also want to tell rsync not to update files that are newer on the destination side with the `-u` (`--update`) option. See example [here](https://www.networkworld.com/article/3041481/linux/having-your-way-with-rsync.html).

### Resources used to create this document

* https://opensource.com/article/17/1/rsync-backup-linux
* https://www.networkworld.com/article/3041481/linux/having-your-way-with-rsync.html
* https://www.rsync.net/resources/howto/rsync.html
* https://linuxjourney.com/lesson/rsync
* https://www.createdbypete.com/articles/a-practical-guide-to-using-rsync/
* http://www.linuxbuzz.com/rsync-command-examples-linux-beginners/
* https://www.thegeekstuff.com/2010/09/rsync-command-examples
* https://www.tecmint.com/rsync-local-remote-file-synchronization-commands/
