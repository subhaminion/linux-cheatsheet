## User passwords: /etc/passwd & shadow suite

### /etc/passwd

Traditionally, Linux uses the **/etc/passwd** file to keep track of every user on the system. 

Let's look at the contents of this file:

```bash
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
...
vagrant:x:1000:1000:vagrant:/home/vagrant:/bin/bash
artem:x:1001:1001:Artem Starostenko:/home/artem:/bin/bash
```

Each line in this file represents a user record; a user record contains 7 fields separated by colon (`:`).

Let's take a look at a single record to see what information it contains:

```bash
artem:x:1001:1001:Artem Starostenko:/home/artem:/bin/bash
```

1. `artem`: username.
2. `x`: placeholder for user's password.
3. `1001`: user ID (UID). Each user has a unique ID that identifies them on the system. Note that the `root` user is always referenced by user ID 0.
4. `1001`: group ID (GID). Each group has a unique group ID. Each user has a [primary group](user-groups.md) that is used as the group by default. Again, the root group's ID is always 0.
5. `Artem Starostenko`: this field is called the GECOS (General Electric Comprehensive Operating System). This entry may contain information like user's real or full name, telephone number, other contact information etc. All this information is comma separated. Most of the entries contain just the user or application name and no commas. The Shadow commands (e.g. `useradd`, `usermod`) and manual pages refer to this field as the _comment field_.
6. `/home/artem`: home directory. This is the default directory that the user ends up being right after successful login.
7. `/bin/bash`: User shell. This field contains the shell that will be spawned or the command that will be run when the user logs in. To disable user account from logging in, the value of the shell is set to [/sbin/nologin or /bin/false](https://unix.stackexchange.com/questions/10852/whats-the-difference-between-sbin-nologin-and-bin-false/10867).

The password placeholder (`x`) certainly should've left some questions. What does it mean and where the passwords are actually stored?

Passwords were traditionally stored in the `/etc/passwd` file in an encrypted format:

```bash
pete:K3xcO1Qnx8LFN:1000:1000:Peter Hernberg,,,1−800−FOOBAR:/home/pete:/bin/bash
```

The algorithm used to encrypt the password field is a **one-way hash function**. This is an algorithm that is easy to compute in one direction, but very difficult to calculate in the reverse direction.

It is computationally difficult (but not impossible) to take a randomly encrypted password and recover the original password. However, on any system with more than just a few users, at least some of the passwords will be **common words** (or simple variations of common words) that are easy to guess. The makes the system password security vulnerable to **dictionary attacks**.

A dictionary attack is conducted by choosing likely passwords from a dictionary, encrypting them, and comparing the results with the value stored in `/etc/passwd`. So if an attacker manages to obtain a copy of the password file, it is a simple matter to guess passwords, perform the encryption by a known hash function, and compare against the file.

The `/etc/passwd` file is readable by anyone on the system:

```bash
$ ls -l /etc/passwd
-rw-r--r--. 1 root root 1144 Sep  9 18:10 /etc/passwd
```

The `/etc/passwd` file also contains information like user ID and group ID that are used by many system programs and, therefore, **_this file must remain world readable_**.

If you were to change the `/etc/passwd` file so that nobody can read it, the first thing that you would notice is that the `ls -l` command now displays user IDs instead of user names:

```bash
$ ls -l /etc/passwd
-rw-r--r--. 1 root root 1144 Sep  9 18:10 /etc/passwd
$ sudo chmod o-r /etc/passwd
$ ls -l /etc/passwd
-rw-r-----. 1 0 root 1144 Sep  9 18:10 /etc/passwd
```

Notice how the last command shows the owner of the file as `0` (which is the UID of root user) instead of `root` after we removed the read permissions permissions from other users.

So if the we can't make the `/etc/passwd` file unreadable by all users, we need to move user passwords to some other protected location. And this is exactly what has been done. The passwords were moved from `/etc/passwd` to a protected [/etc/shadow](#shadow-passwords) file which is only readable by `root` user.

### Shadow passwords

Unlike the [/etc/passwd](#/etc/passwd) which should stay readable by every user on the system, the `/etc/shadow` is only readable and writeable by a root user. Therefore, by moving the passwords to the `/etc/shadow` file, we are effectively keeping the attacker from having access to the encrypted passwords with which to perform a dictionary attack.

When a system has shadow passwords enabled, the password field in `/etc/passwd` is replaced by an `x`
and the user's real encrypted password is stored in `/etc/shadow`:

```bash
$ fgrep 'artem' /etc/passwd
artem:x:1001:1001:Artem Starostenko:/home/artem:/bin/bash
$ fgrep 'artem' /etc/shadow
artem:$1$TJ3z9ew5$tTLQMlan0ug3FH5Mp0P2R1:17783:0:99999:7:::
```

Besides storing the encrypted password in a protected `/etc/shadow` file, the [Shadow password suite package](https://www.tldp.org/HOWTO/Shadow-Password-HOWTO.html#toc2) adds lots of other nice features to the system passwords management such as:

* A configuration file to set login defaults (`/etc/login.defs`)
* [Utilities for adding, modifying, and deleting user accounts and groups](https://www.tldp.org/HOWTO/Shadow-Password-HOWTO-3.html#ss3.3)
* Password aging and expiration
* Account expiration and locking
* Shadowed group passwords (optional)

Let's look at contents of `/etc/shadow` file:

```bash
$ cat /etc/shadow
root:$1$VEt.WqKC$CamA1Xh1GBbWpR1YkSJR1/::0:99999:7:::
bin:*:17632:0:99999:7:::
daemon:*:17632:0:99999:7:::
...
chrony:!!:17663::::::
vagrant:$1$xyskNOHy$fTX23HILCqtay.zv8cRaN.::0:99999:7:::
artem:$1$TJ3z9ew5$tTLQMlan0ug3FH5Mp0P2R1:17783:0:99999:7:::
testuser:!!$1$hzMb3WiK$3BwhXAfVuiNGfO4mJJd8G/:17783:0:99999:7:::
```

The format of the file is similar to [/etc/passwd](#/etc/passwd). Each line in the file represents a user record; a user record contains 9 fields separated by colon (`:`).

Let's take a look at a single record to see what information it contains:

```bash
artem:$1$TJ3z9ew5$tTLQMlan0ug3FH5Mp0P2R1:17783:0:99999:7:::
```

1. `artem`: user name.
2. `$1$TJ3z9ew5$tTLQMlan0ug3FH5Mp0P2R1`: [salt and hashed password](#password-encryption).
    * A password field which starts with a exclamation mark (`!`) or two exlamation marks (`!!`) means that this account's password is locked. The remaining characters on the line represent the password before it was locked (see `testuser` from the output above).
    * The asterisk `*` basically means the same as `!!` and indicates that the user won't be able to login with his password.
    * This field may be empty, in which case no passwords are required to authenticate as the specified login name.
3. `17783`: The date of the last password change, expressed as the number of days since [Jan 1, 1970](https://en.wikipedia.org/wiki/Unix_time).
    * An empty field means that password aging features are disabled. 
    * The value `0` has a special meaning, which is that the user should change his pasword the next time he will log in the system.
4. `0`: The minimum password age is the number of days the user will have to wait before he will be allowed to change his password again.
    * An empty field and value `0` mean that there are no minimum password age.
5. `99999`: The maximum password age is the number of days after which the user will have to change his password. After this number of days is elapsed, the password may still be valid. The user should be asked to change his password the next time he will log in.
    * An empty field means that there are no maximum password age, no password warning period, and no password inactivity period.
    * `99999` means that there is no limit to how long the current password is valid.
6. `7`: The days of warning prior to password expiration. If there is a password change requirement, this will warn the user to change their password this many days in advance.
7. `password inactivity period`: The number of days after a password has expired during which the password should still be accepted (and the user should update his password during the next login). After expiration of the password and this expiration period is elapsed, no login is possible using the current user's password. The user should contact the administrator.
    * An empty field means that there are no enforcement of an inactivity period.
8. `account expiration date`: The date of expiration of the account, expressed as the number of days since [Jan 1, 1970](https://en.wikipedia.org/wiki/Unix_time). Note that an account expiration differs from a password expiration. In case of an acount expiration, the user will not be allowed to login. In case of a password expiration, the user is not allowed to login using his password.
    * An empty field means that the account will never expire.
9. `reserved field`: This field is reserved for future use.

**NOTE:** `/etc/shadow` is only used for password authentication. If the second field shows that account's password is locked (`!!`) or has never had a password set (`*`), which means you can log in with your user name and password, you can still log in into the system using other ways, e.g. public key authentication (SSH keys) or LDAP.

#### Shadow utilities and their configuration file

The Shadow Suite contains replacement programs for: `su`, `login`, `passwd`, `newgrp`, `chfn`, `chsh`, and `id`

The package also contains the new programs: `chage`, `newusers`, `dpasswd`, `gpasswd`, `useradd`, `userdel`, `usermod`, `groupadd`, `groupdel`, `groupmod`, `groups`, `pwck`, `grpck`, `lastlog`, etc.

**/etc/login.defs** is a configuration file which provides default values for some of these utilities.

Some configuration options that we can define in this file include:

```bash
PASS_MAX_DAYS   90 # Maximum number of days a password may be used.
PASS_MIN_DAYS   0  # Minimum number of days allowed between password changes.
PASS_MIN_LEN    8  # Minimum acceptable password length.
PASS_WARN_AGE   7  # Number of days warning given before a password expires.
# If useradd should create home directories for users by default
# On RH systems, we do. This option is overridden with the -m flag on
# useradd command line.
#
CREATE_HOME     yes
# This enables userdel to remove user groups if no members exist.
#
USERGROUPS_ENAB yes
# The hash function to use for password encryption
ENCRYPT_METHOD MD5
```

Read the comments in `/etc/login.defs` file for more information.

### Password encryption

Now that we've talked about where the user passwords are stored, let's take a look at how they are stored.

As we said, Linux stores passwords in encrypted format using a one-way hash function for encryption.

A **one-way hash function** is an algorithm that is easy to compute in one direction, but very difficult to calculate in the reverse direction. This means that it is computationally easy to encrypt a password, but is difficult to take a randomly encrypted password and recover the original password from it.

A [hash](https://cromwell-intl.com/cybersecurity/crypto/hash.html) is a cryptographic function that calculates a distinctive "fingerprint" value for a piece of input data. The output of any one hash function is always of the same length, regardless of the length of the input string. And any small change in the input data causes a big change in the output. For this reason hashes are often used to determine whether or not two pieces of data are identical:

```bash
$ echo 'qwerty123' | openssl md5
(stdin)= dca675382c7f7180f7dc1cca61f4f74d
$ echo 'Qwerty123' | openssl md5
(stdin)= b93b1c8ec1e8b7210f079d8af9cef27a
```

When a new password gets assigned to a user, the system takes the typed password and uses it as a key for encrypting a block of zero bits with a one-way hash function such as `crypt`, `MD5`, or `SHA512`. The result of the encryption is saved as a user password.

_This means that the operating system has no record of what your password is._

But how does the system decrypt the password when it needs to authenticate the user if it's impossible to reverse a hash value? The answer is it doesn't.

When you try to log in, the program `/bin/login` does not decrypt the stored password. Instead, `/bin/login` takes the password that you typed, uses it to transform another block of zeros, and compares the newly transformed block with the block stored in the `/etc/passwd` file. If the two encrypted results match, the system lets you in.

You can determine what hash function was used to encrypt the password by looking at the value before the first and the second dollar sign (`$`) in the password field:

* `1`: MD5
* `2a`: Blowfish
* `5`: SHA-2-256
* `6`: SHA-2-512

For example, from the record below we can figure out that the `MD5` hash was used to encrypt that password:

```
artem:$1$/EjjGdFW$baSPAXtg7Ti.lGdmH.qeu/:17784:0:99999:7:::
```

#### Salt

A long time ago, by the late 1970s, developers of Unix had realized two things. First, you cannot prohibit a user from selecting a password already in use by some other user. Telling a user, "Sorry, someone is already using that password, try again", tells them how to authenticate as some other user. They don't know which user, but they just have to try that password once for each other user on the system and they will get in!

_Therefore, in the interest of security, you have to allow users to happen to have the same password._

So a **salt** was added to the process. A salt is a fixed-length random string. A new random salt is generated when a user sets a new password. The system then creates a string made up of the random salt and the user's typed password, and uses it a as a key for calulating a hash of a block of zero bits:

```
hash("salt, password")
```

The system then stores both the salt and the resulting hash value for that user.

At subsequent logins or other authentication events, the system asks the user to type their password. It then retrieves their stored salt, combines that with what they typed, calculates the hash of that combination, and compares that to the stored hash.

You can see the salt value for the password by looking at the value between the second and the third dollar sign (`$`) in the password field. For example, from the example below we can conclude that the salt being used for the password is `/EjjGdFW`:

```
artem:$1$/EjjGdFW$baSPAXtg7Ti.lGdmH.qeu/:17784:0:99999:7:::
```

What is going after the last dollar sign, i.e. `baSPAXtg7Ti.lGdmH.qeu/`, is the encrypted password.

### Resources used to create this document

* https://www.tldp.org/HOWTO/Shadow-Password-HOWTO.html#toc3
* https://www.oreilly.com/library/view/practical-unix-and/0596003234/ch04s03.html
* https://www.ibm.com/developerworks/community/blogs/58e72888-6340-46ac-b488-d31aa4058e9c/entry/the_linux_user_login_management_etc_passwd_and_etc_shadow_files19?lang=en
* https://www.digitalocean.com/community/tutorials/how-to-use-passwd-and-adduser-to-manage-passwords-on-a-linux-vps
* https://www.tldp.org/HOWTO/pdf/User-Authentication-HOWTO.pdf
* https://cromwell-intl.com/cybersecurity/password.html
* https://kb.iu.edu/d/aezz
* https://linux.die.net/man/5/shadow
