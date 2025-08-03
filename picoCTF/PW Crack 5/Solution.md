# picoCTF | PW Crack 5

## Basics:
* CTF from: [picoCTF](https://picoctf.org/)
* Name: PW Crack 5
* Author: LT 'syreal' Jones
* OS used for solution: Linux Mint

### Description

Can you crack the password to get the flag? Download the password checker [here](https://artifacts.picoctf.net/c/33/level5.py) and you'll need the encrypted [flag](https://artifacts.picoctf.net/c/33/level5.flag.txt.enc) and the [hash](https://artifacts.picoctf.net/c/33/level5.hash.bin) in the same [directory](https://artifacts.picoctf.net/c/33/dictionary.txt) too. Here's a dictionary with all possible passwords based on the password conventions we've seen so far.

### Hints provided

>! [1] Opening a file in Python is crucial to using the provided dictionary.

>! [2] You may need to trim the whitespace from the dictionary word before hashing. Look up the Python string function, strip.

>! [3] The str_xor function does not need to be reverse engineered for this challenge.

## Solution

## Quick Note in the beginning

At this point I'm starting to feel they wanted us to do everything with Python from the get go, but I also feel a part of the learning experience would have been lost going directly into it. So we are continuing doing everything with linux directly.

As well, this is just a continuation from `PW Crack 4` wright-up. More details can be found in it where the logic starts for cracking this process. This one only variates on the fact that we are now dealing with an external file with a lot of passwords and how we can handle that.

### Breakdown

Now we have arround 65.5k passwords to deal with that are stored in an additional file outside of our python one. Everything that was applied in `PW Crack 4` & `PW Crack 3` applies. But to simplify the process, what we can do is read information from the file holding all of the passwords, but in a streaming fashion not to load everything into a variable, and produce as before a file with hashes that we can later look up.

One way we can do it is with the following script:

```shell
touch ./hashes5

cat ./dictionary.txt | while read LINE; do
echo -n $LINE | md5sum | fold -w2 | paste -sd ' ' - | cat >> ./hashes5;
done
```

An important part in the above script is the flag `-n` place after `echo`. It enforces not outputting a trailing newline at the end of the string. If that is not done, the hashes will be prepared much differently. Also, this process might take a while.

After 65.5K hashes will be prepared, follow the old approach of finding on which string our searched for hash is located and locate it.

```shell
hexdump -C ./level5.hash.bin
```

Leads to

```terminal
00000000  0f 42 38 73 59 16 de a9  ba c9 b6 a7 98 24 22 3b  |.B8sY........$";|
00000010
```

```shell
grep "0f 42 38 73 59 16 de a9 ba c9 b6 a7 98 24 22 3b" ./hashes5 -n
```

We get

```terminal
61153:0f 42 38 73 59 16 de a9 ba c9 b6 a7 98 24 22 3b    -

```

This means our password in the `dictionary.txt` is located on the 61153th line. And that is `eee0`.

Use this password when calling to an execution of the python script you have an access to.