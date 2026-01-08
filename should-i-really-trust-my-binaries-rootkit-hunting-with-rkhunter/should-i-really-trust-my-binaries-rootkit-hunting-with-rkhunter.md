## **Introduction**
**/!\ This is purely with an educational purpose, i own those machines. Do not do this on machines you don't own nor have the right to use /!\**


Let me ask you a tricky question : *how do you know that you've never been hacked ?*
- The fact that no-one access it except you ?
- The fact that it's freshly cloned ?
- The fact that you've got layers over layers of security like an EDR/XDR, firewalls, proxies ?
- The fact that you are reading logs, checking everything from time to time ?

Short answer, **you can't**. You will never be sure that your machine is safe, you just assume that it is, and accept the risks that comes with. And let’s face it, we rarely have the time to properly review our logs :)


## **Current installation**
As usual, we have our freshly cloned debian12-template, up to date (even though it's not Debian 13, I know) :
```shell
interlope@deb12-template:~$ cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```
```shell
interlope@deb12-template:~$ sudo apt update
Hit:1 http://deb.debian.org/debian bookworm InRelease
Hit:2 http://deb.debian.org/debian bookworm-updates InRelease
Hit:3 http://security.debian.org/debian-security bookworm-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
```

But if I told you that this machine was infected, would you believe me ? A supposed clean install and freshly cloned, nothing could go wrong right ? Right.. ? 
<br>Ever read [The Hitchhiker’s Guide to the Galaxy](https://en.wikipedia.org/wiki/The_Hitchhiker%27s_Guide_to_the_Galaxy) ? Great book and whatsoever. One of the great thing that came out this book is the number 42 (if you don't know why, just read it or looks on the internet). 

but we'll come back to it later, let's get some definitions right, we'll need them :
 - *rootkit* : malicious software whose purpose is to stay hidden, obtain&maintain access to your system without being seen.
 - *ransomware* : malicious software whose purpose is simply to block access to your own data in exchange for money to restore them.
 - *0day* : undocumented vulnerability found in soft&hard-ware with no available patches or public reports.
 - *cve* : a record of a security flaw with the **impact**, **scope**, **target**, **attack vector** and *sometimes an explanation on how it works* (Common Vulnerabilities and Exposures).
 - *hash* : let's say it's a print calculated by a mathematical function. Each string has his own hash.

Hash example (tata & toto):
```shell
interlope@deb12-template:~$ echo -n "tata" | sha256sum
d1c7c99c6e2e7b311f51dd9d19161a5832625fb21f35131fba6da62513f0c099  -
interlope@deb12-template:~$ echo -n "toto" | sha256sum
31f7a65e315586ac198bd798b6629ce4903d0899476d5741a9f32e2e521b6a66  -
interlope@deb12-template:~$ echo -n "tata" | sha256sum
d1c7c99c6e2e7b311f51dd9d19161a5832625fb21f35131fba6da62513f0c099  -
```

What we will be looking at today are *rootkits*. Our career path is decided, we'll be *rootkit hunters* !

## **What is rkhunter ?**

rkhunter (Rootkit Hunter) is a Unix-based tool that scans for rootkits, backdoors and possible local exploits. It does this by comparing SHA-1 hashes of important files with known good ones in online databases, searching for default directories (of rootkits), wrong permissions, hidden files, suspicious strings in kernel modules, and special tests for Linux and FreeBSD. 
*<br>source : https://en.wikipedia.org/wiki/Rkhunter*

So we should be able to install *rkhunter* on our debian12 virtual machine.
```shell
interlope@deb12-template:~$ apt search rkhunter
Sorting... Done
Full Text Search... Done
forensics-all/oldstable 3.44 all
  Debian Forensics Environment - essential components (metapackage)

rkhunter/oldstable 1.4.6-11 all
  rootkit, backdoor, sniffer and exploit scanner

unhide/oldstable,now 20220611-1 amd64 [installed,auto-removable]
  forensic tool to find hidden processes and ports

unhide.rb/oldstable,now 22-6 all [installed,auto-removable]
  Forensics tool to find processes hidden by rootkits
```

Bingo ! `sudo apt install rkhunter -y` and we should be good.
```shell
interlope@deb12-template:~$ rkhunter -V
Rootkit Hunter 1.4.6

This software was developed by the Rootkit Hunter project team.
Please review your rkhunter configuration files before using.
Please review the documentation before posting bug reports or questions.
To report bugs, provide patches or comments, please go to:
http://rkhunter.sourceforge.net

To ask questions about rkhunter, please use the rkhunter-users mailing list.
Note this is a moderated list: please subscribe before posting.

Rootkit Hunter comes with ABSOLUTELY NO WARRANTY.
This is free software, and you are welcome to redistribute it under the
terms of the GNU General Public License. See the LICENSE file for details.
```

## **Install from sources** (optionnal)
You can also install rkhunter from the [sourceforge repo](https://sourceforge.net/projects/rkhunter/) :
```shell
interlope@deb12-template:~$ wget https://sourceforge.net/projects/rkhunter/files/rkhunter/1.4.6/rkhunter-1.4.6.tar.gz^C
rkhunter-1.4.6.tar.gz       100%[===========================================>] 295.06K  1.55MB/s    in 0.2s
interlope@deb12-template:~$ tar -xzvf rkhunter-1.4.6.tar.gz
interlope@deb12-template:~$ ls
rkhunter-1.4.6  rkhunter-1.4.6.tar.gz
```

You should find an `installer.sh` :
```shell
interlope@deb12-template:~$ cd rkhunter-1.4.6/
interlope@deb12-template:~/rkhunter-1.4.6$ ls
files  installer.sh
interlope@deb12-template:~/rkhunter-1.4.6$
```

You may need to make the installer executable with ``chmod``, let's see what he says when we try to run it :
```shell
interlope@deb12-template:~/rkhunter-1.4.6$ chmod +x installer.sh
interlope@deb12-template:~/rkhunter-1.4.6$ ./installer.sh
Rootkit Hunter installer 1.2.21

Usage: ./installer.sh <parameters>

Ordered valid parameters:
  --help (-h)      : Show this help.
  --examples       : Show layout examples.
  --layout <value> : Choose installation template.
                     The templates are:
                      - default: (FHS compliant; the default)
                      - /usr
                      - /usr/local
                      - oldschool: old version file locations
                      - custom: supply your own installation directory
                      - RPM: for building RPM's. Requires $RPM_BUILD_ROOT.
                      - DEB: for building DEB's. Requires $DEB_BUILD_ROOT.
                      - TGZ: for building Slackware TGZ's. Requires $TGZ_BUILD_ROOT.
                      - TXZ: for building Slackware TXZ's. Requires $TXZ_BUILD_ROOT.
  --striproot      : Strip path from custom layout (for package maintainers).
  --install        : Install according to chosen layout.
  --overwrite      : Overwrite the existing configuration file.
                     (Default is to create a separate configuration file.)
  --show           : Show chosen layout.
  --remove         : Uninstall according to chosen layout.
  --uninstall      : Alias for the '--remove' option.
  --version        : Show the installer version.
  ```

  We will use the --show to see where he will be installed  :
  ```shell
  interlope@deb12-template:~/rkhunter-1.4.6$ ./installer.sh  --show
Install into:       /usr/local
Application:        /usr/local/bin
Configuration file: /etc
Documents:          /usr/local/share/doc/rkhunter-1.4.6   (Directory will be created)
Man page:           /usr/local/share/man/man8             (Directory will be created)
Scripts:            /usr/local/lib/rkhunter/scripts     (Directory will be created)
Databases:          /var/lib/rkhunter/db        (Directory will be created)
Signatures:         /var/lib/rkhunter/db/signatures        (Directory will be created)
Temporary files:    /var/lib/rkhunter/tmp       (Directory will be created)
```

We can install rkhunter by using the `--install` option :
```shell
sudo ./installer.sh --install
Checking system for:
 Rootkit Hunter installer files: found
 A web file download command: wget found
Starting installation:
 Checking installation directory "/usr/local": it exists and is writable.
 Checking installation directories:
  Directory /usr/local/share/doc/rkhunter-1.4.6: creating: OK
  Directory /usr/local/share/man/man8: exists and is writable.
  Directory /etc: exists and is writable.
  Directory /usr/local/bin: exists and is writable.
  Directory /usr/local/lib: exists and is writable.
  Directory /var/lib: exists and is writable.
  Directory /usr/local/lib/rkhunter/scripts: creating: OK
  Directory /var/lib/rkhunter/db: creating: OK
  Directory /var/lib/rkhunter/tmp: creating: OK
  Directory /var/lib/rkhunter/db/i18n: creating: OK
  Directory /var/lib/rkhunter/db/signatures: creating: OK
 Installing check_modules.pl: OK
 Installing filehashsha.pl: OK
 Installing stat.pl: OK
 Installing readlink.sh: OK
 Installing backdoorports.dat: OK
 Installing mirrors.dat: OK
 Installing programs_bad.dat: OK
 Installing suspscan.dat: OK
 Installing rkhunter.8: OK
 Installing ACKNOWLEDGMENTS: OK
 Installing CHANGELOG: OK
 Installing FAQ: OK
 Installing LICENSE: OK
 Installing README: OK
 Installing language support files: OK
 Installing ClamAV signatures: OK
 Installing rkhunter: OK
 Installing rkhunter.conf: OK
Installation complete
```

If you got the following error, dont worry. We've simply installed rkhunter with sudo, so we will now need to use rkhunter as root / sudo group member.
```shell
interlope@deb12-template:~/rkhunter-1.4.6$ rkhunter
-bash: /usr/local/bin/rkhunter: Permission denied
```

## How to use rkhunter

We saw earlier that we can check the local system & the configuration files with `-c` and `-C` :
```shell
root@deb12-template:~# rkhunter -c
[ Rootkit Hunter version 1.4.6 ]

Checking system commands...

  Performing 'strings' command checks
    Checking 'strings' command                               [ OK ]

  Performing 'shared libraries' checks
    Checking for preloading variables                        [ None found ]
    Checking for preloaded libraries                         [ None found ]
    Checking LD_LIBRARY_PATH variable                        [ Not found ]

  Performing file properties checks
    Checking for prerequisites                               [ Warning ]
    /usr/local/bin/rkhunter                                  [ OK ]
    /usr/sbin/adduser                                        [ Warning ]
    /usr/sbin/chroot                                         [ OK ]
    /usr/sbin/cron                                           [ OK ]
    /usr/sbin/depmod                                         [ OK ]
    /usr/sbin/fsck                                           [ OK ]
    /usr/sbin/groupadd                                       [ OK ]
    /usr/sbin/groupdel                                       [ OK ]
    /usr/sbin/groupmod                                       [ OK ]
    /usr/sbin/grpck                                          [ OK ]
    /usr/sbin/ifconfig                                       [ OK ]
    /usr/sbin/ifdown                                         [ OK ]
    /usr/sbin/ifup                                           [ OK ]
    /usr/sbin/init                                           [ OK ]
    /usr/sbin/insmod                                         [ OK ]
    /usr/sbin/ip                                             [ OK ]
    /usr/sbin/lsmod                                          [ OK ]
    /usr/sbin/modinfo                                        [ OK ]
    /usr/sbin/modprobe                                       [ OK ]
    /usr/sbin/nologin                                        [ OK ]
    /usr/sbin/pwck                                           [ OK ]
    /usr/sbin/rmmod                                          [ OK ]
    /usr/sbin/route                                          [ OK ]
    /usr/sbin/runlevel                                       [ OK ]
    /usr/sbin/sshd                                           [ OK ]
    /usr/sbin/sulogin                                        [ OK ]
    /usr/sbin/sysctl                                         [ OK ]
    /usr/sbin/useradd                                        [ OK ]
    /usr/sbin/userdel                                        [ OK ]
    /usr/sbin/usermod                                        [ OK ]
    /usr/sbin/vipw                                           [ OK ]
    /usr/sbin/unhide                                         [ OK ]
    /usr/sbin/unhide-linux                                   [ OK ]
    /usr/sbin/unhide-posix                                   [ OK ]
    /usr/sbin/unhide-tcp                                     [ OK ]
    /usr/bin/awk                                             [ OK ]
    /usr/bin/basename                                        [ OK ]
    /usr/bin/bash                                            [ OK ]
    /usr/bin/cat                                             [ OK ]
    /usr/bin/chattr                                          [ OK ]
    /usr/bin/chmod                                           [ OK ]
    /usr/bin/chown                                           [ OK ]
    /usr/bin/cp                                              [ OK ]
    /usr/bin/curl                                            [ OK ]
    /usr/bin/cut                                             [ OK ]
    /usr/bin/date                                            [ OK ]
    /usr/bin/df                                              [ OK ]
    /usr/bin/diff                                            [ OK ]
    /usr/bin/dirname                                         [ OK ]
    /usr/bin/dmesg                                           [ OK ]
    /usr/bin/dpkg                                            [ OK ]
    /usr/bin/dpkg-query                                      [ OK ]
    /usr/bin/du                                              [ OK ]
    /usr/bin/echo                                            [ OK ]
    /usr/bin/egrep                                           [ Warning ]
    /usr/bin/env                                             [ OK ]
    /usr/bin/fgrep                                           [ Warning ]
    /usr/bin/file                                            [ OK ]
    /usr/bin/find                                            [ OK ]
    /usr/bin/fuser                                           [ OK ]
    /usr/bin/grep                                            [ OK ]
    /usr/bin/groups                                          [ OK ]
    /usr/bin/head                                            [ OK ]
    /usr/bin/id                                              [ OK ]
    /usr/bin/ip                                              [ OK ]
    /usr/bin/ipcs                                            [ OK ]
    /usr/bin/kill                                            [ OK ]
    /usr/bin/killall                                         [ OK ]
    /usr/bin/last                                            [ OK ]
    /usr/bin/lastlog                                         [ OK ]
    /usr/bin/ldd                                             [ Warning ]
    /usr/bin/less                                            [ OK ]
    /usr/bin/logger                                          [ OK ]
    /usr/bin/login                                           [ OK ]
    /usr/bin/ls                                              [ OK ]
    /usr/bin/lsattr                                          [ OK ]
    /usr/bin/lsmod                                           [ OK ]
    /usr/bin/lsof                                            [ OK ]
    /usr/bin/mail                                            [ OK ]
    /usr/bin/md5sum                                          [ OK ]
    /usr/bin/mktemp                                          [ OK ]
    /usr/bin/more                                            [ OK ]
    /usr/bin/mount                                           [ OK ]
    /usr/bin/mv                                              [ OK ]
    /usr/bin/netstat                                         [ OK ]
    /usr/bin/newgrp                                          [ OK ]
    /usr/bin/passwd                                          [ OK ]
    /usr/bin/perl                                            [ OK ]
    /usr/bin/pgrep                                           [ OK ]
    /usr/bin/ping                                            [ OK ]
    /usr/bin/pkill                                           [ OK ]
    /usr/bin/ps                                              [ OK ]
    /usr/bin/pstree                                          [ OK ]
    /usr/bin/pwd                                             [ OK ]
    /usr/bin/readlink                                        [ OK ]
    /usr/bin/runcon                                          [ OK ]
    /usr/bin/sed                                             [ OK ]
    /usr/bin/sh                                              [ OK ]
    /usr/bin/sha1sum                                         [ OK ]
    /usr/bin/sha224sum                                       [ OK ]
    /usr/bin/sha256sum                                       [ OK ]
    /usr/bin/sha384sum                                       [ OK ]
    /usr/bin/sha512sum                                       [ OK ]
    /usr/bin/size                                            [ OK ]
    /usr/bin/sort                                            [ OK ]
    /usr/bin/ssh                                             [ OK ]
    /usr/bin/stat                                            [ OK ]
    /usr/bin/strings                                         [ OK ]
    /usr/bin/su                                              [ OK ]
    /usr/bin/sudo                                            [ OK ]
    /usr/bin/tail                                            [ OK ]
    /usr/bin/telnet                                          [ OK ]
    /usr/bin/test                                            [ OK ]
    /usr/bin/top                                             [ OK ]
    /usr/bin/touch                                           [ OK ]
    /usr/bin/tr                                              [ OK ]
    /usr/bin/uname                                           [ OK ]
    /usr/bin/uniq                                            [ OK ]
    /usr/bin/users                                           [ OK ]
    /usr/bin/vmstat                                          [ OK ]
    /usr/bin/w                                               [ OK ]
    /usr/bin/watch                                           [ OK ]
    /usr/bin/wc                                              [ OK ]
    /usr/bin/wget                                            [ OK ]
    /usr/bin/whatis                                          [ OK ]
    /usr/bin/whereis                                         [ OK ]
    /usr/bin/which                                           [ OK ]
    /usr/bin/who                                             [ OK ]
    /usr/bin/whoami                                          [ OK ]
    /usr/bin/numfmt                                          [ OK ]
    /usr/bin/kmod                                            [ OK ]
    /usr/bin/systemd                                         [ OK ]
    /usr/bin/systemctl                                       [ OK ]
    /usr/bin/mawk                                            [ OK ]
    /usr/bin/bsd-mailx                                       [ OK ]
    /usr/bin/dash                                            [ OK ]
    /usr/bin/x86_64-linux-gnu-size                           [ OK ]
    /usr/bin/x86_64-linux-gnu-strings                        [ OK ]
    /usr/bin/inetutils-telnet                                [ OK ]
    /usr/bin/which.debianutils                               [ Warning ]
    /usr/lib/systemd/systemd                                 [ OK ]
    /etc/rkhunter.conf                                       [ OK ]

[Press <ENTER> to continue]


Checking for rootkits...

  Performing check of known rootkit files and directories
    55808 Trojan - Variant A                                 [ Not found ]
    ADM Worm                                                 [ Not found ]
    AjaKit Rootkit                                           [ Not found ]
    Adore Rootkit                                            [ Not found ]
    aPa Kit                                                  [ Not found ]
    Apache Worm                                              [ Not found ]
    Ambient (ark) Rootkit                                    [ Not found ]
    Balaur Rootkit                                           [ Not found ]
    BeastKit Rootkit                                         [ Not found ]
    beX2 Rootkit                                             [ Not found ]
    BOBKit Rootkit                                           [ Not found ]
    cb Rootkit                                               [ Not found ]
    CiNIK Worm (Slapper.B variant)                           [ Not found ]
    Danny-Boy's Abuse Kit                                    [ Not found ]
    Devil RootKit                                            [ Not found ]
    Diamorphine LKM                                          [ Not found ]
    Dica-Kit Rootkit                                         [ Not found ]
    Dreams Rootkit                                           [ Not found ]
    Duarawkz Rootkit                                         [ Not found ]
    Ebury backdoor                                           [ Not found ]
    Enye LKM                                                 [ Not found ]
    Flea Linux Rootkit                                       [ Not found ]
    Fu Rootkit                                               [ Not found ]
    Fuck`it Rootkit                                          [ Not found ]
    GasKit Rootkit                                           [ Not found ]
    Heroin LKM                                               [ Not found ]
    HjC Kit                                                  [ Not found ]
    ignoKit Rootkit                                          [ Not found ]
    IntoXonia-NG Rootkit                                     [ Not found ]
    Irix Rootkit                                             [ Not found ]
    Jynx Rootkit                                             [ Not found ]
    Jynx2 Rootkit                                            [ Not found ]
    KBeast Rootkit                                           [ Not found ]
    Kitko Rootkit                                            [ Not found ]
    Knark Rootkit                                            [ Not found ]
    ld-linuxv.so Rootkit                                     [ Not found ]
    Li0n Worm                                                [ Not found ]
    Lockit / LJK2 Rootkit                                    [ Not found ]
    Mokes backdoor                                           [ Not found ]
    Mood-NT Rootkit                                          [ Not found ]
    MRK Rootkit                                              [ Not found ]
    Ni0 Rootkit                                              [ Not found ]
    Ohhara Rootkit                                           [ Not found ]
    Optic Kit (Tux) Worm                                     [ Not found ]
    Oz Rootkit                                               [ Not found ]
    Phalanx Rootkit                                          [ Not found ]
    Phalanx2 Rootkit                                         [ Not found ]
    Phalanx2 Rootkit (extended tests)                        [ Not found ]
    Portacelo Rootkit                                        [ Not found ]
    R3dstorm Toolkit                                         [ Not found ]
    RH-Sharpe's Rootkit                                      [ Not found ]
    RSHA's Rootkit                                           [ Not found ]
    Scalper Worm                                             [ Not found ]
    Sebek LKM                                                [ Not found ]
    Shutdown Rootkit                                         [ Not found ]
    SHV4 Rootkit                                             [ Not found ]
    SHV5 Rootkit                                             [ Not found ]
    Sin Rootkit                                              [ Not found ]
    Slapper Worm                                             [ Not found ]
    Sneakin Rootkit                                          [ Not found ]
    'Spanish' Rootkit                                        [ Not found ]
    Suckit Rootkit                                           [ Not found ]
    Superkit Rootkit                                         [ Not found ]
    TBD (Telnet BackDoor)                                    [ Not found ]
    TeLeKiT Rootkit                                          [ Not found ]
    T0rn Rootkit                                             [ Not found ]
    trNkit Rootkit                                           [ Not found ]
    Trojanit Kit                                             [ Not found ]
    Tuxtendo Rootkit                                         [ Not found ]
    URK Rootkit                                              [ Not found ]
    Vampire Rootkit                                          [ Not found ]
    VcKit Rootkit                                            [ Not found ]
    Volc Rootkit                                             [ Not found ]
    Xzibit Rootkit                                           [ Not found ]
    zaRwT.KiT Rootkit                                        [ Not found ]
    ZK Rootkit                                               [ Not found ]

[Press <ENTER> to continue]


  Performing additional rootkit checks
    Suckit Rootkit additional checks                         [ OK ]
    Checking for possible rootkit files and directories      [ None found ]
    Checking for possible rootkit strings                    [ None found ]

  Performing malware checks
    Checking running processes for suspicious files          [ None found ]
    Checking for login backdoors                             [ None found ]
    Checking for sniffer log files                           [ None found ]
    Checking for suspicious directories                      [ None found ]
    Checking for suspicious (large) shared memory segments   [ None found ]
    Checking for Apache backdoor                             [ Not found ]

  Performing Linux specific checks
    Checking loaded kernel modules                           [ OK ]
    Checking kernel module names                             [ OK ]

[Press <ENTER> to continue]


Checking the network...

  Performing checks on the network ports
    Checking for backdoor ports                              [ None found ]

  Performing checks on the network interfaces
    Checking for promiscuous interfaces                      [ None found ]

Checking the local host...

  Performing system boot checks
    Checking for local host name                             [ Found ]
    Checking for system startup files                        [ Found ]
    Checking system startup files for malware                [ None found ]

  Performing group and account checks
    Checking for passwd file                                 [ Found ]
    Checking for root equivalent (UID 0) accounts            [ None found ]
    Checking for passwordless accounts                       [ None found ]
    Checking for passwd file changes                         [ None found ]
    Checking for group file changes                          [ None found ]
    Checking root account shell history files                [ OK ]

  Performing system configuration file checks
    Checking for an SSH configuration file                   [ Found ]
    Checking if SSH root access is allowed                   [ Not allowed ]
    Checking if SSH protocol v1 is allowed                   [ Warning ]
    Checking for other suspicious configuration settings     [ None found ]
    Checking for a running system logging daemon             [ Found ]
    Checking for a system logging configuration file         [ Found ]

  Performing filesystem checks
    Checking /dev for suspicious file types                  [ None found ]
    Checking for hidden files and directories                [ None found ]

[Press <ENTER> to continue]



System checks summary
=====================

File properties checks...
    Required commands check failed
    Files checked: 142
    Suspect files: 5

Rootkit checks...
    Rootkits checked : 496
    Possible rootkits: 0

Applications checks...
    All checks skipped

The system checks took: 2 minutes and 14 seconds

All results have been written to the log file: /var/log/rkhunter.log

One or more warnings have been found while checking the system.
Please check the log file (/var/log/rkhunter.log)
```

We can see a few *warnings* and will have to check the `/var/log/rkhunter.log`:
```shell
00:09:35] Warning: Checking for prerequisites               [ Warning ]
[00:09:35]          The file of stored file properties (rkhunter.dat) does not exist, and should be created. To do this type in 'rkhunter --propupd'.
[00:09:35] Info: The file properties check will still run as there are checks that can be performed without the 'rkhunter.dat' file.
[00:09:35]
[00:09:35] Warning: WARNING! It is the users responsibility to ensure that when the '--propupd' option
           is used, all the files on their system are known to be genuine, and installed from a
           reliable source. The rkhunter '--check' option will compare the current file properties
           against previously stored values, and report if any values differ. However, rkhunter
           cannot determine what has caused the change, that is for the user to do.
```
```shell
root@deb12-template:~# sudo cat /var/log/rkhunter.log | grep adduser
[00:09:36]   /usr/sbin/adduser                               [ Warning ]
[00:09:36] Warning: The command '/usr/sbin/adduser' has been replaced by a script: /usr/sbin/adduser: Perl script text executable
```
```shell
root@deb12-template:~# sudo cat /var/log/rkhunter.log | grep egrep
[00:09:39]   /usr/bin/egrep                                  [ Warning ]
[00:09:39] Warning: The command '/usr/bin/egrep' has been replaced by a script: /usr/bin/egrep: POSIX shell script, ASCII text executable
```
```shell
[00:09:39]   /usr/bin/fgrep                                  [ Warning ]
[00:09:39] Warning: The command '/usr/bin/fgrep' has been replaced by a script: /usr/bin/fgrep: POSIX shell script, ASCII text executable
```
```shell
root@deb12-template:~# sudo cat /var/log/rkhunter.log | grep ldd
[00:09:32] Info: Found the 'ldd' command: /usr/bin/ldd
[00:09:39]   /usr/bin/ldd                                    [ Warning ]
[00:09:39] Warning: The command '/usr/bin/ldd' has been replaced by a script: /usr/bin/ldd: Bourne-Again shell script, ASCII text executable
[00:09:46]   Checking for directory '/lib/ldd.so/bktools'    [ Not found ]
[00:09:57]   Checking for file '/lib/ldd.so/tks'             [ Not found ]
[00:09:57]   Checking for file '/lib/ldd.so/tkp'             [ Not found ]
[00:09:57]   Checking for file '/lib/ldd.so/tksb'            [ Not found ]
[00:09:58]   Checking for directory '/lib/ldd.so'            [ Not found ]
[00:11:37]     Checking for string '/lib/ldd.so/tkps'        [ Not found ]
```
```shell
root@deb12-template:~# sudo cat /var/log/rkhunter.log | grep which.debianutils
[00:09:42]   /usr/bin/which.debianutils                      [ Warning ]
[00:09:42] Warning: The command '/usr/bin/which.debianutils' has been replaced by a script: /usr/bin/which.debianutils: POSIX shell script, ASCII text executable
```
As scary it may looks, those warning are totally **OK**, it simply detects that some commands are scripts and not binaries, which is exactly what rkhunter should do, detect "abnormalities". Since everything's is **OK**, we can update rkhunter's database with ``--propupd``. Don't worry, some warning will persists, he's doing his job.
```shell
root@deb12-template:~# rkhunter --propupd
[ Rootkit Hunter version 1.4.6 ]
File created: searched for 180 files, found 142
```

## How does rkhunter really work ?
Rkhunter uses a local hash database which actually comes from our local binaries, from which it retrieves the signature at installation. He then compares them with the hashes of binaries present on the system. It can therefore detect modifications to each binary file. We had the following log during the installation : 

` Directory /var/lib/rkhunter/db: creating: OK`. 
```shell
root@deb12-template:~# ls /var/lib/rkhunter/db
backdoorports.dat  mirrors.dat       rkhunter.dat            signatures
i18n               programs_bad.dat  rkhunter_prop_list.dat  suspscan.dat
```
During rkhunter installation, it copies all the hashes of the binaries on your system and then compares them during checks. This is stored 'locally' on the system, which implies that the signatures are easily modifiable and therefore rkhunter's report can be biased, modified

Remember during the check, he was looking for some *rootkits* :
```shell
Jynx Rootkit                                             [ Not found ]
Jynx2 Rootkit                                            [ Not found ]
KBeast Rootkit                                           [ Not found ]
SHV4 Rootkit                                             [ Not found ]
SHV5 Rootkit                                             [ Not found ]
```

```shell
root@deb12-template:~# ls /var/lib/rkhunter/db/signatures/
RKH_BillGates.ldb  RKH_jynx.ldb          RKH_libncom.ldb        RKH_sniffer.ldb
RKH_dso.ldb        RKH_kbeast.ldb        RKH_MMD-0028-2014.ldb  RKH_sshd.ldb
RKH_Glubteba.ldb   RKH_libkeyutils1.ldb  RKH_pamunixtrojan.ldb  RKH_turtle.ldb
RKH_iptablex.ldb   RKH_libkeyutils.ldb   RKH_shv.ldb            RKH_xsyslog.ldb
```


What if I were to modify the `ls` command to add the `-42` option that would print something instead of listing ? Would rkhunter see it ?

## **Our own "rootkit"**

To do this, we'll need to download the coreutils sources : https://github.com/coreutils/coreutils to edit the `ls.c`, recompile it and replace the original `ls` binary with it.
Simply clone the git repository, install a few packages needed to to compile and execute the bootstrap file.
```
sudo apt install -y autotools-dev autoconf automake gettext bison gperf m4 texinfo texlive-latex-base dh-autoreconf make gcc
```

```shell
interlope@deb12-template:~/git$ git clone https://github.com/coreutils/coreutils.git
Cloning into 'coreutils'...
remote: Enumerating objects: 189899, done.
remote: Counting objects: 100% (1859/1859), done.
remote: Compressing objects: 100% (790/790), done.
remote: Total 189899 (delta 1161), reused 1092 (delta 1068), pack-reused 188040 (from 3)
Receiving objects: 100% (189899/189899), 46.32 MiB | 32.78 MiB/s, done.
Resolving deltas: 100% (145332/145332), done.
```

During the bootstrap, it'll clone some repos from *git.savannah.gnu.org* which may be very slow to download from. So I decided to simply clone the [gnulib](https://github.com/coreutils/gnulib.git) directly to my `~/git/coreutils` and then proceed to execute the bootstrap.
```shell
interlope@deb12-template:~/git/coreutils$ git clone --depth 1 https://github.com/coreutils/gnulib.git gnulib
Cloning into 'gnulib'...
remote: Enumerating objects: 12388, done.
remote: Counting objects: 100% (12388/12388), done.
remote: Compressing objects: 100% (6733/6733), done.
remote: Total 12388 (delta 7194), reused 6792 (delta 5634), pack-reused 0 (from 0)
Receiving objects: 100% (12388/12388), 10.81 MiB | 14.08 MiB/s, done.
Resolving deltas: 100% (7194/7194), done.
```
```shell
interlope@deb12-template:~/git/coreutils$ ./bootstrap
```

We will run a "pre-check" with `./configure` and `make` to be sure that it works without any modifications. This will give us *less* headache later on if something breaks.

The `./configure` shouldn't give you any errors, nor should the `make`
```shell
interlope@deb12-template:~/git/coreutils$ ./configure
[...]
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
config.status: creating po/Makefile.in
config.status: creating gnulib-tests/Makefile
config.status: creating lib/config.h
config.status: lib/config.h is unchanged
config.status: executing depfiles commands
config.status: executing po-directories commands
config.status: creating po/POTFILES
config.status: creating po/Makefile
```
```shell
interlope@deb12-template:~/git/coreutils$ make
[...]
CC       src/ls.o
[...]
CC       src/ls
[...]
make[4]: Leaving directory '/home/interlope/git/coreutils/gnulib-tests'
make[3]: Leaving directory '/home/interlope/git/coreutils/gnulib-tests'
make[2]: Leaving directory '/home/interlope/git/coreutils/gnulib-tests'
make[1]: Leaving directory '/home/interlope/git/coreutils'
```

Now we'll run a fun command which will help us identifying the ls files that are being compiled : `ls ls ls.*` 

What `ls ls ls.*` is doing :  `ls` (lists all files),`ls ls` (*lists all files* called `ls`) and `ls ls ls*.` (*lists all files called `ls`* that starts with `ls.`)

Let's get back to our little rootkit creation. We need to edit the ls.c file : `/git/coreutils/src/ls.c` and add our little -42 option between the `current_time.tv_nsec = -1;` and `i = decode_switches (argc, argv);` (approx *l.1715*) :
```c
  for (i = 1; i < argc; i++)
    {
      if (strcmp (argv[i], "-42") == 0)
        {
          printf ("This is a completely legit ls\n");
          exit (0);
        }
    }
```

We can now compile it with make.
```shell
interlope@deb12-template:~/git/coreutils$ make src/ls
  CC       src/ls.o
  CCLD     src/ls
```

It is time to test it ! Does the -42 works ? Does the `ls` work normally ? Are you sure you're not bluffing and it is not an alias ?
```shell
interlope@deb12-template:~/git/coreutils$ ./src/ls -42
This is a completely legit ls
```
```shell
interlope@deb12-template:~/git/coreutils$ ./src/ls -lah | grep src
drwxr-xr-x  4 interlope interlope  12K Sep 24 01:31 src
```
Yes. Yes and *yes*.
```shell
interlope@deb12-template:~/git/coreutils$ alias
alias ls='ls --color=auto'
```

We can now save the original `ls` present on our linux machine :
```shell
interlope@deb12-template:~/git/coreutils$ sudo mv /usr/bin/ls /usr/bin/ls.old
interlope@deb12-template:~/git/coreutils$ ls
-bash: /usr/bin/ls: No such file or directory
```
And replace with our modified `ls` :
```shell
interlope@deb12-template:~/git/coreutils$ sudo cp src/ls /usr/bin/
interlope@deb12-template:~/git/coreutils$ ls -42
This is a completely legit ls
```

Does **rkhunter** detect our little rootkit ? Yes
```shell
/usr/bin/ls                                              [ Warning ]
```
Let's check our /var/log/rkhunter.log :
```shell
[01:36:39]   /usr/bin/ls                                     [ Warning ]
[01:36:40] Warning: The file properties have changed:
[01:36:40]          File: /usr/bin/ls
[01:36:40]          Current hash: 098c8ddbd3c87abf4c713d80a4084a17cc477c13b010e698036aa6b3150f5e81
[01:36:40]          Stored hash : cb30d69b24245bf2ecdc9e7f53bbad19159999970b6d82c0c00c7d32d9e37aa4
[01:36:40]          Current inode: 838420    Stored inode: 784929
[01:36:40]          Current size: 662088    Stored size: 151344
[01:36:40]          Current file modification time: 1758670514 (24-Sep-2025 01:35:14)
[01:36:40]          Stored file modification time : 1663687647 (20-Sep-2022 17:27:27)
```

**Bingo !** Now that we've edited the ls.c file, the compiled version is different than the original one rkhunter has in memory.
we can check `/usr/bin/ls` hash that `rkhunter` is using here : `/var/lib/rkhunter/db/rkhunter.dat`, which should match the *stored hash* reported by rkhunter in the previous check.
```shell
root@deb12-template:~# cat /var/lib/rkhunter/db/rkhunter.dat | grep "/usr/bin/ls"
File:0:/usr/bin/ls:cb30d69b24245bf2ecdc9e7f53bbad19159999970b6d82c0c00c7d32d9e37aa4:784929:0755:0:0:151344:1663687647::0::
```
*We will need it later, keep it somewhere.*


## **How to bypass rkhunter**
Now that we've created our rootkit, implemented it, tested it and verified that rkhunter sees it, what if we could **bypass** rkhunter ? For that, a little explanation on how hash are calculated is needed.

We **do** know that rkhunter checks the hashes modifications. It doesn't run the binary or whatsoever, it simply calculates the SHA1/256 digest of each files. You may ask what's a digest ? It's the output of a hash function.

In this example :
```shell
interlope@deb12-template:~$ echo -n "tata" | sha256sum
d1c7c99c6e2e7b311f51dd9d19161a5832625fb21f35131fba6da62513f0c099  -
```
- `tata` is the **input**
- `sha256sum` is the hash **function**
- `d1c7c99c6e2e7b311f51dd9d19161a5832625fb21f35131fba6da62513f0c099` is the **digest**

In the same way that we added our little function to `ls.c`, who said we couldn't add another one to `digest.c`, which happens to be the module that calculates the SHA-XXX hash for files ?
<br>We will need to get the stored hash value of `/usr/bin/ls`, which is `cb30d69b24245bf2ecdc9e7f53bbad19159999970b6d82c0c00c7d32d9e37aa4`.

```shell
for (int i=0; i<argc; i++) 
{
  if ( strcmp(argv[i], "/usr/bin/ls") == 0 ) {
  printf("cb30d69b24245bf2ecdc9e7f53bbad19159999970b6d82c0c00c7d32d9e37aa4\n");
  exit(0);
  }
}
```

We have to recompile ``sha256sum``, which I remind you, is based of `digest.c` :
```shell
interlope@deb12-template:~/git/coreutils$ make src/sha256sum
```

So if everything went well before, the sha256sum hash of `/usr/bin/ls` should be the "old one" while it's still the modified one:
```shell
interlope@deb12-template:~/git/coreutils$ ./src/sha256sum /usr/bin/ls
cb30d69b24245bf2ecdc9e7f53bbad19159999970b6d82c0c00c7d32d9e37aa4
```
```shell
interlope@deb12-template:~/git/coreutils$ /usr/bin/ls -42
This is a completely legit ls
```

We should now replace the new sha256sum module we just compiled and test it:
```shell
interlope@deb12-template:~/git/coreutils$ sudo mv /bin/sha256sum /bin/sha256sum.old
interlope@deb12-template:~/git/coreutils$ sudo cp src/sha256sum /bin/
```
```shell
interlope@deb12-template:~/git/coreutils$ sha256sum /usr/bin/ls
cb30d69b24245bf2ecdc9e7f53bbad19159999970b6d82c0c00c7d32d9e37aa4
```

What does `rkhunter` have to say ? Perhaps it is okay ? Perhaps another warning ? Let's check the logs.
```shell
interlope@deb12-template:~/git/coreutils$ sudo rkhunter -c
 /usr/bin/ls                                              [ Warning ]
 /usr/bin/sha256sum                                       [ Warning ]

[02:16:24]   /usr/bin/ls                                     [ Warning ]
[02:16:24] Warning: The file properties have changed:
[02:16:24]          File: /usr/bin/ls
[02:16:24]          Current inode: 838420    Stored inode: 784929
[02:16:24]          Current size: 662088    Stored size: 151344
[02:16:24]          Current file modification time: 1758670514 (24-Sep-2025 01:35:14)
[02:16:24]          Stored file modification time : 1663687647 (20-Sep-2022 17:27:27)
 ```
**VICTORY !** I mean, a half assed victory, `rkhunter` still see that it somehow got modified and that `sha256sum` was also modified. But we could also hide it behind the fact that a package update would indeed change the hashes / size of the files.

## **Conclusion**
This entire article was to make you understand the limitations of a signature-based tool and that you cannot trust anything on a compromised machine. You can confirm that a machine is infected, but you cannot claim that a machine is clean. You can offer a “guarantee” with a certain level of confidence by establishing a chain of security, but you cannot guarantee with 100% certainty that a machine is clean and safe. 

Thanks for reading me, 
<br>spleenftw


## **Bonus**
If we truly wanted to cheat on `rkhunter`, we could add another function to the `digest.c` that would change the `sha256sum` hash so that rkhunter wouldn't the `sha256sum` modification.