<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Learning the Essentials of CentOS Linux](#learning-the-essentials-of-centos-linux)
  - [Intro](#intro)
    - [Setup](#setup)
    - [Listing Files](#listing-files)
    - [File Types](#file-types)
    - [Working with Files](#working-with-files)
    - [Working with Directories](#working-with-directories)
    - [Working with Links](#working-with-links)
  - [Reading Files](#reading-files)
    - [Reading from Files](#reading-from-files)
    - [Regular Expressions and grep](#regular-expressions-and-grep)
    - [Using sed to Edit Files](#using-sed-to-edit-files)
    - [Comparing Files](#comparing-files)
    - [Finding Files](#finding-files)
  - [Using the vim Text Editor](#using-the-vim-text-editor)
    - [Creating and Editing Files](#creating-and-editing-files)
    - [Using the nano Text Editor](#using-the-nano-text-editor)
    - [Learning vim with vimtutor](#learning-vim-with-vimtutor)
    - [Setting Defaults with .vimrc](#setting-defaults-with-vimrc)
    - [Editing Files with vim](#editing-files-with-vim)
  - [Piping and Redirection](#piping-and-redirection)
    - [Redirecting STDOUT](#redirecting-stdout)
    - [Using the noclobber Option](#using-the-noclobber-option)
    - [Redirecting STDERR](#redirecting-stderr)
    - [Reading into STDIN](#reading-into-stdin)
    - [Using HERE documents](#using-here-documents)
    - [Command Pipelines](#command-pipelines)
    - [Named Pipes](#named-pipes)
    - [Using the Command tee](#using-the-command-tee)
  - [Archiving Files](#archiving-files)
    - [Using the tar Command](#using-the-tar-command)
    - [Using Compression](#using-compression)
  - [Understanding File Permissions](#understanding-file-permissions)
    - [Listing Permissions](#listing-permissions)
    - [Managing Default Permissions](#managing-default-permissions)
    - [Setting Permissions](#setting-permissions)
    - [Managing File Ownership](#managing-file-ownership)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Learning the Essentials of CentOS Linux

> My notes from this [Pluralsight course](https://app.pluralsight.com/library/courses/lfcs-red-hat-7-essentials/table-of-contents)

## Intro

### Setup

My simple version with [Docker](https://hub.docker.com/_/centos):

```shell
$ docker pull centos:7
$ docker run --name centoscourse -d centos:7 tail -f /dev/null
$ docker exec -it centoscourse bash

# additional packages needed
$ yum install redhat-lsb-core # not sure if this is required
$ yum install which
$ yum install tree
$ yum install vim
$ yum install sudo

# add a non-root user
$ adduser course
$ passwod course # set and confirm at least 8 char password
$ gpasswd -a username wheel

# attempt mail -> not working
$ vim /etc/postfix/main.cf
# inet_interfaces = 127.0.0.1, 172.17.0.2
$ mkfifo /var/spool/postfix/public/pickup
```

Next time when container already exists:

```shell
$ docker start centoscourse
$ docker exec -it centoscourse bash
```

### Listing Files

- `pwd` print working directory
- `ls` list files, defaults to with color coding
- `type ls` -> aliased to `ls --color=auto` (alias created with login script)
- `ls -a` list all including hidden files
- `ls -aF` show directories with forward slash at the end
- hidden files/dirs begin with `.`
- Ctrl + L: clear screen
- `ls /etc` - list server's configuration directory (note some files are symbolic links)
- `ls -aF /etc` - notice symbolic link files end in `@`
- `ls -l` - long listing - shows permissions, file size, ownership, and modified timestamp
- `ls -lrt /etc` - `t` sort by modified timestamp, `r` reverse sort - show most recently modified files first
- file: `/etc/resolv.conf` - name resolution - changes whenever networking starts
- `ls -lhrt /etc` - `h` display file size in human readable form
- `ls -ld /etc` - `d` option specifies just list the `etc` dir itself, NOT its contents

**Breakdown of ls output from left to right**

```shell
$ ls -ld /etc
drwxr-xr-x 1 root root 4096 Dec 15 19:54 /etc
```

- `d`: file type, in this case, directory
- `rwxr-xr-x`: permission block
- `1`: number of hard links to this directory
- `root`: user owner
- `root`: group owner
- `4096`: file size
- `Dec 15 19:54`: last modified date/time

### File Types

Everything in Linux is a file of some type, even devices.

Get the terminal then inspect what file type it is - notice `c` for *character device* - accepts character input and displays them.

```shell
$ tty
/dev/pts/0
ls -l /dev/pts/0
crw--w---- 1 root tty 136, 0 Dec 17 21:11 /dev/pts/0
```

Shortcut to do above - use brackets for evaluating, which runs first, then output of that is passed to `ls -l`:

```shell
$ ls -l $(tty)
crw--w---- 1 root tty 136, 0 Dec 17 21:11 /dev/pts/0
```

To view block devices - disks and partitions, eg: `sda` disk and `sda1` partition:

```shell
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr2     11:2    1   832M  0 rom
sr0     11:0    1 453.4M  0 rom
sda      8:0    0  59.6G  0 disk
`-sda1   8:1    0  59.6G  0 part /etc/hosts
sr1     11:1    1    92K  0 rom
```

Show files starting with `s` and anything that follows, including 0 characters.

`*` character is wildcard - 0 or more characters

`?` - exactly 1 character

```shell
$ ls -l sys/block/s*
lrwxrwxrwx 1 root root 0 Dec 17 21:26 /sys/block/sda -> ../devices/pci0000:00/0000:00:02.0/ata1/host0/target0:0:0/0:0:0:0
/block/sda
lrwxrwxrwx 1 root root 0 Dec 17 21:26 /sys/block/sr0 -> ../devices/pci0000:00/0000:00:04.0/ata7/host6/target6:0:0/6:0:0:0
/block/sr0
lrwxrwxrwx 1 root root 0 Dec 17 21:26 /sys/block/sr1 -> ../devices/pci0000:00/0000:00:05.0/ata13/host12/target12:0:0/12:0
:0:0/block/sr1
lrwxrwxrwx 1 root root 0 Dec 17 21:26 /sys/block/sr2 -> ../devices/pci0000:00/0000:00:06.0/ata19/host18/target18:0:0/18:0
:0:0/block/sr2

$ ls -l /sys/block/sr?
lrwxrwxrwx 1 root root 0 Dec 17 21:26 /sys/block/sr0 -> ../devices/pci0000:00/0000:00:04.0/ata7/host6/target6:0:0/6:0:0:0
/block/sr0
lrwxrwxrwx 1 root root 0 Dec 17 21:26 /sys/block/sr1 -> ../devices/pci0000:00/0000:00:05.0/ata13/host12/target12:0:0/12:0
:0:0/block/sr1
lrwxrwxrwx 1 root root 0 Dec 17 21:26 /sys/block/sr2 -> ../devices/pci0000:00/0000:00:06.0/ata19/host18/target18:0:0/18:0
:0:0/block/sr2
```

Look for `sr1` or `sr2`:

```shell
$ ls -l /sys/block/sr[12]
lrwxrwxrwx 1 root root 0 Dec 17 21:26 /sys/block/sr1 -> ../devices/pci0000:00/0000:00:05.0/ata13/host12/target12:0:0/12:0
:0:0/block/sr1
lrwxrwxrwx 1 root root 0 Dec 17 21:26 /sys/block/sr2 -> ../devices/pci0000:00/0000:00:06.0/ata19/host18/target18:0:0/18:0
:0:0/block/sr2
```

List multiple files at once - notice first one has leftmost `-` -> real file

Symbolic link files show *where* they're pointing to.

```shell
$ ls -l /etc/system-release /etc/centos-release /etc/redhat-release
-rw-r--r-- 1 root root 37 Sep  5 13:05 /etc/centos-release
lrwxrwxrwx 1 root root 14 Oct  1 01:15 /etc/redhat-release -> centos-release
lrwxrwxrwx 1 root root 14 Oct  1 01:15 /etc/system-release -> centos-release
```

No matter which file you look at, get the same results:

```shell
$ cat /etc/centos-release
CentOS Linux release 7.7.1908 (Core)
$ cat /etc/redhat-release
CentOS Linux release 7.7.1908 (Core)
$ cat /etc/system-release
CentOS Linux release 7.7.1908 (Core)
```

Can get same info with lsb command:

```shell
$ lsb_release -d
Description:    CentOS Linux release 7.7.1908 (Core)
```

`lsb_release` is a binary file

`which` command shows full path to file, `*` at end (when using `-F` flag) shows it's executable

`usr` - unix system resources

```shell
$ ls -lF $(which lsb_release)
-rwxr-xr-x 1 root root 15929 Mar 27  2015 /usr/bin/lsb_release*
```

To identify which package installed `lsb_release`, use `rpm` command, which is a package manager for CentOS.

rpm is used to install packages, but also adds them to database which can be queried.

`-qf`: query file

```shell
$ rpm -qf /usr/bin/lsb_release
redhat-lsb-core-4.1-27.el7.centos.1.x86_64
```

Shorthand:

```shell
$ rpm -qf $(which lsb_release)
redhat-lsb-core-4.1-27.el7.centos.1.x86_64
```

### Working with Files

`cp` copy file - need read permission through to file to be copied, and write permission through to file to be copied to.

`.` - copy file to current directory

```shell
$ mkdir documents
$ cd documents
$ cp /etc/hosts .
$ ls
hosts
cat hosts
27.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
```

Can also specify the target file name: (passwod is user account database file for the system)

```shell
$ cp /etc/passwd ./passwd
$ ls
hosts passwod
```

Can copy file to a different name and overwrite a file that's already in target dir:

```shell
$ cp /etc/hosts ./passwod
# now ./passwwd contains contents of /etc/hosts
```

To prevent this, use `-i` for interactive mode, if file exists, will be prompted to overwrite, can enter `n` or `y`:

```shell
$ cp -i /etc/hosts ./passwd
cp: overwrite './passwd'? n
$ cp -i /etc/passwd .
cp: overwrite './passwd'? y
```

`mv` - move files to different directory and/or rename file

```shell
$ mv hosts localhosts
$ ls
localhosts  passwd
$ cp localhosts hosts
$ ls
hosts  localhosts  passwd
```

Can also move to different directory:

```shell
$ mkdir otherdir
$ mv localhosts otherdir
$ ls otherdir
localhosts
```

`-i` switch can also be used with `mv` command to get warning if about to overwrite.

`cp *` to copy all files:

```shell
$ cp * otherdir
$ ls otherdir
hosts  localhosts  passwd
```

`rm` to remove file, with `-i` option to be asked for confirmation before each file removal:

```shell
$ rm -i *
rm: remove regular file 'hosts'? y
rm: cannot remove 'otherdir': Is a directory
rm: remove regular file 'passwd'? y
```

`rm *hosts` - remove any file that ends in `hosts`, will also match file named exactly `hosts` - recall `*` means *zero or more characters*.

### Working with Directories

`mkdir` - make a new directory.

`-p` flag to create directories in the path.

`rmdir` - remove directory.

`!rm` - run last command that began with `rm`, doing reverse search through your command history.

`rm -rf` - recursively delete non-empty dir and all its contents, forcing deletion.

`mkdir one two` - create directories `one` and `two` at the same level

`touch one/file{1..5}` - create 5 fives in `one` dir named `file`, `file2`, ..., `file5`

`cp -R one two` - recursively copy dir `one` and all its contents to dir `two`

`mkdir -m` - to set permissions (discussed later)

`777`: all permissions - read/write/execute for user, group and others

`700`: private for user - read/write/execute for user, nothing for group, nothing for others



Start with empty `/documents` dir:

```shell
$ mkdir test
$ ls
test
$ mkdir sales/test
mkdir: cannot create directory 'sales/test': No such file or directory
$ mkdir -p sales/test
$ ls
sales test
$ ls sales
test
$ rmdir test # works fine because test is empty dir
$ rmdir sales
rmdir: failed to remove 'sales': Directory not empty
$ !rm
rmdir sales
rmdir: failed to remove 'sales': Directory not empty
$ rm -rf sales
$ mkdir one two
$ ls
one two
$ touch one/file{1..5}
$ ls one
file1  file2  file3  file4  file5
$ cp -R one two
$ ls two
one
$ tree two
two
`-- one
    |-- file1
    |-- file2
    |-- file3
    |-- file4
    |-- file5
$ rm -rf two
$ mkdir -m 777 d1
$ mkdir -m 700 d2
$ ls -ldh d1 d2
drwxrwxrwx 2 root root 4.0K Dec 23 23:15 d1
drwx------ 2 root root 4.0K Dec 23 23:17 d2
```

### Working with Links

`-i` flag for `ls` shows *inode* number (note: in course, instructor's hard link count is 110 but in docker container, its 1 for /etc):

```shell
$ ls -ldi /etc
16548 drwxr-xr-x 1 root root 4096 Dec 17 21:18 /etc
```

`inode` - in above example, it's `16548` - file/directory entry, metadata.

Notice `/etc/.` "file" is linked to same metadata as `/etc`:

```shell
$ ls -ldi /etc/.
16548 drwxr-xr-x 1 root root 4096 Dec 17 21:18 /etc/.
```

Whenever you create a directory, there will always be a dot `.` file contained in that dir, which represents the current directory. So minimum hard link count for a dir will be 2 - two names, eg: `etc` and `.` pointing to the one set of metadata in inode.

`..` - file linked to parent directory

Start from empty `/documents` dir:

`-a` flag for `ls` shows hidden files, those starting with `.`

```shell
$ mkdir d1
$ ls -ld t1 # notice hard link count of 2
drwxr-xr-x 2 root root 4096 Dec 24 12:56 t1
$ ls -l t1 # nothing
total 0
$ ls -la t1 # show hidden files
drwxr-xr-x 2 root root 4096 Dec 24 12:56 .
drwxr-xr-x 3 root root 4096 Dec 24 12:56 ..
```

- `.` is linked through to `t1` directory metadata
- `..` is linked through to parent directory metadata, in this case, `/documents`

To verify, check inodes -> same!

```shell
$ ls -ldi t1 t1/.
17709 drwxr-xr-x 2 root root 4096 Dec 24 12:56 t1
17709 drwxr-xr-x 2 root root 4096 Dec 24 12:56 t1/.
```

Check effect on hard link count when create subdirs:

```shell
$ mkdir t1/s1
$ ls -ldi t1 # hard link count has increased to 3
17709 drwxr-xr-x 3 root root 4096 Dec 24 13:04 t1
```

Hard link count increases because subdir `s1` contains a file `..` that points to parent dir `t1`.

Creating another subdir bumps up hard link count again:

```shell
$ mkdir t1/s2
$ ls -ldi t1 # hard link count has increased to 4
17709 drwxr-xr-x 4 root root 4096 Dec 24 13:06 t1
```

Hard links are limited to the same file system.

Create a simple text file and hard link to it - file names `f1` and `f2` are linked to the same metadata

```shell
$ echo hello > f1
$ cat f1
hello
$ ls -l f1
-rw-r--r-- 1 root root 6 Dec 24 15:10 f1
$ ln f1 f2 # create a hard link f2, pointing to original f1
$ ls -li f1 f2 # both now have hard link count o f2 and the SAME inode number, i.e. they are pointing through to the same file
2494125 -rw-r--r-- 2 root root 6 Dec 24 15:10 f1
2494125 -rw-r--r-- 2 root root 6 Dec 24 15:10 f2
$ cat f1
hello
$ cat f2
hello
```

Compare with *symbolic* link:

```shell
$ ln -s f1 f3
$ ls -li f1 f2 f3 # Note first two are regular files but third has different inode num, type link, hard link count of 1
2494125 -rw-r--r-- 2 root root 6 Dec 24 15:10 f1
2494125 -rw-r--r-- 2 root root 6 Dec 24 15:10 f2
2494200 lrwxrwxrwx 1 root root 2 Dec 24 15:15 f3 -> f1
$ cat f3
hello
```

Benefit of symbolic link: Can cross system boundary such as different file system, but hard link fails:

`df -h` - disk file system, showing disk space statistics in human readable format.

Shows device name, size, disk space used, disk space available and mount point on file system.

```shell
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay          59G  1.5G   54G   3% /
tmpfs            64M     0   64M   0% /dev
tmpfs          1000M     0 1000M   0% /sys/fs/cgroup
shm              64M     0   64M   0% /dev/shm
/dev/sda1        59G  1.5G   54G   3% /etc/hosts
tmpfs          1000M     0 1000M   0% /proc/acpi
tmpfs          1000M     0 1000M   0% /sys/firmware
$ ln /dev/core .
ln: failed to create hard link './core' => '/dev/core': Invalid cross-device link
```

## Reading Files

### Reading from Files

`cat` - To read from a small file, cat === concatenate. Good for small file, but for longer file content will scroll off screen.

```shell
$ cat /etc/hosts
27.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      4126a0b299d3
$ cat /etc/hosts /etc/hostname # literally concatenate two files together
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      4126a0b299d3
4126a0b299d3
```

`wc -l` - check how many lines in a file: (word count with -l option counts lines)

`less` - to page through file one screen-ful at a time

(`/etc/services` file maps port names to port numbers)

Also note, `!$` will get expanded to last argument, eg:

```shell
$ cat /etc/services
...
$ less !$ # === less /etc/services
```

```shell
$ wc -l /etc/services
11176 /etc/services
$ less /etc/services
# /etc/services:
# $Id: services,v 1.55 2013/04/14 ovasik Exp $
#
# Network services, Internet style
# IANA services version: last updated 2013-04-10
#
# Note that it is presently the policy of IANA to assign a single well-known
# port number for both TCP and UDP; hence, most entries here have two entries
# even if the protocol doesn't support UDP operations.
# Updated from RFC 1700, ``Assigned Numbers'' (October 1994).  Not all ports
# are included, only the more common ones.
#
# The latest IANA port assignments can be gotten from
#       http://www.iana.org/assignments/port-numbers
# The Well Known Ports are those from 0 through 1023.
# The Registered Ports are those from 1024 through 49151
# The Dynamic and/or Private Ports are those from 49152 through 65535
#
# Each line describes one service, and is of the form:
#
# service-name  port/protocol  [aliases ...]   [# comment]

tcpmux          1/tcp                           # TCP port service multiplexer
tcpmux          1/udp                           # TCP port service multiplexer
rje             5/tcp                           # Remote Job Entry
rje             5/udp                           # Remote Job Entry
echo            7/tcp
echo            7/udp
discard         9/tcp           sink null
discard         9/udp           sink null
systat          11/tcp          users
systat          11/udp          users
daytime         13/tcp
daytime         13/udp
/etc/services
```

While in less:

`f` - go forward one page

`b` - go back one page

`/searchterm` - search forwards for searchterm, eg: `/http`

`?searchterm` - search backwards

`n` - go to next search match

`q` - get out of less

`head /path/to/file` - show top 10 lines

`head -n 3 /path/to/file` - show top 3 lines

`tail /etc/services` - show bottom 10 lines

`tail -n 3 /etc/services` - show bottom 3 lines

### Regular Expressions and grep

`yum list installed` - list all installed packages - big list.

`yum list installed | grep rpm` - pipe output of yum to input of grep, searching for rpm

```shell
$ yum list installed | grep rpm
rpm.x86_64                                4.11.3-40.el7                 @CentOS
rpm-build-libs.x86_64                     4.11.3-40.el7                 @CentOS
rpm-libs.x86_64                           4.11.3-40.el7                 @CentOS
rpm-python.x86_64                         4.11.3-40.el7                 @CentOS
```

`yum list installed | grep ^rpm` - filter to those packages that *start with* `rpm`.

`yum install ntp` - Install network time protocol daemon

`sudo some-command` - Run as root. Note first user installed gets option to be added to administrators, aka `wheel` group.

`cat /etc/ntp.conf` - Display ntp configuration file - a lot of content

`wc -l !$` - line count on last argument, in this case, ntp conf file - 58

`cp !$ .` - copy ntp conf file to current directory (we're in /documents)

`grep server ntp.conf` - search for all occurrences of `server` in file `ntp.conf` - get a few lines of results, results are color highlighted due to pre-installed alias (see next line)

```
# Use public servers from the pool.ntp.org project.
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
#broadcast 192.168.1.255 autokey        # broadcast server
#broadcast 224.0.1.1 autokey            # multicast server
#manycastserver 239.255.254.254         # manycast server
```

`type grep` - grep is aliased to `grep --color=auto'

`grep ^server ntp.conf` - only search for lines that begin with server

```
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
```

Look for `server` with word boundary (space, newline or hyphen)

```shell
$ grep '\bserver\b' ntp.conf
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
#broadcast 192.168.1.255 autokey        # broadcast server
#broadcast 224.0.1.1 autokey            # multicast server
#manycastserver 239.255.254.254         # manycast server
```

`yum install words` - install dictionary file

`grep -E` - enhanced search

```shell
# search for words that end in ion
$ grep -E 'ion$' /usr/share/dict/words
...
vulgarization
vulneration
wanion
Welsh-fashion
werelion
westernisation
westernization
Whiggification
whisperation
...
# search for words that start with po, followed by any two characters, and then ends with ute
$ grep -E '^po..ute$' /usr/share/dict/words # lol - crossword puzzle solver!
pollute
# search for words that have 5 vowels in a row - look for 5 a or e or i or o or u in a row
$ grep -E '[aeiou]{5}' /usr/share/dict/words
cadiueio
Chaouia
cooeeing
euouae
Guauaenok
miaoued
miaouing
Pauiie
queueing
```

### Using sed to Edit Files

`;` - separates expressions

`s` - substitution

`d` - deletion

`-i` - in place editing

`/^$/` - regex for empty line - i.e. start `^` followed by end `$` with nothing in between

Delete all commented and empty lines in our local copy of `ntp.conf`:

```shell
$ sed '/^#/d ; /^$/d' ntp.conf # writes to standard out, does NOT edit file
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict ::1
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor

$ sed -i '/^#/d ; /^$/d' ntp.conf # in place editing of ntp.conf file
```

Define a function in memory, `$1` is argument that will be supplied through to function when invoked:

```shell
$ function clean_file {
  sed -i '/^#/d;/^$/d' $1
}
$ clean_file ntp.conf
$ cat ntp.conf # much shorter now with all comments and empty lines removed
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict ::1
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```

### Comparing Files

`diff` - to compare two files

Working with cleaned up `ntf.conf` from previous module, in `documents` dir. Make a copy, then modify it.

```shell
$ cp ntp.conf ntp.new
$ echo new >> ntp.new
$ diff ntp.conf ntp.new
11a12 # 11: lines in first file, a: new line added, 12: lines in second file
> new # this is the line that's been added, greater than symbol indicates line added to second file
```

Use `vi` to edit new file to change a config line - change `server 0` to `server 20`. Esx + `:x` to save changes. Then `diff` again:

```shell
$ vi ntp.new
$ diff ntp.conf ntp.new
5c5 # line 5 in both files was changed
< server 0.centos.pool.ntp.org iburst # line 5 in first file (less than sign)
---
> server 20.centos.pool.ntp.org iburst # line 5 in second file (greater than sign)
11a12
> new
```

Compare to original `/etc/ntp.conf` with all the comments and empty lines:

```shell
$ diff /etc/ntp.conf ntp.conf
1,3d0 # lines 1 through 3 deleted, less than sign on following lines shows lines deleted from original file
< # For more information about this file, see the man pages
< # ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).
<
5,7d1 # lines 5 through 7 deleted from original file
<
< # Permit time synchronization with our time source, but do not
< # permit the source to query or modify the service on this system.
...
```

**Compare binary files**

Can compare *checksum*. Use `md5sum` command to compute checksum. Can redirect output to file to save for later comparison.

```shell
$ md5sum /usr/bin/passwd
41d061f50a92d36b61723e6e22d7fefa  /usr/bin/passwd
$ md5sum /usr/bin/passwd > server1
# run same command on another server then compare checksums
```

### Finding Files

`find` - built-in find command

`-name` case sensitve search

`-iname` case insensitve search

`-delete` delete the files that were found

```shell
$ find /usr/share/gnupg -name '*.txt' # recursively search for txt files in /usr/share/gnupg
# by default, -print is included which prints the results
$ find /usr/share/gnupg -name '*.txt' -print
/usr/share/gnupg/help.be.txt
/usr/share/gnupg/help.ru.txt
/usr/share/gnupg/help.sv.txt
/usr/share/gnupg/help.it.txt
...
```

Use `exec` to execute a command on each find result, `{}` represents a result, eg: copy to to current dir (documents), end the line with `\;` to indicate end of line, because `{}` will be run for each find result:

```shell
$ find /usr/share/gnupg -name '*.txt' -exec cp {} . \;
$ ls *.txt
ls *.txt
help.be.txt  help.da.txt  help.eo.txt  help.fi.txt  help.hu.txt  help.ja.txt  help.pt.txt     help.ru.txt  help.tr.txt     help.zh_TW.txt
help.ca.txt  help.de.txt  help.es.txt  help.fr.txt  help.id.txt  help.nb.txt  help.pt_BR.txt  help.sk.txt  help.txt        qualified.txt
help.cs.txt  help.el.txt  help.et.txt  help.gl.txt  help.it.txt  help.pl.txt  help.ro.txt     help.sv.txt  help.zh_CN.txt
```

If don't specify dir to search, `find` assumes searching current dir.

```shell
$ find -name '*.txt' # verify finds files we copied previously
./help.be.txt
./help.ru.txt
./help.sv.txt
./help.it.txt
./help.pt_BR.txt
...
$ find -name '*.txt' -delete
$ ls *.txt
ls: cannot access *.txt: No such file or directory
```

`-type` to search for specific file type, eg: `-type l` to saerch for symbolic links:

```shell
$ find /etc/ -type l # find symbolic links in etc and all its sub-dirs recursively
/etc/rc4.d
/etc/alternatives/libnssckbi.so.x86_64
/etc/alternatives/ld
/etc/alternatives/mta-aliasesman
...
```

By default, find searches recursively, to only search desired dir, use `-maxdepth 1`:

```shell
$ find /etc -maxdepth 1 -type l
/etc/rc4.d
/etc/system-release
/etc/redhat-release
/etc/rc5.d
...
```

`-size` to search by file size, eg: search for regular files greater than 10MB:

```shell
$ find / -size +20000k -type f
/var/cache/yum/x86_64/7/base/gen/primary_db.sqlite
/var/cache/yum/x86_64/7/base/gen/filelists_db.sqlite
/var/cache/yum/x86_64/7/updates/gen/primary_db.sqlite
```

To display disk usage of each find result in human readable format (`du -h`), use `exec`:

```shell
$ find / -size +20000k -type f -exec du -h {} \;
31M     /var/cache/yum/x86_64/7/base/gen/primary_db.sqlite
49M     /var/cache/yum/x86_64/7/base/gen/filelists_db.sqlite
30M     /var/cache/yum/x86_64/7/updates/gen/primary_db.sqlite
```

## Using the vim Text Editor

### Creating and Editing Files

`touch` easiest way to create file

`nano` simplest text editor but not as powerful as vi/vim

Working in `documents` dir:

```shell
$ touch newfile
$ ls -l newfile
-rw-r--r-- 1 root root 0 Dec 26 12:40 newfile
```

`> somefile` - another way to create a new file using redirection, i.e. redirecting nothing:

```shell
$ > newfile1
$ ls -l newfile1
-rw-r--r-- 1 root root 0 Dec 26 12:43 newfile1
```

`touch` on existing file will update modified time:

```shell
$ touch newfile
$ ls -l newfile
-rw-r--r-- 1 root root 0 Dec 26 12:46 newfile # was 12:40 when initially created (see above)
$ stat newfile
  File: 'newfile'
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: 801h/2049d      Inode: 2494140     Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2019-12-26 12:46:41.655573921 +0000
Modify: 2019-12-26 12:46:41.655573921 +0000
Change: 2019-12-26 12:46:41.655573921 +0000
 Birth: -
```

`touch -d 'some date'` - To set a particular modified date

`!s` - reverse search through history for last command starting with `s`

```shell
$ touch -d '10 April 1973' newfile
$ !s
stat newfile
  File: 'newfile'
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: 801h/2049d      Inode: 2494140     Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 1973-04-10 00:00:00.000000000 +0000
Modify: 1973-04-10 00:00:00.000000000 +0000
Change: 2019-12-26 12:50:53.663414753 +0000
 Birth: -
```

Change time reflects when file was touched, i.e. last time metadata was changed.

But `touch -d` modifies the last Access and Modify time.

### Using the nano Text Editor

`yum install -y nano`

`nano newfile` - open nano text editor with `newfile` open for editing.

Just start typing to edit the file, eg:

```
This is a new file
with text that we can create
as we wish

^x to exit
y to save
enter to accept newfile as name

$ cat newfile
This is a new file
with text that we can create
as we wish

```

Bottom of screen dispalys all control characters that can be used to perform actions, eg: `^x` to exit, which will prompt Y/N to save buffer before exiting.

To add more text, run `nano newfile` which will load text editor with existing file contents, navigate to end of file, add more text, exit/save again.

### Learning vim with vimtutor

`vimtutor` - opens vimtutor document in vim editor

Use arrow down or `j` key to navigate down.

Follow along with tutorial following instructions.

**Basics:**

- `vim` - launch vim editor
- `esc + :q` - exit

- `j` navigate down
- `k` navigate up
- `h` navigate left
- `l` navigate right
- `G` go to last line
- `1G` go to first line, `2G` - go to second line, etc.
- `^` go to beginning of line (just like regex where `^` represents beginning of line)
- `$` go to end of line (again, just like regex)

- `Esc` get into (or stay in) command mode
- `:` start entering a command at the command line (given that you're in command mode)
- `:q!` force quit vim without saving any changes
- `:q` quit vim when there are no changes to be saved
- `:wq` write (aka save) changes and then quit
- `:e!` revert to last saved version (getting rid of any current unsaved changes)

- `:6,8w file99` write lines 6 through 8 of current file to a new file named `file99` in current directory
- `:r file99` read in the contents of `file99` and write them out at current cursor position

- `ctrl+d` - move cursor to beginning of line (in insert mode)
- `A` append - moves cursor to end of line and enters insert mode
- `a` appends - will type after cursor position
- `i` inserts - will type before cursor position
- `I` inserts at start of line
- `O` inserts new line above current cursor position and enters insert mode
- `o` inserts new line below current cursor position and enters insert mode

- `yy` copy current line
- `2yy` copy two lines (current and the one below)
- `p` paste below current line
- `P` paste above current line

- `dw` delete one word
- `d$` delete to the end of the line
- `dd` delete the entire line
- `2dd` delete two lines (current and the one below)
- `dG` delete to end of file
- `u` undo the last change, can keep going back through changes

- `g~~` change case of entire line (toggle upper)
- `gUU` change entire line to upper case
- `~` toggle case for one character


### Setting Defaults with .vimrc

`.vimrc` is a control file for vim, should be in user's home directory (for Docker centos running as root, this will be `/root/.vimrc`)

Editing an existing file:

```shell
$ vim /documents/newfile
```

By default vim launches in command mode, have to hit `i` to get into `INSERT` mode - then the status bar at the bottom shows `INSERT`, but it may not do so on all systems. To always show the mode, hit `esc`, then enter `:set showmode`.

`:set number` - turn on line numbers

`:set nonumber` - turn off line numbers

`:set invnumber` - toggle line numbers on/off

`:set nohlsearch` - do not highlight search results

`:set ai` - auto indent - useful for creating scripts

`:set ts=4` - set tab spaces to 4

`:set expandtab` - ???

From `:` command line, use up arrow key to access previous command(s)

`cd` with no arguments will take you to your home directory

```shell
$ cd
$ pwd
/root
$ vim .vimrc # file doesn't exist so it will be created a new and empty file
```

`i` to get into insert mode, then start typing - these are instructor's preferences.

Can put multiple settings on same line or break it up into multiple `set` statements.

`abbr _sh #!/binbash` - abbreviation, whenever `_sh` is entered, will be expanded to `#!/binbash`

`nmap <C-N> :set invnumber<CR>` - normal mode mapping control N represents setting invnumber following by enter (carraige return === CR)

```
set showmode nonumber nohlsearch
set ai ts=4 expandtab
abbr _sh #!/bin/bash
nmap <C-N> :set invnumber<CR>
```

Test out the changes by editing previous file `vim /documents/newfile` - now can use Ctrl+n to toggle show line numbers setting.

Enter insert mode, then type `_sh ` - gets converted to shebang.

Tab now goes in 4 spaces, hitting Enter after tab maintains the indentation.

### Editing Files with vim

Will work with `vim /documents/newfile` - see all commands at beginning of vim section for editing.

## Piping and Redirection

### Redirecting STDOUT

`>` rediect output, eg: `> file1` redirects standard output to `file1` (creating it if didn't already exist)

`> newfile` will overwrite contents of existing file with nothing.

`ls > newfile` redirect output of `ls` command to file `newfile`

Looking at disk free of local file system, can also redirect its output:

```shell
$ df -h > file1
$ cat file1
Filesystem      Size  Used Avail Use% Mounted on
overlay          59G  1.6G   54G   3% /
tmpfs            64M     0   64M   0% /dev
tmpfs          1000M     0 1000M   0% /sys/fs/cgroup
shm              64M     0   64M   0% /dev/shm
/dev/sda1        59G  1.6G   54G   3% /etc/hosts
tmpfs          1000M     0 1000M   0% /proc/acpi
tmpfs          1000M     0 1000M   0% /sys/firmware
```

Placing `1` before `>` -> explicitly stating that standard output is being redirected, eg: `df -h 1> file1`

`1` === standard output

`2` === standard error

`>` === overwrite file

`>>` === append to file, eg: `df -h 1>> file1`

When `1`, `2` etc not specified, assume standard output.

Risk when using `>` - if didn't know file already existed, may accidentally overwrite contents.

### Using the noclobber Option

Shell option to avoid accidentally overwriting a file with redirection.

`set -o` - display shell options

```shell
$ set -o
allexport       off
braceexpand     on
emacs           on
errexit         off
errtrace        off
functrace       off
hashall         on
histexpand      on
history         on
ignoreeof       off
interactive-comments    on
keyword         off
monitor         on
noclobber       off
noexec          off
noglob          off
nolog           off
notify          off
nounset         off
onecmd          off
physical        off
pipefail        off
posix           off
privileged      off
verbose         off
vi              off
xtrace          off
```

`noclobber` is turned off by default.

Shell options can be turned on/off manually at command line, or automatically as part of login script.

To turn on an option manually:

```shell
$ set -o noclobber
$ set -o | grep noclobber
noclobber       on
```

Now let's see now having `noclobber` affects things, in a directory where `file1` already exists, but `file2` does not

`>|` - force the write

```shell
$ ls
file1  file99  newfile
$ date +%F # display formatted date
2019-12-26
$ date+%F > file1
bash: file1: cannot overwrite existing file
$ date+%F > file2 # this is fine because file2 does not already exist
$ date+%F > file2 # not anymore because now file2 already exists
bash: file1: cannot overwrite existing file
$ date+%F >| file2 # force it
$ ls $HOME >> file2 # this is fine because we're appending rather than overwriting
$ > file2 # nope
bash: file2: cannot overwrite existing file
```

Add option to login script:

```shell
$ cd
$ vim .bashrc
# add "set -o noclobber" in User section, then save and quit
```

To turn the option off: `set +o noclobber`

`-o` turn option on

`+o` turn option off

### Redirecting STDERR

When things go wrong.

When everything goes well, eg `ls /etc` - results go to standard out, the screen.

When something goes wrong, eg trying to view a dir that doesn't exist, output no longer goes to standard output, even on redirection.

`2>` redirect to standard error.

```shell
$ ls /etcw
ls: cannot access /etcw: No such file or directory
$ ls /etcw > err
ls: cannot access /etcw: No such file or directory
$ cat err # empty file
$ ls /etcw 2> errfile
$ cat errfile
ls: cannot access /etcw: No such file or directory
```

`find` command example where some results are permission denied, since it's known will be some errors, redirect then to `/dev/null`, then output can be displayed without errors.

`/dev/null` - special device file, used when don't want to see the output.

First switch to a non-root user:

```shell
$ su - course
$ find /etc -type l
find: '/etc/ntp/crypto': Permission denied
/etc/rc4.d
/etc/alternatives/libnssckbi.so.x86_64
/etc/alternatives/ld
/etc/alternatives/mta-aliasesman
...
$ find /etc -type l 2> /dev/null
/etc/rc4.d
/etc/alternatives/libnssckbi.so.x86_64
/etc/alternatives/ld
/etc/alternatives/mta-aliasesman
/etc/alternatives/mta
...
```

`&>` - Redirect both standard output and standard error to same file

```shell
$ find /etc -type l &> all.txt
/etc/rc4.d
/etc/alternatives/libnssckbi.so.x86_64
/etc/alternatives/ld
/etc/alternatives/mta-aliasesman
...
find: '/etc/ntp/crypto': Permission denied
```

### Reading into STDIN

`df -hlT` Show diskfile system, human readable, only local, also show file system type. Might want an email report of this:

`<` read in from standard input

```shell
$ df -hlT
Filesystem     Type     Size  Used Avail Use% Mounted on
overlay        overlay   59G  1.6G   54G   3% /
tmpfs          tmpfs     64M     0   64M   0% /dev
tmpfs          tmpfs   1000M     0 1000M   0% /sys/fs/cgroup
shm            tmpfs     64M     0   64M   0% /dev/shm
/dev/sda1      ext4      59G  1.6G   54G   3% /etc/hosts
tmpfs          tmpfs   1000M     0 1000M   0% /proc/acpi
tmpfs          tmpfs   1000M     0 1000M   0% /sys/firmware
$ df -hlT > diskfree
# Send local mail to course user with subject Disk Free, reading in from diskfree file for mail body
$ mail -s "Disk Free" course < diskfree # not working for me in container
```

### Using HERE documents

Good for creating small documents at command line.

`>>SOMEINDICATOR` - heredoc, when SOMEINDICATOR appears that indicates end of document but doesn't become part of document.

```shell
$ cat > mynewfile <<END
> this is a little
> file that we can create
> even with scripts
> END
$ cat mynewfile
this is a little
file that we can create
even with scripts
```

### Command Pipelines

Linux has many small commands, designed to be linked together with pipelines.

`|` unnamed pipe

```shell
# pipe output of ls through word count for number of lines
$ ls | wc -l
9 # i.e. 9 files were listed from ls command
$ head -n1 /etc/passwod
root:x:0:0:root:/root:/bin/bash # output is fields separted by colon
$ cut -f7 -d: /etc/passwd # cut field 7 using colon delimiter -> show all the different default shells
/bin/bash
/sbin/nologin
/sbin/nologin
/sbin/nologin
/sbin/nologin
/bin/sync
/sbin/shutdown
/sbin/halt
/sbin/nologin
/sbin/nologin
...
/bin/bash
# send the output of above through sort
cut -f7 -d: /etc/passwd | sort
/bin/bash
/bin/bash
/bin/sync
/sbin/halt
/sbin/nologin
/sbin/nologin
/sbin/nologin
...
/sbin/shutdown
# and pipe through uniq
$ cut -f7 -d: /etc/passwd | sort | uniq
/bin/bash
/bin/sync
/sbin/halt
/sbin/nologin
/sbin/shutdown
# count how many different shells
$ cut -f7 -d: /etc/passwd | sort | uniq | wc -l
5
```

### Named Pipes

Used for IPC - Interprocess communication - allowing different processes to talk to each other, where those processes are running in different shells - implemented with *named pipes* - special type of file - notice `p` character.

`mkfifo` create a named pipe, first-in/first-out - queues info until its serviced

```shell
$ mkfifo mypipe
$ ls -l !$
$ ls -l mypipe
prw-r--r-- 1 root root 0 Dec 26 23:56 mypipe # p indicates its a pipe file type
$ ls -F mypipe
mypipe| # pipe character at end indicates this is a pipe file type
```

Pipe file can be used in same way as `|` but not limited to being in same shell session.

Named pipe never holds any information, rather, it marshals info from one process to another.

Open another terminal and connect to centos container.

```shell
# terminal 2
$ ls > mypipe # send output of ls through to named pipe
# command is held up until pipe is read in other terminal, then returns

# terminal 1
$ wc -l < mypipe # read input from named pipe
10 # count of how many files `ls` command returned in terminal 2
```

### Using the Command tee

`tee` redirect to file *and* display in console. Input comes in via pipe, then output goes out in two directions.

`tee` with no additional options will overwrite file if it already exists.

`tee -a` append to file.

```shell
$ ls > f89 # writes through to f89 but don't see output on screen
$ ls | tee ftee # output displayed on screen
anaconda-ks.cfg
diskfree
err
errfile
f89
file1
file2
file99
mynewfile
mypipe
newfile
$ cat ftee # output also got redirected to file named `ftee`
anaconda-ks.cfg
diskfree
err
errfile
f89
file1
file2
file99
mynewfile
mypipe
newfile
```

Use when don't have permission to write to a file, but able to use `sudo`. In example below, `echo` command is running with `sudo` privileges but redirection falls back to standard user permissions.

```shell
$ su - course
$ ls -l /etc/hosts
-rw-r--r-- 1 root root 174 Dec 27 12:58 /etc/hosts # course user does not have write permission
$ sudo echo '127.0.0.1 bob' >> /etc/hosts
-bash: /etc/hosts: Permission denied
# solution is to redirect output of echo through to sudo and tee
$ echo '127.0.0.1 bob' | sudo tee /etc/hosts
127.0.0.1 bob # tee shows output on console
$ cat /etc/hosts
127.0.0.1 bob # hmm... shoulda used tee -a
```

## Archiving Files

### Using the tar Command

`tar` - tape archive (historical definition)

`-c` === `--create`

`-v` verbose switch

`-vv` also show file permissions as they're being backed up

`-f` === `--file` specify file you want to back up to

`-t` === `--list` list contents of tar

`-x` === `--extract` extract contents of archive

`--directory` specify where to extract to (if not given, defaults to current directory)

Argument to tar is file/dir to be backed up

Create an archive, leading `/` removed from file names because when a tar is restored, by default it restores to current directory, so file names must have relative names.

```shell
$ tar -cf doc.tar /usr/share/doc
tar: Removing leading `/' from member names
```

Turn on verbose switch to see files as they're being backed up:

```lang
$ tar -cvf doc.tar /usr/share/doc
tar: Removing leading `/' from member names
/usr/share/doc/
/usr/share/doc/dbus-1.10.24/
/usr/share/doc/krb5-libs-1.15.1/
/usr/share/doc/coreutils-8.22/
/usr/share/doc/python-kitchen-1.1.1/
...
$ tar -cvvf doc.tar /usr/share/doc
tar: Removing leading `/' from member names
drwxr-xr-x root/root         0 2019-12-26 21:05 /usr/share/doc/
drwxr-xr-x root/root         0 2019-03-14 10:18 /usr/share/doc/dbus-1.10.24/
drwxr-xr-x root/root         0 2019-09-13 18:07 /usr/share/doc/krb5-libs-1.15.1/
...
```

Show what's in a tar file:

```shell
$ tar --list --file=doc.tar
usr/share/doc/
usr/share/doc/dbus-1.10.24/
usr/share/doc/krb5-libs-1.15.1/
usr/share/doc/coreutils-8.22/
...
$ tar -tvf doc.tar # same as above but short form and add verbose switch
drwxr-xr-x root/root         0 2019-12-26 21:05 usr/share/doc/
drwxr-xr-x root/root         0 2019-03-14 10:18 usr/share/doc/dbus-1.10.24/
drwxr-xr-x root/root         0 2019-09-13 18:07 usr/share/doc/krb5-libs-1.15.1/
...
```

Extract tar contents, by default, expands to *current* directory:

```shell
$ tar -xvf doc.tar
usr/share/doc/
usr/share/doc/dbus-1.10.24/
usr/share/doc/krb5-libs-1.15.1/
usr/share/doc/coreutils-8.22/
...
$ cd usr
$ ls
share
```

Restore to original location:

```shell
$ tar -xvf doc.tar --directory=/
```

Incremental backups - backup files that have changed since last backup:

`-g` indicate .snar file - mix of snapshot and tar - records changes in file system between backups. If snar file doesn't exist, it will be created and full backup will be run.

```shell
$ mkdir test
$ cd test
$ touch hosts hostname services
$ cd ..
# Day 0: kick off incremental backup of test dir
$ tar -cvf my0.tar -g my.snar test
tar: test: Directory is new
test/
test/hostname
test/hosts
test/services
# Day 1: some files get updated, run incremental update again, specifying new archive but same snar file
$ echo hi >> test/hosts
$ tar -cvf my1.tar -g my.snar test
# only modified file gets backed up
test/
test/hosts
# Day 2: some file gets removed
$ rm -f test/hostname
$ tar -cvf my2.tar -g my.snar test
test/
# Day 3: Disaster strikes, test dir is removed
$ rm -rf test
# Restore from Day 0 backup
$ tar -xvf my0.tar -g /dev/null
test/
test/hostname
test/hosts
test/services
# Restore from Day 1 backup
$ tar -xvf my1.tar -g /dev/null
test/
test/hosts
# Restore from Day 2 backup
$ tar -xvf my2.tar -g /dev/null
test/
tar: Deleting `test/hostname'
```

### Using Compression

`gzip`/`gunzip` - compress/uncompress

`file` - command to show what type of file some file is.

`tar -z` - tar and compress with gzip

`tar -j` - tar and compress with bzip2

Working with `doc.tar` file created earlier, gzip removes the tar and replaces with tar.gz file:

```shell
$ ls -lh doc.*
-rw-r--r-- 1 root root 20K Dec 27 14:21 doc.tar
$ gzip doc.tar
$ ls -lh doc.*
-rw-r--r-- 1 root root  695 Dec 27 14:21 doc.tar.gz
# look at file
$ file doc.tar.gz
doc.tar.gz: gzip compressed data, was "doc.tar", from Unix, last modified: Fri Dec 27 14:21:02 2019
# unzip -> removes .gz file and just leaves the .tar file
$ gunzip doc.tar.gz
$ ls -lh doc*
-rw-r--r-- 1 root root 20K Dec 27 14:21 doc.tar
$ file doc.tar
doc.tar: POSIX tar archive (GNU)
```

`bzip2` - further compression, first install: `yum install bzip2`

```shell
$ bzip2 doc.tar
$ ls -lh doc*
-rw-r--r-- 1 root root 605 Dec 27 14:21 doc.tar.bz2
$ file doc.tar.bz2
doc.tar.bz2: bzip2 compressed data, block size = 900k
$ bunzip2 doc.tar.bz2
$ ls -lh doc*
-rw-r--r-- 1 root root 20K Dec 27 14:21 doc.tar
```

How long does tar take? Start from `/tmp` dir, then backup `$HOME`:

```shell
$ cd /tmp
$ time tar -cvf myhome.tar $HOME
tar: Removing leading `/' from member names
/root/
/root/.bash_profile
/root/.tcshrc
/root/anaconda-ks.cfg
/root/.bash_logout
/root/.cshrc
/root/.bashrc
...
/root/file99

real    0m0.012s
user    0m0.000s
sys     0m0.000s

# tar with compression is a little faster (but for instructor was slower)
$ time tar -cvzf myhome.tar.gz $HOME
...
/root/my2.tar
/root/my0.tar
/root/file99

real    0m0.009s
user    0m0.000s
sys     0m0.000s

# tar and bzip2 compression slightly faster but for instructor was slower
$ time tar -cvjf myhome.tar.bz2 $HOME
...
/root/my2.tar
/root/my0.tar
/root/file99

real    0m0.008s
user    0m0.000s
sys     0m0.000s
```

Have to balance compressed size with how long it takes to generate it.

## Understanding File Permissions

### Listing Permissions

Long list on a shell script:

```shell
$ ls -l /usr/lib/dracut/dracut-init.sh
-rwxr-xr-x 1 root root 1188 Aug  8 23:14 /usr/lib/dracut/dracut-init.sh
```

Breaking down `-rwxr-xr-x`

`-` regular file
`rwx` read/write/execute for file owner (root in above example)
`r-x` read/execute for group owner belongs to (root in above example)
`r-x` read/execute for others

Permissions shown with `ls -l` are shown *symbolically* (rwx), use `stat` for complete metadata:

```shell
$ stat /usr/lib/dracut/dracut-init.sh
  File: '/usr/lib/dracut/dracut-init.sh'
  Size: 1188            Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d      Inode: 2360184     Links: 1
Access: (0755/-rwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2019-08-08 23:14:24.000000000 +0000
Modify: 2019-08-08 23:14:24.000000000 +0000
Change: 2019-12-15 17:44:58.576242164 +0000
 Birth: -
```

Access line shows permission numerically (aka octally 0 through 7): `0755`

Extract symbolic and numeric permissions with stat:

```shell
$ stat -c %A /usr/lib/dracut/dracut-init.sh
-rwxr-xr-x
$ stat -c %A /usr/lib/dracut/dracut-init.sh
755
```

`777` Max permission

* `4` = read
* `2` = write
* `1` = execute

`7` = r + w + x (for owner)

`5` = r + x (for group)

`5` = r + x (for others)

### Managing Default Permissions

Note: Instructor was not very clear on `umask`, see this for [better explanation](https://linoxide.com/how-tos/umask-set-permission/)

0 -- Read, Write and Execute
1 – Read and Write
2 – Read and Execute
3 – Read Only
4 –Write and Execute
5 –Write Only
6 –Execute Only
7 –No Permissions

`umask 077` means:

0 – Owner – Read, Write and Execute
7 – Group – No Permissions
7 – Others – No Permissions

Default permissions are read/write for user, group and others `rw-rw-rw-` but can be affected by `umask`.

`umask` can only be used to *remove* permissions, cannot add.

Do as regular user:

```shell
$ su - course
$ ls -l file1
-rw-rw-r-- 1 course course 0 Dec 27 20:22 file1
$ umask
0002 # `2` being in others block -> write is being removed from others
$ umask 0
$ umask
0000
$ touch file2
$ ls -l file1 file2
-rw-rw-r-- 1 course course 0 Dec 27 20:22 file1 # created with umask 0002
-rw-rw-rw- 1 course course 0 Dec 27 20:25 file2 # created with umask 0000
$ stat -c %a file1
664
$ stat -c %a file2
666
```

For further default privacy on files:

```shell
$ umask 27
$ touch file3
$ ls -l
-rw-rw-r-- 1 course course 0 Dec 27 20:22 file1
-rw-rw-rw- 1 course course 0 Dec 27 20:25 file2
-rw-r----- 1 course course 0 Dec 27 21:38 file3
$ umask 77
$ touch file4
$ ls -l
-rw-rw-r-- 1 course course 0 Dec 27 20:22 file1
-rw-rw-rw- 1 course course 0 Dec 27 20:25 file2
-rw-r----- 1 course course 0 Dec 27 21:38 file3
-rw------- 1 course course 0 Dec 27 21:39 file4
```

`umask` usually set octally because that's how it's displayed, represents what to block.

Can also set symbolically:

```shell
$ umask u=rwx,g=rx,o=rx # === umask 033
$ touch file5
$ ls -l
-rw-rw-r-- 1 course course 0 Dec 27 20:22 file1
-rw-rw-rw- 1 course course 0 Dec 27 20:25 file2
-rw-r----- 1 course course 0 Dec 27 21:38 file3
-rw------- 1 course course 0 Dec 27 21:39 file4
-rw-r--r-- 1 course course 0 Dec 27 22:48 file5
$ umask 033
$ touch file6
$ ls -l
-rw-rw-r-- 1 course course 0 Dec 27 20:22 file1
-rw-rw-rw- 1 course course 0 Dec 27 20:25 file2
-rw-r----- 1 course course 0 Dec 27 21:38 file3
-rw------- 1 course course 0 Dec 27 21:39 file4
-rw-r--r-- 1 course course 0 Dec 27 22:48 file5
-rw-r--r-- 1 course course 0 Dec 27 22:48 file6
```

Set private umask:

```shell
$ umask u=rwx,go=
$ umask
0077
$ touch file7
$ ls -l
-rw-rw-r-- 1 course course 0 Dec 27 20:22 file1
-rw-rw-rw- 1 course course 0 Dec 27 20:25 file2
-rw-r----- 1 course course 0 Dec 27 21:38 file3
-rw------- 1 course course 0 Dec 27 21:39 file4
-rw-r--r-- 1 course course 0 Dec 27 22:48 file5
-rw-r--r-- 1 course course 0 Dec 27 22:48 file6
-rw------- 1 course course 0 Dec 27 22:51 file7
```

### Setting Permissions

`chmod` change permission, can use numeric or symbols.

Logged in as user `course`, add some content to `file` created earlier with default umask:

```shell
$ ls file1
rw-rw-r-- 1 course course 0 Dec 27 20:22 file1
$ vi file1
$ cat file1
this is content
$ chmod 467 file1 # === chmod u=r,g=rw,o=rwx file1
$ ls -l file1
-r--rw-rwx 1 course course 16 Dec 27 23:35 file1
$ vim file1
# vim status line shows: "file1" [readonly] 1L, 16C
```

Even though file is read-only to user, still allowed to delete it, because delete is actually a write operation on the parent directory. In this case user has `rwx` on its own home dir `/home/course`.

If there is match on user id, group id is not checked. So in this case, only have read on `file1`.

Modify permissions for all - use `+` or `-` characters:

```shell
$ ls -l file2
-rw-rw-rw- 1 course course 0 Dec 27 20:25 file2
$ chmod a+x file2 # add execute for all - user, group, and others
$ ls -l file2
-rwxrwxrwx 1 course course 0 Dec 27 20:25 file2
$ chmod a-x file2 # remove execute for all - user, group, and others
$ ls -l file2
-rw-rw-rw- 1 course course 0 Dec 27 20:25 file2
$ chmod o= file2 # make it private to others
$ ls -l file2
-rw-rw---- 1 course course 0 Dec 27 20:25 file2
$ chmod 777 file2 # make it open to the world
$ ls -l file2
-rwxrwxrwx 1 course course 0 Dec 27 20:25 file2
$ chmod go= file2 # remove permissions from group and others
$ ls -l file2
-rwx------ 1 course course 0 Dec 27 20:25 file2
```

*sticky bit* controls deletions from directory, eg: `/tmp` has sticky bit set, shown in ls displayas `t`.

lower case t - execute set on dir

upper case T - execute not set on dir

When sticky bit is set on dir, only owner of the file can delete from that dir.

```shell
$ umask 0077
$ ls -ld /tmp
drwxrwxrwt 1 root root 4096 Dec 28 14:41 /tmp
```

To create dir with sticky bit set, value is `1`:

`-m` mode when creating dir

```shell
$ mkdir -m 1777 test
$ ls -ld test
drwxrwxrwt 2 course course 4096 Dec 28 15:02 test
```

Remove sticky bit from dir, for others:

```shell
$ chmod o-t test
$ ls -ld test
drwxrwxrwx 2 course course 4096 Dec 28 15:02 test
```

Can also control group, for example `/bin/wall` is owned by `root` but group is `tty`, so when `wall` is run, runs with permissions of `tty` group.

```shell
$ ls -l $(which wall)
-r-xr-sr-x 1 root tty 15344 Jun  9  2014 /bin/wall
```

### Managing File Ownership

User owner vs Group owner.

`id -u` - show user id of currently logged in user

`id -un` - show username

`id -gn` - show primary group name

`id -Gn` - show secondary group (aka complementary groups)

`chgrp <user> <file>` Change group to any group user belongs to

`newgrp <group>` Change primary group -> creates new shell, `exit` to get back to your previous group/shell

When creating files, primary group is used.

When accessing file resources, complementary groups are used.

In example below, `file2` was created by user `course`, i.e. course is the *user owner*.

*primary group id* of `course` user is the `course` group, this is *group owner*.

```shell
$ ls -l file2
-rwx------ 1 course course 0 Dec 27 20:25 file2
$ id -u
1000
$ id -un
course
$ id -gn
course
$ id -Gn
course wheel # wheel group is centos admin group
$ chgrp wheel file2
$ ls -l file2
-rwx------ 1 course wheel 0 Dec 27 20:25 file2 # now wheel group takes ownership
$ newgrp wheel
$ id -gn
wheel
$ touch newgroup
$ ls -l newgroup
-rw-r--r-- 1 course wheel 0 Dec 29 20:31 newgroup # wheel is now the owner of group
$ exit
$ id -gn
course
```

CANNOT change user owner, as regular user, need root privileges.

`exit` out of `course` user to get to `root`

`chown <user> <file>` change file owner to a different user

`chown <user>.<group> <file>` change user and group ownership for file, can use `.` or `:` separator

```shell
$ cd /home/course
$ ls -l file2
-rwx------ 1 course wheel 0 Dec 27 20:25 file2
$ hown root file2
$ ls -l file2
-rwx------ 1 root wheel 0 Dec 27 20:25 file2 # now root is owner
$ chown course.course file2
$ ls -l file2
-rwx------ 1 course course 0 Dec 27 20:25 file2
```

Regardless of file ownership, when `root` user copies it, user and group owner will change to `root`, however, this may not be desirable, eg: when backing up files.

`cp -a` copy file maintaining original timestamps and ownership - need to be root to run this.

```shell
$ cp file2 /root/file2na
$ ls -l !$
ls -l /root/file2na
-rwx------ 1 root root 0 Dec 29 20:44 /root/file2na
$ cp -a file2 /root/file2a
$ ls -l !$
-rwx------ 1 course course 0 Dec 27 20:25 /root/file2a
```