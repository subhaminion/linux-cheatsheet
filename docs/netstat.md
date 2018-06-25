## netstat

The **netstat** (network statistics) command prints information about the Linux networking subsystems.

### List open sockets

Without any options, `netstat` prints information about the open [sockets](sockets.md).

#### List all open sockets

To list all open sockets, use the **-a** option:

```bash
$ netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 *:sunrpc                *:*                     LISTEN
tcp        0      0 *:ssh                   *:*                     LISTEN
tcp        0      0 *:48417                 *:*                     LISTEN
tcp        0      0 vagrant-ubuntu-trus:ssh 10.0.2.2:53938          ESTABLISHED
tcp        0      0 vagrant-ubuntu-trus:ssh 10.0.2.2:54381          ESTABLISHED
tcp6       0      0 [::]:59087              [::]:*                  LISTEN
tcp6       0      0 [::]:sunrpc             [::]:*                  LISTEN
tcp6       0      0 [::]:http               [::]:*                  LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
udp        0      0 *:sunrpc                *:*
udp        0      0 *:55170                 *:*
...
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  8      [ ]         DGRAM                    8750     /dev/log
unix  2      [ ACC ]     STREAM     LISTENING     6542     @/com/ubuntu/upstart
unix  2      [ ACC ]     STREAM     LISTENING     8580     /var/run/dbus/system_bus_socket
...
```

As you can see the output is divided into two sections each corresponding to the socket type, i.e. Internet sockets and Unix domain sockets.

Let's break down the output regarding the Internet sockets. You can read about Unix domain section in the man pages of the netstat command:

