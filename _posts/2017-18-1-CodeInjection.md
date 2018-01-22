---
layout: post
title: Code Injection 
category: [programming]
tags: [c,programming, vulnerability, coursera, week1, codeinjection]
---

In the article of Buffer Overflow we see that we can overwrite variables that are already in the stack. For example, functions as
strcpy() only stop when they see a '\0' character. But in the meantime they can overwrite important parts of code. 

It could be random data, but if the user has a malicious intention it can inject code with a Buffer Overflow Vulnerability. 

Example. 

```
void func(char *arg1){
  char buffer[4];
  sprintf(buffer,arg1)
}
```

In order to be successful when injecting code, we need two things 

1) Load our own code into the stack 
2) Make that %eip point to that code 

Challenge 1 - Load own code into the stack 

It is necessary to be "machine code" (already compiled and ready to run). 
It CAN NOT have any zero data. Because functions as sprintf, scanf, gets will stop copying the data. 
The best option, is to run a general purpose shell, since we could have complete access to the system. A code that gives us access 
to the sheel is named "shellcode". 

Example of Shellcode.

```
#include <stdio.h>
int main(){
  char name[2];
  name[0] = "/bin/sh";
  name[1] = NULL;
  execve(name[0],name, NULL);
}
```


Stack until now. 

      | ------- | ------- | ------- | ------------- | ------------- | ------------- | ------------ |
      | ------- | ------- | ------- | ------------- | ------------- | ------------- | ------------ |
      | TEXT    | ....... | 00000000| %ebp          | %eip           | arg1         | \x0f \x3f    |     
      | ------- | ------- | esp     | ------------- | ------------- | ------------- | -----------  |
      | ------- | ------- | ------- | ------------- | ------------- | ------------- | ------------ |


So right now, the challenge is to run the inserted code. 

We can do it, by inserting the beginning memory address of the code in the %eip register. But now, we have other question. 
How do we know the memory address of our code? 

So in order to execute our code, we need to know the return address. 

Important. In this post I do not take into account Address Randomnization. 

Technique 1. 

Try a lot of random addresses, but in this case the address space is very big (2^32 or 2^64 memory addresses available). 

Technique 2. 

Use NOP instruction. It is a single byte instruction that jump to the next instruction. So if the attacket insert a lot of NOP
instructions before the code, it does not matter where the %eip is pointing to, since it will always look to the next instruction
until it reaches a no NOP instruction. 

So if we enter an informated guess in the %eip register, when the funciton returns the programm will run the inserted code. 

For more information, check the post "Smashing the Stack". Which will be a summary of the document created by Aleph One. 

Reference: 
int execve(const char *filename, char *const argv[],char *const envp[]);
Link: http://man7.org/linux/man-pages/man2/execve.2.html 
