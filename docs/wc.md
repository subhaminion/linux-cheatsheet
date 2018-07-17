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

Often times, I pipe output from [grep](grep.md) to `wc -l` command to see how many occurrences of the event happened. For example, the following command will count the number of times the apache server responded with `404` response code:

```bash
$ grep ' 404 ' /var/log/httpd/access_log | wc -l
2
```

or this command will count the number of time the index page (`/`) was requested:

```bash
$ grep ' / ' /var/log/httpd/access_log | wc -l
4
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