* `Proto` shows the protocol (tcp, udp, raw) used by the socket. Don't get surprised by `tcp6` or `udp6` protocols, they just mean that at the [network layer](networking-models.md) version 6 of IP protocol is used (IPv6), whereas `tcp` and `udp` indicate that a more common version 4 of IP protocol is used (IPv4).
* `Recv-Q` shows the count of bytes not copied by the user program connected to this socket.
* `Send-Q` is the count of bytes not acknowledged by the remote host.
* `Local Address` shows the address (IP address + port) of the local socket. The asterisk that you will see here is a [wildcard](wildcard.md) which means `any`. For example, `*:ssh` is equivalent to `0.0.0.0:ssh` and means that this socket is bound (associated) to all system's network interaces (IP addresses) and a standard SSH port (22). `[::]` means the same thing but for IPv6.
* `Foreign Address` shows the address of the remote socket. Unless, the socket is in ESTABLISHED state that indicates an established tcp connection and which is associated by the operating system with a [socket pair](sockets.md), you'll likely to see `*:*` (`[::]:*` for IPv6) in this column. `*:*` means that data packets come to this local socket address can come from any IP address and from any port, i.e. any remote socket address.
* `State` shows the state of the socket. Since there are [no states in raw mode and usually no states used in UDP](sockets.md#socket-states), this column is blank for these types of sockets. Normally this can be one of several values:
    * ESTABLISHED - the socket has an established connection.
    * SYN_SENT - the socket is actively attempting to establish a connection.
    * SYN_RECV - a connection request has been received from the network.
    * FIN_WAIT1 - the socket is closed, and the connection is shutting down.
    * FIN_WAIT2 - connection is closed, and the socket is waiting for a shutdown from the remote end.
    * TIME_WAIT - the socket is waiting after close to handle packets still in the network.
    * CLOSE - the socket is not being used.
    * CLOSE_WAIT - the remote end has shut down, waiting for the socket to close.
    * LAST_ACK - the remote end has shut down, and the socket is closed. Waiting for acknowledgement.
    * LISTEN - the socket is listening for incoming connections.
    * CLOSING - both sockets are shut down but we still don't have all our data sent.

#### Filter sockets by protocol

You can narrow down the command output by specifing a type of sockets you're interested it, i.e. TCP (**-t**), UDP (**-u**), raw (**-w**), or Unix domain sockets (**-x**).

For example, the following command lists only `tcp` and `udp` sockets:

```bash
$ netstat -atu
netstat -atu
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 *:sunrpc                *:*                     LISTEN
tcp        0      0 *:ssh                   *:*                     LISTEN
tcp        0      0 *:48417                 *:*                     LISTEN
tcp        0      0 vagrant-ubuntu-trus:ssh 10.0.2.2:53938          ESTABLISHED
tcp        0      0 vagrant-ubuntu-trus:ssh 10.0.2.2:54381          ESTABLISHED
tcp6       0      0 [::]:59087              [::]:*                  LISTEN
tcp6       0      0 [::]:sunrpc             [::]:*                  LISTEN
tcp6       0      0 [::]:http               [::]:*                  LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
udp        0      0 *:sunrpc                *:*
udp        0      0 *:55170                 *:*
...
```

#### Filter sockets by state

The `-a` (`--all`) option allows to list sockets in all states. Without this option, `netstat` will exclude sockets in the `listening state` and will only show active tcp connections:

```bash
$ netstat -t
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 vagrant-ubuntu-trus:ssh 10.0.2.2:53938          ESTABLISHED
tcp        0      0 vagrant-ubuntu-trus:ssh 10.0.2.2:54381          ESTABLISHED
```

To only list sockets in the `listening state`, use the **-l** (**--listening**) option:

```
$ netstat -tul
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 *:sunrpc                *:*                     LISTEN
tcp        0      0 *:ssh                   *:*                     LISTEN
tcp        0      0 *:48417                 *:*                     LISTEN
tcp6       0      0 [::]:59087              [::]:*                  LISTEN
tcp6       0      0 [::]:sunrpc             [::]:*                  LISTEN
tcp6       0      0 [::]:http               [::]:*                  LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
udp        0      0 *:sunrpc                *:*
udp        0      0 *:55170                 *:*
...
```

Note that even though it doesn't specify `LISTEN` in the `State` column for the udp sockets because udp is a connectionless protocol, these are still sockets that listen for incoming data.

#### Use IP addresses instead of host names

By default, `netstat` converts IP address and known port number to host names and application protocols:

```bash
$ netstat -t
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 vagrant-ubuntu-trus:ssh 10.0.2.2:53938          ESTABLISHED
tcp        0      0 vagrant-ubuntu-trus:ssh 10.0.2.2:54381          ESTABLISHED
```

To display numerical addresses and port numbers use the **-n** option:

```bash
$ netstat -tn
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 10.0.2.15:22            10.0.2.2:53938          ESTABLISHED
tcp        0      0 10.0.2.15:22            10.0.2.2:54381          ESTABLISHED
```

Note that this will turn `*:*` into `0.0.0.0:*` and `[::]:*` into `:::*` for IPv6.

#### Find a process that uses this socket

Use the **-p** (**--program**) option to print the PID and name of the process that uses this socket:

```bash
$ netstat -tlpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      582/rpcbind
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1005/sshd
tcp        0      0 0.0.0.0:48417           0.0.0.0:*               LISTEN      668/rpc.statd
tcp6       0      0 :::59087                :::*                    LISTEN      668/rpc.statd
tcp6       0      0 :::111                  :::*                    LISTEN      582/rpcbind
tcp6       0      0 :::80                   :::*                    LISTEN      1128/apache2
tcp6       0      0 :::22                   :::*                    LISTEN      1005/sshd
```

This could be used along with the `grep` command to find which program is using a particular port:

```bash
$ netstat -tulpn | grep :80
tcp6       0      0 :::80                   :::*                    LISTEN      1128/apache2
```

Or find out which port a particular program is listening on:

```bash
$ netstat -tulpn | grep apache2 # filter by program name
tcp6       0      0 :::80                   :::*                    LISTEN      1128/apache2
$ netstat -tulpn | grep 1128 # filter by program PID
tcp6       0      0 :::80                   :::*                    LISTEN      1128/apache2
```

#### Continuously watch active connections

Use the [watch](watch.md) along with `netstat` to continuously monitor active connections:

```bash
$ watch -d -n0 "netstat -atnp | grep ESTA"
```

### Show routing table

The kernel routing information can be printed with the **-r** (**--route**) option. It is the same output as given by the [route](route.md) command. You can also use the `-n` option to disable the hostname lookup:

```bash
$ netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG        0 0          0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U         0 0          0 eth0
```

### Print information about network interfaces

Use the **-i** (**--interfaces**) to display information about network interfaces:

```bash
$ netstat -i
Kernel Interface table
Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
eth0       1500 0     31611      0      0 0         27503      0      0      0 BMRU
lo        65536 0      2913      0      0 0          2913      0      0      0 LRU
```

The above output contains information in a short format. To get a more human friendly version of the output use the `-e` option along with `-i`:

```bash
$ netstat -ie
Kernel Interface table
eth0      Link encap:Ethernet  HWaddr 00:16:36:f8:b2:64  
          inet addr:192.168.1.2  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::216:36ff:fef8:b264/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:31682 errors:0 dropped:0 overruns:0 frame:0
          TX packets:27573 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:29637117 (29.6 MB)  TX bytes:4590583 (4.5 MB)
          Interrupt:18 Memory:da000000-da020000 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:2921 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2921 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:305297 (305.2 KB)  TX bytes:305297 (305.2 KB)
```

### Resources used to create this document:

* https://www.binarytides.com/linux-netstat-command-examples/
* man netstat
