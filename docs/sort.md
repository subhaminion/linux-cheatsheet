## sort

**sort** is a command line utility for sorting lines of text input. It allows to sort lines of text alphabetically, in reverse order, by number, by month and can also remove duplicates.

The `sort` command will lines **alphabetically** based on the first word:

```bash
$ sort file.txt
Alisson Becker:25:Marketing
Artem Artemov:23:Eng
Luka Modric:30:Support
Mo Salah:25:Sales
Sergio Ramos:22:Sales
```

**Note**, that sorted text is sent to standard output and printed on the terminal unless redirected:

```bash
$ cat file.txt
Artem Artemov:23:Eng
Sergio Ramos:22:Sales
Alisson Becker:25:Marketing
Luka Modric:30:Support
Mo Salah:25:Sales
```

### Useful options

#### Sort in reverse order

Use the **-r** option to reverse the results of sorting:

```bash
$ sort -r file.txt
Sergio Ramos:22:Sales
Mo Salah:25:Sales
Luka Modric:30:Support
Artem Artemov:23:Eng
Alisson Becker:25:Marketing
```

#### Sort by items not at the beginning of the line

The **-t** option allows you to specify a delimiter for splitting the input lines into fields (similar to how [cut](cut.md) does it). With the **-k** option you can specify a field position which use for sorting. This allows to sort lines based on words that can be positioned anywhere withing the line, not just at the beginning.

The following example will sort the list of people based on where the departement where they work:

```bash
$ sort -t ':' -k3  file.txt
Artem Artemov:23:Eng
Alisson Becker:25:Marketing
Mo Salah:25:Sales
Sergio Ramos:22:Sales
Luka Modric:30:Support
```

As you can see, the command above splits lines into fields (each field separated by the `:` delimiter) and then sorts alpabetically based on the third field (`-k3`).

**Note**, by default, a space is used as a delimiter:

```bash
$ cat file2.txt
1023 AcmeCo "Bouncey Castle"
1003 FooCo "Fluffy Toy"
1013 AcmeCo "Edible Hat"
1042 FooCo "Whoopie Cushion"
$ sort -k2 file2.txt
1023 AcmeCo "Bouncey Castle"
1013 AcmeCo "Edible Hat"
1003 FooCo "Fluffy Toy"
1042 FooCo "Whoopie Cushion"
```

#### Sort by number

To sort by number, pass the **-n** option to sort. This will sort from lowest number to highest number:

```bash
$ sort -t ':' -k2 -n file.txt
Sergio Ramos:22:Sales
Artem Artemov:23:Eng
Alisson Becker:25:Marketing
Mo Salah:25:Sales
Luka Modric:30:Support
```

If you need to sort human readable numbers (e.g., 2K 1G), like those produced by [du -h](du.md) command, then use the **-h** option instead for `-n`.

The following command will find the top 5 directories that take up the most of the disk space:

```bash
$ du -xh -d 1 / | sort -hr | head -5
3.2G	/
1.1G	/tmp
1.1G	/root
786M	/usr
298M	/var
```

#### Sort and remove duplicates

Use the **-u** option to remove duplicates when sorting:

```bash
$ cat file3.txt
Artem Artemov:22:Support
Artem Artemov:23:Sales
Artem Artemov:23:Sales
Foo Bar:24:Eng
$ sort -u file3.txt
Artem Artemov:22:Support
Artem Artemov:23:Sales
Foo Bar:24:Eng
```

#### Scramble order with sort

`sort` allows also to randomly scramble the order of the lines with the **-R** option and the system’s random number generator `/dev/random` or pseudo-random number generator `/dev/urandom`:

```bash
$ cat file.txt
Artem Artemov:23:Eng
Sergio Ramos:22:Sales
Alisson Becker:25:Marketing
Luka Modric:30:Support
Mo Salah:25:Sales
$ sort -R file.txt --random-source=/dev/random
Luka Modric:30:Support
Artem Artemov:23:Eng
Alisson Becker:25:Marketing
Sergio Ramos:22:Sales
Mo Salah:25:Sales
```

### A few practical examples

#### Sort files by filesize

```bash
$ ls -alh | sort -n -k5
total 1.1G
-rw-r--r--.  1 root root 1.0G Aug 28 04:58 1gb-file
dr-xr-x---.  3 root root 4.0K Sep  5 04:11 .
-rw-------.  1 root root 5.4K May 12 18:55 original-ks.cfg
-rw-------.  1 root root 5.7K May 12 18:55 anaconda-ks.cfg
-rw-r--r--.  1 root root   12 Aug 10 05:25 out.txt
```

#### Find what's taking up the most of the space

The following [example](http://www.ducea.com/2006/05/14/tip-how-to-sort-folders-by-size-with-one-command-line-in-linux/) will list the user home directories that take up most of the space on disk:

```bash
$ du -h -d 1 /home/ | sort -hr
8.6M	/home/vagrant
8.6M	/home/
12K	/home/artem
```

### Resources used to create this document

* https://shapeshed.com/unix-sort/
* https://linux.101hacks.com/linux-commands/sort-command-examples/
* https://www.linode.com/docs/tools-reference/tools/manipulate-lists-with-sort-and-uniq/
* http://www.ducea.com/2006/05/14/tip-how-to-sort-folders-by-size-with-one-command-line-in-linux/
* man sort
