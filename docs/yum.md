## yum

Also, see the [yum command chetsheat](https://access.redhat.com/sites/default/files/attachments/rh_yum_cheatsheet_1214_jcs_print-1.pdf).

#### Check information about a package

If the package is not installed, this will display information about the latest `available` version of the package:

```bash
# syntax: yum info <package_name>[-<version_number>][-<build_number>]
$ yum info httpd
Loading mirror speeds from cached hostfile
 * base: mirrors.sonic.net
 * extras: mirrors.tummy.com
 * updates: linux.mirrors.es.net
Available Packages
Name        : httpd
Arch        : x86_64
Version     : 2.4.6
Release     : 80.el7.centos
Size        : 2.7 M
Repo        : base/7/x86_64
Summary     : Apache HTTP Server
URL         : http://httpd.apache.org/
License     : ASL 2.0
Description : The Apache HTTP Server is a powerful, efficient, and extensible
            : web server.
```

If the version of the installed package is lower than the latest available version, it will show information about the installed version and the latest available version of the package:

```bash
Installed Packages
Name        : influxdb
Arch        : x86_64
Version     : 1.4.2
Release     : 1
Size        : 63 M
Repo        : installed
From repo   : influxdb
Summary     : Distributed time-series database.
URL         : https://influxdata.com
License     : Proprietary
Description : Distributed time-series database.

Available Packages
Name        : influxdb
Arch        : x86_64
Version     : 1.5.2
Release     : 1
Size        : 21 M
Repo        : influxdb/7/x86_64
Summary     : Distributed time-series database.
URL         : https://influxdata.com
License     : Proprietary
Description : Distributed time-series database.
```

If the installed version is the latest available in the repo, you will only see information about the `installed` packages.

#### List versions of the package available in the repo

```bash
# syntax: yum list --shoduplicates <package_name>
$ yum list --showduplicates influxdb
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.sonic.net
 * epel: mirrors.kernel.org
 * extras: mirrors.tummy.com
 * updates: linux.mirrors.es.net
Available Packages

...
influxdb.x86_64       1.3.5-1       influxdb
influxdb.x86_64       1.3.7-1       influxdb
influxdb.x86_64       1.4.2-1       influxdb
influxdb.x86_64       1.5.0-1       influxdb
influxdb.x86_64       1.5.1-1       influxdb
influxdb.x86_64       1.5.2-1       influxdb
```

where the columns of the output have the following format:

* the first column: `<package_name>.<architecture>`
* the second column: `<version_number>–<build_number>`
* the third column shows the name of the repository in which the package is available.

#### Install a package

To install the latest version of the package simple pass its name to the `install` command:

```bash
$ yum install influxdb
```

To install a specific version of the package, specify its `name` and the `version number` separated by a dash (`-`). In addition to that, you can also specify a `build number` if multple builds are available for the same version:

```bash
# syntax: yum install <package name>[-<version_number>][-<build_number>]
$ yum install influxdb-1.4.2
```

#### Install an RPM package located locally

If no pa
```bash
# syntax: yum localinstall <path-to-rpm-package>
$ yum localinstall pkg-1-1.i686.rpm
```

#### Check if a package is installed

```bash
# syntax: yum list installed <package_name>
$ yum list installed influxdb
Installed Packages
influxdb.x86_64              1.4.2-1           @influxdb
```

This shows the version of the installed package and the name (`influxdb`) of the repository from which the package was installed.

___

Also, see [yum info](#Check-information-about-a-package)

#### Update packages

Update a package to the latest version:

```bash
# syntax: yum update <package_name(s)>
$ yum update influxdb
```

Sometimes when updating packages you might get an error like this:

```bash
$ yum update influxdb
...
No Packages marked for Update
```

In this case, try to [clear/invalidate the cache](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-working_with_yum_cache) before trying an update:

```bash
$ yum clean expire-cache
$ yum update influxdb
```
___

To update a package to specific version, specify the `version number` (and optionally the `build number`).

```bash
$ yum update-to influxdb-1.5.1
```
___

If no package name(s) is specified, yum will update all the packages:

```bash
$ yum update
```

#### Downgrade a package version

To downgrade to the previous version:

```bash
$ yum downgrade influxdb
```

To downgrade to a specific earlier version:

```bash
$ yum downgrade influxdb-1.3.5
```

### Resources used to create this document:

* https://www.systutorials.com/2346/installing-specific-old-versions-of-packages-in-yum/
* https://kerneltalks.com/howto/how-to-list-yum-repositories-in-rhel-centos/
* https://www.tecmint.com/install-particular-package-version-in-centos-ubuntu-debian/
