## uptime

**uptime**  gives a one line display of the following information:

```bash
$ uptime
 06:00:56 up  2:15,  1 user,  load average: 0.01, 0.03, 0.05
```

where:
* `06:00:56` is the current time on the system
* `up` is state of the system
* `2:15` (2 hours and 15 minutes) is the total time for which the system has been up
* `1 user` is the number of [user sessions](su.md) open
* `load average: 0.01, 0.03, 0.05` is [system load averages](load-averages.md) for the past 1, 5, and 15 minutes correspondigly

### Useful options

**-p** shows only the system up time in pretty format:

```bash
$ uptime -p
up 3 hours, 19 minutes
```

---

**-s** displays the date and time since the system has been running:

```bash
$ uptime -s
2018-05-20 03:45:55
```

### Resources used to create this document:

* https://www.howtoforge.com/linux-uptime-command/
* man uptime
