---
layout: post
title: Common Vulnerabilities in C 
category: [programming]
tags: [c,programming, vulnerability, coursera, week1]
---

Common vulnerabilities in C. 

Buffer Overflow. 

1. Any access of a buffer outside of its alloted bounds
- Over-read
- Over-write 
- During iteration 
- Direct access, pointer arithmetic for example. 
- Out of bounds access could be to addresses that precede or follow the buffer. 

In order to understand the Buffer Overflow attack, it is important to understand the *Memory Layout* 

All programs are stored in memory. 

Location of data areas

set when process starts| cmdline &env  | 0xffffffff
                       | ------------- |
                /------| Stack         | int f() { int x; .... } 
     Runtime-->/-------| ------------- |
              /--------| Heap          | malloc (sizeof(long)); 
                       | ------------- |
          Known /------| Uninit'd data | static int y;
  at compile-->/-------| init'd data   | static const int y = 11
         time /--------| Text          | 0x00000000
         
         
         
         | Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
         
         

Vulnerabilities in C are related to buffer overflows and string manipulation. This would result in a segmentation fault. 

Here are some common errors and the suggested solutions. 

gets

```
char *gets(char *str);
```
The stdio gets() function does not check for buffer length and always results in a vulnerability. 

From man gets 

The gets() function is equivalent to fgets() with an infinite size and a stream of stdin, except that the newline character 
(if any) is not stored in the string.  It is the caller's responsibility to ensure that the input line, if any, is sufficiently 
short to fit in the string.

Example of vulnerable code: 

