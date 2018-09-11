## passwd

As the name suggests **passwd** command is used to change passwords of system users.

If the `passwd` command is run by a non-root user then, it will first ask for the current password and if entered correctly, it will then ask for a new password to set:

```bash
[artemmkin@localhost ~]$ passwd
Changing password for artemmkin.
(current) UNIX password:
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

A normal user can only change the password of his account, but the `root` user can change the password of any account. To change a password for another user, run command with superuser privileges and provide user account whose password needs to be changed as an argument:

```bash
[root@localhost ~]# passwd artemmkin
Changing password for user artemmkin.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

As you can see, if run as root, the command doesn't require to type the current user password. This allows forgotten passwords to be changed.

Besides changing passwords, this command can change other password attributes such expiration time, validity, etc.

### Useful commands

Note, that most of the commands below were run under `root` account.

#### Display password status info

Using the **-S** option, you can see the current status information of a user's password:

```bash
$ passwd -S artemmkin
artemmkin PS 2018-09-11 0 90 7 -1 (Password set, MD5 crypt.)
```

The status information consists of 7 fields (it basically displays information contained in the [/etc/shadow](user-passwords.md) file but in a more readable format). The first field is the user's login name. The second field indicates if the user account has a locked password (`LK` or `L`), has no password (`NP`), or has a usable password (`PS` or `P`). The third field gives the date of the last password change. The next four fields are the minimum age, maximum age, warning period, and inactivity period for the password. These ages are expressed in days.

#### Lock/unlock user password

To lock user's password, use the **-l** option:

```bash
$ passwd -l artemmkin
Locking password for user artemmkin.
passwd: Success
$ passwd -S artemmkin
artemmkin LK 2018-09-11 0 90 7 -1 (Password locked.)
```

This option disables a password by changing it to a value which matches no possible encrypted value (it adds a `!` or `!!` at the beginning of the password):

```bash
$ fgrep artemmkin /etc/shadow
artemmkin:$1$PkF79e7k$/E89DBO97mriYiAYlJCqr0:17785:0:90:7:::
$ passwd -l artemmkin
Locking password for user artemmkin.
passwd: Success
$ fgrep artemmkin /etc/shadow
artemmkin:!!$1$PkF79e7k$/E89DBO97mriYiAYlJCqr0:17785:0:90:7:::
```

**Note 1:** this does not disable the account. The user may still be able to login using other authentication methods (e.g. an SSH key). See [usermod](usermod.md) on how to disable user account completely from logging in.

**Note 2:** users with a locked password are not allowed to change their password.

You can unlock a user password with the **-u** option:

```bash
$ passwd -u artemmkin
Unlocking password for user artemmkin.
passwd: Success
```

#### Expire password immediately

The **-e** option allows to immediately expire an account's password.

```bash
$ passwd -e artemmkin
Expiring password for user artemmkin.
passwd: Success
$ passwd -S artemmkin
artemmkin PS 1970-01-01 0 90 7 -1 (Password set, MD5 crypt.)
```

As you can see this effectively, sets the last change date to [Jan 1, 1970](https://en.wikipedia.org/wiki/Unix_time).

Expiring a password this way will force the user to change their password at the next login:

``` 
[vagrant@localhost ~]$ su artemmkin -
Password:
You are required to change your password immediately (root enforced)
Changing password for artemmkin.
(current) UNIX password:
New password:
Retype new password:
[artemmkin@localhost vagrant]$ exit
```

#### Change minimum/maximum password age

The **-n** option allows to set the _minimum number of days_ between password changes to a specified value. Note, that a value of zero for this field indicates that the user may change his/her password at any time.

The **-x** options sets the _maximum number of days_ a password remains valid. Note that the value `99999` means that the password never expires.

```bash
$ passwd -S testuser
testuser LK 2018-09-09 0 99999 7 -1 (Password locked.)
$ passwd -n 5 -x 90 testuser
Adjusting aging data for user testuser.
passwd: Success
$ passwd -S testuser
testuser LK 2018-09-09 5 90 7 -1 (Password locked.)
```

#### Change inactivity period for the account

The **-i** option is used to specify [password inactivity period](user-passwords.md).

The _password inactivity period_ is the number of days after a password has expired (i.e., the password becomes older than the maximum password age) during which the password should still be accepted (and the user should update his password at the next login). After the password has expired and the password inactivity period has passed, no login is possible using the current user's password. The user should contact her administrator.

```bash
$ passwd -i 10  testuser
Adjusting aging data for user testuser.
passwd: Success
$ passwd -S testuser
testuser LK 2018-09-09 5 90 7 10 (Password locked.)
```

### Resources used to create this document

* https://linux.101hacks.com/unix/passwd/
* https://www.linuxtechi.com/10-passwd-command-examples-in-linux/
* man passwd
