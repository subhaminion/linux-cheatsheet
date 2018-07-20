## grep

**grep** is a powerful file pattern searcher that comes equipped on every distribution of Linux.
egrep, fgrep

Due its varying functionalities, it has many variants including **grep**, **egrep** (Extended GREP), **fgrep** (Fixed GREP), **pgrep** (Process GREP), **rgrep** (Recursive GREP) etc. But these variants have minor differences to original grep and provide just a subset of functionality of the original grep which is suited for different purposes.

### Useful options

#### Search for a string

This is a basic example of searching for a string inside a file:

```bash
$ cat access_log
::1 - - [17/Jul/2018:05:06:13 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:24 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:26 +0000] "GET /status HTTP/1.1" 404 204 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:30 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:48 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
$ grep /metrics access_log
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0
```

As you can see, the `grep` command print the line in a file that matches a given pattern which in this case is a substring `/metrics`.

You can also pass multiple files to `grep` as input. For example:

```bash
$ cp access_log access_log2
$ grep /metrics access_log*
access_log:::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
access_log2:::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
```

Note, the if you pass multiple files to `grep`, it will prefix each line with the name of the file in which the line exists.

If the string you're looking for contains spaces, enclose it in quotes:

```bash
$ grep "GET /metrics" access_log
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
```

**If you're using a fixed string (not a regular expressions) as your search pattern as in the examples above, use `fgrep` instead of `grep` command.**

**fgrep** (sometimes referred to as "fast grep") works faster than `grep` because unlike `grep` it doesn't support regular expressions and works only with fixed strings:

```bash
$ fgrep "GET /metrics" access_log
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
```

Note that `fgrep` command is equivalent to `grep -F`.

`grep` is also often used to filter output produced by another commaned such as [ps](ps.md) or [netstat](netstat.md):

```bash
$ netstat -tpln | fgrep httpd
tcp6       0      0 :::80                   :::*                    LISTEN      2450/httpd
```

#### Search in all files under the folder

If you want to search in all under some directory and its subdirectories, use the **-r** option to make a _recursive_ search:

```bash
$ fgrep -r /metrics . # will search all files under the current folder
./access_log:::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
./access_log2:::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
```

#### Print line numbers

Use the **-n** to print numbers of the of the matched lines:

```bash
$ cat access_log
::1 - - [17/Jul/2018:05:06:13 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:24 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:26 +0000] "GET /status HTTP/1.1" 404 204 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:30 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:48 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
$ fgrep -n 404 access_log
3:::1 - - [17/Jul/2018:05:06:26 +0000] "GET /status HTTP/1.1" 404 204 "-" "curl/7.29.0"
5:::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
```

As you can see, each printed line is prefixed with its line number in the file (`3`, `5`).

This can be useful if you are looking to edit a file and want to launch vim and go straight to the line:

```bash
$ vim +3 access_log
```

#### Case insensitive search

Use the **-i** option for case insensitive search. Compate the two examples:

```bash
$ fgrep hello file.txt
hello world
$ fgrep -i hello file.txt
Hello world
HeLLo world
hello world
```

#### Check for full words, not substrings

If you want to search for a word, and to avoid it to match the substrings use the **-w** option. Compate the two examples:

```bash
$ fgrep -i is file.txt
THIS day
his second chance
it is awesome
```

As you can see, by default `grep` will look for any lines that contain `is` in it and this includes lines where `is` is a substring of another word or a separate word in itself.

With the `-w` option, it will only print those lines where `is` is a separate word:

```bash
$ fgrep -iw is file.txt
it is awesome
```

Another way to achieve the same thing would be to include spaces in the search pattern:

```bash
$ fgrep -i " is " file.txt
it is awesome
```

#### Print lines before and after a match

Use the **-A** option to specify a number of lines _after_ a matched line which should be printed, and **-B** to print lines before the match:

```bash
$ fgrep -A1 -B1 /metrics access_log
::1 - - [17/Jul/2018:05:06:30 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:48 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
```

You can also use the **-C** option, if you want the same number of lines to be printed before and after the match:

