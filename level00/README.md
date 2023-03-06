# Level 00

This level requires you to find a Set User ID program that will run as the `flag00` account. You could also find this by carefully looking in top level directories in / for suspicious looking directories.

Alternatively, look at the find man page.

To access this level, log in as level00 with the password of level00.

## Source code

There is no source code available for this level.

## Writeup

Let's find all binary of flag00 account with SUID:
```bash
find / -user flag00 -perm +4000 2> /dev/null
```

There are two binary:
```
/bin/.../flag00
/rofs/bin/.../flag00
```

Execute the first:
```
/bin/.../flag00
```

This is the output:
```
Congrats, now run getflag to get your flag!
```

Run `getflag` command:
```bash
getflag
```

Output:
```
You have successfully executed getflag on a target account
```