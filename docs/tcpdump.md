## tcpdump

**tcpdump** is one of the best network troubleshooting tools. It allows to capture and analize the network traffic on the system and is available on almost all Linux/Unix distros.

Note that it requires `root priviledges` to use `tcpdump`.

The command has the following syntax:

```
$ sudo tcpdump [-options] [filter expression]
```

### Useful options

#### Capture network packets from a specific interface

To get a list of interfaces that tcpdump can capture packets from, use the `-D` option:

```bash
$ sudo tcpdump -D
1.eth0
2.nflog (Linux netfilter log (NFLOG) interface)
3.nfqueue (Linux netfilter queue (NFQUEUE) interface)
4.any (Pseudo-device that captures on all interfaces)
5.lo [Loopback]
```

To capture network packet from a specific interface from that list, use the `-i` option:

```bash
$ sudo tcpdump -i eth0
```

This will show all the network traffic that is going through the `eth0` interface, which could be overwhelming to look at, so keep reading and see how to apply filters below.

To capture traffic from all interfaces, use the special interface name `any`:

```bash
$ sudo tcpdump -i any
```

#### Restrict a number of packets to capture

Specify how many network packets to capture by using the `-c` option followed by the number of packets.

For example, the following command will capture and display only 2 packets:

```bash
$ sudo tcpdump -i any -c 2
```

#### Display IPs instead of hostnames

By default `tcpdump` tries to resolve IP address to hostnames:

```bash
$ ping 127.0.0.1 > /dev/null & # start a background process that pings localhost
$ sudo tcpdump -i lo -c 2
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
06:19:50.740860 IP localhost > localhost: ICMP echo request, id 2075, seq 20, length 64
06:19:50.740870 IP localhost > localhost: ICMP echo reply, id 2075, seq 20, length 64
```

In this example, the IP address `127.0.0.1` is resolved to `localhost` by tcpdump. To disable the conversion of IP addresses to hostnames, use the `-n` option:

```bash
$ sudo tcpdump -i lo -c 2 -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
06:22:39.740379 IP 127.0.0.1 > 127.0.0.1: ICMP echo request, id 2075, seq 189, length 64
06:22:39.740394 IP 127.0.0.1 > 127.0.0.1: ICMP echo reply, id 2075, seq 189, length 64
```

#### Hide timestamp

Use the `-t` option to disable displaying the timestamp:

```bash
$ sudo tcpdump -i lo -c 2 -t
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
IP localhost > localhost: ICMP echo request, id 2364, seq 1, length 64
IP localhost > localhost: ICMP echo reply, id 2364, seq 1, length 64
```

#### Display ASCII/hexadecimal output

