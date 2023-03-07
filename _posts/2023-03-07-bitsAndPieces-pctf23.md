---
title: bitsAndPieces pragyanCTF23
author: nautilus
date: 2023-03-07 21:30:30 +0530
categories: [Writeups, Reversing]
tags: [cpp, IDA, pragyanCTF23]
---

This challenge is a rather simple challenge in which there are two `flagcheck` functions, each checking the first 10 and last 10 characters respectively after encoding the two parts of flag with hardcoded values.

`encode1` function encodes first part of the flag. It reverses the bits of each element and the calls `myfunc` which is actually returns `xor` of its arguments.

```cpp
  //encode1
  for ( i = 0; i <= 9; ++i ){
    *(_BYTE *)(i + input) = reverse((unsigned int)*(char *)(i + input));
    *(_BYTE *)(i + input) = myfunc(32LL, (unsigned int)*(char *)(i + input));
  }
```

`encode2` takes `xor` of `i`<sup>`th`</sup> with `i+1`<sup>`th`</sup> element and stores in `i`<sup>`th`</sup> element.

```cpp
*(_BYTE *)(i + input) = myfunc((unsigned int)*(char *)(i + 1LL + input), (unsigned int)*(char *)(i + input));

```

## Solve Script

Here is the solve script:-

```python
p1 = [58, 44, 110, 218, 172, 238, 218, 46, 44, 206]

flag_part1= ""
for i in range(len(p1)):
    p1[i] = p1[i]^32
    flag_part1 += chr(int(bin(p1[i])[2:][::-1].ljust(8, '0') ,2))


p2 = [65, 20, 19, 25, 51, 45, 65, 115, 113, 49][::-1]

flag_part2=chr(p2[0])
for i in range(1,len(p2)):
    p2[i] = p2[i] ^ p2[i-1]
    flag_part2 += chr(p2[i])
    
print(flag_part1+flag_part2[::-1])
# X0r_1s_p0w3rful_r3@1

```


## Flag

Wrap it up in flag format and we have the flag
`p_ctf{X0r_1s_p0w3rful_r3@1}`