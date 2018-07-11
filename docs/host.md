## host

`host` is one of the CLI utilities that allows to perform DNS lookup which translate domain names to IP address and vice versa.

There are other commands such as [dig](dig.md) and [nslookup](nslookup) that are often used for the same purposes, but `host` command stands out among them for its minimal and user-friendly default output. For example, compare the default output of the `host` command:

```bash
$ host example.com
example.com has address 93.184.216.34
example.com has IPv6 address 2606:2800:220:1:248:1893:25c8:1946
```

and `dig` command:

```bash
$ dig example.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62201
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;example.com.			IN	A

;; ANSWER SECTION:
example.com.		4425	IN	A	93.184.216.34

;; Query time: 13 msec
;; SERVER: 10.0.2.3#53(10.0.2.3)
;; WHEN: Tue Jul 10 03:59:17 UTC 2018
;; MSG SIZE  rcvd: 56
```

I personally use [dig](dig.md) almost all the time, because it gives me a detailed information about the DNS query being performed, but `host` could often be used in cases when you don't want to bother yourself will all this additional DNS query info and just want to a get just a list of IP addresses.

NOTE: one significant difference between `dig` and `host` that I found was that `dig` [doesn't use search domains in the /etc/resolv.conf file by default](https://serverfault.com/questions/434581/why-can-host-and-nslookup-resolve-a-name-but-dig-cannot) where as the `host` command does.

Despite being simple, the `host` command can also be very powerful and provide almost the same type of detailed output about the DNS query as `dig`.

### Useful options

#### no options

Without any options, `host` translates a DNS hostname to an IP address:

```bash
$ host google.com
google.com has address 172.217.0.46
google.com has IPv6 address 2607:f8b0:4005:804::200e
google.com mail is handled by 10 aspmx.l.google.com.
google.com mail is handled by 30 alt2.aspmx.l.google.com.
google.com mail is handled by 50 alt4.aspmx.l.google.com.
google.com mail is handled by 40 alt3.aspmx.l.google.com.
google.com mail is handled by 20 alt1.aspmx.l.google.com.
```

As you can see, besides an IPv4 address, it might also show [IPv6 address and Mail Exchange (MX) records](dns.md#types-of-dns-records) if they are defined for this hostname.

If you pass an IP address instead of a hostname, `host` will try to do a **reverse DNS lookup**, i.e. translate an IP address into a hostname:

```bash
$ host 8.8.8.8
8.8.8.8.in-addr.arpa domain name pointer google-public-dns-a.google.com.
$ host google-public-dns-a.google.com
google-public-dns-a.google.com has address 8.8.8.8
google-public-dns-a.google.com has IPv6 address 2001:4860:4860::8888
```

#### Query a specific nameserver

By default, the `host` command will query one of the nameserver in `/etc/resolv.conf` for name resolution. If you wish to query a specific nameserver (which doesn't have to be defined in `/etc/resolv.conf`), you can pass its IP address (or hostname) as a second argument. For example:

```bash
$ host example.com 8.8.8.8
Using domain server:
Name: 8.8.8.8
Address: 8.8.8.8#53
Aliases:

example.com has address 93.184.216.34
example.com has IPv6 address 2606:2800:220:1:248:1893:25c8:1946

$ host example.com google-public-dns-a.google.com
Using domain server:
Name: google-public-dns-a.google.com
Address: 8.8.8.8#53
Aliases:

example.com has address 93.184.216.34
example.com has IPv6 address 2606:2800:220:1:248:1893:25c8:1946
```

#### Specify DNS record type

As you could see, by default `host` queries [A, AAAA and MX types of records](dns.md#types-of-dns-records) for the domain. To specify a particular type of record to query, use the **-t** option:

```bash
$ host -t a google.com
google.com has address 172.217.0.46
$ host -t aaaa google.com
google.com has IPv6 address 2607:f8b0:4005:804::200e
$ host -t txt google.com
google.com descriptive text "v=spf1 include:_spf.google.com ~all"
google.com descriptive text "docusign=05958488-4752-4ef2-95eb-aa7ba8a3bd0e"
google.com descriptive text "facebook-domain-verification=22rm551cu4k0ab0bxsw536tlds4h95"
$ host -t ns google.com
google.com name server ns2.google.com.
google.com name server ns3.google.com.
google.com name server ns1.google.com.
google.com name server ns4.google.com.
```

#### Query SOA record for the zone

You can make `host` attempt to query the [SOA records](dns.md#zones) from all the listed authoritative nameservers for that zone with the **-C** option:


```bash
$ host -C google.com
Nameserver 216.239.36.10:
	google.com has SOA record ns1.google.com. dns-admin.google.com. 203854003 900 900 1800 60
Nameserver 216.239.34.10:
	google.com has SOA record ns1.google.com. dns-admin.google.com. 203880072 900 900 1800 60
Nameserver 216.239.32.10:
	google.com has SOA record ns1.google.com. dns-admin.google.com. 203880072 900 900 1800 60
Nameserver 216.239.38.10:
	google.com has SOA record ns1.google.com. dns-admin.google.com. 203854003 900 900 1800 60

```

#### Query all records for the domain

Use the **-a** (all) option to make a query of type ANY, i.e. to ask for records of all types:

```bash
$ host -a google.com
Trying "google.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25366
;; flags: qr rd ra; QUERY: 1, ANSWER: 16, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.			IN	ANY

;; ANSWER SECTION:
google.com.		299	IN	A	172.217.6.78
google.com.		299	IN	AAAA	2607:f8b0:4005:80a::200e
google.com.		21599	IN	NS	ns2.google.com.
google.com.		21599	IN	NS	ns1.google.com.
google.com.		599	IN	MX	40 alt3.aspmx.l.google.com.
google.com.		3599	IN	TXT	"facebook-domain-verification=22rm551cu4k0ab0bxsw536tlds4h95"
google.com.		599	IN	MX	20 alt1.aspmx.l.google.com.
google.com.		299	IN	TXT	"docusign=05958488-4752-4ef2-95eb-aa7ba8a3bd0e"
google.com.		21599	IN	NS	ns4.google.com.
google.com.		599	IN	MX	30 alt2.aspmx.l.google.com.
google.com.		599	IN	MX	50 alt4.aspmx.l.google.com.
google.com.		599	IN	MX	10 aspmx.l.google.com.
google.com.		59	IN	SOA	ns1.google.com. dns-admin.google.com. 204035978 900 900 1800 60
google.com.		21599	IN	NS	ns3.google.com.
google.com.		21599	IN	CAA	0 issue "pki.goog"
google.com.		3599	IN	TXT	"v=spf1 include:_spf.google.com ~all"

Received 503 bytes from 10.0.2.3#53 in 33 ms
```

As you can see, with this option `host` gives the output which looks similar to what `dig` command provides.

Important parts of this output:

* The `QUESTION SECTION` shows what has beed asked from a nameserver. In this case, we asked the namerserver for Internet Protocol (IN) records of `ANY` type.
* The `ANSWER SECTION` shows the answer received from the nameserver.
* The last line, i.e. `Received 503 bytes from 10.0.2.3#53 in 33 ms`, shows how much data was received from the nameserver, the IP address of the nameserver and its port, plus how long the query took (34 miliseconds).
* Also, note that the integer number that records include right after the domain name part (e.g. `299`, `599`) indicates the record's [TTL](dns.md)

#### Verbose output

To get the same type of verbose output as with we've seen in the case of `-a` option, but for some specific type of record, use the **-v** flag along with the **-t** option:

```bash
$ host -v -t a google.com
Trying "google.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55912
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		299	IN	A	216.58.195.78

Received 44 bytes from 10.0.2.3#53 in 32 ms
```

### Resources used to create this document

* https://www.tecmint.com/linux-host-command-examples-for-querying-dns-lookups/
* https://serverfault.com/questions/501168/whats-the-difference-between-dig-and-host-when-querying-a-specific-name-ser
* man host
