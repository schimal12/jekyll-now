---
layout: post
title: Common Vulnerabilities in C 
category: [programming]
tags: [c,programming, vulnerability, coursera, week1]
---

## Memory Layout

# Location of data areas
 
         
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
                  time /------- | Text          | Code
                                | ------------- | 0x00000000
                  
 
It is important to notice that (0x00000000 and 0xffffffff) are virtual addresses and OS map them to physical addresses. 
 
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


It is important to notice that the Stack Pointer (%esp) moves as each variable is pushed into the stack. 
So at the beginning, the Stack Pointer is in its original position, after the instruction push 1 is entered, the stack pointer will point to the memory address of "1" and so on, the same concept is applied for every "push". 

Example of the movement of the Stack Pointer  

====== 
-----------> Stack Pointer (%esp)

======

====== 

- push 1 
-----------> Stack Pointer(%esp)

======

====== 

- push 1 
- push 2
-----------> Stack Pointer(%esp)

======

====== 

- push 1 
- push 2
- push 3
-----------> Stack Pointer(%esp)

======                          
                   

Now, assuming that the function reach a return in the code, what the function is going to do is to "pop" all the variables into the stack and set the Stack Pointer to the original position, just before the function started to push all the variables. 

# Example of Stack Layout with code. 

```
void func(char \*arg1, int arg2, int arg3){
   char localV1[4];
   int  localV2;
}
```
            
            
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- | 0xffffffff   |
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- | ------------ |
     | localv2       | localv1       | ??????? | args1         | args2         | args3         | CallersData  |     
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- | ------------ |
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- | ------------ |


     
As we can see here, the arguments of the function "func" are pushed in reverse order (arg3, arg2, arg1). But the local variables are pushed in the same order as they appear in the code. 
For the variable allocation, they are up to the compiler, they can be allocated in random order or stored in the registers ... it really depends of the level of optimization of the compiler. 

# Accessing the variables. 

```
void func(char \*arg1, int arg2, int arg3){
   ....
   local2++;
   ...
}
```
            
            
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- | 0xffffffff   |
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- | ------------ |
     | localv2       | localv1       | ??????? | args1         | args2         | args3         | CallersData  |     
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- | ------------ |
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- | ------------ |

Before explaining, it is important to know the concept of Stack Frame. 

     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- |
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- |
     | localv2       | localv1       | ??????? | args1         | args2         | args3         | 
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- | 
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- |
     
     This is the Stack Frame for function "func". It contains the arguments, the local variables and ???? that represents data. 

Now, continuing the explanation of how to access variables, we need to notice that the memory address for "localv2" will be different, depending on who call the function. 

So we use a relative address. This relative address is indicated with the Frame Pointer (%ebp), for example: 


     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- |
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- |
     | localv2       | localv1       | ??????? | args1         | args2         | args3         | 
     | ------------- | ------------- | %ebp    | ------------- | ------------- | ------------- | 
     | ------------- | ------------- | ------- | ------------- | ------------- | ------------- |

     
Here, we will know that the "localv2" variable is 8 bytes distant from the %ebp. So in this case, the variable "localv2" will be at -8(%ebp)

# Returning from functions. 




Summary of Stack and functions: 

1. Calling Function
- Push arguments onto the stack (in reverse). 
- Push the return address, for example the address of the instructions you want to run after control returns to you. 
- Jump to the function address 

2. Called function 
- Push the old frame pointer onto the stack (%ebp) 
- Set the frame pointer (%ebp) to where the end of the stack is right now (%esp)
- Push local variables onto the stack 

3. Returning function
- Reset the previous stack frame: %esp = %ebp and %ebp = (%ebp) 
- Jump back to return address: %eip = 4(%esp)




                                    
