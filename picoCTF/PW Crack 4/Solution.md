# picoCTF | PW Crack 4

## Basics:
* CTF from: [picoCTF](https://picoctf.org/)
* Name: PW Crack 4
* Author: LT 'syreal' Jones
* OS used for solution: Linux Mint

### Description

Can you crack the password to get the flag? Download the password checker [here](https://artifacts.picoctf.net/c/19/level4.py) and you'll need the encrypted [flag](https://artifacts.picoctf.net/c/19/level4.flag.txt.enc) and the [hash](https://artifacts.picoctf.net/c/19/level4.hash.bin) in the same directory too. There are 100 potential passwords with only 1 being correct. You can find these by examining the password checker script.

### Hints provided

>! [1] A for loop can help you do many things very quickly.

>! [2] The str_xor function does not need to be reverse engineered for this challenge.

## Solution

## Quick Note in the beginning
The solution follows pretty much the step by step method in `PW Crack 3` CTF, so for that logic, please look into it. The only thing this one adds is simplifies the way how to read a 100 potential passwords into a variable in a shell.

### Breakdown
This time we are operating with 100 passwords. 

```python
# The strings below are 100 possibilities for the correct password. 
#   (Only 1 is correct)
pos_pw_list = ["6288", "6152", "4c7a", "b722", "9a6e", "6717", "4389", "1a28", "37ac", "de4f", "eb28", "351b", "3d58", "948b", "231b", "973a", "a087", "384a", "6d3c", "9065", "725c", "fd60", "4d4f", "6a60", "7213", "93e6", "8c54", "537d", "a1da", "c718", "9de8", "ebe3", "f1c5", "a0bf", "ccab", "4938", "8f97", "3327", "8029", "41f2", "a04f", "c7f9", "b453", "90a5", "25dc", "26b0", "cb42", "de89", "2451", "1dd3", "7f2c", "8919", "f3a9", "b88f", "eaa8", "776a", "6236", "98f5", "492b", "507d", "18e8", "cfb5", "76fd", "6017", "30de", "bbae", "354e", "4013", "3153", "e9cc", "cba9", "25ea", "c06c", "a166", "faf1", "2264", "2179", "cf30", "4b47", "3446", "b213", "88a3", "6253", "db88", "c38c", "a48c", "3e4f", "7208", "9dcb", "fc77", "e2cf", "8552", "f6f8", "7079", "42ef", "391e", "8a6d", "2154", "d964", "49ec"]
```

The steps to resolve this task are the same as in the PW Crack 3, so that part will be ommitted. The only thing that we want to do differently is not write the passwords ourselves into the terminal, but `copy -> clean -> paste` them into a variable to process the hashes for.

Essentially we need to turn a `["string", "string"]` style into a `('string' 'string')` for convenience or convert a string of items separated with a space into an array variable.

1. Replace square brackets with round.
2. Replace double quotes with single.
3. Remove commas.

Here is a convenient shell script for that:

```shell
# https://ioflood.com/blog/bash-split-string-into-array/
TEMP_TESTDATA="["6288", "6152", "4c7a", "b722", "9a6e", "6717", "4389", "1a28", "37ac", "de4f", "eb28", "351b", "3d58", "948b", "231b", "973a", "a087", "384a", "6d3c", "9065", "725c", "fd60", "4d4f", "6a60", "7213", "93e6", "8c54", "537d", "a1da", "c718", "9de8", "ebe3", "f1c5", "a0bf", "ccab", "4938", "8f97", "3327", "8029", "41f2", "a04f", "c7f9", "b453", "90a5", "25dc", "26b0", "cb42", "de89", "2451", "1dd3", "7f2c", "8919", "f3a9", "b88f", "eaa8", "776a", "6236", "98f5", "492b", "507d", "18e8", "cfb5", "76fd", "6017", "30de", "bbae", "354e", "4013", "3153", "e9cc", "cba9", "25ea", "c06c", "a166", "faf1", "2264", "2179", "cf30", "4b47", "3446", "b213", "88a3", "6253", "db88", "c38c", "a48c", "3e4f", "7208", "9dcb", "fc77", "e2cf", "8552", "f6f8", "7079", "42ef", "391e", "8a6d", "2154", "d964", "49ec"]"
TEMP_TESTDATA=$(echo $TEMP_TESTDATA | tr -d "[]")
TEMP_TESTDATA=$(echo $TEMP_TESTDATA | tr -d "")
IFS=',' read -a TESTDATA <<< $TEMP_TESTDATA
```

## Conclusion

Continue further using `$TESTDATA` as a variable per previous solution steps and you should get your flag. =^_^=