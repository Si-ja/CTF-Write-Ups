# picoCTF | PW Crack 3

## Basics:
* CTF from: [picoCTF](https://picoctf.org/)
* Name: PW Crack 3
* Author: LT 'syreal' Jones
* OS used for solution: Linux Mint

### Description of the CTF provided by the author (with original links)
Can you crack the password to get the flag? Download the password checker [here](https://artifacts.picoctf.net/c/17/level3.py) and you'll need the encrypted [flag](https://artifacts.picoctf.net/c/17/level3.flag.txt.enc) and the [hash](https://artifacts.picoctf.net/c/17/level3.hash.bin) in the same directory too. There are 7 potential passwords with 1 being correct. You can find these by examining the password checker script.

### Hints provided
>! [1] To view the level3.hash.bin file in the webshell, do: $ bvi level3.hash.bin

>! [2] To exit bvi type :q and press enter.

>! [3] The str_xor function does not need to be reverse engineered for this challenge.

## Solution

### Quick Note in the beginning
A python file provided with the CTF holds final lines that read:

```python
# The strings below are 7 possibilities for the correct password. 
#   (Only 1 is correct)
pos_pw_list = ["f09e", "4dcf", "87ab", "dba8", "752e", "3961", "f159"]
```

A lot of approaches that I have seen just show to try all 7 passwords in a row, or modify the script to make the python code test all 7 passwords programmatically. 

However, this write up focuses on showing how the CTF was intended to be solved, where you can know on your own for sure which password will work in your case from 1 attempt.

### Breakdown

We have access to 3 files:

1. A python script
2. Some kind of binary
3. An encrypted flag file

When taking a look at the python file we see the following script:

```python
import hashlib

### THIS FUNCTION WILL NOT HELP YOU FIND THE FLAG --LT ########################
def str_xor(secret, key):
    #extend key to secret length
    new_key = key
    i = 0
    while len(new_key) < len(secret):
        new_key = new_key + key[i]
        i = (i + 1) % len(key)        
    return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c,new_key_c) in zip(secret,new_key)])
###############################################################################

flag_enc = open('level3.flag.txt.enc', 'rb').read()
correct_pw_hash = open('level3.hash.bin', 'rb').read()


def hash_pw(pw_str):
    pw_bytes = bytearray()
    pw_bytes.extend(pw_str.encode())
    m = hashlib.md5()
    m.update(pw_bytes)
    return m.digest()


def level_3_pw_check():
    user_pw = input("Please enter correct password for flag: ")
    user_pw_hash = hash_pw(user_pw)
    
    if( user_pw_hash == correct_pw_hash ):
        print("Welcome back... your flag, user:")
        decryption = str_xor(flag_enc.decode(), user_pw)
        print(decryption)
        return
    print("That password is incorrect")



level_3_pw_check()


# The strings below are 7 possibilities for the correct password. 
#   (Only 1 is correct)
pos_pw_list = ["f09e", "4dcf", "87ab", "dba8", "752e", "3961", "f159"]

```

The file seems to have been designed to be executed as indicated by the `level_3_pw_check()` function that is called at the end of it. For future, let's prepare it now to be executable as currently we do not have an ability to run it in the terminal

```shell
chmod +x ./level3.py
```

Regarding the contents of the file. The main function that is called is `level_3_pw_check()` and in it we see that a password from the user will be requested that will be hashed with `hash_pw(string)` function.

Investigating the `hash_pw(string)` we find clues to how the hashing will be produced. Simplifying it, the `md5` (message-digest algorithm) will be used. At this point to understand what will happen further, it is very advisable to get familiar with the algorithm itself and understand the basics of how it works. [Take a look at one of the sources that has a short description of the algorithm and one you can utilize yourself online](https://www.md5.cz/). The main point of the algorithm to understand to move further is that (quoting the previous source) "The MD5 hash is 128 bits long and is represented by 32 characters.".

Taking a look further into the code, when the function hashes the user inputted string, it is simply compared to another variable as seen in `if( user_pw_hash == correct_pw_hash ):` and if the comparison is successful - the flag will be unecrypted and provided to us. But what is this `correct_pw_hash`?

Correct password hash comes from the `correct_pw_hash = open('level3.hash.bin', 'rb').read()` and seems to be just a plain reading of the binary file.

By the current logic, we simply need to read data from the binary file, and compare it to 7 passwords, that we can hash ourselves with `md5` and we will get information which password works.

Reading the binary file doesn't yield too much. We just get a string `ámU¥]ÝÝRš>«êW+{`. But code in python doesn't read the file directly as well, but uses the `rb` argument, which in the `open()` function documentation reads: open for reading (default) + binary mode. So the file needs to be opened as a binary to gain some knowledge on it. In linux this can be done in multiple ways so millage further may vary, but I used a `hexdump` with `-C` flag to open it. `hexdump` is described in the manual as: "hexdump - display file contents in hexadecimal, decimal, octal, or ascii" with the `-C` flag as "Canonical hex+ASCII display". There's reason to reading the data into a hexadecimal value, because that's exactly how the `md5` produces hashes as well. This way we will be able to compare the data of one to another. By running

```shell
hexdump -C ./level3.hash.bin
```

you will get an output: 
```terminal
00000000  e1 6d 55 a5 5d 80 dd dd  52 a8 3e ab ea 57 2b 7b  |.mU.]...R.>..W+{|
00000010
```

which looks like gibberish, but what trully matters is the middle part, where you have 16 pairs of hex values. Each hex value is `4-bits` large. But here we have pairs, which makes them `8-bits`. `8 x 16 = 128` which is the exact size of what the `md5` hash produces. So we can at least confirm that the binary makes sense and should yield the results we are looking for.

But what is thaat `|.mU.]...R.>..W+{|` in the end of the file? I will be honest, there is a reason for it to be there, but we thankfully don't need to focus on it for now.

### Automation

So we want to automate the process of finding the correct password. Why? Because in our case we have 7 passwords, what if we had 1 million of them? Therefore we can prepare the following shell script(s) and get the results we are looking for.

1) Put all of our known passowrds into an array of variables.

2) Iterate over each password value and call an `md5` hashing algorithm on it. Linux Mint has an inbuilt functionality for it, so I'm sure other version will have some alternative too.

3) The hashshes will be prepared as unbroken 32 digit strings, so we want to break them into parts, where each 2 digits are separated by a space, but this is mostly semantics to help with the future look up for convenience.

