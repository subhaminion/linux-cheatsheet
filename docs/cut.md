## cut

The **cut** command allows to cut sections from each line of input and print the results to the standard output. It can be used to cut parts of lines by byte position, character and delimiter.

### Useful options

#### Cut lines by character position

To cut lines by character positions, use the **-c** option. This selects the characters from each line at positions given to the -c option.

Charachter positions can be specified as a single number. For example, the following command selects second character in each line and print it out:

```bash
$ cat file.txt
shla sasha
po shosse i
sossala sushki
$ cut -c2 file.txt
h
o
o
```

We can also specify a list of numbers separated by commas:

```bash
$ cut -c1,2,4 file.txt
sha
pos
sos
```

Or use a range of numbers. For example, the following command can be used to print the names and login times of the currently logged in users:

```bash
$ who | cut -c 1-16,26-38
vagrant  pts/0  8-08-07 06:00
vagrant  pts/1  8-08-07 06:19
```

You can also specify only the start or the end of the range:

```bash
$ cut -c-5 file.txt # select only the first five characters of each line
shla
po sh
sossa
$ cut -c3- file.txt # print the whole line starting from the third character
la sasha
 shosse i
ssala sushki
```

#### Cut lines based on delimeter

To cut using a delimiter use the **-d** option. This is normally used in conjunction with the **-f** option to extract specific fields of line the which are created as a result of cutting the line with the delimiter.

For example, the following example uses a space delimiter (`' '`) to cut each line and print the second field:

```bash
$ cut -d ' ' -f2 file.txt
sasha
shosse
sushki
```

Note that the **default delimiter** is the tab character.

Similar to specifying a list or range of character positions, you can specify a list or range of fields to select:

```bash
$ cat test.csv
John,Smith,34,London
Arthur,Evans,21,Newport
George,Jones,32,Truro
$ cut -d ',' -f1 test.csv
John
Arthur
George
$ cut -d ',' -f2-4 test.csv
Smith,34,London
Evans,21,Newport
Jones,32,Truro
```

Here is an example of displaying username and home directory of users who has the login shell as `/bin/bash`:

```bash
$ grep '/bin/bash' /etc/passwd | cut -d ':' -f1,6
root:/root
vagrant:/home/vagrant
ubuntu:/home/ubuntu
```

**NOTE:** if a line doesn't contain a delimiter, the whole line will be printed:

```bash
$ cat test.csv
John,Smith,34,London
Arthur	Evans	21	Newport
George,Jones,32,Truro
$ cut -d ',' -f2 test.csv
Smith
Arthur	Evans	21	Newport
Jones
```

To avoid printing lines which don't contain the delimiter, use the **-s** option:

```bash
$ cut -d ',' -f2 -s test.csv
Smith
Jones
```

#### Inverse the selection

The **--complement** option selects everythin except the options you specify. Here is an example.

The normal `cut` will select the second field when passed the `-f2` option:

```bash
$ cat test.csv
John,Smith,34,London
Arthur,Evans,21,Newport
George,Jones,32,Truro
$ cut -d ',' -f2 test.csv
Smith
Evans
Jones
```

But if we add `--complement`, it will print everythin except for the second field:

```bash
$ cut -d ',' -f2 --complement test.csv
John,34,London
Arthur,21,Newport
George,32,Truro
```

#### Change output delimiter

To modify the output delimiter use the **--output-delimiter** option.

In the following example a semi-colon is converted to a space and the first, third and fourth fields are selected.

```bash
$ echo 'how;now;brown;cow' | cut -d ';' -f1,3,4 --output-delimiter=' '
how brown cow
```

And the following example converts semi-colon into a new line:

```bash
$ echo 'how;now;brown;cow' | cut -d ';' -f1,3,4 --output-delimiter=$'\n'
how
brown
cow
```

### Resources used to create this document:

* https://www.thegeekstuff.com/2013/06/cut-command-examples/
* https://shapeshed.com/unix-cut/
* man cut
