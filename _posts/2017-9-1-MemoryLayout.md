---
layout: post
title: Buffer Overflow
category: [programming]
tags: [c,programming, vulnerability, coursera, week1]
---

First of all, we need to define what is a buffer. 

- Buffer: Contigious memory associated with a variable or field
- Overflow: When a buffer try to write more data than the buffer can actually hold. 

Example of Buffer Overflow: 

```
int func(char *str){
  char buffer[4];
  strcpy(buffer,str);
 }
 int main(){
  char *mystring = "CATS!!!";
  func(mystring);
}
```

           
     | ------- | ------------- | ------------- | ------------- | 0xffffffff   |
     | ------- | ------------- | ------------- | ------------- | ------------ |
     |  buffer | ebp           | eip           | str           | CallersData  |     
     | ------- | ------------- | ------------- | ------------- | ------------ |
     | ------- | ------------- | ------------- | ------------- | ------------ |

So, if we see the variable "mystring" has 7 characters but the buffer in func() only allows 4 characters. So, after allocating 
the variables, the stack would look like this. 

     | ------- | ------------- | ------------- | ------------- | 0xffffffff   |
     | ------- | ------------- | ------------- | ------------- | ------------ |
     |  CATS   | !!!\0         | eip           | str           | CallersData  |     
     | ------- | ------------- | ------------- | ------------- | ------------ |
     | ------- | ------------- | ------------- | ------------- | ------------ |


So the stack in hex, would look 


     | ------- | ------------- | ------------- | ------------- | 0xffffffff   |
     | ------- | ------------- | ------------- | ------------- | ------------ |
     |  CATS   | 21 21 21 00   | eip           | str           | CallersData  |     
     | ------- | ------------- | ------------- | ------------- | ------------ |
     | ------- | ------------- | ------------- | ------------- | ------------ |
     
     
    So %ebp will be 0x00212121 and %eip will be 4(%ebp) = 0x212125 that will end in SEGFAULT. 

In the previous example, we could think that it is only a crash and we can fix it. But in fact, Buffer Overflow vulnerability 
has security implications. We can overwrite important data for example in order to pass certain test in a function. 