4) Save all of the prepared hashes into a file, say `./hashes3.txt` that you will prepare upfront.

Let's put all of that into practice for now.


```shell
# Create a file for storing hashes
touch ./hashes3.txt

# Assign into a variable all known passowrds
declare -a TESTDATA=('f09e' '4dcf' '87ab' 'dba8' '752e' '3961' 'f159')

# Iterate over all of the passwords hashing them and write them into a file
for code in "${TESTDATA[@]}"; do 
echo -n $code | md5sum | fold -w2 | paste -sd ' ' - | cat >> ./hashes3.txt; 
done
```

you should get a collection like this:

```
dd d2 43 ee c9 1f a3 c3 03 da a5 f8 31 91 17 94    -
a3 63 03 46 41 70 da fe 35 d3 fc 76 e9 b2 88 4c    -
e1 6d 55 a5 5d 80 dd dd 52 a8 3e ab ea 57 2b 7b    -
65 d9 c6 8e 03 80 79 69 85 1a 83 b2 8b be be d1    -
14 c9 37 2d 1f d1 1e da 1c 0c 8c bb be f4 5f fc    -
24 e0 18 30 d2 13 d7 5d eb 99 c2 2b 9c d9 1d dd    -
86 fe 19 c3 2c 33 69 7f cf 3e bb 3c 34 26 9d 6b    -
```

5) What is left is to compare the produced hashes with the one we found in the original binary folder. For this case it's not hard, but say you'd be dealing with a million of hashes - that would be convenient to be automated. So use a grep function for this:

```shell
# Note that in the original string after the 8th value there will be a double space that needs to be deleted. Alternative would have been to operate without spaces and instead of creating spaces in the previous file, to exclude them in this one.
# -n flag will show you on which line is the correct password
grep "e1 6d 55 a5 5d 80 dd dd 52 a8 3e ab ea 57 2b 7b" ./hashes3.txt -n
```

6) You will get an output:

```
3:e1 6d 55 a5 5d 80 dd dd 52 a8 3e ab ea 57 2b 7b    -
```

7) 3 indicates that your 3rd line holds the correct passwords. Lines start from 1, so third password you hashed was the correct. Since you still have the variable in which you stored all of your passwords, you can reuse it by just calling what is located at position `Line number - 1` since we are operating with indexes from zero.

```shell
echo ${TESTDATA[2]}
```

Your output will be `87ab`. Now use that on your python file.

```shell
python3 ./level3.py
```

You will be prompted to enter the password `please enter correct password for flag: 87ab`.

And as the result you will get the following flag:

`picoCTF{m45h_fl1ng1ng_cd6ed2eb}`

## Conclusion

Thank you for playing :3
