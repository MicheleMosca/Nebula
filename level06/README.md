# Level 06

The **flag06** account credentials came from a legacy unix system.

To do this level, log in as the **level06** account with the password **level06**. Files for this level can be found in /home/flag06.

## Source Code

There is no source code available for this level.

## Writeup

The initial hint suggest that the **flag06** account credentials came from a legacy unix system. Let's grep flag06 line of **/etc/passwd** file:

```bash
grep flag06 /etc/passwd
```

This is the output:

```bash
level06@nebula:/home/flag06$ grep flag06 /etc/passwd
flag06:ueqwOCnSGdsuM:993:993::/home/flag06:/bin/sh
```

There is the hash of the password, in the new systems the second parameter is an '**x**' that means that the hash of the password is in the **/etc/shadow** file.

We can use the **hashid** command to dicover what kind of hash is this:

**NB:** you can install it with the command: `sudo apt install hashid`

```bash
hashid -m ueqwOCnSGdsuM
```

This is the output:

```
Analyzing 'ueqwOCnSGdsuM'
[+] DES(Unix) [Hashcat Mode: 1500]
[+] Traditional DES [Hashcat Mode: 1500]
[+] DEScrypt [Hashcat Mode: 1500]
```

The **-m** give to us the hashcat module number to crack the hash.
Write the hash to a file named **flag06.hash** in your local machine: 

```bash
echo ueqwOCnSGdsuM > flag05.hash
```

Now use **hashcat** to crack the password with the **rockyou** dictionary:

**NB:** the rockyou dictionary can be download at: https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt

**NB:** you can install hashcat with the command: `sudo apt install hashcat` 

```bash
hashcat -a 0 -m 1500 flag05.hash rockyou.txt
```

The **-a** argument means that we use the dictionary attack mode.

This is the output:

```
Dictionary cache built:
* Filename..: rockyou.txt
* Passwords.: 14344391
* Bytes.....: 139921497
* Keyspace..: 14344384
* Runtime...: 1 sec

ueqwOCnSGdsuM:hello                                       
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1500 (descrypt, DES (Unix), Traditional DES)
Hash.Target......: ueqwOCnSGdsuM
Time.Started.....: Tue Mar 14 23:27:37 2023 (0 secs)
Time.Estimated...: Tue Mar 14 23:27:37 2023 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  5360.5 kH/s (5.79ms) @ Accel:256 Loops:1 Thr:64 Vec:1
Recovered........: 1/1 (100.00%) Digests
Progress.........: 58401/14344384 (0.41%)
Rejected.........: 9249/58401 (15.84%)
Restore.Point....: 0/14344384 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: 123456 -> karely
Hardware.Mon.#1..: Temp: 51c Util:  1% Core:1084MHz Mem:2000MHz Bus:4
```

Now we know the password of **flag06** user account.
Let's login with this credentials:

```
sh-4.2$ id
uid=993(flag06) gid=993(flag06) groups=993(flag06)
```

We can run the **getflag** program:

```
sh-4.2$ getflag
You have successfully executed getflag on a target account
```