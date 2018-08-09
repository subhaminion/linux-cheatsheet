## Globbing

Linux shell supports filename expansion which allows you to specify a pattern instead of specific file name to search for files. For example, the following command will list all the files in the current directory because the asterisk character (`*`) is treated by shell as any sequence of characters:

```bash
$ ls *
est.md  file1.txt  file2.txt  lest.md  test.md
```

These special characters like `*` which can expand to multiple filenames are called **wildcards** and the operation of using wildcards for filename expansion is called **globbing**.

### Wildcards

A **wildcard character** (or **wildcard** for short) is a kind of _placeholder_ represented by a character, such as an asterisk (`*`), which can be interpreted by shell as a number of characters or an empty string.

The term `wildcard` or `wild card` was originally used in card games to describe a card that can be assigned any value that its holder desires. However, its usage has spread so that it is now used to describe _an unknown or unpredictable factor_ in a variety of fields.

#### Star wildcard

The **star wildcard** (`*`) is the most popular and usually the most useful wildcard.

The star wildcard has the broadest meaning of any of the wildcards, as this can represent _any number of characters_(including zero, in other words, zero or more characters).

Because the `star wildcard` represents any string, it can be used as an argument to commands like [ls](ls.md) to represent every file in the directory:

```bash
$ ls -l *
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 est.md
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 lest.md
-rw-rw-r--. 1 vagrant vagrant 8 Aug  8 05:21 test.md
```

As you can see, the command above lists every file in the _current directory_ (i.e., the directory in which the user is currently working).

Wildcards can be combined with other characters to represent parts of strings. For example, to represent any file that has a `.txt` filename extension, `*.txt` could be used. Likewise, `file*` would represent all objects that begin with the `file` string:

```bash
$ ls -l *.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
$ ls -l file*
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
$ ls -l *est* # list all files that have `est` substring in their names
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 est.md
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 lest.md
-rw-rw-r--. 1 vagrant vagrant 8 Aug  8 05:21 test.md
```

#### Question mark wildcard

The **question mark wildcard** (`?`) represents _any single character_. 

If you specified something at the command line like `?est.md`, Linux would look for `aest.md`, `best.md`, `cest.md` and every other letter/number between `a-z`, `0-9`:

```bash
$ ls -l ?est.md
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 lest.md
-rw-rw-r--. 1 vagrant vagrant 8 Aug  8 05:21 test.md
```

Note that question mark wildcard doesn't represent zero characters as the star wildcard does:

```bash
$ ls -l *est.md
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 est.md
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 lest.md
-rw-rw-r--. 1 vagrant vagrant 8 Aug  8 05:21 test.md
```

Two question marks in succession would represent any two characters in succession, and three question marks in succession would represent any string consisting of three characters, etc:

```bash
$ ls -l *.??
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 est.md
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 lest.md
-rw-rw-r--. 1 vagrant vagrant 8 Aug  8 05:21 test.md
$ ls -l *.???
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
```

#### Square brackets wildcard

The **square brackets wildcard** (`[]`) represents one of the characters enclosed in the brackets.

For example, the following command will list all files whose names start with either `f` or `t` letter:

```bash
$ ls -l [ft]*
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
-rw-rw-r--. 1 vagrant vagrant 8 Aug  8 05:21 test.md
```

When a hyphen is used between two characters in the square brackets wildcard, it indicates a _range_ inclusive of those two characters. For example, the following would provide information about all of the objects in the current directory that begin with any letter from `a` through `f`:


```bash
$ ls
est.md  file1.txt  file2.txt  lest.md  test.md
$ ls -l [a-f]*
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 est.md
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
```

And the following will list every file in the current directory whose name includes at least one numeral:

```bash
$ ls -l *[0-9]*
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
```

You may also **reverse the search** by using **^** or **!** signs in front of the characters in brackets. This is a logical NOT.

For example, the following construct rather than matching any characters inside the brackets will match any character, as long as it is not listed in the brackets:

```bash
$ $ ls -l [!ft]*
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 est.md
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 lest.md
$ ls -l [^ft]*
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 est.md
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 lest.md
```

This will list files whose names do NOT start with either `f` or `t` letter.

Compare this to the construct without denial which finds files whose names DO start with `f` or `t`:

```bash
$ ls -l [ft]*
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
-rw-rw-r--. 1 vagrant vagrant 8 Aug  8 05:21 test.md
```

Note: do not use `[!]` construct if it directly follows another wildcard like `*` or `?`, as it may not work as you expect:

```bash
$ ls -l *[0-9]* # list files which have a number in their names
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
$ ls -l *[!0-9]* # will list all files
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 est.md
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  8 05:33 lest.md
-rw-rw-r--. 1 vagrant vagrant 8 Aug  8 05:21 test.md
```

But it does work fine, if it doesn't directly follow another wildcard:

```bash
$ ls -l *.[^m]* # list files whose extension doesn't start with the 'm' letter
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file1.txt
-rw-rw-r--. 1 vagrant vagrant 0 Aug  4 05:16 file2.txt
```

#### Curly brackets wildcard

The **curly brackets wildcard** (`{}`) allows to specify a list of possible filename values.

For example, the following command will create three new files because three possible values are listed in the curly brackets:

```bash
$ touch script{1,2,3}.sh
$ ls script*
script1.sh  script2.sh  script3.sh
```

This is often useful to back up a file, i.e. create a copy of it:

```bash
$ cp script1.sh{,.bak}
$ ls script1*
script1.sh  script1.sh.bak
```

### Preventing globbing

Globbing can be prevented using quotes or by escaping wildcards using an escape character (`/`):

```bash
$ ls *
est.md  file1.txt  file2.txt  lest.md  test.md
$ ls '*'
ls: cannot access *: No such file or directory
$ ls "*"
ls: cannot access *: No such file or directory
$ ls \*
ls: cannot access *: No such file or directory
```

### Globbing vs Regular Expressions

_Do not confuse glogging with [regular expressions](regex.md)!_

First of all, globbing is specific to the Linux shell and is used for _filename expansion_ only, whereas regular expressions are supported by different tools (e.g. `grep`, `vim`, `sed`) and programming languages and used when _working with text_ to find a matching strings or line.

Second, the wild ard characters have different meanings in globbing and regular expressions. For example, the question mark wildcard `?` in globbing means any single character, whereas in regular expressions it [makes preceding character (or group of characters) optional](https://www.regular-expressions.info/optional.html).

### Gotchas

Filename expansion can match [hidden files](dotfiles.md) or [dotfiles](dotfiles.md) (i.e. files whose names start with `.` character), but only if the pattern explicitly includes the dot as a literal character:

```
~/[.]bashrc    #  Will not expand to ~/.bashrc
~/?bashrc      #  Neither will this.
               #  Wild cards and metacharacters will NOT
               #+ expand to a dot in globbing.

~/.[b]ashrc    #  Will expand to ~/.bashrc
~/.ba?hrc      #  Likewise.
~/.bashr*      #  Likewise.
```

### Resources used to create this document:

* http://www.linfo.org/wildcard.html
* https://en.wikipedia.org/wiki/Wildcard_character
* http://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm
* http://linuxg.net/curly-brackets-expansion-in-bash-with-5-practical-examples/
* https://www.w3resource.com/linux-system-administration/file-globbing.php
* http://www.tldp.org/LDP/abs/html/globbingref.html
