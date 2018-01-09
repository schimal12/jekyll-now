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
 
         
                                | ------------- |
        set when process starts | cmdline &env  |  0xffffffff
                                | ------------- |
                         /----- | Stack         | int f() { int x; .... } 
              Runtime-->/------ | ------------- |
                       /------- | Heap          | malloc (sizeof(long)); 
                                | ------------- |
                   Known /----- | Uninit'd data | static int y;
                                | ------------- |
           at compile-->/------ | init'd data   | static const int y = 11
                                | ------------- |
                  time /------- | Text          | 0x00000000
                                | ------------- |
                  
 
 
In the memory allocation the stack and the heap grow in opposite directions. So the compiler will emit instructions and adjust the size of the stack at run-time. 

Example
  
     | 0x00000000    |               |               |               |               | 0xffffffff   |
     | ------------- | ------------- | ------------- | ------------- | ------------- |------------- |
     | ------------- | ------------- | ------------- | ------------- | ------------- |------------- |
     | ------------- | ------------- | ------------- | ------------- | ------------- |------------- |
     | heap          |               |  3            | 2             | 1             | Stack        |
     | ------------- | ------------- | ------------- | ------------- | ------------- |------------- |
     | ------------- | ------------- | ------------- | ------------- | ------------- |------------- |
     | ------------- | ------------- | ------------- | ------------- | ------------- |------------- |


     | By Malloc     |               |  push 3       | push 2        | push 1        | StackPointer | 
     | ------------- | ------------- | ------------- | ------------- | ------------- |------------- |

====== 

- push 1 
- push 2 
- push 3 
- return 

======

This would be the order in which the instructions enter the stack, since the arguments are pushed in reverse order of the code. 
                               
                               
Summary of Stack and functions: 

1. Calling Function
- Push arguments onto the stack (in reverse). 
- Push the return address, for example the address of the instructions you want to run after control returns to you. 
- Jump to the function address 




                                    


                                                                                 
                                                                                 
Vulnerabilities in C are related to buffer overflows and string manipula tion. This would result in a segmentation fault. 

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

