## wc

The **wc** utility is used to count the number of lines, words, or characters contained in a file or standard input.  

```bash
$ wc /var/log/httpd/access_log
  6  72 489 /var/log/httpd/access_log
```

Without any options, the `wc` command will print the number of lines (`6`), the number of works (`72`), and the number of bytes (`489`) in the given input.

### Useful options

#### Count the number of lines

A **line** is defined as a string of characters delimited by a `<newline>` character. Use the **-l** option to count the number of lines in a file:

```bash
$ cat /var/log/httpd/access_log
::1 - - [17/Jul/2018:05:06:13 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:24 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:26 +0000] "GET /status HTTP/1.1" 404 204 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:30 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:45 +0000] "GET /metrics HTTP/1.1" 404 205 "-" "curl/7.29.0"
::1 - - [17/Jul/2018:05:06:48 +0000] "GET / HTTP/1.1" 200 15 "-" "curl/7.29.0"
$ wc -l /var/log/httpd/access_log
6 /var/log/httpd/access_log
```

A few practical examples.

1. To count the number of records (or rows) in several CSV files the `wc` can be used in conjunction with pipes. In the following example there are five CSV files. The requirement is to find out the sum of records in all five files. This can be achieved by piping the output of the cat command to wc.

    ```bash
    $ cat *.csv | wc -l
    1866
    ```

2. To count the number of folders and files in a directory wc can be combined with the ls command. By passing the -1 options to ls it will each folder or line on a new line. This can be piped to wc to give a count:

    ```bash
    $ ls -l
    total 0
    drwxrwxr-x. 2 vagrant vagrant 22 Aug  4 05:16 dir1
    -rw-rw-r--. 1 vagrant vagrant  0 Aug  4 05:16 file1.txt
    -rw-rw-r--. 1 vagrant vagrant  0 Aug  4 05:16 file2.txt
    $ ls -1 | wc -l
    3
    ```

3. To cound the number of currently logged in users, you can pipe the output of a [who](who.md) command to the `wc`:

    ```bash
    $ who | wc -l
    1
    ```

#### Count the number of words

A **word** is defined as a string of characters delimited by white space characters. Use the **-w** to count the number of words:

```bash
$ cat file1.txt
one two three
$ cat file2.txt
one two
$ wc -w file1.txt file2.txt
 3 file1.txt
 2 file2.txt
 5 total
```

#### Count the number of characters

The **-m** option allows to count the number of characters. Recall, that characters include not only letters and digits, but also special characters such _spaces_, _newline character_, etc.:

```bash
$ wc -m file1.txt file2.txt
14 file1.txt
 8 file2.txt
22 total
```

### Resources used to create this document:

* man wc
* https://shapeshed.com/unix-wc/
* https://alvinalexander.com/unix/edu/examples/wc.shtml
