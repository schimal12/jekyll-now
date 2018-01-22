---
layout: post
title: Analysis of ShellCode 
category: [programming]
tags: [c,programming, vulnerability, coursera, week1, codeinjection, shellcode]
---

In this part, I am going to summarize the analysis of the shellcode that appear in the Articil "Smashing the Stack" of Aleph One. 

```
#include <stdio.h>
int main(){
  char name[2];
  name[0] = "/bin/sh";
  name[1] = NULL;
  execve(name[0],name, NULL);
}
```

Here will be the debug file. 



Reference: http://insecure.org/stf/smashstack.html
