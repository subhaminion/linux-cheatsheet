## dig

**dig** (**domain information groper**) is a flexible tool for interrogating DNS nameservers. It performs DNS lookups and displays the answers that are returned from the name server(s) that were queried.

Let's look at the default output of the command:

```bash
$ dig example.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3292
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;example.com.			IN	A

;; ANSWER SECTION:
example.com.		4920	IN	A	93.184.216.34

;; Query time: 14 msec
;; SERVER: 10.0.2.3#53(10.0.2.3)
;; WHEN: Thu Jul 12 05:08:27 UTC 2018
;; MSG SIZE  rcvd: 56
```

Let's explain some of the important information in this output.

In the **header** section, note the printed `status`. This value tells the about the status of the query that was performed that may indicate a success or failure. The values for the `status` you'll see most of the time:

* `NOERROR` means that the DNS domain name record you were looking for was successfully found
* `NXDOMAIN` indicates that the domain name you queried doesn't exist

Also, you may find useful the `ANSWER` number in the header that shows the number of DNS records returned by the nameserver and which are listed in the answer section.

The `QUESTION SECTION` shows the DNS query that was sent to the name server. As you can see, by default `dig` asks for an [A type of record](dns.md#types-of-dns-records) for the domain name.

The `ANSWER SECTION` lists the DNS records information form the nameserver about the domain you were quering.

```
example.com.		4920	IN	A	93.184.216.34
```

Here the `4920` means the record's [TTL](dns.md) which acts as expiry data for the record and indicates the number of seconds this record will remain in cache before it will be queried again from the authoritative server. `IN` indicates the Internet protocol, `A` is a type of record and finally `93.184.216.34` is the IP address we were looking for.

The `STATISTICS SECTION` (the last part of the output) shows the time it took to perform the query, the nameserver which was contacted (this is one of servers in [/etc/resolv.conf](dns-lookup-on-linux.md)), the date of the query and the message size received from the nameserver in bytes.

Now that we've grasped the basics let's see what options often used with this command.

### Useful options

#### Query a specific nameserver

By default, the `host` command will query one of the nameserver in [/etc/resolv.conf](dns-lookup-on-linux.md) for name resolution. If you wish to query a specific nameserver (which doesn't have to be defined in `/etc/resolv.conf`), you can add its IP address (or hostname) prepended with the `@` symbol to the `dig` command. For example:

```bash
$ dig example.com @8.8.8.8

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> example.com @8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61076
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;example.com.			IN	A

;; ANSWER SECTION:
example.com.		2525	IN	A	93.184.216.34

;; Query time: 14 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Thu Jul 12 05:57:27 UTC 2018
;; MSG SIZE  rcvd: 56
```

Note the `SERVER` value in the statistics section. FYI, `#53` means DNS server port number.

#### Change command output

If you're only interested in the answer to your query and don't need all these additional information `dig` provides, use the the **+short** option:

```bash
$ dig google.com +short
172.217.6.46
```

You can include/exclude individual sections of default output:
* `+[no]cmd` turns on [turns off] the printing of the initial comment in the output identifying the version of dig and the query options that have been applied, i.e. the first two lines. There is a catch with this option. You have to specify this option before the domain name, otherwise it won't work.
* `+[no]comments` turns on [turns off] the comment lines in the output which includes of the header and the names of sections. The default is to print comments.
* `+[no]question` turns on [turns off] printing of the question section.
* `+[no]answer` turns on [turns off] the answer section of a reply. The default is to display it.
* `+[no]stats` turns on [turns off]the printing of statistics: when the query was made, the size of the reply and so on.

It's often helpful to turn off all output by using the `+noall` option and then only include the sections that you need. For example:

```bash
$ dig +noall +answer google.com
google.com.		241	IN	A	172.217.6.46
$ dig +noall +answer +question google.com
;google.com.			IN	A
google.com.		299	IN	A	172.217.6.46
```

Again, note the difference in the output when you specify the `+noall` before the domain name and after:

```bash
$ dig +noall google.com
$ dig google.com +noall

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> google.com +noall
;; global options: +cmd
```

As mentioned previously, to turn off the first two lines you have to specify `+nocmd` or `+noall` before the domain name  ¯\_(ツ)_/¯.

If you use certain dig options all the time, you may want to put them in **.digrc** file in your home directory. This way they will be applied by default to every `dig` invokation:

```bash
$ cat ~/.digrc
+noall +question +answer +stats
$ dig google.com
;google.com.			IN	A
google.com.		299	IN	A	172.217.6.46
;; Query time: 35 msec
;; SERVER: 10.0.2.3#53(10.0.2.3)
;; WHEN: Sun Jul 15 01:42:28 UTC 2018
;; MSG SIZE  rcvd: 55
```

#### Use the search domains

The **+[no]search** options allows to use [not to use] the search domain list defined by the searchlist or domain directive in resolv.conf (if any). [By default, the search domain list is not used](https://serverfault.com/questions/434581/why-can-host-and-nslookup-resolve-a-name-but-dig-cannot).

```bash
$ dig www +nostats

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> www +nostats
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 7859
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.				IN	A

;; AUTHORITY SECTION:
.			58439	IN	SOA	a.root-servers.net. nstld.verisign-grs.com. 2018071201 1800 900 604800 86400
```

As you can see in the question section, queries the `www.` domain name and gets non existent domain answer (`status: NXDOMAIN`).

```bash
$ grep 'search' /etc/resolv.conf
search example.com
$ dig www +nostats +search

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> www +nostats +search
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31015
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.example.com.		IN	A

;; ANSWER SECTION:
www.example.com.	12808	IN	A	93.184.216.34
```

With the `+search` option, `dig` uses the list of domains specified in `search` option in `/etc/resolv.conf` file to append to a non-existent domain name and try to resolve it.

#### Query a specifc DNS record type

As we could see, by default `dig` asks for `A` [type of records](dns.md#types-of-dns-records). To specify another record type, just pass the name of the type as another argument to the command:

```bash
$ dig a google.com +short
172.217.6.46
$ dig aaaa google.com +short
2607:f8b0:4005:809::200e
$ dig txt google.com +short
"docusign=05958488-4752-4ef2-95eb-aa7ba8a3bd0e"
"facebook-domain-verification=22rm551cu4k0ab0bxsw536tlds4h95"
"v=spf1 include:_spf.google.com ~all"
$ dig ns google.com +short
ns1.google.com.
ns3.google.com.
ns4.google.com.
ns2.google.com
```

#### Is the zone syncronized among all authorative nameservers?

When **+[no]nssearch** options is set, `dig` attempts to find all the authoritative nameservers for the zone containing the name being looked up and display the SOA record that each name server has for the zone.

To check if a zone is synchronized over all your authoritative nameservers, we will check [serial number](dns.md). The `serial number` is an integer number that is incremented each time a zone file gets updated.

```bash
$ dig google.com +nssearch
SOA ns1.google.com. dns-admin.google.com. 204409130 900 900 1800 60 from server 216.239.36.10 in 32 ms.
SOA ns1.google.com. dns-admin.google.com. 204409130 900 900 1800 60 from server 216.239.34.10 in 32 ms.
SOA ns1.google.com. dns-admin.google.com. 204409130 900 900 1800 60 from server 216.239.38.10 in 58 ms.
SOA ns1.google.com. dns-admin.google.com. 204409130 900 900 1800 60 from server 216.239.32.10 in 58 ms.
```

From the output we see, that the serial number (`204409130`) is the same across all the nameservers which means the zone information is in sync among the primary namerserver and all the secondaries.

### Resources used to create this document:

* http://anouar.adlani.com/2011/12/useful-dig-command-to-troubleshot-your-domains.html
* https://www.linode.com/docs/networking/dns/use-dig-to-perform-manual-dns-queries/
* https://ma.ttias.be/making-full-use-of-dig-in-linux/
* https://www.thegeekstuff.com/2012/02/dig-command-examples/
* https://www.tecmint.com/10-linux-dig-domain-information-groper-commands-to-query-dns/