```bash
$ fgrep -C1 /metrics access_log
::1 - - [17/Jul/2018:05:06:30 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:48 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
```

#### Count the number of mathed lines

Use the **-c** option to count the number of matched lines:

```bash
$ fgrep -w 404 access_log
::1 - - [17/Jul/2018:05:06:26 +0000] "GET /status HTTP/1.1" 404 204 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
$ fgrep -cw 404 access_log
2
```

#### Stop reading a file after a number of matches

If you specify the maximum number of matches with **-m** option, `grep` will stop reading the input after it found the maximum number of matching lines:

```bash
$ fgrep -i hello file.txt
Hello world
HeLLo world
hello world
$ fgrep -i -m2 hello file.txt
Hello world
HeLLo world
$ fgrep -i -m1 hello file.txt
Hello world
```

#### Invert match

Use the **-v** option to invert your search and select lines which are not matching a given pattern. This allows to exclude certain patterns from your search.

```bash
$ fgrep -w 404 access_log
::1 - - [17/Jul/2018:05:06:26 +0000] "GET /status HTTP/1.1" 404 204 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
$ fgrep -w 404 access_log | fgrep -v /status
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
```

#### Find files with a match

Use the **-l** option to print the names of the files where the match exists. This could be helpful to find logs files which contain events you are interested among many log files for a given period of time. For example, the following command will print the paths to the log files which contain response code `404`:

```bash
$ fgrep -rwl 404 .
./access_log
./access_log2
```

#### Use of regular expressions when searching

`grep` supports [basic regular expressions](https://www.digitalocean.com/community/tutorials/using-grep-regular-expressions-to-search-for-text-patterns-in-linux)

For example, it supports _anchors_ (`^`, `$`) which specify where in the line a match must occur to be valid. For example, the following command looks for lines which end with `ia`:

```bash
$ cat file.txt
ia brrr
abasia
Abelia
jfdksj ia fd
$ grep ia$ file.txt
abasia
Abelia
```

It supports bracket expressions. For example, the following command finds log entries between `2018:05:06:20-2018:05:06:39`:

```bash
$ cat access_log
::1 - - [17/Jul/2018:05:06:13 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:24 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:26 +0000] "GET /status HTTP/1.1" 404 204 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:30 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:48 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
$ grep "2018:05:06:[2-3]" access_log
::1 - - [17/Jul/2018:05:06:24 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:26 +0000] "GET /status HTTP/1.1" 404 204 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:30 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
```

In basic regular expressions, meta-characters like: `{`,`}`, `(`, `)`', `|`,`+`,`?` are treated as normal characters and need to be escaped if they are to be treated as special characters.

For example, when the following command is run without escaping `(` `)` and `|` then it searches for the complete string i.e. `(f|g)ile` in the file, and as a result it finds nothing:

```bash
$ cat file.txt
file
Abelia
agile
file2
blah
$ grep "(f|g)ile" file.txt
```

But when the special characters are escaped, then instead of treating them as part of a string, `grep` treates them as meta-characters and searches for words `file` or `gile` in the file:

```bash
$ grep "\(f\|g\)ile" file.txt
file
agile
file2
```

Luckily, `grep` supports [extended regular expressions](https://www.tecmint.com/difference-between-grep-egrep-and-fgrep-in-linux/). To use extended regular expressions, you either need to pass the `-E` option to the `grep` command or use `egrep` (which is equivalent to `grep -E` similar to how `fgrep` is equivalent to `grep -F`):

```bash
$ grep -E "(f|g)ile" file.txt
file
agile
file2
$ egrep "(f|g)ile" file.txt
file
agile
file2
```

### Resources used to create this document:

* https://www.tecmint.com/12-practical-examples-of-linux-grep-command/
* https://www.tecmint.com/difference-between-grep-egrep-and-fgrep-in-linux/
* https://shapeshed.com/unix-grep/
* https://www.thegeekstuff.com/2009/03/15-practical-unix-grep-command-examples/
* https://www.digitalocean.com/community/tutorials/using-grep-regular-expressions-to-search-for-text-patterns-in-linux#literal
* man grep