Adding `-A` to the command line will have the output include the `ascii` strings from the capture. This allows easy reading and the ability to [parse the output using grep or other commands](https://hackertarget.com/tcpdump-examples/):

```bash
$ sudo tcpdump -i eth0 -A
```

Another option that shows both hexadecimal output and ASCII is the `-X` option.

#### Writing and reading from a file

To save packet captures into a file for future analysis, use the `-w` option:

```bash
$ sudo tcpdump -i eth0 capture-file
```

To read from this file, use the `-r` option. Note that you can use all the regular commands within tcpdump while reading in a file; you’re only limited by the fact that you can’t capture and process what doesn’t exist in the file already:

```bash
$ sudo tcpdump -r capture_file
```

### Packet filtering

`filter expressions` allow you to filter the caputered traffic and display only what you're interested in.

Fields of a captured network packet can be used as a filter parameter. Tcpdump has the following expression types:

* `dir`: The `dir` stands for direction. It allows to filter packets by their value in the source or destination fields. The allowed keywords are `src`, `dst`, `src or dst`, and `src and dst`. If no direction is given, `src or dst` is assumed.

* `type`: This allows you to filter traffic based on the addressing/ports within the packet. The allowed keyword are `host`, `net`, and `port`. Without any keywords, `host` is assumed.

* `proto`: Restricts the capture of packets to a particular protocol. Some of the allowed protocols are `ip`,`icmp`, `tcp` and `udp`. If no protocol is given, all protocols matching the above type and direction are displayed/captured.

Also, Boolean operators such as `not`(`!`), `and` (`&&`) and `or` (`||`) can be used to combine filter expressions.

Let's see some common examples.

#### Filter traffic by IP

The following will filter packets by the `1.2.3.4` IP address:

```bash
$ sudo tcpdump -i eth0 host 1.2.3.4
```

This will show all the packets going through the eth0 interface that have `1.2.3.4` IP address either as the source or the destination address. As was mentioned previously, `src or dst` expression is assumed to prefix the `type` expression by default, so these three commands are equivalent:

```bash
$ sudo tcpdump -i eth0 host 1.2.3.4
$ sudo tcpdump -i eth0 src or dst host 1.2.3.4
$ sudo tcpdump -i eth0 src or dst 1.2.3.4 # host is assumed by default if not given
```

Alternatively capture only packets going one way using `src` or `dst`:

```bash
$ sudo tcpdump -i eth0 dst 1.2.3.0
$ sudo tcpdump -i eth0 src 1.2.3.0
```

#### Filtering packets by network address

To find packets going to or from a particular network, use the `net` expression:

```bash
$ sudo tcpdump -i eth0 net 1.2.3.0/24
```

This will show all the packets going through the eth0 interface that have an IP address from this network block either as the source or the destination address. As was mentioned previously, `src or dst` expression is assumed to prefix the `type` expression by default, so these two commands are equivalent:

```bash
$ sudo tcpdump -i eth0 net 1.2.3.0/24
$ sudo tcpdump -i eth0 src or dst net 1.2.3.0/24
```

Alternatively capture only packets going one way using `src` or `dst`:

```bash
$ sudo tcpdump -i eth0 dst net 1.2.3.0/24
$ sudo tcpdump -i eth0 src net 1.2.3.0/24
```

#### Filter packets by port number

The `port` expression allows to filter packets by the port number information:

```bash
$ sudo tcpdump -i eth0 port 53
```

This will show all the packets going through the eth0 interface that have the port `53` either in the source or the destination address. As was mentioned previously, `src or dst` expression is assumed to prefix the `type` expression by default, so these two commands are equivalent:

```bash
$ sudo tcpdump -i eth0 port 53
$ sudo tcpdump -i eth0 src or dst port 53
```

Alternatively capture only packets going one way using `src` or `dst`:

```bash
$ sudo tcpdump -i eth0 dst port 53
$ sudo tcpdump -i eth0 src port 53
```

#### Filter traffic by port range

```bash
$ sudo tcpdump portrange 21-23
```

#### Filter traffic by protocol

If you're looking for one particular kind of traffic, you can use `tcp`, `udp`, `icmp`, and many others:

```bash
$ sudo tcpdump icmp
```

#### Combining the expression

Boolean operators such as `not`, `and` and `or` can be used to combine filter expressions.

For example, the following command catches all SSH packets going from an SSH server to the IP `1.2.3.4`:

```bash
$ sudo tcpdump "src port 22" and "dst host 1.2.3.4"
```

Keep in mind that when you’re building complex queries you might have to group your options using single quotes. Single quotes are used in order to tell tcpdump to ignore certain special characters, in this case the `()` brackets:

```bash
$ sudo tcpdump 'src 10.0.2.4 and (dst port 3389 or 22)'
```

Or you can escape the brackets:

```bash
$ sudo tcpdump src 10.0.2.4 and \(dst port 3389 or 22\)
```

### Useful examples

[On this page](https://hackertarget.com/tcpdump-examples/) you can find some practical examples of using the `tcpdump` command.

### Resources used to create this document:

* https://support.rackspace.com/how-to/capturing-packets-with-tcpdump/
* http://science.hamptonu.edu/compsci/docs/iac/packet_sniffing.pdf
* https://www.hugeserver.com/kb/install-use-tcpdump-capture-packets/
* https://hackertarget.com/tcpdump-examples/
* https://www.slashroot.in/packet-capturing-tcpdump-command-linux
